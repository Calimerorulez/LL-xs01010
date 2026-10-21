# NAND / UBI layout

Observed read-only via `/proc/mtd`, `ubinfo -a`, and `mount` on 2026-10-21. Hexadecimal lengths below are observed partition/volume device lengths. **UBI volume device entries are not independent physical NAND partitions.**

## Physical MTD partitions

| Device | Size (hex) | Erase size (hex) | Name |
|---|---|---|---|
| mtd0 | 00100000 | 00020000 | uboot |
| mtd1 | 00040000 | 00020000 | ubootconfigA |
| mtd2 | 00040000 | 00020000 | ubootconfigB |
| mtd3 | 00040000 | 00020000 | gphyfirmware |
| mtd4 | 00100000 | 00020000 | calibration |
| mtd5 | 01000000 | 00020000 | bootcore |
| mtd6 | 06c00000 | 00020000 | system_sw |
| mtd7 | 00140000 | 00020000 | res |

The additional mtd8–mtd15 entries describe UBI volumes exposed by the platform. Their erase size is reported as `0001f000` (124 KiB), the UBI logical eraseblock size.

## UBI device 0 (on system_sw)

- Total: 864 logical eraseblocks; 104.6 MiB usable logical space
- Available: 307 logical eraseblocks; ~37.1 MiB
- LEB size: 126976 bytes
- Minimum I/O unit: 2048 bytes
- Bad physical eraseblocks: 0 at observation time
- Reserved physical eraseblocks: 20

| Volume ID | Name | LEBs | Bytes | Approx. MiB |
|---|---|---:|---:|---:|
| 0 | kernelB | 18 | 2,285,568 | 2.18 |
| 1 | bootcoreB | 43 | 5,459,968 | 5.21 |
| 2 | rootfsB | 49 | 6,221,824 | 5.93 |
| 3 | kernelA | 18 | 2,285,568 | 2.18 |
| 4 | bootcoreA | 43 | 5,459,968 | 5.21 |
| 5 | rootfsA | 49 | 6,221,824 | 5.93 |
| 6 | rootfs_data | 265 | 33,648,640 | 32.09 |
| 7 | ptconf | 48 | 6,094,848 | 5.81 |

Mounts:
- `/rom`: read-only SquashFS
- `/overlay`: UBIFS volume `ubi0_6`
- `/`: overlayfs
- `/ptconf`: UBIFS volume `ubi0:ptconf`

## Safety

Do not write any MTD/UBI partition before obtaining private, verified backups and a tested recovery procedure. In particular, `uboot`, `ubootconfigA/B`, `calibration`, `ptconf`, and `res` may contain device-specific or operational data. A/B existence does not guarantee rollback.
