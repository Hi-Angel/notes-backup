# TAGS

Emacs loads that into a buffer, and parses whole buffer every time searching for a tag *(barring caches in various places through the stack)*. This is done by `find-tag-in-order` func. So if you wanna something like "extract all methods in file1", bad news for ya.

# Misc

* Vanilla Emacs defines minor-modes through `easy-mmode` package. `smerge-mode` is one example, its `smerge-next`, `smerge-prev` are defined though `easy-mmode` mode.
* Pasting into incremental search <kbd>C</kbd>-<kbd>s</kbd> from clipboard: press <kbd>Enter</kbd>, which gonna make it non-incremental, then insert with a hotkey as you would in a buffer.
* to script regexp-replacing: use `-batch -l mycode.el` or `-batch --eval '(progn …)'`, and then either `replace-regexp` to replace all occurrences or `(when (re-search-forward "ORIG") (replace-match "NEW"))` to control amount/instances of replaces.

# Contributing

This is hard. They're using some custom changelog format which you definitely not gonna want to do by hand. You may use hotkey `C-x 4 a` to produce changelog entried from currently uncommited changes, and then copy it to a commit.

# Plugin in a native language (aka "dynamic module")

Steps:

1. Define a `plugin_is_GPL_compatible` symbol *(required for .so to get loaded)*
2. Define a function `foo` that does something
3. In `emacs_module_init` init plugin function:
   1. Register `foo` as a known function in Emacs. The odd thing here is that you can't define the function name just yet.
   2. Create a symbol that will become the function name later
   3. Tie the two together with `defalias`

Example:

```c
#include <emacs-module.h>

volatile int plugin_is_GPL_compatible; // necessary for plugin to get loaded

static int register_function(emacs_env *env, const emacs_value func, const char *function_name) {
    const emacs_value symbol = env->intern (env, function_name);
    if (!func || !symbol)
        return 1;
    emacs_value args[] = {symbol, func};
    env->funcall(env, env->intern(env, "defalias"), sizeof(args) / sizeof(emacs_value), args);
    return 0;
}

static emacs_value hello_world(emacs_env *env, ptrdiff_t nargs, emacs_value args[], void *data) {
    (void)nargs;
    (void)args;
    (void)data;

    const char hello_string[] = "Hello, World!";
    emacs_value message = env->make_string(env, hello_string, sizeof(hello_string));
    env->funcall(env, env->intern(env, "message"), 1, &message);

    return env->intern(env, "nil");
}

int emacs_module_init(struct emacs_runtime *ert) {
    emacs_env *env = ert->get_environment(ert);
    emacs_value func = env->make_function(env, 0, 0, hello_world,
                                          "Print Hello, World!", NULL);
    return register_function(env, func, "hello");
}
```

To make use of it later add the `.so` directory to `'load-path`, `load "module-filename"` *(the `.so` ending is optional)* and then you can call functions. Example that works combined with the code above:

```bash
$ gcc -shared -o hello.so hello.c -g3 -O0
$ emacs --batch --eval "(progn (add-to-list 'load-path \"$(pwd)/\") (load \"hello.so\") (hello))"
Loading module.so (module)...
Hello, World
```

## Misc

* `free`ing values returned from `funcall()` and whatnot is not needed. The values may be garbage-collected once the C code returns. Which in turn implies: may you want to keep some value between calls from LISP to your function, you have to do it some other way. I am not sure, but `make/free_global_ref()` might be an API to make `emacs_value` stick. And of course you can just save a `emacs_value` content in a `malloc`ed C structure or call `setq` as you'd usually do in LISP code.
* calling ELisp code from native-lang plugin is possible with `eval` function — but note that you can't call `(eval "(foo)")` because `eval`ing a string would yield nothing useful even though won't fail either. Something along the lines of `env->intern(env, "(foo)")` might be needed.

# SMIE

## TODO questions:

* `smie-bnf->prec2`:
  * how do you define "any text"?
  * how do you define that after a backslash text may continue on new line?
* how to integrate with imenu?

## what to mention in the article

* the `offset` that `smie-rules-function` should return isn't always an offset, depending on `kind` param it may be a `t/nil`.
* purpose of a parser needs to be considered: in this case it's indentation, so indentation function needs to be defined that decides what to do depending on the parser output. **keywords** are terminals in the grammar.
* lexer may use parser to ask for context.
* syntax table may be used to show matching pairs of parentheses
* `smie-forward-token` callback will not be called at `(point)` you asked for indentation, but instead in a completely arbitrary position.

# Tasks

## Make projectile file listing in `$HOME` less dramatic

* it shows a buffer with `fatal: not a git repository (or any of the parent directories): .git`. It's because projectile-files-via-ext-command prints err to buffer, Idk yet how to work around it.

## Unrecognized entry in undo list undo-tree-canary

Some info I gathered:

* the `undo-tree.el` package logs actions in `buffer-undo-tree` var, and then somehow they get transferred to `buffer-undo-list`.
* the error is about `buffer-undo-list` being set to just `(nil undo-tree-canary)`. The only place, I think, that could result in that is the function `undo-list-transfer-to-tree ()`, which has a line `(setq buffer-undo-list '(nil undo-tree-canary))`.
* `undo-tree.el` has only 2 functions that deal with `undo-tree-canary` thing.

### Scratches for a post

Looking at how Alan recently updated a bunch of packages on SO and made a post on reddit, I figured I could do likewise, and get some free points from making this work and then posting an answer on stackexchange.

----

I was recently amazed to find unit-testing framework in Emacs and how nicely it being used for regress-testing `rust-mode`. I figured I could do the same with `undo-tree`, and thus try to find the combination that leads to the infamous problem *(making the project more robust on the way)*.

# font-lock

To check font-lock-face set on the symbol/word under cursor use `describe-text-properties`.

font-lock is managed by `font-lock-keywords` list, which can store regexps as well as actual functions that do scan and stuff. Color-identifies is using that to scan a buffer.

## symbol-overlay mode highlights wrong text

For rust-mode turned out, the reason is that `(racer-eldoc)` changes the `(point)`, and deep the stack calls a `(accept-process-output)` to wait for docs at point, and then while on it, an idle timer gets triggered, and calls `overlay-mode` hook. The bug is in Emacs: the timer calls it with the same environment as the rust-mode function. Like, if you trigger a debugger, you'll literally see the stack with `accept-process-output` and then the `overlay-mode-foo` on top of it. As result, `overlay-mode` gets a different `(point)` than the actual.

Not motivated enough to make a testcase for a bugreport though. In part because I'm sure nobody gonna fix it: Emacs project is opposed to multithreading because their design is completely broken, and this stacktrace thingy in async call is a part of it. And I doubt GNU project can handle it.

Note that lsp-mode also has its highlighting function, which looks identical to symbol-overlay except its shown even if `symbol-overlay` is disabled in the buffer, and it occasionally has unrelated problem with a similar pattern, see below.

## lsp-mode highlights wrong text

I'm unclear yet what could be causing it, but it may be a bug in lsp-server *(`clangd`)* or lsp-mode. Doing `(setq lsp-enable-symbol-highlighting nil)` *(being checked in `lsp--document-highlight` function)* works around it. When inside `lsp--document-highlight-callback` function I print the `highlights` argument, I see it has the wrong line, and even the wrong boundary *(I guess when setting highlight, lsp-mode crops it down a bit)*.

If I ever going to debug it further, gotta check where the `highlights` argument is coming from. FTR: the function is called from `lsp--document-highlight` *(the `lsp-request-async` call in it)*.

# Done:

## diff-mode: fixing up wrong line count when working with patchset

Problem: upon updating such header: `@@ -25,7 +25,6 @@` it sets the 2nd number *(i.e. 7 here)* wrong. It's one bigger than should be, because patchset ends with `--` thing, which Emacs considers as `removed`.

A patch sent https://debbugs.gnu.org/cgi/bugreport.cgi?bug=37395

## Emacs resize works wrong in KWin

It's due to `program specified resize increment` in `WM_SIZE_HINTS`. Works in other WMs simply because they ignore it *(which is bad though)*.

Can be worked around by setting `frame-resize-pixelwise` to `t` before frame is created.

Being set with `size_hints.width_inc` and `size_hints.height_inc` in Emacs code. First occurrence is found in `xterm.c`, commit `Initial revision` in 1991 year, function `x_wm_set_size_hint`. First occurrence in GTK related file is at `gtkutil.c`, commit `GTK files gtkutil.c and .h` in 2003. Both commits lack any description, and no comments on the resize matter provided.

I sent a patch to fix it, but turned out someone indeed needs it https://debbugs.gnu.org/cgi/bugreport.cgi?bug=36250
