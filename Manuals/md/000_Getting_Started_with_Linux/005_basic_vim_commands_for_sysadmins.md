# Basic `vim` Commands for Sysadmins

## Contents

- [Basic `vim` Commands for Sysadmins](#basic-vim-commands-for-sysadmins)
  - [Contents](#contents)
  - [Introduction](#introduction)
    - [Understanding Vim Modes](#understanding-vim-modes)
  - [Navigating Files](#navigating-files)
    - [Normal Mode](#normal-mode)
    - [Searching Within a File](#searching-within-a-file)
    - [Jumping to Specific Lines](#jumping-to-specific-lines)
  - [Basic Editing Operations](#basic-editing-operations)
    - [Insert Mode](#insert-mode)
    - [Deleting Text](#deleting-text)
    - [Copy and Paste](#copy-and-paste)
    - [Undo and Redo](#undo-and-redo)
  - [Saving and Exiting Vim](#saving-and-exiting-vim)
  - [Practical Tips](#practical-tips)
    - [Command-line Mode](#command-line-mode)

## Introduction

### Understanding Vim Modes

Unlike many modern text editors you might be familiar with, Vim operates in different "modes." Think of it like a specialized tool with various settings, where each setting allows you to perform a specific type of action. You can't just start typing right away when you open Vim; you need to be in the correct mode for what you want to do.

Here are the main modes we'll focus on:

1. **Normal Mode (or Command Mode)**: This is the default mode when you open Vim. In Normal Mode, your keyboard inputs are interpreted as commands to move the cursor, delete text, copy text, paste text, or switch to other modes. This is where Vim's power for quick, efficient text manipulation truly shines. It's like the "control panel" of Vim.

2. **Insert Mode**: This is the mode you need to be in to actually type and insert text into your file, similar to how most other text editors work. You typically enter Insert Mode from Normal Mode using commands like `i` (insert at cursor) or `a` (append after cursor).

3. **Visual Mode**: This mode allows you to select blocks of text, similar to highlighting text with a mouse in other editors. Once selected, you can perform operations like copying, deleting, or changing the selected text.

4. **Command-line Mode (or Last-line Mode)**: You enter this mode by typing : (colon) from Normal Mode. This mode is used for entering special commands, such as saving files, quitting Vim, searching for text, or running external shell commands. The commands you type appear at the bottom of the Vim window.

**Why are modes crucial for sysadmins?**

Imagine you're quickly editing a server configuration file. Instead of constantly reaching for your mouse to select text or navigating with arrow keys, Vim's Normal Mode commands allow you to perform these actions with just a few keystrokes, keeping your hands on the keyboard and your workflow efficient. This speed and efficiency are invaluable when managing remote servers, especially over SSH where graphical interfaces might not be available or are slower.

## Navigating Files

### Normal Mode

In **Normal Mode**, Vim offers incredibly efficient ways to move your cursor without using arrow keys (though arrow keys usually work too!). The traditional Vim movement keys are right there on the home row, which is a fantastic advantage for speed:

- `h`: Move left (like your left hand on the keyboard)
- `j`: Move down (the 'j' key often has a small bump, helping you find it without looking, and it points downwards)
- `k`: Move up (just above 'j')
- `l`: Move right (like your right hand on the keyboard)

Beyond single character movements, you can also move by words:

- `w`: Move to the beginning of the next word
- `b`: Move to the beginning of the previous word
- `e`: Move to the end of the current word

And for line-specific navigation:

- `0` (zero) or `^`: Move to the beginning of the current line (the very first character). `^` specifically moves to the first non-blank character.
- `$`: Move to the end of the current line.

Finally, for larger jumps:

- `gg`: Move to the beginning of the file (the very first line).
- `G`: Move to the end of the file (the very last line).

Imagine you're reviewing a `/etc/ssh/sshd_config` file. Instead of hitting the down arrow a hundred times, `G` gets you to the end instantly. If you realize you need to check the first few lines, `gg` takes you right back to the top.

To help visualize this, imagine a simple text file:

```text
This is a sample configuration file.
It has multiple lines of settings.
We will practice moving around in this text.
```

If your cursor is currently on the 's' in "sample" on the first line:

- Typing `w` would move your cursor to the 'c' in "configuration".
- Typing `j` would move your cursor down to the 'I' in "It" on the second line.
- Typing `$` would move your cursor to the '.' at the end of the first line.

### Searching Within a File

Imagine you're troubleshooting an issue on a server and need to find all instances of a specific error message in a log file. Vim provides powerful search capabilities right from **Normal Mode**:

- `/search_term`: To search forward through the file for `search_term`. After typing `/`, you'll see the cursor jump to the bottom of the screen where you can type your search pattern. Press `Enter` to initiate the search.
- `?search_term`: To search backward through the file for `search_term`.

Once you've found an instance of your search term, you can quickly jump to the next or previous occurrences:

- `n`: Move to the **next** occurrence of the search term (in the same direction as your initial search).
- `N`: Move to the **previous** occurrence of the search term (in the opposite direction of your initial search).

### Jumping to Specific Lines

Sometimes you'll know exactly which line number you need to go to, perhaps from an error message that specifies a line.

- `:[line_number]`: From Command-line Mode (remember, you enter this by typing `:` in Normal Mode), you can type a line number and press Enter to jump directly to that line. For example, `:50` will take you to line `50`.
- `:set nu` (or `:set number`): This command will display line numbers along the left side of your Vim window. This is temporary for your current Vim session, but it's invaluable for orientation and when dealing with error messages that reference line numbers. To turn them off, you can use `:set nonu`.

These commands will help you quickly locate the exact content or line you need to modify or review.

## Basic Editing Operations

### Insert Mode

This section is all about modifying text. We'll start with how to **insert text**, which means getting into **Insert Mode** from **Normal Mode**. Remember, you can't just type in Normal Mode – your keystrokes are commands!

Here are the key commands to enter Insert Mode, each placing your cursor slightly differently, which can be a huge time-saver:

- `i`: **Insert** text **at** the current cursor position. This is the most common way to switch to Insert Mode.
- `I`: Insert text at the **beginning of the current line**. No matter where your cursor is on the line, `I` will jump it to the first non-blank character and put you in Insert Mode.
- `a`: **Append** text **after** the current cursor position. If your cursor is on a character, typing `a` will move it one space to the right and then let you type.
- `A`: Append text at the **end of the current line**. This is very useful when you want to quickly add something to the end of a line without having to navigate there first.
- `o`: Open a **new line below** the current line and enter Insert Mode. Perfect for adding a new configuration directive on its own line.
- `O`: Open a **new line above** the current line and enter Insert Mode. Also great for adding a new line, but above your current position.

**Important**: Once you're in Insert Mode and finished typing, always press the `Esc` key to return to **Normal Mode**. This is a fundamental habit you'll quickly develop.

Think of it like this: if you're writing a new entry in a log file, `o` would give you a fresh line to start typing. If you just need to add a small detail to the end of an existing configuration parameter, `A` is your friend.

### Deleting Text

All of these commands are executed from **Normal Mode**. If you're in **Insert Mode**, remember to press `Esc` to switch back to **Normal Mode** before attempting to delete.

Here are the essential deletion commands:

- `x`: Delete the single character under the cursor. Think of it like hitting the `Delete` key on a standard keyboard.
- `X`: Delete the single character before the cursor. Similar to hitting `Backspace`.
- `dw`: Delete from the current cursor position to the **beginning of the next word** (including the space after the word).
- `de`: Delete from the current cursor position to the **end of the current word**.
- `d$`: Delete from the current cursor position to the **end of the current line**.
- `dd`: Delete the **entire current line**. This is one of the most frequently used deletion commands in Vim for sysadmins!

**Combining commands with numbers**: You can also prefix many commands with a number to repeat the action. For example:

- `5x`: Delete 5 characters from the cursor.
- `3dd`: Delete 3 entire lines starting from the current line.

Let's look at an example. Suppose you have this line in a configuration file:

```Bash
# This is an old configuration entry.
```

- If your cursor is on the `#` and you type `dw`, it will delete `# This`.
- If your cursor is on the 'o' in "old" and you type `de`, it will delete "old".
- If your cursor is anywhere on that line and you type `dd`, the entire line will be removed.

These deletion commands are very powerful for quickly tidying up or modifying configuration files.

### Copy and Paste

Just like deletion, these commands are executed from **Normal Mode**.

- `yy`: **Yank** (copy) the **entire current line**. This is one of the most common copy commands.
- `p`: **Put** (paste) the yanked (copied) text **after** the cursor or **on the line below** the current line if you copied full lines.
- `P`: **Put** (paste) the yanked (copied) text **before** the cursor or **on the line above** the current line if you copied full lines.

You can also combine `y` with movement commands, similar to `d` for delete:

- `yw`: **Yank** from the current cursor position to the **beginning of the next word**.
- `y$`: **Yank** from the current cursor position to the **end of the current line**.

And, of course, you can prefix with a number to yank multiple lines:

- `5yy`: Yank (copy) 5 lines, starting from the current line.

Let's imagine you have a block of configuration settings that you want to duplicate or move to another part of your file.

Original text:

```Bash
# Server settings
Port 22
ListenAddress 0.0.0.0
```

If your cursor is on `Port 22` and you type `yy`, that line is copied. Then, if you move your cursor to a new location and type `p`, the line `Port 22` will be pasted below the line where your cursor currently is. If you type `P`, it will be pasted above.

For sysadmins, this is invaluable when you're setting up similar configurations for multiple services or user accounts, or when you need to quickly comment out a line by duplicating it and then modifying one of the copies.

### Undo and Redo

Mistakes happen, especially when you're learning a new editor! Fortunately, Vim has robust undo and redo capabilities, allowing you to easily revert changes or reapply them. These commands are executed from **Normal Mode**.

- `u`: **Undo** the last change. This is your "oops" button. If you accidentally delete a line or make a change you regret, simply type `u`, and Vim will revert it. Vim keeps a history of your changes, so you can press `u` multiple times to undo several previous actions.

- `Ctrl-r`: **Redo** a previously undone change. If you undo something and then realize you actually wanted it back, `Ctrl-r` (hold `Ctrl` and press `r`) will reapply it. Think of it as the "oops, never mind" button after pressing "oops."

These two commands are fundamental for any text editing, but particularly when you're making critical changes to system configuration files. It gives you a safety net to experiment or correct errors without fear of irreversible damage.

For example, if you typed `dd` and deleted a line, a quick `u` brings it right back. If you then decide you did want it deleted, `Ctrl-r` will remove it again.

## Saving and Exiting Vim

All these commands are entered from **Normal Mode** by first typing a colon (`:`) which brings you into Command-line Mode at the bottom of the screen.

Here are the essential commands:

- `:w` (write): This command saves the changes you've made to the file, but keeps you in Vim. You'll often use this if you want to save periodically while still working.
- `:q` (quit): This command exits Vim. However, it will only work if you haven't made any unsaved changes. If you have unsaved changes, Vim will warn you.
- `:wq` (write and quit): This is a very common command that saves your changes and then exits Vim. It's a shorthand for `:w` followed by `:q`.
- `:x`: This command is similar to `:wq`. It saves the file only if changes have been made, and then exits Vim. It's slightly more "intelligent" than `:wq` in that it won't touch the file's modification timestamp if no changes were actually made. For daily use, `:wq` and `:x` are largely interchangeable for saving and quitting.
- `:q!` (quit forcefully): This command exits Vim without saving any changes. The `!` means "force." Use this when you've made a mess and just want to abandon all changes and revert to the last saved version of the file. Be careful with this one, as any unsaved work will be lost!

**A word of caution for sysadmins**: When editing critical system files (like `/etc/fstab` or `/etc/network/interfaces`), always double-check your changes before saving. And never use :q! unless you are absolutely sure you want to discard everything!

## Practical Tips

### Command-line Mode

Sometimes, while you're in Vim editing a file, you might need to quickly execute a shell command without actually quitting Vim. This is where the `!` command in Command-line Mode comes in handy.

- `:!command`: This allows you to execute an external shell `command` from within Vim. The output of the command will be displayed in your terminal, and then you'll be returned to your Vim session.

For example, if you're editing a configuration file and want to quickly check the status of a service without leaving Vim, you could type:

```vim
:!systemctl status sshd
```

Or, if you just saved a change and want to immediately apply it by restarting a service:

```vim
:!sudo systemctl restart nginx
```

This command is incredibly powerful because it lets you interact with the underlying system while staying focused on your editing task. It prevents the need to constantly exit Vim, run a command, and then re-open Vim.
