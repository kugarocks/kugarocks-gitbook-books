# Systemd

## Systemd

```
ps aux | grep systemd
```

```
root  1  0.0  0.2 191052  4092 ?  Ss  Jul19  0:10 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
```

systemd 进程号为 1，主要功能是系统初始化、进程管理和日志记录。名称后面加 `.d` 是 Unix 守护进程的命名规范。[System D](https://en.wikipedia.org/wiki/System\_D) 也是一个术语，表示快速思考和解决问题的能力。它是 2010 年诞生的，在这之前用的是 SysVinit。

## SysVinit

Unix System V 这种初始化方法现在已经不怎么用了，但在一些旧的发行版中还能见到。

### Runlevel

runlevel 也是这个 SysVinit 的产物，在 CentOS 中可以通过下面的方式查询 runlevel。

```
runlevel
```

```
N 3
```

3 表示 `multi-user.target`，N 表示上一次的 runlevel 为 No，下面是 who 的例子。

```
who -r
```

```
run-level 3  2024-07-19 15:10
```

`/etc/inittab` 文件定义了系统的默认运行级，看注释已经不再用了。

```
cat /etc/inittab
```

```
# inittab is no longer used when using systemd.
#
# ADDING CONFIGURATION HERE WILL HAVE NO EFFECT ON YOUR SYSTEM.
#
# Ctrl-Alt-Delete is handled by /usr/lib/systemd/system/ctrl-alt-del.target
#
# systemd uses 'targets' instead of runlevels. By default, there are two main targets:
#
# multi-user.target: analogous to runlevel 3
# graphical.target: analogous to runlevel 5
#
# To view current default target, run:
# systemctl get-default
#
# To set a default target, run:
# systemctl set-default TARGET.target
#
```

### rc.d

rc 是 run commands 的缩写，d 是目录的意思，目的是为了避免单一配置文件与包含它们的目录之间的命名冲突。另外以前 `/etc/init` 和 `/etc/init.d` 在同一个目录，现在 `/etc/init` 已经搬到 `/sbin/init`，而且你会发现他指向的是 systemd。

```
ll /sbin/init
```

```
lrwxrwxrwx 1 root root 22 Jun 28 12:29 /sbin/init -> ../lib/systemd/systemd
```

`/etc/rc.d` 这个目录包含了特定运行级下启动的进程，例如运行级 3 对应 `/etc/rc.d/rc3.d`。

## Unit Files

单元文件是 systemd 的配置文件，用于定义和管理系统服务、设备、挂载点、套接字和其他系统资源。每个单元文件都描述了 systemd 如何启动、停止和监视相关资源。

### 基本分类

* **服务单元（Service Unit）**：用于定义系统服务，扩展名为 `.service`。
* **挂载单元（Mount Unit）**：用于定义文件系统挂载点，扩展名为 `.mount`。
* **设备单元（Device Unit）**：用于定义设备，扩展名为 `.device`。
* **套接字单元（Socket Unit）**：用于定义套接字激活的服务，扩展名为 `.socket`。
* **计时器单元（Timer Unit）**：用于定义定时任务，扩展名为 `.timer`。
* **目标单元（Target Unit）**：用于定义系统的目标状态，扩展名为 `.target`。

### 所在目录

* `/etc/systemd/system/`：系统管理员定义的单元文件。
* `/usr/lib/systemd/system/`：发行版提供的单元文件。
* `/run/systemd/system/`：运行时生成的单元文件。

### Nginx 单元文件

```
ll /etc/systemd/system/multi-user.target.wants/nginx.service
```

```
...nginx.service -> /usr/lib/systemd/system/nginx.service
```

```
cat /usr/lib/systemd/system/nginx.service
```

```
# 定义了服务的描述和依赖关系
[Unit]
Description=The nginx HTTP and reverse proxy server
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

# 定义了如何启动、停止和重启服务
[Service]
# forking 表示服务在启动时会创建子进程，父进程会退出
Type=forking
# 存储主进程的 ID
PIDFile=/run/nginx.pid
# Nginx will fail to start if /run/nginx.pid already exists but has the wrong
# SELinux context. This might happen when running `nginx -t` from the cmdline.
# https://bugzilla.redhat.com/show_bug.cgi?id=1268621

# 启动准备，删除 pid 文件
ExecStartPre=/usr/bin/rm -f /run/nginx.pid
# 启动准备，测试 nginx 配置
ExecStartPre=/usr/sbin/nginx -t
# 启动命令
ExecStart=/usr/sbin/nginx
# 重载配置命令
ExecReload=/usr/sbin/nginx -s reload
# 停止服务时发送给主进程的信号
KillSignal=SIGQUIT
# 终止服务的等待时间，超过 5 秒则强制终止
TimeoutStopSec=5
# 终止进程的方式，仅终止主进程，不会影响子进程
KillMode=process
# 创建独立的临时目录，/tmp 和 /var/tmp
PrivateTmp=true

# 如何安装和启用服务
[Install]
WantedBy=multi-user.target
```

### Nginx 状态

```
systemctl status nginx
```

```
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2024-07-24 16:21:13 CST; 4 days ago
  Process: 23494 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 23491 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 23488 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 23496 (nginx)
   CGroup: /system.slice/nginx.service
           ├─23496 nginx: master process /usr/sbin/nginx
           ├─23665 nginx: worker process
           └─23666 nginx: worker process

Jul 24 16:21:13 systemd[1]: Stopped The nginx HTTP and reverse proxy server.
Jul 24 16:21:13 systemd[1]: Starting The nginx HTTP and reverse proxy server...
Jul 24 16:21:13 nginx[23491]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Jul 24 16:21:13 nginx[23491]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Jul 24 16:21:13 systemd[1]: Started The nginx HTTP and reverse proxy server.
```

### 常用命令

* `systemctl start [unit]`：启动单元。
* `systemctl stop [unit]`：停止单元。
* `systemctl enable [unit]`：启用单元，使其在引导时自动启动。
* `systemctl disable [unit]`：禁用单元。
* `systemctl status [unit]`：查看单元状态。
* `systemctl daemon-reload`：在修改或添加单元文件后重新加载 systemd 配置。

## Targets

从上面可以看到，target 是一种单元文件的类型，和一般单元文件不同，它代表的是系统当前的运行状态，target 定义了该状态下需要启动哪些进程和服务，所以 target 可以包含多个单元文件。用下面的命令可以查看当前系统的 target。

```
systemctl get-default
```

```
multi-user.target # 多用户模式，不包含图形界面
```

### multi-user.target

target 文件通常存储在 `/usr/lib/systemd/system/` 或 `/etc/systemd/system/` 目录中。每个 target 文件都包含关于该 target 的依赖关系和启动顺序的信息。我们可以看一下 `multi-user.target` 这个文件内容。

```
cat /usr/lib/systemd/system/multi-user.target
```

```
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit]
Description=Multi-User System
Documentation=man:systemd.special(7)
Requires=basic.target
Conflicts=rescue.service rescue.target
After=basic.target rescue.service rescue.target
AllowIsolate=yes
```

下面两个目录包含了 `multi-user.target` 状态所需要的内容。

```
/etc/systemd/system/multi-user.target.wants/
/usr/lib/systemd/system/multi-user.target.wants/
```

上面的是用户定义配置，下面的是系统默认配置

* 用户定义配置：优先级更高，同一服务 systemd 会优先执行。
* 系统默认配置：系统或软件包更新时，会随之更新。

### 常用命令

**设置默认 target**：

```
systemctl set-default multi-user.target
```

**切换到指定 target**：

```
systemctl isolate graphical.target
```

**查看可用 targets**：

```
systemctl list-units --type=target
```
