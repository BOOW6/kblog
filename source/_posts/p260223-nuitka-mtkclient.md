---
title: 记一次利用 Nuitka 编译打包 MTKClient
date: 2026-02-23 18:32:07
tags:
    - "Python"
---

记录使用 Nuitka 编译、打包 MTKClient 的 `mtk_gui.py` 为 amd64 架构 Windows 系统上的可执行文件，在 Windows 11, Python 3.13 下进行。

## 0 开始前

首先要获取 MTKClient 并安装，可以按照这里来：

[bkerler/mtkclient/README-INSTALL.zh-CN.md - GitHub](https://github.com/bkerler/mtkclient/blob/main/README-WINDOWS.zh-CN.md)

建议在虚拟环境中进行操作：

```shell
python -m venv env
env/Scripts/activate
```

了解 Nuitka 的使用方法或遇到问题，在用户文档中查找信息：

[User Documentation - Nuitka the Python Compiler](https://nuitka.net/user-documentation/)

## 1 正题

### 1.1 安装 Nuitka

```shell
pip install nuitka zstandard imageio
```

`zstandard` 用于在编译 onefile 时压缩文件。

`imageio` 指定 .png 图片作为图标时需要。

验证安装：

```shell
python -m nuitka --version
```

简单地编译一个 Python 程序为 onefile ：

```shell
python -m nuitka --onefile program.py
```

### 1.2 编译 MTKClient

直接用前面的指令来编译 MTKClient 显然是不可行的，我们还要解决依赖问题，以及指挥 Nuitka 将数据文件进行打包。

```bat
python -m nuitka ^
--output-dir=out --mode=onefile --lto=yes ^
--enable-plugin=pyside6 --windows-icon-from-ico=./mtkclient/gui/images/logo_512.png ^
--file-description=https://github.com/bkerler/mtkclient --product-version=2.1.3 --file-version=1.0 ^
--user-package-configuration-file=my.nuitka-package.config.yml mtk_gui.py
```

该命令指定使用 onefile 模式进行编译并打包为单文件，启用 LTO 优化。由于 MTKClient 使用了 PySide6 ，需要启用对应的插件。另外对文件属性进行调整，指定文件图标。利用 `my.nuitka-package.config.yml` 配置打包。

由于已经有 zstd 用于压缩资源，就不需要再开启 upx 插件，否则很可能起到反效果。

编译产生的文件将位于文件夹 out 中，mtk_gui.exe 是最终打包好的可执行文件。mtk_gui.dist 中的文件则对应程序运行会前释放到临时文件夹的文件，程序结束后它们会被删除。

新建文件 `my.nuitka-package.config.yml` ，用于告诉 Nuitka 哪些文件需要打包。

```yaml
- module-name: 'mtkclient'
  data-files:
    - dest_path: '.'
    - dirs:
      - 'mtkclient'
      - 'mtkclient/config'
      - 'mtkclient/gui/images'
      - 'mtkclient/Library/Exploit/test'
      - 'mtkclient/Library/Filesystem/bin'
      - 'mtkclient/Loader'
      - 'mtkclient/payloads'
      - 'mtkclient/Windows'
    - patterns:
      - '**/*.dll'
      - '**/*.bin'
      - '**/*.bin*'
      - '**/*.img'
      - '**/*.txt'
      - '**/*.png'
      - '**/*.ico'
```

## 2 完成

就那么用。这是我编译打包的 mtk_gui.exe 。

[mtkclient_mtk_gui_v2.1.3.zip - 蓝奏云](https://boow.lanzoum.com/i6jSL3i9elwd)
