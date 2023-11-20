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
