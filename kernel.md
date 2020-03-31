# Installing one module

Use something like

    ```
    make -C /lib/modules/5.4.28-050428-generic/build M=/home/user/Projects/mymodule_dir INSTALL_MOD_DIR=extra/dev_handlers CONFIG_MODULE_SIG_ALL= modules_install
    ```
