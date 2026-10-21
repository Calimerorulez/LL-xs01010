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


## Installed loadable module files (2026-10-21)

The following individual `.ko` files were confirmed in `/lib/modules/4.9.308+/` using read-only `find`, `ls` and `modinfo`:

| File | Reported size (ls -lh) |
| --- | ---: |
| `gwdpa-dpm.ko` | 821.4 KiB |
| `ltq_eth_drv_xrx500.ko` | 56.4 KiB |
| `mod_pon_eth.ko` | 79.2 KiB |
| `mod_pon_mbox.ko` | 182.8 KiB |
| `mod_pon_mcc.ko` | 24.2 KiB |
| `mod_pon_mcc_chk.ko` | 11.1 KiB |
| `mod_pon_ptp.ko` | 8.3 KiB |
| `mod_sfp_i2c.ko` | 46.2 KiB |
| `qos_tc.ko` | 148.3 KiB |
| `xrx500_phy_fw.ko` | 16.1 KiB |

Also present: `dp_oam.ko` (32.6 KiB), `dp_eth_reinsert.ko` (8.1 KiB), `mcast_bridge_filter.ko` (5.5 KiB), `pp_net_adapter.ko` (2.2 KiB), and various generic Linux networking modules.

`modinfo mod_pon_eth` resolved `/lib/modules/4.9.308+/mod_pon_eth.ko` and showed `license: GPL`, `depends: mod_pon_mbox,qos_tc,gwdpa-dpm`. `modinfo gwdpa_dpm` resolved `gwdpa-dpm.ko`, showing `license: GPL`, no listed dependencies. Neither printed a `vermagic` field on device.

**Interpretation:** Vendor PON/datapath components are supplied as discrete kernel module binaries, not solely statically compiled into the kernel. These binaries remain tied to an as-yet-unverified kernel ABI; their mere availability does not permit loading them under another kernel.

Next: inspect only read-only `modinfo -F vermagic` and, on a trusted offline analysis host if necessary, `readelf -p .modinfo` on a **privately retained copy** of a module. Do not commit binaries or full dumps to public GitHub.


## Metadata inspection limitations (2026-10-21)

A follow-up read-only test established:

- On-device `modinfo -F vermagic <module>` prints `Usage: modinfo module`: this `modinfo` implementation **does not support** the `-F` option. No vermagic conclusion can be drawn from this test.
- Confirmed module paths include `mod_pon_eth.ko`, `mod_pon_mbox.ko`, `qos_tc.ko`. Vendor datapath module filename is `gwdpa-dpm.ko`, not `gwdpa_dpm.ko`.
- Attempt to read `/proc/config.gz` using `zcat` and grep yielded no output; the pipeline suppressed errors. It is not known whether the config file was present. `CONFIG_MODVERSIONS` remains unknown.
- Next recommended action: copy selected modules **privately** to an analysis machine and use a full host `readelf` to inspect `.modinfo`, `__versions`, ELF headers and undefined symbols. Do not commit binaries or unredacted extracts to public GitHub.

## Undefined-symbol sample from offline mod_pon_eth.ko inspection (2026-10-21)

The first 50 alphabetically sorted undefined ELF symbol names were inspected locally with GNU readelf. Notable vendor datapath API references include `dp_alloc_port_ext`, `dp_register_dev_ext`, `dp_register_subif_ext`, `dp_get_netif_subifid`, `dp_rx_enable`, `dp_xmit`, `dp_set_pmapper`, and `dp_pmac_set`. Other references include `gem_port_id_remove_update`, `gem_port_id_write_update`, `gsw_get_swcore_ops`, and standard Linux netdevice/ethtool symbols.

This is **only a sample**, not a complete dependency inventory. Undefined symbols may be provided by the main kernel or other modules. Export ownership and CRC compatibility have not been established. The vendor-specific datapath API is a key obstacle to assuming drop-in compatibility with an upstream newer kernel. No binaries or device identifiers were published.

## gwdpa-dpm symbol inspection (2026-10-21)

Offline `readelf -sW` confirms local functions `dp_register_dev_ext2`, `dp_alloc_port_ext2`, `dp_xmit2` in `gwdpa-dpm.ko`, as well as absolute CRC symbols for these `*2` names. In the same module, `dp_register_dev_ext`, `dp_alloc_port_ext`, `dp_xmit`, and `dp_*_fn` names appeared as GLOBAL UND (unresolved) symbols. This means `gwdpa-dpm.ko` does **not** itself define the original un-suffixed symbols observed in the sample, notwithstanding its local `*2` implementations. This may involve an abstraction/dispatch layer; symbol ownership and runtime call direction are not confirmed.

Next: inspect exported symbols using `readelf -sW` and `__ksymtab`/`__kstrtab` metadata across the private module collection, and check whether platform kernel built-ins provide the unsuffixed functions. Do not publish binary objects.

## Live kallsyms observation (2026-10-21)

The live PRX126 device shows the following `T` (text symbol) entries in `/proc/kallsyms`:

- `dp_alloc_port_ext`
- `dp_register_dev_ext`
- `dp_xmit`

The kernel addresses are intentionally omitted. This confirms that all three symbol names are present in the running kernel symbol namespace. **It does not establish whether the symbols come from vmlinux or a loaded module**, nor prove their export status or ABI compatibility. Follow-up: inspect complete kallsyms lines and the relevant exporting module or built-in symbol tables.
