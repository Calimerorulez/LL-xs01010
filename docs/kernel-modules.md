# Kernel module metadata — LL-XS1010

Read-only observations made 2026-10-21 on running Linux 4.9.308+ (MaxLinear/OpenWrt vendor build).

## Toolchain and kernel

```text
Linux 4.9.308+
GCC 8.3.0 (OpenWrt GCC 8.3.0 v19.07.8_maxlinear)
Build timestamp: 2024-07-10 08:08:11 UTC
```

The upstream build string contained a personal build username/hostname. This has been removed from the public record.

## Reported modinfo dependencies and licenses

| Module | Depends | License |
| --- | --- | --- |
| mod_pon_mbox | i2c-core | Dual BSD/GPL |
| mod_pon_eth | mod_pon_mbox, qos_tc, gwdpa-dpm | GPL |
| mod_pon_mcc | mcast_bridge_filter, gwdpa-dpm | Dual BSD/GPL |
| mod_sfp_i2c | none listed | GPL |
| gwdpa_dpm | none listed | GPL |
| qos_tc | none listed | GPL |
| ltq_eth_drv_xrx500 | xrx500_phy_fw | GPL v2 |

Dependency spelling is retained as reported by `modinfo`; dashes and underscores may be normalized in Linux module naming.

`/sys/module/<module>` exists for all seven modules, establishing that they are loaded. `modinfo` did not print `filename`, `version`, or `vermagic` for these module queries. The reason has not been established, and the absence of those fields does not establish kernel portability.

## Interpretation

- `mod_pon_eth` depends directly on mailbox, QoS and vendor datapath components.
- `mod_pon_mcc` also relies on the vendor datapath.
- `ltq_eth_drv_xrx500` depends on PHY firmware support.
- These relationships are not a complete map of ioctl/netlink/sysfs interfaces or runtime firmware loading.
- GPL license metadata is not proof that sources for this exact build are available.

## Next read-only investigation

- Inspect presence and properties of corresponding `.ko` files in `/lib/modules/4.9.308+/`.
- Check `modinfo -n` and full `modinfo` output for one module.
- Review kernel symbols and safe sysfs metadata if necessary.
- Compare against published vendor source/SDK only with verified matching board and build references.

Never upload binaries, complete kernel command lines, calibration data or device-specific identifiers.
