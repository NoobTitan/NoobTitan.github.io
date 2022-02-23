---
title: Ubuntu系统备份与还原
date: 2022-02-23 16:38:28
tags:
- 系统
categories: 
- Ubuntu系统相关
---

# Ubuntu系统备份与还原

## 1.备份

- 当前使用的系统

```
sudo su
cd /
tar czvpf sys_backup_$(date "+%Y%m%d-%H%M%S").tar.gz --exclude=/proc --exclude=/lost+found   --exclude=/mnt --exclude=/sys  --exclude=/media --exclude=/home/vetec/.cache --exclude=/sys_backup_$(date "+%Y%m%d-%H%M%S").tar.gz /
# 这一步可能会提示’tar: Error exit delayed from previous errors’，忽略即可
```

- 当前路径的系统

```
sudo su
tar czvpf /sys_backup_$(date "+%Y%m%d-%H%M%S").tar.gz --exclude=./proc --exclude=./lost+found   --exclude=./mnt --exclude=./sys --exclude=./home/fengji/.ros --exclude=./home/fengji/.config/Code --exclude=./media --exclude=./home/fengji/.cache --exclude=./sys_backup_$(date "+%Y%m%d-%H%M%S").tar.gz ./
```

以上为系统打包，排除了大部分旧系统中冗余数据。

```
#打包完成后使用此命令查看精确大小
du -h backup.tar.gz
```

## 2.重装系统

使用U盘启动盘安装ubuntu16或者ubuntu20系统，安装系统可以分出512MB的efi分区，和/根分区。安装完成后进入U盘系统进行系统恢复。

## 3.系统恢复

进入临时系统，把新系统中的grub.cfg和fstab取出保存，再把name.tar.gz复制进入系统盘/目录下

```
sudo su
sudo cp /media/(Ubuntu)/boot/grub/grub.cfg ./
sudo cp /media/(Ubuntu)/etc/fstab ./
cp /media/(U盘)/backup.tgz ./
sudo tar xvpfz backup.tgz -C ./
sudo mkdir proc lost+found mnt sys media cdrom
```

用新系统中的UUID替换老系统中的grub.cfg和fstab的UUID，其中fstab根据分区情况更改
vim替换:`%s/老UUID/新UUID/gc`
如果硬件相同，那么已经可以进入新系统了。

## 4.实操汇总

1. 首先备份旧系统ubuntu16为sys_backup_20211214-102530.tar.gz，使用该系统覆盖新电脑新系统，只能使用内核4.15进入系统，此时nouveau禁用，N卡驱动需要重装，有线网卡和无线网卡为最新硬件，没有有效驱动，无法ubuntu16中运行。尝试更换5.10的内核，成功安装内核后重启无法进入系统（无错误黑屏），安装5.11内核出现问题，旧系统依赖版本过低，无法成功安装，而ubuntu16无法升级所需依赖。
2. 更换新旧电脑的无线网卡，新电脑可以在ubuntu16，内核4.15的条件下，使用旧电脑无线网卡上网，但其有线网卡仍然无法识别，随后再次重装成功升级内核为5.4+，但仍然无法安装有线网卡驱动，该次重装优化了旧系统备份，卸载了N卡驱动，修复了apt问题，同样禁用nouveau，打包为sys_backup_20211215-141023.tar.gz
3. 最后换回新旧电脑的无线网卡，安装ubuntu20.04.3, 内核版本为5.11，发现有线网卡能正常使用，无线网卡在解决驱动问题后也能正常使用。但在新系统ubuntu20下，原先代码环境不兼容，需要调整。
4. 修复210无线网卡
```
# fix 210 wifi
sudo mv iwlwifi-ty-a0-gf-a0.pnvm  iwlwifi-ty-a0-gf-a0.pnvm.bak
mv /lib/firmware/iwlwifi-ty-a0-gf-a0.pnvm /lib/firmware/iwlwifi-ty-a0-gf-a0.pnvm.bak
```
