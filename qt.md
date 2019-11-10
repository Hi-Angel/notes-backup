# slots'n'signals

Declared via like `slots:`, `signals:`. It's a macros that's supposedly parsed by Qt MOC compiler.

There's a function `connect()` to connect them. It's also possible that predefined slots does not require `connect()`ing *(needs confirmation)*.

# Debugging

## GammaRay

An introspection tool for Qt GUI. Can be attached to processes.

Tabs on the left:

* `Objects` has widgets tree, and their properties.
* `Widgets` allows to select widgets and see their preview. Also: the preview has toolbox, which has `picker` tool. Unfortunately you can't click with this on the process itself, but instead you can "pick" a widget in the preview.
