# Misc

If bluetooth device looks detected but GUI says "no bluetooth devices available", make sure you have `bluez` and `bluez-utils` installed.

# Headphones

For me the audio just worked. Microphone didn't though. It turns out, headset supports multiple profiles, one of them lacks microphone in exchange for better audio quality. It's called a2dp, whereas the one with microphone is HSP/HFP. On pulseaudio you can switch profile with `pacmd` command, see https://wiki.archlinux.org/index.php/Bluetooth_headset#Switch_between_HSP/HFP_and_A2DP_setting
