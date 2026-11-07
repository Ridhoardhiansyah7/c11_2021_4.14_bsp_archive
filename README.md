# BSP Build & Patched image archive for RC11 2021 & Narzo 50i
* This repository contains only release archives specifically for the Realme C11 2021 and Narzo 50i with 4.14 kernel; archives with 5.4 kernel will be published later
* OpenSourceURL: https://github.com/Ridhoardhiansyah7/bsp_realme_c11_2021_4_14/tree/4.14.270

### Patched UBoot
* Remove unlock warning on boot
* Hold power and vol down = fastboot(bootloader mode)
* Hold power and vol up = recovery mode
* When a bootloop occurs due to an AVB issue, a kernel startup failure, an invalid DTBO structure, etc; the solution is to reboot into fastboot (bootloader) rather than letting the device remain in a bootloop.

### Patched TrustOS
* The TrustOS stock image has been patched to disable AVB, allowing unsigned custom boot.img images to run even when the bootloader is locked.

### Custom Kernel for RMX3231 / RMX3235

* Linux version: 4.14.270
* Arch: armv8l/aarch64 (support 32/64bit userspace port & gsi rom)
* SELinux: Both (Enforcing & Permissive)
* FileSystem: EXT4 & EROFS


#### Kernel Features list

* Overclock CPU\&GPU
* Backported BORE scheduler
* Backported MGLRU
* Backported EROFS
* Backported WIREGUARD
* Backported SUSFS (next update)
* SDFAT support (This is only useful if you're using a Samsung Port ROM)
* ResukiSU root
* Etc


#### Important Notes:

* If you use a custom Unisoc LittleKernel (LK) BSP or a leaked U-Boot, both can boot on this device; however, the handshake process with the kernel for the display has not been successful, so even if the probe is successful, touch input does not work (likely due to an issue with interrupts or an incorrect reset GPIO).
* The 5.4 kernel boots normally, but the device sometimes reboots randomly—especially while charging. In terms of partition structure, this device comes with an empty vendor_boot.img partition by default, with a size of 100 MB. You’ll need to flash the boot, socko, and vendor_boot partitions if you want to use the 5.4 kernel.

