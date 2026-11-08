# bixi 6.6.77 精确基线内核 — SukiSU-Ultra builtin + SUSFS

Xiaomi MIX Flip 2 (`bixi`, SM8750) `OS3.0.304.0.WOHCNXM` 专用:在**与原厂逐字节一致**的
6.6.77 基线(AOSP tag `android15-6.6-2025-03_r15`,uname 对应 `g4a507830d890`)上,
集成 **SukiSU-Ultra builtin 分支**(手动钩子,零 kprobe)+ **SUSFS v2.3.0(Inline Hook)**。

内核版本串、构建者、序号、时间戳与原厂官方构建**逐字节一致**
(`kleaf@build-host` / `#1 SMP PREEMPT Thu Jun 12 08:20:39 UTC 2025`)——
基于 uname 的"Tampered kernel"类检测自然通过,无任何运行时 uname 伪装。

## 特性

- SukiSU-Ultra `builtin` 分支:手动钩子直连(无 kprobe 探针可寻)
- SUSFS v2.3.0(HEAD):SUS_PATH / SUS_MOUNT / SUS_KSTAT / SPOOF_UNAME /
  SPOOF_CMDLINE_OR_BOOTCONFIG(可伪装 /proc/bootconfig 的 BL 状态)/ OPEN_REDIRECT /
  PRIORITY_UID_FILTER / HIDE_KSU_SUSFS_SYMBOLS
- `MODULE_SIG_PROTECT` 关闭:system_dlkm 的 GKI 模块(rfkill/cfg80211/bluetooth 等)
  因受保护符号检查需要可信签名而拒载,关闭后全部正常加载
  (代价:任意内核模块可加载,root 设备上属可接受面)
- 构建身份与原厂一致;`config.diff` 仅 KSU/SUSFS 与上述两项

## 要求

- bixi(2505APX7BC)+ **OS3.0.304.0.WOHCNXM**(uname 必须是
  `6.6.77-android15-8-g4a507830d890-ab13636293-4k`;其他基线请改 tag 重编)
- Bootloader 已解锁
- 手头有原厂 boot.img 备份(回滚用)

## 下载

GitHub Releases:取 `boot-bixi-6.6.77-sukisu-susfs.img` +
`ksu_module_susfs.zip` + `SHA256SUMS.txt`,校验后使用。

## 刷入

方式一(fastboot,推荐):

```bat
adb reboot bootloader
fastboot flash boot_a boot-bixi-6.6.77-sukisu-susfs.img
fastboot reboot
```

方式二(已有任意 root 时,免重启进 fastboot):

```sh
su -c dd if=/data/local/tmp/boot.img of=/dev/block/by-name/boot_a bs=4M
su -c sync
```

写入后重启即可。**只动 boot 分区**;vbmeta 永远不要碰。

## 激活 root

1. 安装 [SukiSU-Ultra 管理器](https://github.com/SukiSU-Ultra/SukiSU-Ultra/releases)
   (v4.3.0-r2 或更新)。
2. 管理器首页应显示 **Built-in 工作中**;点「直接安装」重写 init_boot
   (写入 ksud 环境),按提示重启。
3. 重启后 `su` 可用(`context=u:r:ksu:s0`)。

## 安装 SUSFS 用户态模块

管理器 → 模块 → 从本地安装 → `ksu_module_susfs.zip` → 重启。
工具在 `/data/adb/ksu/bin/ksu_susfs`(add_sus_path / set_cmdline_or_bootconfig 等)。

## 可选:BL 状态伪装

内核已启用 `SPOOF_CMDLINE_OR_BOOTCONFIG`(GKI 上 BL 状态在 /proc/bootconfig)。
配合 `/data/adb/service.d/` 启动脚本:

- resetprop 层:`ro.boot.verifiedbootstate=green`、`ro.boot.vbmeta.device_state=locked`、
  `ro.boot.flash.locked=1`、`ro.secureboot.lockstate=locked`、`sys.oem_unlock_allowed=0`、
  `ro.boot.warranty_bit=0`、`ro.warranty_bit=0`
- bootconfig 层:`ksu_susfs set_cmdline_or_bootconfig <已翻绿的 bootconfig 副本>`

两脚本会在每次开机由 ksud 自动执行。对未授权 App(uid≥10000)生效,
root 会话读到的仍是真值。

## 回滚

```bat
adb reboot bootloader
fastboot flash boot_a <原厂 boot.img>
fastboot reboot
```

## 自行编译

Actions → `build-bixi-kernel-sukisu-susfs` → Run workflow(参数全默认)。
产物自动校验:vendor 模块符号 CRC 闸门(module_layout 必须等于原厂 `0x4e276f37`)
+ uname 与原厂逐字节一致性在构建身份层保证。

## 已知边界

- 只对 `bixi` + `OS3.0.304.0` + 6.6.77 验证;OTA 换基线后必须改 tag 重编。
- KPM 未启用(SukiSU builtin 分支的 KPM 需内嵌 KernelPatch 引擎,本配方不含)。
- 硬件级 attestation(Google strong)无解,属平台限制。

## 来源与致谢

- 内核基线:AOSP `kernel/common` `android15-6.6-2025-03_r15`(GPL-2.0)
- 构建底架:[237389749/bixi-droidspaces-kernel](https://github.com/237389749/bixi-droidspaces-kernel)
  (bixi 原厂 IKCONFIG、vendor CRC 闸门、原厂 AVB 打包)
- Root:[SukiSU-Ultra](https://github.com/SukiSU-Ultra/SukiSU-Ultra) `builtin` 分支(GPL-2.0/3.0)
- SUSFS:[susfs4ksu](https://gitlab.com/simonpunk/susfs4ksu) `gki-android15-6.6`
- 集成配方参考:[daixu0502/xiaomi_kernel_bixi_ksu_susfs](https://github.com/daixu0502/xiaomi_kernel_bixi_ksu_susfs)

免责声明:刷内核有变砖风险,请自备 9008/EDL 救砖手段与原厂 boot.img 备份。
