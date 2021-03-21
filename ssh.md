# Logging in with ssh keys

1. `ssh-keygen -t rsa` to generate. It gonna ask for filename, if you're not using the default `id_rsa` along with the path, you gotta modify commands below, because they default to that path/name.
2. `ssh-copy-id user@IP` will install the key to the remote server.

# Executing a particular command

There is a way with `ssh … mycommand` but it doesn't work for logging into specific shell.

A work around is possible if you logging with a key. In this case edit `~/.ssh/authorized_keys` and prepend in the line with your key a text `command="mycommand"`. This way it works. Gotchas: 1. the double quotes are mandatory, 2. `ssh … mycommand` stops working. 3. `sshfs` stops working too.

# Using git with key in a particular directory

1. Generate key somewhere: `ssh-keygen -t rsa -b 4096 -C "username@example.com" -f /tmp/foo/id_rsa`
2. Point to the key through git env. variable: `GIT_SSH_COMMAND='ssh -i /tmp/foo/id_rsa' git push origin HEAD`
