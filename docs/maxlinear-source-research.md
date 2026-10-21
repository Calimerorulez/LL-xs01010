# Public PRX300 / MaxLinear source research

Research date: 2026-10-21. This page contains **public reference pointers and analysis only**; no proprietary binaries, device identifiers or extracted firmware.

## Key findings

### 1. Historical Intel / MaxLinear UGW PRX300 kernel patch set (confirmed public)

The prpl Foundation GitLab project **prpl-foundation/intel/linux**, branch `ugw_8.4.2`, exposes a `linux_patches` directory with PRX300-related work.

- Source tree: https://gitlab.com/prpl-foundation/intel/linux/-/tree/ugw_8.4.2/linux_patches
- Representative clock/PON patch: https://gitlab.com/prpl-foundation/intel/linux/-/blob/ugw_8.4.2/linux_patches/0368-PONRTSYS-2772-Add-pondef-as-parent-of-g-pon.patch
- PRX300 bootcore notification patch: https://gitlab.com/prpl-foundation/intel/linux/-/blob/ugw_8.4.2/linux_patches/0642-DRVLIB-SW-2354-PRX300-bootcore-up-notification.patch
- PRX300 GPIO compile patch: https://gitlab.com/prpl-foundation/intel/linux/-/blob/ugw_8.4.2/linux_patches/0677-DRVLIB-SW-2520-Fix-PRX300-GPIO-driver-compiler-warni.patch

The directory includes patches referencing PON, datapath (DP), QoS, bootcore, switch and PRX300 clock drivers. The seen branch reflects 2020-era UGW 8.4.2 release material, **not** the exact observed 2024 LL-XS1010 vendor kernel build. Source completeness, licensing, matching toolchain and availability of the complete PON driver source remain unverified.

### 2. Same PON IP firmware version observed elsewhere (confirmed)

The LL-XS1010 boot log reported PON mailbox firmware version `3.21.0.3.16-1674463172`. This string also appears in documented WAS-110 status output and third-party 8311 firmware instructions:

- https://www.fibermall.com/file/datasheet/xgspon-onu-stick-8311-firmware-version-user--guide.pdf
- https://www.ispreview.co.uk/talk/threads/virgin-media-xgs-pon-fttp-install-2024-video.42154/

This is evidence of a **shared PON IP firmware component version**, not image-level or hardware compatibility.

### 3. FS.com PRX126 modernized vendor baseline (third-party field report)

A July 2026 field report describes the FS.com XGS-SFP-ONT-MAC-I (SKU 378865), PRX126 based, running `v22.03.3_maxlinear` / LEDE 1.23.1:

- https://gist.github.com/arapov/a2019f5dac903864f36102184ff2755a

It describes a vendor `pon_net_lib` / OMCI data path and an observed bridge issue on O2/CETIN. **Those operator-specific settings do not apply automatically to Odido.** The report does not establish the exact running Linux kernel, SDK source availability, boot compatibility or LL-XS1010 image interchangeability. Independently verify against FS.com documentation when supplied.

### 4. Mainline OpenWrt compatibility caveat (confirmed upstream)

OpenWrt 22.03 introduced 64-bit `time_t` for 32-bit musl, changing the userspace ABI. Its standard releases also use newer Linux kernels and firewall4/nftables. A stock 19.07-to-22.03 `sysupgrade` is not supported:

- https://openwrt.org/releases/22.03/notes-22.03.0

The LL-XS1010 ships a vendor-modified LEDE/OpenWrt 19.07.8 MIPS soft-float system; its existing `omcid` and vendor libraries must **not** be assumed binary-compatible with standard 22.03 or the FS.com vendor 22.03 branch.

## Practical implications / research ranking

1. **Best confirmed public source lead:** inspect UGW 8.4.2 PRX300 kernel patches, identify whether `dp_alloc_port_ext`, `dp_register_dev_ext` and `dp_xmit` implementations or their headers appear, and map relevant PON drivers/bootcore.
2. **Most informative external lead:** obtain FS.com kernel version, SDK release, matching GPL source package, patches, toolchain, device tree and binary ABI details.
3. **Possible interim engineering direction:** keep the working 4.9.308+ vendor kernel and investigate a carefully selected compatible userspace, but avoid claiming stock OpenWrt compatibility.
4. **Longer-term:** only evaluate modern kernel builds when platform datapath and PON driver sources plus recovery path have been established.

## Gaps / open questions

- Which exact MaxLinear SDK/UGW branch corresponds to the LL-XS1010 vendor kernel compiled July 2024?
- Do available public patch trees contain complete PRX126 `mod_pon_*`, `gwdpa-dpm`, QoS and OMCI component source, or only patches/reference code?
- Where are the built-in `dp_* ` exports defined, and which kernel ABI/CRC revisions are required?
- Is FS.com's `v22.03.3_maxlinear` based on a 4.9-derived or newer kernel?
- What is the FS.com `omcid` ABI, libc version, OpenSSL version and `libubus` version?
- Are firmware bank structures, device trees and rescue paths equivalent? **Currently unknown.**

## Publication boundary

Do not upload any module binaries, full MTD/UBI dumps, PON serials, ONU identities, MAC addresses, calibration/EEPROM data, credentials or unredacted environment/log dumps. This document uses only generic technical facts and public URLs.
