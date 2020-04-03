# Installing one module

Use something like

    ```
    make -C /lib/modules/5.4.28-050428-generic/build M=/home/user/Projects/mymodule_dir INSTALL_MOD_DIR=extra/dev_handlers CONFIG_MODULE_SIG_ALL= modules_install
    ```

# Errors

* `FATAL: parse error in symbol dump file`: in my experience of building an external module, this means you've got a `Module.symvers` file cached from a different kernel version. If you're building an external module, just run `find -name Module.symvers -delete` over your source code, it should get regenerated.
