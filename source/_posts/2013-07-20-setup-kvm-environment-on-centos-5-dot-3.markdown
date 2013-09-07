---
layout: post
title: "在CentOS 5.3 中搭建kvm虚拟机"
date: 2013-07-20 15:41
comments: true
categories: 
- kvm
- cloud
- virtulization
---

0.准备环境
---------

1.  确认cpu是否支持硬件虚拟化

        grep -o 'vmx\|svm' /proc/cpuinfo

    如果有输出，表示cpu可以支持。intel的cpu会输出vmx，amd的cpu输出svm.

<!-- more -->

2.  设置BIOS，开启虚拟化。

    平时修改BIOS的操作很容易，重启进BIOS即可。但由于现在是远程登录到server上,需要远程修改BIOS,可以按照如下操作进行.

    查看BIOS是否已经开启虚拟化选项:

        omreport chassis biossetup|grep Virtual

    设置BIOS开启虚拟化：

        omconfig chassis biossetup attribute=cpuvt setting=enabled

3.  关闭selinux

    修改/etc/sysconfig/selinux:
    
        SELINUX=disable

    重启server

1.安装kvm及启动服务
-------------------

1.  安装

        yum groupinstall KVM

2.  检查kvm module是否加载

        lsmod|grep kvm
    输出

        kvm_intel              85256  2 
        kvm                   225824  2 ksm,kvm_intel

    如果没有加载，可以手动加载
    
        modprobe kvm_intel

3. 开启libvirtd服务

        /etc/init.d/libvirtd start
        chkconfig libvirtd on #start at boot

2.配置网络桥接
--------------

1.  编辑/etc/sysconfig/network-scripts/ifcfg-eth0:

        DEVICE=eth0
        BOOTPROTO=none
        HWADDR=78:2B:CB:10:B3:14
        ONBOOT=yes
        TYPE=Ethernet
        NM_CONTROLLED=no
        BRIDGE=br0
2.  编辑/etc/sysconfig/network-scripts/ifcfg-br0:

        DEVICE=br0
        BOOTPROTO=static
        ONBOOT=yes
        TYPE=Bridge
        NETMASK=255.255.255.0
        IPADDR=192.168.6.88
        NM_CONTROLLED=no
        DELAY=0
3.  重启网络
    
        service network restart


3.安装guest
-----------

这里使用virt-manager远程安装一个虚拟机。virt-manager是一个图形化的工具，可以方便的安装、管理虚拟机。
    
1.  连接到server

    终端下执行：

        virt-manager -c qemu+ssh://username@server-host/system

2.  在弹出的图形界面中，按照向导一步一步安装即可。
    
    要求填写的虚拟机名字，也叫domain,在通过vir×工具管理时，会用的该名字。
    
    在选择网络的时候，选择我们配置的网桥.

3.  安装完之后会提示重启，进入虚拟机后，需要配置一下ip地址

4.其他工具
---------

以下工具都可以通过ssh远程使用,只要指定正确的URI:<code>qemu+ssh://username@server/system</code>

1.  virsh

    命令行的virt-manager

2.  virt-viewer

    可以直接连接到虚拟机的图形界面.例如：
        
        virt-viewer -c URI domain

3.  virt-install

    命令行的安装工具,需要配置virt-viewer使用。可以将已有的磁盘镜像加入到vir*工具的管理中


4.  virt-clone

    用于克隆虚拟机镜像,克隆后的镜像可以通过vir*工具管理

