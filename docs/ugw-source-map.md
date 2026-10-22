# UGW 8.4.2 PRX300 source map and compatibility boundaries

Research updated 2026-10-21. All paths below are public, source-level research leads. **This is not a validated LL-XS1010 build or vendor SDK match.**

## Verified public source tree

The prpl Foundation GitLab repository [intel/linux, UGW 8.4.2 branch, linux_patches](https://gitlab.com/prpl-foundation/intel/linux/-/tree/ugw_8.4.2/linux_patches) contains 2020-era vendor kernel patches for Lantiq/Intel/PRX300. The branch release is identified as UGW 8.4.2.60, dated July 2020. Its underlying kernel branch reference includes `intel-4.9.206`, whereas the LL-XS1010 reports Linux `4.9.308+`, compiled July 2024. These **are not identical kernel builds**.

### Priority patch files from directory listing

| Filename | Suspected relevance | Evidence strength |
| --- | --- | --- |
| `0001-Add-support-for-lantiq-mips-dts-files.patch` | Historical board/device-tree sources | Exact filename seen; individual matching PRX126 board not yet verified |
| `0016-Add-support-for-lantiq-datapath-driver-common.patch` | Datapath API, potential `dp_*` implementations | Filename seen; exact functions **not yet verified** |
| `0025-Add-support-for-lantiq-ppv4-driver.patch` | PPv4 packet processor | Filename seen |
| `0043-Add-support-for-lantiq-defconfig-files.patch` | Historical kernel configuration options | Filename seen |
| `0078-Add-support-for-directconnect-driver.patch` | Direct-connect datapath support | Filename seen |
| `0081-Add-support-for-intel-ppv4-qos-driver.patch` | QoS control and datapath integration | Filename seen |
| `0301-DRVLIB-SW-919-datapath-LCT-address-review-comments.patch` | LCT management interface behavior | Filename seen |
| `0440-Rename-falconmx-to-prx300-for-combo-phy-wan-xpcs-and.patch` | PRX300 SoC transition | Filename seen |
| `0610-Insertion-support-in-DP.patch` | Datapath insertion API | Filename seen |

Source: [UGW 8.4.2 patch directory](https://gitlab.com/prpl-foundation/intel/linux/-/tree/ugw_8.4.2/linux_patches).

An individually opened patch, [0368-PONRTSYS-2772](https://gitlab.com/prpl-foundation/intel/linux/-/blob/ugw_8.4.2/linux_patches/0368-PONRTSYS-2772-Add-pondef-as-parent-of-g-pon.patch), changes `drivers/clk/intel/clk-prx300.c` by making `g_pon` use the `pondef` parent. **That file path and modification are directly verified.** The other candidate files above are identified by filenames only; specific function implementations have not yet been inspected.

## Current runtime evidence from the LL-XS1010

Private inspection of `mod_pon_eth.ko` shows the unresolved external functions `dp_alloc_port_ext`, `dp_register_dev_ext`, and `dp_xmit`. Private inspection of `gwdpa-dpm.ko` shows local `dp_alloc_port_ext2`, `dp_register_dev_ext2`, and `dp_xmit2`, while the unsuffixed names remain undefined references.

Live `/proc/kallsyms` on the LL-XS1010 lists `dp_alloc_port_ext`, `dp_register_dev_ext`, and `dp_xmit` as text symbols **without module suffixes**. This is consistent with built-in vendor-kernel definitions, but we have not yet traced them to individual source files or verified matching exported-symbol CRCs.

**Open research issue:** match these exact functions against `0016` and other datapath patch contents. The public patch directory alone does not prove the functions are present.

## Board layout warning: older WAS-110 example differs

An independently documented WAS-110 PRX126 boot command line shows `system_sw=102 MiB` plus a separate `ptdata=6 MiB` partition. On the observed LL-XS1010, `system_sw=108 MiB` with `ptconf` as a UBI volume (no matching separate `ptdata` partition in the eight physical MTD entries). See [WAS-110 on PON.wiki](https://pon.wiki/xgs-pon/ont/bfw-solutions/was-110/).

This is concrete evidence that **nominally related PRX126 devices can use different physical flash layouts**. It prohibits assuming firmware images and recovery procedures are interchangeable.

## Safe next research work

1. Examine the contents of the public `0016`, `0001`, `0043` patch files and identify source paths and function implementations, recording exact repository links.
2. Identify whether UGW 8.4.2 includes specific `PRX126-SFP-PON` DTS/DTSI definitions, not merely generic PRX300.
3. Look for newer published UGW releases matching Linux 4.9.308+; a 2020 UGW 8.4.2 tree is only an architectural reference.
4. Obtain FS.com firmware kernel and SDK sources before proposing a kernel replacement.
5. Keep raw module binaries, PON identifiers and full boot logs private.

## Conclusion

Strong public evidence of historical PRX300 kernel/datapath patch availability. No verified full LL-XS1010 SDK, no verified matching `dp_*` source location, and no basis for a safe cross-device firmware flash.


## Confirmed implementation discovered in MaxLinear's own public Linux tree (2026-10-21)

**Major finding:** the publicly readable [`maxlinear/linux` repository](https://github.com/maxlinear/linux), default branch `updk_9.1.90`, has the *actual definitions* of all three APIs originally observed as unresolved references in LL-XS1010 modules:

- [`drivers/net/datapath/dpm/datapath_api_fn.c`](https://github.com/maxlinear/linux/blob/updk_9.1.90/drivers/net/datapath/dpm/datapath_api_fn.c): `dp_alloc_port_ext`, `dp_register_dev_ext`, `dp_xmit` are each defined and exported with `EXPORT_SYMBOL`.
- [`include/net/datapath_api_fn.h`](https://github.com/maxlinear/linux/blob/updk_9.1.90/include/net/datapath_api_fn.h): declares `dp_alloc_port_ext_fn`, `dp_register_dev_ext_fn`, `dp_xmit_fn` as function pointers.
- [`include/net/datapath_api.h`](https://github.com/maxlinear/linux/blob/updk_9.1.90/include/net/datapath_api.h): public function declarations / datapath API types.

The functions in the newer tree are wrappers: they dispatch through matching `*_fn` function pointers if installed. The `dp_xmit` wrapper releases the packet buffer and returns `DP_FAILURE` if no handler is registered. This is **direct source confirmation of the API/dispatch design** hypothesized from the LL-XS1010's ELF undefined symbols and the local `*2` implementations in `gwdpa-dpm.ko`.

**Compatibility limit:** the MaxLinear default-branch [kernel Makefile](https://github.com/maxlinear/linux/blob/updk_9.1.90/Makefile) identifies **Linux 5.15.180**, whereas the LL-XS1010 is running **Linux 4.9.308+**. The source confirms shared design and naming; it does *not* establish the same ABI, matching patches, PRX126 board support, compatible PON drivers, or a safe upgrade path.

### Refined research question

The next meaningful comparison is no longer simply whether `dp_*` exists. It is whether the `struct dp_*` layouts, exported-symbol CRCs, dispatcher registration, PON mailbox firmware, and PRX126 board hardware configuration match or can be rebuilt for a newer SDK. Function names alone are insufficient.

### Candidate follow-up source checks

1. Check exact `maxlinear/linux` branch support for PRX300/PRX126 device trees and whether the LL-XS1010 board identifier occurs.
2. Search `datapath_api_fn.c` and related DPM source for assignment of the `*_fn` hooks; compare with local `gwdpa-dpm` symbol findings.
3. Look for other public MaxLinear branches closer to `4.9.308+` and document kernel baseline.
4. Keep firmware-bank, image format, chip and board differences in scope; do not flash vendor binaries from other products.

This note contains only upstream source references, generic versions and sanitized symbol names.


## Verified PRX300 build integration in current public MaxLinear tree

Targeted public code search of `maxlinear/linux` branch `updk_9.1.90` also confirms:

- [`drivers/net/datapath/dpm/Kconfig`](https://github.com/maxlinear/linux/blob/updk_9.1.90/drivers/net/datapath/dpm/Kconfig) explicitly depends on `PRX300_CQM || LGM_CQM` for datapath-related configuration.
- [`drivers/net/ethernet/mxl/Kconfig`](https://github.com/maxlinear/linux/blob/updk_9.1.90/drivers/net/ethernet/mxl/Kconfig) identifies Ethernet support for GRX500/PRX300/LGM.
- [`drivers/net/datapath/Kconfig`](https://github.com/maxlinear/linux/blob/updk_9.1.90/drivers/net/datapath/Kconfig) mentions PRX300 as an NP platform.
- [`drivers/net/datapath/dpm/datapath_api_fn.c`](https://github.com/maxlinear/linux/blob/updk_9.1.90/drivers/net/datapath/dpm/datapath_api_fn.c) declares/exports `dp_xmit_fn` initialized to NULL, reinforcing the dynamic-dispatch interpretation.

A search for the exact board string `prx126-sfp-pon` in the default-branch indexed files returned no matches; this is **not** a comprehensive proof the device tree is absent. The branch contains PRX300 functionality, but no verified complete LL-XS1010 platform definition.

## 8311 builder kernel provenance (2026-10-22)

Audited upstream `djGrrr/8311-was-110-firmware-builder` scripts (`build.sh`, `extract.sh`, `mods/binary-mods.sh`) and its v2.8.0 release notes. The builder requires two vendor firmware inputs: an Azores/BFW local-upgrade `.img`, from which `extract.sh` extracts `bootcore.bin`, `kernel.bin`, and `rootfs.img`, and a separate Potrontec/basic image directory already containing the same three artifacts. The `-k`/`-K` selection chooses which supplied kernel image to carry into the output; `build.sh` copies that `kernel.bin` and repackages images. It does not compile Linux or produce a DTS. `mods/binary-mods.sh` modifies selected user-space PON/OMCI binaries, not the compiled kernel. The 8311 v2.8.0 release states that its basic base was updated to Potrontec `PTXG_CX_V0.03`. This identifies the immediate binary provenance but not a public source SDK or source DTS.

Source: https://github.com/djGrrr/8311-was-110-firmware-builder/blob/master/build.sh ; https://github.com/djGrrr/8311-was-110-firmware-builder/blob/master/extract.sh ; https://github.com/djGrrr/8311-was-110-firmware-builder/releases/tag/v2.8.0

An indexed code search for literal `intel,prx126-sfp-pon` yielded no source DTS; the historical `paldier/feed_target_mips/image/prx300.mk` instead specifies `DEVICE_DTS := prx126-sfp-pon` as a target name without including the matching source DTS in the checked-out tree. Exact DTS source remains unlocated. The live device-tree can furnish an effective DTB for a private sanitized decompilation, but that is not identical to locating the original editable vendor DTS source and could expose unique calibration/identity data if published unfiltered.

## Identified 8311 v2.8.3 Basic kernel binary (2026-10-22)

The 8311 v2.8.3 Basic release archive contains `kernel.bin` (2,274,800 bytes). `file`, `dumpimage -l` and `binwalk` independently identify it as a **legacy U-Boot uImage** named `PTXG_CX_V0.03`: MIPS Linux kernel, LZMA-compressed, with a 64-byte uImage header, 2,274,736-byte payload, kernel image timestamp 2024-07-23 05:10:15 UTC (local `dumpimage` display 07:10:15), and load/entry address `0xA0020000`. Its SHA-256 is `d66b24cf873cc1071a3aa2d155bf677f503f10d221513e469083f8a591f6b96c`. The LZMA header reports uncompressed size 7,644,599 bytes. An FDT-magic scan of the **compressed** uImage found no candidates; this does not exclude an FDT embedded in the decompressed payload. Next extract the uImage payload, decompress the LZMA stream offline, and scan/validate prospective embedded FDTs. These are public firmware image-format observations, not a provenance proof of an exact source SDK commit.
