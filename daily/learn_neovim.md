;# Learning Neovim with the Built-in Help

Neovim has a comprehensive and well-structured documentation that you can access from within the editor. This guide will show you how to use the `:help` command to learn everything from basic movements to advanced scripting.

## The Absolute Beginner's Start: The Tutor

If you are completely new to Vim/Neovim, the best place to start is the built-in tutor. It is an interactive session that teaches the very basics of editing.

```vim
:Tutor
```

## The Core of Learning: `:help`

The `:help` command is your gateway to the documentation. You can use it to get information on almost any topic.

```vim
:help {topic}
```

For example, to learn about Normal mode:

```vim
:help normal-mode
```

### Navigating the Help System

The help pages are interconnected. You can navigate them with these keybindings:

- `CTRL-]` - Jump to the definition of the topic under the cursor.
- `CTRL-T` - Go back to the previous topic.
- `/` - Search for text within the current help page.

## A Structured Path: The User Manual

For a structured learning experience, Neovim provides a user manual that you can read from start to finish.

```vim
:help user-manual
```

This command will take you to the table of contents for the user manual. It is organized in a way that you can start from the basics and gradually move to more advanced topics.

## Learning Specifics

### Motions and Text Objects

To learn about motions (how to move around) and text objects (how to operate on blocks of text), you can use these commands:

```vim
:help motion.txt
:help text-objects
```

### Learning about Plugins

Most well-written plugins come with their own help files. You can access them using the `:help` command followed by the plugin's name. For example, for the popular `fugitive` plugin:

```vim
:help fugitive
```

## Discovering New Commands with `:helpgrep`

If you don't know the exact name of a command or topic, you can search through all the help files using `:helpgrep`.

```vim
:helpgrep {pattern}
```

For example, to find all help pages that mention "git":

```vim
:helpgrep git
```

This will open a quickfix list with all the matches. You can navigate this list to find the information you need.

## Conclusion

The built-in help system is the most powerful tool at your disposal for learning Neovim. By mastering how to use `:help` and `:helpgrep`, you can find the answer to almost any question you might have about the editor.
