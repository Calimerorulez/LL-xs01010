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

## FS XGS-SFP-ONT-MAC-I 22.03.3 firmware acquisition search

Public web searches for an original downloadable `v22.03.3_maxlinear` image for FS XGS-SFP-ONT-MAC-I (PRX126; field report SKU 378865) found **no verifiable direct firmware archive or mirror**. The documented FS upgrade interface accepts user-supplied firmware files, but the existence of an upload interface is not evidence of a public download. FS.com Q&A quoted in third-party community reports says its XGS-PON ONU stick is not intended for post-shipping firmware updates and that community 8311 firmware is not compatible; this should be confirmed with support for the exact product/SKU and firmware revision. Other products labelled `XGS-ONU-25-20NI` or FS GPON sticks may use different hardware and must not be used as firmware substitutes.

Best next route: request the exact firmware package and its build/kernel version, release notes and compatibility details from FS.com support, referring to `XGS-SFP-ONT-MAC-I`, the specific SKU and installed `v22.03.3_maxlinear` (LEDE 1.23.1). Alternatively contact the author of the public July 2026 PRX126 field report to request non-sensitive `uname -a` output and the source of the firmware package. Do not flash any unverified image on the LL-XS1010.

Sources: https://gist.github.com/arapov/a2019f5dac903864f36102184ff2755a ; https://resource.fs.com/mall/resource/xgs-sfp-ont-mac-i-configuration-guide-20260114141439.pdf ; https://community.odido.nl/thuisnetwerk-566/tragere-uploadsnelheid-na-eigen-ont-sfp-registratie-382970

## Broader MaxLinear firmware discovery (2026-10-22)

Search expanded beyond the FS branded stick to public MaxLinear PRX126/PRX300 firmware images, MaxLinear UGW download locations, GitHub/GitLab firmware mirrors, and prplOS release artifacts.

**Findings:** (1) No verified direct downloadable `v22.03.3_maxlinear` firmware archive for the PRX126 FS XGS-SFP-ONT-MAC-I, or a matching generic PRX126 MaxLinear release, was located in the searched public sources. (2) prpl Foundation release pipelines do publish MaxLinear/Gemtek OSPv2 images, but these target **intel_x86/lgm** equipment, not the MIPS PRX126 SFP stick; such images are not substitutes. (3) The prplOS MaxLinear SDK profile explicitly says that its source feeds require access to `git.maxlinear.com`; the published profile targets x86 URX, not PRX126. (4) The public 8311-related PRX126 firmware builds (including Chinese community variants) are independent community image repacks, not verified MaxLinear 22.03.3 originals. (5) Official MaxLinear public documentation search did not turn up a PRX126 system image download; the vendor has a support portal for requesting technical material.

References:
- https://gitlab.com/prpl-foundation/prplos/prplos/-/blob/mainline-23.05/.gitlab/release.yml
- https://gitlab.com/prpl-foundation/prplos/prplos/-/blob/a17afef7ca0ef6e21674e91284a6b096f136b9ce/profiles/mxl_x86.yml
- https://maxlinear-support.microsoftcrmportals.com/
- https://www.right.com.cn/forum/thread-8423574-1-1.html

Next research priority: obtain original FS/MaxLinear PRX126 image or GPL source package, or verifiable kernel `uname` output from a 22.03.3 PRX126 owner. The known `PTXG_CX_V0.03` 8311 Basic uImage remains the only verified local firmware baseline for this exact board. No firmware flashing is planned.

## Comparison of historical PRX300 and public Linux 5.15 Kconfig

Read-only Debian searches confirmed `SOC_GRX500`, `SOC_PRX300_BOOTCORE` and `PRX300_CQM` declarations in historical Intel Linux 4.9 source. The public MaxLinear 5.15.180 checkout's `drivers/net/datapath/cqm/Kconfig` declares `MXL_CBM` and only offers `LGM_CQM` in its SoC selection. However, its datapath DPM Kconfig still refers to `PRX300_CQM` in `DPM_DATAPATH_PARTIAL_FEEDS` and `DPM_DATAPATH_HAL_GSWIP31` dependencies. These references alone do not provide an enabled PRX300 backend; the necessary configuration definition was not found in the searched modern tree. This points to an incomplete PRX300 platform integration in the published 5.15 checkout, not proof that no newer proprietary branch exists.

Verified source paths: `paldier/linux/drivers/net/ethernet/lantiq/cqm/Kconfig` on `intel-4.9.206`, `maxlinear/linux/drivers/net/datapath/cqm/Kconfig` and `drivers/net/datapath/dpm/Kconfig`. Next seek an intermediate UGW / PRX300-capable kernel tree before attempting an unsupported cross-version port.

## PRX300 CQM implementation inventory (2026-10-22)

Read-only source inventories confirmed that public MaxLinear Linux 5.15.180 `drivers/net/datapath/cqm/` contains common wrappers plus only a concrete `lgm/` implementation. No filenames containing `prx300` or `prx126` were found anywhere in that checkout. In contrast, historical Intel Linux 4.9.206 includes a `drivers/net/ethernet/lantiq/cqm/prx300/` implementation (`cqm.c`, configuration and debugfs files, Kconfig, Makefile and hardware register headers). These observations reinforce that the modern public tree does not contain a directly selectable PRX300 CQM implementation. They do not demonstrate that the historical driver will compile with Linux 5.15, nor exclude vendor-only sources.

Next inspect the two CQM platform implementations side-by-side (source-level overlap, control structure and register differences) and the 5.15 CQM Makefile/headers to scope a potential port. Do not attempt live kernel changes.

## PRX300 versus LGM CQM source structure (2026-10-22)

The historical PRX300 CQM implementation (`intel-linux-4.9/drivers/net/ethernet/lantiq/cqm/prx300`) has 4444 lines in `cqm.c` and 218 in `cqm_config.c`. The public MaxLinear Linux 5.15 LGM implementation (`drivers/net/datapath/cqm/lgm`) has 11199 and 955 lines respectively. Device-tree bindings are distinct: `intel,prx300-cqm` for PRX300, versus `mxl,lgm-cqm`, `mxl,lgmc-cqm`, and `mxl,lgm-cqm-urx651` for LGM variants. Their `reg/` folders share seven header *names*: `cbm_ls.h`, `cqem_deq.h`, `cqem_desc64b.h`, `cqem_enq.h`, `cqem.h`, `cqem_pon_ip_if.h`, `fsqm.h`. Name overlap is not evidence that registers, structures, or values are compatible. Neither source inventory nor line counts establish porting feasibility; source-level comparisons are required. Continue offline, with no image flashing.

## Codex Sol 6.1 offline assessment — preliminary status

User-provided completion screenshot reports that four local analyses were produced under `/root/prx126-offline-research/reports/`: `prx300-cqm-comparison.md`, `prx300-kernel-api-gap.md`, `prx126-driver-dependencies.md`, and `prx126-linux515-feasibility.md`. Sol describes a 4.9-to-5.15 port as technically plausible but subject to MIPS platform integration, DMA changes and correct SKB-buffer return. Missing current DPM core/HAL blocks the modern datapath route, and firmware compatibility for hardware revision 1.3 remains unconfirmed. Its recommended first milestone is a minimal kernel without datapath. **This entry records the tool's summary only; the four full reports have not yet been independently reviewed.** Codex reports no source edits, builds, downloads, device access, or GitHub operations. Next: obtain and review the four Markdown reports before deciding the build plan.

## Review of all four Codex portability reports (2026-10-22)

The four complete local reports were supplied for review: `prx300-cqm-comparison.md`, `prx300-kernel-api-gap.md`, `prx126-driver-dependencies.md`, `prx126-linux515-feasibility.md`. This review relies on their documented local source examination, not an independent source rebuild.

Key points: PRX300/LGM share seven register-header filenames but differ substantially in interrupt, enqueue/dequeue port mappings and hardware ownership; an LGM CQM drop-in is unsupported. Historic PRX300 CQM, HDMA, BM/QoS and GSWIP31 DPM sources exist. The newer 5.15 checkout has some PRX300-aware PHY, timer, switch, PP and PON support, but lacks a selectable PRX300 MIPS boot platform, the PRX300 CQM backend and complete modern DPM core/HAL. Important concrete API blockers include removal of `DMA_ATTR_NON_CONSISTENT`, historic `lantiq_dmax.h`/HDMA integration, modified SKB pool-free contracts, and distinct CQM buffer metadata sizes (4 versus 12 bytes). Public PON mailbox/ethernet sources do not substitute for a missing data path or compatible external PON firmware.

Roadmap agreed for research: P0 baseline/ABI and toolchain inventory; P1 minimal MIPS platform and clocks/resets; P2 isolated offline kernel ELF build without datapath; P3 DMA/CQM/BM as an integrated memory domain; P4 coherent DPM packet path; P5 PON and firmware contract integration. Codex was instructed not to modify devices, build or flash firmware, or use GitHub. Full reports remain local and should not be published without separate review for sensitive details. Do not characterize a proposed port as proven bootable.

## Codex P0 build baseline completed (user-reported, 2026-10-22)

Codex reports completion of `reports/prx126-p0-build-baseline.md` on Debian LXC 130. Three MIPS cross-toolchains and functioning `bc` executables were found outside PATH. All five source Git revisions were recorded. The research supports a MIPS32r2, 32-bit big-endian o32 baseline, with EVA requirements considered separately from CPU ISA and the main Linux CPU distinguished from the bootcore. Toolchains have not yet been validated with compilation. Five source repositories reportedly remain clean; no kernel build, downloads, installation, device access, flashing, or GitHub actions were performed by Codex. This is a summary provided by the user; the complete P0 Markdown report has not yet been independently examined.

P1 recommendation: use an isolated working tree; port only SoC platform selection, early entry, clocks/resets, timer/GIC, and console, keeping datapath and PON disabled. Unresolved constraints: EVA and memory mapping, CPU/VPE topology, minimal DT, bootloader/image ABI. Establish a build-only checkpoint before contemplating any device test.

## P0 full report reviewed before P1 (2026-10-22)

The complete `prx126-p0-build-baseline.md` was provided and read. The three locally found cross-compilers are OpenWrt **mips64_octeonplus** GCC 14.3/14.4 installations configured by default for mips64/ABI64 and `--disable-multilib`; only their acceptance of `-mabi=32 -march=mips32r2 -EB -msoft-float` was tested, not object compilation, linking or EVA assembly. TC1 is a provisional choice, not validated PRX126 SDK tooling. Its GCC specs require `STAGING_DIR`, which needs isolation to prevent pulling Octeon userspace headers/libraries. `bc` exists outside PATH. Baseline: main CPU is 32-bit big-endian o32 / MIPS32r2 with distinct EVA/MIPS32r3.5 extensions; `SOC_PRX300_BOOTCORE` and 4Kec image contract are **not** the main Linux CPU. Historical UGW patch differences include relocation/EVA-loadaddress/earlycon changes absent from the older Intel source snapshot. Outstanding hardware and boot-contract choices: EVA mapping, SMP/VPE topology, DT providers and loader image/entry semantics. P1 should be a source-only isolated platform port, while first full kernel compilation belongs to P2; a small standalone toolchain smoke test in P1 can establish compiler feasibility. No P1 instructions have yet been issued to Codex.

## P1 isolated PRX126 platform port — partial completion (user report, 2026-10-22)

Codex reports an isolated Linux 5.15 platform-port worktree at `work/prx126-linux515-p1`. The TC1 freestanding MIPS compiler smoke test and Kconfig/Kbuild consistency checks passed. Three detailed local reports were produced: `reports/prx126-p1-platform-port.md`, `reports/prx126-p1-toolchain-validation.md`, and `reports/prx126-p1-open-blockers.md`. **P1 is partially complete**: a justified EVA/memory mapping, compatible ELF link address, and the remaining architecture audit are prerequisites before a full kernel compilation. Codex explicitly avoided compilation rather than guessing these values. All five original source repositories reportedly remained clean and unchanged. No kernel build, firmware image, network activity, or live-device access occurred. This entry records the user's summary; the three full P1 reports and modified worktree were not independently reviewed. Next: review the three reports; resolve boot/memory assumptions from safe offline evidence before authorizing P2.

## P1 full reports reviewed

Three full P1 reports confirm an isolated platform source port with 12 existing files changed and 15 files added. The freestanding TC1 MIPS32r2/o32/big-endian compile/link and EVA assembly test succeeded. Nine Kconfig/Kbuild consistency checks passed. None of the new kernel source translation units were compiled, and no kernel image was generated. The platform configuration intentionally leaves EVA mapping and ELF link address unresolved. P2 requires an offline boot/memory architecture study first. The generated patch has not yet been inspected independently.
