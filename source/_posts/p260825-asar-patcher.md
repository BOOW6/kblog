---
title: 在 Android 环境中为 Tyrano 游戏打补丁
date: 2026-08-25 02:10:24
tags:
---

为解决官方补丁工具仅提供 Windows 二进制、手机用户无法直接使用的痛点，我制作了 asar-patcher，让玩家可以在安卓手机上完成补丁安装，无需电脑。

asar-patcher 是一个在 Android（Termux）环境下对 Tyrano 引擎游戏 `app.asar` 数据包应用 `*.tpatch` 补丁的命令行工具。

工具核心是 `patch.sh` 脚本，它会自动识别当前系统架构（Android ARM64 或 Windows x64），调用 `asar` 和 `7za` 二进制文件执行解包、覆盖补丁、重新打包的完整流程。



## 工具组成

- `patch.sh` - 主脚本，负责解包、应用补丁、重新打包

- `tools/` - 存放不同平台的二进制文件

  - `linux-arm64-android/asar` - 用于解包/打包 asar
  
  - `linux-arm64-android/7za` - 7-Zip 的 ARM64 Android 版本
  
  - `windows-x64/asar.exe` 和 `7za.exe` - 对应 Windows x64 版本

所有二进制均已预编译，用户无需自行安装 Bun 或编译任何东西，开箱即用。

若想了解二进制构建方式，可参考项目 README 中的 Bun 编译命令。



## 适用场景

- Tyrano 引擎制作的视觉小说/冒险游戏

- 游戏数据包为 `app.asar`

- 补丁为 `*.tpatch`

- 运行于 Android ARM64 的设备 或 Windows x64 的设备



## 实操教程

以《ラブコメスイッチ》（恋爱喜剧突转）为例

本文以这款在 [ノベルゲームコレクション](https://novelgame.jp/games/show/14081?lang=JA) 发布的游戏为例，演示如何使用 asar-patcher 应用 [晓柯同文馆](https://www.kyoka.cn/p/lvcs/) 提供的汉化补丁。

同样的方法，也可以用于其它基于 Tyrano 引擎，并且能获取 app.asar 的游戏。

Termux 的使用需要一定 Linux Shell 基础，遇到不懂的可以善用搜索引擎/AI工具喵。



## 准备工作

| 项目 | 说明 |
|------|------|
| Android 手机 | 建议 Android 8.0+ |
| Termux | 从 F-Droid 安装（不建议从 Google Play 安装，版本过旧） |
| 存储空间 | 至少 4 GB 可用 |
| 游戏文件 | 原始 `app.asar`（从游戏安装目录获取） |
| 补丁文件 | `*.tpatch`（从汉化补丁发布页下载） |



## 详细步骤

### 1. 安装 Termux 并更新 (可选)

打开 Termux，执行：

```bash
pkg update && pkg upgrade -y
```



### 2. 授予 Termux 文件管理权限

执行：

```bash
termux-setup-storage
```

系统会弹出提示，点击“允许”。执行成功后，会出现 `~/storage` 目录，其中是许多个指向手机存储的软链接。

其中会用到 `~/storage/downloads`，它指向手机存储的 `Download` 目录。



### 3. 下载 asar-patcher

通过浏览器下载工具包，存放到手机存储的 `Download` 目录。

下载地址: [asar-patcher.zip - 蓝奏云](https://boow.lanzoum.com/iquYk44ef9sf)



### 4. 准备数据包和补丁

从游戏本体中取出 `app.asar`，位于：`parallellovecomedy_win.zip/resources/app.asar`

将汉化补丁 `parallellovecomedy.tpatch` 重命名为 `patch.zip`

将这两个文件存放到手机存储的 `Download` 目录。



### 5. 放置工具包、游戏数据包和补丁

将工具包复制到 Termux 并解压缩：

```bash
cp ~/storage/downloads/asar-patcher.zip .
pkg in 7zip
7z x asar-aptcher.zip
```

进入工具包目录：

```bash
cd asar-aptcher
```

将游戏的 `app.asar` 和 `patch.zip` 复制到 `asar-patcher` 目录下：

```bash
cp ~/storage/downloads/app.asar .
cp ~/storage/downloads/patch.zip .
```

最终工具包目录下文件应为：

```bash
ls
```

```
README.md  app.asar  patch.sh  patch.zip  tools
```



### 4. 执行补丁脚本

运行脚本：

```bash
./patch.sh
```

脚本将依次输出以下信息（示例）：

```
~/asar-patcher $ ./patch.sh
- pwd: /data/data/com.termux/files/home/asar-patcher
- whoami: u0_a288
- detected OS: Linux
- detected ARCH: aarch64
- extracting app.asar to workspace
- applying patch.zip to workspace

7-Zip (a) 26.02 (arm64) : Copyright (c) 1999-2026 Igor Pavlov : 2026-06-25
 64-bit arm_v:8-A locale=C.UTF-8 Threads:8 OPEN_MAX:32768, ASM

Scanning the drive for archives:

...

- re-packaging workspace to out/app.asar
~/asar-patcher $
```

### 5. 获取打补丁后的文件

补丁完成后，新的 `app.asar` 会生成在 `out/` 目录下，

将文件复制到手机存储的 `Download` 目录：

```bash
cp out/app.asar ~/storage/downloads/app.asar.out
```

将 `app.asar.out` 复制回游戏原目录，覆盖原始 `app.asar`（建议先备份原文件）。



### 6. 启动游戏

覆盖后，启动游戏即可享受汉化内容。



## 技术原理（简要）

`patch.sh` 的核心逻辑：

1. 检测操作系统和 CPU 架构，选择对应的工具目录

2. 调用 `asar e` 解包 `app.asar` 到 `workspace` 目录

3. 调用 `7za x` 将 `patch.zip` 解压到 `workspace`，自动覆盖同名文件（`-aoa` 参数）

4. 调用 `asar p` 将修改后的 `workspace` 重新打包为 `out/app.asar`



## 相关链接

- [asar-patcher.zip - 蓝奏云](https://boow.lanzoum.com/iquYk44ef9sf)

- [Termux - F-Droid](https://f-droid.org/zh_Hans/packages/com.termux/)

- [ラブコメスイッチ - ノベルゲームコレクション](https://novelgame.jp/games/show/14081?lang=JA)

- [恋爱喜剧突转 - 晓柯同文馆](https://www.kyoka.cn/p/lvcs/)



## 版权声明

本工具仅供学习交流，请勿用于商业用途。游戏版权归原作者所有，汉化补丁版权归汉化组所有。使用前请确保您是通过正版渠道获取游戏。


