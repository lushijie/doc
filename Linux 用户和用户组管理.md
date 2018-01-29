# Linux 用户和用户组管理

完成用户管理的工作有许多种方法，但是每一种方法实际上都是对有关的系统文件进行修改。与用户和用户组相关的信息都存放在一些系统文件中，这些文件包括/etc/passwd, /etc/shadow, /etc/group等

## 1. 用户组
### 1.1 新建用户组
	
groupadd 【选项】 用户组

选项：

	-g GID 指定新用户组的组标识号（GID）。
	-o 一般与-g选项同时使用，表示新用户组的GID可以与系统已有用户组的GID相同。
	
### 1.2 修改用户组
groupmod 【选项】用户组

选项：

	-g GID 为用户组指定新的组标识号。
	-o 与-g选项同时使用，用户组的新GID可以与系统已有用户组的GID相同。
	-n新用户组 将用户组的名字改为新名字

### 1.3 删除用户组

groupdel 用户组

## 2.用户
### 2.1 新建用户
	
useradd 【选项】用户名

选项：

	-c comment 指定一段注释性描述。
	-d 目录 指定用户主目录，如果此目录不存在，则同时使用-m选项，可以创建主目录。
	-g 用户组 指定用户所属的用户组。
	-G 用户组，用户组 指定用户所属的附加组。
	-s Shell文件 指定用户的登录Shell。
	-u 用户号 指定用户的用户号，如果同时有-o选项，则可以重复使用其他用户的标识号。

### 2.2 用户口令

账户刚创建的时候没有口令，但是被系统锁定没法使用，必须为其指定口令之后才可以使用，即便是指定空口令。

sudo passwd 【选项】 user1

选项：

	-l 锁定口令，即禁用账号。
	-u 口令解锁。
	-d 使账号无口令。
	-f 强迫用户下次登录时修改口令。	
	
### 2.3 修改账号

修改用户账号就是根据实际情况更改用户的有关属性，如用户号、主目录、用户组、登录Shell等，选项与useradd一致。

usermod 【选项】用户名

### 2.4 删除账号

userdel [-r] 用户名 

-r 会把用户目录一并删除


## 3. 登陆相关文件

### 3.1 /etc/passwd

	daemon:x:1:1:System daemons:/etc:
	用户名:口令:用户标识号:组标识号:注释性描述:主目录:登录Shell
	
这些用户在/etc/passwd文件中也占有一条记录，但是不能登录，因为它们的登录Shell为空。它们的存在主要是方便系统管理，满足相应的系统进程对文件属主的要求。

	伪用户含义 
	bin 拥有可执行的用户命令文件 
	sys 拥有系统文件 
	adm 拥有帐户文件 
	uucp UUCP使用 
	lp lp或lpd子系统使用 
	nobody NFS使用
	等

### 3.2 /etc/shadow

/etc/shadow中的记录行与/etc/passwd中的一一对应，它由pwconv命令根据/etc/passwd中的数据自动产生

### 3.3 /etc/group

将用户分组是Linux 系统中对用户进行管理及控制访问权限的一种手段。


## 4. 创建脚本

```sh
#!/bin/bash
# Script to add a user to Linux system
if [ $(id -u) -eq 0 ]; then
	read -p "Enter username : " username
	read -s -p "Enter password : " password
	egrep "^$username" /etc/passwd >/dev/null
	if [ $? -eq 0 ]; then
		echo "$username exists!"
		exit 1
	else
		pass=$(perl -e 'print crypt($ARGV[0], "password")' $password)
		useradd -m -p $pass $username
		[ $? -eq 0 ] && echo "User has been added to system!" || echo "Failed to add a user!"
	fi
else
	echo "Only root may add a user to the system"
	exit 2
fi
```
