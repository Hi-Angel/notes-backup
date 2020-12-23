A CISCO like command shell, used for example to limit permissions to modify the system by creating a special user and making its shell klish. A fork of unmaintained `clish`.

# Misc

## Completion

In UX it's done upon pressing second space or tab. This invokes a `clish_shell_tinyrl_key_space`. It has many branches, but for me completion was eventually triggering `clish_shell_tinyrl_complete`.

`clish_shell_tinyrl_complete` searches for completion, displays them in the shell if found by means of `tinyrl_complete` *(a wrapper to `tinyrl_do_complete`)*, and returns a status on whether completions were found.

`tinyrl_do_complete` calls `this->attempted_completion_function()` to get completions, then, if any, displays them with `tinyrl_display_matches`.

`this->attempted_completion_function` callback by default is `clish_shell_tinyrl_completion`.

`clish_shell_tinyrl_completion` repeatedly calls `clish_shell_find_next_completion` till no matches left, and adds them all to `matches` variable.

`clish_shell_find_next_completion` calls  `clish_view_find_next_completion()` with local and global "views". For me completions were successfully searched in `this->global`. The completions are stored as `clish_command_t`, which stores most or all info about the command from POV of xml *(at the very least it has the command and its info text)*

Search for completions *(as opposed to printing)* is done somewhere down the `clish_view_find_next_completion()`.

Later, inside `find_next_completion`, `lub_bintree_findnext` gets repeatedly called with the second argument being the previous command name, and it returns the next command every time till `line` prefix in the cycle is gonna match the completion.

Matches are kept in

```
struct lub_argv_s {
	unsigned argc;
	lub_arg_t *argv;
};
```

where `lub_arg_t` is a typedef to:

```
struct lub_arg_s {
	char *arg;
	size_t offset;
	bool_t quoted;
};
```
