# Lab Sheet 5: vi/vim Editor

Vim has three commonly used modes: **Normal** (commands), **Insert** (enter text), and **Command-Line** (commands beginning with `:`). On a minimal Ubuntu installation, install Vim with `sudo apt update && sudo apt install vim`.

**Q1. Create `lab5.txt` with vi/vim and enter at least five lines.**
```text
vim lab5.txt
i
Enter at least five lines of text.
Press Esc, type :wq, then press Enter.
```

**Q2. Open an existing `lab5.txt` and display its contents.**
```bash
vim lab5.txt
cat lab5.txt
```

**Q3. Switch between Normal, Insert, and Command-Line modes.**
- Normal to Insert: `i`, `a`, or `o`.
- Insert to Normal: `Esc`.
- Normal to Command-Line: `:` (for example, `:set number`).
- Command-Line to Normal: `Esc` cancels; `Enter` runs the command.

**Q4. Insert text before and after the cursor.**
- `i`: insert before the cursor; `a`: append after it.
- `I`: insert at the start of the line; `A`: append at the end.

**Q5. Add a line below or above the current line.**
- `o`: open a new line below and enter Insert mode.
- `O`: open a new line above and enter Insert mode.

**Q6. Save without exiting.**
```text
Press Esc, type :w, then press Enter.
```

**Q7. Save and exit.**
```text
:wq
```
Alternatively, use `:x` or `ZZ` (the latter in Normal mode).

**Q8. Exit without saving changes.**
```text
:q!
```
If no changes were made, `:q` is sufficient.

**Q9. Move to the beginning or end of a line.**
- `0`: first column; `^`: first non-blank character.
- `$`: end of the line.

**Q10. Move to the beginning or end of a file.**
- `gg`: first line.
- `G`: last line.

**Q11. Delete a character, a word, or a whole line.**
- `x`: character under the cursor.
- `dw`: word from the cursor.
- `dd`: whole line; `3dd`: three lines.

**Q12. Copy the current line and paste it below.**
```text
yy    copy (yank) the line
p     paste below (P pastes above)
```

**Q13. Copy multiple lines and paste them elsewhere.**
```text
3yy
Move the cursor to the destination.
p
```
Alternatively, use Visual Line mode: `V`, select lines with `j`, press `y`, move to the destination, then press `p`. The Ex command `:2,4y` yanks lines 2 through 4.

**Q14. Search forward for a word and move between matches.**
```text
/word
Enter
n     next match
N     previous match
```

**Q15. Search backward for a word.**
```text
?word
Enter
n     next match in the backward search direction
N     match in the opposite direction
```

**Q16. Replace the first occurrence of a word on the current line.**
```text
:s/old/new/
```

**Q17. Replace all occurrences throughout the file.**
```text
:%s/old/new/g
:%s/old/new/gc    ask for confirmation for each replacement
```

**Q18. Undo and redo the last change.**
- `u`: undo.
- `Ctrl+r`: redo.

**Q19. Display line numbers and jump to a line.**
```text
:set number       display line numbers (:set nonumber hides them)
Ctrl+g            display current line and file information
:10               jump to line 10 (or use 10G in Normal mode)
```

**Q20. Complete workflow: create, edit, search, replace, copy, paste, save, and exit.**
```text
vim lab5.txt
i
Enter a few lines that include the word linux.
Esc
yy, then p            copy and paste a line
/linux, then Enter    search
:%s/linux/Linux/g     replace
dd, then u, then Ctrl+r
:set number
:wq
```
Verify the saved result with `cat lab5.txt`.
