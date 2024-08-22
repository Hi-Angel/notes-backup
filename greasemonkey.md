It's an extension for many browsers *(major ones and not)* that enables a user to write some JS for small automation or improvements on various websites. It reads js files that a user created and executes them based on the conditions coded in.

A script contains a header declaring the environment to run in, and a code immediately follows.

Simplest code might be directly operating on the DOM-tree with `document` variable. So e.g.: `document.head.foo.bar.value = 'mytext'`. IRL DOM-tree is often complicated, so a better way me be to use standard js searching functions. For example:

```js
// ==UserScript==
// @name     pre filling some text
// @version  1
// @match    https://example.com/search*
// ==/UserScript==
my_txtbox = document.getElementById('some_text_box');
my_txtbox.value = 'my pre-filled value';
```

`@match` is a URL for the script to run in, supports globbing with `*`.

# Debugging

In Firefox pressing <kbd>F12</kbd> will get you a console at the bottom-right, and then upon reloading the page where script is supposed to run you shall see any errors or `console.log()` output that may be coming from the script.

# Qutebrowser

Scripts have to be put under `~/.config/qutebrowser/greasemonkey/` directory. To make QB reload them execute `:greasemonkey-reload`.
