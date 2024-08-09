# Systemd

## ENV

```
System: Ubuntu 22.04.4 LTS
Kernel: Linux 5.15.0-113-generic
Archit: x86-64
```

## Systemd

查看进程号为 1 的进程。

```
ps -p 1
```

```
PID TTY          TIME CMD
  1 ?        00:00:04 systemd
```

但当我们想显示详细信息的时候，结果会有些不一样。

```
ps -p 1 -f
```

```
UID  PID PPID C STIME TTY TIME     CMD
root   1    0 0 Aug07 ?   00:00:04 /sbin/init noibrs
```

其实这两个进程是一样的，因为 `init` 指向的是 `systemd`。

```
file /sbin/init
```

```
/sbin/init: symbolic link to /lib/systemd/systemd
```

`init` 是 Unix 最早的初始化进程，由于 `systemd` 取代了 `init`，为了兼容性，`/sbin/init` 通常是一个指向 `systemd` 的软链接。名称后面加 `d` 是 Unix 守护进程的命名规范，[System D](https://en.wikipedia.org/wiki/System\_D) 是一个术语，表示快速思考和解决问题的能力。`systemd` 诞生于 2010 年，在这之前用的是 SysVinit。

## SysVinit

Unix System V 这种初始化方法现在已经不怎么用了，但在一些旧的发行版中还能见到。

### runlevel

runlevel 也是这个 SysVinit 的产物。

```
runlevel
```

```
N 5
```

5 对应 `graphical.target`，N 表示上一次的 runlevel 为 No。

```
who -r
```

```
run-level 5  2024-08-07 21:30
```

阿里云的 Ubuntu 默认目标为 `graphical.target`，‌这是为了方便用户使用图形界面（VNC）来管理操作系统，不需要的话可以把目标换成 `multi-user.target`。

```
systemctl set-default multi-user.target
```

### /etc/rcX.d

rc 是 run commands 的缩写，`.d` 是目录的意思，目的是为了避免命名冲突。这个目录包含了特定运行级下启动的进程，例如运行级 5 对应 `/etc/rc5.d`。

```
ls -l /etc/rc5.d
```

```
lrwxrwxrwx 1 root root 13 Jul 10 11:08 K01fio -> ../init.d/fio
lrwxrwxrwx 1 root root 20 Apr 21  2022 K01irqbalance -> ../init.d/irqbalance
lrwxrwxrwx 1 root root 17 Jul 10 11:05 K01sysstat -> ../init.d/sysstat
```

虽然现在使用的是 `systemd`，但为了兼容性，这些目录和脚本仍然保留着。

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

### Nginx 状态

可以通过 Nginx 的状态查看服务的单元文件路径。

```
systemctl status nginx
```

```
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2024-08-08 12:43:17 CST; 2h 1min ago
       Docs: man:nginx(8)
   Main PID: 11037 (nginx)
      Tasks: 3 (limit: 1917)
     Memory: 8.6M
        CPU: 29ms
     CGroup: /system.slice/nginx.service
             ├─11037 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             ├─11039 "nginx: worker process" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""
             └─11040 "nginx: worker process" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" "" ""

Aug 08 12:43:17 guitarocks systemd[1]: Starting A high performance web server and a reverse proxy server...
Aug 08 12:43:17 guitarocks systemd[1]: Started A high performance web server and a reverse proxy server.
```

### Nginx 单元文件

```
cat /lib/systemd/system/nginx.service
```

> `/lib` 和 `/usr/lib` 的内容在现代的 Linux 发行版中可能是相同的，原因是文件系统层次结构（Filesystem Hierarchy Standard, FHS）规范的变化。最近几年，许多 Linux 发行版将 `/lib` 视为 `/usr/lib` 的符号链接，以减少目录结构的复杂性。

```
# Stop dance for nginx
# =======================
#
# ExecStop sends SIGSTOP (graceful stop) to the nginx process.
# If, after 5s (--retry QUIT/5) nginx is still running, systemd takes control
# and sends SIGTERM (fast shutdown) to the main process.
# After another 5s (TimeoutStopSec=5), and if nginx is alive, systemd sends
# SIGKILL to all the remaining processes in the process group (KillMode=mixed).
#
# nginx signals reference doc:
# http://nginx.org/en/docs/control.html
#
# 定义了服务的描述和依赖关系
[Unit]
Description=A high performance web server and a reverse proxy server
Documentation=man:nginx(8)
After=network.target nss-lookup.target

[Service]
# forking 表示服务在启动时会创建子进程，父进程会退出
Type=forking
# 存储主进程的 ID
PIDFile=/run/nginx.pid
# 启动准备，测试 nginx 配置
ExecStartPre=/usr/sbin/nginx -t -q -g 'daemon on; master_process on;'
# 启动命令，-g 设置全局指定，会覆盖Nginx配置文件的选项
ExecStart=/usr/sbin/nginx -g 'daemon on; master_process on;'
# 重载配置命令
ExecReload=/usr/sbin/nginx -g 'daemon on; master_process on;' -s reload
# 终止进程的命令
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid
# 终止服务的等待时间，超过 5 秒则强制终止
TimeoutStopSec=5
# 终止进程的方式，SIGTERM，超时，SIGKILL，优雅->强制
KillMode=mixed

# 如何安装和启用服务
[Install]
WantedBy=multi-user.target
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

设置默认 target：

```
systemctl set-default multi-user.target
```

切换到指定 target：

```
systemctl isolate graphical.target
```

查看可用 targets：

```
systemctl list-units --type=target
```
