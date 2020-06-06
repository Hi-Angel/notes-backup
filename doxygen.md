# Debugging

It helps to insert a `-d Preprocessor` option before config filename to see what exactly Doxygen sees. Sometimes stuff can be expanded, removed, etc. depending on situations (like what language is used, whether filters are used, etc).

# Typical problems

* Missing docs
  * "common configuration": check is whether the file has a `@file` doc-string somewhere at top.
  * "with a plugin": the plugin likely filters out some important pieces of text, so use `-d Preprocessor`

# Writing a plugin

A plugin/filter can be an extension of existing documenting API or addition of documentation for a new language. It basically prints the file provided on the first argument. The text being printed is changed in a way that doxygen can understand.

[Here's a small example for documenting bash](https://github.com/Anvil/bash-doxygen). It basically works by making doxygen perceive `.sh` files as `.c` ones *(well, specifically you have to make the changes necessary to your doxygen config manually, per README)*, and then replacing `##` comments style to `//!` style.
