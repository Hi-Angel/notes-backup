# vCenter

Some things are not intuitive, so warrant a place in notes.

## Installation

1. Install at least one ESXi, because vCenter is installed over it.
2. Run a GTK app from the vCenter ISO file, and as part of setting up you write the ESXi IP address and password.

    Among steps the only one worth mentioning is setting up "SSO". It will be used as part of the username to log into vCenter. So if your domain was chosen to be `foo.local`, then the username will be `administrator@foo.local`.
3. Prev. step has only created a VM with vCenter on the ESXi, but no ESXi is managed just yet. Now you open the vCenter address *(if you use DHCP, you can look it up by opening the vCenter VM console in the ESXi, address should appear there)*. Then in vCenter:
   1. Click its IP address and chose `New Datacenter…`. Follow steps to create it.
   2. For every ESXi you want to add: Click the datacenter → `Add Host…`. As part of steps point out the ESXi address.

## Adding a new network

1. Click a node out of the cluster *(e.g. 1.1.1.1)*
2. Chose tab `Configure`
3. Go to `Virtual Switches`
4. There you have a list of switches, click `Add Networking` on one of them, then go through menus. It's recommended to assign a `VLAN ID`. Apparently all networks with same ID *(or no ID for that matter)* share the same network space.

## Renaming/removing a network

1. Go to the same `Virtual Switches` tab as in prev. paragraph
2. Unroll the switch that contains the network
3. Click triple-dot on the network and chose `Edit Settings` or `Remove`.

## Removing a directory in a datastore

In certain circumstances after a VM was removed there might be a directory with files left in a datatsore. Then, if you chose it the "Delete" button will be grayed out. To remove it gotta chose the parent instead, and then the dir can by selected in the list in the right window.

## Sharing a disk between VMs (aka active/active, active/passive)

The first VM would be created as usual. Then for other VMs, in settings/`Add new device` instead of "Hard disk" gotta chose a "Existing Hard disk", and chose a datastore path to an existing disk. Also set *(both in the original and "existing" disks; ESXi might not allow changing them for "original" though so you'd have to re-create)*:

* `Type` → `Thick Provision Eager Zeroed`
  * In case you're creating this via API, it's achieved by setting `EagerlyScrub = True` and `ThinProvisioned = False`
* `Sharing` → `Multi-writer`
* `Disk Mode` → `Independent - Persistent`

In absence of these 3 options various problems gonna arise, such as unable to power on the VM with `Failed to lock the file`. These options have been used for years in a company, so once I realized it doesn't work without them, I didn't dig further.

## Boot logs

Located on a ESXi that contains the VM at `/vmfs/volumes/ds_name/*.log`, where `ds_name` is the datastore name that contains the VM.

# API

There's `pyvmomi`, `govmomi`, etc. An example of a `pyvmomi` that lists VMs and their resource pools:

```python
from pyVim import connect
from pyVmomi import vim
import ssl

if __name__ == '__main__':
    ssl_context = ssl.SSLContext(ssl.PROTOCOL_TLS)
    ssl_context.verify_mode = ssl.CERT_NONE
    si = connect.SmartConnect(host='1.1.1.1',
                              user='foo@bar.buzz',
                              pwd='*********',
                              sslContext=ssl_context)
    print('Authenticted into vCenter successfully')
    data_center = si.content.rootFolder.childEntity[0]
    vms = data_center.vmFolder.childEntity
    for vm in vms:
        if isinstance(vm, vim.VirtualMachine):
            print(f'{vm.name} {vm.resourcePool}')
```

# Misc

* Networking
    * `virtual switch` is a network device that refers to a physical one.
    * `distributed switch` same as `virtual switch`, except it's only creatable from vCenter and it basically allows to deduplicate some work by choosing both ESXi that gonna be on the network at once.
    * `port group` is a thing to be created over a `switch` and basically is a virtual network where IPs are unique.
        Basically, you can have multiple `group`s over a `switch` over a physical network, so basically you end up multiple VMs who potentially use the same IP talking to each other on the same physical network — and you get no IP conflicts because vSphere transparently handles low-level IP reassignment to avoid conflicts.

# Vsphere cons

* vague and useless errors. One example "insufficient resources" upon starting up a VM, which has only one reason documented: lack of RAM. So you go check your RAM, you find that everything's alright, you conclude it's a bug in vsphere/ESXi. Well, it turns out there may be other reasons as well, such as passthrough of a SR-IOV NIC while a limit is reached, but vsphere won't tell you anything besides the two words "insufficient resources", both in the logs and via API.

# ESXi cons

* terminal:
  * lacks many standard utilities like `lsblk`
  * what's available is implemented via `busybox` *(expect jumping through the hoops whenever you have to use terminal)*
  * history gets lost on reboot

# ESXi

## Misc

* `Changing management IP address`: go do `Networking → VMKernel NICs` and edit the address.
* Maintenance mode: getting `esxcli system maintenanceMode get`, changing `esxcli system maintenanceMode set --enable false`. It also optionally supports `--server=` arg.

## Passing through devices

As of 6.7 it's only possible directly via ESXi host interface, but not via vCenter that manages these hosts. So if you are in vCenter, you gotta leave it and open the URL of the ESXi you're interested in.

Open `Host → Manage → Hardware`, there should be a hardware devices list. You can enable either `SR-IOV` or `Passthrough` for those that don't have a "Not available" entry.

Afterwards, open a VM, "edit" it and chose a `Add other device → PCI device`. If the device didn't appear in the list, that might imply a ESXi reboot is needed. In such cases you should the message about it earlier, right after enabling passthrough due to ESXi changing "Disabled" to something like "Enabled/Reboot needed".

Some *(or maybe all, Idk)* hardware *(e.g. JBODs)* requires the VM that devices were passed to, to reserve all RAM. ESXi shall tell you that as well. ESXi will refuse to boot VM unless you do that.

### Passing through disks

Special case *(note, this applies to ESXi not to vSphere)*: depending on some circumstances, there may be two variants:

* "Edit" the VM, click `Add hard disk → New raw disk`, there should be a list of hardware disks present.
* This requires tinkering:
  1. Follow [this steps](https://kb.vmware.com/s/article/1017530) to create a disk binding that takes shape of a locally stored file.
  2. "Edit" the VM, click `Add hard disk → Existing hard disk`, chose the file created in the prev. step. The pass-through should work, so e.g. `smartctl` returns the correct data.
