---
title: 'Terraria游戏服搭建'
date: '2026-07-11T01:31:12+08:00'
draft: false
pinned: false
description: 泰拉瑞亚服务器
tags: [游戏服,terraria]
cover:
      image: tar.avif
      alt: "A sample cover image"
      caption: "Terraria"
---

## 原版服

在 [官网](https://terraria.org/) 底部下载游戏开服文件，给文件夹上权限

启动 `./TerrariaServer` 会自动创建地图文件夹，推荐在PC上创建一个地图上传 `/root/.local/share/Terraria` 重新启动即可

## Tmod服

![Owner avatar](https://avatars.githubusercontent.com/u/36521970?s=48&v=4) [tModLoader](https://github.com/tModLoader/tModLoader) 下载需要的版本即可

地图和mod目录 `/.local/share/Terraria/tModLoader`

和原版一样需要先启动一次 `./start-tModLoaderServer.sh`

正常情况下会自动处理NET组件，在PC创建地图，导出mod上传

## Tshock 服务器

[TShock](https://github.com/Pryaxis/TShock) 下载对应系统的包，给777权限，先运行 `TShock.Installer` ，他会自动下载dotnet NET组件，可能会提示安装`libicu`  Debian13 安装 `libicu76`

添加环境变量 `export DOTNET_ROOT=/opt/test/ts/dotnet` ，之后就可以用 `TShock.Server` 

> TShock 6.1版本前缀不正常的问题，修改config `"EnableChatAboveHeads": false,` 

开服之后控制台会出现 `/set xxxx` 使用之后会临时获得最高权限，可以用来创建服主账号 `/user add test 123456 owner` 但是owner默认没有权限，如果要管理服务器可以切换成 superadmin

### 插件

[插件收集仓库](https://tr.monika.love/resources/104/)

插件目录 `ServerPlugins` 下载插件把dll文件放入即可

[TShockWorldModify](https://github.com/hufang360/TShockWorldModify)

修改地图属性

```
/wm help           查看帮助
/wm mode           修改世界难度
/wm name <世界名>   修改世界名字
/npc help
```

### Tshock多开

需要复制两份文件

然后修改一下配置文件的 游戏端口 和 REST API 端口

配置文件目录 `Tshock/tshock/config.json`

### 常见问题

强制开荒
编辑 `sscconfig.json` 开了之后就需要用login登录才能移动 除了管理之外 物品都由服务器记录了

给自己物品
`/item ID` 

切换用户的组
`user group xxxx superadmin`

   > {{< collapse summary="**更改前缀名称和颜色**" >}}

```
前缀+颜色
/group prefix guest [未登录]
/group prefix default [[c/00FFFF:玩家]]
/group prefix newadmin [[c/89C4E9:副管理]]
/group prefix admin [[c/FAF0A0:管理员]]
/group prefix GM [[c/ed1941:超][c/f47920:管]]
/group prefix vip [[c/FAF0A0:物管]]
/group prefix owner [[c/FEA550:服][c/878AE5:主]]

聊天颜色
/group color default 045,235,203
/group color newadmin 045,235,203
/group color admin 193,223,186
/group color GM 193,223,186
/group color vip 193,223,186
/group color owner 193,223,186
```

{{</ collapse >}}

   > {{< collapse summary="**config.json**" >}}

```json
{
"InvasionMultiplier": 1,//入侵规模，计算公式：入侵怪物数量=100+（X*HP＞200的玩家）
"DefaultMaximumSpawns": 5,//怪物刷新最大数值（设置越高怪物越多）
"DefaultSpawnRate": 600,//刷新怪物时间间隔，数值越大刷新越慢
"ServerPort": 7777,//服务器端口【默认】
"EnableWhitelist": false,//是否开启白名单【true代表是，false代表否，以下都是】
"InfiniteInvasion": false,//是否无限制怪物入侵【开启后使用命令召唤的怪物入侵将达到200万只左右】
"PvPMode": "normal",//PVP模式【normal表示可以正常使用PVP，always表示强制PVP，disabled表示强制PVE】
"SpawnProtection": false,//是否保护出生点【强烈建议设置】
"SpawnProtectionRadius": 10,//出生点保护范围【一格一个】
"MaxSlots": 6,//服务器人数上限
"RangeChecks": true,//不明
"DisableBuild": false,//是否禁止建筑【开启后将无法破坏地图里任何东西】
"SuperAdminChatRGB": [
255.0,
0.0,
0.0
],//GameMaster发言颜色【设置与人物初始设置颜色数值相同】
"SuperAdminChatPrefix": "[GM]",//GameMaster发言前缀（位于名字之前）
"SuperAdminChatSuffix": "~~",//GameMaster发言后缀（位于名字之后）
"BackupInterval": 0,//地图备份时间间隔/分钟
"BackupKeepFor": 60,//备份的地图保留时间
"RememberLeavePos": false,//记录离开位置，再次登录服务器将传送到上一次离开服务器的位置
"HardcoreOnly": false,//仅允许困难模式的玩家进入服务器
"MediumcoreOnly": false,//仅允许中等模式的玩家进入服务器
"KickOnMediumcoreDeath": false,//移除（kick）死亡的中等难度的玩家
"BanOnMediumcoreDeath": false,//封禁（ban）死亡的中等难度的玩家
"AutoSave": true,//是否自动保存地图，强烈建议开启
"AnnounceSave": true,//自动保存的时候是否进行提示
"MaximumLoginAttempts": 3,//登录次数尝试最大限制，尝试次数过多将被移除（kick）服务器
"RconPassword": "",//没用就对了，建议不要改动
"RconPort": 7777,//同上
"ServerName": "Terraria Small Team",//服务器名称
"UseServerName": true,//是否使用服务器名称
"MasterServer": "127.0.0.1",//本机IP连接地址，改动后自己可能进不去服务器
"StorageType": "sqlite",//数据库类型，建议不要改动
"MySqlHost": "localhost:3306",//下面的都没用，建议不要改动
"MySqlDbName": "",
"MySqlUsername": "",
"MySqlPassword": "",
"MediumcoreBanReason": "Death results in a ban",//中等难度的玩家被封禁（ban）时的理由
"MediumcoreKickReason": "Death results in a kick",//中等难度的玩家被移除（kick）时的理由
"EnableDNSHostResolution": false,//不明，大概和网络有关
"EnableIPBans": true,//是否可以封禁（ban）ip地址
"EnableUUIDBans": true,//是否开启封禁（ban）UUID
"EnableBanOnUsernames": false,//是否可以封禁（ban）用户名
"DefaultRegistrationGroupName": "default",//注册用户的默认用户组【如不了解组的规划请暂时不要改动】
"DefaultGuestGroupName": "guest",//未注册用户的默认用户组
"DisableSpewLogs": true,//禁止将服务器日志展示给玩家
"HashAlgorithm": "sha512",//不明
"BufferPackets": true,//不明，大概和buff有关
"ServerFullReason": "Sorry,you can`t into the server,because the Server is full",//因服务器人满而被拒绝进入服务器的提示
"WhitelistKickReason": "You are not on the whitelist.",//因不在白名单而被拒绝进入服务器的提示
"ServerFullNoReservedReason": "Server is full. No reserved slots open.",//因服务器人满并预留给管理员的位置也满的情况下被拒绝进入服务器的提示
"SaveWorldOnCrash": true,//服务器崩溃时是否及时保存地图
"EnableGeoIP": false,//显示玩家IP的所在地【有可能侵犯他人隐私，建议不要开启】
"EnableTokenEndpointAuthentication": false,//不明QAQ
"RestApiEnabled": false,//上同下
"RestApiPort": 7878,//呵呵
"DisableTombstones": true,//是否移除墓碑
"DisplayIPToAdmins": false,//是否将玩家的IP地址展示给管理员
"KickProxyUsers": true,//移除（kick）使用外挂的玩家
"DisableHardmode": false,//禁止让世界进入困难模式（即肉山后）
"DisableDungeonGuardian": false,//禁止攻打地牢守护者，与old man对话将会被立即传送到出生点
"DisableClownBombs": false,//禁止小丑往出生点扔炸弹【大概】
"DisableSnowBalls": false,//禁止使用雪球
"ChatFormat": "{1}{2}{3}: {4}",//聊天格式【{1}为前缀，{2}为玩家名称，{3}为后缀，{4}为聊天内容】
"ChatAboveHeadsFormat": "{2}",//在玩家头顶显示的内容【参考上一条】
"ForceTime": "normal",//THE WORLD！【normal表示昼夜正常交替，day表示出现极昼现象，night表示出现极夜现象】
"TileKillThreshold": 60,//一秒挖掘，破坏物块的上限，否则将被冻结【可以用来检测外挂，下同】
"TilePlaceThreshold": 20,//一秒摆放物块的上限
"TileLiquidThreshold": 2,//一秒释放液体的上限
"ProjectileThreshold": 50,//一秒使用弹药数量的上限【包括魔法攻击】
"ProjIgnoreShrapnel": true,//计算弹药使用上限是否忽略爆炸产生的碎片
"RequireLogin": false,//是否开启强制注册登录
"DisableInvisPvP": false,//PVP状态下是否使隐身药水失效
"MaxRangeForDisabled": 10,//被冻结后最大移动距离
"ServerPassword": "",//服务器的密码，不设置表示无密码
"RegionProtectChests": true,//领地内的箱子是否受到保护，PVE服务器强烈建议设置成true
"DisableLoginBeforeJoin": false,//大概意思是踢出登录失败的玩家？
"DisableUUIDLogin": false,//是否禁止UUID登录
"KickEmptyUUID": false,//是否移除（kick）空UUID的玩家
"AllowRegisterAnyUsername": false,//是否允许注册任何用户名，PVE服务器强烈建议设置成false
"AllowLoginAnyUsername": true,//是否允许登录任何用户名
"MaxDamage": 175,//玩家所受到的最大伤害点数，超过这个数值会被冻结
"MaxProjDamage": 175,//玩家受到弹药的最大伤害点数，同上
"IgnoreProjUpdate": false,//不明，下同
"IgnoreProjKill": false,//下同
"IgnoreNoClip": false,//下不同
"AllowIce": true,//是否禁止冰的扩散【这啥？】
"AllowCrimsonCreep": false,//是否允许血腥之地扩散，PVE强烈建议设置成false，下同
"AllowCorruptionCreep": false,//是否允许腐化之地扩散
"AllowHallowCreep": false,//是否允许神圣之地扩散
"StatueSpawn200": 3,//不明，但是可能和出生点有关，建议不要改动，下同
"StatueSpawn600": 6,//赞成上一条
"StatueSpawnWorld": 10,//雕像召唤物品的最高上限
"PreventBannedItemSpawn": false,//是否禁止用item指令和give指令获得被封禁（ban）掉的物品
"PreventDeadModification": true,//不明真相的吃瓜群众
"EnableChatAboveHeads": false,//楼上带我一个
"ForceXmas": false,//是否开启圣诞节
"AllowAllowedGroupsToSpawnBannedItems": false,//是否允许有权限使用被封禁（ban）物品的用户组使用被封禁（ban）的物品
"IgnoreChestStacksOnLoad": false,//加载地图的时候是否检测箱子里物品堆叠上线
"LogPath": "tshock",//日志文件存放路径
"PreventInvalidPlaceStyle": true,//不明
"BroadcastRGB": [
127.0,
255.0,
212.0
],//系统广播颜色，和上述GameMaster的设置颜色方式一样
"RestUseNewPermissionModel": true,//是否刷新的时候使用新的模特【？】
"ApplicationRestTokens": {},//不明
"ReservedSlots": 3,//预留给管理员的通道数量
"LogRest": false,//日志是否刷新【？】
"RespawnSeconds": 3,//玩家死亡后复活时间/秒
"TilePaintThreshold": 15,//一秒刷漆上限
"EnableMaxBytesInBuffer": false,//不懂
"MaxBytesInBuffer": 5242880,//还是不懂
"ForceHalloween": false,//是否开启万圣节
"AllowCutTilesAndBreakables": false,
"CommandSpecifier": "/"//指令标志，在聊天框里首位输入该符号视为指令
}
```

{{</ collapse >}}

