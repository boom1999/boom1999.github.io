---
title: Ubuntu lose internet
date: 2024-05-02
tags: 
    - Debug
    - Ubuntu
    - VMware
categories: Summary
---

:pushpin: Ubuntu网络配置丢失

- 起因是这样，突然有一天远程开发连不上了，于是乎先开始排查是不是校园网的部分端口被Ban了，因为最近的$Bing.com$就一直访问不上，但是改变网络接入后仍然没用。
- 想到会不会是因为之前系统没有正常关机导致一些VMware网络配置错误，排查和重置了桥接、Nat模式，发现都没有问题
- 开始检查Ubuntu的的网络配置，发现在`Settings-NetWork`中，只有一个$VPN$配置，没有其他的网络配置，按理说应该有`Wired`，这就很奇怪了，因为之前是有配置的，而且也没有删除过对应的配置文件。

:dash::clock16::sleeping:

<!--more-->

----------

## 找到问题

在 Ubuntu 中，NetworkManager 是一个用于管理网络连接的系统守护进程。通过修改 NetworkManager.conf 文件，你可以对 NetworkManager 的行为进行一些配置。

一般来说，通过修改 NetworkManager.conf，你可以实现以下功能：

1. 修改 DNS 配置：
   - 你可以在 NetworkManager.conf 中指定 DNS 服务器的地址，这样 NetworkManager 在配置网络连接时将使用你指定的 DNS 服务器。
2. 配置网络管理：
   - 通过修改 NetworkManager.conf，你可以对 NetworkManager 的行为进行一些配置，比如配置连接优先级、禁用特定的网络功能等。
3. 修改插件行为：
   - NetworkManager 通过插件来管理不同类型的网络连接，你可以通过修改 NetworkManager.conf 来配置这些插件的行为。

- 问题就处在这个`NetworkManager`中，配置文件`/etc/NetworkManager/NetworkManager.conf`中的`ifupdown`插件没有启用，导致`/etc/network/interfaces`中的配置无法生效。

## 解决方法

1. 检查`/etc/network/interfaces`文件
   - `sudo vim /etc/network/interfaces`
   - 确认是否有配置文件，如果没有，可以参考如下配置：

     ``` conf
     auto lo
     iface lo inet loopback
     ```

   - `auto lo`：这个指令告诉系统在启动时自动激活本地回环接口。本地回环接口是一个特殊的网络接口，用于系统内部的网络通信，允许计算机与自身进行通信。通过设置 `auto lo`，系统在启动时会自动激活本地回环接口，确保系统内部的网络通信正常工作。
   - `iface lo inet loopback`：这行指令定义了本地回环接口的配置。`iface` 表示接口，`lo` 是本地回环接口的名称，`inet` 表示使用 IPv4 协议，`loopback` 表示这是一个本地回环接口。在这行指令中，`loopback` 参数表明这个接口是一个本地回环接口，它用于系统内部的循环通信，而不会连接到外部网络。
   - 这两行指令的作用是确保系统在启动时激活本地回环接口，并为本地回环接口配置了正确的参数，以确保系统内部的网络通信正常工作。

2. 打开配置文件`/etc/NetworkManager/NetworkManager.conf`，修改`managed=True`
   - `sudo vim /etc/NetworkManager/NetworkManager.conf`
   - 修改如下内容：
  
     ``` conf
     [main]
     plugins=ifupdown,keyfile

     [ifupdown]
     managed=True

     [device]
     wifi.scan-rand-mac-address=no
     ```

3. `reboot`
4. 原因
   1. 虚拟机的网络配置发生了变化。在虚拟化环境中，在VMware中，网络适配器的状态可能会受到宿主机或VMware的设置影响，从而导致虚拟机的网络配置发生变化。
      1. 宿主机网络设置变化：如果宿主机的网络适配器配置发生了变化，可能会影响到虚拟机的网络连接，导致 NetworkManager 在虚拟机中将 managed 设置为 false。
      2. VMware 网络设置变化：VMware 的网络设置可能会影响到虚拟机的网络配置。例如，当虚拟机所连接的网络适配器发生变化时，可能会导致 NetworkManager.conf 中 managed 的状态发生变化。
      3. 更新或重装 VMware Tools：在虚拟机中更新或重装 VMware Tools 时，可能会重新配置网络适配器，这也可能导致 NetworkManager.conf 中 managed 的状态发生变化。（很有可能是这个原因，之前更新了VM tools）
