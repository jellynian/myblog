---
layout: post
title: "openstack kali linux 镜像制作"
tagline: "sysadmin"
description: ""
category:
key: openstack-kali-image
tags: [openstack,sysadmin]
last_updated: 2018年 2月7日 星期一 13时26分20秒 CST
---

# 概述

> 公司网络安全部门需要在我司私有云平台上搭建kali linux的云主机，用以进行渗透测试等服务，在互联网上搜索了很久也没有发现有现成的镜像可以使用，因此就自己制作了,镜像下载地址:[http://jellynian.ufile.ucloud.com.cn/kali_xfce.qcow2](http://jellynian.ufile.ucloud.com.cn/kali_xfce.qcow2), 镜像默认用户`debian`默认密码`kalidebian` 在此附上具体的制作教程。

# 环境准备

操作系统: ubuntu操作系统（作者使用的是ubuntu16.04 主机IP为：10.0.3.102）
到任意一个镜像站下载kali linux的镜像，这里使用的是南阳理工学院的开源镜像
[https://mirror.nyist.edu.cn/kali-images/](https://mirror.nyist.edu.cn/kali-images/)

    # 安装virsh  libvirt-bin
    apt-get install libvirt-bin qemu-kvm  qemu-utils -y
    # 下载基础镜像(存储目录为 /var/lib/libvirt/)
    wget  https://mirror.nyist.edu.cn/kali-images/current/kali-linux-xfce-2018.1-amd64.iso \ 
	-O /var/lib/libvirt/kali-linux-xfce-2018.1-amd64.iso 

   
# 创建img 磁盘
执行以下命令创建qcow2磁盘

	qemu-img create -f qcow2 /var/lib/libvirt/kali.qcow2 10G   # 由于kali预装了大量软件，经测试 10G的基础磁盘大小是较为合适的，
	#作者曾经测试过2G，3G，5G等大小都失败了（或许是作者的测试方式有问题，您可以做更多的尝试）
   
# 开始制作

      virt-install --virt-type kvm --name kali --ram 1024 \
        --cdrom=/var/lib/libvirt/kali-linux-xfce-2018.1-amd64.iso \
        --disk /var/lib/libvirt/kali.qcow2 \
        --network network=default \
        --graphics vnc,listen=0.0.0.0 --noautoconsole \
        --os-type=linux --os-variant=debianwheezy


## 使用vnc客户端连接vnc

在vnc客户端直接输入宿主机的IP 即可连接,按照正常的流程安装此虚拟机，不过要注意的是磁盘分区不要使用lvm分区，将所有文件放入一个分区即可，使用整个磁盘,因为openstack在创建虚拟机时默认是自动分区，剩余的磁盘空间可以自动的加入此分区，如果你选择了lvm或者分了多个分区可能会导致自动分区无法正常工作(此为本人测试后的结果，可能存在一定的疏漏)。安装完成后,镜像会关机退出，我们使用 `virsh list --all` 来查看当前运行的所有机器，并使用`virsh start kali` 来启动该机器并再次使用vnc连接

	root@cagetest:~# virsh list --all
	 Id    Name                           State
	----------------------------------------------------
	 -     kali                           shut off

	root@cagetest:~# virsh start kali
	Domain kali started


## 安装 cloud-init
cloud-init 脚本将在虚拟机启动的时候搜寻元数据服务获取公钥。公钥将会放在镜像默认用户内。
安装 cloud-init 软件包：

    
    # apt-get install cloud-init

### 配置数据源

在创建 Ubuntu 镜像时，cloud-init 必须明确的配置元数据源。OpenStack 元数据服务仿效 Amazon EC2 元数据服务。
运行 `dpkg-reconfigure` 命令设置镜像 cloud-init 软件包使用的元数据源。当屏幕出现提示时，选择 EC2 数据源。

    # dpkg-reconfigure cloud-init

### 配置账户
不同的发行版保存的账户不同，在基于 Ubuntu 的虚拟机，账户是 ubuntu ，基于 Fedora 的虚拟机，账户是 ec2-user ,基于debian的虚拟机账户是debian，因此kali的cloud-init安装后默认的用户是debian。

你可以编辑 /etc/cloud/cloud.cfg 文件修改 cloud-init 使用的账户名，例如，添加以下这行到配置文件，公钥将会存放到debian用户下。

	default_user:
		name: debian

### 配置镜像源
修改 `/etc/cloud/cloud.cfg` 中的配置，这里改为我母校南阳理工学院的镜像源 
![](/img/sourcelist.png)


 关闭虚拟机
在虚拟机内，以root用户运行：

    # /sbin/shutdown -h now

# 清理（删除 MAC 地址相关信息）

操作系统会在`/etc/sysconfig/network-scripts/ifcfg-eth0` 和 `/etc/udev/rules.d/70-persistent-net.rules` 这类文件记录下网卡MAC地址，但是，虚拟机的网卡MAC地址在每次虚拟机创建的时候都会不同，因此这些信息必须从配置文件删除掉。

目前有 virt-sysprep 工具可以完成清理虚拟机镜像内的 MAC 地址相关的信息。

	# apt-get install libguestfs-tools  #你可能需要先安装这个安装包
    # virt-sysprep -d kali

# 删除 libvirt 虚拟机域

现在你可以上传虚拟机镜像到镜像服务了，所以不再需要 libvirt 来管理虚拟机镜像，使用 `virsh undefine vm-image` 命令来完成。

    # virsh undefine kali

# 镜像准备完成
前面你使用 qemu-img create 命令创建的镜像已经准备好可以上传了，你可以上传 `/var/lib/libvirt/kali.qcow2` 文件到 Openstack 镜像服务。

