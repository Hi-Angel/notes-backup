# Notes

* Any modification requires opening a file with `-w` option. At least in older versions modifications were pretending to be working *(i.e. no error, warning)*, however upon printing the content you'd find it's unchanged.

# Examples

* Loading a binary file `r2 -nn file`.
* Loading a header *(inside the r2)* "to header.h"
* Printing struct at address: `tp mystruct = <address>`
* Print hex at offset 4096 `px@4096`
* Search for hex `/x b00b55`
* Arithmetic operations, with result being printed as char, hex, decimal, octal, binary, etc `"? 2 + 2"`

# References

* Inspecting ZFS on-disk format with Radare https://habr.com/ru/post/348354/
