# bixi 6.6.77 + KernelSU + SUSFS

Xiaomi MIX Flip 2 (bixi, SM8750) `OS3.0.304.0.WOHCNXM` 专用:原厂 6.6.77 精确基线
(AOSP tag `android15-6.6-2025-03_r15`,uname 对应 `g4a507830d890`)+ 官方 KernelSU
(固定提交 `932014ab`)+ SUSFS v2.3.0(`eba2a88a`),全部 builtin 编译进 boot.img。

- 基座流程来自 `237389749/bixi-droidspaces-kernel`(已验证 6.6.77 在 bixi 可编可跑):
  bixi 原厂 IKCONFIG、vendor 模块 CRC 闸门、304 原厂 AVB 参数打包
- KSU/SUSFS 集成配方来自 `daixu0502/xiaomi_kernel_bixi_ksu_susfs`
  (官方 setup.sh + susfs4ksu `gki-android15-6.6` 分支)
- 不打 SYSVIPC kABI 补丁(不开容器选项,配置 diff 只有 KSU/SUSFS 与版本串)
- `MODULE_SIG_PROTECT` 保留原厂开启值

## 一、编译(GitHub Actions,免费,约 1~2 小时)

1. 把本目录全部内容推到你自己的 GitHub 仓库(新建公开仓库即可)。
2. 仓库页 → Actions → **build-bixi-kernel-ksu-susfs** → Run workflow,参数全用默认值。
3. 跑完在运行页底部下载两个 artifact:
   - `bixi-kernel-ksu-susfs-android15-6.6-2025-03_r15`(内核 + boot.img)
   - `ksu_module_susfs`(用户态 SUSFS 模块)

已知风险点:`50_add_susfs_in_gki-android15-6.6.patch` 是对新版 6.6 树写的,在
6.6.77 tag 上 context 可能漂移(workflow 已带 `patch --fuzz` 兜底);若仍失败,
把 Actions 日志末尾贴回来修。

## 二、产物自检(刷前)

打开 `BUILD-INFO.txt`:
- `kernel banner:` 应为 `6.6.77-android15-8-g4a507830d890-KSU-SUSFS-ab13636293-4k`
- `module_layout:` 应为 `0x4e276f37`(与 bixi 原厂 vendor 模块一致)
- `KernelSU commit:` `932014ab…`,`SUSFS commit:` `eba2a88a…`

并核对 `SHA256SUMS.txt` 与下载文件一致。

## 三、刷写(先试启动,后落盘)

前置:BL 已解锁、PC 有 304 原厂 boot.img 备份(回滚用)、电量 >50%。

```bat
cd /d <PC 上的 platform-tools 目录>
adb reboot bootloader
fastboot getvar current-slot

:: 1) 临时启动试运行(不写入分区,重启即回原内核)
fastboot boot boot-bixi-6.6.77-ksu-susfs.img

:: 进系统后核验:
adb shell uname -r        :: 应带 KSU-SUSFS 后缀
adb shell su -c id        :: uid=0 ... context=u:r:ksu:s0
:: WiFi/蓝牙/内外屏正常后,重启回 bootloader 落盘:

:: 2) 正式刷入 boot(双槽一致)
fastboot flash boot_a boot-bixi-6.6.77-ksu-susfs.img
fastboot flash boot_b boot-bixi-6.6.77-ksu-susfs.img

:: 3) 恢复原厂 init_boot(清掉残留的 SukiSU LKM hook,root 改由内核提供)
fastboot flash init_boot_a init_boot.img
fastboot flash init_boot_b init_boot.img
::  (init_boot.img = 304 原厂备份,在 PC 的线刷包 images 目录)

fastboot reboot
```

若 `fastboot boot` 不可用,直接从第 2) 步开始;原厂 boot.img 在手,回滚有底。

## 四、装管理器与 SUSFS 模块

1. 安装 `KernelSU_v3.3.1_32680-release.apk`(官方 KernelSU 管理器)。
2. 管理器 → 模块 → 从本地安装 → `ksu_module_susfs.zip` → 重启。
3. 验证:
   - `adb shell uname -r` 带 KSU-SUSFS 后缀
   - `adb shell su -c id` = uid=0, context=u:r:ksu:s0
   - 管理器正常识别;`/data/adb/ksu/bin/ksu_susfs` 存在
   - 银行 App / Eros 复测

## 五、回滚

```bat
:: 回原厂内核(304 原厂 boot.img 双槽刷回)
fastboot flash boot_a boot.img
fastboot flash boot_b boot.img
fastboot reboot

:: 若要回到 SukiSU LKM root,再刷:
fastboot flash init_boot_a kernelsu_patched_initboot.img
fastboot flash init_boot_b kernelsu_patched_initboot.img
```

备份位置:PC 上的 304 线刷包 images 目录(bixi_images_OS3.0.304.0.WOHCNXM)与
mixflip2-unlock 工具目录。

## 注意

- 只对 bixi + OS3.0.304.0 + 6.6.77 有效;OTA 换系统版本后必须重新对 tag。
- 只动 boot / init_boot;**vbmeta 永远不要动**(1 字节错 = ABL HASH_MISMATCH 掉 fastboot)。
- 内核版本串带 `-KSU-SUSFS-` 标记:MODVERSIONS 下 vendor 模块 vermagic 比对会忽略
  版本串 token,只比符号 CRC(已过闸门),WiFi/蓝牙等厂商模块正常加载。
