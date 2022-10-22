# Imports

Python supports "relative" and "absolute" imports, but there's a catch: "absolute imports" may actually be local files. I.e. they don't have to be system imports; and as a matter of fact, you should not use "relative" ones at all *(more below)*

* absolute imports:
  * `import foo`
  * `from foo import bar`
* relative imports
  * `from . foo import bar`
  * `from . foo import foo`
  * WILL NOT WORK: `import . foo`.

There are 2 problems with relative imports that I know of:

* they badly designed: for some reason, if you call main script file from a different directory, the local imports will no longer be found *(even though they supposed to be relative to the importer's dir, i.e. disregarding where main file is being located)*.
* `mypy` is gonna complain that the files hasn't opted-in to typing-system. Although that one is clearly a mypy problem, and there are workarounds, but together with the prev. point that increases questionability of the feature.

Instead you should import local files with absolute path. So, given this tree:

```
main.py
dir/myimport.py
dir/myfile.py
```

The `myfile.py` should import the myimport as `import dir.myimport`. That utilizes PEP420, and should work in modern python3. This absolute import is truly relative *(to `main.py`)*, and you may call it as `./main.py`, `/tmp/main.py`, etc, the import always just works.

Mypy has by default PEP420 disabled, you need to enable it with `namespace_packages = True` and `explicit_package_bases = True` options.

## Trivia

* paths to evaluate "absolute" imports against are stored in `sys.path` *(`import sys`)*, which can be modified at runtime. It's a list that has one special element `''`, an empty string, which means "use the location of the script as one of directories for absolute imports".
