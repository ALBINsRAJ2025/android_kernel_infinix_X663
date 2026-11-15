# kernel/infinix/X663

Prebuilt kernel for the **Infinix Note 11 (X663)**, MediaTek MT6768
(Helio G88).  No kernel source is available in the released firmware, so this
tree ships the stock binaries — the same approach used by the existing
community kernel repo for this device.

## Contents

| File | Description |
| --- | --- |
| `Image.gz` | Stock kernel, `4.19.191-g20c2dd72f5fd-dirty` (10,855,427 bytes) |
| `dtb/mt6768.dtb` | DTB container from the stock boot image (119,464 bytes, magic `0xd7b7ab1e`, one FDT entry) |
| `dtbo.img` | Unsigned DTBO table (49,472 bytes); the build pads and signs it to the 8 MiB partition |
| `defconfig` | Kernel config extracted from `/proc/config.gz` |

### Why the file names differ from the unpacked image

Decoding the stock `boot.img` with `unpack_bootimg` produces three files named
after the boot-image segments, while this tree uses the names the Android
build system expects.  The bytes are identical — only the names differ:

| `unpack_bootimg` output | This tree | Bytes | SHA-256 |
| --- | --- | ---: | --- |
| `kernel` | `Image.gz` | 10,855,427 | `2a9dc96352c99a9462935f0024b53696a3f087320596a412125a41a997be72c5` |
| `dtb` | `dtb/mt6768.dtb` | 119,464 | `6265c98f0439f053e1b27358d937b9e432f2f38fad93fcae8b2846861a750815` |
| `ramdisk` | *(not shipped)* | 16,201,880 | gzip cpio, 213 files / 205 symlinks |
| — | `dtbo.img` | 49,472 | first 49,472 bytes of the stock 8 MiB `dtbo.img` |

`Image.gz` is the name AOSP requires (`BOARD_KERNEL_IMAGE_NAME := Image.gz`);
`dtb/` is the directory AOSP concatenates into `dtb.img`
(`BOARD_PREBUILT_DTBIMAGE_DIR`).  The stock ramdisk is not shipped here — the
build generates its own first-stage ramdisk from `device/infinix/X663`.

The `.dtb` file is a `dt_table` container (not a raw FDT): the single entry
points at an FDT with the usual `0xd00dfeed` magic.  This is exactly what the
stock boot image carries in its DTB region, so `BOARD_PREBUILT_DTBIMAGE_DIR`
+ `cat *.dtb` reproduces the 119,464-byte blob byte for byte.  The container
header is big-endian: magic `0xd7b7ab1e`, total size 119,464, header and entry
size 32, one entry, entries at offset 32, page size 2,048; the FDT itself sits
at offset 64 with `totalsize = 119,400`.

Decompiled with `dtc`, the DTB reports `model = "MT6768"`,
`compatible = "mediatek,MT6768"` and the bootargs

```
console=tty0 console=ttyS0,921600n1 root=/dev/ram vmalloc=400M
slub_debug=OFZPU swiotlb=noforce cgroup.memory=nosocket,nokmem
firmware_class.path=/vendor/firmware page_owner=on loop.max_part=7
```

The same `dtc` pass over the stock `dtbo.img` yields the separate DTBO
device tree (one `dt_table` entry, 45,708-byte table, FDT of 45,644 bytes at
offset 64).

`dtbo.img` was extracted from the stock 8 MiB partition and had its stock AVB
hash footer removed (the footer reported `original_image_size = 49472`).  It
must stay unsigned: the build copies it and runs `avbtool add_hash_footer
--partition_size 8388608` itself, which would fail on an already-signed image.

The device tree references these files directly from `BoardConfig.mk`:

```make
TARGET_PREBUILT_KERNEL      := kernel/infinix/X663/Image.gz
BOARD_PREBUILT_DTBIMAGE_DIR := kernel/infinix/X663/dtb
BOARD_INCLUDE_DTB_IN_BOOTIMG := true
BOARD_KERNEL_SEPARATED_DTBO := true
BOARD_PREBUILT_DTBOIMAGE   := kernel/infinix/X663/dtbo.img
```

## Rebuilding from source

If Infinix/Transsion publishes the 4.19.191 source for `x663_h6915`
(check the Transsion open-source portal), the stock config is in `defconfig`.
A source build only needs to produce:

* `arch/arm64/boot/Image.gz`
* `arch/arm64/boot/dts/mediatek/mt6768.dtb`
* `arch/arm64/boot/dtbo.img` (or keep the prebuilt one)

and the resulting `Image.gz`/`dtb` can replace the files here without touching
`BoardConfig.mk`.

## Notes

* The kernel is **not GKI**: boot header v2, kernel 4.19, main DTB inside the
  boot image and a separate `dtbo` partition.
* Boot header values (base `0x40078000`, kernel offset `0x8000`, ramdisk offset
  `0x07c08000`, tags/dtb offset `0x0bc08000`, page size 2048) were read from
  the stock `boot.img` and are set in the device tree.  The stock header v2
  also records `dtb_size = 119,464` and `dtb_addr = 0x4bc80000`; AOSP
  `mkbootimg` writes the same values because
  `BOARD_DTB_OFFSET = 0x0bc08000`.  The DTB region starts at byte 27,062,272
  and is byte-identical to `dtb/mt6768.dtb`.  A local AOSP 12 `mkbootimg`
  simulation with the stock kernel/ramdisk and this DTB produced a
  27,183,104-byte unsigned boot image, leaving 6,371,328 bytes of the 32 MiB
  boot partition for the AVB footer and recovery-ramdisk growth.
* The vendor partition contains the matching kernel modules
  (`/vendor/lib/modules`, 23 files).  They are installed from
  `vendor/infinix/X663/proprietary` into the source-built vendor partition;
  no `vendor.img` is used.
* Every file in this tree was re-verified against the stock SP Flash Tool
  package: `Image.gz` and `dtb/mt6768.dtb` are byte-identical to the
  components `unpack_bootimg` extracts from `boot.img`, and `dtbo.img` is
  byte-identical to the first 49,472 bytes of the package's 8 MiB `dtbo.img`.
  The package contains no kernel source and no kernel configuration blob, so
  `defconfig` remains the only artefact that cannot be verified against it.
* The stock kernel image itself is gzip-compressed and expands to a
  25,264,128-byte raw kernel.

## Licensing

The extracted `defconfig` is licensed under **GPL-2.0-only**; see `LICENSE`
and `NOTICE`.  The prebuilt `Image.gz`, `dtb/mt6768.dtb` and `dtbo.img` are
stock firmware binaries and are not relicensed by this tree.
