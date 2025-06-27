+++
authors = ["Jun Wu"]
title = "国内程序员科学网络配置指南 - OpenWRT"
date = "2025-06-26"
description = "a guide to cross gfw."
tags = [
    "proxy"
]
categories = [
    "infra",
    "devops"
]
series = ["proxy"]
+++

> openwrt 是目前性价比最高的科学上网的解决方案，但凡你用过了一个 OpenWRT 的路由器作为你办公的路由器，你不会再执着于在客户端使用 Clash/Surge/ShadowRocket 了。

先立意说明一下本文到底要指南的是什么。

1. 让你的 docker pull 可以拉取官方镜像仓库，而不是拉取国内蹩脚的 阿里云/腾讯云/科大的镜像
2. 可以让你访问 Claude/Google/ChatGPT 可以根据你的喜好选取代理，虽然所有的科学代理客户端软件都支持这个功能，但是如果你配置在路由器上，你就不需要在你所有的 电脑/iPad/手机上多处配置了
3. 可以让你的虚拟机/docker 容器可以访问 GitHub/Google 等海外资源，你一定遇到过你的电脑可以访问海外资源，但是你的代码在 docker 容器里面也需要访问的时候，你就需要让你的容器走你的电脑的代理去访问，这个配置过程很多时间会让人抓狂
4. 当你熟练使用了 OpenWRT 的诸多功能之后，你甚至可以作为企业网管，统一管理所有员工的电脑的上网行为，并且为开发技术人员提供科学上网的功能，但是对行政和人事部门禁止他们访问海外资源，这一点对于一些敏感业务的企业是一个很值得借鉴的方向


这里我需要分情况分类指导你如何使用 OpenWRT 来配置你的工作网络

### 1. 我手上没有可以刷 OpenWRT 的设备，但是我想体验一下

恭喜你，本文将是你入坑的最佳实践，这里假设你使用的是 Windows 的电脑作为你的办公电脑，假设你是 MacOS 用户，请发邮件给我，超过 10 个读者有需求我也可以做教程，另外 Linux 用户我觉得您不需要这篇教程，因为 Linux 用户必然已经解决了科学上网的问题，如果没有你也可以借鉴这篇教程，因为只要你能创建一个虚拟机和一个虚拟交换机，大部分配置是一模一样的。


#### 准备工作 - Windows 上开启 HypverV


![1](/images/guides/openwrt-for-engineer/1.png)


![2](/images/guides/openwrt-for-engineer/2.png)


#### 在线编译 OpenWRT X86 架构的镜像

打开 https://openwrt.ai/?target=x86%2F64&id=generic
1.注册/登录之后，架构选择 Generic x86/64
2.定制软件包如下图一样选择，附上两个内核组件 kmod-tun kmod-ipt-nat

![3](/images/guides/openwrt-for-engineer/3.png)


3.其他选项如下图一样选择，注意后台地址一定不能与你电脑目前的路由器 IP 冲突，例如很多家庭常用的路由器是 192.168.1.1 或者是 10.0.0.1 这些常见的 IP，你可以改成 192.168.20.1 这种就不会冲突

![4](/images/guides/openwrt-for-engineer/4.png)

4. 点击构建新固件, 等待 2 分钟即可下载

![5](/images/guides/openwrt-for-engineer/5.png)

5. 将 kwrt-09.20.2024-x86-64-generic-squashfs-combined-efi.img.gz 解压成文件夹，该文件夹里面只有一个 img 文件 kwrt-09.20.2024-x86-64-generic-squashfs-combined-efi.img, 注意它文件的命名是根据你当前的日期决定的


#### 下载镜像转换器 Starwindsoftware V2V Converter / P2V Migrator

1.https://www.starwindsoftware.com/starwind-v2v-converter
2.点击下载，它会问你要邮箱地址和一些信息，提供之后，它会发送下载连接给你的邮箱
3.去邮箱点击下载就会下载好 starwindconverter.exe
4.双击安装 starwindconverter.exe，之后如下图找到安装之后的 convertor

![6](/images/guides/openwrt-for-engineer/6.png)

5.启动之后，按照下面的步骤一步一步将 img 文件转化为 vhd 文件，这里面的逻辑就是将 efi img 镜像转化为 windows hyper-v 的 vhd 文件

![7](/images/guides/openwrt-for-engineer/7.png)

8.选择你刚才解压过的 img 文件

![8](/images/guides/openwrt-for-engineer/8.png)

![9](/images/guides/openwrt-for-engineer/9.png)

9.转化成为 local file


![10](/images/guides/openwrt-for-engineer/10.png)

10.下一步之后如下选择

![11](/images/guides/openwrt-for-engineer/11.png)

11.下一步默认

![12](/images/guides/openwrt-for-engineer/12.png)

![13](/images/guides/openwrt-for-engineer/13.png)

#### 正式开始创建 openwrt 虚拟机

1.打开 Hyper-V Manager

![14](/images/guides/openwrt-for-engineer/14.png)

2.管理虚拟交换机

![15](/images/guides/openwrt-for-engineer/15.png)

3.选择 Internal -> Create Virtual Switch, 取名随意，推荐取名叫做 virtual-lan

![16](/images/guides/openwrt-for-engineer/16.png)

![17](/images/guides/openwrt-for-engineer/17.png)

4.再创建一个新的 Virtual Switch，如下图

![18](/images/guides/openwrt-for-engineer/18.png)

#### 从 VHD 文件创建虚拟机

![19](/images/guides/openwrt-for-engineer/19.png)

1.输入虚拟机的名字，这里你随意

![20](/images/guides/openwrt-for-engineer/20.png)

2.下一步选择 Generation 1(中文：第一代)

![21](/images/guides/openwrt-for-engineer/21.png)

3.这里的资源分配，内存给 1G 绝对够了，没有哪个 openwrt 的系统需要吃掉 512 的内存，256 都能跑




### 2. 我拥有一台可以刷 OpenWRT 的路由器

关于如何购买指导，我推荐一个网站 [猫点饭](https://mao.fan/)，你可以选取一款【支持刷机的】路由器，然后自行购买。

