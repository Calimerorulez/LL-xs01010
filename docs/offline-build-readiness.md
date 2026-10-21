# Offline build readiness: PRX126 PON drivers on MaxLinear Linux 5.15

Status: **PREPARED / NOT BUILT** — 2026-10-21. No kernel or module was compiled, loaded or flashed. The LL-XS1010 remains unchanged.

## Candidate sources and verified Kbuild targets

| Input | Confirmed source | Build implication |
| --- | --- | --- |
| Vendor Linux | [maxlinear/linux](https://github.com/maxlinear/linux/tree/updk_9.1.90) `updk_9.1.90`, kernel 5.15.180 | Candidate source; **not yet shown to have a complete PRX126/MIPS configuration** |
| PON driver sources | [maxlinear/pon_mbox_drv](https://github.com/maxlinear/pon_mbox_drv), package version 1.28.2 in [Makefile](https://github.com/maxlinear/pon_mbox_drv/blob/master/Makefile) | External Kbuild project; invokes `make -C $(KERNEL_SRC) M=$(SRC)` |
| Mailbox | [src/Makefile](https://github.com/maxlinear/pon_mbox_drv/blob/master/src/Makefile) | `CONFIG_PON_MBOX_DRV=m` builds `mod_pon_mbox.ko`; objects include mailbox, SFP, SerDes and counters |
| Ethernet | [eth/Makefile](https://github.com/maxlinear/pon_mbox_drv/blob/master/eth/Makefile) | `CONFIG_PON_ETH_DRV=m` builds `mod_pon_eth.ko`; objects include `pon_eth`, `pon_eth_fw_alloc`, `pon_eth_fw_gem` and `pon_eth_iphost` |
| Datapath | [drivers/net/datapath/dpm/Kconfig](https://github.com/maxlinear/linux/blob/updk_9.1.90/drivers/net/datapath/dpm/Kconfig) | `DPM_DATAPATH` depends on `LTQ_CBM || MXL_CBM`; PRX300-specific `DPM_DATAPATH_PARTIAL_FEEDS` and `DPM_DATAPATH_HAL_GSWIP31` require `PRX300_CQM` |
| SFP-I2C | [feed_pon/pon-sfp-i2c/Makefile](https://github.com/maxlinear/feed_pon/blob/updk_9.1.80/pon-sfp-i2c/Makefile) | Recipe known, matching C sources not publicly retrieved |

## Readiness gate: architecture

A generic Linux 5.15 source checkout is **not sufficient** for a legitimate PRX126 build. Confirm these before attempting compilation:

1. A PRX300-capable **MIPS** defconfig and compatible SoC Kconfig entries actually exist and are selectable. GitHub's indexed source search did not find an exact PRX300 defconfig or `CONFIG_SOC_PRX300`; this is a search limitation, not proof they are absent.
2. Select correct MIPS target **endianness**, CPU ISA and EVA settings based on an unprivileged ELF header check of the user's *private* `mod_pon_eth.ko` and the exact configured toolchain. Do not guess `mips` versus `mipsel`.
3. The chosen kernel tree builds successfully with a suitable cross compiler, has `modules_prepare` / required generated headers and **matching** `Module.symvers` for modversion CRC resolution. `modules_prepare` alone does not generate a complete `Module.symvers` when `CONFIG_MODVERSIONS=y`.
4. Vendor headers `net/datapath_api.h`, `net/datapath_br_domain.h`, `net/datapath_api_qos.h`, `net/qos_tc.h`, `net/dp_cpu_ing_qos.h`, `pon/pon_ip_msg.h`, `pon/pon_mbox_ikm.h`, and `secure_services.h` are available and compatible.
5. Verify package compatibility and module symbol resolution **before** testing hardware: PON firmware, PPv4, datapath backend, switch core and SFP-I2C emulation are interdependent.

## Safe offline preparation (Debian builder; no device access required)

These commands are **preparation and manifest checks, not a verified build recipe**. They do not use, access or change the live LL-XS1010.

```sh
mkdir -p "$HOME/prx126-offline-research"
cd "$HOME/prx126-offline-research"
git clone --filter=blob:none --single-branch --branch updk_9.1.90 \
  https://github.com/maxlinear/linux.git maxlinear-linux
git clone https://github.com/maxlinear/pon_mbox_drv.git pon_mbox_drv

git -C maxlinear-linux rev-parse HEAD
git -C pon_mbox_drv rev-parse HEAD
sed -n '1,5p' maxlinear-linux/Makefile
find maxlinear-linux/arch/mips/configs -maxdepth 1 -type f \
  | grep -Ei 'prx|lantiq|intel|mxl' || true
grep -R -l -E 'config (PRX300_CQM|SOC_PRX300|MACH_PRX300)' \
  maxlinear-linux/arch/mips maxlinear-linux/drivers/net/datapath 2>/dev/null | head -30
```

**Stop if no selectable PRX300/MIPS kernel platform is present.** Do not replace this with arbitrary `defconfig` and claim PON build compatibility. The source project's default MIPS defconfig is not necessarily a PRX300 defconfig.

## Subsequent controlled compile (only after architecture gate passes)

Once a validated MIPS kernel config, cross compiler and `Module.symvers` exist, the PON project has Kbuild flags `CONFIG_PON_MBOX_DRV=m` and `CONFIG_PON_ETH_DRV=m`. The required build invocation and third-party headers must be derived from the exact platform's SDK, not assumed from the 5.15 source tree. Record logs/errors locally, without identifiers or firmware dumps. Build objects privately; publish only sanitized errors and conclusions.

## Current blockers

- Missing confirmed PRX126-capable Linux 5.15 MIPS configuration/device tree.
- Missing publicly accessible `pon_sfp_i2c` driver C sources and complete firmware dependency checks.
- Unproven kernel 4.9-to-5.15 struct/API/CRC compatibility and vendor SDK source closure.
- No off-device compile completed in this research step.

## Publication boundary

Do not publish private module binaries, full DTB images, EEPROM content, LOID/PON serial, unique MAC addresses, calibration, credentials, full bootloader environment or complete raw firmware dumps.

## Build-host architecture gate results (2026-10-21)

The Debian LXC checkout of public `maxlinear/linux` `updk_9.1.90` reports Linux `5.15.180`. A name-based search of `arch/mips/configs` found no PRX300/Lantiq/Intel/MXL defconfig. A limited grep for `config PRX300_CQM`, `config SOC_PRX300`, or `config MACH_PRX300` under `arch/mips` and `drivers/net/datapath` returned no matching file. These negative results are scoped to the commands run; they do not prove the platform is absent from every repository or branch.

**New historical source lead:** [`paldier/linux`, branch `intel-4.9.206`](https://github.com/paldier/linux/blob/intel-4.9.206/Makefile), reports Linux `4.9.206` and explicitly declares `config PRX300_CQM` under [`drivers/net/ethernet/lantiq/cqm/Kconfig`](https://github.com/paldier/linux/blob/intel-4.9.206/drivers/net/ethernet/lantiq/cqm/Kconfig), dependent on `LTQ_CBM` and `SOC_GRX500`. This is closer to the LL-XS1010 kernel family, but still older than `4.9.308+` and not established as its matching SDK. It does **not** establish Linux 5.15 PRX126 support.

Recommended next investigation, without device access: scan both checked-out 5.15 source and the historical 4.9.206 kernel tree for actual SoC Kconfig/Makefiles, available MIPS defconfig names and DTS files. Continue to avoid builds until an appropriate target/toolchain is confirmed.

## Historical Linux 4.9.206 platform investigation (2026-10-21)

On the Debian build host, inspection of `paldier/linux` branch `intel-4.9.206` returned `config SOC_GRX500` and `config SOC_GRX500_BOOTCORE` in `arch/mips/lantiq/Kconfig` and `config PRX300_CQM` in `drivers/net/ethernet/lantiq/cqm/Kconfig`. The first grep did **not** search for `SOC_PRX300_BOOTCORE`; checking [the actual Kconfig](https://github.com/paldier/linux/blob/intel-4.9.206/arch/mips/lantiq/Kconfig) confirms **`config SOC_PRX300_BOOTCORE` also exists**. This is a separate bootcore option, not proof of a complete main-kernel PRX126 target.

The inspected device-tree directory under `arch/mips/boot/dts/lantiq` contains generic XRX500 and older Lantiq board trees but no filename matching `prx126-sfp-pon`. Its [Makefile](https://github.com/paldier/linux/blob/intel-4.9.206/arch/mips/boot/dts/lantiq/Makefile) lists only `easy50712.dtb`, `easy350_anywan.dtb`, and `easy350550_bootcore.dtb` as conditional DTB targets. Thus this historical kernel source is a partial platform reference; the exact PRX126 SFP device tree is not established there.

Next verify whether a separate UGW `feed_target_mips` or patches adds the PRX126 board DTS at image-build time, and audit the full historical Kconfig dependency chain before cross-compiling. These findings are based on read-only local source inspection and public repository source; no live device changes or proprietary material were used.

## UGW target feed checkout verified (2026-10-21)

On the Debian research host, `paldier/feed_target_mips` branch `ugw-8.4.1-cleanup` was checked for filenames containing PRX126, PRX300 or SFP-PON. The only matching files were `image/prx300.mk` and `image/fakeroot-prx300.conf`. Recursive content search located `DEVICE_DTS := prx126-sfp-eva-pon` at `image/prx300.mk:91` and `DEVICE_DTS := prx126-sfp-pon` at `image/prx300.mk:106`. **No matching DTS source was found in this target-feed checkout.** This indicates the feed contains image build targets but does not itself supply the required board tree. Whether a separate kernel patch set or another vendor source repository provides the DTS remains open.

Next trace the UGW build include chain to determine the expected location of `DEVICE_DTS` source and inspect the available kernel patch series for DTS additions. Keep any reconstructed live DTB private.

## Historical source grep and target-feed patches (2026-10-21)

Debian source checks returned zero matches for the literal `prx126` in the historical `intel-linux-4.9` checkout. Within `arch/mips/boot/dts` and `arch/mips/lantiq`, `prx300` references were present in `arch/mips/lantiq/Kconfig`, GRX500 PROM implementation and PRX300 bootcore clock/early console/system control files. This is consistent with reuse of GRX500 platform facilities; the exact LL-XS1010 mainline board integration remains unverified.

**Correction to prior inference:** the historical `feed_target_mips` checkout **does** contain a `patches-4.9/` kernel patch directory. Earlier searches restricted to filenames with `prx126`/`prx300` did not expose this broader patch set. Sample patch names concern legacy Lantiq/Falcon SPI, GPIO, NAND, XRX200, generic kernel and SDK compatibility. No exact PRX126 DTS content has been established from these patch filenames.

Next run a content-level `git grep` over `feed_target_mips/patches-4.9` for exact PRX126 names, `intel,prx126`, `intel,prx300-pon`, `prx300` and `arch/mips/boot/dts` additions. Examine actual matches before claiming the DTS exists. Compare independently with the larger historical UGW patch tree. No flashing or live-device changes.

## UGW 8.4.2 dedicated DTS patch inspected (2026-10-21)

Inspected the actual public [UGW 8.4.2 `0001-Add-support-for-lantiq-mips-dts-files.patch`](https://gitlab.com/prpl-foundation/intel/linux/-/blob/ugw_8.4.2/linux_patches/0001-Add-support-for-lantiq-mips-dts-files.patch), not just its filename. Its change list adds Lantiq/XRX500 DTS and DTSI files including `xrx500.dtsi`, `xrx500_anywan_bc.dtsi` and bootcore-related trees. **No file named `prx126-sfp-pon.dts` appears in that patch's file-addition list.** A commented example boot command line in the `easy350_anywan.dts` addition mentions a NAND partition arrangement containing a 16 MiB bootcore and 108 MiB `system_sw` region, broadly similar to the observed LL-XS1010 flash structure. This is historical layout context, **not** evidence that this board file is interchangeable or that its bootargs should be applied.

The larger 8.4.2 patch series also contains distinct patches for datapath, CQM, SPI NAND, clocks and defconfig, but the contents of those additional patches have not all been verified. Next search the full UGW 8.4.2 `linux_patches` tree for literal `prx126-sfp-pon` and relevant DTS paths rather than inferring board compatibility from the generic XRX500 example.
