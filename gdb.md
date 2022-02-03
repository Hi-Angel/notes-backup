# interactive python

`py` and `pi` allow for python inside gdb session.

Then, to call a gdb command from python use `gdb.execute('cmd', to_string=True)`

`gdb.inferiors()` returns "currently running applications", typically just one.

More docs: https://sourceware.org/gdb/onlinedocs/gdb/Basic-Python.html#Basic-Python

# scripting in python

Python allows creating both convenience_variables-like functions, and just commands. It requires creating a class with different properties, where `__init__` shows the name by which it gonna be called.

`invoke` usually have parameters passed, but "how" depends. For "commands" it accepts a single parameter that have to be broken with `gdb.string_to_argv`. For "convenience functions" `invoke` directly accepts arguments one after another.

For conv. variables, example:

    class Greet (gdb.Function):
    """Return string to greet someone.
    Takes a name as argument.
    Call it like `p $greet("hey")`"""

    def __init__ (self):
        super (Greet, self).__init__ ("greet")

    def invoke (self, name):
        return "Hello, %s!" % name.string ()

    Greet () # needed to get it registered

For commands, example:

    class GrepCmd (gdb.Command):
        """Execute command, but only show lines matching the pattern
        Usage: grep_cmd <cmd> <pattern> """

        def __init__ (_):
            super ().__init__ ("grep_cmd", gdb.COMMAND_STATUS)

        def invoke (_, args_raw, __):
            args = gdb.string_to_argv(args_raw)
            if len(args) != 2:
                print("Too many parameters. Usage: grep_cmd <cmd> <pattern>")
            else:
                for line in gdb.execute(args[0], to_string=True).splitlines():
                    if args[1] in line:
                        print(line)

    GrepCmd() # needed to get it registered

Here, `gdb.COMMAND_STATUS` and other types are only influencing documentation.

# Using gdbserver

* Starting up gdbserver: `gdbserver :PORT path_to_app app_args`
* Connecting to the gdbserver:
  1. `gdb`
  2. In gdb cmd line: `target extended-remote :PORT`

# kgdb

Ultimately, you need to enable module that allows you to connect from gdb with `target remote` command. With the kernel there's a driver for debugging over a serial port. There's also off-tree driver `gdboe` *(gdb over ethernet)*, which I used. Enabling it is simple: `insmod kgdboe.ko device_name=enp0s25 udp_port=3333`. It even prints in dmesg how to connect from gdb to it. However when I connected gdb, the system hanged. Idk why.

There's also multiple configs that may need to be enabled to be able to debug over gdb.

# Pretty printers

Given a `struct MyStruct{const char* a; int b};:`, its pretty printer is:

```
class MyStructPrinter:
    def __init__(self, val):
        self.val = val

    # it's the actual printing
    def to_string(self):
        return f'MyStruct\n\ta = {self.val["a"]}\n\tb = {self.val["b"]}'

def register_printers(val):
    return MyStructPrinter(val) if str(val.type)=='MyStruct'\
        else None

gdb.pretty_printers.append(register_printers)
```

FTR: the `val['foo']` is a Python object, it has various functions and fields.

---------

If you want to recurse into struct fields, there's automation too. Function `children(self)` should return a list of pairs `{"field name (arbitrary string)", field_address}`, and then gdb will descend into these and apply printers.

It's a bit complicated though. You need to manually check if gdb is able to descend into the field. Specifically to check if the field is a null pointer or not.

Example:

test.c:

```C
#include <stdio.h>

struct MyStruct;

struct MyStruct {
    const char* a;
    int b;
    struct MyStruct *m;
};

int main() {
    struct MyStruct p = {"world", 2, 0};
    struct MyStruct m = {"hey", 1, &p};
    puts(m.a);
}
```

and the pretty printer:

```python
class MyStructPrinter:
    def __init__(self, val):
        self.val = val

    # it's the actual printing
    def to_string(self):
        return f'MyStruct\n\ta = {self.val["a"]}\n\tb = {self.val["b"]}'

    # this function returns a list of pairs, where first element if an arbitrary
    # string to be printed as the field name, and the second one is
    def children(self):
        return [('m', self.val['m'].dereference())] \
            if self.val['m'] != 0 \
            else []

def register_printers(val):
    return MyStructPrinter(val) if str(val.type).endswith('MyStruct')\
        else None

# gdb.pretty_printers.append(register_printers)
```

# Misc

* `build-id` is an id that can be viewed in `file ./executable` output. It is a hash of arbitrary sections of the executable. When debug-info file is separate from the executable, BuildID for both of them is supposed to match.
    * expected filename of the debuginfo file can be seen with `readelf -x.gnu_debuglink ./executable`. There're various dirs where it's expected to be, one of them is the location of executable.
    * worth noting that you can't point out to a debug info file inside gdb. Or rather, you can, but it will result in bad information such as question marks on the stack. For debug information to work out gdb must find it itself, so you gotta jump around with file naming etc.
