# Systemd

## 系统的第一个进程

```
root  1  0.0  0.2 191052  4092 ?  Ss  Jul19  0:10 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
```

systemd 进程号为 1，主要功能是系统初始化、进程管理和日志记录。名称后面加 d 是 Unix 守护进程的命名规范。[System D](https://en.wikipedia.org/wiki/System\_D) 也是一个术语，表示快速思考和解决问题的能力。它是 2010 年诞生的，在这之前用的是 SysVinit。

## SysVinit

这种初始化方法现在已经不怎么用了，但在一些旧的发行版中还能见到。

### Runlevel

runlevel 也是这个 SysVinit 的产物。在 CentOS 中可以通过下面的方式查询 runlevel。

```
$ runlevel
N 3

// 3 表示 multi-user mode，N 是上一次 runlevel，表示 No。

$ who -r
run-level 3  2024-07-19 15:10
```

/etc/inittab 文件定义了系统的默认运行级，看注释已经不再用了。

```
$ cat /etc/inittab
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

rc 是 run commands 的意思，

/etc/rc.d 这个目录包含了特定运行级下启动的的进程，例如运行级 3 对应 /etc/rc.d/rc3.d，现在已经被 systmd  代替了。

## Unit Files

单元文件是 systemd 的配置文件，用于定义和管理系统服务、设备、挂载点、套接字和其他系统资源。每个单元文件都描述了 systemd 如何启动、停止和监视相关资源。

### 单元文件的分类

* **服务单元（Service Unit）**：用于定义系统服务，扩展名为 `.service`。
* **挂载单元（Mount Unit）**：用于定义文件系统挂载点，扩展名为 `.mount`。
* **设备单元（Device Unit）**：用于定义设备，扩展名为 `.device`。
* **套接字单元（Socket Unit）**：用于定义套接字激活的服务，扩展名为 `.socket`。
* **计时器单元（Timer Unit）**：用于定义定时任务，扩展名为 `.timer`。
* **目标单元（Target Unit）**：用于定义系统的目标状态，扩展名为 `.target`。

### 单元文件的目录

* `/etc/systemd/system/`：系统管理员定义的单元文件。
* `/usr/lib/systemd/system/`：发行版提供的单元文件。
* `/run/systemd/system/`：运行时生成的单元文件。

### Nginx 的单元文件

```ini
[Unit]
Description=My Example Service
After=network.target

[Service]
ExecStart=/usr/bin/my-service
Restart=always
User=nobody

[Install]
WantedBy=multi-user.target
```

解释如下：

* `[Unit]`部分定义了服务的描述和依赖关系。
* `[Service]`部分定义了如何启动、停止和重启服务。
* `[Install]`部分定义了如何安装和启用服务。

可以使用以下命令管理单元文件：

* `systemctl start [unit]`：启动单元。
* `systemctl stop [unit]`：停止单元。
* `systemctl enable [unit]`：启用单元，使其在引导时自动启动。
* `systemctl disable [unit]`：禁用单元。
* `systemctl status [unit]`：查看单元状态。
* `systemctl daemon-reload`：在修改或添加单元文件后重新加载 systemd 配置。

单元文件是 systemd 配置和管理系统资源的核心机制，通过灵活的配置和强大的依赖管理能力，使系统管理员能够更高效地管理系统服务和资源。

## Targets

是一种特殊类型的 unit files，用于组织和管理系统的不同状态。它们类似于传统 System V init 系统中的 runlevels，但更加灵活和强大。targets 可以定义一组服务和其他 targets，以实现复杂的依赖关系和启动顺序。

以下是一些常见的 systemd targets 及其用途：

1. **basic.target**：基本系统服务已启动但尚未进行多用户操作的状态。
2. **multi-user.target**：多用户模式，不包含图形界面，适用于服务器环境。
3. **graphical.target**：多用户模式，包含图形界面，适用于桌面环境。
4. **rescue.target**：单用户模式，用于系统恢复和维护。
5. **emergency.target**：紧急模式，仅启动最小的系统环境，通常用于修复严重问题。
6. **reboot.target**：重启系统。
7. **poweroff.target**：关闭系统。
8. **halt.target**：停止系统。
9. **default.target**：默认启动的 target，通常是 multi-user.target 或 graphical.target。

**查看当前默认 target**：

```sh
systemctl get-default
```

**设置默认 target**：

```sh
systemctl set-default multi-user.target
```

**切换到指定 target**：

```sh
systemctl isolate graphical.target
```

**查看可用 targets**：

```sh
systemctl list-units --type=target
```

targets 文件通常存储在 `/usr/lib/systemd/system/` 或 `/etc/systemd/system/` 目录中。每个 target 文件都包含关于该 target 的依赖关系和启动顺序的信息。例如，一个简单的 multi-user.target 文件可能如下：

```ini
iniCopy code[Unit]
Description=Multi-User System
Documentation=man:systemd.special(7)
Requires=basic.target
Wants=remote-fs.target
After=basic.target remote-fs.target
Conflicts=rescue.service rescue.target
AllowIsolate=yes
```
