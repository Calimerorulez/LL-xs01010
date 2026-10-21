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
