---
layout:     post
title:      "Linux + Busybox + gdb 远程调试"
subtitle:   "qemu 编译"
date:       2022-12-03
author:     "Mld"
header-img: "img/post-bg-js-version.jpg"
tags:
    - kernel
    - qemu
    - x86
---

终于还是来到了 Linux 内核的山脚下, 潜力执行的第一步 果然还是搭建一个顺手的环境吧😂

# 1. 环境准备 

作为一个对系统环境有着某种洁癖的人, 多少有点难以容忍日常使用的操作系统被折腾的乱糟糟的, 

各种配置文件和临时文件到处都是, 想删又不敢删, 留着又碍事,

 尤其要命的是, 有些软件安装失败之后 就会陷入装装不上, 卸卸不掉的境地.

所以我把实验环境选在了虚拟机, 不仅不用担心污染日常使用的操作系统, 换电脑时还能随时打包带走, 很方便.

**虚拟机: kvm 管理工具: libvirt**

**系统: debian11 安装方式: [官方云镜像](https://cloud.debian.org/images/cloud/bullseye/latest/debian-11-nocloud-amd64.qcow2)**

> 选型原因: 
>
> 1. 相比于vmware和virtualbox, kvm 和 libvirt都是Linux系统下原生的开源软件, 没有版权和费用问题(当然对于用惯了盗版软件的我们 这好像不构成问题)  
> 2. libvirt 和 qemu 提供了一整套命令行操作的工具, 远程SSH连接也能完成实验
> 3. 为什么不是ubuntu? 我试了, 但build内核的时候各种奇奇怪怪的问题, debian下异常流畅, 可能是ubuntu下软件包的版本太激进了吧😂
> 4. debian11的iso安装过程很麻烦, 直接用官方的云镜像就可以了

----
废话好多 作废作废



