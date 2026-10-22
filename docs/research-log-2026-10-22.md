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

## New source-chain leads

- The public prplOS `mxl_x86.yml` profile identifies MaxLinear UGW feeds (including GWDPA DPM, PON, PP network adapter and QoS) using `git.maxlinear.com` SSH URLs, with pinned revisions. Those repositories may require vendor credentials and are not assumed publicly downloadable.
- A publicly documented FS PRX126 SFP+ stick runs `v22.03.3_maxlinear` firmware, demonstrating that a newer MaxLinear userland/kernel distribution exists for this SoC family. The kernel release and exact source/DTS availability still need verification. This does not establish compatibility with the LL-XS1010 board.

Sources: https://gitlab.com/prpl-foundation/prplos/prplos/-/blob/a17afef7ca0ef6e21674e91284a6b096f136b9ce/profiles/mxl_x86.yml and https://gist.github.com/arapov/a2019f5dac903864f36102184ff2755a

## MaxLinear 22.03.3 on another PRX126 (public-source verification)

A July 2026 field report identifies the FS.com `XGS-SFP-ONT-MAC-I` (SKU 378865, PRX126) as running `LEDE 1.23.1` / `v22.03.3_maxlinear` and documents working PON/OMCI behavior. Its public write-up does **not** provide `uname -r`, `/proc/version`, a kernel boot banner, a firmware image, or a source SDK revision. Consequently **22.03.3 is a distribution version, not proof of a Linux 5.15 kernel**. Independent contemporary FS.com documentation confirms vendor firmware management on this model but does not establish a publicly downloadable image or source code. The vendor's reported compatibility statements also discourage treating 8311 firmware as a drop-in replacement for the FS stick.

Sources:
- https://gist.github.com/arapov/a2019f5dac903864f36102184ff2755a
- https://resource.fs.com/mall/resource/xgs-sfp-ont-mac-i-configuration-guide-20260114141439.pdf

Next evidence required: actual `uname -r`, `uname -v`, `cat /etc/openwrt_release`, and (if permissible) the firmware image or kernel header from this specific FS model. Do not flash an unknown manufacturer's image on the LL-XS1010. The older Potrontec kernel and its byte-identical boot FDT remain the verified baseline.
