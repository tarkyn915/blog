---
title: 'Windows跳过网络安装'
date: '2026-06-27T17:37:50+08:00'
draft: false
pinned: false
description: win跳过网络安装
tags: [win]
---

## 跳过网络安装

使用快捷键 `Shift + F10` 打开命令提示符窗口，部分笔记本电脑需要使用 `Fn + Shift + F10` 快捷键

```powershell
oobe\BypassNRO.cmd
```

回车重启选择 `我没有 Internet 连接`

## 创建本地账户

```powershell
start ms-cxh:localonly
```

