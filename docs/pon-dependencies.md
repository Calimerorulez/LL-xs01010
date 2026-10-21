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
