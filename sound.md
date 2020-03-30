# misc
## finding ALSA microphone sample rate

First execute `cat /proc/asound/pcm` to find PCM devices.

`grep rate /proc/asound/card0/pcmXY/sub0/hw_params` directory, where `X` is the device number from the `pcm` file and `Y` is `c` *(stands for `capture`; likewise `p` for `playback`)*.
