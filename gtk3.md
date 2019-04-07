# Misc

## Undo

### Misc

I've been linked to this page https://wiki.gnome.org/Projects/GTK/Undo Haven't found anything interesting though. A lot of functional being discussed I don't see as something needed right now, and can be added later by someone if they see it useful.

* Undo/redo GTK enablement:
    * GTK3: Opt-out by default, but opt-in if Ctrl+z already bound.
    * GTK4: Opt-out by default.

```
    -- implement for widgets and stuff
    class Undoable widget where
    undoStack :: [undoAtom]
    undoAtom :: undoAtom -> widget -> widget
    redoAtom :: undoAtom -> widget -> widget

```

### todo

* create undo/redo stack on GtkEntry init, and save it with it.
* wire up on-character-typed action for GtkEntry (don't save it to the undo stack every time though — good for CPU cache)
* wire up Ctrl+z action for GtkEntry
* wire up Ctrl+Shift+z action for GtkEntry
* profit!
