Abstractly speaking, Ansible is an utility for management of remote *(and local ones too, though that sounds boring)* PC instances. I personally have used it for managing VMs under vCenter, but the Ansible objective is far more broad. It's useful to see what it does and how.

First you have a task. It may be commands to run, or something special such as connecting to vCenter and powering off a VM. Tasks seems to always be described through means of a plugin/module, which is written in python.

Ansible just executes the task.

You may not trust the plugin, in this case you may set up a host with ssh connection on which you'd like to execute the task. Ansible requires setting the variable `hosts:`, which describes the host on which you want this to be executed *(usually it's localhost though)*. How it works: Ansible connects to the host with ssh *(unless it's localhost)*, and copies the plugin code there along with a small wrapper. Then executes it.

Tasks may be described as a "playbook" or "ad-hoc commands" *(read as "a playbook written on the command line to `ansible` executable")*.

# Misc

* *Plugins* and *Modules* are different things.
* Authentication involves writing a config which is parsed by a plugin. E.g. managing vSphere/vCenter involves writing a `myconfig.yaml` file with host addresses to connect, etc; the format is usually the same between all plugins, but it seems it may differ. Example of such plugin:
  ```
  - hosts: localhost
    connection: local
    tasks:
      - vmware_guest_powerstate:
          hostname: "vcenter-host"
          username: "my-user"
          password: "********"
          validate_certs: no
          name: "vm name"
          state: powered-on
        register: deploy
  ```
* listing available VMs: `ansible-inventory --list -i myconfig.yaml`

# Plugins

Installation involves putting into a special directory. An example `~/.ansible/plugins/inventory/vmware_vm_inventory.py`. To get the actual directories list where a plugin being looked up use `ansible-doc -t <plugin_type> foo`, for example `ansible-doc -t inventory foo`. The idea is that the command gonna fail because sure you don't have a plugin `foo`, and then it gonna print the directories.

# Ad-hoc commands

Supposedly, you can run `ansible` from command line by writing down stuff you'd otherwise put in a playbook. Though for me the examples I used from the official docs were throwing errors. I deemed it wasn't worth putting further efforts.

# Playbook

A configuration file used to execute commands/actions against VMs. Use with `ansible-playbook` command.

## Running a single task

It's possible by providing a `tags:` field on tasks, and then exploiting `--tags=` option of `ansible-playbook`, but it doesn't look like a mature feature because if the tag did not match anything, there will be no errors. I had to work this around by splitting a playbook to multiple smaller ones, so I could run everything in it explicitly, without any tags.
