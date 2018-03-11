1. highlight a focused field
1.1 (optional) keep focus on high-level, and save focued field (may not work because there might be multiple "high-level" elements with scrolls. Keep out only of "editable"s?).
2. remove blinking cursor
3. add a variale to disable 1 and 2 for funkies.

Alternatively:

1. Do not care of highlight, it's done by webengine
2.↓

onFocusedEnter state elem =
    if elem.editable then return state{ mode = "insert" }
    else return state
