# isorespin：IA32 UEFI 适配说明

本项目基于原版 Linuxium `isorespin.sh` 8.7.1。目标硬件是 **64 位 x86
处理器配合 32 位 UEFI 固件**，不是安装 32 位操作系统。项目包含旧版兼容
路径和面向新镜像的内容识别路径。

## 目录与范围

- `isorespin.sh`：主要入口。Mint 20.3 继续使用旧版流程；Mint 22.3 和
  Ubuntu/Xubuntu 26.04 使用现代适配路径。
- `lib/atom/`：原版 `--atom` 模式的历史驱动安装脚本和固件。
- `lib/isorespin/`：现代 IA32 引导适配器、安装阶段脚本和维护脚本。
- `tests/`：自动化回归测试和 ISO 结构测试。

当前 `isorespin.sh` 已移除旧的 `--atom` 自动驱动注入。`lib/atom/` 中的
脚本和固件仅作为历史资源保留，不代表新内核仍然支持这些设备。

## 支持的镜像

| 镜像 | 路径 | 当前状态 |
| --- | --- | --- |
| Mint 20.3 | 原版 8.7.1 流程 | 保留旧路径，尚未重新进行完整实机安装验证 |
| Mint 22.3 Cinnamon/Xfce | Ubiquity IA32 适配 | 已完成内容探测、结构测试和 Xfce 完整构建；未完成平板实测 |
| Ubuntu/Xubuntu 26.04 Desktop | Subiquity/curtin IA32 适配 | 已完成合成镜像和安装器适配测试；尚未使用完整官方 ISO 实机验证 |

成功生成 ISO 不等于硬件兼容认证。老平板可能无法满足新版桌面和安装器的
内存、显卡或设备要求。

## 使用

现代路径需要 amd64 Linux 主机、Python 3.10+、PyYAML、xorriso、
squashfs-tools、mtools、dpkg-dev 和 ubuntu-keyring，并且完整构建需要 root：

```sh
./isorespin.sh --inspect -i ~/Downloads/iso/linuxmint-22.3-xfce-64bit.iso
sudo ./isorespin.sh -i ~/Downloads/iso/linuxmint-22.3-xfce-64bit.iso
```

现代路径参数为 `-i/--iso`、`-w/--work-directory`、`--inspect` 和
`--keep-work`。默认输出名为 `linuxium-<原镜像文件名>.iso`，并在旁边生成
JSON 构建报告。已有输出不会覆盖；失败时保留 `isorespin-build-*` 工作目录。

旧版路径仍支持原脚本的完整选项集合。请使用 `./isorespin.sh --help` 查看。
现代路径不接受旧版的换内核、升级系统、任意命令和持久化等高级选项，遇到
不支持的参数会明确报错。

## 现代实现原理

构建器根据 squashfs 内的 `os-release`、Linux Mint 信息、内核配置和安装器
文件识别镜像，而不是根据文件名或卷标判断。它检查 `CONFIG_X86_64`、
`CONFIG_EFI_MIXED`、`CONFIG_EFI_STUB` 和 `CONFIG_EFI_HANDOVER_PROTOCOL`。

它保留原始内核、initramfs、Casper UUID、软件包数据库、squashfs 层关系、
压缩设置和安装源，只修改安装器所需的 IA32 引导阶段。构建时不运行目标文件
系统中的程序，也不安装或删除普通桌面软件包。

Mint 22.3 使用 Ubiquity 适配器：只有 Live 系统通过 32 位 UEFI 启动时，才在
真正的引导器安装阶段调用 `lib/isorespin/install-ia32`；其他固件继续使用原
安装路径。Ubuntu/Xubuntu 26.04 使用 Subiquity/curtin 适配器，并将修改过的
安装器 snap 标记为本地 unasserted，禁用其自更新，避免覆盖 IA32 修改。

ISO 内包含经过校验的离线仓库，其中有可共存的 IA32 GRUB 模块和
`isorespin-ia32` 维护包。构建器使用空软件包状态和输入镜像中的软件包状态
分别模拟依赖，不替换发行版的签名 GRUB/shim，也不删除 Mint 安装器。安装后，
维护包可在内核或 GRUB 更新后重新生成 IA32 引导文件；更新时 ESP 必须挂载到
`/boot/efi`。

输出 ISO 保留原始 BIOS El Torito、UEFI、混合 MBR/GPT/APM 启动布局和 x64 EFI
文件，并从实际 EFI FAT 镜像中读取检查 IA32 文件。项目使用未签名 IA32 GRUB，
必须关闭 Secure Boot。

## 验证与限制

自动测试覆盖 shell 语法、镜像识别、内核能力检查、安装器源码结构检查、旧版
嵌入式引导资源回归、合成 ISO 重打包、EFI 分区读回和分层安装器 snap 适配。
这些测试不等同于真实固件启动测试。

已对本地 Mint 22.3 Xfce 完成文件和 ISO 结构级构建检查：MD5 校验通过，原始
内核、initramfs 和软件包数据库保持不变，`sudo` 的 setuid 元数据保留，IA32
和 x64 EFI 文件已从输出 FAT 镜像中读回校验。尚未在 32 位 UEFI 平板上完成
Live 启动、安装、首次重启或后续内核/GRUB 更新测试。

新版桌面可能超出老 Atom 平板的内存或图形能力。无线、音频、显卡、触摸屏、
休眠、存储和电源管理都必须在目标硬件上单独验证。旧 Atom 驱动包不是长期
维护的内核驱动替代品。

## 测试命令

```sh
bash -n isorespin.sh
bash -n lib/isorespin/install-ia32
bash -n lib/isorespin/refresh-ia32
python3 -m unittest discover -s tests -v
```
