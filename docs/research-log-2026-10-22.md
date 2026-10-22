# PRX126 research log — 2026-10-22

## Verified

- 8311 v2.8.3 Basic distributes the Potrontec `PTXG_CX_V0.03` MIPS LZMA uImage, dated July 23, 2024 in its U-Boot header.
- The decompressed kernel contains one valid FDT at offset `0x742630`, measuring 32647 bytes.
- SHA-256 of the embedded FDT and the independently captured running `/sys/firmware/fdt` are identical: `c5ee709e0b1db5a16dc3474f3f4a54a62943720b6963f1cef257b54f378bbd65`.
- The original boot FDT can be decompiled and rebuilt byte-identically with `dtc`.
- Six vendor modules share 264 matching imported-symbol CRC comparisons with no mismatches across 15 module pairs. This confirms internal expected-ABI consistency only.

## Still unknown

- Location of the original editable `prx126-sfp-pon.dts` and related include hierarchy.
- Complete, newer PRX126-capable kernel build tree and SDK.
- Whether a rebuilt kernel could support all original PON hardware and firmware services.

## Research method

Keep firmware, binaries, captured FDT/DTS, configuration and unique device information private. Publish only non-identifying technical findings. No flashing or writes to the live ONU.
