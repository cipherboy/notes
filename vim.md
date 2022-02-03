# Vim

Switch to the next file:

    :next
    :n

Go to beginning of line:

    ^ -- relative (to whitespace)
    0/<HOME> -- absolute

Move based on words:

    b/B -- back a word
    w/W -- forward a word

To deal with panes (separate buffers):

    CTRL+W CTRL+X -- swap panes
    CTRL+W h/j/k/l or left/down/up/right -- move cursor between panes
    :b# -- go back to last buffer (e.g., directory listing -> file -> back to dir listing)

To use visual mode:

    ESC + v -- begin visual mode
    h/j/k/l or left/down/up/right -- move cursor to complete selection
    d -- yanks
    :s/// -- substitution
