---
title: Keepalived浅入浅出
date: 2024-07-04
tags: 
    - Keepalived
    - Load Balance
categories: Summary
copyright: true
---

> Record some about `Keepalived` installation and usage skills.

<!--more-->

## :books: 1. 前言

### 1.1. 出现的背景

> 起初用来监控虚拟服务器集群(Linux Virtual Server, LVS)中各服务节点的状态，剔除故障节点，重新加入恢复的节点；后又加入虚拟路由冗余协议(Vritrual Router Redundancy Protocol, VRRP)，解决静态路由出现单点故障。主要实现服务器状态检测和故障隔离，高可用集群。

- `Client`向`Server`发送请求，随着服务量的增加，单机`Server`无法支持
- 使用分布式架构，用`Nginx`反向代理和多`Server`实现分布式架构和负载均衡
- 流量全部打在`Nginx`上，出现宕机，导致无法访问`Server`
- 使用`Keepalived`+多`Nginx`实现分布式的`Nginx`，主从架构实现高可用

### 1.2. 和HeartBeat比较

- `Keepalived`轻量级的高可用解决方案，通过VRRP实现，模拟路由器的多机，当主机的下一跳路由出现故障，由另一台主机替代工作，部署和使用相对简单，配置文件单一，无管理功能；`HeartBeat`基于主机或网络服务的高可用，拆分为多个子项目，安装配置以及使用相对复杂，具有管理功能，用户服务的多机。
- `HeartBeat`除了走网络通信还可以用串口通信，可靠性更高。
- LVS高可用建议用`Keepalived`，业务高可用建议用`HeartBeat`。

## 2. VRRP

> 多主机之间通过静态路由完成通信，一旦路由器故障，可以通过VRRP协议进行切换，保证网络的连通性。VRRP由两台或多台物理路由器设备虚拟成一个虚拟路由器，通过虚拟IP对外提供服务。

- 虚拟路由器：对外服务，拥有虚拟IP+MAC地址
- 主路由器(master)：虚拟路由器内部通常只有一台物理路由器对外提供服务，由选举算法产生master
- 备份路由器(backup)：除主路由器外的所有路由器，不对外提供服务，只接受主路由的通知，master挂掉之后由选举算法产生新的master

### 2.1. VRRP工作模式

- 工作状态
  - `Initialize`状态
  - `Master`状态
  - `Backup`状态
- 选举机制
  - 默认抢占模式，一旦有优先级高的路由器加入，立即成为Master。原来的Master重新恢复后，会重新接管服务，多了一次不必要的VirtualIP切换。
  - 非抢占模式下，只要Master不挂掉，优先级高的路由器只能等待。原来的Master重新恢复后，状态会变为Backup，不会接管服务。

### 2.2 抢占模式

- `Example 1`，只要优先级更高，就会获取Master，与状态无关
  - `Server1`为`Master`，`Server2`为`Master`或`Backup`，且`Server1`优先级更高，启动后`Server1`成为`Master`，`Server2`降级成为`Backup`；若此时`Server1`宕机，将由`Server2`接管服务，后续当`Server1`恢复后又变为`Master`，`Server2`降级成为`Backup`，属于抢占式。
  - `Server1`为`Master`，`Server2`为`Backup`，但`Server1`优先级低，启动后`Server2`成为`Master`，`Server1`降级成为`Backup`；若此时`Server2`宕机，将由`Server1`接管服务，后续当`Server2`回复后又变为`Master`，`Server1`降级成为`Backup`，属于抢占式。
- `Example 2`，两个`Backup`节点，必须配置`nopreempt`
  - `Server1`和`Server2`都为`Backup`，先启动的升级为`Master`，必须配置`nopreempt`，若`Server1`宕机，`Server2`会接管服务，`Server1`恢复后不会重新接管，`Server1`成为`Backup`，属于非抢占式。

## 3. 工作原理

> 工作在`TCP/IP`模型的第三、四和五层，即网络层、传输层和应用层。
> 网络层：通过`IMCP`协议向集群的每个节点发送数据包，检测节点的存活状态，若无响应则报告节点失效，从集群中剔除；
> 传输层:通过`TCP`协议的端口连接和扫描判断集群节点连接是否正常，若探测到没有响应数据判定为发生故障，从集群中剔除；
> 应用层：可以写脚本来检测服务是否正常以及剔除节点。

- Scheduler I/O Multiplexer
  - IO服用分发器，调度Keepalived所有的内部任务请求
- Memory Management
  - 内存管理机制，提供访问内从的通用方法
- Control Plane，Keepalived运行时会启动`core`、`check`和`vrrp`三个进程
  - Core Components
    - Watch Dog
      - 看门狗，针对被监视目标设置计数器和阈值，等待目标周期性重置计数器，若发生错误无法重置将被检测到。用来监控Checkers和VRRP进程
    - Checkers
      - 实现对服务器的运行状态检测和故障隔离
    - VRRP Stack
      - 实现失败切换功能，通过VRRP和LVS即可部署高性能的负载均衡集群
    - IPVS Wrapper
      - 实现IPVS，将设置好的IPVS规则发送到内核空间并提交给IPVS模块，实现最终的负载均衡
    - Netlink Reflector
      - VIP的设置和切换

## 4. 选举策略

- 控制节点角色由两个值决定，配置文件中的`priority`；vrrp_scritp模块中配置的`weigth`，可正可负。
  - 不设置`weight`时，集群中的优先级由`priority`决定，优先级高的节点成为`Master`，优先级低的节点成为`Backup`；但如果`Master`出现故障后`Backup`的`priority`小于`Master`的`priority`，导致主备切换失败。
  - 设置`weight`
    - 若`weight`为正
      - 主成功，`master priority + weight > backup priority + weight`时，主依然为主
      - 主失败，`master priority < backup priority + weight`时，主从切换
    - 若`weight`为负
      - 主成功`master priority > backup priority`时，主依然为主
      - 主失败，`master priority - abs(weight) < backup priority`时，主从切换

## 5. 相关配置

- 备份配置

  ``` bash
  cp /etc/keepalived/keepalived.conf /etc/keepalived/keepalived.conf.bak
  ```

- 全局配置
  
  ``` bash
  ! Configuration File for keepalived

  global_defs {
    notification_email {
      acassen@firewall.loc
      failover@firewall.loc
      sysadmin@firewall.loc
    }
    notification_email_from Alexandre.Cassen@firewall.loc
    smtp_server 192.168.200.1
    smtp_connect_timeout 30
    router_id LVS_DEVEL
    vrrp_skip_check_adv_addr
    vrrp_strict
    vrrp_garp_interval 0
    vrrp_gna_interval 0
  }
  ```

  ``` conf
  global_defs #全局配置标识
  notification_email #用于设置报警的邮件地址，可以设置多个，每行一个。如果要开启邮件报警，需要开启本机的sendmail服务
  notification_email_from #邮件的发送地址
  smtp_server #设置邮件的smtp server地址
  smtp_connect_timeout #设置连接smtp server的超时时间
  router_id # 运行keepalived的一个标识，唯一
  vrrp_skip_check_adv_addr ##对所有通告报文都检查，会比较消耗性能，启用此配置后，如果收到的通告报文和上一个报文是同一个路由器，则跳过检查，默认值为全检查
  vrrp_strict ##严格遵守VRRP协议,启用此项后以下状况将无法启动服务:1.无VIP地址 2.配置了单播邻居 3.在VRRP版本2中有IPv6地址，开启动此项并且没有配置vrrp_iptables时会自动开启iptables防火墙规则，默认导致VIP无法访问,建议不加此项配置
  vrrp_iptables #此项和vrrp_strict同时开启时，则不会添加防火墙规则,如果无配置vrrp_strict项,则无需启用此项配置
  vrrp_garp_interval 0 #gratuitous ARP messages 报文发送延迟，0表示不延迟
  vrrp_gna_interval 0 ##unsolicited NA messages （不请自来）消息发送延迟
  ```

- VRRP配置

  ``` bash
  vrrp_script nginx_check {
      script"/tools/nginx_check.sh"
      interval 1
  }
  vrrp_instance VI_1 {
      state MASTER
      interface ens33
      virtual_router_id 52
      priority 100
      advert_int 1
      authentication {
          auth_type PASS
          auth_pass test
      }
      virtual_ipaddress {
          192.168.149.100
      }
      track_script {
          nginx_check
      }
      notify_master /tools/master.sh
      notify_backup /tools/backup.sh
      notify_fault /tools/fault.sh
      notify_stop /tools/stop.sh
  }
  ```

  ``` conf
  vrrp_instance VI_1 #VRRP实例开始的标识 VI_1为实例名称
  state #指定keepalived的角色，MASTER表示此主机是主服务器，BACKUP表示此主机是备服务器
  interface #指定检测网络的网卡接口
  virtual_router_id #虚拟路由标识，数字形式，同一个VRRP实例使用唯一的标识，即在同一个vrrp_instance下，master和backup必须一致
  priority #节点优先级，数字越大表示节点的优先级越高，在一个VRRP实例下，MASTER的优先级必须要比BACKUP高，不然就会切换角色
  advert_int #用于设定MASTER与BACKUP之间同步检查的时间间隔，单位为秒
  auth_type PASS #预共享密钥认证，同一个虚拟路由器的keepalived节点必须一样
  auth_pass #设置密钥
  virtual_ipaddress #设置虚拟IP地址，可以设置多种形式：10.0.0.100不指定网卡，默认为eth0,注意：不指定/prefix,默认为/32；10.0.0.101/24 dev eth1 指定VIP的网卡；10.0.0.102/24 dev eth2 label eth2:1  #指定VIP的网卡label
  nopreempt # 设置为非抢占模式，同一实例下主备设置必须一样
  preemtp_delay # 设置抢占模式的延时时间，单位为秒
  ```

## 6. 集群资源监控

> 通过`track_script`和`vrrp_script`实现集群资源监控及改变优先级，实现主从切换

- `killall`探测服务运行状态

> 定义服务监控模块`nginx_check`，如果发现进程异常或关闭返回状态码`1`

  ``` bash
  vrrp_script nginx_check {
      script"killall -0 nginx"
      interval 1
  }
  track_script {
      nginx_check
  }
  ```

- 检测端口状态

> `fail`表示检测到失败的最大次数，也就是说如果请求失败两次，就认为此节点资源发生故障，进行切换操作；`rise`表示请求一次成功，就认为此节点资源恢复正常

  ``` bash
  vrrp_script nginx_check {
      script "</dev/tcp/127.0.0.1/80"
      interval 1
      fail 2
      rise 1
  }
  track_script {
      nginx_check
  }
  ```
