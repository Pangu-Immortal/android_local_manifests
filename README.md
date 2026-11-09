# LineageOS 18.1 (Android 11) for LeEco Le Max 2 (x2 / LEX820)

<div align="center">

![萌萌计数器](https://count.getloli.com/get/@android_local_manifests?theme=rule34)

</div>

<p align="center">
  <b>如果觉得有帮助，请点击 <a href="https://github.com/Pangu-Immortal/android_local_manifests/stargazers">Star</a> 支持一下，关注不迷路！</b>
</p>

从源码构建、可长期维护的 LeEco Le Max 2（代号 `x2`，型号 LEX820 全网通，SoC 高通骁龙 820 / MSM8996）的 **LineageOS 18.1 (Android 11)** 移植。这是该设备**官方 LineageOS 支持的最高版本**（官方与社区设备树均止步 18.1）。

设备原厂/RevengeOS 为 Android 9；本移植将其升级到 Android 11。

## 仓库结构（全部自有仓库，非 fork）

| 仓库 | 路径 | 说明 |
|---|---|---|
| [android_device_leeco_x2](../../../android_device_leeco_x2) | `device/leeco/x2` | x2 设备树（含本地修复） |
| [android_device_leeco_msm8996-common](../../../android_device_leeco_msm8996-common) | `device/leeco/msm8996-common` | MSM8996 公共设备树（含本地修复） |
| [android_kernel_leeco_msm8996](../../../android_kernel_leeco_msm8996) | `kernel/leeco/msm8996` | 内核源码（msm-3.18 CAF） |
| proprietary_vendor_leeco（私有） | `vendor/leeco` | 专有 blobs（源自 TheMuppets） |
| **android_local_manifests**（本仓库） | `.repo/local_manifests/x2.xml` | 组装以上仓库的清单 |

上游基于 LineageOS / TheMuppets，遵循各自 LICENSE（Apache-2.0 等），保留原始版权与署名。

## 构建步骤

```bash
# 1) 拉取 lineage-18.1 源码树
mkdir lineage-18.1 && cd lineage-18.1
repo init -u https://github.com/LineageOS/android.git -b lineage-18.1 --depth=1
# 2) 放入本仓库的 local_manifest
mkdir -p .repo/local_manifests
cp /path/to/x2.xml .repo/local_manifests/x2.xml
# 3) 同步
repo sync -c -j8 --no-tags --no-clone-bundle
# 4) 配置并编译
source build/envsetup.sh
breakfast x2
brunch x2         # 或 mka bacon
```

产物：`out/target/product/x2/lineage-18.1-*.zip` + `boot.img` / `recovery.img` / `system.img` / `vendor.img`。

## 在新版宿主机（如 Ubuntu 26.04）上构建的坑与修复

lineage-18.1（2021 年代码）在较新的宿主机上编译需要这些修复：

1. **老预编译 clang 缺 `libncurses.so.5` / `libtinfo.so.5`**（系统只有 .so.6）：
   ```bash
   sudo ln -sf /usr/lib/x86_64-linux-gnu/libncurses.so.6 /usr/lib/x86_64-linux-gnu/libncurses.so.5
   sudo ln -sf /usr/lib/x86_64-linux-gnu/libtinfo.so.6   /usr/lib/x86_64-linux-gnu/libtinfo.so.5
   sudo ldconfig
   ```
2. **`external/chromium-webview` 的 `webview.apk` 是 Git LFS 指针**（repo sync 不带 --git-lfs 时只有指针）：
   单独 `git clone -b main https://github.com/LineageOS/android_external_chromium-webview_prebuilt_arm64` + `git lfs pull`，把真身 `webview.apk` 拷到 `external/chromium-webview/prebuilt/arm64/webview.apk`。
3. **`mke2fs: Invalid filesystem option set: ...orphan_file`**（ART APEX / system 镜像生成失败）：新版 e2fsprogs 的 `/etc/mke2fs.conf` 含老 mke2fs 不认的 `orphan_file`：
   ```bash
   sudo sed -i 's/,orphan_file//g' /etc/mke2fs.conf
   ```

## 刷机步骤（bootloader 需解锁）

```bash
adb reboot bootloader
fastboot flash boot boot.img
fastboot flash system system.img
fastboot flash vendor vendor.img
fastboot flash recovery recovery.img
fastboot format userdata          # 必须 format(建文件系统)，不要用 erase
fastboot format cache
fastboot reboot
```

> ⚠️ **换 ROM 必须 `format userdata`/`cache`（建立文件系统），不要只 `erase`**。仅 erase 会让 `/data` 没有文件系统 → `credstore` 因 `/data/misc/credstore` 不存在崩溃循环，卡在开机动画。
> 也可用官方方式：刷 recovery → `adb sideload lineage-18.1-*.zip`（OTA updater-script 会正确处理 /data 与 `devinfo.sh`）。

## 本移植包含的本地修复

- **device/leeco/x2**：`PRODUCT_ADB_KEYS` 内置构建机的 adb 公钥 → recovery 与系统自动信任该机 adb，便于远程运维（可选，按需替换成自己的公钥）。
- **device/leeco/msm8996-common** `init_leeco_msm8996.cpp`：`ro.leeco.devinfo` 为 `NULL`/空 时兜底为 `le_x2_whole_netcom`（LEX820）。纯 fastboot 直刷会跳过 recovery 里的 `devinfo.sh`（它负责从 `devinfo` 分区读变体并替换 build.prop 的 `NULL`），导致机型识别失败显示 `UNKNOWN` 且丢失双卡/电话配置；此兜底修复该问题。

## 设备信息

- 代号 `x2`，型号 `LEX820`（le_x2_whole_netcom 全网通），SoC MSM8996（骁龙 820）
- 布局：非 A/B、非 dynamic partition、独立 system/vendor，legacy AVB（无 vbmeta/dtbo 分区）
- Treble：支持（可选刷通用 GSI）
---

## Star 趋势

[![Star History Chart](https://api.star-history.com/svg?repos=Pangu-Immortal/android_local_manifests&type=Date)](https://star-history.com/#Pangu-Immortal/android_local_manifests&Date)

## 联系与交流

<p align="center">
  <a href="https://mengqihome.cn"><img src="https://img.shields.io/badge/官网-mengqihome.cn-00e5a0?style=flat-square" alt="官网"></a>
  <a href="https://github.com/Pangu-Immortal"><img src="https://img.shields.io/badge/GitHub-Pangu--Immortal-181717?style=flat-square&logo=github" alt="GitHub"></a>
  <a href="https://t.me/+MH0wU421vyQwYzBl"><img src="https://img.shields.io/badge/Telegram-交流群-26A5E4?style=flat-square&logo=telegram" alt="Telegram"></a>
  <img src="https://img.shields.io/github/stars/Pangu-Immortal/android_local_manifests?style=flat-square&logo=github&label=Stars" alt="Stars">
</p>

<p align="center">
  <b>微信公众号</b><br>
  <img src="https://mengqihome.cn/assets/wechat-qr.jpg" width="140" alt="微信公众号二维码"><br>
  扫码关注 · 案例复盘与方法论
</p>

**更多入口**：[GitHub 主页](https://github.com/Pangu-Immortal) · [Telegram 群](https://t.me/+MH0wU421vyQwYzBl) · [官网](https://mengqihome.cn)

## 许可

本项目采用 [PolyForm Noncommercial License 1.0.0](https://polyformproject.org/licenses/noncommercial/1.0.0) 授权：**允许非商业目的的使用、修改与分发；禁止任何商业性使用**。商业授权请联系 [yugu88@126.com](mailto:yugu88@126.com)。

<!-- managed-block:v2 -->
