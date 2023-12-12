# vCenter Misc

Some things are not intuitive, so warrant a place in notes.

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
