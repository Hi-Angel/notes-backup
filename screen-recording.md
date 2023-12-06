As I forget every time what works and what not, here.

# OBS

A cross-platform tool for that kind of tasks. While it has lots of options, for basic stuff it's simple.

Basic steps:

1. On the panel at the bottom, add a source, which can be a "window", "screen", "camera", etc.
2. Check audio on the next rectangle. You cam mute it if you won't want any sound by clicking the audio icon.
3. Click `Settings` on the far right rectangle of the panel, and configure "Output" for location/format of the output file.
4. Click `Start Recording` *(you can also assign shortcuts to it in `Settings → Hotkeys`. I usually set start/stop pair to <kbd>F1</kbd>)*

## Misc

* To crop a window hold <kbd>Alt</kbd> and resize the image of the window over the canvas.
* To reset resizes on a "source" click it and chose `Transform → Edit Transform`
* `To fit canvas to a source`, right-click the source and chose "Resize output (source size)"

## Gotchas

* There are pipewire-based options in the "sources", however for me nothing appears when I chose them, just a black screen. But `X*`-based sources instead works fine. Idk why is that, maybe some bugs in implementation on X11. It doesn't matter much though, non-PW based sources are working.
* On NVidia `obs` may crash with `CUDA_ERROR_UNKNOWN: unknown error`. Have to `sudo rmmod nvidia_uvm && sudo modprobe nvidia_uvm` to make it work 🤷‍♂️

# simplescreenrecorder

Supports quite a few formats, has quite a few options. Works for me. Has a tray icon, so you can start/pause/continue recording by clicking it.

I haven't used it for long recording sessions, but for short ones works well.

## Gotchas

It has good docs on various errors. Ones I stumbled upon:

* make sure to use a codec that is supported by video container *(it says in a codec description when it's not supported)*. I think codecs that are not supported by a container should be grayed out, Idk why they aren't.
