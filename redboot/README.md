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
# Without CRC verification (nocrc)
## Changed `bnez    s0,0x990` to a `noop`
# Without CRC verification AND without size verification to allow booting kernels larger than 3932160 bytes (nocrc-sz)
## Changed `bltz    t0,0x990` to a `noop`

The no-CRC and no-size version of the bootloader is sufficient to allow you to boot 3.18.123 directly from SPI.

## Compressed kernels

Compressed kernels do not boot. Looking at the kernel decompressor stub, it seems the bootloader is supposed to pass arguments to the kernel in registers s0, s1, s2, and s3:
```
start:
        /* Save boot rom start args */
        move    s0, a0
        move    s1, a1
        move    s2, a2
        move    s3, a3
```

RedBoot has part of the CRC in s1, and I don't know the contents of s0, s2, or s3 when it jumps into the Linux kernel.

If anyone knows the expected contents of s0, s1, s2, and s3 when the bootloader jumps to the kernel entry address, please submit a PR.
