---
layout: post
title: "Linux find and xargs"
date:   2018-07-23
tags:   Linux_System
---

## find [指定查找目录] [查找规则] [查找完后执行的action]

### [指定查找目录]
	shell> find /etc /home -name passwd
	/etc/pam.d/passwd
	/etc/default/passwd
	/etc/passwd
 
这里要注意的是目录之间要用空格分开

### [查找规则] 
#### 1. 根据文件名查找
	-name	// 根据文件名查找(精确查找)
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
> -a 连接两个不同的条件(两个条件必须同时满足)
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

> -o 连接两个不同的条件(两个条件满足其一即可)  
> -not 对条件取反

#### 5. 根据文件时间戳的相关属性来查找文件, 我们可以使用stat命令来查看一个文件的时间信息 如下: 
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

这里atime,mtime,ctime就是分别对应的"最近一次访问时间", "最近一次内容修改时间", "最近一次属性修改时间", 这里的time的单位指的是"天", min的单位是分钟.

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
	shell> find /tmp -perm +222	// 表示只要有一类用户(属主, 属组, 其他)满足有写权限即可
	shell> find /tmp -perm -222	// 表示必须所有类别用户都满足有写权限

#### 10. -nouser 和 –nogroup
	shell> find / -nogroup –a –nouser	// 在整个系统中查找既没有属主又没有属组的文件(这样的文件通常是很危险的, 我们应该及时清除掉).

### [查找完执行的action]
> -print			默认情况下的动作  
> -ls				查找到后用ls 显示出来  
> -ok [command]		查找后执行命令的时候询问用户是否要执行  
> -exec [command]	查找后执行命令的时候不询问用户, 直接执行. 

	shell> find /tmp -name "*.sh" -exec chmod u+x {} \; 
这里要注意{}的使用: 替代查找到的文件
 	
	shell> find /tmp -name "*.sh" -exec cp {} {}.old \;
	shell> ls /tmp
	a.sh a.sh.old b.sh b.sh.old

	shell> find /tmp -atime +30 –exec rm –rf {} \; // 删除查找到的超过30天没有访问过文件

我们也可以使用xargs来对查找到的文件进一步操作:

	shell> find /tmp -name "*.old" | xargs chmod 700



## xargs 
### xargs概述
#### 管道:	将前面的标准输出作为后面的标准输入  
#### xargs:	xargs命令是给其他命令传递参数的一个过滤器, 也是组合多个命令的一个工具. xargs能够处理管道或者stdin并将其转换成特定命令的命令参数. xargs也可以将单行或多行文本输入转换为其他格式, 例如多行变单行, 单行变多行. xargs的默认命令是echo, 空格是默认定界符. 这意味着通过管道传递给xargs的输入将会包含换行和空白, 不过通过xargs的处理, 换行和空白将被空格取代. xargs是构建单行命令的重要组件之一. 

	shell> echo "--help" | cat  
	shell> echo "--help" | xargs cat

解释: 命令行输入cat且不加任何参数, 则cat会等待标准输入, 并将输入原样打印到标准输出.
xargs将内容作为普通的参数传递给cat, 相当于cat --help(cat会在标准输出打印其帮助文档).



### xargs命令用法
#### 1. xargs用作替换工具, 读取输入数据重新格式化后输出. 

定义一个测试文件, 内有多行文本数据: 

	shell> cat test.txt
	a b c d e f g
	h i j k l m n
	o p q
	r s t
	u v w x y z

多行输入单行输出: 

	shell> cat test.txt | xargs
	a b c d e f g h i j k l m n o p q r s t u v w x y z

-n选项多行输出: 

	shell> cat test.txt | xargs -n3
	a b c
	d e f
	g h i
	j k l
	m n o
	p q r
	s t u
	v w x
	y z

-d选项可以自定义一个定界符: 

	shell> echo "nameXnameXnameXname" | xargs -dX
	name name name name

结合-n选项使用: 

	shell> echo "nameXnameXnameXname" | xargs -dX -n2
	name name
	name name

#### 2. 读取stdin, 将格式化后的参数传递给命令

假设一个命令为 sk.sh 和一个保存参数的文件arg.txt: 

	#!/bin/bash
	#sk.sh命令内容, 打印出所有参数. 

	echo $*

arg.txt文件内容: 

	shell> cat arg.txt
	aaa
	bbb
	ccc

xargs的一个选项-I, 使用-I指定一个替换字符串{}, 这个字符串在xargs扩展时会被替换掉, 当-I与xargs结合使用, 每一个参数命令都会被执行一次: 

	shell> cat arg.txt | xargs -I {} ./sk.sh -p {} -l
	-p aaa -l
	-p bbb -l
	-p ccc -l

复制所有图片文件到 /data/images 目录下: 

	shell> ls *.jpg | xargs -n1 -I cp {} /data/images

## xargs结合find使用
在使用 find命令的-exec选项处理匹配到的文件时,  find命令将所有匹配到的文件一起传递给exec执行. 但有些系统对能够传递给exec的命令长度有限制, 这样在find命令运行几分钟之后, 就会出现溢出错误. 错误信息通常是"参数列太长"或"参数列溢出". 这就是xargs命令的用处所在, 特别是与find命令一起使用.   
find命令把匹配到的文件传递给xargs命令, 而xargs命令每次只获取一部分文件而不是全部, 不像-exec选项那样. 这样它可以先处理最先获取的一部分文件, 然后是下一批, 并如此继续下去.   


### 使用实例
1) 用rm 删除太多的文件时候, 可能得到一个错误信息: /bin/rm Argument list too long. 用xargs去避免这个问题: 

	shell> find . -type f -name "*.log" -print0 | xargs -0 rm -f

	xargs -0将\0作为定界符. 

2) 统计一个源代码目录中所有php文件的行数: 

	shell> find . -type f -name "*.php" -print0 | xargs -0 wc -l
	
3) 查找所有的jpg 文件, 并且压缩它们: 

	shell> find . -type f -name "*.jpg" -print | xargs tar -czvf images.tar.gz

3) 假如你有一个文件包含了很多你希望下载的URL, 你能够使用xargs下载所有链接: 

	shell> cat url-list.txt | xargs wget -c
