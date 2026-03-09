
### Complete Technical Documentation





---

## Table of Contents

1. [What Are We Building](#1-what-are-we-building)
2. [The Core Problem This Architecture Solves](#2-the-core-problem-this-architecture-solves)
3. [The Solution  Architecture Overview](#3-the-solution--architecture-overview)
4. [The 7-Stage Pipeline  Full Detail](#4-the-7-stage-pipeline--full-detail)
5. [Flux.1-dev  Internal Architecture](#5-flux1-dev--internal-architecture)
6. [The LoRA System  Brand Consistency](#6-the-lora-system--brand-consistency)
7. [ControlNet Glyph Injection Text Rendering in Detail](#7-controlnet-glyph-injection--text-rendering-in-detail)
8. [Complete Model Inventory](#8-complete-model-inventory)
9. [Datasets](#9-datasets)
10. [Deployment Architecture](#10-deployment-architecture)
11. [Non-Technical Challenges](#11-non-technical-challenges)
12. [Key Research Papers](#12-key-research-papers)
13. [Architecture Decision Summary](#13-architecture-decision-summary)

---

## 1. What Are We Building

A  system that takes a plain English marketing brief and outputs a commercial-quality image  complete with accurate text rendering, brand consistency, and 2K/4K resolution output.

### 1.1 Core Requirements

- Long prompt handling  marketing briefs are verbose, not short prompts
- High resolution output 1024px native, 2K/4K after upscaling
- Reliable text rendering brand names, slogans, taglines must be accurate
- Brand consistency each client has their own visual style baked in
- Fine-tuning support  we can train on client-specific images
- Production inference fast enough for real agency turnaround
- Open-source, self-hosted  no API dependency, full control

### 1.2 Pipeline Type: LLM-Guided Diffusion (Hybrid)

This pipeline is a hybrid architecture. It is not a pure diffusion system and not a pure LLM image generation system. It is a deliberate combination of both.

| Approach | What it is | Why we rejected it |
|---|---|---|
| Pure Diffusion | Stable Diffusion, no LLM | Cannot understand long marketing briefs |
| Pure LLM Image Gen | LlamaGen, etc. | Low photorealism ceiling, slow at high-res |
| Unified Multimodal | GPT-4o, Gemini | Not open-source, no LoRA/ControlNet support |
| **LLM-Guided Diffusion** | **Our approach ✓** | **Best of both: LLM reasoning + diffusion photorealism** |
![[Pasted image 20260306184559.png]]
> **Key Principle:** LLM handles language and reasoning. Diffusion model handles pixel synthesis. Each component does what it is best at.

---

## 2. The Core Problem This Architecture Solves

Before diving into the architecture, it is important to understand the exact problems we are solving. There are three distinct problems that standard AI image generation fails at for marketing use.

### 2.1 Problem 1 — Vague Prompt Interpretation

Marketers write in marketing language. They say things like "warm premium feel" or "clean and luxurious". Diffusion models expect precise, weighted, technical prompt syntax. Without a translation layer, the same brief will produce inconsistent results every time.

> **Example:** Brief says "luxury skincare hero shot". Model needs: `(moisturizer bottle:1.5), luxury editorial photography, (warm diffused studio light:1.3), marble surface, macro lens, shallow depth of field...`

### 2.2 Problem 2 — Broken Text Rendering

This is the deepest and most important problem. When you ask a standard diffusion model to put the words "Made by Lakme" on a product, it will almost always produce garbled, misspelled, or illegible text. This is not a bug  it is a fundamental architectural limitation. It has three root causes.

#### Root Cause A  Character-Blind Text Encoder (CLIP)

The standard text encoder used in most diffusion models is CLIP. CLIP was trained on billions of image-text pairs to understand semantic meaning  the concept of things. It was never trained to understand individual characters or spelling.

For CLIP, "Lakme", "LAKME", and "lakme" are all identical  they all represent the same cosmetics brand concept. The spelling information is discarded entirely at the encoding stage.

#### Root Cause B  Semantic Drift

Even if the encoder passes spelling information forward, the diffusion model itself suffers from semantic drift. When the model sees the token "GOLD", its training associations pull it toward the concept of gold  shiny, yellow, metallic, luxurious. These semantic associations compete with and override the visual instruction to render the letters G, O, L, D.

This is why you see outputs that look gold-coloured and luxurious but do not actually spell GOLD correctly. Concept won over letterform.

#### Root Cause C — VAE Stroke Destruction

Diffusion models do not work directly on pixels. They compress the image into a latent space using a VAE (Variational Autoencoder), do all processing in that compressed space, then decode back to pixels at the end.

The standard VAE used in SDXL compresses images into a 4-channel latent space. This compression was optimised for natural image features: large shapes, gradients, textures. Fine letter strokes — the thin lines and curves that make letters legible  are not well preserved. They blur out during encoding and are poorly reconstructed during decoding.

| Problem | Where it happens | What you see in the output |
|---|---|---|
| Character blindness | CLIP text encoder | Spelling ignored, wrong letters chosen |
| Semantic drift | Diffusion attention layers | Word becomes its concept, not its letters |
| Stroke destruction | VAE compression | Letters are blurry, merged, or missing strokes |

> **Why this matters:** For marketing, text is not decoration — it is the brand. A misspelled brand name on a campaign image is a catastrophic failure, not a minor issue.


# diagram

![[Screenshot 2026-03-06 184748 2.png]]
### 2.3 Problem 3 :- Brand Inconsistency Across Jobs

Every time you run a standard model, it has no memory of who the client is. The same client will get different visual styles across different jobs. Without a system to encode and enforce brand identity at the model level, consistency is impossible.

---

## 3. The Solution — Architecture Overview

Each problem described in Section 2 maps to a specific architectural decision. This is not a collection of random tools — every component exists because a specific failure mode demanded it.

| Problem | Solution | Where in pipeline |
|---|---|---|
| Character-blind encoder | T5-XXL as text encoder | Built into Flux.1-dev |
| Semantic drift | ControlNet glyph injection | Stage 4 — Glyph Renderer |
| VAE stroke destruction | 16-channel VAE | Built into Flux.1-dev |
| Vague brief interpretation | LLM decomposition layer | Stage 2 + Stage 3 |
| Brand inconsistency | Per-client LoRA adapters | Stage 5 — Image Generator |
| Low output resolution | Real-ESRGAN upscaling | Stage 7 — Upscaler |
| Unreliable quality | CLIP + aesthetic auto-retry | Stage 6 — Quality Gate |
![[Pasted image 20260306185121.png]]
---

## 4. The 7-Stage Pipeline Full Detail

The pipeline takes a plain English brief as input and produces an upscaled, quality-checked image as output. Every stage is modular  each can be tested, replaced, or improved independently.

---

### Stage 1 — User Brief
*Plain English input from the marketer*

- Input: a natural language description of the required image
- No special syntax or formatting required from the user
- Can include product description, style references, text to appear in image, mood, etc.
- Example: *"Luxury skincare hero shot on marble, warm golden light, Made by Lakme on the label"*

---

### Stage 2 — Brief Decomposer
*LLM converts brief into structured JSON schema*

- **Model:** Mistral 7B or LLaMA 3.1 8B (running locally via Ollama)
- Takes the raw brief and outputs a structured JSON with isolated fields
- Fields extracted: `subject`, `style`, `lighting`, `camera`, `mood`, `background`, `colors`, `text_to_render`, `negative`
- Key function: isolates `text_to_render` as its own field — critical for Stage 4
- Fallback: rule-based decomposition if LLM is unavailable

**Example output:**
```json
{
  "subject": "skincare moisturizer bottle",
  "style": "luxury editorial photography",
  "lighting": "warm diffused golden studio light",
  "camera": "macro lens, shallow depth of field",
  "mood": "clean, premium, minimal",
  "background": "marble surface",
  "colors": "warm gold tones",
  "text_to_render": "Made by Lakme",
  "text_font_style": "serif elegant",
  "negative": "clutter, harsh shadows, distortion, blurry, low quality"
}
```

> By isolating `text_to_render` into its own field, we ensure the text is handled through the glyph pipeline (Stage 4) rather than being left to the diffusion model to hallucinate.

---

### Stage 3 — Prompt Composer
*LLM assembles weighted diffusion prompt from schema*

- Same LLM as Stage 2, different system prompt
- Converts the structured schema into a properly weighted Flux prompt
- Applies attention weights: main subject gets `1.4–1.6`, supporting elements `1.1–1.3`
- Injects text rendering instruction if `text_to_render` is present
- Always appends quality suffix: `sharp focus, high resolution, commercial photography, 8k`
- Builds negative prompt with base negatives always included

**Example positive prompt:**
```
(moisturizer bottle:1.5), luxury editorial photography, (marble surface:1.3),
(warm golden light:1.4), macro lens, shallow depth of field, (clean minimal:1.2),
sharp focus, high resolution, commercial photography, 8k
```

**Example negative prompt:**
```
blurry text, garbled letters, wrong spelling, illegible text, blurry, low quality,
watermark, text artifacts, distortion, overexposed, amateur photography
```

---

### Stage 4 — Glyph Renderer ⭐
*Pre-renders text as clean image for ControlNet — the text rendering fix*

- Only runs when `text_to_render` field is populated
- Renders the exact text string in the appropriate font using system font files
- Output: clean black text on white background, 1024×1024
- Font selection based on style hint from brief (`serif elegant`, `bold sans`, etc.)
- Result is cached to disk — same text+font combo is never rendered twice
- This glyph image is fed to ControlNet in Stage 5

> **Why this solves semantic drift:** We are no longer asking the model to generate letters. We give it the exact strokes it needs to follow. The model's job becomes: take this spatial structure and apply style, colour, and texture to it. Semantic drift becomes impossible.

---

### Stage 5 — Image Generator
*Flux.1-dev + LoRA + ControlNet + IP-Adapter*

- **Core model:** Flux.1-dev (12B parameters, DiT architecture)
- **Text encoders:** CLIP-L + T5-XXL dual encoder — T5 handles spelling, CLIP handles concepts
- **VAE:** 16-channel — 4× more information capacity than SDXL, strokes survive compression
- **ControlNet:** receives glyph image and enforces letter shapes during every denoising step
- **LoRA:** per-client adapter loaded at inference — hot-swappable between clients
- **IP-Adapter:** optional brand reference image for visual conditioning
- **Hardware-aware:** runs on CPU for dev, GPU for production, no code changes required

**Why Flux.1-dev over SDXL:**

| Feature | SDXL | Flux.1-dev |
|---|---|---|
| Architecture | UNet (older) | DiT — Diffusion Transformer |
| Text encoder | CLIP only (character-blind) | CLIP + T5-XXL (character-aware) |
| Prompt length | 77 tokens max | 4096 tokens |
| VAE channels | 4 channels | 16 channels |
| Text rendering | Poor | Significantly better |
| Photorealism | Good | State of the art |

---

### Stage 6 — Quality Gate
*CLIP score + Aesthetic score + automatic retry*

- Runs two scoring models on every generated image
- **CLIP score:** measures how well image matches the prompt (threshold: `0.25` minimum)
- **Aesthetic score:** measures visual quality using LAION Aesthetic Predictor (threshold: `5.5/10`)
- If either score fails: automatically retries with a new random seed
- Maximum 3 retries — if all fail, returns best image with a quality warning
- Both scoring models are lightweight and run fine on CPU
- Thresholds are configurable per client in their config file

---

### Stage 7 — Upscaler + Output
*Real-ESRGAN 2K/4K upscaling + metadata logging*

- **Upscaler:** Real-ESRGAN  GAN-based upscaling trained on photorealistic images
- Default: 2× upscale (1024px → 2048px). Option for 4× (1024px → 4096px) for print
- Fallback: PIL Lanczos if Real-ESRGAN is not installed
- Output: PNG image saved to `/outputs/` with job ID in filename
- **Metadata:** full JSON record saved alongside each image  brief, prompt, schema, scores, seed, model, duration, client ID, LoRA used


[flow you can see by clicking this link](https://mermaid.live/edit#pako:eNqNVM2OKkUUfpWTuou7GGDoX6BjvGH4mWCAQXpGM96-mdR0H6AzTRWprnZAhsSVS03UuDO600dw4dPcF9BH8HQ3MIzmJtYCqqu-833fOaeqtiyUETKPzRL5GC640nDdDQTQGIzfngfs719--BluUlRwoWKcBYF4PczWmdpA-hCLkCuswIhHCPcbGPKHJYIUkPB7TF4H7PxdIA5kUK1-DL55WEiz-7niqwUMh6O7Yfu2N4W3udpvv-YrRLUhyfdf_0gzMc_4HGEgNCZJPEcRYsDelTT58E2K9HWOMb3SJnQxlMuVJN_keBSnWvEEGhdwTux81AarZtBGiX3_zffwiX81Bj9c4JK_5LaO3JYHE0WkGjrP1GVIQfE5xvOFxgj6SbbeQ__ls6yBVS6hiA7F8K1ip3M17m41rvWdlneK9guJlcIUhX6zO6BzWI5_usX0CXz76ND24DLZUE2nRWwR3ZdCQ8lVuOwkyAXMC1i8pKi8DAkPHyAXzpv3uIg1glE37XX-c0zhKDuWpOr8p5HdQb9_2slvf4duPJtlaUykz_38LE4z6oW_EXqBaZy-LJFzTMbxYJD7g0sUqLiWRTZU2ppRjfBLOIOhnLbpr0MZKpmMURPgDAaTajviK03ZH5hPK20XKTzHQChFFGvyGIv5aV6-U3bLPTpyPfiUrMd6A5dc54XrDAcTOjVSIfloY0oZ6TgsV47qvltIljAtJdBNe4Jp73p6u52iVps3xMS1xvxofQTWrgwrAIc2V0DgI6SIUWnxJWAsK9RinSkB92SCII1jFqX4hKdpEds4ZtPw4GaVhjyhvpzBVaZXWV7AKfKk2vOnl-0xmGuQCux1UdcRah5xzYurUiS3V2gUdbq6uS6fi5_-_OuP76AfC2ryYH_AzLrdXJVk9ZZLs8n48vR1SPUmwfyNmMVJ4r2qd40Ls10JZSKV92o2m52iSGgPM-oXrabxARg1ukT1-jaND6Hc_8P1_ESV4H6vb3Uap4iTs79X7fZa_R6rsLmKI-ZplWGFLVEtef7JtnlwwOi8LOmgeDSNuHoIWCB2FLPi4gspl4cwJbP5gnkznqT0la2oC9iNOd265XG1vOEdmQnNvIbVKkiYt2Vr5hmuWXPrdctxDatlGLbdqLAN86pNq9ay6vWm6zRds2U6rV2FfVXoGjWz5TQcxzGtVsOxmqa9-wcu1vUl)

---

## 5. Flux.1-dev — Internal Architecture

Flux.1dev is the backbone of the generation stage. Understanding its internals is important because several of its design choices are specifically why it was chosen for this use case.

### 5.1 DiT — Diffusion Transformer

Traditional diffusion models use a UNet architecture. Flux uses a DiT Diffusion Transformer. The key difference is how text and image information interact.

In UNet-based models, text conditioning is applied at specific cross-attention layers. The text information is injected at particular points but does not participate in the full processing.

In the DiT, text tokens and image tokens are processed together in the same attention matrix at every layer throughout the entire network. This deep integration means character-level information from T5 is present throughout the entire image formation process, not just at injection points.

### 5.2 Dual Text Encoder Strategy

Flux uses two text encoders simultaneously and combines their outputs.

| Encoder | What it contributes | Why it is needed |
|---|---|---|
| CLIP-L | Semantic meaning — what things are, visual concepts, style associations | Guides overall composition and aesthetics |
| T5-XXL | Character-level structure — exact spelling, word boundaries, sequence | Ensures `text_to_render` information is character-accurate |

> **Combined signal:** CLIP says "this is a luxury skincare brand called Lakme". T5 says "the characters are L-a-k-m-e in this exact sequence". Both signals are projected and fed to the DiT together, giving the model both semantic and structural understanding.



### [5.3 Flow Matching Training](https://mermaid.live/edit#pako:eNp9k9FO2zAUhl_lyFxwU1BbJ01jaZNICxsSUC46IW2ZkJuctFkTO3Ic2g5xubs9wvZye5IdJzBRNuYb2_Hv38ff79yzRKfIBMsKvUlW0liYT2MF1OpmsTSyWsHN-fz97MMcPsXs18_vcJPblW4svCt2tHiuvmBic61i9rnb59rJgMTXRpeVFXC4MblFuJQpwmIHF3Jd4uG-fEjyycX5tehW4Q0sjFQpJFolWNl9MSfxibWo3LECaiwlDRMoUapcLeNYpbrMlbRYg75DAwWS2kC9khXW-14eec0aWzWuzkuJ6VOF_BDosi8uBUdHb6nYruNd53UCVGms_gGupfbjW0vtv8gih8zi1t5afWvIDk1b0qvQIgftTCsLndrpq3yLxVGFJqMTCMSikMkanCtoBZuVyyGR6k7uY4gc0glZGV1cIaFAlWmTED_cytaopqU1whJ1idbsAInrDmqL1b6R43lWNFsBsqqKnBxquyuwR0kWujE9snLVNAZhdgXz2TXoDDrzFyX5-8k8xwAE9AW7NouoSybqkom8rvP_DsiVRMlBlheFODg986j1XIVGHGRZ9lwV-Y-qQT8Kx4PXVMNHVb8fedPxayr-qAom_OR0-kzFemxp8pQJaxrssRJNKd2U3bv99C5WWGLMBA1TadYxi9UD7amk-qh1-bTN6Ga5YiKTRU2zpkrpF5jmkh5j-edr91YmulGWiXDQejBxz7ZM-P6xFwQjbxB4fMD7o8DvsR0TgzA89sdhGPS553Gvz4cPPfa1PbZ_HHLOQz4KhyPfGwV8_PAb-G1UoQ)

Flux was not trained using the traditional DDPM objective. It uses Flow Matching — a more recent and efficient training paradigm.

Practically, this means Flux produces high-quality results in fewer denoising steps. Flux.1-schnell (the fast variant) produces good results in just **4 steps**. Flux.1-dev produces excellent results in **20–30 steps**. This makes production inference significantly faster compared to SDXL which typically needed 30–50 steps.

---

## 6. [The LoRA System — Brand Consistency](https://mermaid.live/edit#pako:eNp9k81ymzAQx19lR7naDEI4GA6dAdtp3SHxTJxeGnqQQdhMQGKEaJIm6fQh-oR9ki4Yx8lMEp12l__-9gPpgaQqEyQgealu0x3XBq7miQQ8UbheXCfkrGzvLDrOxE-IeCPgHPVlkkjqRFBzzSthhG4wECueiQyUTAVwA41BWFsn5Ecij0AYjz_B8uLseilzoUWnXchtIcWzqmk3W83rHcSry3AN2EGsLkMIM153heDfn7_wRZlxc8vrmm9K0VeA4cQ2JvR9hqIxO2GKFLp87O83te3zCI0rzbHg0KmSEIfL1cU41OY1iSJpVhZCGgj7qnF7J2B9U8iUa_E2EJgNG81lBkXFt6J5TXSOxKgnrmuFC4-6hHd4zuQDHjvyZj3Psqwj5yAVCB92G9uH_Q8-7fzHr2oDudJwmPbxhcJ5SxG9VLAjcx9Bsw-tvl1hg5-FFJobHGfZjYD93RZmB-metB-uMffDfxzuQOfv70telGVwYs9p5ISjVJVKByd5nr_UdfX2Mm_GwsX8HRm2M8ioHflT-o4MdzLUtCN3Pj2oaEhDZ_FK6HwgJCOy1UVGAqNbMSKV0BXvXPLQIRKCV7PCiQM0M65vEpLIJ8ypufyuVHVI06rd7kiQ87JBr60z3OO84Pg-qucoPqJM6JlqpSGBP-kZJHggdySYTCzX805d6rmMMvvUw6_3JKC-b02mvu_ZzHWZazPnaUR-9WVty2eM-cyjp1PqUMamT_8BgppFiw)

LoRA (Low-Rank Adaptation) is how we achieve per-client brand consistency without storing and running a separate full model for each client.

### 6.1 What LoRA Does

When we train a LoRA on a client's brand images, we are not retraining the entire 12-billion parameter Flux model. Instead, we train a small set of additional weight matrices — typically 50–200MB — that modify the model's behaviour for that client's specific visual style.

At inference time, we load the base Flux model once (expensive) and then hot-swap a different LoRA for each client's job (cheap — a few seconds). This is the only practical way to serve multiple clients at scale without running multiple GPU instances.

### 6.2 Three Types of LoRA in Our Pipeline

| LoRA Type | What it learns | Training data | Training time | When trained |
|---|---|---|---|---|
| Base Aesthetic LoRA | Overall quality and photographic style baseline | 500–2000 images (LAION-Art + Unsplash) | 2–4 hours on A100 | Once, before launch |
| Client Brand LoRA | Specific client's colours, product shapes, visual identity | 20–50 curated brand images per client | 30–60 mins on A100 | Once per client onboarding |
| Style LoRA (optional) | Specific art direction styles | Curated style-specific images | 1–2 hours on A100 | On demand |

### 6.3 Client LoRA Registry

Each client has a configuration file stored in the `client_configs/` directory. This contains their LoRA file path, LoRA strength, style overrides to append to every prompt, and additional negative prompt terms specific to their brand.

**Example client config:**
```json
{
  "client_id": "luxe_skincare",
  "client_name": "Luxe Skincare Co.",
  "lora_file": "luxe_skincare.safetensors",
  "lora_scale": 0.90,
  "style_override": "warm golden tones, marble textures, minimalist luxury",
  "negative_additions": "plastic backgrounds, harsh lighting, cluttered composition",
  "clip_threshold": 0.27,
  "aesthetic_threshold": 6.0
}
```

---

## 7. ControlNet Glyph Injection — Text Rendering in Detail

This is the most important non-standard component in the pipeline.

### 7.1 The Standard Approach (Broken)

1. Model reads "Made by Lakme" in the prompt
2. CLIP encodes it as a semantic concept (Lakme brand identity)
3. T5 encodes the character sequence, but attention may not preserve it to image
4. Diffusion process interprets "Lakme" through its training associations
5. Output: something that looks like Lakme branding but may not spell it correctly

> Even with Flux's improved T5 encoder, purely prompt-based text rendering is not reliable enough for professional marketing output where exact brand spelling is non-negotiable.

### 7.2 Our Approach (Reliable)

1. Brief Decomposer extracts `text_to_render`: `"Made by Lakme"`
2. Glyph Renderer renders this text in the appropriate font on a white canvas — pure font rendering, no AI involved
3. The resulting glyph image is 1024×1024, black text on white, pixel-perfect
4. This image is passed to ControlNet as spatial conditioning
5. ControlNet enforces the stroke structure at every denoising step
6. Flux adds style, colour, texture, integration — but cannot deviate from the given stroke geometry
7. Result: "Made by Lakme" is spelled correctly, styled appropriately, integrated into the image

> **The core insight:** We separated the typographic problem (which computers have solved perfectly for decades via font rendering) from the artistic problem (which diffusion models solve well). We use the right tool for each sub-problem.

### 7.3 ControlNet Scale Parameter

The `controlnet_scale` parameter (default `0.75`) controls how strongly ControlNet's spatial conditioning influences the output.

| Scale | Effect |
|---|---|
| `1.0` | Very strictly follows the glyph structure |
| `0.75` | Recommended balance — reliable text, natural integration |
| `0.5` | Loose following — more creative freedom, less reliable |

---

## 8. Complete Model Inventory

| Model | Role in Pipeline | Parameters | Needs Training? | Source |
|---|---|---|---|---|
| Mistral 7B / LLaMA 3.1 8B | Brief Decomposer + Prompt Composer | 7–8B | Light fine-tune (once) | HuggingFace / Ollama |
| Flux.1-dev | Core image generation | 12B | No — frozen | Black Forest Labs |
| T5-XXL | Character-aware text encoding | 11B | No — frozen | Built into Flux |
| CLIP-L | Semantic text encoding | ~400M | No — frozen | Built into Flux |
| Base Aesthetic LoRA | Quality baseline | ~100MB | Yes — once before launch | You train this |
| Client Brand LoRA | Per-client visual identity | ~100MB each | Yes — per client | You train per client |
| ControlNet (Flux Union) | Glyph spatial conditioning | ~1.2B | No — frozen | InstantX / HuggingFace |
| IP-Adapter | Brand reference conditioning | ~100M | No — frozen | Tencent AI Lab |
| CLIP Scorer | Quality gate — prompt adherence | ~400M | No — frozen | OpenAI (open weights) |
| LAION Aesthetic Predictor | Quality gate — visual quality | ~300M | No — frozen | LAION |
| Real-ESRGAN | 2K/4K upscaling | ~17M | No — frozen | xinntao / GitHub |

> **Training summary:** You train exactly 3 things: the LLM (lightly, once), the Base Aesthetic LoRA (once before launch), and a Client Brand LoRA for each client onboarding. Everything else is pre-trained and frozen.

---

## 9. Datasets

### 9.1 Public Datasets

| Dataset | Size | Used For | License | How to Get |
|---|---|---|---|---|
| LAION-Art (score > 8) | ~8M images | Base Aesthetic LoRA training | Open | img2dataset from HuggingFace |
| Amazon Berkeley Objects | 400K+ images | Product photography LoRA | CC BY-NC 4.0 | Direct download |
| Unsplash Lite | 25K images | Lifestyle photography supplement | Commercial OK | Unsplash API |
| Open Images V7 | 9M images | Object coverage breadth | CC Attribution 2.0 | Open Images download tool |
| Pexels / Pixabay | Unlimited | Supplemental high quality | CC0 | API or direct download |

### 9.2 Datasets You Build Yourself

| Dataset | Size | Used For | How to Build |
|---|---|---|---|
| LLM fine-tune pairs | 500–1000 examples | Brief Decomposer training | Generate synthetic examples via Claude/GPT-4: brief → schema pairs |
| Client brand dataset | 20–50 images per client | Client Brand LoRA training | Structured onboarding: collect curated brand images with captions |

---

## 10. [Deployment Architecture -- click for architecture](https://mermaid.live/edit#pako:eNp1kmFv2jAQhv_Kyf04yhIcGpIPkygNXbus0ACqtGWqTHKBtI7NHGctK_z3OQmrmNb6k31-77n3fH4hiUyR-CTj8ilZM6UhjGIBZo3Cq-Bm_j0mI56j0PARFlcx-QGnp592n-fzKUwnszksVY7ZDobTK6Mcs1KbHUSBuTEbI29ZdbDOQ_GzwgrhQS53cLsIFoHJijDNS7htLoblViRwLZcw01LhK6DR1gi4m0Rfgqi2hRzVFu6kekQFE85ZwSAMv4559QzTfIM8F2iwjOd6C5dMIyw2ZcJM1iu2hTXWSvYLIS_YCuEDFKhZyjTbwWw-iYaXtc0ZhcnyARPdWDO6tyimMUhkseGosXmVf_tXqCslDmUWUbg7vHIs_iNxyVIIZTTcQWgsGAPcVL1_wny11iVMUZ0m7WC6JctMOVFKVb7lqSElUmT5ytQbXxpUm3nfBssj1vVsctMwWkqptxwPHiHLOfdPrAv7vDfsJNLY8U-yLDtW1n22snHfC6zzd2TtNFthMHbMekd46KJVuiM6DC7eUR7mdJDa1rk3sI-kpENWKk-Jr1WFHVKgKlh9JC81JCZ6jYUZqG-2KVOPMYnF3uRsmPgmZfE3TclqtSZ-xnhpTtXGfBG8yNlKseI1qlCkqEayEpr4PcttIMR_Ic_E7_e7juueObbrUJtaZ26_Q7bEtz2v2x94nmtRx6GORXv7Dvnd1LW6HqXUo659NrB7NqWD_R_J1Cbz)

### 10.1 Hardware Requirements

| GPU | DEVICE_MODE setting | Speed | Use case |
|---|---|---|---|
| A100 40GB / RTX 4090 | `cuda` | 15–30 sec / image | Production — cloud deployment |
| RTX 3090 / A6000 | `cuda` | 20–45 sec / image | Production — cloud deployment |
| RTX 4080 / 3080 (8–16GB) | `cuda_low` | 60–90 sec / image | Low VRAM production |
| GT 730 / no GPU | `cpu` | 20–60 min / image | Development and testing only |

> **Important:** CPU mode exists for testing pipeline logic and development. It is not suitable for production. For real client work, use a cloud GPU provider such as RunPod, Lambda Labs, or vast.ai.

### 10.2 Infrastructure Stack

| Component | Role | Why this choice |
|---|---|---|
| ComfyUI | Workflow engine for Flux | Native Flux support, LoRA hot-swap, API mode, active community |
| Celery + Redis | Async job queue | Image generation takes 15–60s — requests cannot block synchronously |
| FastAPI | REST API layer | Simple, fast, Python-native, async-friendly |
| S3 / Object Storage | Image and metadata storage | Scalable, durable, easy CDN integration |
| Ollama | Local LLM runner | Runs Mistral/LLaMA locally, simple API, no GPU required for LLM |

### 10.3 Multi-Client Strategy

Each client in the system has:
- A JSON config file with their preferences and thresholds
- A LoRA adapter file (`.safetensors`) in the `lora_weights/` directory
- Optional IP-Adapter reference images for visual conditioning
- Custom prompt overrides that are always appended to their generations

The pipeline loads the 12B Flux model once at startup and keeps it in memory. For each client job, it hot swaps only the LoRA (a few seconds) rather than reloading the entire model (several minutes). This is the critical optimisation that makes multi-client production scale viable.

---

## 11. Non-Technical Challenges

The technical architecture is the easier part. These four challenges are where most production image generation systems actually fail.

### 11.1 Client Data Quality

A client LoRA is only as good as the images used to train it. Clients often want to provide 5 images. The pipeline needs 20–50 high-quality, diverse, representative images. Structured onboarding with clear requirements for what images to provide — and what makes a good training image  is essential and must be built before client work begins.

### 11.2 Quality Gate Calibration

The CLIP score and aesthetic score thresholds are not universal. An abstract artistic campaign image might score lower on CLIP similarity but be exactly what the client wants. A technically sharp but compositionally boring product shot might score high but be rejected by the client. Human feedback loops are needed early to calibrate per-client thresholds against actual client approval rates.

### 11.3 Inference Cost at Scale

On an A100, Flux.1-dev costs approximately $0.25–0.75 per image in cloud GPU time depending on resolution and number of inference steps. At volume  hundreds of images per day  this adds up fast. Pricing models for clients must account for this. The architecture supports using Flux.1-schnell as a cost optimisation at the expense of some quality ceiling.

### 11.4 Client Expectations

Current open-source models produce excellent results but are not pixel perfect for every use case every time. For high-stakes campaigns —product launch hero images,  website visuals  human review is recommended before delivery. The pipeline should be positioned as a powerful creative acceleration tool, not a fully autonomous system that removes human review entirely.

---

## 12. Key Research Papers

| Paper | What it covers | Relevance to our pipeline |
|---|---|---|
| [FLUX Technical Report (2408.06072)](https://arxiv.org/abs/2408.06072) | Flux.1 architecture, DiT, flow matching, dual encoder | Core model — the backbone of Stage 5 |
| [Attention Is All You Need (1706.03762)](https://arxiv.org/abs/1706.03762) | Original transformer architecture | Foundation for T5, CLIP, and DiT |
| [LoRA (2106.09685)](https://arxiv.org/abs/2106.09685) | Low-rank adaptation for fine-tuning | Our entire brand consistency system |
| [ControlNet (2302.05543)](https://arxiv.org/abs/2302.05543) | Spatial conditioning for diffusion models | The glyph injection mechanism in Stage 4 |
| [AnyText (2311.03054)](https://arxiv.org/abs/2311.03054) | Multilingual text rendering in diffusion | Direct reference for our text rendering approach |
| [GlyphDraw2 (2407.02252)](https://arxiv.org/abs/2407.02252) | LLM + diffusion for poster generation | Closest existing work to our full pipeline |
| [TextDiffuser (2305.10855)](https://arxiv.org/abs/2305.10855) | Character-level segmentation for text | Alternative text rendering approach |
| [IP-Adapter (2308.06721)](https://arxiv.org/abs/2308.06721) | Image prompt adapter for reference conditioning | Our brand reference image conditioning |
| [Real-ESRGAN (2107.10833)](https://arxiv.org/abs/2107.10833) | GAN-based photorealistic super-resolution | Stage 7 upscaling |
| [CLIP (2103.00020)](https://arxiv.org/abs/2103.00020) | Contrastive image-text learning | Quality gate scoring + explains why T5 is needed |

---

## 13. Architecture Decision Summary

| Decision | What was chosen | Why |
|---|---|---|
| Pipeline type | LLM-Guided Diffusion Hybrid | Only approach combining LLM reasoning with diffusion photorealism in open-source |
| Core model | Flux.1-dev over SDXL | T5 encoder, 16-ch VAE, DiT joint attention — all required for text rendering |
| Text rendering | ControlNet glyph injection | Only reliable method — removes semantic drift entirely |
| Text encoder | T5-XXL + CLIP dual | T5 handles spelling, CLIP handles concepts — both needed |
| Brand consistency | LoRA per client | Cheap to train, hot-swappable, no full model per client |
| LLM choice | Mistral 7B / LLaMA 3.1 8B | Best quality/size ratio, runs locally on CPU via Ollama |
| Quality control | CLIP + Aesthetic + retry | Two independent signals are more reliable than one |
| Upscaling | Real-ESRGAN | GAN-based, trained on photorealistic images, fast |
| Inference engine | ComfyUI | Best Flux support, LoRA hot-swap, active community, API mode |
| Job queue | Celery + Redis | Generation is slow — async is mandatory for production |

---

> **Final Assessment:** Within the open-source ecosystem as of 2026, this is the best available architecture for production-grade marketing image generation with reliable text rendering and consistency at scale. The only approaches that may exceed it are proprietary closed systems (Adobe Firefly, Midjourney internals) which are not available for self-hosted deployment.
