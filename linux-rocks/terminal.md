# Terminal

## 早期终端

早期的终端是一种独立的硬件设备，用于连接到主计算机或服务器。它们通常包括：

* **输入设备**：如键盘，用于输入命令和数据。
* **输出设备**：如显示器或打印机，用于显示计算结果和系统反馈。

例如：早期的 Teletype (tty) 终端、DEC VT100 终端。

## 现代终端

现代终端更多是以软件形式存在的，即终端模拟器（Terminal Emulator），它们在现代操作系统（如Linux、macOS、Windows）中作为应用程序运行，模拟早期硬件终端的功能。

## 常见的终端模拟器

1. **GNOME Terminal**：常见于 Linux 系统的默认终端模拟器，功能强大且易于使用。
2. **iTerm2**：macOS 系统上的高级终端模拟器，提供了丰富的功能，如分屏、标签页、搜索等。
3. **Windows Terminal**：Windows 系统上的现代终端模拟器，支持多个命令行环境，如 PowerShell、CMD、WSL。
4. **PuTTY**：Windows 系统上广泛使用的 SSH 客户端，适用于远程连接和管理。

## Teletype

虽然 Teletype 设备现在已经不用了，但它的概念和功能依然存在于 Linux 中。

```
crw-rw-rw- 1 root tty 5, 0 Jul 24 16:40 /dev/tty
```
