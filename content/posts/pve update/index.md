---
title: '解决Pve更新报错'
date: '2026-07-08T11:41:43+08:00'
draft: false
pinned: false
description: pve更新报错
tags: [pve]
cover:
      image: pveupdate.png
      alt: "A sample cover image"
      caption: "pve更新报错"
---

## 关闭默认的企业源

PVE默认会启动企业源，所以自动更新会报错

`nano /etc/apt/sources.list.d/pve-enterprise.sources`

```
Types: deb
URIs: https://enterprise.proxmox.com/debian/pve
Suites: trixie
Components: pve-enterprise
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
#在最后一行加上
Enabled: yes
```

## 创建一个免费无订阅仓库

`nano /etc/apt/sources.list.d/pve.sources`

>需要注意版本 9.0 >> trixie

```
Types: deb
URIs: http://download.proxmox.com/debian/pve
Suites: trixie
Components: pve-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
```

## 修改Ceph的软件仓库配置

`nano /etc/apt/sources.list.d/ceph.sources`

```
Types: deb
URIs: http://download.proxmox.com/debian/ceph-squid
Suites: trixie
Components: no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
```

`apt update && apt full-upgrade`

