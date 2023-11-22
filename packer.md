Allows automating a system installation from ISO and other sources. Supports many plugins, in particular for Proxmox and vCenter.

The first interesting that runs is `builder` that uses `boot_command` and other properties to create a machine.

Then `communicator` runs that tries to connect to the freshly created machine. It is possible to disable communicator completely by setting `communicator = "none"`.

# Glossary

* `machine image` in docs basically refers to the resulting system after the installation has completed.
* `builder` is a component that "builds" the "machine image". Examples: Qemu/VirtualBox/VMWare builder. It also describes where to "click" while traversing the installation GUI.
* `post-processor` is a component that take the result of a builder or another post-processor and creates a new artifact from it. Examples: compressing artifacts, uploading artifacts…
* `templates` define how to build a `machine image`. Langs supported: HCL2 and JSON; however JSON is officially deprecated and may lack newer features. HCL2 format uses `.pkr.hcl` file extension.
* `provisioner`: stuff that runs after installation is complete.

# HCL2 lang

Syntax:

```hcl
<BLOCK TYPE> "<BLOCK LABEL>" "<BLOCK LABEL>" {
  # Block body
  <IDENTIFIER> = <EXPRESSION> # Argument
  <IDENTIFIER> = <FUNC>(ARG1, ARG2)
}
```

Types supported: string, number, bool, list, map. There's also `null` identifier that is applicable to all types and has special behavior.

# Debugging

[There's a good section on that in official docs](https://developer.hashicorp.com/packer/docs/debugging).

## Builder

In HCL it's defined in `source`. Its syntax is `source "my_builder_type" "my_id" {…}`, and then it can be referenced from `build` block as `"source.my_builder_type.my_id"`.

Once machine started, a builder executes keypresses set in `boot_command` array. By using these keypresses you can navigate through installation GUI *(e.g. by using TAB)*. The important bit is that Packer does not actually do any parse of the GUI changes on screen. Instead you have to use a `<wait>` *(wait 1 second)*, or `<waitXX>` key combination which means "wait for XX time", where `XX` is seconds by default, but can be also stuff like `wait77ms`. Docs for `waitXX` and other special keypresses [can be found here](https://developer.hashicorp.com/packer/integrations/hashicorp/virtualbox/latest/components/builder/iso#boot-configuration).

So yeah, the workflow here is quite brittle *(having to rely on timeouts that may change for random reasons)*, but apparently there's no other way.

Someone's example:

```hcl
source "vsphere-iso" "bastionserver" {
  vcenter_server = "*****"
  username = *****
  password = var.vsphere_password
  insecure_connection = "true"
  host = "****"
  datastore = "******"
  vm_name = "bastionserver-template"
  CPUs = 2
  RAM = 2000
  guest_os_type = "ubuntu64Guest"

  network_adapters {
    network = "VM Network"
    network_card = "vmxnet3"
    mac_address = "00:50:56:00:00:01"
  }

  storage {
    disk_size = 20000
    disk_thin_provisioned = true
  }

  boot_order = "disk,cdrom"
  convert_to_template = true

  iso_paths = [
    "[qnap-vm-datastore-1] /iso-images/ubuntu-20.04.1-live-server-amd64.iso"
  ]

  cd_files = [
    "./cloud-init/meta-data",
    "./cloud-init/user-data"]
  cd_label = "cidata"

  boot_wait = "2s"

  boot_command = [
    "<esc><esc><esc>",
    "<enter><wait>",
    "/casper/vmlinuz ",
    "initrd=/casper/initrd ",
    "autoinstall ",
    "<enter>"
  ]

  # here "ssh" communicator is used, but we can also set e.g. "none"
  ssh_username = "******"
  ssh_password = var.ssh_password
  ssh_pty = true
  ssh_timeout = "20m"
  ssh_handshake_attempts = "20"
}

build { sources = ["source.vsphere-iso.bastionserver"] }
```

# TODO notes

* required_plugins block
* `boot_command` describes keypresses
