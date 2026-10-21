# Luleey LL-XS1010 — firmware research

Public, **sanitized** technical notebook for the Luleey LL-XS1010 XGS-PON ONU SFP+ (8311 Community Edition).

**Status:** Hardware inventory only. No custom image has been built or tested. Initial inventory was collected on 2026-10-21 with no optical fiber connected.

## Confirmed device profile

| Property | Observed |
|---|---|
| Board | `PRX126-SFP-PON` |
| SoC | PRX300 rev 1.3 |
| CPU | MIPS interAptiv, MIPS32r2 |
| RAM | ~1 GiB |
| Kernel | `4.9.308+` |
| Vendor OS | LEDE `v19.07.8_maxlinear`, target `intel_mips/prx300` |
| Firmware | 8311 Community MOD `v2.8.3_basic_7d89440` |
| Bootloader | `U-Boot 2016.07-MXL-v-3.1.261` |
| Storage | NAND + UBI, A/B firmware volumes |
| Booted rootfs | `rootfsA` (when observed) |

## Documentation

- [Hardware and boot environment](docs/hardware.md)
- [Flash / UBI partition layout](docs/flash-layout.md)
- [Kernel modules and software inventory](docs/software-inventory.md)
- [Research roadmap and FS.com questions](docs/research-roadmap.md)
- [Safety and publication policy](SECURITY.md)

## Objective

Build a reproducible understanding of this device's hardware, boot process and MaxLinear dependencies before considering newer firmware. Investigate possible compatibility with the [8311 firmware builder](https://github.com/djGrrr/8311-was-110-firmware-builder) and with documentation from FS.com when available.

## Security boundary

**Do not upload PON identifiers, MAC addresses, credentials, bootloader/environment dumps, calibration/EEPROM contents or raw firmware/configuration dumps.** Only sanitized technical summaries belong in this public repository.

The existence of A/B firmware banks **does not** prove a safe fallback mechanism. No Odido PON registration or replacement has been tested by this inventory.
