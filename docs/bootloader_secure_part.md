### Patched UBootloader & Trustos
Pada perangkat ini, setelah bootloader di unlock,verifikasi pada boot.img dan seluruh part yg di chained vbmeta tetep di verifikasi dan dicek hash nya,tetapi khusus boot.img yg di modifikasi, 
kamu tetep dapat menggunakannya,tetapi biasanya, kamu harus melakukan sign terlebih dahulu dengan avbtool,untuk kunci .pem nya anda bisa generate sendiri, sedangkan dtbo tidak bisa di boot 
walaupun bootloader sudah di unlock, dan dtbo custom tsb sudah di sign. Patch pada repository ini memungkinkan kamu menggunakan vbmeta custom pada realme c11 2021/narzo 50i, cara ini juga
seharusnya dapat berfungsi pada realme c21y/c25y

### Apa yg dipatch pada stock UBoot
* Remove warning unlock when booting up
* Add function to boot into recovery by holding power and vol down button
* Add function to boot into fastboot(bootloader) by holding power and vol up button
* Replace the public key in U-Boot with a custom public key(in this func ```sprd_get_vboot_key*()```), so that a vbmeta signed with that key can be booted

### Apa yg dipatch pada stock trustos
* Patch pengecekan avb,sehingga unsign boot.img bisa di boot walaupun bootloader terkunci
* Saat ini custom dtbo baik sign/unsigned tidak dapat di boot di device ini jika kamu hanya melakukan patching trustos, kamu membutuhkan UBoot yg sudah dipatch pada bagian public key

### Bagaimana UBoot di patch
* Remove warning
  * Buka stock uboot-sign.bin with HxD
  * kemudian cari HEX yg menghasilkan kalimat ```INFO: LOCK FLAG IS : UNLOCK!!!``` dan ```WARNING: LOCK FLAG IS : UNLOCK, SKIP VERIFY!!!``` ganti dengan ```00``` sampai text tsb hilang
* Menambahkan fungsi boot ke recovery dan fastboot dengan tombol yg di kustomisasi
  * Open stock uboot with ghidra
  * Cari fungsi ```FUN_00003470```
  * Perhatikan bagian listing
  * ```asm                         
        FUN_00003470                                    XREF[1]:     FUN_00005910:0000595c(c)  
        00003470 01 1c 00 53     uxtb       w1,w0
        00003474 3f cc 01 71     cmp        w1,#0x73 ; KEY_VOLUMEUP
        00003478 80 00 80 52     mov        w0,#0x4 ; Fastboot mode
        0000347c e0 00 00 54     b.eq       LAB_00003498
        00003480 3f 98 01 71     cmp        w1,#0x66 ; KEY_HOME,this device not have home key gpio, you can change it later
        00003484 80 00 80 52     mov        w0,#0x4; Fastboot mode
        00003488 80 00 00 54     b.eq       LAB_00003498
        0000348c 3f c8 01 71     cmp        w1,#0x72 ; KEY_VOLUMEDOWN
        00003490 60 00 80 52     mov        w0,#0x3 ; Recovery mode
        00003494 e0 13 80 1a     csel       w0,wzr,w0,ne
        LAB_00003498                                    XREF[2]:     0000347c(j), 00003488(j)  
        00003498 c0 03 5f d6     ret
     ```
  * Kamu dapat mengetahui nilai KEY_VOLUMEUP, KEY_VOLUMEUP,KEY_HOME melalui [contoh code ini dan link ini](https://github.com/Ridhoardhiansyah7/bsp_bootloader_u-boot15_sharkl3/blob/main/include/dt-bindings/input/input.h#L114)
    * ```c
      #define KEY_HOME		102 // konvert ke hexadecimal, akan menghasilkan nilai 0x66 
      #define KEY_VOLUMEDOWN		114 // konvert ke hexadecimal, akan menghasilkan 0x72
      #define KEY_VOLUMEUP		115 // konvert ke hexadecimal, akan menghasilkan nilai  0x73
      ```
  * Kamu dapat mengetahui nilai CMD_FASTBOOT_MODE, CMD_RECOVERY_MODE melalui [contoh code ini dan link ini](https://github.com/Ridhoardhiansyah7/bsp_bootloader_u-boot15_sharkl3/blob/main/include/boot_mode.h)
    * ```c
      typedef enum {
	    CMD_UNDEFINED_MODE=0, // 0x00
    	CMD_POWER_DOWN_DEVICE, // 0x01
    	CMD_NORMAL_MODE, // 0x02
	    CMD_RECOVERY_MODE, //0x03
	    CMD_FASTBOOT_MODE, // 0x04
	    CMD_ALARM_MODE, // 0x05
	    CMD_CHARGE_MODE, // etc
	    CMD_ENGTEST_MODE,
    	CMD_WATCHDOG_REBOOT,
	    CMD_AP_WATCHDOG_REBOOT,
    	CMD_SPECIAL_MODE,
	    CMD_UNKNOW_REBOOT_MODE,
    	CMD_PANIC_REBOOT,
    	CMD_VMM_PANIC_MODE,   //0xd
    	CMD_TOS_PANIC_MODE,
    	CMD_EXT_RSTN_REBOOT_MODE,
    	CMD_CALIBRATION_MODE,
    	CMD_FACTORYTEST_MODE,
    	CMD_AUTODLOADER_REBOOT,
    	CMD_AUTOTEST_MODE,
    	CMD_IQ_REBOOT_MODE,
	    CMD_SLEEP_MODE,
    	CMD_SPRDISK_MODE,
    	CMD_APKMMI_MODE,
    	CMD_UPT_MODE,
    	CMD_APKMMI_AUTO_MODE,
    	CMD_ABNORMAL_REBOOT_MODE,

	    /*this is not a mode name ,beyond CMD_MAX_MODE means overflow*/
    	CMD_MAX_MODE
      }boot_mode_enum_type;
      ```
* Mengganti publickey pada uboot agar custom signed vbmeta bisa di boot
  * Menggapa tidak cukup hanya patch trustos ?
    * Saya sudah mencobanya, tetapi walaupun trustos sudah di patch agar avb tidak diverifikasi,hasilnya saat ini custom boot.img dapat di boot dengan baik, baik signed/unsigned dan dalam kondisi bootloader terkunci ataupun tidak; tetapi walaupun avb sudah di patch pada trustos, custom dtbo tidak bisa di boot pada device ini walaupun bootloader unlocked, sehingga kamu harus menghapus dtbo,dll(jika kamu mau) dari chained vbmeta agar tidak diverifikasi, dengan cara mengganti pubkey yg ada pada stock uboot, dengan pubkey yg kamu gunakan untuk melakukan sign custom vbmeta; berikut caranya
    * Ketahui terlebih dahulu, publickey yg di log ke ```uboot_log```
      * ```sh
        $ adb pull /dev/block/by-name/uboot_log
        ```
    * Buka ```uboot_log.bin``` menggunakan notepad, cari log kurang lebih seperti ini
      * ```log
        [00002130] uboot consume time:1378ms, lcd init consume:882ms, backlight on time:2130ms 
        [00002550] check_secboot_enable: t_lcs:5
        [00002552] cdt_table: check_secboot_enable ret:5sprd_get_vboot_key(): load_buf is 0x000000009efffe00.
        [00002560] sprd_get_vboot_key(): sechdr_offset is 0xcdb60.
        [00002565] sprd_get_vboot_key(): sechdr_addr is 0x000000009f0cd960.
        [00002570] sprd_get_vboot_key(): sechdr_addr: 000000009f0cd960. cert_addr: 000000009f0cd9c0. 
        [00002577] sprd_get_vboot_key(): sechdr_offset is 0xcdb60, sechdr_addr->cert_offset is 0xcdbc0.
        [00002584] cert_key
        [00002585] dumpHex:32 bytes
        [00002587] 0c382136f01e2f9469d63b359d88cb36
        [00002590] d52b7cfbd774936e6d80af88d0e3ed31
        [00002593] 
        ```
    * Salin ```0c382136f01e2f9469d63b359d88cb36d52b7cfbd774936e6d80af88d0e3ed31```, kemudian patch dengan pubkey kamu
      * ```
         sh uboot_pubkey_patcher.exe uboot.bin u-boot-patched.bin 0c382136f01e2f9469d63b359d88cb36d52b7cfbd774936e6d80af88d0e3ed31 eb0619c347731386b26d1a116a5705a437e3268d840d892d7fa83400ea89466f
        ```
      * note  ```uboot_pubkey_patcher``` dicompile code C ini
      * ```
        #include <stdio.h>
        #include <stdlib.h>
        #include <stdint.h>
        #include <string.h>
 
        #define ERR_EXIT(...)                 \
            do                                \
            {                                 \
                fprintf(stderr, __VA_ARGS__); \
                exit(1);                      \
            } while (0)

        /* Generic byte-sequence find & replace, meant for patching a hardcoded
         * key/hash blob (e.g. sprd_get_vboot_key()'s stock_vboot_key[32]) inside
         * a compiled binary. Both sequences must be the same length, so the file
         * size never changes and no wrapper/signature header needs to be
         * re-parsed or resized. */

        static uint8_t *loadfile(const char *fn, size_t *num)
        {
            size_t n = 0;
            uint8_t *buf = NULL;
            FILE *fi = fopen(fn, "rb");
            if (!fi)
                return NULL;
            fseek(fi, 0, SEEK_END);
            n = ftell(fi);
            fseek(fi, 0, SEEK_SET);
            buf = (uint8_t *)malloc(n);
            if (buf && fread(buf, 1, n, fi) != n)
            {
                free(buf);
                buf = NULL;
            }
            fclose(fi);
            if (num)
                *num = n;
            return buf;
        }

        static int hex_to_bytes(const char *hex, uint8_t *out, size_t out_len)
        {
            size_t hlen = strlen(hex);
            if (hlen != out_len * 2)
                return 0;
            for (size_t i = 0; i < out_len; i++)
            {
                unsigned v;
                if (sscanf(hex + i * 2, "%2x", &v) != 1)
                    return 0;
                out[i] = (uint8_t)v;
            }
            return 1;
        }

        int main(int argc, char **argv)
        {
            if (argc < 5)
                ERR_EXIT("Usage: %s <input.bin> <output.bin> <old_hex> <new_hex> [--force]\n"
                          "  old_hex / new_hex: hex string, same byte length (e.g. 64 hex chars for a 32-byte hash)\n"
                          "  --force: patch even if more than one match is found (default: abort and list offsets)\n",
                          argv[0]);

            const char *in_path = argv[1];
            const char *out_path = argv[2];
            const char *old_hex = argv[3];
            const char *new_hex = argv[4];
            int force = (argc >= 6 && strcmp(argv[5], "--force") == 0);

            size_t hex_len = strlen(old_hex);
            if (hex_len == 0 || hex_len % 2 != 0 || hex_len != strlen(new_hex))
                ERR_EXIT("old_hex and new_hex must be non-empty, even-length, and the same length\n");

            size_t needle_len = hex_len / 2;
            uint8_t *old_bytes = (uint8_t *)malloc(needle_len);
            uint8_t *new_bytes = (uint8_t *)malloc(needle_len);
            if (!hex_to_bytes(old_hex, old_bytes, needle_len) ||
                !hex_to_bytes(new_hex, new_bytes, needle_len))
                ERR_EXIT("Failed to parse hex input\n");

            size_t size = 0;
            uint8_t *mem = loadfile(in_path, &size);
            if (!mem)
                ERR_EXIT("loadfile(\"%s\") failed\n", in_path);
            if (size < needle_len)
                ERR_EXIT("File smaller than the search pattern\n");

            /* Find every occurrence first, so we never patch blind. */
            size_t matches[64];
            int match_count = 0;
            for (size_t i = 0; i + needle_len <= size; i++)
            {
                if (memcmp(&mem[i], old_bytes, needle_len) == 0)
                {
                    if (match_count < 64)
                        matches[match_count] = i;
                    match_count++;
                    printf("match at offset 0x%zx\n", i);
                }
            }
       
            if (match_count == 0)
            {
                printf("No match found -- old_hex not present in \"%s\" as given.\n", in_path);
                free(mem);
                return 1;
            }
            if (match_count > 1 && !force)
            {
                printf("%d matches found -- refusing to patch automatically.\n"
                       "Inspect the offsets above by hand, then re-run with --force if they're all legitimate.\n",
                       match_count);
                free(mem);
                return 1;
            }

            int to_patch = match_count < 64 ? match_count : 64;
            for (int i = 0; i < to_patch; i++)
            {
                memcpy(&mem[matches[i]], new_bytes, needle_len);
                printf("patched offset 0x%zx\n", matches[i]);
            }
       
            FILE *out = fopen(out_path, "wb");
            if (!out)
                ERR_EXIT("Failed to create \"%s\"\n", out_path);
            if (fwrite(mem, 1, size, out) != size)
                ERR_EXIT("Failed to write \"%s\"\n", out_path);
            fclose(out);

            printf("Wrote %zu bytes to \"%s\" (%d replacement%s).\n",
                   size, out_path, to_patch, to_patch == 1 ? "" : "s");

            free(mem);
            free(old_bytes);
            free(new_bytes);
            return 0;
        }
        ```
      * script akan menghasilkan output seperti ini
      * ```log
          match at offset 0xcdcec
          patched offset 0xcdcec
          Wrote 1048576 bytes to "u-boot-patched.bin" (1 replacement).
        ```
      *  Selanjutnya silakan buat vbmeta custom berdasarkan vbmeta stock,kamu bisa menghapus dan mengganti chained verification, tetapi ganti .pem dan pubkey dengan kunci milik anda sendiri, kemudian lakukan pad; kamu bisa menggunakan [alat ini](https://github.com/TomKing062/vendor_sprd_proprietories-source_packimage/releases/tag/1.5)
      * Langkah terakhir silakan flash trustos,uboot,dan vbmeta yg sudah di patch, dan hasilnya hp kamu saat ini menjalankan vbmeta custom, dengan beberapa chain yg telah kamu hapus;silakan coba flash dtbo dan lainnya yg telah kamu hapus dari chain verification
     
### Bagaimana TrustOS di patch
* TrustOS dipatch menggunakan [alat ini](https://github.com/TomKing062/action_spd_dump_it/blob/main/gen_tos-noavb.c)
*  Karna vbmeta custom bisa di boot pada device ini, mungkin kedepannya patch pada trustos ini tidak lagi diperlukan, kita hanya tinggal menghapus beberapa chain yg diverifikasi oleh vbmeta

### Notes
* Beberapa patch, harus di bypass signnya menggunakan [alat ini](https://github.com/TomKing062/unisoc_chipram_signcheck_exploit#patch-pre-verification)
* Patch ini belum di test pada device yg menggunakan LK sebagai bootloadernya & dengan BSP yg lebih baru
     
      
