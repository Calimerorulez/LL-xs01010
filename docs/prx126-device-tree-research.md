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


## Public mirrors / exact DTS search, negative result and access lead (2026-10-21)

A follow-up search for `pon_sfp_i2c`, `CONFIG_PON_SFP_I2C`, and the exact file name `prx126-sfp-pon.dts` found **no verified publicly readable C driver source or exact DTS file**. The first-party MaxLinear feed recipe remains the only direct `pon_sfp_i2c` source reference found, and `maxlinear/pon_sfp_i2c` was not accessible through the GitHub repository API at the time of the check. Avoid mistaking unrelated `sfp_i2c.c` files from other hardware vendors for this driver.

A potentially valuable source-access lead is the prpl Foundation's [MaxLinear platform build profile](https://gitlab.com/prpl-foundation/prplos/prplos/-/blob/a17afef7ca0ef6e21674e91284a6b096f136b9ce/profiles/mxl_x86.yml): it documents that some MaxLinear SDK and feed sources are hosted at `git.maxlinear.com` and require granted access. **That profile targets x86/LGM, not PRX126**, so it is an access/provenance lead, not a substitute for a PRX300/PRX126 SDK.

The publicly indexed [`paldier/feed_target_mips` `image/prx300.mk`](https://github.com/paldier/feed_target_mips/blob/ugw-8.4.1-cleanup/image/prx300.mk) confirms `DEVICE_DTS := prx126-sfp-pon`, but searches did not locate the source `.dts`. This is evidence of a named target, not evidence that all required device-tree sources are public.

**Practical next action:** request from MaxLinear or the module supplier the exact matching PRX126-SFP-PON DTS/DTSI sources, `pon_sfp_i2c` v1.2.1 source package, associated ACA firmware provenance and kernel/toolchain support matrix. Independently reconstructed DT data from the running device must be reviewed and stripped of identity/EEPROM/calibration information before any public release.

## Controlled device-tree reconstruction plan (2026-10-21)

Further searches for literal `prx126-sfp-pon.dts`, `prx300.dtsi` and `prx126-sfp-eva-pon` yielded no verified complete board DTS. The historical target declaration is still the strongest exact-name evidence, but a complete board source has not been established.

To reconstruct only the **non-sensitive driver-binding map** from the running system, first read solely `compatible` and `status` properties for an explicit allowlist of PON Ethernet, mailbox, PTP, SFP-I2C, switch-core/MAC and PON pinctrl nodes. Do not recursively print all properties or publish a raw DTB: the tree contains EEPROM nodes and may contain device identifiers, calibration or other sensitive fields.

Next correlate each compatible string to the already located MaxLinear PON mailbox/Ethernet and switch source. Afterward, inspect only necessary structural properties, avoiding identity and calibration data. A source-level DTS reconstruction must remain labeled partial until clocks, interrupts, GPIOs, memory reservations and pinmux are validated.

## PON PTP, switch MAC and pinctrl compatibility inventory (2026-10-21)

Read-only `compatible` / `status` inspection confirms:

| Node | compatible | status |
| --- | --- | --- |
| `pon_ethernet@2` | `intel,prx300-pon` | `okay` |
| `ponmbox@100000` | `intel,prx300-pon-mbox` | `okay` |
| `pon_ptp@0` and `pon_ptp@1` | `intel,prx300-pon-ptp` | `okay` |
| `sfp_i2c@0x50000` | `intel,pon-sfp-i2c` | `okay` |
| `gsw_core@0` | `intel,gsw_core` | status absent |
| `gsw_mac@0`, `@1`, `@2` | `intel,gsw_mac` | `okay` |
| pinctrl `pon_ip` and `pon_ip_sfp_tx_dis` | compatible absent | status absent |

The exact `intel,prx300-pon-ptp` match is found in publicly available first-party [MaxLinear `pon_mbox_drv/ptp/pon_ptp.c`](https://github.com/maxlinear/pon_mbox_drv/blob/master/ptp/pon_ptp.c). The three `gsw_mac` nodes and the two PON PTP nodes are separate nodes; no claim is made that all are physically used at once. Pinctrl child groups generally need no standalone `compatible` declaration. An `okay` status permits probing but does not establish successful driver binding.

Next step: inspect read-only `/sys/bus/platform/devices/*/driver` symlinks for selected nodes and only the structural `compatible`/`status` of the pinctrl controller. No EEPROM data, firmware or calibration should be read or published.

## Verified live platform-driver bindings (2026-10-21)

The running LL-XS1010's read-only `/sys/bus/platform/devices/*/driver` inspection confirms all twelve observed compatible-selected platform devices have a driver symlink:

| Device | Compatible | Bound driver |
| --- | --- | --- |
| `14050000.sfp_i2c` | `intel,pon-sfp-i2c` | `pon-sfp-i2c` |
| `18000000.pon_ptp`, `18000001.pon_ptp` | `intel,prx300-pon-ptp` | `pon_ptp_drv` |
| `18000002.pon_ethernet` | `intel,prx300-pon` | `prx300-pon` |
| `18100000.ponmbox` | `intel,prx300-pon-mbox` | `pon_mbox` |
| `18400000.gswitch` | `intel,gswdev` | `gswdev` |
| `18700000.eip160`, `18720000.eip160` | `intel,gsw_macsec` | `gsw_macsec` |
| `gsw_core.3.auto` | `intel,gsw_core` | `gsw_core` |
| `gsw_mac.0.auto`, `.1.auto`, `.2.auto` | `intel,gsw_mac` | `gsw_mac` |

This adds two **MACsec/EIP160** platformdevices to the driver inventory. Driver symlinks establish successful binding, but are not end-to-end proof of optical link function, packet flow, MACsec usage or a compatible newer-kernel implementation.

Safe next inspection: check the publicly available MaxLinear kernel source for exact `intel,gsw_macsec`, `intel,gsw_core` and `intel,gsw_mac` bindings; then review the matching module ownership without dumping sensitive configuration.

## Exact switch and MACsec source matches in MaxLinear Linux 5.15 (2026-10-21)

GitHub source search in official `maxlinear/linux` branch `updk_9.1.90` found the three newly observed compatible strings:

- `intel,gsw_macsec`: [`drivers/net/datapath/gswip/mac/src/macsec_drv.c`](https://github.com/maxlinear/linux/blob/updk_9.1.90/drivers/net/datapath/gswip/mac/src/macsec_drv.c)
- `intel,gsw_core`: [`drivers/net/datapath/gswip/gswip_dev/gsw_dev.c`](https://github.com/maxlinear/linux/blob/updk_9.1.90/drivers/net/datapath/gswip/gswip_dev/gsw_dev.c)
- `intel,gsw_mac`: [`drivers/net/datapath/gswip/mac/src/mac_drv.c`](https://github.com/maxlinear/linux/blob/updk_9.1.90/drivers/net/datapath/gswip/mac/src/mac_drv.c)

These are exact matching source-level device-tree strings and driver candidates, not evidence of a successful driver port or binary ABI compatibility. The running LL-XS1010 has two bound `gsw_macsec` instances and three bound `gsw_mac` instances; whether those features are actively used has not been established.

## Original boot FDT recovered and decompiled (2026-10-22)

The running LL-XS1010 exposes a 32647-byte FDT at `/sys/firmware/fdt`; its header begins with DTB magic `d00dfeed` and declares the same total size. The binary was transferred privately to the iMac and decompiled with `dtc` into a 1662-line DTS; `dtc` emitted 24 warning lines, not yet individually classified. **The DTB and full DTS are private and must not be uploaded or committed.** The root compatible string includes `intel,prx126-sfp-pon`, `intel,prx126-sfp-eva-pon`, `intel,prx126`, `intel,prx300`. Selected generic device compatibles include `intel,prx300-pon-mbox`, `intel,prx300-pon`, `intel,prx300-pon-ptp`, `intel,prx300-cqm`, `lantiq,prx300-eth`, `lantiq,spinand`, and `intel,prx300-pcie`. These identify the actual boot-time hardware description; a decompiled DTB lacks original DTS include hierarchy, labels, and comments and may contain private board or factory properties.

Next: classify warnings locally; verify disabled/enabled duplicate peripherals and match `compatible` strings against candidate MaxLinear kernel drivers. A successful compile roundtrip proves syntax, not bootability on other kernel versions. Avoid publishing unfiltered DTS, bootargs, serial or calibration values.

## Decompiled DTS roundtrip validation (2026-10-22)

The private 1662-line recovered PRX126 DTS successfully compiled back into a DTB with `dtc -I dts -O dtb`; it has **not** been booted or flashed. The original DTB-to-DTS operation produced 24 warning lines with types: `unit_address_vs_reg` (8), `reg_format` (4), `unique_unit_address` (3), and nine other individual reports spanning PCI, SPI, I2C, interrupts and address properties. The reverse DTS-to-DTB invocation produced 114 **diagnostic lines**, not yet classified as 114 separate warnings. Recompilation alone does not establish byte-for-byte equivalence to the boot FDT or suitability for another kernel. Original and regenerated binaries remain private.
