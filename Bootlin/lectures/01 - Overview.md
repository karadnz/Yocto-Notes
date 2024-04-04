
https://bootlin.com/doc/training/yocto/

19-46
## Linux System Architecture

![[Pasted image 20240403162717.png]]

- User Space:
- Kernel:
- Bootloader:
- Hardware:

## Boot Sequence

![[Pasted image 20240403162813.png]]

## Embedded linux work

- **BSP Work**: porting the bootloader and Linux Kernel, developing Linux Device Drivers.
- **System Integration Work**: assembling all the user space components needed for the system, configure them, develop the upgrade and recovery mechanisms etc. (rootfs).
- **Application Development**: Write the company-specific applications and libraries.

## System Integration
![[Pasted image 20240403162942.png]]

## Embedded Linux build systems
![[Pasted image 20240403163433.png]]

- A wide range of solutions: Yocto/OpenEmbedded, PTXdist, Buildroot, OpenWRT, and more.
- Today, two solutions are emerging as the most popular ones
	- **Yocto/OpenEmbedded**
		- Builds a complete Linux distribution with binary packages. Powerful, but somewhat complex, and quite steep learning curve.
	- **Buildroot**
		- Builds a root filesystem image, no binary packages. Much simpler to use, understand and modify.
## Overview of Yocto build system
![[Pasted image 20240403163628.png]] 

- Yocto always builds binary packages (a “distribution”).
- The final root filesystem is generated from the package feed.

