# Misc

* PCM is a "pulse-code modulation". [This claims](https://alsa.opensrc.org/Pcm-device) it's like an abstract sound card.

## Finding ALSA microphone sample rate

1. Execute `cat /proc/asound/pcm` to find PCM devices.
2. Start the application that uses the microphone.
3. `grep rate /proc/asound/card0/pcmXY/sub0/hw_params` directory, where `X` is the device number from the `pcm` file and `Y` is `c` *(stands for `capture`; likewise `p` for `playback`)*.

Note: the `hw_params` file will have just `closed` text if nothing is using it.
