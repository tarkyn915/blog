---
title: 'ImmortalWrt 旁路由'
date: '2026-07-03T16:50:37+08:00'
draft: false
pinned: false
description: 为中国大陆优化的openwrt
tags: [immortalwrt,openwrt,旁路由]
cover:
      image: images.png
      alt: "A sample cover image"
      caption: "Immortalwrt"
---

## 下载固件

[官网下载](https://downloads.immortalwrt.org/)

默认有网页管理，只需要安装所需的软件就可以

## 设置静态IP和DNS

```BASH
# 1. 配置 LAN 口为静态 IP 模式
uci set network.lan.proto='static'
uci set network.lan.ipaddr='10.0.0.100'
uci set network.lan.netmask='255.255.255.0'
uci set network.lan.gateway='10.0.0.1'

# 2. 清理并重新指定 LAN 口的 DNS
uci delete network.lan.dns
uci add_list network.lan.dns='10.0.0.1'
uci commit network

# 3. 关闭 LAN 口的 DHCP 服务（避免与主路由冲突）
uci set dhcp.lan.ignore='1'
uci commit dhcp

# 4. 重启网络与 DNS 服务使配置生效
/etc/init.d/network restart
/etc/init.d/dnsmasq restart
```

## 安装软件

按需下载即可

```
# homeproxy
luci-app-homeproxy
# passwall
luci-i18n-passwall-zh-cn
# sing-box
sing-box
# 网页终端
luci-i18n-ttyd-zh-cn
# sftp服务端
openssh-sftp-server
# 主题
luci-theme-argon
```

## 配置旁路由

网络 - 防火墙 - lan - wan IP动态伪装 开启

设置设备网关和DNS指向旁路由即可

如果要使用singbox-tun 那么就要设置路由转发

## 设置旁路由转发

```bash
# 1. 开启伪装 (NAT)：确保回程流量能走旁路由，解决“单向断网，网页打不开”的问题
uci set firewall.@zone[0].masq='1'

# 2. 开启 MTU 修复：自动调整加密数据包大小，解决“微信能发字，但网页图片加载不出来”的卡顿问题
uci set firewall.@zone[0].mtu_fix='1'

# 3. 允许流量转发：给旁路由下发“中转通行证”，允许流量从 LAN 进、再从 LAN 出
uci set firewall.@zone[0].forward='ACCEPT'

# 4. 保存并重启防火墙使配置生效
uci commit firewall
/etc/init.d/firewall restart
```

## 设置singbox

```bash
uci show sing-box

uci set sing-box.main.enabled='1'
uci set sing-box.main.config_file='/etc/sing-box/config.json'
uci commit sing-box
/etc/init.d/sing-box restart
/etc/init.d/sing-box status
```



