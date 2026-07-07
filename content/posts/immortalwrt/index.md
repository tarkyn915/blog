---
title: 'ImmortalWrt 旁路由'
date: '2026-07-03T16:50:37+08:00'
draft: false
pinned: false
description: 为中国大陆优化的openwrt
tags: [immortalwrt,openwrt,旁路由,singbox]
cover:
      image: images.png
      alt: "A sample cover image"
      caption: "Immortalwrt"
---

## 下载固件

[官网下载](https://downloads.immortalwrt.org/)

默认有网页管理，只需要安装所需的软件就可以

> 如果是PVE需要先创建实例，然后用脚本添加镜像，上传的时候记得复制路径

```
qm importdisk xxx机器ID /var/lib/vz/template/iso/openwrt.img local-lvm
```



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

## 设置旁路由转发(TUN)

在设置前最好确认一下`firewall.@zone[0]` 是 LAN

`uci show firewall.@zone[0]`

```bash
# 1. 开启伪装 (NAT)：确保回程流量能走旁路由，解决“单向断网，网页打不开”的问题
uci set firewall.@zone[0].masq='1'

# 2. 开启 MTU 修复：自动调整加密数据包大小，解决“微信能发字，但网页图片加载不出来”的卡顿问题
uci set firewall.@zone[0].mtu_fix='1'

# 3. 允许LAN区域内部转发，允许流量从 LAN 进、再从 LAN 出
uci set firewall.@zone[0].forward='ACCEPT'

# 4. 关闭软件流量分载，避免透明代理/TUN 规则被绕过
uci set firewall.@defaults[0].flow_offloading='0'

# 5. 关闭硬件流量分载，避免连接不进 sing-box
uci set firewall.@defaults[0].flow_offloading_hw='0'

# 6. 保存并重启防火墙使配置生效
uci commit firewall
/etc/init.d/firewall restart
```

## 设置singbox

```bash
uci show sing-box

uci set sing-box.main.enabled='1'
uci set sing-box.main.config_file='/etc/sing-box/config.json'
uci commit sing-box
/etc/init.d/sing-box enable
/etc/init.d/sing-box restart
/etc/init.d/sing-box status
```

## 配置旁路由

设置设备网关和DNS指向旁路由即可

## Sing-box 模板

{{< collapse summary="**TUN**" >}}

```json
{
  "dns": {
    "servers": [
      {
        "tag": "local",
        "type": "udp",
        "server": "119.29.29.29"
      },
      {
        "tag": "public",
        "type": "https",
        "server": "dns.alidns.com",
        "domain_resolver": "local"
      },
      {
        "tag": "foreign",
        "type": "https",
        "server": "dns.google",
        "domain_resolver": "local"
      },
      {
        "tag": "fakeip",
        "type": "fakeip",
        "inet4_range": "198.18.0.0/15",
        "inet6_range": "fc00::/18"
      }
    ],
    "rules": [
      {
        "clash_mode": "direct",
        "server": "local"
      },
      {
        "clash_mode": "global",
        "server": "fakeip"
      },
      {
        "query_type": "HTTPS",
        "action": "reject"
      },
      {
        "rule_set": [
          "geosite-cn",
          "geosite-steamcn",
          "geosite-apple"
        ],
        "server": "local"
      },
      {
        "query_type": [
          "A",
          "AAAA"
        ],
        "server": "fakeip",
        "rewrite_ttl": 1
      }
    ],
    "final": "foreign",
    "strategy": "ipv4_only",
    "independent_cache": true
  },
  "outbounds": [
{ "tag": "🚀 默认代理", "type": "selector", "outbounds": ["🐸 手动选择", "♻️ 自动选择"] },
  { "tag": "🧠 AI", "type": "selector", "outbounds": ["🚀 默认代理", "🐸 手动选择", "♻️ 自动选择"] },
  { "tag": "📹 YouTube", "type": "selector", "outbounds": ["🚀 默认代理", "🐸 手动选择", "♻️ 自动选择"] },
  { "tag": "🍀 Google", "type": "selector", "outbounds": ["🚀 默认代理", "🐸 手动选择", "♻️ 自动选择"] },
  { "tag": "👨‍💻 Github", "type": "selector", "outbounds": ["🚀 默认代理", "🐸 手动选择", "♻️ 自动选择"] },
  { "tag": "📲 Telegram", "type": "selector", "outbounds": ["🚀 默认代理", "🐸 手动选择", "♻️ 自动选择"] },
  { "tag": "🎵 TikTok", "type": "selector", "outbounds": ["🚀 默认代理", "🐸 手动选择", "♻️ 自动选择"] },
  { "tag": "🎥 Netflix", "type": "selector", "outbounds": ["🚀 默认代理", "🐸 手动选择", "♻️ 自动选择"] },
  { "tag": "💶 PayPal", "type": "selector", "outbounds": ["🚀 默认代理", "🐸 手动选择", "♻️ 自动选择"] },
  { "tag": "🎮 Steam", "type": "selector", "outbounds": ["🚀 默认代理", "🐸 手动选择", "♻️ 自动选择"] },
  { "tag": "🪟 Microsoft", "type": "selector", "outbounds": ["🚀 默认代理", "🐸 手动选择", "♻️ 自动选择"] },
  { "tag": "🐬 OneDrive", "type": "selector", "outbounds": ["🚀 默认代理", "🐸 手动选择", "♻️ 自动选择"] },
  { "tag": "🍏 Apple", "type": "selector", "outbounds": ["🎯 全球直连", "🚀 默认代理", "🐸 手动选择", "♻️ 自动选择"] },
  { "tag": "🐠 漏网之鱼", "type": "selector", "outbounds": ["🚀 默认代理", "🎯 全球直连"] },
  { "tag": "🐸 手动选择", "type": "selector", "outbounds": ["节点1","节点2"] },
  { "tag": "♻️ 自动选择", "type": "urltest", "outbounds": ["节点1","节点2"], "interval": "10m", "tolerance": 100 },
  { "tag": "🍃 延迟辅助", "type": "urltest", "outbounds": ["🚀 默认代理", "🎯 全球直连"] },
  { "tag": "GLOBAL", "type": "selector", "outbounds": ["🚀 默认代理", "🧠 AI", "📹 YouTube", "🍀 Google", "👨‍💻 Github", "📲 Telegram", "🎵 TikTok", "🎥 Netflix", "💶 PayPal", "🎮 Steam", "🪟 Microsoft", "🐬 OneDrive", "🍏 Apple", "🐠 漏网之鱼", "🐸 手动选择", "♻️ 自动选择", "🍃 延迟辅助", "🎯 全球直连"] },
  { "tag": "🎯 全球直连", "type": "direct" },

      {
        "type":"vless",
        "tag":"节点1",
        "server":"xxxxx",
        "server_port":12345,
        "uuid":"xxxxxxx",
        "flow":"xtls-rprx-vision",
        "domain_resolver": {
        "server": "proxyDns",
        "rewrite_ttl": 60,
        "client_subnet": "8.8.8.8"
      },
        "tls":{
           "enabled":true,
           "server_name":"www.amd.com",
           "utls":{
              "enabled":true,
              "fingerprint":"chrome"
           },
           "reality":{
              "enabled":true,
              "public_key":"xxxxx",
              "short_id":"0123456789abcdef"
           }
        },
        "packet_encoding":"xudp",
        "multiplex":{
           "enabled":false
        }
      }


  ],
  "route": {
    "rules": [
      {
      "domain_suffix": [
        "auto.hylabpowered.com",
        "auto.hylab.im",
        "tjq.one"
      ],
      "outbound": "🎯 全球直连"
      },

      {
        "action": "sniff",
        "sniffer": [
          "http",
          "tls",
          "quic",
          "dns"
        ]
      },
      {
        "type": "logical",
        "mode": "or",
        "rules": [
          {
            "port": 53
          },
          {
            "protocol": "dns"
          }
        ],
        "action": "hijack-dns"
      },
      {
        "ip_is_private": true,
        "outbound": "🎯 全球直连"
      },
      {
        "clash_mode": "direct",
        "outbound": "🎯 全球直连"
      },
      {
        "clash_mode": "global",
        "outbound": "GLOBAL"
      },
      {
        "rule_set": "geosite-adobe",
        "action": "reject"
      },
      {
        "rule_set": "geosite-ai",
        "outbound": "🧠 AI"
      },
      {
        "rule_set": "geosite-youtube",
        "outbound": "📹 YouTube"
      },
      {
        "rule_set": "geosite-google",
        "outbound": "🍀 Google"
      },
      {
        "rule_set": "geosite-github",
        "outbound": "👨‍💻 Github"
      },
      {
        "rule_set": "geosite-onedrive",
        "outbound": "🐬 OneDrive"
      },
      {
        "rule_set": "geosite-microsoft",
        "outbound": "🪟 Microsoft"
      },
      {
        "rule_set": "geosite-apple",
        "outbound": "🍏 Apple"
      },
      {
        "rule_set": "geosite-telegram",
        "outbound": "📲 Telegram"
      },
      {
        "rule_set": "geosite-tiktok",
        "outbound": "🎵 TikTok"
      },
      {
        "rule_set": "geosite-netflix",
        "outbound": "🎥 Netflix"
      },
      {
        "rule_set": "geosite-paypal",
        "outbound": "💶 PayPal"
      },
      {
        "rule_set": "geosite-steamcn",
        "outbound": "🎯 全球直连"
      },
      {
        "rule_set": "geosite-steam",
        "outbound": "🎮 Steam"
      },
      {
        "rule_set": "geosite-!cn",
        "outbound": "🚀 默认代理"
      },
      {
        "rule_set": "geosite-cn",
        "outbound": "🎯 全球直连"
      },
      {
        "rule_set": "geoip-google",
        "outbound": "🍀 Google"
      },
      {
        "rule_set": "geoip-apple",
        "outbound": "🍏 Apple"
      },
      {
        "rule_set": "geoip-telegram",
        "outbound": "📲 Telegram"
      },
      {
        "rule_set": "geoip-netflix",
        "outbound": "🎥 Netflix"
      },
      {
        "rule_set": "geoip-cn",
        "outbound": "🎯 全球直连"
      }
    ],
    "rule_set": [
      {
        "tag": "geosite-adobe",
        "type": "remote",
        "format": "binary",
        "url": "https://github.com/qljsyph/ruleset-icon/raw/refs/heads/main/sing-box/geosite/adobe.srs",
        "download_detour": "🚀 默认代理"
      },
      {
        "tag": "geosite-ai",
        "type": "remote",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/sing/geo/geosite/category-ai-!cn.srs",
        "download_detour": "🚀 默认代理"
      },
      {
        "tag": "geosite-youtube",
        "type": "remote",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/sing/geo/geosite/youtube.srs",
        "download_detour": "🚀 默认代理"
      },
      {
        "tag": "geosite-google",
        "type": "remote",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/sing/geo/geosite/google.srs",
        "download_detour": "🚀 默认代理"
      },
      {
        "tag": "geosite-github",
        "type": "remote",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/sing/geo/geosite/github.srs",
        "download_detour": "🚀 默认代理"
      },
      {
        "tag": "geosite-onedrive",
        "type": "remote",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/sing/geo/geosite/onedrive.srs",
        "download_detour": "🚀 默认代理"
      },
      {
        "tag": "geosite-microsoft",
        "type": "remote",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/sing/geo/geosite/microsoft.srs",
        "download_detour": "🚀 默认代理"
      },
      {
        "tag": "geosite-apple",
        "type": "remote",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/sing/geo/geosite/apple.srs",
        "download_detour": "🚀 默认代理"
      },
      {
        "tag": "geosite-telegram",
        "type": "remote",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/sing/geo/geosite/telegram.srs",
        "download_detour": "🚀 默认代理"
      },
      {
        "tag": "geosite-tiktok",
        "type": "remote",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/sing/geo/geosite/tiktok.srs",
        "download_detour": "🚀 默认代理"
      },
      {
        "tag": "geosite-netflix",
        "type": "remote",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/sing/geo/geosite/netflix.srs",
        "download_detour": "🚀 默认代理"
      },
      {
        "tag": "geosite-paypal",
        "type": "remote",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/sing/geo/geosite/paypal.srs",
        "download_detour": "🚀 默认代理"
      },
      {
        "tag": "geosite-steamcn",
        "type": "remote",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/sing/geo/geosite/steam@cn.srs",
        "download_detour": "🚀 默认代理"
      },
      {
        "tag": "geosite-steam",
        "type": "remote",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/sing/geo/geosite/steam.srs",
        "download_detour": "🚀 默认代理"
      },
      {
        "tag": "geosite-!cn",
        "type": "remote",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/sing/geo/geosite/geolocation-!cn.srs",
        "download_detour": "🚀 默认代理"
      },
      {
        "tag": "geosite-cn",
        "type": "remote",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/sing/geo/geosite/cn.srs",
        "download_detour": "🚀 默认代理"
      },
      {
        "tag": "geoip-google",
        "type": "remote",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/sing/geo/geoip/google.srs",
        "download_detour": "🚀 默认代理"
      },
      {
        "tag": "geoip-apple",
        "type": "remote",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/sing/geo-lite/geoip/apple.srs",
        "download_detour": "🚀 默认代理"
      },
      {
        "tag": "geoip-telegram",
        "type": "remote",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/sing/geo/geoip/telegram.srs",
        "download_detour": "🚀 默认代理"
      },
      {
        "tag": "geoip-netflix",
        "type": "remote",
        "format": "binary",
        "url": "https://raw.githubusercontent.com/MetaCubeX/meta-rules-dat/sing/geo/geoip/netflix.srs",
        "download_detour": "🚀 默认代理"
      },
      {
        "tag": "geoip-cn",
        "type": "remote",
        "format": "binary",
        "url": "https://github.com/qljsyph/ruleset-icon/raw/refs/heads/main/sing-box/geoip/China-ASN-combined-ip.srs",
        "download_detour": "🚀 默认代理"
      }
    ],
    "final": "🐠 漏网之鱼",
    "auto_detect_interface": true,
    "default_domain_resolver": {"server": "public"}
  },
  "inbounds": [
    {
      "tag": "tun-in",
      "type": "tun",
      "address": [
        "172.19.0.1/30",
        "fdfe:dcba:9876::1/126"
      ],
      "mtu": 9000,
      "auto_route": true,
      "auto_redirect": true,
      "strict_route": true,
      "sniff": true,
      "sniff_override_destination": true
    }
  ],



    "experimental": {
      "clash_api": {
        "external_controller": "0.0.0.0:9090",
        "external_ui": "ui",
        "secret": "",
        "external_ui_download_url": "https://github.com/MetaCubeX/Yacd-meta/archive/gh-pages.zip",
        "external_ui_download_detour": "🚀 默认代理",
        "default_mode": "rule"
      }
    },
  
  
  "log": {
    "disabled": false,
    "level": "info",
    "timestamp": true
  }
}
```

{{</ collapse >}}
