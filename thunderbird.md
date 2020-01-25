# debugging extensions

More useful than randomly putting `alert("foo");` or `debug;` statements I found using the actual debugger from `Tools → Developer Tools → Developer Toolbox`.

Overall workflow is:

1. If you don't know yet what to debug, use the cursor icon to "pick an element" from Thunderbird interface *(well, if the addon in question works with the interface anyway)*. This would open the corresponding html, and if there're any JS hooks set, they will be highlighted with little boxes like `event`, `flex`, etc. Clicking one should give you the source: file:line.
2. switch to `Debugger` tab, and use <kbd>Ctrl</kbd>+<kbd>p</kbd> to "go to file". <kbd>Ctrl</kbd>+<kbd>f</kbd> to search within. Clicking a line number sets a breakpoint.

A catch: if the "tools" window resized too small, some elements may be "invisible".
