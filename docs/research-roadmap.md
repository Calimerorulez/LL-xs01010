# Firmware research roadmap

## Current status
- [x] Identify SoC, board, CPU architecture and RAM
- [x] Record current firmware and bootloader versions
- [x] Record physical MTD and logical UBI volume layout
- [x] Identify initial PON-related kernel modules and daemons
- [x] Establish local management connectivity
- [ ] Determine exact boot selection / image update / fallback behavior (read-only first)
- [ ] Inventory exact PON binary dependencies, ELF ABI and library linkage
- [ ] Locate board-specific device tree sources and kernel patch sets
- [ ] Locate MaxLinear proprietary component licensing and sources
- [ ] Design and verify private read-only backup procedure
- [ ] Compare WAS-110 and LL-XS1010 board profiles / firmware image format
- [ ] Evaluate FS.com vendor information when available
- [ ] Create an LL-XS1010 build target only after compatibility evidence exists
- [ ] Document a separately validated recovery procedure before any flash testing

## Candidate modernization paths
1. Maintain/improve 8311 userspace on existing MaxLinear kernel baseline.
2. Investigate newer userspace with existing vendor kernel (ABI/runtime feasibility unknown).
3. Investigate newer kernel only with compatible board support and PON source/driver strategy.

No path has been shown to be bootable on this device.

## Questions for FS.com
- Exact PRX126/PRX300 board variant and device tree?
- Kernel release, MaxLinear SDK version, toolchain and rootfs base?
- Corresponding source, patches, license notices and build instructions?
- PON driver/module versions and userspace/OMCI dependencies?
- Firmware image format, partition map, signatures and recovery path?
- Hardware differences from the Luleey LL-XS1010?
