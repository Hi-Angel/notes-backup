# Logging in with ssh keys

1. `ssh-keygen -t rsa` to generate. It gonna ask for filename, if you're not using the default `id_rsa` along with the path, you gotta modify commands below, because they default to that path/name.
2. `cd ~/.ssh`
3. `ssh-copy-id user@IP` will install the key to the remote server.
