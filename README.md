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

The `.dtb` file is a `dt_table` container (not a raw FDT): the single entry
points at an FDT with the usual `0xd00dfeed` magic.  This is exactly what the
stock boot image carries in its DTB region, so `BOARD_PREBUILT_DTBIMAGE_DIR`
+ `cat *.dtb` reproduces the 119,464-byte blob byte for byte.

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

## Licensing

The extracted `defconfig` is licensed under **GPL-2.0-only**; see `LICENSE`
and `NOTICE`.  The prebuilt `Image.gz`, `dtb/mt6768.dtb` and `dtbo.img` are
stock firmware binaries and are not relicensed by this tree.
