# kernel/infinix/X663

Prebuilt kernel for the Infinix Note 11 (X663), MediaTek MT6768 / Helio G88,
from the stock firmware. No kernel source is published, so this tree ships
the stock binaries.

| File | Contents |
| --- | --- |
| `Image.gz` | Stock kernel, `4.19.191-g20c2dd72f5fd-dirty` (10,855,427 B) |
| `dtb/mt6768.dtb` | MTK `dt_table` container from the stock boot image (119,464 B; FDT at offset 64, `0xd00dfeed`) |
| `dtbo.img` | Unsigned DTBO table (49,472 B); the build pads and signs it to the 8 MiB partition |
| `defconfig` | Stock kernel config, GPL-2.0-only |

## File names vs the unpacked boot image

`unpack_bootimg` yields `kernel`, `dtb` and `ramdisk`; this tree uses the
Android build-system names. The bytes are identical:

| unpack_bootimg | This tree | Bytes | SHA-256 |
| --- | --- | ---: | --- |
| `kernel` | `Image.gz` | 10,855,427 | `2a9dc96352c99a9462935f0024b53696a3f087320596a412125a41a997be72c5` |
| `dtb` | `dtb/mt6768.dtb` | 119,464 | `6265c98f0439f053e1b27358d937b9e432f2f38fad93fcae8b2846861a750815` |

`Image.gz` is required by `BOARD_KERNEL_IMAGE_NAME`; the DTB directory is
concatenated by `BOARD_PREBUILT_DTBIMAGE_DIR`.

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
