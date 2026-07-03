---
title: 'Rime输入法'
date: '2026-06-29T15:46:03+08:00'
draft: false
pinned: false
description: rime
tags: [rime,输入法]
cover:
      image: rime.webp
      alt: "A sample cover image"
      caption: "rime"
---

## 安装 Rime

### Windows & macOS

直接下载可执行文件安装

https://rime.im/download/

### Linux

Fedora默认使用iBus框架，默认没有打包lua插件

```bash
sudo dnf install ibus-rime librime-lua -y
```

```bash
ibus restart
```

Arch fcitx5-im提供本体+配置工具+输入法模块

```bash
pacman -S fcitx5-im fcitx5-rime
```

## 安装 雾凇拼音

https://github.com/iDvel/rime-ice

手动安装，下载full包解压到下面配置目录

- iBus 为 `$HOME/.config/ibus/rime/`
- Fcitx5 为 `$HOME/.local/share/fcitx5/rime/`

## 输入法配置

[鼠须管输入法配置](https://www.hawu.me/others/2666#section_8)

[薄荷输入法](https://www.mintimate.cc/zh/)

待更新...
