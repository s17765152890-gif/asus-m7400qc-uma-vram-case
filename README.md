# ASUS M7400QC：Vega 8 UMA 显存从 512MB 调整到 2GB

**实机案例，2026-11-10。** ASUS Vivobook Pro 14X OLED **M7400QC / Ryzen 7 5800H / BIOS 311**，通过原生 U 盘启动 Smokeless UMAF，调整后 macOS 实际报告核显显存为 **2GB**。这是固件预留共享内存的调整，不是仅改显示名称，也不是增加物理内存。

English summary: A verified UMA allocation case on ASUS M7400QC, Ryzen 7 5800H, BIOS 311. Native USB boot of Smokeless UMAF was the successful route; macOS subsequently reported 2GB instead of 512MB. This is a machine-specific report, not a universal BIOS recipe.

## 本机配置

| 项目 | 实机配置 |
| --- | --- |
| 型号 | ASUS Vivobook Pro 14X OLED / M7400QC |
| CPU | AMD Ryzen 7 5800H，8 核 16 线程 |
| 核显 | AMD Radeon Vega 8 / Renoir，PCI ID `1002:1638` |
| 独显 | NVIDIA GeForce RTX 3050 Laptop GPU，4GB，PCI ID `10de:25a2` |
| 内存 | 16GB DDR4-3200，共享内存分配后仍是原来这 16GB |
| 内屏 | 14 英寸 OLED，2880×1800，90Hz |
| 原厂 BIOS | **311** |
| 系统 | macOS 15.8.1，build 24H32 |
| macOS 系统盘 | 外置 512GB HIKSEMI 固态盘 |
| 显存结果 | Vega **512MB → 2GB**；RTX 3050 保持 **4GB** |

## 成功过程与失败尝试

1. 原厂 BIOS 常规界面没有找到可直接修改 UMA 容量的选项，先在 F2 首页确认真实 BIOS 版本为 311。macOS 的 SMBIOS 固件版本可能已被 OpenCore 替换，不能代替原厂版本。
2. 准备 FAT32 U 盘和官方 Smokeless UMAF 的非 BETA 压缩包。早期尝试从 OpenCore 的 Tools 入口运行工具。
3. 在 `Device Manager → AMD CBS → NBIO Common Options → GFX Configuration` 找到了 `iGPU Configuration` 和 `UMA Frame buffer Size`。但右侧参数和选中提示被黑块遮住，不能可靠辨认值。
4. OpenCore 的 TextMode=true / false 两种入口均未改善。它们仍共享同一个 OpenCore 文字渲染器，因此不能据此排除整个 OpenCore 启动链路的影响。
5. **成功路线：把 U 盘改为原版 UMAF 直接启动，绕过 OpenCore。** 用户确认采用该路线完成 2GB 设置。原生布局为根目录的 `DisplayEngine.efi`、`SetupBrowser.efi`、`UiApp.efi`，以及 `EFI/BOOT/BOOTx64.efi` 工具加载器。
6. 进入 macOS 后，独立查询确认 Vega 变为 2GB；随后恢复 U 盘的 OpenCore 救急引导，移除 UMAF 文件和菜单项。

这是实际成功的路线记录。最终 `iGPU Configuration` 的枚举名称、保存确认界面的原始截图没有独立保留，因此这里**不把 UMA_SPECIFIED 等枚举写成已核实的逐键操作**。也没有证据证明所有黑块都完全消失，或已定位到某条渲染器代码；原生启动与成功完成设置之间的关系不能代替完整根因分析。

## 如何确认结果

在 macOS 终端运行只读命令：

```sh
system_profiler SPDisplaysDataType
```

本机相关输出的脱敏摘录：

```text
AMD Radeon RX Renoir Graphics
    VRAM (Total): 2 GB
    Vendor: AMD (0x1002)
    Device ID: 0x1638
    Metal Support: Metal 3

NVIDIA GeForce RTX 3050 Laptop GPU
    VRAM (Total): 4 GB
    Vendor: NVIDIA (0x10de)
    Device ID: 0x25a2
```

最初目标是试 1GB，最终实际成功并验证的是 **2GB**。没有独立验证过 1GB，不能将其也列为本机成功结果。2GB 预留会减少可供 CPU 使用的内存；这次没有做性能基准对比，不能据此声称帧率、解码或 Metal 性能一定提升。

## 复现边界

UMAF 改的是真实固件设置，设置错误可能导致无法开机；EFI 备份无法修复固件层面的启动故障。只有在能看清当前值、确认恢复方式并保留原设置记录时才考虑修改。**不要照着猜按次数，不在黑块遮挡下盲选，不进入工具的 Boot Manager**。该入口的已知问题及其他限制以作者文档为准。

本仓库不提供 BIOS 刷写包、变量偏移写入脚本、自动按键序列或完整 EFI。相同 CPU 的其他笔记本，以及 M7400QC 的其他 BIOS 版本，均不在本案例验证范围内。

## 相关项目与致谢

- [Smokeless UMAF 原项目及风险说明](https://github.com/DavidS95/Smokeless_UMAF)：工具来源，不在此镜像二进制。
- [M7400QC / Ryzen 5 5600H 的既有 Hackintosh 案例](https://github.com/SekiroKenjii/Vivobook-Pro-14X-M7400QC-5600H-Hackintosh)：提供了同机型使用 UMAF 的线索；它的 BIOS 307 / 5600H 与本机 311 / 5800H 不同。
- [本机 OLED 亮度修复案例](https://github.com/s17765152890-gif/asus-m7400qc-oled-brightness-case)：独立问题，增加 UMA 显存不等于修复亮度。

记录由机主与 AI 协作整理。仅公开硬件配置与脱敏结果，不含序列号、设备 UUID、个人路径、原始照片、ACPI 表、账号凭据或完整 EFI。仓库许可证仅适用于本仓库原创内容，上游工具遵循各自许可证。
