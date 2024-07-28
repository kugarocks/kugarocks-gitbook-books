# Systemd

## Systemd 定义

Systemd 是内核创建的第一个进程，进程号为 1，是一个系统和服务管理器。它的主要功能包括系统初始化、进程管理和日志记录。Systemd 这个名字遵循 Unix 命名守护进程时在名称后添加字母 d 的惯例。它还借用了 System D 这一术语，指的是一个人快速适应和即兴解决问题的能力。Systemd 是 2010 年诞生的，在这之前使用的是 SysVinit。

## 单元文件

单元文件（Unit Files）是 systemd 使用的配置文件，用于定义和管理系统服务、设备、挂载点、套接字和其他系统资源。每个单元文件都描述了 systemd 如何启动、停止和监视相关资源。单元文件的文件扩展名根据其类型不同而有所区别，例如：

* **服务单元（Service Unit）**：用于定义系统服务，扩展名为 `.service`。
* **挂载单元（Mount Unit）**：用于定义文件系统挂载点，扩展名为 `.mount`。
* **设备单元（Device Unit）**：用于定义设备，扩展名为 `.device`。
* **套接字单元（Socket Unit）**：用于定义套接字激活的服务，扩展名为 `.socket`。
* **计时器单元（Timer Unit）**：用于定义定时任务，扩展名为 `.timer`。
* **目标单元（Target Unit）**：用于定义系统的目标状态，扩展名为 `.target`。

单元文件通常位于以下目录中：

* `/etc/systemd/system/`：系统管理员定义的单元文件。
* `/usr/lib/systemd/system/`：发行版提供的单元文件。
* `/run/systemd/system/`：运行时生成的单元文件。

以下是一个示例服务单元文件的内容，用于定义一个简单的服务：

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

