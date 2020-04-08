# Installing one module

Use something like

    ```
    make -C /lib/modules/5.4.28-050428-generic/build M=/home/user/Projects/mymodule_dir INSTALL_MOD_DIR=extra/dev_handlers CONFIG_MODULE_SIG_ALL= modules_install
    ```

# Errors

* `FATAL: parse error in symbol dump file`: in my experience of building an external module, this means you've got a `Module.symvers` file cached from a different kernel version. If you're building an external module, just run `find -name Module.symvers -delete` over your source code, it should get regenerated.

# Potential improvements, per 4.19 kernel

* `__tcp_transmit_skb` at the end has following code:

    ```
        /* Cleanup our debris for IP stacks */
        memset(skb->cb, 0, max(sizeof(struct inet_skb_parm),
                       sizeof(struct inet6_skb_parm)));
    ```

    I am not sure this is really needed, from the cursory look I am not seeing why would this field have debris; and if function don't need that info anymore, why wouldn't it just ignore it?
* in `tcp_v4_rcv` there's branching upon `sk->sk_state`. The values are enum, but the `sk_state` is a char. See if we can do anything about it.
* `inet_ehashfn` takes some time, the stack ends in there but I'm guessing it's the hashing part that takes time. Can we optimize it somehow?
* in various functions, a bunch of time may be spent to release `skb`: (`skb_release_data`, `kfree`; and the opposite `sk_stream_alloc_skb`). Makes me wonder if we could reserve some hundreds of bytes to make it go easier. UPD: actually, I gotta measure how much time it really takes. It is especially interesting for newer kernel as there was a lot of reworks around memory management.
