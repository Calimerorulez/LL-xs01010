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
