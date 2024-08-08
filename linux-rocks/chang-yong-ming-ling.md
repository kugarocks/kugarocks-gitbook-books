# 常用命令

## ENV

```
System: Ubuntu 22.04.4 LTS
Kernel: Linux 5.15.0-113-generic
Archit: x86-64
```

## MAN

该指令可以查看手册页中不同的章节（Section）。

```
$ man man
man - an interface to the system reference manuals

The table below shows the section numbers of the manual followed by the types of pages they contain.

1 Executable programs or shell commands
2 System calls (functions provided by the kernel)
3 Library calls (functions within program libraries)
4 Special files (usually found in /dev)
5 File formats and conventions, e.g. /etc/passwd
6 Games
7 Miscellaneous (including macro packages and conventions), e.g. man(7), groff(7)
8 System administration commands (usually only for root)
9 Kernel routines [Non standard]
```

### Section

```
man 1 ls/cp/mv
man 2 write/open/close
man 3 printf/malloc/free
man 4 tty/null/random
man 5 passwd/fstab
man 7 man
man 8 ifconfig/iptables/mount
man 9 fork # not work
```

### Games

`fortune` 可以随机的名言警句和笑话。

```
apt install fortune/fortune-mod/fortunes-zh
```

```
$ man 6 fortune
fortune - print a random, hopefully interesting, adage
```

可以结合 `cowsay` 和 `lolcat` 一起玩。

```
fortune | cowsay | lolcat
```

## LS

```
$ man ls
ls - list directory contents
```

### 文件详细信息

```
ls -lh
```

```
drwx------ 9 root root 4.0K   Aug  7 22:38 root
crw-rw-rw- 1 root tty  5,   0 Aug  7 22:33 /dev/tty
brw-rw---- 1 root disk 252, 0 Aug  7 21:30 /dev/vda
```

* 文件类型：`d` 目录，`-` 文件，`l` 链接，`c` 字符设备，`b` 块设备。
* 文件权限：700，Owner: 7，Group: 0, Other: 0
* 硬链接数：9
* 文件属主：root
* 文件组别：root
* 文件大小：4.0K
* 修改时间：Aug 7 22:38
* 文件名称：root
* 主设备号：Major Number，5/252，标识设备驱动程序
* 次设备号：Minor Number，0，标识同一类设备中的某个设备实例

### 只显示目录

```
ls -ld */
```

包括隐藏文件

```
ls -ld .*/ */
```

### 显示特定目录信息

```
ls -ld /usr
```

### 在文件后面添加类型符号

```
ls -lF /
```

* `/`：目录
* `*`：可执行文件
* `@`：符号链接
* `|`：命名管道
* `=`：套接字

## FILE

显示文件类型

```
$ man file
file — determine file type
```

### 字符/块设备

```
$ file /dev/tty
/dev/tty: character special (5/0)
```

```
$ file /dev/vda
/dev/vda: block special (252/0)
```

### 链接文件

注意结尾斜杆。

```
$ file /bin
/bin: symbolic link to usr/bin
```

```
$file /bin/
/bin/: directory
```

## CAT

```
$ man cat
cat - concatenate files and print on the standard output
```

### 显示行号 - 包括空行

```
cat -n FILE
```

### 显示行号 - 不包括空行

```
cat -b FILE
```

## LESS

`more` 命令只能按回车加载，`less` 命令更强大，因为 less is more。

```
$ man less
less - opposite of more
```

## USERMOD

```
$ man usermod
usermod - modify a user account
```

### 禁止账号登录

```
usermod -s /sbin/nologin kuga
```

此命令会在 `/etc/passwd` 中把 `bin/sh` 改为 `/sbin/nologin`

```
kuga:x:1000:1000::/home/kuga:/sbin/nologin
```

## LN

```
$ man ln
ln - make links between files
```

### 软链接
