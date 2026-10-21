# PON userspace dependencies — initial inventory

Observed on 2026-10-21, Luleey LL-XS1010, 8311 v2.8.3 basic.

## Observed executable files

| Executable | Path | File size (from ls -lh) | Role |
| --- | --- | --- | --- |
| omcid | /usr/bin/omcid | 932.1 KiB | OMCI daemon |
| sfp_i2c | /usr/bin/sfp_i2c | 28.4 KiB | SFP / I2C helper |
| todd | /usr/bin/todd | 16.3 KiB | Time-of-day / PTP helper |

These roles are inferred from process arguments and executable names; exact implementation and dependencies remain to be verified.

## C runtime / tooling

- `/lib/ld-musl-mips-sf.so.1` -> `libc.so` (symbolic link).
- `/lib/libc.so` size: 638964 bytes.
- Musl MIPS soft-float loader name observed; per-binary ELF ABI is **not yet independently confirmed**.
- `/usr/bin/ldd` exists.
- `file` and `readelf` are **not** present on this device, so earlier invocations yielded no dependency data.

**Do not interpret empty output from absent tools as binaries being statically linked.**

## Pending investigation

1. Inspect ELF headers and dynamic sections offline with `readelf -h -l -d` on privately copied binaries.
2. Record `NEEDED` shared libraries, RPATH/RUNPATH, MIPS ABI flags and ELF interpreter.
3. Examine daemon/kernel communication interfaces and precise version dependencies.
4. Compare components with a supported recent MaxLinear SDK, if one is obtained.
5. Check licenses before redistribution: no binary or vendor SDK content belongs in this public documentation repository by default.

## Confidentiality

Only filenames, generic versions, and ABI/dependency metadata should be committed. Keep copied executable files private and out of Git.


## Confirmed runtime linkage (`ldd`, 2026-10-21)

All three executables resolve against the musl MIPS soft-float loader `/lib/ld-musl-mips-sf.so.1`. This is runtime linkage observed on the original device, not a portability test.

| Library | omcid | sfp_i2c | todd |
| --- | :---: | :---: | :---: |
| libcli.so.1 | yes | | |
| libuci.so | yes | | |
| libubus.so.20210603 | yes | | |
| libcrypto.so.1.1 | yes | | |
| libifxos.so.0 | yes | yes | |
| libadapter.so.0 | yes | | |
| libsafec-3.3.so.3 | yes | yes | yes |
| libargp-shared.so.0 | yes | | yes |
| libgcc_s.so.1 | yes | yes | yes |
| libubox.so | yes | | |
| libc.so (musl loader) | yes | yes | yes |

Notable compatibility constraints:
- `omcid` requires `libcrypto.so.1.1` (OpenSSL 1.1 ABI); do not substitute OpenSSL 3 via symlinks.
- `omcid` requires `libubus.so.20210603`; compatibility with newer OpenWrt ABI is not verified.
- `libadapter.so.0` and `libifxos.so.0` require separate dependency and kernel-interface analysis.
- `ldd` resolution on the original system does **not** establish compatibility with newer userspace or kernel.

Runtime load addresses and other device-specific context have been excluded.


## Second-level shared library linkage (2026-10-21)

Observed by running `ldd` on the LL-XS1010:

| Library | Symlink target | Direct runtime library dependencies |
| --- | --- | --- |
| `/usr/lib/libadapter.so.0` | `libadapter.so.0.0.0` | `libgcc_s.so.1`, musl libc |
| `/usr/lib/libifxos.so.0` | `libifxos.so.0.0.0` | `libgcc_s.so.1`, musl libc |
| `/usr/lib/libcli.so.1` | `libcli.so.1.0.0` | `libifxos.so.0`, `libgcc_s.so.1`, musl libc |

`ldd` displays `ldd` itself as the loader alias for these shared objects; this is an observed display detail, not proof that the libraries are statically linked.

There are no additional **direct shared-object** dependencies visible in these three results. This does not rule out direct kernel calls, ioctl/sysfs/netlink interactions, runtime dynamic loading, or version-specific vendor ABI dependencies.

Next: collect read-only module metadata (`modinfo` where available, `/sys/module` metadata) and investigate runtime interface requirements. Do not copy private firmware binaries to this public repository.
