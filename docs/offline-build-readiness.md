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
