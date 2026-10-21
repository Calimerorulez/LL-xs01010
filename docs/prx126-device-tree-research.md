# PRX126-SFP-PON device-tree source research

Research date: 2026-10-21. Public sources and sanitized device observations only.

## Confirmed exact build targets

The public [`paldier/feed_target_mips` UGW 8.4.1-era `image/prx300.mk`](https://github.com/paldier/feed_target_mips/blob/ugw-8.4.1-cleanup/image/prx300.mk) defines both:

```make
define Device/PRX126_SFP_EVA_PON
  $(Device/PRX_SFP)
  FILESYSTEMS := squashfs initramfs
  DEVICE_DTS := prx126-sfp-eva-pon
  DEVICE_TITLE := PRX126 SFP EVA PON WAN
endef
TARGET_DEVICES += PRX126_SFP_EVA_PON

define Device/PRX126_SFP_PON
  $(Device/PRX_SFP)
  DEVICE_DTS := prx126-sfp-pon
  DEVICE_TITLE := PRX126 SFP PON WAN
endef
TARGET_DEVICES += PRX126_SFP_PON
```

These names match the live LL-XS1010's board model `PRX126-SFP-PON` and `compatible` list containing `intel,prx126-sfp-pon` and `intel,prx126-sfp-eva-pon`. The definitions establish a historical build *target*, not the presence of the actual DTS source or a current buildable firmware.

## Vendor-specific board behavior

The official [`maxlinear/feed_pon` script `disable_dying_gasp`](https://github.com/maxlinear/feed_pon/blob/updk_9.1.80/base-files-platform-prx300/files/etc/uci-defaults/disable_dying_gasp) contains a special case for `prx126-sfp-pon` and disables Dying Gasp, describing a hardware design issue. Any firmware modernization must preserve/understand board-specific optoelectronic, GPIO and SFP handling, not simply substitute generic PRX300 settings.

## Similar devices / constraints

[PON.wiki's X-ONU-SFPP documentation](https://github.com/up-n-atom/PONwiki/blob/main/docs/xgs-pon/ont/potron-technology/x-onu-sfpp.md) describes another PRX126 platform with 128 MiB Winbond W25N01GV NAND and approximately 1 GiB RAM. Similarity supports comparative analysis but is **not** evidence of identical pinmux, EEPROM, optics or recoverability. Avoid copying unique serial/identity values from public logs.

## Unresolved

- Exact public source file `prx126-sfp-pon.dts` or equivalent source and any required `.dtsi` include files has **not yet been located**.
- Device-tree build flags, kernel Kconfig, pinmux, reserved memory, clocks, SFP/I2C/GPIO assignments and bootcore hardware partition require comparison against the *running* device tree.
- `maxlinear/linux` `updk_9.1.90` contains PRX300 driver source (Linux 5.15.180) but exact LL-XS1010 board support is **not verified**.
- Modern image compatibility, update format, A/B behavior and recovery remain unknown.

## Safe read-only next step

If the full textual DTS cannot be found publicly, collect only a *privately retained* snapshot of the running device-tree directory or an appropriate boot DTB for **offline analysis**. Publicly publish only redacted node names, compatible strings and generic structural findings. Avoid copying unique MAC addresses, calibration cells, PON credentials, EEPROM contents or full binary dumps to GitHub.

No changes to the LL-XS1010 are required for this research.

## Running device tree root inventory (2026-10-21)

Read-only `/proc/device-tree` root listing showed these notable top-level nodes: `combophy@0`, `cpus`, `i2c-gpio`, `memory@0`, `opp_table`, `pcie@1a400000`, `pcie@1a900000`, `pinselect-1pps`, `pinselect-ntr`, `pinselect-rx-los`, `reserved-memory`, `ssx0@14000000`, `ssx1_0@12000000`, `ssx1_1@18000000`, `ssx1_2@1A000000`, `ssx1_3@1E000000`, `ssx4@16000000`, `ssx7@a0000000`, `thermal-zones`, and `wan_xpcs@18C42000`.

Root compatible strings in order: `intel,prx126-sfp-pon`, `intel,prx126-sfp-eva-pon`, `intel,prx126`, `intel,prx300`.

A `find /proc/device-tree -type d` pipeline with a hardware-node filter returned no results. This is an enumeration limitation or command behavior to investigate; no conclusion should be drawn that nested nodes are absent. The top-level names do **not** yet establish pin assignments, clock/interrupt mappings, full PHY wiring or compatibility with another SFP board. No unique identifiers have been included.


## Running nested device-tree hardware inventory (2026-10-21)

Read-only directory enumeration from `/sys/firmware/devicetree/base` confirms the following **generic structural nodes** (prefix omitted where obvious):

| Path or node | Research relevance |
| --- | --- |
| `ssx1_1@18000000/pon_ethernet@2` | PON Ethernet link |
| `ssx1_1@18000000/ponmbox@100000` | PON mailbox controller |
| `ssx1_1@18000000/pon_ptp@0`, `pon_ptp@1` | PON timing |
| `ssx0@14000000/sfp_i2c@0x50000` | SFP I2C hardware |
| `ssx1_1@18000000/gswitch@400000` | Switch core and MAC/MDIO child nodes |
| `ssx1_1@18000000/eth@a00000` | Ethernet interfaces |
| `ssx1_1@18000000/qspi@1300000/ltq-spinand@0` | SPI NAND device-tree node |
| `i2c-gpio/eeprom@4e`, `eeprom@4f` | EEPROM references; **never read or publish contents** |
| `ssx4@16000000/pinctrl@310000/pon_ip_sfp_tx_dis` | SFP TX-disable pin group |
| `ssx4@16000000/pinctrl@310000/pon_ip` | PON pin group |
| `ssx4@16000000/pinctrl@310000/i2c0_gpio` | GPIO I2C pin group |
| `reserved-memory/bootcore@3E000000` | Bootcore reserved-memory description |
| `reserved-memory/bm_pool@40000000` | Packet-buffer memory pool |
| `reserved-memory/linux,cma@0x2D000000` | Linux CMA reservation |

Additional top-level I2C, GPIO, SPI, PCIe and XPCS nodes were seen. These are *node names only*, not yet validated compatible strings, status values, address ranges, pinmaps or driver bindings. Some nodes can be disabled; their presence does not prove that the device is physically populated or in active use.

`/proc/device-tree` is a symbolic link to `/sys/firmware/devicetree/base`; the earlier empty `find` result was caused by not traversing that symlink.

**Security:** public documentation intentionally excludes the EEPROM contents, MAC addresses, identifiers, calibration values and unique device properties. No binary device-tree dump is published.

Next safe read-only phase: extract `compatible` and `status` strings only from a short allowlist of PON, SFP, NAND, Ethernet and reserved-memory nodes, rather than walking or dumping all properties.


## Verified device-tree binding strings (2026-10-21)

The LL-XS1010 read-only `compatible` / `status` inspection yielded:

| Active device-tree node | `compatible` | `status` | Source mapping |
| --- | --- | --- | --- |
| `pon_ethernet@2` | `intel,prx300-pon` | `okay` | Matching modern source/driver **not yet found** |
| `ponmbox@100000` | `intel,prx300-pon-mbox` | `okay` | Matching modern source/driver **not yet found** |
| `sfp_i2c@0x50000` | `intel,pon-sfp-i2c` | `okay` | Matching modern source/driver **not yet found** |
| `gswitch@400000` | `intel,gswdev` | No status property | **Exact compatible match** in public MaxLinear driver |
| `ltq-spinand@0` | `lantiq,spinand` | `okay` | Matching modern source/driver **not yet found** |
| `wan_xpcs@18C42000` | `intel,xpcs` | `disabled` | Do **not** assume active WAN XPCS path |

In the public `maxlinear/linux` `updk_9.1.90` branch, [`drivers/net/datapath/gswip/gswip_dev/gsw_dev.c`](https://github.com/maxlinear/linux/blob/updk_9.1.90/drivers/net/datapath/gswip/gswip_dev/gsw_dev.c) has an OF match table with the **exact** `intel,gswdev` string. That is confirmed source-level driver binding evidence, not proof the specific older binary is ABI-compatible with Linux 5.15.

Targeted searches in the same public branch returned **no indexed matches** for `intel,prx300-pon`, `intel,prx300-pon-mbox`, `intel,pon-sfp-i2c`, `lantiq,spinand` and `intel,xpcs`. A missing search hit is *not* proof a driver does not exist under another binding/repository; the three active PON-related strings remain unresolved and high priority.

`status` not specified on `gswitch` is consistent with standard DT semantics (normally available unless disabled); it is not an explicit `okay` result. Conversely, `wan_xpcs` is explicitly disabled and must not be treated as the currently active optical path merely because the node exists.

**Next safe step:** read only the non-secret `compatible` and `status` properties on subnodes under `gswitch`, `pon_ethernet`, `sfp_i2c` and selected pinctrl groups; map exact strings to sources. Never read EEPROM cell payloads, calibration or identity fields.


## Driver registrations and first-party PON source discovery (2026-10-21)

A read-only `/sys/module/*/drivers` listing on the LL-XS1010 confirms these registrations:

- `mod_pon_eth`: `platform:prx300-pon`.
- `mod_pon_mbox`: `platform:pon-pinselect`, `platform:pon_mbox`, `spi:pon_mbox`.
- `mod_sfp_i2c`: `platform:pon-sfp-i2c`.

**Critical new source lead:** official public MaxLinear repository [`maxlinear/pon_mbox_drv`](https://github.com/maxlinear/pon_mbox_drv), default branch `master`:

- [`eth/pon_eth.c`](https://github.com/maxlinear/pon_mbox_drv/blob/master/eth/pon_eth.c) contains a device-tree match table referencing `intel,prx300-pon`.
- [`src/drv_pon_mbox.c`](https://github.com/maxlinear/pon_mbox_drv/blob/master/src/drv_pon_mbox.c) contains the PON mailbox implementation, `pon_mbox` match entries, and PON pinselect compatible strings.
- [`ptp/pon_ptp.c`](https://github.com/maxlinear/pon_mbox_drv/blob/master/ptp/pon_ptp.c) contains PON timing device-tree matches, including `intel,prx300-pon-ptp`.

This substantially revises the prior 'no matching source found' status: **public first-party source has been located for PON Ethernet, mailbox, and PTP candidates.** Exact revision/ABI compatibility with the running `4.9.308+` modules has not been tested. The SPI interface may be part of the same module's registration but needs detailed source inspection.

For `intel,pon-sfp-i2c`, exact compatible strings also occur in older third-party Intel target DTS files, e.g. [`paldier/feed_target_mips/dts/EASY98020V18-PONIP.dts`](https://github.com/paldier/feed_target_mips/blob/ugw-8.4.1-cleanup/dts/EASY98020V18-PONIP.dts). These are historical boards (some explicitly simulated) and **are not** the LL-XS1010's final SFP-I2C driver source.

Next: inspect the `maxlinear/pon_mbox_drv` source for precise compatible strings, version and kernel requirements, then locate `mod_sfp_i2c` source and test feasibility by building in a separate offline environment. Never flash or alter production device.


## Official SFP I2C package recipe found (2026-10-21)

Public MaxLinear [`feed_pon/pon-sfp-i2c/Makefile`](https://github.com/maxlinear/feed_pon/blob/updk_9.1.80/pon-sfp-i2c/Makefile) identifies the package and its source relationship:

| Field | Value from recipe |
| --- | --- |
| OpenWrt package | `pon-sfp-i2c` |
| Package version | `1.2.1` |
| Upstream Git source | `maxlinear/pon_sfp_i2c` |
| Kernel module output | `src/mod_sfp_i2c.ko` |
| Firmware filename | `prx_i2c_aca_fw.bin` |
| Firmware package version | `v1.0.2.1.24` |
| License stated in recipe | `GPL-2.0-only or BSD-2-Clause` |

The recipe uses an SSH Git source URL and describes the component as a **PON SFP I2C emulation driver**. A metadata lookup for the exact `maxlinear/pon_sfp_i2c` repository returned GitHub 404; thus **its full source code has not been verified publicly accessible**. A 404 is not conclusive evidence about whether a private repo exists. A public repository search by that project name returned no matches.

The firmware filename is a required dependency in the recipe, **not proof** that this exact firmware version is installed on the observed LL-XS1010. No firmware binaries, EEPROM data or images have been collected or published.

## Current source availability assessment

- Datapath wrapper and PRX300 drivers: publicly available in `maxlinear/linux` (Linux 5.15.180 branch), subject to ABI adaptation.
- PON Ethernet, mailbox and timing: first-party source candidates publicly available in `maxlinear/pon_mbox_drv`; need revision and build-compatibility checks.
- PON SFP I2C: public package recipe establishes precise source name and ACA firmware dependency; **matching C source not found/publicly accessible yet**.
- Exact `prx126-sfp-pon.dts`: only historic build target has been verified, not complete DTS source.

Follow-up: ask MaxLinear or FS.com for the corresponding GPL/source package `pon_sfp_i2c` version 1.2.1, required ACA firmware provenance, kernel compatibility, and the PRX126 board DTS. Retain device-specific contents privately.
