# NOTICE

- Kernel source: Android Common Kernel (`kernel/common`), GPL-2.0.
  Built from tag `android15-6.6-2025-03_r15`.
- Base build flow: adapted from `237389749/bixi-droidspaces-kernel`
  (stock bixi IKCONFIG, vendor module CRC gate, stock AVB boot packing).
- KernelSU: official `tiann/KernelSU` (GPL-2.0), pinned commit
  `932014ab5b2c9b74a3d11e2ec4d17dd10fc9442e`, installed via the upstream
  `kernel/setup.sh` recipe.
- SUSFS: `simonpunk/susfs4ksu` branch `gki-android15-6.6`, pinned commit
  `eba2a88a5ba303e3d79d08e0717b956e9cf784a1` (kernel patches +
  `build_ksu_module.sh` userspace module). Integration steps adapted from
  `daixu0502/xiaomi_kernel_bixi_ksu_susfs`.
- kABI patches (in `patches/`, unused by the default workflow): copied from
  Droidspaces-OSS, authored by nullptr-t-oss and contributors.
- `configs/bixi-stock.config`: extracted from the IKCONFIG embedded in the
  stock `boot.img` of Xiaomi MIX Flip 2 `OS3.0.304.0.WOHCNXM`.
- `configs/bixi-boot-avb.env`: `avbtool info_image` output of the same boot image.
- `tests/*.tsv`: CRC values parsed from `__versions` of vendor modules shipped
  in the same ROM (via the bixi TWRP device tree). These are factual symbol
  CRCs, not source code.

No vendor binaries or ROM images are redistributed in this repository.
