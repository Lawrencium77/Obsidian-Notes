This reading was inspired by [an OSTEP chapter](OSTEP/Part%20III%20-%20Persistence/1%20-%20IO%20Devices.md). 

My main takeaway from this chapter is that device drivers are simply object modules that are included in the OS kernel.

How do these get installed? It turns out there's a couple of ways.

## Pre-Installed Drivers
Most OSs come with a set of pre-installed device drivers. These allow interaction with a range of common devices.

## Adding Drivers to the OS
Integrating driver software into the Linux kernel can be added in two ways:

1. **Built-in drivers** - we can compile them into the kernel image.
2. **Loadable Kernel Modules (LKMs)** - these can be loaded or unloaded from the running kernel as needed. 

[As far as I can tell](https://medium.com/geekculture/linux-device-drivers-1334e5336fc6), the difference between these two is that the former are [statically linked](CSAPP/Chapter%207/Static%20Linking.md), while the latter are [dynamically linked](CSAPP/Chapter%207/Dynamic%20Linking%20with%20Shared%20Libraries.md) at runtime. 

It's worth noting device drivers aren't the only type of [LKM](https://en.wikipedia.org/wiki/Loadable_kernel_module).

You can view all currently-loaded kernel modules in Linux using `lsmod`. 

