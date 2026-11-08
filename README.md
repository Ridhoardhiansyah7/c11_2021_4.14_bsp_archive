# BSP Build, Patched image and port rom archive for RC11 2021 & Narzo 50i
* This repository contains only release archives specifically for the Realme C11 2021 and Narzo 50i with 4.14 kernel; archives with 5.4 kernel will be published later
* OpenSourceURL: https://github.com/Ridhoardhiansyah7/bsp_realme_c11_2021_4_14/tree/4.14.270

### Patched UBoot & ~~TrustOS~~ Overview
* Remove unlock warning on boot
* Hold power and vol down = fastboot(bootloader)
* Hold power and vol up = recovery mode
* The ~~TrustOS~~ & stock UBoot image has been patched to disable AVB, allowing unsigned custom boot,dtbo,vbmeta images to run even when the bootloader is locked.
* You can read complete documentation in [this doc](https://github.com/Ridhoardhiansyah7/c11_2021_4.14_bsp_archive/blob/main/docs/bootloader_secure_part.md)

### Custom Kernel for RMX3231 / RMX3235 Overview

* Linux version: 4.14.270
* Arch: armv8l/aarch64 (support 32/64bit userspace port & gsi rom)
* SELinux: Both (Enforcing & Permissive)
* FileSystem: EXT4 & EROFS
* You can read complete documentation in [this doc](https://github.com/Ridhoardhiansyah7/c11_2021_4.14_bsp_archive/blob/main/docs/kernel.md)

### PORT ROM Overview
This device can boot both 32bit (arm32_binder64) and 64bit (arm64) port rom, for the 64bit port, the vendor source is ported from the ZTE A52, some HAL, services, etc. still use the stock vendor source. You can read complete documentation in [this doc](https://github.com/Ridhoardhiansyah7/c11_2021_4.14_bsp_archive/blob/main/docs/port.md)
