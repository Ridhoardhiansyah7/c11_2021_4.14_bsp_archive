### Patched UBootloader & Trustos
Behavior pada perangkat ini, setelah bootloader di unlock,verifikasi pada boot.img dan seluruh part yg di chained vbmeta tetep di verifikasi dan dicek hash nya,tetapi khusus boot.img yg di modifikasi, 
kamu tetep dapat menggunakannya,tetapi, kamu harus melakukan sign terlebih dahulu dengan avbtool dan dengan kunci 4096.pem kamu sendiri, sedangkan untuk dtbo custom, pada perangkat ini tidak bisa di boot 
walaupun bootloader sudah di unlock dan dtbo custom tsb sudah di sign. Patch pada repository ini memungkinkan kamu menggunakan vbmeta custom pada realme c11 2021/narzo 50i, dengan begitu
kita bisa menghapus boot,dtbo,dll dari mekanisme verifikasi yg dilakukan uboot dan trustos,cara ini juga seharusnya dapat berfungsi pada realme c21y/c25y,tetapi belum di test pada perangkat yg menggunakan
LK sebagai bootloadernya dan pada perangkat dengan BSP yg lebih baru

### Apa yg dipatch pada stock UBoot
* Menghapus warning message yg ditampilkan saat boot
* Ketika tombol power dan vol down ditekan bersamaan = reboot recovery
* Ketika tombol power dan vol up ditekan bersamaan = reboot fastboot
* Mengganti public key di UBoot stock(pada fungsi ```sprd_get_vboot_key*()```) dengan public key vbmeta custom kamu, sehingga custom signed vbmeta dapat di boot dengan baik

### Bagaimana UBoot di patch
* Remove warning
  * Buka stock uboot-sign.bin dengan HxD / GHIDRA
  * kemudian cari HEX yg menghasilkan kalimat ```INFO: LOCK FLAG IS : UNLOCK!!!``` dan ```WARNING: LOCK FLAG IS : UNLOCK, SKIP VERIFY!!!``` ganti dengan ```00``` sampai text tsb hilang
* Menambahkan fungsi boot ke recovery dan fastboot dengan tombol yg di kustomisasi
  * Buka stock uboot dengan GHIDRA
  * Cari fungsi ```FUN_00003470```
  * Perhatikan bagian listing
  * ```asm                         
        					FUN_00003470                                    XREF[1]:     FUN_00005910:0000595c(c)  
        00003470 01 1c 00 53     uxtb       w1,w0
        00003474 3f cc 01 71     cmp        w1,#0x73 ; KEY_VOLUMEUP
        00003478 80 00 80 52     mov        w0,#0x4 ; CMD_FASTBOOT_MODE
        0000347c e0 00 00 54     b.eq       LAB_00003498
        00003480 3f 98 01 71     cmp        w1,#0x66 ; KEY_HOME,this device not have home key gpio, you can change it later
        00003484 80 00 80 52     mov        w0,#0x4; CMD_FASTBOOT_MODE
        00003488 80 00 00 54     b.eq       LAB_00003498
        0000348c 3f c8 01 71     cmp        w1,#0x72 ; KEY_VOLUMEDOWN
        00003490 60 00 80 52     mov        w0,#0x3 ; CMD_RECOVERY_MODE
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
  *  Apa fungsinya ?
    *  Dengan mengganti pubkey yg ada di uboot khususnya pada fungsi ```sprd_get_vboot_key*()``` memungkinkan kita menggunakan vbmeta custom yg ditandatangani dengan kunci kita sendiri,dengan demikian, kita bisa melakukan booting custom modifikasi boot,dtbo, dll yg di chained ke vbmeta baik saat bootloader terkunci maupun tidak, berikut caranya
    * Ketahui terlebih dahulu, publickey yg di log ke ```uboot_log```:
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
    * Salin ```0c382136f01e2f9469d63b359d88cb36d52b7cfbd774936e6d80af88d0e3ed31``` dari ```uboot_log```
  	* Analisa public key yg nantinya key tsb akan kamu gunakan untuk sign custom vbmeta, lakukan hal ini:
  	* ```sh
  	     $ strings rsa4096_vbmeta.pem
  	     $ python avbtool extract_public_key --key rsa4096_vbmeta.pem --output testkey.avbpubkey
  	  ```
  	* Hasil dari command diatas seperti ini,silakan disalin bagian pubickey(hasil dari command kedua diatas(python vbtool extract_public_key ....))
  	* ```sh
  	     -----BEGIN RSA PRIVATE KEY-----
  	     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
  	     -----END RSA PRIVATE KEY-----
  	  
  	     eb0619c347731386b26d1a116a5705a437e3268d840d892d7fa83400ea89466f
  	  ```
  	* salin ```eb0619c347731386b26d1a116a5705a437e3268d840d892d7fa83400ea89466f```, dan gunakan code c, dibawah untuk patch uboot nya
  	* ```sh
  	     $ # [uboot_pubkey_patcher.exe] [stock_uboot_file_name] [result_patched_file_name] [stock uboot_log public key] [your vbmeta.pem public key]
         $ uboot_pubkey_patcher.exe uboot.bin u-boot-patched.bin 0c382136f01e2f9469d63b359d88cb36d52b7cfbd774936e6d80af88d0e3ed31 eb0619c347731386b26d1a116a5705a437e3268d840d892d7fa83400ea89466f
       ```
  	* catatan  ```uboot_pubkey_patcher``` dicompile code C ini
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
  	* setelah di compile dan dijalankan, akan menghasilkan output seperti ini
  	* ```log
          match at offset 0xcdcec
          patched offset 0xcdcec
          Wrote 1048576 bytes to "u-boot-patched.bin" (1 replacement).
      ```
  	*  Selanjutnya silakan buat vbmeta custom berdasarkan vbmeta stock,kamu bisa menghapus dan mengganti chained verification, tetapi ganti .pem dan pubkey dengan kunci milik anda sendiri, kemudian lakukan pad; kamu bisa menggunakan [alat ini](https://github.com/TomKing062/vendor_sprd_proprietories-source_packimage/releases/tag/1.5)
  	* Langkah terakhir silakan flash ~~trustos,~~ uboot,dan vbmeta yg sudah di patch melalui fastbood/spd_dump, setelah itu hp kamu dapat menjalankan vbmeta custom dengan beberapa chain yg telah kamu hapus/modifikasi; silakan coba flash dtbo dan lainnya yg telah kamu hapus dari chain verification
     
### ~~Bagaimana TrustOS di patch~~
* ~~TrustOS dipatch menggunakan [alat ini](https://github.com/TomKing062/action_spd_dump_it/blob/main/gen_tos-noavb.c)~~

### Notes
* DWYOR, sebelum flashing backup terlebih dahulu stock uboot,vbmeta dll, dan pastikan kamu tau cara melakukan flashing via `spd_dump` ataupun `research download` jika terjadi bootloop/ataupun layar hitam(softbrick)
* Karna vbmeta custom bisa di boot pada device ini, patch pada trustos tidak lagi diperlukan, kita hanya harus menghapus beberapa chain yg diverifikasi oleh vbmeta dengan melakukan patch stock uboot dan vbmeta itu sendiri
* Terkadang, beberapa patch, harus di bypass signnya menggunakan [alat ini](https://github.com/TomKing062/unisoc_chipram_signcheck_exploit#patch-pre-verification)
* Patch ini belum di test pada perangkat yg menggunakan LK sebagai bootloadernya & pada perangkat dengan BSP yg lebih baru
     
      
