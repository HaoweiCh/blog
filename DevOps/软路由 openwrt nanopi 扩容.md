---
title: 软路由 openwrt nanopi 扩容
description: 软路由 openwrt nanopi 扩容
tags:
- NanoPi
- NanoPi R2S
- OpenWRT
- FriendlyElec
categories:
- DevOps
date: 2022-03-01T20:26:44+08:00
year: 2022
week: 9
updated: 2022-03-01T20:26:44+08:00
---

![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/CA519FBD6EE41EFF0669B55E35321D30CD894BB3.webp)

<!-- more -->

### 步骤

#### 1. 检查当前设备状态

```shell
# 查看
#   首先查看下当前系统的磁盘名称
fdisk -l 
```

![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/11DE01872F6033E03451D195F8FB9F835FC61326.webp)


#### 2. 安装操作工具

`opkg update && opkg install cfdisk`

#### 3. 挂载 & 再次检查

```shell
cfdisk /dev/mmcblk0
# 检查
fdisk -l
```

上下方向键移动到空磁盘

![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/1EB318CC108E3E3A72139E77D5D8939A53662BBB.webp)

#### 4. 格式化新盘（ext4 格式）

```shell
mkfs.ext4 /dev/mmcblk0p3 
```

选择容量->选择 Primary->Write

![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/2D7BD9CE7B5FECF34444CABA412F5DF90E442DAB.webp)

#### 5. 挂载 (挂载在后台可视化)

```shell
mkdir /mnt/mmcblk0p3
mount /dev/mmcblk0p3  /mnt/mmcblk0p3
```

![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/4E020D6C2B5997C0EBF17200D2BDDAF09D0D4B98.webp)

![](https://cdn.jsdelivr.net/gh/HaoweiCh/imgs/CA519FBD6EE41EFF0669B55E35321D30CD894BB3.webp)