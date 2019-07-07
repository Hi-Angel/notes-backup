# Efi partition

The EFI partition in use is usually mounted on EFI-booted systems in `/boo/efi`.

I'm not sure if that's how it works everywhere, but at least on MacBook for every OS there's a separate small EFI partition.

# Modifying boot behavior

Can be done with `efibootmgr`. Calling it without parameters gonna print available boot options.

* Booting to a system next time: use an option `-n XXXX`, where `XXXX` comes from `BootXXXX` entries. Note: this doesn't seem to work on Mac Air 2013.
