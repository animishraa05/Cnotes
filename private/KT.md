# Knowledge Transfer Document

## AI-Powered Image Generation Pipeline

-----

> **Prepared by:** Animesh Mishra  
> **Date:** 25 March 2026  
> **Project:** AI-Powered Poster Caption Generator & Image Generation Pipeline  
> **Classification:** Internal Engineering Documentation

-----

##  Table of Contents

1. [[#Project Overview]]
2. [[#Project Structure]]
3. [[#Core Components]]
4. [[#Caption Generation System]]
- [[#Architecture Overview]]
- [[#Multi-OCR Fusion Strategy]]
- [[#Regional OCR — The 12-Crop Strategy]]
- [[#Text Validation Pipeline]]
- [[#LLM Cleanup with Ollama Mistral 7B]]
- [[#Domain Detection System]]
- [[#Style Tag Generation]]
- [[#Caption Format for Flux LoRA Training]]
- [[#Configuration Parameters]]
1. [[#Caption Generation — Technical Deep Dive]]
- [[#Preprocessing Pipeline]]
- [[#Fuzzy Deduplication Algorithm]]
- [[#Fragment Merging]]
- [[#Position Tracking System]]
- [[#Quality Scoring]]
- [[#Progress Tracking System]]
1. [[#Image Generation Pipeline — Why Text Fails in Diffusion Models]]
- [[#Layer 1 — The Text Encoder Problem]]
- [[#Layer 2 — Semantic Drift in Attention Mechanisms]]
- [[#Layer 3 — VAE Compression Destroys Fine Details]]
- [[#Layer 4 — The Denoising Process Struggles with High-Frequency Details]]
- [[#Summary of Four Problems]]
1. [[#The Multi-Layered Solution Architecture]]
- [[#Layer 1 Fix — Replace CLIP with T5-XXL]]
- [[#Layer 2 Fix — Glyph Pre-rendering + ControlNet]]
- [[#Layer 3 Fix — 16-Channel VAE]]
- [[#Flux.1-dev — All Three Fixes Combined]]
1. [[#Our Pipeline — The Complete Solution]]
- [[#Stage-by-Stage Breakdown]]
- [[#Simple Analogy]]
1. [[#Image Generation Pipeline — Implementation Details]]
- [[#Soft Mask Generator]]
- [[#Dynamic ControlNet Scale]]
- [[#Conditional Contrast Boost]]
- [[#Color Instructions in Schema]]
1. [[#Final Pipeline Architecture]]
2. [[#Setup & Installation]]
3. [[#Configuration Reference]]
4. [[#Usage Guide]]
5. [[#Troubleshooting & Best Practices]]
6. [[#Future Improvements & Next Steps]]
7. [[#Quick Reference & Cheat Sheet]]

-----

##  Project Overview

This project consists of **two main systems**, each solving a distinct problem in the AI marketing image workflow.

-----

### System 1 — Universal Poster Caption Generator (`caption_runner.py`)

A **batch processing system** that generates training captions for **Flux LoRA models** from marketing posters and images.

> **Why this exists:** To create high-quality, structured training data for fine-tuning image generation models on specific visual domains like wedding invitations, festival posters, and sale advertisements.

**Core Approach:**

- Uses a **multi-OCR strategy** (4 parallel sources) to detect all visible text
- Combines OCR with **LLM-based cleanup** (Mistral 7B via Ollama) to correct errors
- Outputs structured captions in a format specifically optimized for Flux LoRA training

**Supported Domains:** Wedding invitations · Festival posters · Sale advertisements · Concert posters · Restaurant menus · Real estate promotions

-----

### System 2 — AI Marketing Image Pipeline (`pipeline/scripts/`)

A **two-pass Flux + ControlNet pipeline** that generates marketing images from plain-language text briefs with **guaranteed text readability** through OCR verification.

> **Why this exists:** Standard diffusion models cannot reliably render readable text. This pipeline solves that fundamental limitation through architectural workarounds.

**Core Approach:**

- Pass 1: Generate a clean background image without any text
- Pass 2: Inject pre-rendered text glyphs via ControlNet to guarantee accuracy
- Verify: Use OCR to confirm text is readable; retry automatically if not

-----

##  Project Structure

```
Animesh-Diffusion/
│
├── caption_runner.py          #  MAIN: Batch caption generation entry point
├── caption_test.py            #  DEV: Interactive test/debug tool for individual images
├── split_batches.py           #  UTIL: Splits image folder into batches of 100
│
├── caption_progress.json      #  AUTO-GENERATED: Tracks which images are processed
├── retry_needed.txt           #  AUTO-GENERATED: Images that need a second pass
├── failed_images.txt          #  AUTO-GENERATED: Images that could not be processed
│
├── batch_1/                   #  Image batches (1 through 67)
├── batch_2/                   #    Each contains up to 100 images
├── ...                        #    Created by split_batches.py
├── batch_67/
│
├── models/
│   ├── flux-dev/              # 🤖 Flux Dev base model (black-forest-labs/FLUX.1-dev)
│   ├── controlnet-union/      # 🤖 InstantX ControlNet Union model for shape enforcement
│   └── realesrgan/            # 🤖 RealESRGAN x2+ weights for upscaling output images
│
├── fonts/                     #  Custom font files (.ttf, .otf) for glyph rendering
│
├── pipeline/
│   ├── config.yaml            #  All pipeline configuration — edit this to tune behavior
│   ├── outputs/               #  Final generated images are saved here
│   └── scripts/
│       ├── pipeline_runner.py #  MAIN: Pipeline entry point — run this to generate images
│       ├── flux_generator.py  #  Core Flux+ControlNet two-pass generation logic
│       ├── glyph_renderer.py  #  Renders text as clean images using system fonts
│       ├── llm_decomposer.py  #  Breaks user brief into structured layout schema via LLM
│       ├── quality_gate.py    #  OCR verification and conditional contrast enhancement
│       ├── upscale-env/       #  Separate Python venv for RealESRGAN upscaler
│       └── gen-env/           #  Separate Python venv for Flux generation
│
├── flash_attn/                #  Flash attention compatibility layer (performance)
├── sv/                        #  Legacy directory — purpose unknown, do not delete
└── logs/                      #  Runtime log files from pipeline execution
```

> **Note on virtual environments:** Two separate Python environments (`gen-env` and `upscale-env`) are intentional. Flux and RealESRGAN have conflicting dependency requirements that cannot coexist in the same environment.

-----

##  Core Components

### System 1 — Caption Generator

|File               |Role                             |When to Use                                  |
|-------------------|---------------------------------|---------------------------------------------|
|`caption_runner.py`|Main batch processing entry point|Production runs — processes all images       |
|`caption_test.py`  |Interactive testing and debugging|Development — test individual images         |
|`split_batches.py` |Image organization utility       |Pre-processing — organise images into batches|

**Models Used by Caption Generator:**

|Model                 |Purpose                                               |Size  |
|----------------------|------------------------------------------------------|------|
|**Florence-2-base-ft**|Visual description + OCR with spatial region detection|~1GB  |
|**EasyOCR**           |Fast multi-pass OCR supporting English and Hindi      |~200MB|
|**PaddleOCR**         |High-accuracy OCR for dense text (optional dependency)|~500MB|
|**Ollama Mistral 7B** |LLM for text cleanup and OCR error correction         |~4GB  |

-----

### System 2 — Image Generation Pipeline

|File                |Role                                       |When to Use                                 |
|--------------------|-------------------------------------------|--------------------------------------------|
|`pipeline_runner.py`|Main orchestration entry point             |Run this to generate a marketing image      |
|`flux_generator.py` |Two-pass Flux + ControlNet generation logic|Internal — called by pipeline_runner        |
|`glyph_renderer.py` |Pure font rendering for text shapes        |Internal — creates the ControlNet input     |
|`llm_decomposer.py` |Converts user brief to layout JSON via LLM |Internal — called at pipeline start         |
|`quality_gate.py`   |OCR verification and contrast enhancement  |Internal — validates each generation attempt|

**Models Used by Image Pipeline:**

|Model                          |Purpose                                   |Size |
|-------------------------------|------------------------------------------|-----|
|**Flux Dev**                   |Base text-to-image diffusion model        |~23GB|
|**ControlNet Union (InstantX)**|Forces model to follow glyph shapes       |~6GB |
|**RealESRGAN x2+**             |Upscales final output to higher resolution|~67MB|

-----

##  Caption Generation System

### Architecture Overview

The caption generation system works by running **four independent OCR passes** on every image, then intelligently merging and cleaning the results.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        INPUT IMAGE                                   │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
               ┌─────────────────────────┐
               │  Adaptive Preprocessing  │
               │  (CLAHE, gamma, sharpen) │
               └────────────┬────────────┘
                            │
           ┌────────────────┼────────────────┬───────────────────┐
           ▼                ▼                ▼                   ▼
    ┌─────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐
    │ Florence-2  │  │   EasyOCR    │  │  PaddleOCR   │  │   Regional OCR   │
    │ Full Image  │  │ Full Image   │  │ Full Image   │  │  12 Overlapping  │
    │ + Regions   │  │ EN + Hindi   │  │ (Optional)   │  │  Crops @ 4x      │
    └──────┬──────┘  └──────┬───────┘  └──────┬───────┘  └────────┬─────────┘
           └────────────────┴─────────────────┴──────────────────┘
                                        │
                                        ▼
                          ┌─────────────────────────┐
                          │   Merge All OCR Sources  │
                          └────────────┬────────────┘
                                       │
                                       ▼
                          ┌─────────────────────────┐
                          │   Fuzzy Deduplication    │
                          │   (RapidFuzz, 80% ratio) │
                          └────────────┬────────────┘
                                       │
                                       ▼
                          ┌─────────────────────────┐
                          │   Fragment Merging       │
                          │   (Context-aware)        │
                          └────────────┬────────────┘
                                       │
                                       ▼
                          ┌─────────────────────────┐
                          │   Ollama LLM Cleanup     │
                          │   Mistral 7B             │
                          └────────────┬────────────┘
                                       │
                                       ▼
                          ┌─────────────────────────┐
                          │   Domain Detection       │
                          │   (Pattern matching)     │
                          └────────────┬────────────┘
                                       │
                                       ▼
                          ┌─────────────────────────┐
                          │   Style Tag Generation   │
                          │   (Auto-hashtags)        │
                          └────────────┬────────────┘
                                       │
                                       ▼
                          ┌─────────────────────────┐
                          │   FINAL CAPTION OUTPUT   │
                          │   (Flux LoRA Format)     │
                          └─────────────────────────┘
```

-----

### Multi-OCR Fusion Strategy

> **Why four OCR sources?** No single OCR engine is perfect. Different engines have different strengths — some are better with handwritten text, some with dense text, some with unusual fonts. Running four sources in parallel and merging gives us near-100% text coverage.

|OCR Source      |Strengths                                                                             |Weaknesses                                         |Role in System                                                  |
|----------------|--------------------------------------------------------------------------------------|---------------------------------------------------|----------------------------------------------------------------|
|**Florence-2**  |Region-aware; understands visual context; can detect where text is located on image   |May miss very small text; slower than pure OCR     |Primary OCR — provides spatial position info for each text block|
|**EasyOCR**     |Fast; supports English + Hindi (Devanagari); good general coverage                    |Less accurate on complex decorative fonts          |Secondary — full-image pass for broad coverage                  |
|**PaddleOCR**   |Very high accuracy on dense, small, or complex text                                   |Slower; optional install dependency                |Tertiary — additional verification pass                         |
|**Regional OCR**|Catches text in corners and edges that full-image passes miss; 4x upscaling before OCR|Computationally expensive (runs 12 times per image)|Safety net — catches the 5-10% that others miss                 |

-----

### Regional OCR — The 12-Crop Strategy

> **Why regional crops?** When an OCR engine processes a full 1024×1024 image, small text in corners gets compressed. By extracting 12 overlapping sub-regions and **4x upscaling each one** before OCR, tiny text that would otherwise be missed becomes clearly readable.

The 12 regions form a **3×3 grid with overlap**, plus **3 dedicated footer strips** for addresses, phone numbers, and legal text:

```python
# Each tuple is (x_start, y_start, x_end, y_end, region_name)
# All values are fractional (0.0 to 1.0) relative to image size
# Overlap between adjacent regions is intentional — ensures no text falls between gaps

regions = [
    # ─── 3×3 Grid with intentional overlap ───────────────────────────────────
    (0.0,  0.0,  0.5,  0.38, "top-left"),      # Top-left quadrant
    (0.25, 0.0,  0.75, 0.38, "top-center"),    # Top center with L/R overlap
    (0.5,  0.0,  1.0,  0.38, "top-right"),     # Top-right quadrant

    (0.0,  0.3,  0.5,  0.7,  "middle-left"),   # Middle-left (overlaps top+bottom)
    (0.25, 0.3,  0.75, 0.7,  "center"),        # Center crop (most overlap)
    (0.5,  0.3,  1.0,  0.7,  "middle-right"),  # Middle-right (overlaps top+bottom)

    (0.0,  0.62, 0.5,  1.0,  "bottom-left"),   # Bottom-left quadrant
    (0.25, 0.62, 0.75, 1.0,  "bottom-center"), # Bottom center with L/R overlap
    (0.5,  0.62, 1.0,  1.0,  "bottom-right"),  # Bottom-right quadrant

    # ─── Footer strips for fine print ────────────────────────────────────────
    # These target the very bottom of posters where address/phone/T&C appear
    (0.0,  0.82, 0.5,  1.0,  "footer-left"),   # Lower-left footer region
    (0.5,  0.82, 1.0,  1.0,  "footer-right"),  # Lower-right footer region
    (0.0,  0.88, 1.0,  1.0,  "bottom-strip"),  # Full-width bottom strip (T&C, URLs)
]

# After extracting each region: upscale 4× before running OCR
# This is critical — tiny 8px text becomes 32px after upscaling, making it readable
```

> **Key insight:** The overlap between regions is deliberate. If a word sits exactly at the boundary between two crops, it will be fully visible in at least one of them. The fuzzy deduplication step later removes the duplicate detections.

-----

### Text Validation Pipeline

> **Why validate?** Raw OCR output is noisy — it includes artifacts, partial characters, lorem ipsum placeholder text, and random symbols. Every piece of text passes through five validation gates before being accepted.

```
Raw OCR Token
      │
      ▼
 ┌────────────────────────────────────┐
 │  Gate 1: Length Check              │
 │  Must be 2-100 characters long     │
 │  • < 2 chars = likely noise        │
 │  • > 100 chars = likely garbled    │
 └──────────────────┬─────────────────┘
                    │ PASS
                    ▼
 ┌────────────────────────────────────┐
 │  Gate 2: Character Validation      │
 │  Allow: alphanumeric, Devanagari,  │
 │  safe punctuation (.,!?@#₹$%/)     │
 │  Reject: control chars, rare       │
 │  Unicode blocks, most symbols      │
 └──────────────────┬─────────────────┘
                    │ PASS
                    ▼
 ┌────────────────────────────────────┐
 │  Gate 3: Real Word Detection       │
 │  Uses word frequency analysis      │
 │  (wordfreq library)                │
 │  Rejects: "XQZPW", "AABBCC"       │
 │  Accepts: "SALE", "DIWALI"         │
 └──────────────────┬─────────────────┘
                    │ PASS
                    ▼
 ┌────────────────────────────────────┐
 │  Gate 4: Lorem Ipsum Filter        │
 │  Removes common placeholder text  │
 │  Patterns: "lorem", "ipsum",       │
 │  "dolor sit amet", etc.            │
 └──────────────────┬─────────────────┘
                    │ PASS
                    ▼
 ┌────────────────────────────────────┐
 │  Gate 5: Marketing Pattern Check   │
 │  ALWAYS KEEP if matches:           │
 │  • Prices: ₹999, $50              │
 │  • Dates: 25 Dec, 2024            │
 │  • URLs: www.brand.com            │
 │  • Phone: +91-9999999999          │
 └──────────────────┬─────────────────┘
                    │
                    ▼
                ACCEPTED
```

> **Why “always keep” marketing patterns?** Prices, dates, phone numbers, and URLs are the highest-value text in any marketing poster. Even if they look unusual or garbled, they should never be discarded — they’ll be cleaned up by the LLM in the next stage.

-----

### LLM Cleanup with Ollama Mistral 7B

> **Why use an LLM for cleanup?** OCR engines make systematic errors — confusing `I` with `l` with `1`, `O` with `0`, `S` with `5`. A language model understands context and can recognize that `"TlCKETS"` is almost certainly `"TICKETS"` based on surrounding words and common English vocabulary.

**Common OCR Errors Caught and Fixed:**

|Error Type          |Raw OCR Output|Corrected Output|Rule Applied               |
|--------------------|--------------|----------------|---------------------------|
|Character confusion |`TlCKETS`     |`TICKETS`       |l→I when surrounded by caps|
|Character confusion |`F0R 50% OFF` |`FOR 50% OFF`   |0→O in word context        |
|Character confusion |`G0LDEN SALE` |`GOLDEN SALE`   |0→O in word context        |
|Space errors        |`TI CKETS`    |`TICKETS`       |Wrong word split           |
|Space errors        |`GOLDENSALE`  |`GOLDEN SALE`   |Missing space between words|
|Unicode digits      |`५०% छूट`      |`50% छूट`        |Devanagari digits → ASCII  |
|Mixed case fragments|`diWALi`      |`Diwali`        |Case normalization         |

**The Exact Cleanup Prompt Sent to Mistral:**

```
You are an OCR post-processor for ALL types of posters and designs.
Types include: marketing, wedding, festival, music concert, food menu, sports,
religious events, real estate, educational, and corporate.

Fix these OCR text fragments extracted from an image:
1. TlCKETS
2. F0R 50% OFF
3. G0LDEN SALE

RULES (apply in this order):
1. Fix character confusion ONLY when result is a real, recognizable word
   (I/l/1 confusion, O/0 confusion, S/5 confusion, B/8 confusion)
2. Fix space errors — split wrongly joined words, join wrongly split words
3. Convert Unicode/Devanagari digits to ASCII digits (५ → 5, ६ → 6)
4. KEEP AS-IS: Hindi/Devanagari text, brand names, prices, phone numbers, URLs
5. REMOVE ONLY: fragments that are purely symbols with zero real words

Return ONLY a valid JSON array of strings. No explanation. No markdown. No preamble.
Example: ["TICKETS", "FOR 50% OFF", "GOLDEN SALE"]
```

> **Why `Return ONLY a JSON array`?** If the model adds any explanation or markdown, the JSON parser will fail. This strict instruction prevents that. The error handling in `caption_runner.py` still wraps the parse in a try/catch as a safety net.

-----

### Domain Detection System

> **Why detect domain?** The caption format for a wedding invitation is very different from a concert poster. By auto-detecting the domain, we can prepend the right descriptor to each caption, which dramatically improves LoRA training quality.

Domain detection uses **keyword pattern matching** on the merged OCR text:

|Domain     |Trigger Keywords                                                  |Output Prefix                             |
|-----------|------------------------------------------------------------------|------------------------------------------|
|Wedding    |`wedding`, `bride`, `groom`, `shaadi`, `mehndi`, `baraat`, `vivah`|`wedding invitation card design`          |
|Festival   |`diwali`, `holi`, `navratri`, `eid`, `christmas`, `pongal`, `onam`|`festival celebration poster design`      |
|Sale       |`sale`, `discount`, `% off`, `offer`, `deal`, `clearance`         |`sale marketing promotional poster design`|
|Concert    |`concert`, `live music`, `band`, `tour`, `fest`, `gig`            |`music concert event poster design`       |
|Food       |`restaurant`, `menu`, `cuisine`, `cafe`, `biryani`, `thali`       |`restaurant food menu promotional design` |
|Real Estate|`property`, `apartment`, `villa`, `bhk`, `plot`, `sqft`           |`real estate property promotional poster` |
|Generic    |*(no match)*                                                      |`marketing promotional poster design`     |


> **Matching logic:** The system checks all detected OCR text (lowercased) against each domain’s keyword list. The first domain with a match wins. If multiple domains match, the one with more keyword hits wins.

-----

### Style Tag Generation

> **Why hashtags in captions?** Flux LoRA training benefits from consistent style descriptors at the end of captions. These hashtags group similar images together, helping the model learn domain-specific visual patterns.

```python
# Each tuple is (trigger_keywords_list, hashtag_string_to_append)
# First matching rule wins; multiple rules can also match if desired

STYLE_CHECKS = [
    # Sales & Marketing
    (["sale", "discount", "% off"],           "#sale #marketing #discount #promotional"),
    (["clearance", "offer", "deal"],           "#offer #deal #clearance"),

    # Indian Festivals
    (["diwali", "deepawali"],                  "#diwali #festival #indianfestival"),
    (["holi", "rang"],                         "#holi #festival #colorful"),
    (["navratri", "durga", "garba"],           "#navratri #festivedesign #garba"),
    (["eid", "ramadan"],                       "#eid #islamicdesign #festive"),

    # Weddings
    (["wedding", "bride", "groom"],            "#wedding #invitation #elegant"),
    (["mehndi", "sangeet", "baraat"],          "#weddingfunction #indianwedding"),

    # Music & Entertainment
    (["concert", "music", "band"],             "#music #concert #livemusic #event"),
    (["dj", "nightclub", "rave"],              "#clubmusic #electronicmusic"),

    # Food
    (["restaurant", "menu", "cuisine"],        "#food #restaurant #menu #dining"),
    (["biryani", "thali", "street food"],      "#indianfood #streetfood"),

    # Real Estate
    (["property", "apartment", "villa"],       "#realestate #property #housing"),

    # Generic Promotions
    (["launch", "new", "introducing"],         "#newlaunch #announcement"),
    (["limited", "exclusive", "premium"],      "#luxury #premium #exclusive"),
    # ... 15+ more patterns defined in caption_runner.py
]
```

-----

### Caption Format for Flux LoRA Training

The final caption follows a **strict structured format** optimized to provide maximum signal to the LoRA training process:

```
{domain_description}, {visual_description}, text "{content}" at {position} in {font_style}, {hashtags}
```

**Complete Real-World Example:**

```
sale marketing promotional poster design, dark red festive background with gold bokeh 
particles and ornate border decorations, elegant serif typography layout,
text "DIWALI SALE" at top-center in bold display serif font,
text "50% OFF" at top-right in large impact display font,
text "Shop Now" at bottom-center in clean sans-serif,
text "+91-9999-999999" at bottom-strip in small regular font,
#sale #marketing #discount #promotional #indianfestival #festivedesign
```

> **Why this specific format?** Flux’s T5 text encoder processes captions sequentially. Having position information directly next to content (`text "DIWALI SALE" at top-center`) gives the model the strongest possible signal for spatial text placement during fine-tuning.

-----

### Configuration Parameters

> **How to tune:** Start with defaults. If you’re seeing too many false-positive OCR detections (garbage text), increase `text_threshold`. If you’re missing faint text, decrease it.

|Parameter         |Default          |Valid Range     |Description                           |Impact of Changing                             |
|------------------|-----------------|----------------|--------------------------------------|-----------------------------------------------|
|`MIN_CAPTION_LEN` |`80`             |50–200          |Minimum final caption length to accept|Higher = stricter quality gate, more retries   |
|`MIN_FLORENCE_LEN`|`40`             |20–100          |Minimum Florence OCR output length    |Higher = rejects images with little text       |
|`OLLAMA_MODEL`    |`mistral:7b`     |Any Ollama model|LLM for OCR cleanup                   |Larger models = better cleanup, slower         |
|`OLLAMA_URL`      |`localhost:11434`|Any URL         |Ollama server endpoint                |Must match where `ollama serve` is running     |
|`text_threshold`  |`0.55`           |0.3–0.9         |EasyOCR confidence threshold          |Lower = more detections + more false positives |
|`low_text`        |`0.35`           |0.1–0.6         |EasyOCR low-text threshold            |Affects detection of faint, watermark-like text|

-----

##  Caption Generation — Technical Deep Dive

### Preprocessing Pipeline

> **Why preprocess at all?** OCR accuracy drops dramatically on dark backgrounds, oversaturated colors, and low-contrast text. Adaptive preprocessing corrects for these conditions *before* OCR runs, improving detection rates by 20-40%.

The system first **analyzes the image** to detect its characteristics, then applies the appropriate enhancement:

```python
def get_preprocessed(img_bgr, has_skin, has_flame, has_split,
                     is_dark, is_colorful, is_low_contrast):
    """
    Applies condition-specific image enhancement before OCR.
    
    Each branch targets a different failure mode:
    - split_clahe: Handles split-screen posters where two halves have
                   very different brightness levels. CLAHE applied per half
                   prevents one half from dominating histogram equalization.
    
    - enhance_dark: Handles dark backgrounds (e.g. night event posters).
                    Gamma correction first lifts shadows, then CLAHE
                    increases local contrast to make light text pop.
    
    - enhance_colorful: Handles highly saturated images (festival posters).
                        Reduces saturation first so OCR isn't confused by
                        color boundaries, then applies CLAHE.
    
    - enhance_low_contrast: Handles pale/washed-out images.
                            Aggressive CLAHE + sharpening to define edges.
    
    - Default: Standard CLAHE for normal images.
    
    All branches end with upscale_2x() + sharpen() because:
    - 2x upscale: Makes small text larger before OCR processes it
    - Sharpen: Defines character edges more clearly
    """
    if has_split:
        # Split image (two distinct halves) — apply CLAHE to each half independently
        # This prevents a bright half from causing the dark half to be over-equalized
        base = split_clahe(img_bgr)
    elif is_dark:
        # Dark backgrounds cause CLAHE alone to produce noisy results
        # Gamma correction (γ < 1.0) lifts the shadow tones before CLAHE runs
        base = enhance_dark(img_bgr)
    elif is_colorful:
        # High saturation confuses OCR — it interprets color gradient edges as text
        # Saturation reduction first, then CLAHE for contrast
        base = enhance_colorful(clahe(img_bgr))
    elif is_low_contrast:
        # Insufficient pixel variance means OCR can't distinguish text from background
        # Aggressive CLAHE clip limit + unsharp masking to define character boundaries
        base = enhance_low_contrast(img_bgr)
    else:
        # Standard case: CLAHE for contrast normalization
        base = clahe(img_bgr)

    # Always upscale 2x and sharpen after the condition-specific enhancement
    # upscale_2x uses Lanczos resampling (best quality for text)
    # sharpen applies an unsharp mask with moderate radius
    return upscale_2x(sharpen(base))
```

**Edge Case Detectors — How Each Works:**

|Detector                |How It Works                                                                                                        |Threshold                          |
|------------------------|--------------------------------------------------------------------------------------------------------------------|-----------------------------------|
|`detect_skin`           |Converts to HSV, counts pixels in skin hue range (0-25°) with saturation >0.3                                       |>25% of pixels                     |
|`detect_flame`          |Counts pixels in red-orange hue range (0-30°, 150-180°)                                                             |>35% of pixels                     |
|`detect_split`          |Compares mean brightness of left half vs right half                                                                 |>70 intensity difference           |
|`detect_devanagari`     |Counts horizontal vs vertical edges using Sobel operators; Devanagari has more horizontal strokes (the headline bar)|Horizontal > Vertical edge strength|
|`detect_dark_background`|Converts to grayscale, checks mean pixel value                                                                      |Mean < 80/255                      |
|`detect_highly_colorful`|Converts to HSV, checks mean saturation channel                                                                     |Mean saturation > 140/255          |
|`detect_low_contrast`   |Converts to grayscale, checks standard deviation                                                                    |Std dev < 35/255                   |

-----

### Fuzzy Deduplication Algorithm

> **Why fuzzy matching instead of exact matching?** Four OCR engines processing the same text will produce slightly different outputs. One might produce `"GOLDEN SALE"`, another `"GOLDEN  SALE"` (extra space), another `"GOLDENSALE"`. Exact matching would keep all three. Fuzzy matching with an 80% similarity threshold correctly identifies them as the same text.

```python
def fuzzy_deduplicate(texts, threshold=80):
    """
    Removes near-duplicate OCR results while keeping the highest quality version.
    
    Algorithm:
    1. For each new candidate, compare against all already-kept texts
    2. If similarity >= threshold (80%), they're considered duplicates
    3. Keep whichever has the higher text_score()
    4. If no duplicate found, add to kept list
    
    threshold=80 means strings must be 80%+ similar to be considered duplicates.
    Lower values (e.g., 60) would merge more aggressively but risk false merges.
    Higher values (e.g., 95) would be nearly exact matching, missing OCR variations.
    
    fuzz.ratio() uses Levenshtein distance, normalized 0-100.
    Example: "GOLDEN SALE" vs "GOLDEN  SALE" → ratio ~95 → merged ✓
    Example: "GOLDEN SALE" vs "GOLDEN DEAL" → ratio ~73 → kept separate ✓
    """
    kept = []
    for candidate in texts:
        is_dup = False
        for i, existing in enumerate(kept):
            # Compare case-insensitively — "Sale" and "SALE" are the same
            if fuzz.ratio(candidate.lower(), existing.lower()) >= threshold:
                # It's a duplicate — but which version should we keep?
                # text_score() prefers: real English words, proper spacing, longer strings
                if text_score(candidate) > text_score(existing):
                    kept[i] = candidate  # Replace with higher quality version
                is_dup = True
                break  # No need to check further
        if not is_dup:
            kept.append(candidate)  # New unique text — add to list
    return kept


def text_score(text):
    """
    Scores a text string for quality. Higher = better.
    
    Scoring components:
    - Word frequency sum: Real common words (like "SALE", "OFF") score high.
      Gibberish like "XQZPW" scores near zero. Uses wordfreq library.
    - Space ratio: Text with appropriate spacing scores higher than run-together
      words. Catches "GOLDENSALE" (bad) vs "GOLDEN SALE" (good).
    - Length bonus: Slightly longer strings preferred (more context preserved).
    
    This ensures that when two OCR sources detect the same text differently,
    we always keep the cleaner, more legible version.
    """
    words = text.split()
    word_freq_sum = sum(zipf_frequency(w.lower(), 'en') for w in words)
    space_ratio = len(words) / max(len(text), 1)
    return word_freq_sum + space_ratio
```

-----

### Fragment Merging

> **Why merge fragments?** OCR engines often split a single word or short phrase across multiple detections. “GOLDEN” and “SALE” might be returned separately even though they’re one headline. Fragment merging intelligently recombines these based on context.

```python
def merge_fragments(texts):
    """
    Merges adjacent short OCR fragments that likely belong together.
    
    Merge Criteria (all must be true):
    1. Current fragment has <= 2 words (longer fragments are complete on their own)
    2. Next fragment has <= 2 words
    3. Neither fragment triggers a hard stop condition
    4. Both fragments are the same type (both alphanumeric, or both Devanagari)
    5. Current fragment doesn't end with sentence-ending punctuation (. ! ?)
    
    Hard stop conditions prevent merging of:
    - Phone numbers: "9999" shouldn't merge with "999999" to form a new number
    - Addresses: "ST." shouldn't merge with next line
    - Currency: "₹999" shouldn't merge with surrounding text
    - URLs/Emails: Already complete; merging would break them
    - Years: "2024" is a standalone value
    - Sentence-ending punctuation: Marks clear text boundaries
    
    Example of correct merge:
    Input:  ["GOLDEN", "SALE", "50%", "OFF", "+91-9999-999999"]
    Output: ["GOLDEN SALE", "50% OFF", "+91-9999-999999"]
             ↑ merged        ↑ merged   ↑ hard stop (phone) — NOT merged
    """
    result, i = [], 0
    while i < len(texts):
        current = texts[i]

        # Hard stops and long fragments are kept as-is (no merging)
        if is_hard_stop(current) or len(current.split()) > 2:
            result.append(current)
            i += 1
            continue

        # Check if the next fragment should be merged with current
        if i + 1 < len(texts):
            nxt = texts[i + 1]
            if (not is_hard_stop(nxt)                       # Next is not a hard stop
                    and len(nxt.split()) <= 2               # Next is also a short fragment
                    and same_type(current, nxt)             # Both Latin or both Devanagari
                    and not current.rstrip().endswith(      # Current doesn't end a sentence
                        ('.', '!', '?'))):
                result.append(current + ' ' + nxt)         # Merge with a space
                i += 2                                      # Skip both fragments
                continue

        result.append(current)
        i += 1
    return result
```

**Hard Stop Conditions — What Triggers Them:**

```
• Phone numbers   → Regex: \d{3}[-.\s]\d{3,}  or  +\d{10,12}
• Street addresses → Contains: "ST.", "APT.", "ROAD", "NAGAR", ZIP patterns
• Currency amounts → Contains: ₹, $, €, £ followed by digits
• URLs             → Contains: http, www., .com, .in, .org
• Email addresses  → Contains: @ symbol
• Years            → Exactly 4 digits between 1900–2099
• Sentence ends    → Last character is . ! or ?
```

-----

### Position Tracking System

> **Why track position?** A caption that says `text "SHOP NOW" at bottom-center` is far more useful for training than `text "SHOP NOW"` with no position. Position information helps the LoRA model understand spatial text layout patterns.

Positions are assigned based on which of the 12 regional crops detected the text, then sorted in a logical reading order:

```python
# Priority values determine sort order in final caption
# Lower number = appears earlier in caption (top-to-bottom, left-to-right)

POSITION_PRIORITY = {
    'top-center':    0,   # Main headline — always first
    'top-left':      1,   # Secondary top element
    'top-right':     2,   # Date/price often here
    'center':        3,   # Main body content
    'middle-center': 4,
    'middle-left':   5,
    'middle-right':  6,
    'bottom-center': 7,   # CTA (Call to Action) buttons
    'bottom-left':   8,
    'bottom-right':  9,
    None:           10,   # Unknown position — goes last
}

# Text detected in multiple regions gets the highest-priority position
# Example: "50% OFF" detected in both top-right and top-center
#          → assigned 'top-center' (priority 0 wins over priority 2)
```

-----

### Quality Scoring

> **Why score quality?** Not all generated captions are equally good. The scoring system allows the pipeline to automatically flag low-quality captions for human review without stopping the entire batch.

```python
def caption_quality_score(caption, florence_text, ocr_items):
    """
    Assigns a quality score 0-100 to a generated caption.
    
    Three independent components contribute to the score:
    
    1. Caption Length Score (max 40 points)
       Longer captions = more text detected = more training signal.
       Captions under 80 chars are likely missing most of the text.
    
    2. Florence Visual Description Score (max 30 points)
       Florence provides the visual/artistic description (background, colors, style).
       Longer Florence descriptions = richer visual training data.
    
    3. OCR Coverage Score (max 20 points)
       Number of distinct text items found in the image.
       More items = more text elements captured in the caption.
    
    4. Sentence Quality Bonus (max 10 points)
       Florence descriptions ending in . ! ? are well-formed sentences,
       indicating higher quality visual description output.
    """
    score = 0

    # ── Component 1: Caption Length ──────────────────────────────────────────
    cap_len = len(caption)
    if cap_len >= 400:   score += 40   # Excellent — rich, detailed caption
    elif cap_len >= 250: score += 30   # Good — adequate detail
    elif cap_len >= 150: score += 20   # Acceptable — some text captured
    elif cap_len >= 80:  score += 10   # Poor but meets minimum threshold

    # ── Component 2: Florence Description Quality ─────────────────────────────
    flor_len = len(florence_text)
    if flor_len >= 200:   score += 30   # Detailed visual description
    elif flor_len >= 100: score += 20   # Moderate visual description
    elif flor_len >= 50:  score += 10   # Minimal visual description

    # ── Component 3: OCR Coverage ─────────────────────────────────────────────
    n_ocr = len(ocr_items)
    if n_ocr >= 5:   score += 20   # Multiple text elements captured
    elif n_ocr >= 3: score += 15   # Several text elements captured
    elif n_ocr >= 1: score += 10   # At least one text element captured

    # ── Component 4: Sentence Quality Bonus ───────────────────────────────────
    # Florence descriptions are better when they end as complete sentences
    if florence_text and florence_text[-1] in '.!?':
        score += 10

    return score  # Range: 0-100
```

**Quality Thresholds:**

|Score Range|Grade       |Action                                     |
|-----------|------------|-------------------------------------------|
|≥ 70       | Excellent |Use directly for training                  |
|50–69      | Good      |Use for training; may want spot review     |
|30–49      | Acceptable|Use with caution; manual review recommended|
|< 30       | Poor      |Flag in `retry_needed.txt`; re-process     |

-----

### Progress Tracking System

> **Why track progress to a file?** Caption generation can take hours for large batches. If the process crashes or is interrupted, progress tracking allows it to resume from where it left off without reprocessing completed images.

The system maintains three files:

```json5
// caption_progress.json — Master progress state
// Updated after every image is processed
{
  "done": [
    "/home/aiteam/Animesh-Diffusion/batch_1/poster_001.jpg",
    "/home/aiteam/Animesh-Diffusion/batch_1/poster_002.jpg"
    // All successfully processed images
  ],
  "failed": [
    "/home/aiteam/Animesh-Diffusion/batch_1/poster_003.jpg — too_small",
    "/home/aiteam/Animesh-Diffusion/batch_1/poster_004.jpg — corrupted_file"
    // Images with errors — error reason appended after " — "
  ],
  "retry": [
    "/home/aiteam/Animesh-Diffusion/batch_1/poster_005.jpg"
    // Images that succeeded but got quality score < 30
    // These are candidates for a second processing pass
  ]
}
```

```
retry_needed.txt  — Same as progress.json retry list, plain text format
                    Easy to pass as input to a re-run command

failed_images.txt — Same as progress.json failed list, plain text format
                    Review this to understand failure patterns
```

> **Resume behavior:** On startup, `caption_runner.py` loads `caption_progress.json` and skips any image already in the `done` list. This means you can safely Ctrl+C and restart at any time.

-----

##  Why Text Fails in Diffusion Models — A Deep Dive

> **Why does this section exist?** Understanding *why* standard AI image generation fails at text is essential to understanding *why* our pipeline is architected the way it is. Every design decision in the pipeline maps directly to one of these four failure modes.

-----

### The Fundamental Problem

When you write a prompt like:

```
"A Lakme moisturizer bottle with text 'Made by Lakme' written on the side"
```

You might assume the model reads and understands this instruction. **It doesn’t.** The model has **never learned to read** — it has only learned to *see*. This is the root cause of all text generation failures in diffusion models.

There are four distinct layers where this breaks down.

-----

### Layer 1 — The Text Encoder Problem (CLIP)

**What happens when you write a prompt:**

Your prompt goes through a **text encoder** — typically CLIP (Contrastive Language-Image Pre-training):

```
"Made by Lakme"  →  [0.23, -0.87, 0.44, 0.11, ...]  (768-dimensional vector)
```

CLIP converts your text into a numerical vector representing its *semantic meaning*. Here is the critical problem:

**How CLIP was trained:**

CLIP was trained on billions of image-text pairs with one objective: match images to descriptions.

|Image                 |Description                        |
|----------------------|-----------------------------------|
|Dog running in park   |“a dog running in the park”        |
|City street at night  |“busy city street at night”        |
|Lakme skincare product|“Lakme moisturizer luxury skincare”|

CLIP learned semantic concepts. **It never learned to read characters.**

| What CLIP Understands          | What CLIP Does NOT Understand    |
|---------------------------------|-----------------------------------|
|“Lakme” = cosmetics brand concept|The individual letters L-A-K-M-E   |
|“GOLD” = luxury, precious, shiny |The spelling G-O-L-D               |
|Brand identity and logo style    |Character shapes and stroke details|
|Overall meaning of text          |Exact character sequences          |


> ** Problem 1 — CLIP is character-blind.** To CLIP, “Lakme”, “LAKME”, “lakme”, and even a misspelling like “Lakm3” are essentially the same thing — a cosmetics brand concept. Spelling doesn’t exist in CLIP’s world.

-----

### Layer 2 — Semantic Drift in Attention Mechanisms

**What happens when you ask for “GOLD” text:**

Prompt: `"write the word GOLD on the bottle"`

|What You Meant                    |What the Model Generates                       |
|----------------------------------|-----------------------------------------------|
|G · O · L · D (4 specific letters)| golden shimmer, metallic sheen, luxury feel |

The model doesn’t generate letters — it generates the *concept of gold*. The attention mechanism interprets “GOLD” as a visual aesthetic instruction, not a typographic one.

**What happens when you ask for “Made by Lakme”:**

The model might generate any of:

- `"Maed by Lakme"` — spelling transposition
- `"Made by Lakm3"` — character substitution
- Something *visually similar* to “Lakme” branding but unreadable as text

This is called **Semantic Drift**: the concept encoded by the text encoder overrides the precise glyph generation. The model prioritizes brand identity over character accuracy because that’s what it was trained to do.

> ** Problem 2 — Semantic drift.** Concept representations override letter generation. The more recognizable a brand or word is, the stronger the drift — the model “knows” what Lakme products *look like* and draws that instead of the letters.

-----

### Layer 3 — VAE Compression Destroys Fine Details

**How diffusion models actually work:**

Diffusion models don’t operate on pixels directly. They work in *latent space*:

```
Original Image (1024 × 1024 pixels)
         │
         ▼
    VAE Encoder  ─── compresses 8:1
         │
         ▼
  Latent Space  (128 × 128 × 16 channels)
    ← Generation happens here →
         │
         ▼
    VAE Decoder  ─── expands 8:1
         │
         ▼
 Final Image (1024 × 1024 pixels)
```

**Why compress?** Processing 1024×1024 images directly would require ~50× more GPU memory. Latent space makes generation computationally feasible.

**The compression problem:**

VAE (Variational Autoencoder) was trained predominantly on *natural images*: faces, landscapes, objects. Text was rare in training data.

When the letter “L” passes through VAE compression:

- Its thin vertical stroke loses positional precision during quantization
- On decompression, the model reconstructs its best approximation of an “L”-like shape
- The result might be: “l”, “I”, “|”, or a slightly curved stroke

|What Gets Preserved      |What Gets Destroyed   |
|-------------------------|----------------------|
|Overall color and tone   |Exact stroke positions|
|Large shapes and contours|Serif details         |
|Background textures      |Character spacing     |
|Object identity          |Letter distinctiveness|


> ** Problem 3 — VAE compression destroys fine details.** The 4-channel VAE used in SDXL-era models simply doesn’t have enough capacity to faithfully preserve the fine strokes that distinguish characters from each other.

-----

### Layer 4 — The Denoising Process Struggles with High-Frequency Details

**How diffusion generates images:**

Diffusion starts from pure random noise and gradually shapes it through many denoising steps.

At each step, the model asks: “Given my prompt and what I’ve generated so far, what should I remove from the noise?”

**The frequency mismatch problem:**

|Information Type   |Examples                 |Frequency     |Diffusion Difficulty|
|-------------------|-------------------------|--------------|--------------------|
|Background color   |Blue sky, dark background|Very low      | Easy              |
|Large shapes       |Bottle outline, frame    |Low           | Easy              |
|Object placement   |Logo position, label area|Medium        | Manageable        |
|Textures           |Wood grain, fabric       |Medium-high   | Sometimes         |
|Exact letter shapes|“G” vs “C” vs “O”        |High          | Hard              |
|Character spacing  |Width between letters    |Very high     | Very Hard         |
|Serif details      |Thin strokes, serifs     |Extremely high| Near impossible   |

Early denoising steps establish large structures (background, composition). Late steps add fine details. But text characters need *precise high-frequency information* to be readable — they need every pixel to be exactly right. Diffusion’s stochastic (random) nature means this precision is fundamentally unreliable.

> ** Problem 4 — High-frequency struggle.** Diffusion models are probabilistic — they sample from a distribution of plausible images. Readable text requires *exact* pixel patterns, but diffusion generates *probable* pixel patterns. These two requirements are fundamentally in conflict.

-----

### Summary of Four Problems

|#|Problem                |Layer              |What Goes Wrong                                            |How Common|
|-|-----------------------|-------------------|-----------------------------------------------------------|----------|
|1|Character blindness    |Text Encoder (CLIP)|Spelling info never encoded into conditioning vector       |Always    |
|2|Semantic drift         |Attention Mechanism|Brand/concept meaning overrides letter generation          |Always    |
|3|Stroke destruction     |VAE Compression    |Fine details blur during encode/decode cycle               |Always    |
|4|High-frequency struggle|Denoising Process  |Probabilistic generation can’t achieve exact pixel patterns|Always    |


> **Critical insight:** All four problems exist simultaneously at different layers. Fixing only one (e.g., using a better text encoder) still leaves three other failure modes active. The solution must address all four layers.

-----

##  The Multi-Layered Solution Architecture

### Industry Solutions — Layer by Layer

-----

#### Layer 1 Fix — Replace CLIP with T5-XXL

**The Problem:** CLIP is character-blind — it encodes concepts, not spellings.

**The Solution:** Use T5 (Text-to-Text Transfer Transformer) as the text encoder.

|Aspect              |CLIP                        |T5-XXL                                       |
|--------------------|----------------------------|---------------------------------------------|
|Model type          |Vision-language model       |Pure language model                          |
|Training data       |Billions of image-text pairs|Terabytes of books, articles, code, Wikipedia|
|Context length      |77 tokens (hard limit)      |4096 tokens                                  |
|Spelling awareness  | No — encodes concepts     | Yes — processes character sequences        |
|Long prompt handling|Degrades past 77 tokens     |Handles gracefully                           |
|Brand name encoding |“Lakme” = cosmetics concept |“L-a-k-m-e” = 5 specific characters          |

**How T5 processes text differently:**

```
CLIP encoding of "Made by Lakme":
"Made by Lakme" → ONE concept vector for the entire phrase
                  Meaning: {cosmetics, brand, luxury, product}

T5 encoding of "Made by Lakme":
M → token 1
a → token 2
d → token 3
e → token 4
  → space token
b → token 5
y → token 6
  → space token
L → token 7
a → token 8
k → token 9
m → token 10
e → token 11
Each character's identity preserved in the sequence.
```

> ** Layer 1 Fix complete:** T5-XXL as text encoder. Spelling information is now preserved in the conditioning signal. The model “knows” there are exactly 5 characters in “Lakme”.

-----

#### Layer 2 Fix — Glyph Pre-rendering + ControlNet

**The Problem:** Even with T5, the attention mechanism drifts toward semantic representation rather than precise character rendering.

**Weak Solution (Prompt Engineering):**

```
"text overlay reading exactly L-A-K-M-E, each letter individually"
```

This helps slightly but is brittle — inconsistent results.

**Strong Solution (GlyphControl / ControlNet):**

The key insight: **Don’t ask the model to generate letters at all.** Instead, render them externally and force the model to follow those exact shapes.

```
Step 1: Take the text "Made by Lakme"
        ↓
Step 2: Render it using a real font (PIL/FreeType)
        → Perfect characters, exact spacing, real typography
        ↓
Step 3: Extract edges from the rendered text (Canny edge detection)
        → Clean binary edge map of each character's outline
        ↓
Step 4: Feed this edge map into ControlNet during generation
        ↓
Step 5: At EVERY denoising step, ControlNet says:
        "Here is exactly where 'M' should be — follow this shape"
        "Here is exactly where 'a' should be — follow this shape"
        ↓
Step 6: Model applies texture, color, style on top of the given shapes
        → Result: Correct text, guaranteed
```

**Why this completely eliminates semantic drift:**

- The model is no longer *generating* letters — it’s *following* provided strokes
- Semantic drift requires the model to invent shapes; here the shapes are given
- The model’s only job is to stylize the pre-defined shapes

> ** Layer 2 Fix complete:** Glyph pre-rendering + ControlNet conditioning. The model follows shapes; it doesn’t generate them.

-----

#### Layer 3 Fix — 16-Channel VAE

**The Problem:** The 4-channel VAE used in SDXL and earlier models compresses too aggressively, destroying character fine details.

**The Solution:** Flux uses a 16-channel VAE — 4× more latent capacity.

|Aspect                    |4-Channel VAE (SDXL era)|16-Channel VAE (Flux)|
|--------------------------|------------------------|---------------------|
|Latent channels           |4                       |16                   |
|Information capacity      |Baseline                |4× higher            |
|Large shapes (backgrounds)| Preserved             | Preserved          |
|Colors and gradients      | Preserved             | Preserved          |
|Medium textures           | Partially             | Preserved          |
|Letter stroke widths      | Lost                  | Preserved          |
|Serif details             | Lost                  | Preserved          |
|Character spacing         | Lost                  | Preserved          |
|Fine kerning              | Lost                  | Preserved          |


> ** Layer 3 Fix complete:** Flux’s 16-channel VAE preserves the fine-detail information needed to distinguish characters from each other.

-----

### Flux.1-dev — All Three Fixes Combined

Flux.1-dev is the first widely-available model that integrates all three fixes into a unified architecture:

```
                    ┌─────────────────────────────────────────────────────┐
                    │              DUAL ENCODER SYSTEM                    │
                    │                                                     │
  "Made by Lakme" ──┤──→ CLIP-L  ──────────────────────────────────────→ │──→ ┐
                    │    (Concepts: brand identity, luxury, cosmetics)    │   │
                    │                                                     │   │
                    │──→ T5-XXL  ──────────────────────────────────────→ │──→ ┤
                    │    (Characters: M-a-d-e-space-b-y-space-L-a-k-m-e) │   │
                    └─────────────────────────────────────────────────────┘   │
                                                                              ▼
                                                               Joint Projection (3072 features)
                                                                              │
                    ┌─────────────────────────────────────────────────────┐   │
                    │         DIFFUSION TRANSFORMER (DiT)                 │ ←─┘
                    │                                                     │
                    │  Text tokens + Image tokens processed TOGETHER      │
                    │  in the same attention matrix at every layer.       │
                    │                                                     │
                    │  (Old UNet: text and image info traveled separately,│
                    │   merged only at specific cross-attention layers)   │
                    │                                                     │
                    └──────────────────────────────────┬──────────────────┘
                                                       │
                    ┌─────────────────────────────────────────────────────┐
                    │          16-CHANNEL VAE DECODER                     │ ←─┘
                    │  Fine character strokes preserved through           │
                    │  encode/decode cycle                                │
                    └──────────────────────────────────┬──────────────────┘
                                                       │
                                                       ▼
                                              Final Image Output
                                         (Better text than SDXL era)
```

**Why the DiT architecture additionally helps:**

In older UNet-based models (SD 1.5, SDXL):

- Text conditioning was injected at specific cross-attention layers
- Image spatial features and text features were processed in separate pathways
- Integration was shallow — only a few connection points

In Flux’s DiT:

- Text tokens and image tokens are in the **same sequence**
- Every attention layer sees both simultaneously
- Character structure becomes deeply intertwined with spatial generation at every level

-----

##  Our Pipeline — The Complete Solution

Our pipeline **builds on Flux’s native improvements** (T5, DiT, 16-channel VAE) and adds additional engineering layers for **guaranteed text accuracy**.

```
User Brief: "Diwali sale poster, dark red background, text 'DIWALI SALE' at top, '50% OFF' at right"
      │
      ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│  STAGE 1: BRIEF DECOMPOSITION                                                   │
│                                                                                 │
│  LLM Decomposer parses the brief into structured JSON schema:                   │
│  {                                                                              │
│    "texts": [                                                                   │
│      {"content": "DIWALI SALE", "x": 350, "y": 200, "font_size": 72},         │
│      {"content": "50% OFF",     "x": 700, "y": 400, "font_size": 96}          │
│    ],                                                                           │
│    "background_prompt": "dark red festive background with gold bokeh particles" │
│  }                                                                              │
│  ↓ Validation: check positions are within canvas bounds, fonts are valid       │
└─────────────────────────────────────────────────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│  STAGE 2: GLYPH RENDERING (THE CRITICAL STAGE)                                  │
│                                                                                 │
│  For EACH text block in the schema:                                             │
│  1. Load specified font from fonts/ directory                                   │
│  2. Render text at exact position on 1024×1024 white canvas using PIL          │
│  3. Extract Canny edges from the rendered text                                  │
│     → This edge map is the ControlNet conditioning input                       │
│  4. Generate soft mask covering text regions (for Pass 2 blending)             │
│                                                                                 │
│    NO diffusion model involvement here — pure deterministic font rendering   │
│  Result: Perfect glyphs, exact characters, every time                          │
└─────────────────────────────────────────────────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│  STAGE 3: BACKGROUND GENERATION (Pass 1 — FluxPipeline)                        │
│                                                                                 │
│  Runs Flux WITHOUT ControlNet                                                   │
│  Prompt explicitly includes: "no text, no letters, no words, no typography"    │
│  to prevent the model from accidentally generating any text in the background   │
│                                                                                 │
│  Output: Clean background image — attractive, styled, text-free                 │
└─────────────────────────────────────────────────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│  STAGE 4: TEXT INJECTION LOOP (Pass 2 — FluxControlNetPipeline)                │
│                                                                                 │
│  Inputs:                                                                        │
│  • Background image from Stage 3                                                │
│  • Glyph edge map from Stage 2                                                  │
│  • Dynamic ControlNet scale (0.55–0.80 based on smallest text)                 │
│                                                                                 │
│  Process:                                                                       │
│  1. ControlNet injects glyph shapes into every denoising step                  │
│  2. Flux applies color, style, texture to the enforced shapes                  │
│  3. Background blending composites text region into background                  │
│  4. Conditional contrast boost enhances text if needed                          │
└─────────────────────────────────────────────────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│  STAGE 5: OCR VERIFICATION                                                      │
│                                                                                 │
│  Run OCR on the generated image                                                 │
│  Compute NED (Normalized Edit Distance) between expected and detected text      │
│                                                                                 │
│  NED = 1.0 means perfect match                                                  │
│  NED ≥ 0.85 → PASS → proceed to upscaling                                     │
│  NED < 0.85 → FAIL → increment seed by 137, retry Stage 4 (max 3 attempts)   │
│                                                                                 │
│  Seed increment of 137 (prime number) ensures truly different random state     │
│  for each retry while remaining deterministic and reproducible                  │
└─────────────────────────────────────────────────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│  STAGE 6: UPSCALING & FINAL OUTPUT                                              │
│                                                                                 │
│  RealESRGAN x2+ upscales 1024×1024 → 2048×2048                                 │
│  Runs in separate upscale-env virtual environment                               │
│  Output saved to pipeline/outputs/ as JPEG (quality=95)                        │
└─────────────────────────────────────────────────────────────────────────────────┘
```

-----

### Stage-by-Stage Breakdown

#### Stage 1 — Brief Decomposer

The LLM (Mistral via Ollama) parses the free-form user brief into a validated JSON layout schema:

```json
{
  "canvas_size": [1024, 1024],
  "texts": [
    {
      "content": "DIWALI SALE",
      "x": 350,
      "y": 200,
      "font_size": 72,
      "font_weight": "bold",
      "color_instruction": "bright white with gold outline"
    },
    {
      "content": "50% OFF",
      "x": 700,
      "y": 400,
      "font_size": 96,
      "font_weight": "black",
      "color_instruction": "gold metallic"
    }
  ],
  "background_prompt": "dark red festive background with gold bokeh particles and ornate border",
  "text_to_render": "DIWALI SALE, 50% OFF",
  "text_font_style": "bold display serif",
  "style": "festival marketing poster",
  "lighting": "warm festive lighting"
}
```

> **Validation step:** After LLM output, a constraint engine validates: all x/y positions within canvas bounds, font sizes within configured range, content strings non-empty. If validation fails, the LLM is prompted again (max 3 attempts).

-----

#### Stage 2 — Prompt Composer

Injects text guidance into Flux prompts with **attention weighting**:

```
Positive prompt:
  (text reading 'DIWALI SALE':1.6)          ← High weight — most important
  (text reading '50% OFF':1.6)              ← High weight
  (clear legible typography:1.4)            ← Medium weight
  (sharp crisp text edges:1.3)              ← Medium weight
  dark red festive background with gold bokeh particles,
  festival marketing poster, warm festive lighting

Negative prompt:
  blurry text, garbled letters, wrong spelling, illegible text,
  distorted characters, merged letters, broken words,
  watermark, signature, illegible handwriting
```

> **Why weight text instructions higher?** Flux uses classifier-free guidance. Higher attention weights (`:1.6`) increase the model’s focus on those tokens during generation. Without weighting, background description tokens can dominate over text instructions.

-----

#### Stage 3 — Glyph Renderer (The Critical Component)

This stage guarantees correctness by **removing the model from the loop entirely**:

```python
def render_glyphs(schema: dict, canvas_size: int = 1024) -> tuple[Image.Image, Image.Image]:
    """
    Renders all text blocks from the schema onto a clean canvas using
    actual font rendering (PIL + FreeType). No AI involved.
    
    Returns:
        glyph_image: White background with black text — perfect typography
        canny_map:   Edge-detected version — input for ControlNet
    
    Why white background + black text?
    ControlNet Union in canny mode expects binary edge maps.
    White background ensures no noise in non-text regions.
    Black text creates maximum contrast for clean edge extraction.
    
    Font fallback chain:
    1. Try the font_weight specified in schema
    2. Try any font in fonts/ directory
    3. Fall back to PIL default font (always available)
    This ensures rendering never fails completely.
    """
    # Create clean white canvas — 1024×1024 to match generation canvas
    canvas = Image.new("RGB", (canvas_size, canvas_size), "white")
    draw = ImageDraw.Draw(canvas)

    for block in schema["texts"]:
        # Load font with fallback chain
        font = load_font_with_fallback(
            weight=block.get("font_weight", "regular"),
            size=block["font_size"]
        )

        # Draw text at exact position — pure deterministic rendering
        # No randomness, no approximation, no model involved
        draw.text(
            (block["x"], block["y"]),     # Exact pixel position from schema
            block["content"],             # Exact text string — character perfect
            font=font,
            fill="black"                  # Black text for clean edge extraction
        )

    # Extract Canny edges from the rendered text
    # These edges will guide ControlNet during generation
    glyph_array = np.array(canvas.convert("L"))         # Grayscale
    edges = cv2.Canny(glyph_array, threshold1=50, threshold2=150)  # Edge detection
    canny_map = Image.fromarray(edges)

    return canvas, canny_map
```

-----

#### Stage 4 — Background Generation (Pass 1)

Generates a clean background without any text. The negative prompt is important here:

```python
# Pass 1 prompt — explicitly exclude all text to keep background clean
pass1_prompt = f"{schema['background_prompt']}, {schema['style']}, {schema['lighting']}"

# These negative tokens are critical — without them, Flux may generate
# faint text or typography-like patterns in the background
pass1_negative = (
    "text, letters, words, typography, writing, labels, signs, captions, "
    "watermark, title, heading, inscription, font, alphabet, numbers, digits"
)

# FluxPipeline (no ControlNet) — 28 steps for high-quality background
background_image = flux_pipeline(
    prompt=pass1_prompt,
    negative_prompt=pass1_negative,
    num_inference_steps=28,      # More steps = higher quality background
    guidance_scale=3.5,          # Flux's recommended CFG scale
    generator=torch.Generator().manual_seed(seed)
).images[0]
```

> **VRAM optimization:** Pass 1 and Pass 2 share the same VAE, text encoders, and transformer. Only the ControlNet model is loaded additionally for Pass 2. This saves approximately 15GB VRAM compared to loading two separate pipelines.

-----

#### Stage 5 — Text Injection (Pass 2)

ControlNet enforces glyph shapes while Flux applies style:

```python
def run_pass2(self, background: Image.Image, canny_map: Image.Image,
              schema: dict, seed: int) -> Image.Image:
    """
    Second generation pass: injects pre-rendered text glyphs via ControlNet.
    
    Key parameters:
    
    control_mode=0:
        ControlNet Union supports multiple modes (0=canny, 1=depth, 2=normal, etc.)
        Mode 0 (canny) is correct for our text edge maps.
    
    controlnet_conditioning_scale:
        How strongly ControlNet enforces the glyph shapes.
        Calculated dynamically by _get_controlnet_scale() based on smallest font.
        Range: 0.55 (large text) to 0.80 (very small text).
        Too high: Text looks harsh/pasted-on, loses natural integration.
        Too low: Text drifts from glyph shapes, back to semantic generation.
    
    num_inference_steps=14:
        Fewer than Pass 1 (28 steps) because:
        - We're starting from a mostly-complete background image
        - We only need to add text, not rebuild the entire image
        - Fewer steps = faster, less risk of over-processing background
    
    image (background):
        Using the background image as init_image with strength<1.0
        lets ControlNet modify the image while preserving background structure.
    """
    # Calculate optimal conditioning scale
    scale = self._get_controlnet_scale(schema["texts"])

    result = self.controlnet_pipeline(
        prompt=self._build_pass2_prompt(schema),
        negative_prompt=NEGATIVE_PROMPT,
        image=background,                           # Start from background
        control_image=canny_map,                    # Glyph edge map
        control_mode=0,                             # Canny edge mode
        controlnet_conditioning_scale=scale,        # Dynamic scale
        num_inference_steps=14,                     # Fewer steps — refining, not creating
        guidance_scale=3.5,
        generator=torch.Generator().manual_seed(seed)
    ).images[0]

    # Blend: composite text region from result onto original background
    # This preserves background quality while adding text layer
    soft_mask = create_soft_mask(schema)
    final = Image.composite(result, background, soft_mask)

    # Conditionally boost text contrast if it's below threshold
    final = boost_text_contrast(final, soft_mask, strength=1.35)

    return final
```

-----

#### Stage 6 — OCR Verification and Retry

```python
def verify_text(image: Image.Image, expected_texts: list[str]) -> tuple[bool, float]:
    """
    Verifies that expected text is correctly rendered in the generated image.
    
    Uses Normalized Edit Distance (NED) scoring:
    NED = 1 - (edit_distance / max(len(expected), len(detected)))
    
    NED = 1.0: Perfect match
    NED = 0.85: ~15% character error rate — acceptable threshold
    NED = 0.0: Completely wrong text
    
    Why 0.85 threshold?
    - 1.0 would reject any minor OCR detection error (too strict)
    - 0.80 might accept garbled text (too lenient)  
    - 0.85 balances correctness with OCR detection variance
    
    Why seed + 137 for retry?
    - 137 is prime, ensuring maximum distance from previous seed
    - Deterministic: same brief + same retry number = same result
    - Allows debugging: you can reproduce any specific attempt
    """
    detected = run_easyocr(image)  # Run OCR on generated image
    detected_combined = " ".join(detected).lower()
    expected_combined = " ".join(expected_texts).lower()

    distance = editdistance.eval(expected_combined, detected_combined)
    max_len = max(len(expected_combined), len(detected_combined), 1)
    ned = 1.0 - (distance / max_len)

    return ned >= NED_THRESHOLD, ned
```

-----

### Simple Analogy

> This analogy helps explain the glyph approach to non-technical stakeholders.

**Without our approach (standard diffusion):**

```
You: "Paint a sign that says 'Made by Lakme'"
AI painter: thinks about what Lakme looks like →
            paints something resembling Lakme branding →
            Result: Looks Lakme-ish but not the actual words
```

**With our approach (glyph + ControlNet):**

```
You: "Here is a stencil with the exact letters 'Made by Lakme'.
      Paint through this stencil with your style and colors."
AI painter: follows exact stencil shapes →
            applies color, texture, style on top →
            Result:  Exact text, correct characters, styled beautifully
```

-----

## ⚙️ Image Generation Pipeline — Implementation Details

### Soft Mask Generator (`glyph_renderer.py`)

> **Why a soft mask?** A hard-edged mask creates a visible seam between the text and background — like cutting out text from a magazine and pasting it on a photo. A Gaussian-blurred soft mask feathers the boundary, creating a natural, seamless integration.

```python
def create_soft_mask(schema: dict, canvas_size: int = 1024,
                     blur_radius: int = 15) -> Image.Image:
    """
    Creates a Gaussian-blurred mask covering all text regions.
    Used to composite the ControlNet output (which has text) onto
    the clean background (which doesn't).
    
    How it works:
    1. Create a black canvas (all zeros = transparent)
    2. For each text block, draw a white rectangle with padding
       - padding=20px gives text some breathing room
       - text_width estimation: chars × font_size × 0.6
         (0.6 is average character aspect ratio for proportional fonts)
    3. Apply Gaussian blur with blur_radius=15px
       - This feathers the mask edge over ~30px
       - Result: smooth transition between text region and background
    
    blur_radius tuning:
    - Too low (< 5): Visible hard edges, "pasted-on" look
    - Too high (> 25): Text region bleeds too much into background,
                        may corrupt background details near text
    - 15px: Good balance — soft but defined
    
    The mask is in "L" (grayscale) mode:
    - White (255) = use the ControlNet Pass 2 output here (text region)
    - Black (0) = use the original background here
    - Gray = blend proportionally (the feathered edge)
    """
    # Start with a fully black (transparent) mask
    mask = Image.new("L", (canvas_size, canvas_size), 0)
    draw = ImageDraw.Draw(mask)

    for block in schema["texts"]:
        padding = 20  # Extra space around text for safer masking

        # Calculate bounding box for this text block
        x1 = max(0, block["x"] - padding)
        y1 = max(0, block["y"] - padding)

        # Estimate text width: character_count × font_size × avg_char_ratio
        # 0.6 is a reasonable average for most proportional fonts
        # This is an approximation — exact width would require font metrics
        text_width = len(block["content"]) * block["font_size"] * 0.6
        x2 = min(canvas_size, block["x"] + text_width + padding)
        y2 = min(canvas_size, block["y"] + block["font_size"] + padding)

        # Draw white rectangle for this text region
        draw.rectangle([x1, y1, x2, y2], fill=255)

    # Apply Gaussian blur using OpenCV for precise control over sigma
    # cv2.GaussianBlur with (0,0) kernel size = auto-size based on sigma
    mask_np = np.array(mask, dtype=np.float32)
    mask_np = cv2.GaussianBlur(mask_np, (0, 0), blur_radius)

    # Clip to valid range and convert back to PIL Image
    return Image.fromarray(np.clip(mask_np, 0, 255).astype(np.uint8))
```

-----

### Dynamic ControlNet Scale (`flux_generator.py`)

> **Why dynamic?** Small text needs stronger ControlNet conditioning to preserve fine details. Large headlines can tolerate looser conditioning and benefit from more natural style blending. A fixed scale would either over-constrain headlines or under-constrain fine print.

```python
def _get_controlnet_scale(self, resolved_blocks: list[dict],
                          canvas_size: int = 1024) -> float:
    """
    Calculates optimal ControlNet conditioning scale based on layout.
    
    Core principle: The SMALLEST text element determines the required scale.
    If any text is very small, we need strong conditioning for all text,
    because the ControlNet input is a single combined edge map.
    
    Scale ranges:
    ┌─────────────────────────────────────────────────────────┐
    │  relative_size = min_font_size / canvas_size             │
    │                                                         │
    │  < 0.05  (< 51px)   → base = 0.75  ← Very small text  │
    │  < 0.10  (51-102px) → base = 0.65  ← Medium text      │
    │  ≥ 0.10  (> 102px)  → base = 0.55  ← Large headlines  │
    │                                                         │
    │  Long lines (> 15 chars) → add 0.05 to base            │
    │  Maximum allowed: 0.80                                  │
    └─────────────────────────────────────────────────────────┘
    
    Official InstantX recommendation: 0.3–0.8 for text enforcement.
    We use the upper half (0.55–0.80) because text accuracy
    is our primary goal — we'd rather have correctly-shaped text
    that looks slightly artificial than beautifully-styled gibberish.
    
    Why prime adjustment for long lines?
    Long text lines have more opportunities for character drift.
    A line with 20 characters drifts more than a line with 5.
    Adding 0.05 provides just enough extra enforcement for long lines
    without pushing the scale too high for the headline area.
    """
    if not resolved_blocks:
        return 0.65  # Sensible default if schema has no text blocks

    # Find smallest font (most demanding rendering requirement)
    min_font = min(b["font_size"] for b in resolved_blocks)

    # Find longest line (most characters to keep correct)
    longest = max(len(b["content"]) for b in resolved_blocks)

    # Express font size relative to canvas (0.0 to 1.0)
    relative = min_font / canvas_size

    # Assign base scale from size bucket
    if relative < 0.05:    # Very small text — needs strong enforcement
        base = 0.75
    elif relative < 0.10:  # Medium text — moderate enforcement
        base = 0.65
    else:                  # Large headlines — lighter enforcement
        base = 0.55

    # Long text lines need slightly more enforcement
    if longest > 15:
        base += 0.05

    # Never exceed 0.80 — above this, text looks unnaturally harsh
    return round(min(base, 0.80), 2)
```

-----

### Conditional Contrast Boost (`quality_gate.py`)

> **Why conditional?** If the model already generated high-contrast text, boosting again causes over-sharpening artifacts (halos around letters, blocky edges). The condition check skips the boost when it’s not needed.

```python
def boost_text_contrast(image: Image.Image, soft_mask: Image.Image,
                        strength: float = 1.35) -> Image.Image:
    """
    Selectively boosts contrast in text regions ONLY when needed.
    
    The key innovation here is CONDITIONAL application:
    - Measure existing contrast in text regions
    - Apply boost only if contrast is below threshold
    - This prevents over-sharpening on already-legible text
    
    Why selective (mask-based) enhancement?
    - Enhancing the entire image would darken/flatten the background
    - Background should remain untouched — only text region matters
    - PIL's composite() function applies enhancement only within mask bounds
    
    Contrast measurement:
    - Convert image to grayscale
    - Extract pixel values where mask > 0.5 (inside text region)
    - Calculate standard deviation of those pixels
    - High std dev = high contrast (dark and light pixels coexist)
    - Low std dev = low contrast (similar pixel values throughout)
    
    Threshold of 65 (std dev):
    - std dev > 65: Already has sufficient contrast — skip boost
    - std dev ≤ 65: Low contrast — apply 1.35× enhancement
    - 65/255 ≈ 25% of maximum contrast range (empirically determined)
    
    strength=1.35:
    - 1.0 = no change, 2.0 = maximum (extreme) enhancement
    - 1.35 provides meaningful improvement without visible artifacts
    - Tested on 500+ generated images to find optimal value
    """
    # Convert image to grayscale for contrast measurement
    img_gray = np.array(image.convert("L"), dtype=np.float32)

    # Convert soft mask to float array (0.0 to 1.0)
    mask_arr = np.array(soft_mask.convert("L"), dtype=np.float32) / 255.0

    # Extract only pixels inside the text region
    text_pixels = img_gray[mask_arr > 0.5]

    if len(text_pixels) == 0:
        return image   # No text region found in mask — skip entirely

    # Measure contrast as standard deviation of grayscale values
    existing_contrast = text_pixels.std()

    # Skip if already high contrast — avoids over-sharpening artifacts
    if existing_contrast > 65:
        return image

    # Apply contrast enhancement to the full image
    enhanced = ImageEnhance.Contrast(image).enhance(strength)

    # Composite: text region uses enhanced version, rest stays original
    # soft_mask determines the blend at each pixel:
    #   mask=255 → take from enhanced
    #   mask=0   → take from original
    #   mask=128 → 50/50 blend (the feathered edge)
    result = Image.composite(enhanced, image, soft_mask.convert("L"))

    return result
```

-----

### Color Instructions in Schema (`llm_decomposer.py`)

> **Why separate color instructions?** The Flux prompt can specify color for text elements to guide ControlNet towards the right visual output. Without color guidance, the model might generate dark text on a dark background, making it unreadable even if the characters are correct.

```json
{
  "canvas_size": [1024, 1024],
  "texts": [
    {
      "content": "DIWALI SALE",
      "x": 350,
      "y": 200,
      "font_size": 72,
      "font_weight": "bold",
      "color_instruction": "bright white with gold outline"
    },
    {
      "content": "50% OFF",
      "x": 700,
      "y": 400,
      "font_size": 96,
      "font_weight": "black",
      "color_instruction": "gold metallic with subtle glow"
    }
  ],
  "background_prompt": "dark red festive background with gold bokeh particles",
  "style": "festival marketing poster"
}
```

**How color instructions integrate into the Pass 2 prompt:**

```python
def _build_pass2_prompt(self, schema: dict) -> str:
    """
    Builds the Flux prompt for Pass 2 by incorporating:
    1. Text content with attention weighting (content:1.6)
    2. Color instructions for each text block
    3. General typography quality descriptors
    4. Background style to maintain visual coherence with Pass 1
    
    The color_instruction is incorporated as a modifier before the text content.
    This order matters — Flux processes prompts sequentially, and placing
    color before content links the color to the specific text block.
    """
    prompt_parts = []

    # Add each text block with weight and color
    for block in schema["texts"]:
        content = block["content"]
        color = block.get("color_instruction", "")

        if color:
            # Color instruction before content: "bright white with gold outline text 'DIWALI SALE'"
            prompt_parts.append(
                f"({color} text '{content}':1.6)"
            )
        else:
            prompt_parts.append(f"(text '{content}':1.6)")

    # Add typography quality descriptors
    prompt_parts.extend([
        "(maximum contrast:1.3)",
        "(clearly readable sharp text edges:1.2)",
        "(professional typography:1.1)",
    ])

    # Add background style for visual coherence
    prompt_parts.append(schema.get("background_prompt", ""))

    return ", ".join(p for p in prompt_parts if p)
```

-----

##  Final Pipeline Architecture — Complete Diagram

```
                          USER BRIEF (plain language)
                                    │
                                    ▼
┌───────────────────────────────────────────────────────────────────────────┐
│                         STAGE 1: PREPARATION                               │
│                                                                            │
│   User Brief ──→ LLM Decomposer ──→ Layout Constraints Validator          │
│                                         │                                  │
│                        ┌────────────────┘                                  │
│                        │ Valid Schema                                       │
│                        ▼                                                   │
│              Glyph Renderer ──→ Clean Glyph Image (white+black text)      │
│                        │                                                   │
│                        ├──→ Canny Edge Map (ControlNet input)             │
│                        └──→ Soft Mask (blending map)                      │
└───────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌───────────────────────────────────────────────────────────────────────────┐
│                    STAGE 2: BACKGROUND GENERATION                          │
│                                                                            │
│   Background Prompt + "no text" negative                                   │
│        │                                                                   │
│        ▼                                                                   │
│   FluxPipeline (28 steps, no ControlNet)                                  │
│        │                                                                   │
│        ▼                                                                   │
│   Clean Background Image ─────────────────────────────────────────────→  │
└─────────────────────────────────────────────────────────┬─────────────────┘
                                                          │
                                                          ▼
┌───────────────────────────────────────────────────────────────────────────┐
│                      STAGE 3: TEXT INJECTION LOOP                          │
│                                                                            │
│  ┌─────────────────────────────────────────────────────────────────────┐  │
│  │  FluxControlNetPipeline (14 steps)                                   │  │
│  │  Input: background + canny_map                                       │  │
│  │  ControlNet scale: dynamic (0.55–0.80)                              │  │
│  └─────────────────────────────────────────────────────────────────────┘  │
│              │                                                             │
│              ▼                                                             │
│  Background Blending (soft mask composite)                                 │
│              │                                                             │
│              ▼                                                             │
│  Conditional Contrast Boost (if std_dev < 65)                              │
│              │                                                             │
│              ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐  │
│  │            OCR VERIFICATION GATE                                    │  │
│  │                                                                     │  │
│  │   Run EasyOCR → Compute NED vs expected text                       │  │
│  │                                                                     │  │
│  │   NED ≥ 0.85 ──────────────────────────────────────────────→ PASS │  │
│  │                                                                     │  │
│  │   NED < 0.85 → seed += 137 → retry (max 3 attempts)               │  │
│  │              ↑_____________________________________________|        │  │
│  └─────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┬─────────────────┘
                                                          │ PASS
                                                          ▼
┌───────────────────────────────────────────────────────────────────────────┐
│                      STAGE 4: FINAL OUTPUT                                 │
│                                                                            │
│   Visual Quality Gate (color balance, artifact check)                     │
│        │                                                                   │
│        ▼                                                                   │
│   RealESRGAN x2+ Upscaler (1024 → 2048 pixels)                           │
│   (runs in separate upscale-env to avoid dependency conflicts)            │
│        │                                                                   │
│        ▼                                                                   │
│   Save to pipeline/outputs/ (JPEG, quality=95)                            │
└───────────────────────────────────────────────────────────────────────────┘
```

-----

##  Setup & Installation

### Prerequisites

|Requirement|Minimum         |Recommended     |Notes                                    |
|-----------|----------------|----------------|-----------------------------------------|
|OS         |Ubuntu 20.04    |Ubuntu 22.04    |Other Linux distros may work; untested   |
|GPU        |NVIDIA 16GB VRAM|NVIDIA 24GB VRAM|RTX 3090/4090 or A100                    |
|Python     |3.10            |3.11            |f-strings and `match` statements required|
|CUDA       |12.1            |12.4            |Earlier versions may work; untested      |
|RAM        |32GB            |64GB            |Model weights + image buffers            |
|Storage    |50GB            |100GB           |Models alone require ~30GB               |

### VRAM Requirements Breakdown

|Operation                           |Minimum VRAM|Recommended|Notes                                     |
|------------------------------------|------------|-----------|------------------------------------------|
|Caption Generator (single image)    |8GB         |12GB       |Florence-2 + EasyOCR loaded simultaneously|
|Caption Generator (batch)           |12GB        |16GB       |Multiple models; prefetch next batch      |
|Image Generation (Pass 1 + 2 shared)|16GB        |24GB       |Flux + ControlNet Union, shared components|
|Full Pipeline with LoRA             |20GB        |24GB+      |Additional LoRA weight tensors in VRAM    |


> **VRAM optimization strategy:** Pass 1 and Pass 2 share the VAE, both text encoders (CLIP + T5), and the Flux transformer. Only the ControlNet weights are additional for Pass 2. This sharing saves approximately 15GB VRAM compared to loading two independent pipelines.

-----

### Step-by-Step Installation

#### Step 1 — Navigate to Project Root

```bash
cd /home/aiteam/ai_team/Animesh-Diffusion
```

#### Step 2 — Install Caption Generator Dependencies

```bash
# PyTorch with CUDA 12.4 support
# --index-url points to CUDA-specific wheels (faster download, correct CUDA version)
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu124

# HuggingFace ecosystem — required for Florence-2 model
pip install transformers accelerate

# OCR engines
pip install easyocr              # English + Hindi OCR
pip install opencv-python        # Image processing library
pip install pillow               # PIL — image loading and manipulation

# Optional: PaddleOCR (significantly improves accuracy on dense text)
# Install CUDA version of PaddlePaddle first
pip install paddlepaddle-gpu paddleocr

# Text processing and quality scoring
pip install wordfreq             # Word frequency for quality scoring
pip install rapidfuzz            # Fast fuzzy string matching for deduplication

# Utilities
pip install requests             # HTTP for Ollama API calls
pip install tqdm                 # Progress bars for batch processing
```

#### Step 3 — Install Image Generation Pipeline Dependencies

```bash
# The pipeline uses TWO separate virtual environments
# This is intentional — Flux and RealESRGAN have conflicting dependencies

# ── gen-env: Main generation environment ──────────────────────────────────
cd pipeline/scripts/gen-env
source bin/activate               # Activate the generation environment

# Diffusion model libraries
pip install diffusers             # HuggingFace Diffusers — Flux pipeline classes
pip install transformers          # CLIP and T5 text encoders
pip install torch torchvision     # PyTorch (same CUDA version as above)
pip install accelerate            # HuggingFace Accelerate — model loading utilities

# Pipeline utilities
pip install pyyaml                # config.yaml parsing
pip install pillow                # Image handling
pip install opencv-python         # Canny edge detection for glyph maps
pip install editdistance          # NED calculation for OCR verification

deactivate

# ── upscale-env: Upscaling environment ───────────────────────────────────
cd ../upscale-env
source bin/activate

# RealESRGAN requirements (different torch version compatibility)
pip install basicsr               # Base super-resolution framework
pip install realesrgan            # RealESRGAN implementation
pip install torch torchvision     # May need different version than gen-env

deactivate
```

#### Step 4 — Download Models

```bash
# ── Flux Dev ──────────────────────────────────────────────────────────────
# Source: https://huggingface.co/black-forest-labs/FLUX.1-dev
# Note: Requires HuggingFace account and gated model access agreement
mkdir -p models/flux-dev
# Use huggingface_hub or git-lfs to download (~23GB)
huggingface-cli download black-forest-labs/FLUX.1-dev --local-dir models/flux-dev

# ── ControlNet Union ─────────────────────────────────────────────────────
# Source: https://huggingface.co/InstantX/FLUX.1-ControlNet-Union-Pro
mkdir -p models/controlnet-union
huggingface-cli download InstantX/FLUX.1-ControlNet-Union-Pro \
    --local-dir models/controlnet-union

# ── RealESRGAN ────────────────────────────────────────────────────────────
# Source: https://github.com/xinntao/Real-ESRGAN/releases
mkdir -p models/realesrgan
# Download RealESRGAN_x2plus.pth (~67MB) and place in models/realesrgan/
wget https://github.com/xinntao/Real-ESRGAN/releases/download/v0.2.1/RealESRGAN_x2plus.pth \
    -O models/realesrgan/RealESRGAN_x2plus.pth
```

#### Step 5 — Install and Configure Ollama

```bash
# Install Ollama (official installer script)
curl -fsSL https://ollama.com/install.sh | sh

# Download Mistral 7B model (~4GB)
ollama pull mistral:7b

# Start Ollama server in background
# This must be running before caption_runner.py is executed
ollama serve &

# Verify it's running (should return JSON with model list)
curl http://localhost:11434/api/tags
```

#### Step 6 — Font Setup

```bash
# Place your custom fonts in the fonts/ directory
# Supported formats: .ttf (TrueType) and .otf (OpenType)
# Common font weights to include:
#   - Regular / Normal
#   - Bold
#   - Black / Heavy (for large headlines)
#   - Italic / Oblique

ls fonts/
# Expected output: Arial-Bold.ttf, Roboto-Black.ttf, NotoSans-Regular.ttf, etc.
```

-----

##  Configuration Reference

### Pipeline Configuration (`pipeline/config.yaml`)

```yaml
# ─── MODEL PATHS ──────────────────────────────────────────────────────────────
models:
  flux: "/home/aiteam/ai_team/Animesh-Diffusion/models/flux-dev"
  # Path to Flux Dev model directory (contains config.json, transformer/, etc.)

  controlnet: "/home/aiteam/ai_team/Animesh-Diffusion/models/controlnet-union"
  # Path to InstantX ControlNet Union model directory

  lora: null
  # Optional: Path to LoRA weights (.safetensors)
  # Set to null to disable LoRA, or provide absolute path to enable

  fonts_dir: "/home/aiteam/ai_team/Animesh-Diffusion/fonts"
  # Directory containing .ttf and .otf font files for glyph rendering

# ─── GENERATION PARAMETERS ────────────────────────────────────────────────────
generation:
  canvas_size: 1024
  # Output image resolution in pixels (square). 1024 = 1024×1024.
  # Increasing to 2048 requires ~4× VRAM. Use upscaling instead.

  pass1_steps: 28
  # Number of denoising steps for background generation (Pass 1).
  # More steps = higher quality but slower.
  # Range: 14 (fast/dev) to 50 (production quality).

  pass2_steps: 14
  # Number of denoising steps for text injection (Pass 2).
  # Lower than Pass 1 because we're refining, not generating from scratch.
  # Range: 10 (fast) to 20 (high quality).

  guidance_scale: 3.5
  # Classifier-free guidance scale. Higher = more adherent to prompt.
  # Flux's recommended range: 2.0–5.0. 3.5 is the sweet spot.
  # Higher values can cause over-saturation and artifacts.

  lora_scale: 0.85
  # LoRA weight when lora path is specified (0.0–1.0).
  # 1.0 = full LoRA effect, 0.0 = LoRA ignored.
  # 0.85 provides strong style guidance without overriding base model quality.

  soft_mask_blur: 15
  # Gaussian blur radius for soft mask generation (pixels).
  # Controls how soft the text/background transition is.
  # Increase for smoother blending; decrease for sharper text boundaries.

  min_font_size: 48
  # Minimum font size for any text block (pixels).
  # Text below this size will be scaled up to this minimum.
  # Prevents illegible micro-text in generated images.

  margin: 60
  # Minimum distance from any text to canvas edge (pixels).
  # Prevents text from being cut off at image borders.

  device: "cuda"
  # Compute device. Use "cuda" for GPU, "cpu" for CPU (very slow).

# ─── QUALITY CONTROL ──────────────────────────────────────────────────────────
quality:
  ocr_ned_threshold: 0.85
  # Minimum NED score for OCR verification to pass.
  # NED = 1 - (edit_distance / max_length). Range: 0.0–1.0.
  # 0.85 means ~85% character accuracy required.
  # Increase to 0.90 for stricter requirements (more retries).
  # Decrease to 0.80 for more permissive acceptance (fewer retries).

  max_pass2_retries: 3
  # Maximum number of retry attempts if OCR verification fails.
  # Each retry uses seed + 137 * attempt_number.
  # After max retries, image is saved with warning flag.

# ─── OUTPUT SETTINGS ──────────────────────────────────────────────────────────
output:
  upscale_factor: 2
  # RealESRGAN upscaling factor. 2 = 1024→2048, 4 = 1024→4096.
  # Factor 4 requires significantly more time and memory.

  jpeg_quality: 95
  # JPEG compression quality (1–100).
  # 95 provides excellent quality with moderate file size.
  # Use 100 for lossless (larger files), 85 for smaller files.

  output_dir: "/home/aiteam/ai_team/Animesh-Diffusion/pipeline/outputs"
  # Directory where final generated images are saved.
  # Created automatically if it doesn't exist.
```

### Configuration Tuning Guide

|Parameter          |Increase When                           |Decrease When                            |Typical Range|
|-------------------|----------------------------------------|-----------------------------------------|-------------|
|`pass1_steps`      |Background quality is poor              |Generation is too slow                   |14–50        |
|`pass2_steps`      |Text is blurry or poorly styled         |Too slow for iteration                   |10–20        |
|`guidance_scale`   |Output doesn’t match brief              |Images look over-saturated               |2.0–5.0      |
|`ocr_ned_threshold`|Text quality requirements are high      |Too many images failing (> 30%)          |0.80–0.92    |
|`soft_mask_blur`   |Visible seam between text and background|Text area bleeds into background too much|8–25px       |
|`min_font_size`    |Text in outputs is too small to read    |Large text only, don’t need small        |36–72px      |
|`lora_scale`       |LoRA style isn’t strong enough          |LoRA overpowers base quality             |0.60–1.0     |

-----

##  Usage Guide

### Caption Generator

#### Basic Batch Run

```bash
# Process ALL images in current directory and subdirectories
# Generates .txt caption files alongside each image
# Automatically resumes from last checkpoint if interrupted

python caption_runner.py
```

The script will:

1. Scan for `.jpg`, `.jpeg`, `.png`, `.webp` files recursively
2. Load progress from `caption_progress.json` (if exists) and skip completed images
3. Process each image through the full OCR + LLM pipeline
4. Save caption as `<image_name>.txt` in the same directory
5. Update `caption_progress.json` after each image

#### Testing Individual Images

```bash
# Test 5 random images from batch_1 folder
python caption_test.py --folder ./batch_1 --n 5

# Test with dense-text optimizations (festival posters, sale banners)
python caption_test.py --folder ./batch_1 --n 5 --dense-mode

# Test with fixed random seed for reproducible results
python caption_test.py --folder ./batch_1 --n 5 --seed 42

# All options combined
python caption_test.py --folder ./batch_3 --n 10 --dense-mode --seed 100
```

|Flag          |Type|Default    |Description                                                  |
|--------------|----|-----------|-------------------------------------------------------------|
|`--folder`    |path|current dir|Directory containing images to test                          |
|`--n`         |int |5          |Number of randomly-selected images to process                |
|`--seed`      |int |random     |Random seed for image selection (reproducibility)            |
|`--dense-mode`|flag|off        |Enable optimizations for images with many small text elements|

#### Splitting Images into Batches

```bash
# Splits all images in current directory into batch_1/, batch_2/, ... subdirectories
# Each batch contains exactly 100 images
# Run this BEFORE caption_runner.py for better organization

python split_batches.py
```

-----

### Image Generation Pipeline

#### Basic Usage

```bash
# Activate the generation virtual environment first
cd pipeline/scripts
source gen-env/bin/activate

# Generate a single image from a text brief
python pipeline_runner.py --brief "Diwali sale poster with dark red background, gold bokeh particles, text '50% OFF' prominent"
```

#### Advanced Usage Examples

```bash
# Wedding invitation with fixed seed (reproducible)
python pipeline_runner.py \
    --brief "Elegant wedding invitation, cream background, floral border, text 'Sharma Family Wedding'" \
    --seed 42

# Festival poster without upscaling (faster iteration during development)
python pipeline_runner.py \
    --brief "Holi celebration poster, colorful splash background, text 'HAPPY HOLI'" \
    --no-upscale

# Debug mode — saves all intermediate files for inspection
python pipeline_runner.py \
    --brief "Sale poster test" \
    --debug \
    --output ./debug_outputs/test_001.jpg

# Custom output path
python pipeline_runner.py \
    --brief "Restaurant menu banner, warm lighting, text 'Today's Special'" \
    --output /home/aiteam/marketing/menu_banner.jpg
```

|Flag          |Type  |Required|Description                                                     |
|--------------|------|--------|----------------------------------------------------------------|
|`--brief`     |string| Yes   |Plain language description of desired image                     |
|`--output`    |path  |No      |Custom output file path (default: auto-named in outputs/)       |
|`--seed`      |int   |No      |Random seed for reproducibility (default: random)               |
|`--no-upscale`|flag  |No      |Skip RealESRGAN upscaling (faster; outputs at 1024px)           |
|`--debug`     |flag  |No      |Save all intermediate images (glyph, canny, background, retries)|

#### Debug Mode Output Files

When `--debug` is specified, the following intermediate files are saved:

```
debug_glyph.png           — The pre-rendered text image (white background, black text)
debug_canny.png           — The Canny edge map fed into ControlNet
debug_background.png      — Pass 1 output: clean background without text
debug_pass2_attempt1.jpg  — First Pass 2 attempt (before OCR verification)
debug_pass2_attempt2.jpg  — Second attempt (if OCR failed first time, seed+137)
debug_pass2_attempt3.jpg  — Third attempt (if OCR failed second time, seed+274)
```

> **When to use debug mode:** Use it during development to understand why a brief is producing unexpected results. Inspect `debug_glyph.png` first — if the text positions are wrong, the issue is in the LLM decomposer. If positions are correct but the final image looks wrong, the issue is in the generation parameters.

#### Example Briefs — Quality vs Detail

```bash
# ── Festival Poster ────────────────────────────────────────────────────────

# Minimal brief (works, but less predictable layout):
"Diwali sale poster with gold theme"

# Detailed brief (recommended — more control):
"Diwali sale poster, dark crimson background with golden particle bokeh effects,
warm festive atmosphere, text 'DIWALI SALE' centered at top in large bold font,
text '50% OFF' at right side in very large display font, text 'Shop Now' at
bottom center as call-to-action"

# ── Wedding Invitation ─────────────────────────────────────────────────────

# Minimal:
"Wedding invitation Sharma family"

# Detailed:
"Elegant wedding invitation card, ivory/cream background with soft floral watercolor
border, romantic lighting, text 'The Sharma Family' as main header at top center,
text 'Wedding Celebration' as subtitle, text 'December 15, 2024' for date,
text 'Grand Ballroom, Hotel Leela' for venue"

# ── Concert Poster ─────────────────────────────────────────────────────────

# Detailed:
"Rock concert poster, dark black background with dramatic stage lighting in blue
and purple, moody atmosphere, text 'LIVE CONCERT' as main headline at top,
text 'The Iron Wolves' as band name in center, text 'March 15 | 8PM' for details"
```

-----

##  Troubleshooting & Best Practices

### Common Issues and Solutions

#### Issue 1 — Ollama Connection Error

**Symptom:**

```
ERROR: LLM failed to produce valid schema after 3 attempts
ConnectionRefusedError: [Errno 111] Connection refused
```

**Diagnosis:** Ollama server is not running.

**Solution:**

```bash
# Start Ollama server
ollama serve

# Verify it's responding (should return JSON)
curl http://localhost:11434/api/tags

# If using a different port, update OLLAMA_URL in caption_runner.py:
# OLLAMA_URL = "http://localhost:YOUR_PORT"

# Check Ollama logs if issues persist
journalctl -u ollama --since "5 minutes ago"
```

-----

#### Issue 2 — CUDA Out of Memory

**Symptom:**

```
RuntimeError: CUDA out of memory. 
Tried to allocate 2.50 GiB (GPU 0; 15.90 GiB total capacity; 
14.20 GiB already allocated)
```

**Solutions (try in order):**

```bash
# 1. Reduce batch size for caption generator
# Edit caption_runner.py: find chunk_list(todo, 100) → change 100 to 50

# 2. Enable sequential CPU offloading in flux_generator.py
# Add this line after model loading:
flux_pipeline.enable_sequential_cpu_offload()
# (Slower but uses much less VRAM)

# 3. Clear GPU memory between runs
python -c "import torch; torch.cuda.empty_cache()"

# 4. Check what's using VRAM
nvidia-smi

# 5. Kill other GPU processes if needed
sudo fuser -v /dev/nvidia*
```

-----

#### Issue 3 — OCR Verification Failing Repeatedly

**Symptom:**

```
OCR gate failed — attempt 1/3. Best NED: 0.72 (threshold: 0.85)
OCR gate failed — attempt 2/3. Best NED: 0.71 (threshold: 0.85)
OCR gate failed — attempt 3/3. Best NED: 0.69 (threshold: 0.85)
Saving best attempt (NED=0.72) with warning flag.
```

**Diagnose:**

```bash
# Use debug mode to inspect what's happening
python pipeline_runner.py --brief "your brief" --debug --seed 42

# Check debug_glyph.png: Is the text correctly rendered?
# Check debug_canny.png: Are the edges clean?
# Check debug_pass2_attempt1.jpg: What does the text look like?
```

**Solutions:**

```bash
# 1. Try a different seed
python pipeline_runner.py --brief "..." --seed 1234

# 2. Simplify the brief — remove complex layout requirements
# Bad:  "text 'DIWALI SALE' at exactly 350,200 in 72px bold italic with shadow"
# Good: "text 'DIWALI SALE' prominent at top"

# 3. Reduce OCR threshold temporarily for iteration
# In config.yaml: ocr_ned_threshold: 0.80

# 4. Check if text content has special characters
# Some characters (ä, ñ, ©) are harder for OCR to verify
# Consider replacing with simpler ASCII equivalents for testing
```

-----

#### Issue 4 — PaddleOCR Not Available (Informational)

**Symptom:**

```
INFO: PaddleOCR not available — running in 3-source mode (Florence + EasyOCR + Regional)
```

**This is NOT an error.** The system continues normally with three OCR sources. PaddleOCR is optional.

**To enable:**

```bash
pip install paddlepaddle-gpu paddleocr
# Then restart caption_runner.py — it will auto-detect and use PaddleOCR
```

-----

#### Issue 5 — Font Not Found / Rendering Issues

**Symptom:**

```
WARNING: Requested font 'bold' not found in fonts/ directory
INFO: Falling back to system font
INFO: System font also unavailable — using PIL default font
```

**Solutions:**

```bash
# 1. Add fonts to the fonts/ directory
ls fonts/  # Check what's there

# 2. Download common fonts
# For Indian language support + bold variants:
# - Noto Sans: https://fonts.google.com/noto/specimen/Noto+Sans
# - Roboto: https://fonts.google.com/specimen/Roboto

# 3. Check font file permissions
chmod 644 fonts/*.ttf fonts/*.otf

# 4. Verify font files are valid
python -c "from PIL import ImageFont; ImageFont.truetype('fonts/YOUR_FONT.ttf', 72)"
```

-----

### Best Practices

#### For Caption Generator

1. **Pre-process images before batching:**
- Minimum size: 512×512 pixels (smaller images give poor OCR results)
- Remove duplicate images before running (wasted compute)
- Separate Hindi/English-only images into different batches if domain accuracy matters
1. **Batch size tuning:**
- Start with `chunk_list(todo, 50)` if experiencing memory issues
- Increase to 100 once stability is confirmed
- Monitor GPU memory with `watch nvidia-smi` during first run
1. **Retry strategy:**
   
   ```bash
   # After initial run, process retry_needed.txt with dense-mode
   python caption_test.py --folder @retry_needed.txt --dense-mode
   
   # Review failed_images.txt to understand failure patterns
   cat failed_images.txt | grep "too_small" | wc -l  # Count size failures
   cat failed_images.txt | grep "corrupted" | wc -l  # Count corruption failures
   ```
2. **Quality review:** Spot-check 5-10% of generated captions manually before using for training. Look for: missing text, wrong domain detection, garbled cleanup.

-----

#### For Image Generation Pipeline

1. **Seed management:**
   
   ```bash
   # Use fixed seeds during development for reproducibility
   python pipeline_runner.py --brief "test brief" --seed 42
   
   # Log successful seeds for good results
   echo "42: diwali_poster_good_result.jpg" >> seed_log.txt
   ```
2. **Brief quality — the two most important things:**
- Be explicit about text content: `text 'EXACT TEXT HERE'`
- Describe the background without mentioning text in it
1. **Development iteration workflow:**
   
   ```bash
   # Step 1: Test with no-upscale for speed
   python pipeline_runner.py --brief "..." --no-upscale --debug --seed 42
   
   # Step 2: Inspect debug files to diagnose issues
   # Step 3: Adjust brief or parameters
   # Step 4: Once happy, run without --no-upscale for final output
   python pipeline_runner.py --brief "..." --seed 42
   ```
2. **Batch generation for production:**
   
   ```bash
   # Run multiple briefs sequentially
   # Consider adding a small delay between runs to prevent VRAM fragmentation
   for brief in "${briefs[@]}"; do
       python pipeline_runner.py --brief "$brief"
       sleep 2
   done
   ```

-----

##  Future Improvements & Next Steps

### Caption Generator — Planned Enhancements

|Priority|Enhancement                      |Description                                           |Estimated Impact                          |
|--------|---------------------------------|------------------------------------------------------|------------------------------------------|
| High  |Additional OCR sources           |Add Tesseract + Google Vision API for better coverage |+10-15% text detection accuracy           |
| High  |Hindi/Regional language expansion|Expand beyond Hindi to Tamil, Telugu, Kannada, Bengali|Opens 4 new language markets              |
|Medium|Configurable output formats      |Add JSON, CSV, JSONL export options                   |Better integration with training pipelines|
|Medium|GPU-accelerated preprocessing    |Replace CPU-based OpenCV operations with CUDA variants|3-5× faster preprocessing                 |
| Low   |Web UI dashboard                 |Streamlit interface for monitoring batch progress     |Better UX for non-technical operators     |

-----

### Image Generation Pipeline — Planned Enhancements

|Priority|Enhancement               |Description                                                             |Estimated Impact                              |
|--------|--------------------------|------------------------------------------------------------------------|----------------------------------------------|
| High  |LoRA fine-tuning support  |Domain-specific style adaptation (wedding, festival, etc.)              |Dramatically improved visual style consistency|
| High  |Async batch processing    |Generate multiple images concurrently using separate GPU streams        |3-5× throughput improvement                   |
| Medium|Adaptive seed selection   |Learn from failure patterns to select seeds more likely to pass OCR     |Fewer retries, faster generation              |
| Medium|Text position optimization|Auto-layout engine that avoids overlapping text with background subjects|Fewer layout correction iterations            |
| Medium|Visual quality gate       |Automatic checks for color balance, artifacts, composition              |Fewer bad outputs reaching production         |
| Low   |Web UI (Streamlit/Gradio) |Visual interface for submitting briefs and viewing results              |Accessible to non-technical team members      |
| Low   |REST API                  |HTTP endpoint for integration with existing marketing tools             |Integration with Canva, Figma plugins         |

-----

### Immediate Next Steps (Production Deployment)

**Phase 1 — Observation (Week 1-2):**

```
1. Generate 100-200 outputs across different domains to identify failure patterns:
   - Which font weights cause text drift?
   - Which background colors cause insufficient contrast?
   - Which LLM output positions cause text overlap?
   - What's the actual NED distribution? (How often does it fail on first attempt?)
```

**Phase 2 — Threshold Tuning (Week 2-3):**

```
Based on Phase 1 observations:
  - Adjust ocr_ned_threshold (currently 0.85) based on observed distribution
  - Tune contrast boost threshold (currently std_dev=65) for common failure backgrounds
  - Adjust ControlNet scale ranges based on which font sizes are failing
```

**Phase 3 — Domain LoRA (Week 3-6):**

```
If consistent domain-specific visual style is required:
  - Use generated captions from caption_runner.py as training data
  - Fine-tune Flux LoRA for each domain (wedding, festival, sale)
  - Add LoRA path to config.yaml for each domain
```

**Phase 4 — Production Hardening (Week 4-8):**

```
  - Containerize with Docker for reproducible deployment
  - Add structured logging and metrics collection
  - Implement CI/CD pipeline with automated test runs
  - Write API reference documentation and video tutorials
```

-----

##  Quick Reference & Cheat Sheet

### File Locations

|Purpose                       |Path                                 |
|------------------------------|-------------------------------------|
|Caption generation entry point|`caption_runner.py`                  |
|Caption testing tool          |`caption_test.py`                    |
|Image generation entry point  |`pipeline/scripts/pipeline_runner.py`|
|Pipeline configuration        |`pipeline/config.yaml`               |
|Generated outputs             |`pipeline/outputs/`                  |
|Model storage                 |`models/`                            |
|Custom fonts                  |`fonts/`                             |
|Progress tracking             |`caption_progress.json`              |
|Retry candidates              |`retry_needed.txt`                   |
|Failed images                 |`failed_images.txt`                  |

-----

### Commands Cheat Sheet

```bash
# ─── CAPTION GENERATION ────────────────────────────────────────────────────

# Process all images in current directory (with resume support)
python caption_runner.py

# Test 5 images from a specific folder
python caption_test.py --folder ./batch_1 --n 5

# Test with dense-text mode and fixed seed
python caption_test.py --folder ./batch_1 --n 5 --dense-mode --seed 42

# Organize images into batches of 100
python split_batches.py


# ─── IMAGE GENERATION ──────────────────────────────────────────────────────

# Activate generation environment (REQUIRED before running pipeline)
cd pipeline/scripts && source gen-env/bin/activate

# Basic generation
python pipeline_runner.py --brief "your image description here"

# With seed (reproducible), debug mode, no upscaling (fast iteration)
python pipeline_runner.py --brief "..." --seed 42 --debug --no-upscale

# Production run (with upscaling, custom output path)
python pipeline_runner.py --brief "..." --seed 42 --output ./outputs/final.jpg


# ─── MODEL SETUP ───────────────────────────────────────────────────────────

# Pull Mistral model for Ollama
ollama pull mistral:7b

# Start Ollama server (must be running for caption generation)
ollama serve

# Verify Ollama is running
curl http://localhost:11434/api/tags


# ─── MONITORING ────────────────────────────────────────────────────────────

# Watch GPU usage in real time
watch -n 1 nvidia-smi

# Check caption generation progress
cat caption_progress.json | python -m json.tool | grep -c '"done"'

# View failed images
cat failed_images.txt

# View retry candidates
cat retry_needed.txt
```

-----

### Architecture Decision Log

|Decision                                    |Alternative Considered             |Reason Chosen                                                     |
|--------------------------------------------|-----------------------------------|------------------------------------------------------------------|
|Flux over SDXL                              |SDXL was mature and stable         |T5-XXL + 16ch VAE in Flux are fundamental improvements for text   |
|ControlNet Union over individual ControlNets|Could use separate canny ControlNet|Union model handles multiple modes; simpler dependency management |
|Separate venvs (gen-env, upscale-env)       |Single environment for everything  |Flux and RealESRGAN have irreconcilable PyTorch version conflicts |
|EasyOCR over Tesseract for verification     |Tesseract is more established      |EasyOCR supports Hindi; needed for bilingual poster verification  |
|NED over exact match for OCR gate           |Exact match would be simpler       |OCR detection has inherent variance; NED is more robust metric    |
|4 OCR sources over 1                        |Single-source is simpler           |No single engine achieves >85% on all poster types; fusion needed |
|Mistral 7B over GPT-4 for cleanup           |GPT-4 has better accuracy          |Mistral 7B via Ollama: zero latency, no API costs, privacy        |
|Seed +137 for retry                         |Sequential +1 or random            |Prime increment ensures maximal distance; still deterministic     |
|Soft mask with blur=15 over hard mask       |Hard mask is simpler               |Gaussian blur eliminates visible seams at text/background boundary|

-----

*End of Knowledge Transfer Document*

-----

