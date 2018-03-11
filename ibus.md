# misc

TIL: test IMs only under a VM. Testing on a working system gonna screw it up completely, even for applications that seemingly shouldn't be affected at all.

# enabling scroll-lock LED for specific layout

## algo

1. implement mandatory LED for second lang
2. add an option into GUI to map lang into scroll-lock LED

## misc

Configs under /usr/share/X11 don't belong to xkbcommon; and since the problem is specific to ibus anyway, it's better to resolve in there.
