# Banana PI R3 notes

I bought a [Banana PI R3](https://wiki.banana-pi.org/Banana_Pi_BPI-R3) to possibly use as a replacement
AP for my Ubiquiti AP AC Pro.

My plan is to run NixOS on the hardware. That's the easiest way to keep the software running on it up to date.

## References

- https://github.com/ghostbuster91/blogposts/blob/main/router2023/main.md
- https://github.com/ghostbuster91/nixos-router
- https://github.com/nakato/nixos-bpir3-example
- https://github.com/steveej-forks/nixos-bpir3
-

## Hardware

For storage, the hardware has:
- 32MB of NOR flash
- 128MB of NAND flash
- 8GB of eMMC
- SD Slot (bus shared with eMMC)

Booting off the different storage devices is configured with the jumpers.

Its not really clear why it has 3 different on-board flash chips. The [best explanation](https://forum.banana-pi.org/t/sd-vs-nor-vs-nand-vs-emmc-installation/14745/4) I've seen is that this is intended to be a reference board so there are multiple flash chips for the different reference use cases.

```
| Jumper Setting | SW1  | SW2  | SW5  | SW6  |
| SPIM-NoR       | Low  | Low  | Low  | X    |
| SPIM-Nand      | High | Low  | High | X    |
| eMMC           | Low  | High | High | Low  |
| SD card        | High | High | X    | High |
```

We'll install to eMMC.

eMMC is /dev/mmcblk0

```
# fdisk /dev/mmcblk0 -l
Disk /dev/mmcblk0: 7.28 GiB, 7818182656 bytes, 15269888 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

Nor flash:
```
# cat /proc/mtd
dev:    size   erasesize  name
mtd0: 00040000 00010000 "BL2"
mtd1: 00010000 00010000 "u-boot-env"
mtd2: 000b0000 00010000 "Factory"
mtd3: 00080000 00010000 "FIP"
mtd4: 00e00000 00010000 "firmware"

# mtdinfo -a
Count of MTD devices:           5
Present MTD devices:            mtd0, mtd1, mtd2, mtd3, mtd4
Sysfs interface supported:      yes

mtd0
Name:                           BL2
Type:                           nor
Eraseblock size:                65536 bytes, 64.0 KiB
Amount of eraseblocks:          4 (262144 bytes, 256.0 KiB)
Minimum input/output unit size: 1 byte
Sub-page size:                  1 byte
Character device major/minor:   90:0
Bad blocks are allowed:         false
Device is writable:             true

mtd1
Name:                           u-boot-env
Type:                           nor
Eraseblock size:                65536 bytes, 64.0 KiB
Amount of eraseblocks:          1 (65536 bytes, 64.0 KiB)
Minimum input/output unit size: 1 byte
Sub-page size:                  1 byte
Character device major/minor:   90:2
Bad blocks are allowed:         false
Device is writable:             true

mtd2
Name:                           Factory
Type:                           nor
Eraseblock size:                65536 bytes, 64.0 KiB
Amount of eraseblocks:          11 (720896 bytes, 704.0 KiB)
Minimum input/output unit size: 1 byte
Sub-page size:                  1 byte
Character device major/minor:   90:4
Bad blocks are allowed:         false
Device is writable:             true

mtd3
Name:                           FIP
Type:                           nor
Eraseblock size:                65536 bytes, 64.0 KiB
Amount of eraseblocks:          8 (524288 bytes, 512.0 KiB)
Minimum input/output unit size: 1 byte
Sub-page size:                  1 byte
Character device major/minor:   90:6
Bad blocks are allowed:         false
Device is writable:             true

mtd4
Name:                           firmware
Type:                           norhttps://github.com/ghostbuster91/nixos-router/blob/main/flake.nix
Eraseblock size:                65536 bytes, 64.0 KiB
Amount of eraseblocks:          224 (14680064 bytes, 14.0 MiB)
Minimum input/output unit size: 1 byte
Sub-page size:                  1 byte
Character device major/minor:   90:8
Bad blocks are allowed:         false
Device is writable:             true
```
