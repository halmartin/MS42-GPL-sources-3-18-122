# RedBoot

RedBoot is the bootloader supplied by Vitesse in their vendor SDK (likely from VSC6816).

It is used by several vendors shipping products based on the luton26 / jaguar / jaguar dual:
- Cisco Meraki MS220, MS22, MS42, MS220, MS320 (Linux-based firmware)
- Netonix WISP (Linux-based firmware)
- Blackbox LGB5510A (eCos-based firmware)
- Repotec RP-GS2024I (eCos-based firmware)

## Meraki RedBoot

You can build the Meraki RedBoot from source. By default RedBoot is built when you compile kernel 3.18.122 from this repository.

The RedBoot binary can be found at `arch/mips/vcoreiii/loader/loader.bin` however this is not an image suitable for flashing. I do not know how to create a flashable image, no instructions were provided.

You can view the assembly for RedBoot in `arch/mips/vcoreiii/loader/head.S`

I have modified the bootloader and am providing 2 modified versions:
1. Without CRC verification (nocrc)
    1. Changed `bnez    s0,0x990` to a `noop`
1. Without CRC verification AND without size verification to allow booting kernels larger than 3932160 bytes (nocrc-sz)
    1. Changed `bltz    t0,0x990` to a `noop`

The no-CRC and no-size version of the bootloader is sufficient to allow you to boot 3.18.123 directly from SPI.

## Compressed kernels

I have not been able to get compressed kernels to boot.

If you wish to try, you will need to add `select SYS_SUPPORTS_ZBOOT` to the `config MERAKI_MSXX` section in `arch/mips/Kconfig`

The kernel decompressor appears to be at the start of the data region, so the kernel entry point and load address should be the same: `0x80100000`

You can automate this by adding the following to `arch/mips/boot/compressed/Makefile`:
```
--- a/arch/mips/boot/compressed/Makefile
+++ b/arch/mips/boot/compressed/Makefile
@@ -68,6 +68,8 @@ hostprogs-y := calc_vmlinuz_load_addr
 
 ifeq ($(CONFIG_MACH_JZ4740),y)
 VMLINUZ_LOAD_ADDRESS := 0x80600000
+else ifeq ($(CONFIG_MERAKI_MSXX),y)
+VMLINUZ_LOAD_ADDRESS := 0x80100000
 else
 VMLINUZ_LOAD_ADDRESS = $(shell $(obj)/calc_vmlinuz_load_addr \
                $(obj)/vmlinux.bin $(VMLINUX_LOAD_ADDRESS))
```
