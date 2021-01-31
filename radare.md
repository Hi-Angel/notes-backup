# Notes

* Any modification requires opening a file with `-w` option. At least in older versions modifications were pretending to be working *(i.e. no error, warning)*, however upon printing the content you'd find it's unchanged.

# Examples

* Loading a binary file `r2 -nn file`.
* Loading a header *(inside the r2)* "to header.h"
* Printing struct at address: `tp mystruct = <address>`
* Print hex at offset 4096 `px@4096`
* Search for hex `/x b00b55`
* Arithmetic operations, with result being printed as char, hex, decimal, octal, binary, etc `"? 2 + 2"`

## Overwriting an LAA field in PE executable

It's best to use with scripting

```
e cfg.newshell=true      # allows nested $(…) commands
s/ PE\0\0                # search PE file signature
s +4                     # skip the signature
echo "Original content:"
pf.pe_image_file_header.characteristics
echo "Patching the file…"
s+ 0x12                  # go to the characteristics field
wv2 $(?v $(pv2) \| 0x20) # 0x20 is the LAA bit, binary-OR it in the address
s-
echo "The new content:"
pf.pe_image_file_header.characteristics
```

# Scripting

`-i` field is used to pass a file with commands. Make sure you pass the filename to open last *(I had a mistake of putting a `-i file` the last instead)*. For example:

```
r2 -qi /tmp/script.r2 -nnw my_executable_file
```

# References

* Inspecting ZFS on-disk format with Radare https://habr.com/ru/post/348354/
