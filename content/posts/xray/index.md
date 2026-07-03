---
title: 'Xray内核 + Reality模板'
date: '2026-06-28T01:08:48+08:00'
draft: false
pinned: false
description: xray手搓
tags: [xray,proxy,reality]
---

## 安装Xray

https://github.com/XTLS/Xray-install

```bash
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
```

> 用于在支持 systemd 的操作系统

## 默认配置文件

使用`xray uuid`  `xray x25519` 生成

```
/usr/local/etc/xray/config.json
```



   > {{< collapse summary="**VLESS-Vision-REALITY**（防偷模板）" >}}

```json
{
    "log": {
        "loglevel": "debug"
    },
    "inbounds": [
        {
            "listen": "127.0.0.1",
            "tag": "dokodemo-in",
            "port": 54321, // 任意空闲端口，如果要自己修改记得这里和下面的 reality 入站 dest 要同步修改
            "protocol": "dokodemo-door",
            "settings": {
                // reality dest 的实际设置位置 和普通 reality 要求相同
                "address": "www.amd.com",
                "port": 443,
                "network": "tcp"
            },
            "sniffing": { // 这里的 sniffing 不是多余的，别乱动
                "enabled": true,
                "destOverride": [
                    "tls"
                ],
                "routeOnly": true
            }
        },
        {
            "listen": "0.0.0.0",
            "port": 54321,
            "protocol": "vless",
            "settings": {
                "clients": [
                    {
                        "id": "", //xray uuid
                        "flow":"xtls-rprx-vision"
                    }
                ],
                "decryption": "none"
            },
            "streamSettings": {
                "network": "tcp",
                "security": "reality",
                "realitySettings": {
                    // 指向前面设置的 dokodemo-door 入站
                    "dest": "www.amd.com:443",
                    // serverNames 仍然需要在这里正常填写
                    "serverNames": [
                        "www.amd.com"
                    ],
                    "privateKey": "", // 运行 `xray x25519` 生成
                    "shortIds": [
                        "",
                        "0123456789abcdef"
                    ]
                }
            },
            "sniffing": {
                "enabled": true,
                "destOverride": [
                    "http",
                    "tls",
                    "quic"
                ],
                "routeOnly": true
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "freedom",
            "tag": "direct"
        },
        {
            "protocol": "blackhole",
            "tag": "block"
        }
    ],
    "routing": {
        "rules": [
            {
                "inboundTag": [
                    "dokodemo-in"
                ],
                // 重要，这个域名列表需要和 realitySettings 的 serverNames 保持一致
                "domain": [
                    "www.amd.com"
                ],
                "outboundTag": "direct"
            },
            {
                "inboundTag": [
                    "dokodemo-in"
                ],
                "outboundTag": "block"
            }
        ]
    }
}
```



   {{</ collapse >}}



## Xray-examples

https://github.com/XTLS/Xray-examples/tree/main/
