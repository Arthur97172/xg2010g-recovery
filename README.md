# XG2010G Initramfs Recovery

为 **Gemtek XG2010G（Airoha AN7581）** 构建 Initramfs Recovery 镜像的 GitHub Actions 自动化项目。

本项目使用 [ImmortalWrt-for-Gemtek-XG2010G](https://github.com/naoki66/ImmortalWrt-for-Gemtek-XG2010G) 作为编译源码，通过 GitHub Actions 自动完成配置、编译、镜像检查、SHA256 校验和 GitHub Release 发布。

## GitHub Actions 状态

[![Build XG2010G Initramfs Recovery Test](../../actions/workflows/build-xg2010g-recovery.yml/badge.svg)](../../actions)

> 如果徽章无法显示，请检查工作流文件名是否为 `build-xg2010g-recovery.yml`。如果实际文件名不同，请相应修改徽章链接。

## 项目特点

- **自动编译：** 通过 GitHub Actions 在线编译，无需自行准备完整的本地编译环境。
- **仅生成 Recovery：** 移除 XG2010G 设备定义中的 Sysupgrade 镜像生成规则，仅保留 Initramfs Recovery 镜像。
- **精简配置：** 使用最小化目标配置，不包含第三方功能。
- **精简设备软件包：** 从 Recovery 设备定义中移除 PON 相关软件包，避免编译时依赖额外的 PON 源码和私有 SDK。
- **自动校验：** 检查生成的镜像是否唯一、是否非空，以及文件大小是否超过 64 MiB，并生成 SHA256 校验文件。
- **自动发布：** 编译成功后，自动将 Recovery 镜像和校验文件上传到 GitHub Release。
- **保留构建日志：** 即使编译失败，也会尝试上传诊断日志，方便排查问题。

## 目标设备

| 项目 | 说明 |
|---|---|
| 设备型号 | Gemtek XG2010G |
| SoC / Target | Airoha AN7581 |
| 固件基础 | ImmortalWrt |
| 镜像类型 | Initramfs Recovery |
| 输出格式 | `.itb` |
| 构建平台 | GitHub Actions，Ubuntu 24.04 |
| 发布方式 | GitHub Release |
| 校验算法 | SHA256 |

## 下载 Recovery 镜像

1. 打开本仓库的 [Releases 页面](../../releases)。
2. 选择需要的 `XG2010G Initramfs Recovery` 预发布版本。
3. 在 **Assets** 中下载 `.itb` Recovery 镜像。
4. 同时下载 `SHA256SUMS` 校验文件。
5. 按照下方说明验证镜像完整性。

当前工作流默认将每次成功构建发布为独立的 Pre-release，并使用运行编号和运行尝试次数生成标签。

**注意：** GitHub Release 发布成功只表示自动化构建和上传流程完成，不代表镜像已在真实设备上验证。

## 自动编译

本项目通过 GitHub Actions 手动触发构建。

1. 打开仓库的 **Actions** 页面。
2. 选择 `Build XG2010G Initramfs Recovery Test` 工作流。
3. 点击 **Run workflow**。
4. 选择并行编译任务数：
   - `1`：占用资源较少，但编译时间较长。
   - `2`：默认配置。
   - `3`：更高并行度。
   - `4`：更高并行度，可能增加内存占用。
5. 点击 **Run workflow** 开始构建。
6. 等待任务完成，并检查各步骤是否成功。

构建成功后，可以在对应的工作流运行记录中下载 Artifact，也可以在 Releases 页面下载发布的镜像。

## 输出文件

正常构建完成后，工作流会导出以下文件：

| 文件 | 说明 |
|---|---|
| `*gemtek_xg2010g*recovery.itb` | XG2010G Initramfs Recovery 镜像 |
| `SHA256SUMS` | Recovery 镜像的 SHA256 校验值 |

构建过程的诊断 Artifact 可能包含以下文件：

- `feeds-update.log`：更新软件源的日志。
- `feeds-install.log`：安装软件源的日志。
- `config.log`：生成编译配置的日志。
- `download.log`：下载编译依赖的日志。
- `build.log`：主要编译日志。
- `.config`：实际使用的编译配置。
- `xg2010g-recovery.config`：保存的目标配置。
- `selected-package-symbols.txt`：选中的软件包配置符号。

诊断文件用于排查构建问题，不属于 Recovery 固件本身。

## 验证 SHA256

下载 Recovery 镜像和 `SHA256SUMS` 后，建议先验证文件完整性。

### Linux

在两个文件所在的目录执行：

```bash
sha256sum -c SHA256SUMS
```

如果输出对应镜像文件的 `OK`，表示文件的 SHA256 与校验清单一致。

### Windows

在 PowerShell 中执行：

```powershell
Get-FileHash .\your-recovery.itb -Algorithm SHA256
```

将输出的 `Hash` 与 `SHA256SUMS` 中对应镜像的 SHA256 值进行比较。

请将 `your-recovery.itb` 替换为实际下载的文件名。

> SHA256 校验只能检查文件是否与校验值一致，不能证明固件来源可信、固件适用于设备，或刷写操作没有风险。

## Recovery 镜像说明

Initramfs Recovery 与常规 Sysupgrade 固件不同。

- Recovery 镜像用于启动临时恢复系统，具体用途取决于设备启动流程及镜像支持的功能。
- 本工作流不生成常规 Sysupgrade 镜像。
- Recovery 设备配置移除了 PON 相关软件包，因此不能假定该环境具备完整的 GPON/XGS-PON 驱动、光纤注册和管理能力。
- 本项目没有承诺镜像包含 Web 管理界面、SSH 服务、特定网络服务或完整的光纤接入功能。实际能力应以镜像配置和设备测试结果为准。

**本项目生成的 Recovery 镜像不应直接当作可用于日常运行的完整固件。**

## 刷写前注意事项

1. 确认设备型号确实为 Gemtek XG2010G，并核实硬件版本及启动方式。
2. 备份原厂固件、分区布局、重要配置和设备校准数据。
3. 如果设备支持串口控制台，建议提前确认串口连接和恢复方法。
4. 阅读源码项目中的设备说明，确认镜像格式、启动流程及相关限制。
5. 不要在未确认镜像用途的情况下，将 Recovery 镜像当作 Sysupgrade 固件刷入。
6. 不要在没有可靠恢复途径的情况下覆盖引导程序、校准数据或其他关键分区。
7. 首次测试优先采用可恢复的启动方式，并确认网络、串口和恢复流程正常。

错误的镜像、分区操作或启动参数可能导致设备无法启动，甚至需要串口或硬件恢复。

## 源码与致谢

本项目使用以下公开源码作为编译基础：

- [ImmortalWrt-for-Gemtek-XG2010G](https://github.com/naoki66/ImmortalWrt-for-Gemtek-XG2010G)
- [ImmortalWrt](https://github.com/immortalwrt/immortalwrt)

本项目的 GitHub Actions 工作流负责自动化构建和发布。最终镜像的功能、兼容性及稳定性取决于源码版本、设备配置、编译结果和实际测试。

## 免责声明

本项目主要用于技术研究、Recovery 构建与设备恢复测试。

- 固件按照现有源码和配置构建，不保证适用于所有硬件修订或运营商定制版本。
- 使用者应自行确认设备兼容性，并自行承担刷写、启动和配置修改的风险。
- 请勿在没有备份和可靠恢复方案的情况下操作关键分区。
- 本项目不保证设备恢复成功、网络注册成功或特定功能可用。

使用本项目即表示你理解并接受上述风险。
