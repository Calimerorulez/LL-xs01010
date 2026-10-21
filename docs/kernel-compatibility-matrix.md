# PRX126 Linux 4.9 → Linux 5.15 kernel compatibility matrix

Research date: 2026-10-21. **Source-level assessment only; no driver has been compiled, linked, loaded or validated on Linux 5.15 on the LL-XS1010.** Do not flash images or modify the live ONU.

## Baselines and evidence

- **Observed device:** PRX126-SFP-PON / PRX300 rev 1.3; vendor Linux `4.9.308+`, MIPS; actual driver bindings documented in [device-tree research](prx126-device-tree-research.md).
- **Public candidate kernel:** [`maxlinear/linux`, `updk_9.1.90`](https://github.com/maxlinear/linux), `Makefile` declares Linux **5.15.180**. This is a vendor-modified kernel tree, not proof of PRX126 board support.
- **Public PON source:** [`maxlinear/pon_mbox_drv`](https://github.com/maxlinear/pon_mbox_drv), top-level `Makefile` states package **1.28.2** and builds `eth/`, `ptp/`, `vuni/`, `src/` via external kernel-module builds (`KERNEL_SRC`). This source revision has **not** been matched to the running 4.9 binaries.
- **Source missing:** [`feed_pon/pon-sfp-i2c/Makefile`](https://github.com/maxlinear/feed_pon/blob/updk_9.1.80/pon-sfp-i2c/Makefile) names `pon_sfp_i2c` v1.2.1 and `mod_sfp_i2c.ko`; matching standalone C source was not publicly retrieved.

## Driver-by-driver matrix

| Runtime driver / component | Public candidate and exact observed binding | Direct source dependencies / interfaces | Linux 5.15 assessment | Key blocker / test |
| --- | --- | --- | --- | --- |
| **PON Ethernet** `mod_pon_eth`, `prx300-pon` | [`pon_mbox_drv/eth/pon_eth.c`](https://github.com/maxlinear/pon_mbox_drv/blob/master/eth/pon_eth.c); `intel,prx300-pon` | Standard kernel: `linux/netdevice.h`, `linux/ethtool.h`, `linux/skbuff.h`, `net/switchdev.h`; **vendor**: `net/datapath_api.h`, `net/datapath_br_domain.h`, `net/datapath_api_qos.h`, `net/qos_tc.h`, `net/dp_cpu_ing_qos.h`, `pon/pon_ip_msg.h`, `pon/pon_mbox_ikm.h` | **Source available; not yet build-tested.** Requires vendor kernel headers and datapath/QoS/mailbox; possible network API drift | Build against matching vendor 5.15 headers; collect undefined symbols and compare struct layouts / MODVERSIONS CRCs |
| **PON mailbox / pinselect / SPI** `mod_pon_mbox`, `pon_mbox` | [`pon_mbox_drv/src/drv_pon_mbox.c`](https://github.com/maxlinear/pon_mbox_drv/blob/master/src/drv_pon_mbox.c); runtime `intel,prx300-pon-mbox` and driver registration | Kernel: `linux/firmware.h`, `linux/spi/spi.h`, `linux/pinctrl/consumer.h`, `linux/clk.h`, `linux/reset.h`, `linux/regmap.h`, `linux/mfd/syscon.h`, `net/genetlink.h`; vendor `pon/pon_ip_msg.h`, `pon/pon_mbox_ikm.h`, `secure_services.h` | **Source available; not yet build-tested.** Likely API/config dependencies plus proprietary PON firmware/runtime handshake | Check OF matching, resource mappings, firmware naming and mailbox protocol; isolated compile before runtime |
| **PON PTP** `pon_ptp_drv` (two nodes) | [`pon_mbox_drv/ptp/pon_ptp.c`](https://github.com/maxlinear/pon_mbox_drv/blob/master/ptp/pon_ptp.c); `intel,prx300-pon-ptp` | Kernel `linux/ptp_clock_kernel.h`, `linux/platform_device.h`, `linux/version.h`; vendor `pon/pon_ip_msg.h`, `pon/pon_mbox_ikm.h`. Contains `LINUX_VERSION_CODE` conditional with `KERNEL_VERSION(6,2,0)` | **Source available; version-aware code present.** Not proof of 5.15 compile success | Build with mailbox dependency and validate PTP registration / timing |
| **SFP-I²C emulation** `mod_sfp_i2c`, `pon-sfp-i2c` | [`feed_pon` package recipe](https://github.com/maxlinear/feed_pon/blob/updk_9.1.80/pon-sfp-i2c/Makefile); `intel,pon-sfp-i2c` (live) | Recipe references source `maxlinear/pon_sfp_i2c`, build `CONFIG_PON_SFP_I2C=m`; firmware `prx_i2c_aca_fw.bin`; EEPROM initialization file specified (do **not** extract device-specific contents) | **Blocking: full matching C source unavailable**; kernel ABI/API requirements unknown | Obtain licensed matching driver source, verify ACA firmware version and board-specific safety requirements |
| **Datapath wrappers / dispatcher** `dp_alloc_port_ext`, `dp_register_dev_ext`, `dp_xmit` | [`maxlinear/linux/drivers/net/datapath/dpm/datapath_api_fn.c`](https://github.com/maxlinear/linux/blob/updk_9.1.90/drivers/net/datapath/dpm/datapath_api_fn.c); exported symbols | Kernel network datapath types plus vendor `include/net/datapath_api*.h`. Wrappers dispatch through `dp_*_fn` pointers | **Source available in 5.15 vendor kernel.** API names align with running 4.9; ABI **not established** | Inspect actual dispatch hook registration, vendor datapath configuration, struct changes, exported CRCs |
| **Datapath backend** `gwdpa_dpm` | Local module has `dp_*2` symbols; candidate vendor datapath source in [`maxlinear/linux/drivers/net/datapath`](https://github.com/maxlinear/linux/tree/updk_9.1.90/drivers/net/datapath) | Packet processing / CQM / QoS; `drivers/net/datapath/dpm/Kconfig` references `PRX300_CQM || LGM_CQM` | **Partial architectural source match, no exact binary-module source mapping** | Identify equivalent backend implementation and kernel Kconfig for PRX300, then compile |
| **Switch controller/core** `gswdev`, `gsw_core` | [`maxlinear/linux/.../gsw_dev.c`](https://github.com/maxlinear/linux/blob/updk_9.1.90/drivers/net/datapath/gswip/gswip_dev/gsw_dev.c); `intel,gswdev`, `intel,gsw_core` | Vendor `net/switch_api/gsw_dev.h`, `xgmac_common.h`, `xgmac.h`, `gswss_api.h`, `gsw_flow_core.h`; kernel OF/reset | **Matching binding and source in 5.15**, build untested | Resolve vendor switch/PHY dependency graph, DT clocks and reset resources |
| **Switch MAC** `gsw_mac` (three nodes) | [`maxlinear/linux/.../mac_drv.c`](https://github.com/maxlinear/linux/blob/updk_9.1.90/drivers/net/datapath/gswip/mac/src/mac_drv.c); `intel,gsw_mac` | Vendor `xgmac.h`, `lmac_api.h`, `gswss_api.h`, `net/switch_api/adap_ops.h`; kernel clock APIs | **Matching source and binding in 5.15**, build untested | Check MAC-driver dependency versions and chip-specific clock/pin configuration |
| **MACsec / EIP160** `gsw_macsec` (two nodes) | [`maxlinear/linux/.../macsec_drv.c`](https://github.com/maxlinear/linux/blob/updk_9.1.90/drivers/net/datapath/gswip/mac/src/macsec_drv.c); `intel,gsw_macsec` | Standard kernel platform/OF/IO-resource/IRQ APIs; vendor `mac_cfg.h`. Source reads device-tree `e160-devid`, `e160_reg` resource and `e160_irq` | **Matching source and binding in 5.15**, build untested | Check generic DT resource names and subsystem availability; binding does not prove MACsec is actively used |
| **SPI NAND / boot storage** | Runtime `lantiq,spinand` | SPI-NAND and NAND/UBI plus vendor flash controller and board-specific partition mapping | **Not mapped** to a matching 5.15 driver | Flash controller, ECC, UBI, bootcore and dual-bank layout must be proven before any boot experiment |

## Findings / constraints

1. **There is no compile test yet.** This matrix is a source audit, not a compatibility certification.
2. The official public PON mailbox project is at package version **1.28.2** at inspection time, but the installed device module version has not been verified against it.
3. The current PON Ethernet source depends on numerous MaxLinear-specific headers, so standard upstream Linux headers are insufficient.
4. The mailbox source includes generic-netlink, firmware, clock, pinctrl and SPI integration as well as vendor headers, making compile- and runtime-dependency checks essential.
5. The PTP code has explicit kernel-version conditional logic, but that alone does not establish support for Linux 5.15.
6. Device-specific SFP EEPROM contents, unique optical calibration and PON/ONT identity are out of scope and must never be published.
7. A **read-only/off-device** first compile against the public 5.15 vendor tree (with MIPS target configuration and complete external include paths) is the next technical validation. No images should be flashed and no binaries committed to this public repository.

## Proposed off-device validation order

1. Establish a reproducible source manifest with branch, commit, target architecture and licenses.
2. Enumerate the required vendor header dependency closure, including PON mailbox, datapath, QoS, GSWIP and secure-services references.
3. Check whether `maxlinear/linux` contains an available PRX300 MIPS defconfig, device-tree prerequisites and cross-compilation toolchain requirements.
4. Run separate compile-only checks for mailbox/PTP, PON Ethernet and switch components, collecting errors and fixes in a private working directory.
5. Mark build successes distinctly from device runtime validation; no deployment until safe recovery and flash layout are fully established.

## Installed device module dependency metadata (2026-10-21)

A read-only `opkg list-installed | grep -Ei 'pon|sfp|datapath|qos|switch|gsw'` returned no matching package names on the live LL-XS1010. This does **not** establish the modules are absent; their running presence was previously confirmed. Package registration, image embedding or naming may explain the discrepancy.

`modinfo` reports only license and direct `depends` entries for the selected modules, with no version or description lines:

| Module | License | Direct `depends` |
| --- | --- | --- |
| `mod_pon_eth` | GPL | `mod_pon_mbox,qos_tc,gwdpa-dpm` |
| `mod_pon_mbox` | Dual BSD/GPL | `i2c-core` |
| `mod_pon_ptp` | GPL | `mod_pon_mbox` |
| `mod_sfp_i2c` | GPL | none listed |
| `gwdpa-dpm` | GPL | none listed |

Direct dependencies are not a complete functional or runtime dependency graph. These commands did not reveal specific installed driver or MaxLinear SDK release versions. Next consider **only non-sensitive** build metadata (module `srcversion`/`.modinfo` if present, public package recipe correspondence), without dumping EEPROM, firmware blobs, PON identity or unredacted build-path credentials.

## Focused embedded version-string inspection (2026-10-21)

A filtered `strings` check of installed `mod_pon_eth`, `mod_pon_mbox`, `mod_pon_ptp`, `mod_sfp_i2c`, and `gwdpa-dpm` kernel modules found no literal driver or package version values. The sole match was a runtime formatting string in `mod_pon_mbox`: `pon_mbox: FW version: %d.%d.%d.%d.%d-%d` (with a leading kernel logging-format byte). This formatting string does not identify the module revision. Earlier boot logging reported PON firmware `3.21.0.3.16-1674463172`; it is a firmware component version, not a confirmed SDK or module source revision.

Next use the private offline ELF module copies to inspect `.modinfo` fields (`srcversion`, `vermagic`, dependencies), build-id if present, and optional version control metadata, publishing only non-sensitive derived results. Avoid posting complete ELF strings or raw module objects.
