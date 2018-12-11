# writing functions in python

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
