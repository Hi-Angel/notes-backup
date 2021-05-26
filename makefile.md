# Misc

* making a target depend on a file requires some jumping around. An example of a target `foo` *(lowercase)* looks basically like this:
    ```
    FOO = path/to/file
    foo: $(FOO)
    $(FOO):
        echo "a code to actually generate the file" > $(FOO)
    ```
