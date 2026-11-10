### CUSTOM Kernel For RC11 2021 / Narzo 50I
* Linux version: 4.14.270 & 5.4.147
* Arch: armv8l/aarch64
* SELinux: Both (Enforcing & Permissive)
* FileSystem: EXT4 & EROFS

### Compatibility
- [X] Port 32 & 64Bit ROM
- [X] GSI 32 & 64Bit
- [X] Stock 32Bit ROM

### Features Lists
* Backported BORE scheduler
* Backported MGLRU
* Backported EROFS
* Backported SUSFS (next update)
* WIREGUARD
* Overclock CPU(4x 1,4GHz Little clust, 4x 1,8GHz BIG clust) & GPU(250-850MHz)
* SDFAT support (This is only useful if you're using a Samsung Port ROM)
* ResukiSU root
* Etc

### Steps to flash
* Use the ADB platform tools to install a custom kernel
* ```sh
   $ adb reboot fastboot
   $ fastboot --disable_verity flash vbmeta [drag vbmeta.img] # you can get it from pac firmware with name `vbmeta-sign.img` or you can dump it using adb
   $ fastboot flash boot [drag boot image]
   $ fastboot flash socko [drag socko image]
   $ fastboot reboot
  ```

### Notes
* Flashing socko requires dm-verity to be disabled; if you have not disabled dm-verity since purchasing the device or flashing the firmware, please disable it first
* If a bootloop occurs after flashing the boot and socko partitions, you only need to format the data partition using spd_dump. This issue is very rare if you have already disabled verity. However, to be on the safe side, run `$ fastboot -w` before executing `$ fastboot reboot`)
* Some 32-bit GSIs may not boot on this device. This can also happen when using a custom boot.img, whether it contains a custom kernel or is Magisk-patched. I haven't looked into the exact cause yet, as most of my time has been focused on ROM porting and the 5.4 kernel
