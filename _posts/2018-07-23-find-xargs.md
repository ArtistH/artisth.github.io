---
layout: post
title: "Linux find and xargs"
date:   2018-07-23
tags:   Linux_System
---

# find [指定查找目录] [查找规则] [查找完后执行的action]

## [指定查找目录]
	shell> find /etc /home -name passwd
	/etc/pam.d/passwd
	/etc/default/passwd
	/etc/passwd
 
这里要注意的是目录之间要用空格分开

## [查找规则] 
#### 1. 根据文件名查找
	-name	// 根据文件名查找（精确查找）
	-iname	// 根据文件名查找, 但是不区分大小写.


> 这里介绍下文件名通配的知识
	
>> \* 表示通配任意的字符

>> 
	shell> find /etc -name "*passwd*"
	/etc/pam.d/chgpasswd
	/etc/pam.d/passwd
	/etc/pam.d/chpasswd
	/etc/default/passwd
	/etc/passwd
	/etc/passwd-

>> ? 表示通配任意的单个字符
>>
	shell> find /etc -name "passwd?"
	/etc/passwd-
 
>> [] 表示通配括号里面的任意一个字符
>>
	shell> find /tmp -name "[ab].sh"
	/tmp/a.sh
	/tmp/b.sh

#### 2. 根据文件所属用户和组来查找文件
	-user	// 根据属主来查找文件
	-group	// 根据属组来查找文件

#### 3. 根据uid和gid来查找用户
	shell> find /tmp -uid 500	// 查找uid是500的文件
	shell> find /tmp -gid 1000 	// 查找gid是1000的文件

#### 4.-a 和 -o 及 –not 的使用
> -a 连接两个不同的条件（两个条件必须同时满足）
>
	shell> find ./shell/ -name "*.sh" -a -user root
	./shell/bash/install_kernel.sh
	./shell/bash/commit.sh
	./shell/bash/schedule.sh
	./shell/bash/network.sh
	./shell/bash/clean_elf.sh
	./shell/bash/route_add.sh
	./shell/bash/lntree.sh
	./shell/bash/sysmonitor.sh

> -o 连接两个不同的条件（两个条件满足其一即可）  
> -not 对条件取反

#### 5. 根据文件时间戳的相关属性来查找文件，我们可以使用stat命令来查看一个文件的时间信息 如下：
	shell> stat /etc/passwd
	File: ‘/etc/passwd’
	Size: 1233      	Blocks: 8          IO Block: 4096   regular file
	Device: 801h/2049d	Inode: 2099704     Links: 1
	Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
	Context: system_u:object_r:passwd_file_t:s0
	Access: 2018-07-23 10:55:50.095999819 +0800
	Modify: 2018-06-05 19:36:38.189969141 +0800
	Change: 2018-06-05 19:36:38.191969141 +0800
	Birth: -

> -atime  
> -mtime  
> -ctime  
> -amin  
> -mmin  
> -cmin  

这里atime,mtime,ctime就是分别对应的“最近一次访问时间”，“最近一次内容修改时间”，“最近一次属性修改时间”，这里的time的单位指的是“天”，min的单位是分钟。

	shell> find /tmp –atime +5	// 表示查找在五天内没有访问过的文件
	shell> find /tmp -atime -5	// 表示查找在五天内访问过的文件

#### 6. 根据文件类型来查找文件(-type)
f	普通文件  
d	目录文件  
l	链接文件  
b	块设备文件  
c	字符设备文件  
p	管道文件  
s	socket文件  

	shell> find / -type s
	/run/dhcpd.unpriv.sock
	/run/dhcpd.sock
	/run/dbus/system_bus_socket


#### 7. 根据大小来查找文件(-size)
	shell> find /tmp -size 2M	// 查找在/tmp目录下等于2M的文件
	shell> find /tmp -size +2M	// 查找在/tmp目录下大于2M的文件
	shell> find /tmp -size -2M	// 查找在/tmp目录下小于2M的文件

#### 9. 根据文件权限查找文件(-perm)
	shell> find /tmp -perm 755	// 查找在/tmp目录下权限是755的文件
	shell> find /tmp -perm +222	// 表示只要有一类用户（属主，属组，其他）满足有写权限即可
	shell> find /tmp -perm -222	// 表示必须所有类别用户都满足有写权限

#### 10. -nouser 和 –nogroup
	shell> find / -nogroup –a –nouser	// 在整个系统中查找既没有属主又没有属组的文件（这样的文件通常是很危险的，我们应该及时清除掉）。

## [查找完执行的action]
> -print			默认情况下的动作  
> -ls				查找到后用ls 显示出来  
> -ok [command]		查找后执行命令的时候询问用户是否要执行  
> -exec [command]	查找后执行命令的时候不询问用户, 直接执行.  

	shell> find /tmp -name "*.sh" -exec chmod u+x {} \; 
这里要注意{}的使用：替代查找到的文件
 	
	shell> find /tmp -name "*.sh" -exec cp {} {}.old \;
	shell> ls /tmp
	a.sh a.sh.old b.sh b.sh.old

	shell> find /tmp -atime +30 –exec rm –rf {} \； // 删除查找到的超过30天没有访问过文件

我们也可以使用xargs来对查找到的文件进一步操作:

	shell> find /tmp -name "*.old" | xargs chmod 700



# xargs 
#### 管道:	将前面的标准输出作为后面的标准输入  
#### xargs:	将标准输入作为命令的参数

	shell> echo "--help" | cat  
	shell> echo "--help" | xargs cat

解释: 命令行输入cat且不加任何参数, 则cat会等待标准输入, 并将输入原样打印到标准输出.
xargs将内容作为普通的参数传递给cat, 相当于cat --help(cat会在标准输出打印其帮助文档).