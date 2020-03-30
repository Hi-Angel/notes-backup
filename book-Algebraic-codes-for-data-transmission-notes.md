# Misc

Data flow, functional: datatstream -> encoder -> codestream -> decoder


Data flow, hardware: source -> source encoder -> channel encoder -> modulator -> (channel) -> demodulator -> …

`Modulator` converts a codestream into channel signals. The output from `demodulator` is typically not the same as what was inputted because of noise, distortion, interference…

`Channel decoder` uses redundancy to try restoring the codestream it got from demodulator.

`codeword` is a data + redundancy.
