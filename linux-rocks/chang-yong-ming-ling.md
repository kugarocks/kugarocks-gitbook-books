# 常用命令

## man

该指令可以查看手册页中不同的章节（Section）。

```
$ man man
man - an interface to the on-line reference manuals

The table below shows the section numbers of the manual followed by the types of pages they contain.

1   Executable programs or shell commands
2   System calls (functions provided by the kernel)
3   Library calls (functions within program libraries)
4   Special files (usually found in /dev)
5   File formats and conventions eg /etc/passwd
6   Games
7   Miscellaneous (including macro packages and conventions), e.g. man(7), groff(7)
8   System administration commands (usually only for root)
9   Kernel routines [Non standard]
```

### **可执行程序或 Shell 命令**

```
man 1 ls/cp/mv
```

### **系统调用**

```
man 2 write/open/close
```

### **库调用**

```
man 3 printf/malloc/free
```

### 特殊文件

```
man 4 tty/null/random
```

### 文件格式与规约

```
man 5 passwd/fstab
```

### **游戏**

`fortune` 可以随机的名言警句和笑话。

```
yum install fortune-mod
```

```
man 6 fortune
```

### 杂项

```
man 7 man
```

### 系统管理命令

```
man 8 ifconfig/iptables/mount
```

### 内核例程

```
man 9 fork
```

## ls

```
$ man ls
ls - list directory contents
```

### 只显示目录

```
ll -d */
```

包括隐藏文件

```
ll -d .*/ */
```

### 显示特定目录信息

```
ll -d /usr
```

### 在文件后面添加类型符号

```
ll -F /
```

* `/`：目录
* `*`：可执行文件
* `@`：符号链接
* `|`：命名管道
* `=`：套接字
