# Software and driver inventory

Observed 2026-10-21 on running 8311 Community Firmware MOD v2.8.3 basic.

## Core
- Linux `4.9.308+` (vendor-modified)
- LEDE `v19.07.8_maxlinear`, `intel_mips/prx300`
- BusyBox 1.36.1
- `procd`, `netifd`, `ubusd`, `rpcd`
- `uhttpd` / LuCI web interface
- Dropbear SSH
- `omcid` at `/usr/bin/omcid`
- `sfp_i2c` and `todd`

## Notable loaded PON / datapath / Ethernet modules

```text
mod_pon_eth
mod_pon_mcc
mod_pon_mbox
mod_pon_ptp
mod_sfp_i2c
ltq_eth_drv_xrx500
xrx500_phy_fw
gwdpa_dpm
qos_tc
dp_oam
dp_eth_reinsert
```

These names demonstrate loaded components, **not** the availability of corresponding source code or compatibility with newer kernels.

## 8311 daemons/scripts seen in process listing

```text
8311-rx_los.sh
8311-rarpd.sh
8311-pingd.sh
8311-vlansd.sh
```

## Libraries and packaging observations
- `/opt/pon/lib/` contains libnl-3, libnl-genl-3 and libnl-route-3 shared objects.
- `/lib/modules/4.9.308+/` exists.
- `opkg list-installed` produced no entries in the supplied diagnostic output; reason not yet established.

## Management routing finding

Changing the 8311 management gateway to a different on-link router resulted in:
- `8311_gateway` boot environment updated to the correct value
- `network.lct.gateway` remaining at the module's own IP address
- Linux default route remaining incorrectly via the module's own IP address

After explicitly synchronizing OpenWrt UCI and the active default route, management access worked without router-side SNAT. Network-specific addresses are intentionally omitted.

This is a single observed configuration behavior, **not yet a reproduced or confirmed upstream firmware bug**.
