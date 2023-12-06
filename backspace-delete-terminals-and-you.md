# Backspace Delete Terminals and You!

I forget and have to relearn all the messy details about how backspace/delete/ctrl-H/ctrl-?/ctrl-backspace work in linux terminals every few years. So this time I'm writing it all down to make the process faster for next time.

## Background

For the point of this document, when we talk about terminals we're referring to things that emulate the [DEC vt220](https://vt100.net/docs/vt220-rm/contents.html). The linux virtual console targets the vt220. Xterms can target a number of different terminals but the vt220 is base (xterm actually targets the vt420 by default).

## The Problem

In a graphical window system like X11, each key press includes details about both the original key that was pressed and any modifier keys that were also active. In a terminal we just get plain keycodes. If you were starting from scratch you can imagine a coding system to still make this work reasonably. But the vt220 maintained backwards compatibility with older terminals (vt100) which had fewer modifier keys (no alt/meta, no f5-20). So we ended up with something that is fairly awkward.

### Why are things the way they are?

Because punchcards?

In the 1960s, what did you do when you made a mistake typing a character on a keypunch machine? First you pressed the key labeled 'Backspace' (ascii 0x08 or `\b`) to move back to the previous position on the punchcard. Then you pressed the 'Delete' (ascii 0x7F) key to delete the character that was punched incorrectly. How do you delete a physical whole in a punchcard? You punch out all the bits so that the reader knows to ignore and skip over that character. So that is what 'Delete' did. And thus ascii `DEL` is 0x7F, or all holes punched in a 7 bit encoding[~mcrob].

Once you get digital terminals, there's no reason to have two buttons do that action anymore. So if you are making a digital terminal you pick either Backspace or Delete to perform both of those actions (move backward 1 character and delete it). Then you have another ascii code that could now do something different, like delete the character currently under the cursor. This is how we ended up with the keys on our keyboards sending codes that are swapped with what you'd expect them to be. In vt100 emulation mode you will usually see this:

| Keyboard key |  Hex | Ctrl Sequence | ASCII Char           |
|--------------|------|---------------|----------------------|
| Backspace    | 0x7F | Ctrl-?        | DEL                  |
| Del          | 0x08 | Ctrl-H        | BS  '\b' (backspace) |

That's a bit confusing but is simple enough. But why are there two ways to send these characters, either with the dedicated key Backspace/Del or by pressing Ctrl-?/Ctrl-H ? That is a legacy of how control keys were implemented in ascii. To send a control code say 'End of Transmission', you would push Ctrl + the character that is forward 64 characters in the ascii table. So EOT(0x04) + 64(0x40) == 'D' (0x44) i.e ctrl-D sends EOT.

This is nice and simple, but as soon as we want to make more interesting applications it starts to get in the way. Interactive applications that use the terminal in raw mode might want to map ctrl-h to a help function, but that won't work if we also expect the delete key to delete characters. There's also the problem of wanting to use ctrl-Backspace to do something interesting. If the terminal just applies the default mapping of add 0x40 to any key when ctrl is also held, that would just send a 'H' which we probably don't want to remap to something besides `insert character H`.

The solution to this problem is to have the terminal advanced control sequences that are outside of the simple ascii table. The VT100 already did this with its F1-F4 keys and its arrow keys. The VT220 expanded this with its F5-F12 keys. The terminal sends a multi-byte control sequence to support additional keys outside the ascii range. So the up arrow key sends `CSI A`, down `CSI B` (`CSI` is `ESC` followed by `[`, so the full sequence of bytes is `0x1B 0x5B 0x41` or `^[A`). Delete then becomes `CSI 3 ~` or `^[3~`. Thus on a normal linux virtual console or xterm you'll usually see the following (and if you don't this might be why an application isn't behaving as expected):

| Keyboard key   |                 Hex | Ctrl Sequence | ASCII Char |
|----------------|---------------------|---------------|------------|
| Backspace      |                0x7F | Ctrl-?        | DEL        |
| Ctrl-Backspace |                0x08 | Ctrl-H        | BS '\b'    |
| Del            | 0x1B 0x5B 0x33 0x7E | ^[3~          |            |


## Tips for understanding how your terminal is behaving

- XTerm will show you the key code sequence it will send if you press `ctrl-v` followed by the key in question.
- You can run `infocmp` to see the full configuration for your terminal's key sequences

# Sources

- [Consistent BackSpace and Delete Configuration](https://www.cs.colostate.edu/~mcrob/toolbox/unix/keyboard.html) (originally http://www.ibb.net/~anne/keyboard.html)
This is a very useful resource to understand how things work and why.

- [Xterm Control Sequences](https://www.xfree86.org/current/ctlseqs.html)
Concise table of key mappings

- [VT220 Programmer Reference Manual](https://vt100.net/docs/vt220-rm/)

- https://invisible-island.net/xterm/ctlseqs/ctlseqs-contents.html
