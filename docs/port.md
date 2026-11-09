### PORT ROM For RC11 2021 / Narzo 50I
Port ROM allows us to run ROMs from other phones to this device. For 32-bit ROMs, the vendor used is the stock vendor, while for 64-bit ROMs, the vendor is ported from the ZTE A52.


### List Port ROM
* 32BIT ROM
  * Android 12 (SDK31)
    * [HIOS11 From Tecno POP7](https://www.mediafire.com/file/mkfaoxpdldpxt7v/POP7_FOR_RMX3231_RMX3235.rar/file?action=check)
    * ItelOS From Itel S18
    * [Realme UI GO From Realme c30s](https://www.mediafire.com/file/j7smfae2medkurc/RC30S_ROM_FOR_RC11_2021_AND_NARZO_50I.rar/file?action=check)
  * Android 13 (SDK33)
    * DOKEUI4.0 From BlackView wave 6c
* 64BIT ROM
  * Android 11 (SDK30)
    * MYOS11 From ZTE A52
  * Android 13 (SDK33)
    * XOS13 From Infinix Smart8
    * [MYOS13 From ZTE A54](https://www.mediafire.com/file/43j0j7f4o4fga9j/ZTE_A54_FOR_RMX3231_RMX3235.rar/file?action=check)
    * [ONEUI5 From Samsung A03](https://www.mediafire.com/file/zc24icp2cgytsen/A035F_FOR_RMX3231_RMX3235.rar/file?action=check)
    * [HIOS13 From Tecno Pop8](https://www.mediafire.com/file/ds7wu459l21jmfx/TECNO_POP8_FOR_RMX3231_RMX3235.rar/file?action=check)
    * DOKEUI4.0 From BlackView A52
    * ColPadOS From Colpad cp12
  * Android 14 (SDK34)
    * [HyperOS (this is GSI but ported, not recomendded very lags in this devices)](https://drive.google.com/file/d/1xrwi7Eh9Jf0Cj762EGPvWSmqiQhy_Tn1/view?usp=sharing)

### BUGS
Some ports have BUGS, some don't, if the ROM has BUGS, you can fix them by flashing the modules in the port archive via a root manager such as BakaSU/KernelSU after flashing port rom

### Compatibility
* Both custom boot images and ports ROM can be installed on any PAC firmware version for this device, provided the installation is performed correctly. For more detailed information, please refer to Telegram or submit a question through a GitHub issue

### Steps to flash port rom
* Use the ADB platform tools to install a ported ROM
* Preparation
* ```sh
    $ adb shell getprop ro.boot.slot_suffix # result must _a
    $ adb reboot fastboot
  ```
* Disable verity
* ```sh
    $ fastboot --disable_verity flash vbmeta [drag vbmeta.img] # you can get it from pac firmware with name `vbmeta-sign.img` or you can dump it using adb
  ```
* Flash port rom
* ```sh
    $ fastboot flash boot [drag boot.img file inside port_rom.zip]
    $ fastboot flash socko [drag socko.img file inside port_rom.zip]
    $ fastboot flash super [drag super.img file inside port_rom.zip]
    $ fastboot flash logo [drag logo.img file inside port_rom.zip] # this step is optional
   ```
* Last step format data and reboot
* ```sh
    $ fastboot -w
    $ fastboot reboot
  ```
* Done

### Notes
* If you have the 2/32 variant, it is best to use the 32bit port so that there is no lag when used, for 3/32 and 4/64 you are free to use both(32/64bit port)
* The result of the step `adb shell getprop ro.boot.slot_suffix ` must produce `_a`
* Flashing `logo.img` is optional
* Do not add `--disable_verification` when flashing `vbmeta`
