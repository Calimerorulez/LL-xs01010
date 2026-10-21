# Hardware and boot environment

Observed 2026-10-21 on Luleey LL-XS1010, without fiber attached.

| Field | Value |
|---|---|
| CPU system type | PRX300 rev 1.3 |
| Board/machine | PRX126-SFP-PON |
| CPU | MIPS interAptiv (multi) V2.0, MIPS32r2 |
| RAM | 998676 KiB reported |
| Kernel | Linux 4.9.308+, build 2024-07-10 |
| OpenWrt/LEDE | PTXG_CX_V0.03, v19.07.8_maxlinear |
| Target | intel_mips/prx300 |
| Architecture | mips_24kc_nomips16 |
| 8311 firmware | v2.8.3 basic, revision 7d89440 |
| U-Boot | 2016.07-MXL-v-3.1.261, built 2024-12-24 |
| Active rootfs | rootfsA (boot argument at observation time) |

Device-tree model: `PRX126-SFP-PON`.

Device-tree compatible strings:
- `intel,prx126-sfp-pon`
- `intel,prx126-sfp-eva-pon`
- `intel,prx126`
- `intel,prx300`

Boot parameters (sanitized, selected):
- `ubi.mtd=system_sw,2048`
- `rootfsname=rootfsA`
- `rootfstype=squashfs`
- `machtype=PRX126_SFP_EVA_PON`
- `cpuclk=400000000`
- `mem=1024M@512M`

The original boot command line contains unique Ethernet identifiers and network arguments and should not be published unredacted.

## Notes
- Running rootfs A does **not** establish that fallback to rootfs B works.
- No claim of Odido optical registration or service compatibility is made.
