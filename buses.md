# general

IO registers mapped into memory. The only difference between accessing IO registers and RAM is that the former has side-effects. This has 3 implications: Memory Cache (solved by disabling caching for the specific address range), and compiler/hw optimization/reordering, solved by inserting a memory barrier. IO ports allocation is done with `*_request_region()`. For example on PCI the device gets notified about addresses of DMA.

**NOTE**: the programming guide is very confusing on distinction of IO registers and IO ports in kernel. It seems like DMA as a concept is not used, but instead IO ports do (i.e. outb, outw, etc).

Registering a driver happens with `foo_register_driver` and `foo_unregister_driver`. It ties the driver and device *(apparently it creates a device object that could later be used by udev for generation acc. dev at /dev/)*. If a bus supports vendor/product ids, drivers register matching ones. Afterwards `foo_probe()` is called in which device driver should check whether it really matches.

Out of context: in I2C If `foo_probe()` reports success (zero not a negative status code) it may save the handle and use it until `foo_remove()` returns.

Functions marked by `__init` are kept in special sections that are dropped after boot
(or module loading) is completed. Functions marked by `__exit` are called on module
exiting, and are dropped by the compiler when the code is built into the kernel.

**Power Management**: If your device needs special handling when entering a system
low power state — like putting a transceiver into a low power mode, or activating a
system wakeup mechanism — do that by implementing the appropriate callbacks for the
`dev_pm_ops` of the driver (like suspend and resume).

**System Shutdown**: if your device needs special handling when the system shuts down
or reboots (including kexec) — like turning something off — use a shutdown() method.

Device driver doesn't know where the device lives. The info is passed at probe time using `platform_device` object, and is described in "device tree".

Signal edge (фронт сигнала) — is a transition in a digital signal either from low to high (0 to 1) (нарастание, фронт) or from high to low (1 to 0) (спад).

Bear in mind code deduplication: if code α does something that's done in another kernel subsystem, then code α should use that other code from that other subsystem.

When don't know what to initialize a field to, zero often works.

Compiling a module is like `make M=fs/btrfs`, where the `fs/btrfs` could be found with `find` or `modinfo`. The module can then be loaded by path with `insmod`.

## references

https://lwn.net/Kernel/LDD3/ Linux Device Drivers, Third Edition
https://stackoverflow.com/questions/25161555/dma-and-i-o-memory-region-under-linux
Linux Device Drivers, Third Edition, chapter "Interrupt Handling": how to probe for IRQ manually.
https://unix.stackexchange.com/questions/47306/how-does-the-linux-kernel-handle-shared-irqs

# i2c

## misc

Consist of two lines: Serial Data Line (SDA) and Serial Clock Line (SCL). For example can be implemented with 2 GPIOs.

https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/Documentation/i2c/

A super-set of SMBus, so typically devices compatible with SMBus are also compatible with i²c. Except the obvious caveat that devices might claim to be compatible, but lack something.

General code is at least in algorithm driver and "driver driver".

Low level protocol is pretty simple, described at https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/Documentation/i2c/i2c-protocol along with function names for reading/writing. There are also present and described flags for device quirks.

`busses` subdir of docs contains info about various specific bus drivers.

i2c objects created for every device. Normally (e.g. on desktop) their presence is
determined with `SMBUS_QUICK` protocol, but sometimes requires there may be a table, in
the kernel or the boot loader, identifying I2C devices and linking them to
board-specific configuration information about IRQs and other wiring artifacts, chip
type, and so on. Either way drivers have to provide `foo_probe()/foo_remove()` to (un)bind the device.

Device can be unregistered either explicitly with `i2c_unregister_device()`, or the function automatically gets called when the bus on which device resides gets destroyed.

Overview of creating a device driver and communication (including SMBus) is at `writing-clients` docs file.

Device drivers can be instantiated from user-space by writing into sysfs (useful for developing and reverse-engineering). Aside of that, device-tree works, acpi does, and manual configuration in c driver for the board.

## Device Creation (as I understand, the POV of a chip driver)

If you know for a fact that an I2C device is connected to a given I2C bus,
you can instantiate that device by simply filling an i2c_board_info
structure with the device address and driver name, and calling
i2c_new_device().  This will create the device, then the driver core will
take care of finding the right driver and will call its probe() method.
If a driver supports different device types, you can specify the type you
want using the type field.  You can also specify an IRQ and platform data
if needed.

Sometimes you know that a device is connected to a given I2C bus, but you
don't know the exact address it uses.  This happens on TV adapters for
example, where the same driver supports dozens of slightly different
models, and I2C device addresses change from one model to the next.  In
that case, you can use the i2c_new_probed_device() variant, which is
similar to i2c_new_device(), except that it takes an additional list of
possible I2C addresses to probe.  A device is created for the first
responsive address in the list.  If you expect more than one device to be
present in the address range, simply call i2c_new_probed_device() that
many times.

The call to i2c_new_device() or i2c_new_probed_device() typically happens
in the I2C bus driver. You may want to save the returned i2c_client
reference for later use.

# SPI

There're 2 types of drivers: driver for the SPI controller (that manages power, suspend, buffering, etc.), and a protocol driver, (just sends/recives a data to/from a slave device).

Usually SPI controller is a master, and devices are slaves. It can be the other way round, for now let's not dig into it (acc. to docs it's not even implemented in Linux kernel, so it seems extremely uncommon).

There are 4 types of wires/signals:
1. CSLK (aka SCK) (Serial Clock) — used by master for synchronization of data transmission. Upon getting signalled data in MISO and MOSI are shifted. It seems data in master and slave are stored in some shift register, and for some number of shifts their datas gets swapped. This means that sometimes reading or writing register may contain a junk; and also that amount of bytes read equals to amount of bytes written.
2. MISO (Master Input, Slave Output) — data transmission master → slave.
3. MOSI (Master Output, Slave Input) — data transmission slave → master.
4. CS (Chip Select) (aka SS — Slave Select) — a wire that activates a slave device. Low power means the active device.

There's `spidev` driver that allows communication with the device from userspace through read()/write() of an acc. symbol device.

Communication in protocol drivers is using (explicitly or implicitly) struct `spi_message`, where the list of transactions is that of `spi_transfer`. Note that data in `spi_transfer` should be allocated through `kmalloc()` (or a variant of) with a flag for being DMA accessible (an example uses `GFP_KERNEL`).

Data should be read/write in word sizes — an attempt to push a smaller amount of bytes would result in error.

**Finding device** is not supported. Usually devices are soldered into a chip, and don't imply hot enabling/disabling.

**menuconfig** Device Drivers → SPI suppport.

## writing a driver

Given a protocol module for SPI, you'd have to add the module to a list of `spi_board_info` inside the source code for the board.

Cross-compilation `make` for the kernel is:

    ARCH=arm CROSS_COMPILE=/opt/arm-2010q1/bin/arm-none-linux-gnueabi- make

## references

https://habrahabr.ru/post/123145/ part 1.
https://habrahabr.ru/post/123266/ part 2. Which includes a module code and a Makefile example.

# GPIO

A generic pin on an integrated circuit or computer board whose behavior—including whether it is an input or output pin—is controllable by the user at run time. It's typical to have free pins that go unused by default (for the safe case of a need to add a new device).

**Usage Examples**: on a given board each GPIO is used for one specific purpose
like monitoring MMC/SD card insertion/removal, detecting card write-protect status,
driving a LED, configuring a transceiver, bit-banging a serial bus, poking a hardware
watchdog, sensing a switch, and so on.

**From userspace** it's easy to use: writing a number of the line into /sys/class/gpio/export, which creates a new dir there. Then writing to "direction" file in the subdir "out" and to "value" file "1" or "0" (the symbol, not a number).

**From the kernel** using a GPIO is similar to that of userspace (as in, setting a direction, then writing values).

with the descriptor-based interface, GPIOs are identified with an opaque, non-forgeable handler that must be obtained through a call to one of the `gpiod_get()` functions. When a functional is implemented with multiple GPIOs (e.g. a LED with digits), used a variant of the function with index (see https://www.kernel.org/doc/Documentation/gpio/consumer.txt).

Note: also there's a deprecated interface in kernel that's still in use in some places.

## references

https://lwn.net/Articles/533632/ new GPIO API, and managing multiple GPIOs as a block.

# USB

The protocol complexity is handled by USB-core in the kernel. Then USB-core hands structs to device drivers, who simply use them, as briefed below, to manage the device.

USB uses a tree structure, with the host as the root (the system’s master), hubs as interior nodes, and peripherals as leaves (and slaves).

Consist of 4 wires (ground, power, 2 signal wires). It's pretty simple, but possesses a few interesting properties, such as devices being able to request a fixed bandwidth (useful for video-cameras).

USB drivers running in peripheral devices are called gadget drivers. Host drivers talk to "usbcore" APIs. There are two. One is intended for general-purpose drivers (exposed through driver frameworks), and the other is for drivers that are part of the core. Such core drivers include the hub driver (which manages trees of USB devices) and several different kinds of host controller drivers, which control individual busses.

There's a usb-skeleton driver.

USB drivers typically hook into another sybsystem (being e.g. a HDD, or network adapter, etc.). But some of them don't have a kernel subsystem (such is MP3 players), thus require interaction from userspace.The USB subsystem provides a way to register a minor device number and a set of `file_operations` function pointers that enable this user-space interaction.

## Endpoints

An endpoint is a uniquely addressable portion of the peripheral that's either destination or receiver of data. Address is defined by 4 bits. Upon establishing connection each endpoint returns a descriptor which tells stuff like transfer type, max size of data packets, bandwidth, transfers interval.

Types:

1. **Control**: used for configuring device. There's guaranteed to be 0 endpoint that's used by usb-core at insertion time.
2. **Interrupt** sends small number of data. Have a guaranteed bandwidth, used e.g. by mouses, keyboards.
3. **Bulk** used for sending large amounts of data. Don't have a guaranteed bandwidth, so data can be held and broken to multiple pieces.
4. **Isochronous** used for sending large amounts of data. These send as much data as the bandwidth allows, everything else is dropped. Used e.g. by audio, video cameras.

Endpoint info is stored in kernel struct `usb_host_endpoint::usb_endpoint_descriptor`. `bEndpointAddress` of the latter is the usb address; can be used with bitmasks `USB_DIR_OUT`, `USB_DIR_IN` to determine the direction. The struct have values named exactly after their counterpart in usb specification.

## Interfaces

Endpoints are bundled into interfaces. A single physical device might have multiple of them with a different functional. E.g. a USB speaker might consist of 2 interfaces: an audio stream, and a keyboard for the buttons.

Interfaces might have different settings, numbered from 0 (being default). Alternate settings can be used to control individual endpoints in different ways, such as to reserve different amounts of USB bandwidth for the device.

Described in `usb_interface` struct that is passed by usb core to the underlying driver. Acc. to docs USB driver is in charge of controlling that struct.

## Configurations

Interfaces in turn are bundled into configurations. It's rare to have multiple. Only single configuration might be enabled in a given point of time. Examples include a device with a configuration to apply firmware.

## Kernel Misc

Communication in kernel is using struct `urb` (USB request blocks). There can be used one for many devices, multiple for one device. A typical lifecycle of a urb is: creating in device driver, sending to USB-core, and getting a notification that USB-core is done with the urb. For more info see "USB urbs" chapter in Linux Device Drivers.

There's also a simpler interface for sending data such as `usb_bulk_msg()`, though internally those are using urbs anyway.

Registering is: Driver registers itself within a USB, and supplies vid:pid of the matching device.

# PCI

Each PCI peripheral is identified by a bus number, a device number, and a function number. The PCI specification permits a single system to host up to 256 buses, but because 256 buses are not sufficient for many large systems, Linux now supports PCI domains. Each bus hosts up to 32 devices, and devices host up to 8 functions (so every bus+device+function is uniquely identified by 16 bits).

Multiple PCI buses plugged into a system with "bridges", a PCI-peripheral whose task is joining two buses.

Every PCI device has a set of registers called the device's configuration space which, among other things, display the device ID (DID), the vendor ID (VID) and the device class.

Memory and I/O region accesses are visible to all devices on the bus, whilst configuration accesses to only the target.

On boot, when power is applied to a PCI device, the hardware remains inactive. In other words, the device responds only to configuration transactions. At power on, the device has no memory and no I/O ports mapped in the computer’s address space; every other device-specific feature, such as interrupt reporting, is disabled as well. At system boot, the firmware (or the Linux kernel, if so configured) performs configuration transactions with every PCI peripheral in order to allocate a safe place for each address region it offers. By the time a device driver accesses the device, its memory and I/O regions have already been mapped into the processor’s address space. The driver can change this default assignment, though never needs to.

There's a `pci-skeleton.c` driver.

PCI requires devices to announce which IRQ they're going to use, so figuring out which IRQ the device have by driver amounts to simply requesting it from the PCI.

Tho the standard is hw-independent, but values in configuration space is stored at little-endian.

Devices differentiated by at least 3 registers: vid, pid, class. Additionally sometimes there're subsystem's pid and vid.

# PCIe

From sw POV there's no difference against plain PCI. Worth noting though MSI, but it was present in later standards of PCI either.

# Thunderbolt

Encapsulates PCIe, DP, and USB inside a single interface. These protocols are "tunnelled" over thunderbolt.

NHI (native host interface) is a name of Thunderbolt controller on Apple systems. At least on Apple systems, on resume from suspend, NHI needs to [re-establish PCIe tunnels before PCIe ports are resumed](https://www.spinics.net/lists/linux-usb/msg199555.html).
