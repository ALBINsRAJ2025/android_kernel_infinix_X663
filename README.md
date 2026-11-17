# kernel/infinix/X663

Prebuilt kernel for the Infinix Note 11 (X663), MediaTek MT6768 / Helio G88,
from the stock firmware. No kernel source is published, so this tree ships
the stock binaries.

| File | Contents |
| --- | --- |
| `Image.gz` | Stock kernel, `4.19.191-g20c2dd72f5fd-dirty` (10,855,427 B) |
| `dtb/mt6768.dtb` | Raw FDT from the stock boot image (119,400 B); the boot field is a 119,464 B `dt_table` around it |
| `dtbo.img` | Unsigned DTBO table (49,472 B); the build pads and signs it to the 8 MiB partition |
| `defconfig` | Stock kernel config, GPL-2.0-only |

## File names vs the unpacked boot image

`unpack_bootimg` yields `kernel`, `dtb` and `ramdisk`; this tree uses the
Android build-system names. The `dtb` field is a `dt_table` container
(32 header + 32 entry bytes) whose single entry is the FDT kept here:

| unpack_bootimg | This tree | Bytes | SHA-256 |
| --- | --- | ---: | --- |
| `kernel` | `Image.gz` | 10,855,427 | `2a9dc96352c99a9462935f0024b53696a3f087320596a412125a41a997be72c5` |
| `dtb` entry 0 | `dtb/mt6768.dtb` | 119,400 | `5dbaf472e07ee93c527404a4c5d8964f17a2f639d196ccf81eb7729f007d2d2c` |

`Image.gz` is required by `BOARD_KERNEL_IMAGE_NAME`; the DTB directory is
turned back into a table by `BOARD_PREBUILT_DTBIMAGE_DIR`.

## Building

```make
TARGET_PREBUILT_KERNEL      := kernel/infinix/X663/Image.gz
BOARD_PREBUILT_DTBIMAGE_DIR := kernel/infinix/X663/dtb
BOARD_INCLUDE_DTB_IN_BOOTIMG := true
BOARD_KERNEL_SEPARATED_DTBO  := true
BOARD_PREBUILT_DTBOIMAGE     := kernel/infinix/X663/dtbo.img
```

Boot header values from the stock `boot.img` (v2, page 2048): base
`0x40078000`, kernel offset `0x8000`, ramdisk offset `0x07c08000`, tags/DTB
offset `0x0bc08000`.

## Licensing

`defconfig` is GPL-2.0-only. `Image.gz`, the DTB and `dtbo.img` are stock
firmware binaries, not relicensed; see `LICENSE` and `NOTICE`.
