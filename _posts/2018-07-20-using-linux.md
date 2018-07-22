---
layout: post
title: "Using Linux System"
date:   2018-07-20
tags:   Linux_System
---

## useful cmd
    1) apropos timer    列出所有timer相关函数
        出现问题:
            timer: nothing appropriate.
        解决方法:
            Install man-pages
            shell> mandb
	
    2) ulimit
        shell> ulimit -a 用来显示当前的各种用户进程限制

        我们在用这个命令的时候主要是为了产生core文件, 就是程序运行发行段错误时的文件:
        shell> ulimit -c unlimited

    3) type 命令名
        查看命令类型
        内部命令(buildin): shell本身已实现此功能(cd, pwd, echo等)
        外部命令: 根据存在的可执行文件来执行

	4) man -a
		显示匹配查询名的所有信息

	5) history -c 
		清空历史记录

	6) grep
		grep -r 递归
		grep -n 显示行号
		grep -l 显示文件
		grep -i 忽略大小写
		grep -v 取反查询
	
	7) MKNOD(1)
        NAME
            mknod - make block or character special files

        SYNOPSIS
            mknod Name { b | c } Major Minor
            mknod Name { p }
            创建FIFO(已命名的管道)

        DESCRIPTION
            建立一个目录项和一个特殊文件的对应索引节点. 第一个参数是Name项设备的名称, 选择一个描述性的设备名称.

            mknod命令有两种形式, 它们有不同的标志.

            mknod命令的第一种形式只能由root用户或系统组成员执行.
            在第一种形式中, 使用了b或c标志.
            b标志表示这个特殊文件是面向块的设备(磁盘, 软盘或磁带).
            c标志表示这个特殊文件是面向字符的设备(其它设备).
            第一种形式的最后两个参数是指定主设备的数目, 它帮助操作系统查找设备驱动程序代码和指定次设备的数目,
            也就是单元驱动器或行号, 它们是十进制或八进制的.
            一个设备的主要和次要编号由该设备的配置方法分配, 它们保存在ODM中的CuDvDr类里.
            在这个对象类中定义了主要和次要编号以确保整个系统设备定义的一致性, 这是很重要的.

            在mknod命令的第二种形式中, 使用了p标志来创建 FIFO(已命名的管道).

        标志
            b 表示特殊文件是面向块的设备(磁盘, 软盘或磁带). 
            c 表示特殊文件是面向字符的设备(其它设备). 
            p 创建FIFO(已命名的管道). 

        示例
            要创建一个新软盘驱动器的特殊文件, 输入:
            mknod /dev/fd2 b 1 2

            这创建了/dev/fd2特殊文件, 它是一个特殊块文件, 主设备号为1, 次设备号为2.


## Troubleshooting
### MBR
	在总共512字节的主引导扇区中, MBR只占用了其中的446个字节.
	另外的64个字节交给了DPT（Disk Partition Table硬盘分区表).
	最后两个字节"55, AA"是分区的结束标志.
	这个整体构成了硬盘的主引导扇区。
	(64/16=4个分区, 因为每16字节可以标记一个分区, 所以一个硬盘最多4个主分区.)

首先备份: dd < /dev/sda > filename bs=446 count=1<br>
还原: dd < filename > /dev/sda bs=446 count=1<br>
(破坏: dd < /dev/zero > /dev/sda bs=446 count=1)

排错:
1) stage1出错<br>
	症状: Boot failed<br>
	解决: 光盘或USB启动 linux rescue, chroot /mnt/sysimage, grub-install /dev/sda.<br>
2) stage2出错<br>
	症状: 显示GRUB<br>
	解决: 同1)<br>
3) grub.conf出错<br>
	症状: 显示grub><br>
	解决: 手工启动修改grub.conf


## 正则表达式
	.	任意单个字符
	*	任意几个字符
	^	^word 以word开头
	$	word$ 以word结尾
	\<	\<word 以word开头的单词(严格)
	\>	word\> 以word结尾的单词(严格)
