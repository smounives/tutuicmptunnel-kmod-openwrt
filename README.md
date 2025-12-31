# tutuicmptunnel-kmod-openwrt

本项目为 **多种主流架构** 的 OpenWrt 目标平台提供 [tutuicmptunnel-kmod](https://github.com/hrimfaxi/tutuicmptunnel-kmod) 的自动化构建环境。

该项目利用 GitHub Actions 拉取上游源代码，自动根据选择的架构配置 OpenWrt SDK，并交叉编译内核模块及用户态工具。

## 功能特性

- **多架构支持**：一键编译适用于 x86_64、MediaTek Filogic (AX3000T等)、Ramips MT7621 (AC2100等) 以及通用 ARMv8 的固件。
- **自动化交叉编译**：构建过程完全在 GitHub Actions 云端运行器上执行，无需用户配置本地编译环境。
- **静态链接**：依赖库（`libsodium`、`libmnl`）被静态链接至二进制文件中，确保在不同 OpenWrt 版本间的可移植性，无需在目标设备上额外安装库文件。
- **高度可配置**：支持自定义 OpenWrt SDK 版本（如 `24.10.5` 或 `snapshot`）及目标硬件平台。

## 使用方法

无需将源代码克隆到本地。构建流程完全由 GitHub Actions 托管。

1. **Fork** 本仓库到您的 GitHub 账号。
2. 进入您 Fork 后的仓库页面，点击 **Actions** 标签页。
3. 在左侧栏选择 **Build OpenWrt Multi-Arch Client** 工作流。
4. 点击右侧的 **Run workflow** 按钮。
5. 配置构建参数：
   - **OpenWrt SDK Version**：输入 SDK 版本号（默认：`24.10.5`）。
   - **Target Architecture**：在下拉菜单中选择您的路由器架构：
     - `x86/64` (软路由)
     - `mediatek/filogic` (如小米 AX3000T, 红米 AX6000)
     - `ramips/mt7621` (如红米 AC2100, K2P)
     - `armsr/armv8` (通用 ARMv8)
6. 点击绿色 **Run workflow** 按钮开始构建。

工作流执行完成后，编译产物可在该次运行记录底部的 **Artifacts** 区域下载。

## 构建产物

下载并解压归档文件（例如 `tutuicmptunnel-openwrt-mediatek-filogic-24.10.5.tar.gz`）后，目录结构如下：

- `bin/`: 包含用户态二进制程序 `tuctl`（客户端/服务端主程序）和 `ktuctl`（控制工具）。
- `modules/`: 包含内核模块 `tutuicmptunnel.ko`。

## 安装与部署

1. 将编译好的二进制文件和内核模块传输至 OpenWrt 设备（例如使用 `scp`）。
2. 复制库到`openwrt`设备：

```bash
scp stripped/usr/local/bin/tuctl* router:/usr/bin/
scp stripped/usr/local/sbin/ktuctl router:/usr/sbin/
scp ../libsodium/src/libsodium/.libs/libsodium.so router:/usr/lib/
scp kmod/tutuicmptunnel.ko router:/lib/modules/6.6.119/
```

`ssh`到`openwrt`设备，设置开机启动模块:

```
echo tutuicmptunnel > /etc/modules.d/99-tutuicmptunnel
modprobe tutuicmptunnel
```

此时就可以在`openwrt`设备上使用`tuctl`，`tuctl_client`，`ktuctl`等程序。

## tuctl_client内存使用量问题

在内存受限的设备上，可通过降低密码哈希的内存占用来减少`tuctl_client`的内存使用。
设置环境变量`TUTUICMPTUNNEL_PWHASH_MEMLIMIT`（单位：字节）即可实现。

示例

    服务器（单元文件）：
    Environment=TUTUICMPTUNNEL_PWHASH_MEMLIMIT=1024768

    客户端：
    TUTUICMPTUNNEL_PWHASH_MEMLIMIT=1024768 tuctl_client ...

要点

> 服务器与客户端必须使用相同的数值，否则将无法建立通信。 \
> 该参数数值越小，内存占用越低，但安全性也会相应下降，请根据设备能力和风险评估进行权衡。 \
> 服务器与客户端的TUTUICMPTUNNEL_PWHASH_MEMLIMIT必须一致；不一致将导致握手失败、无法通信。

## 上游项目

本项目仅作为构建包装器（Build Wrapper）。核心逻辑与源代码归上游项目所有：
https://github.com/hrimfaxi/tutuicmptunnel-kmod

## 许可证

MIT License
