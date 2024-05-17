---
title: Learning GFS - Google File System 
date: 2024-05-05
tags: 
    - Distributed Systems
    - GFS
categories: Summary
---
:pushpin: 博客系统的Graphviz支持

- GFS是Google在2003年前后创建的可扩展分布式文件系统 ，用来满足 Google 不断扩展的数据处理需求。GFS 为大型网络和连接的节点提供容错、可靠性、可扩展性、可用性和性能。
:dash::clock11::sleeping:

<!--more-->

![GFS structure][1]

1. `Client`将file name和byte offset转换为chunk index，向`Master`发送包含file name和chunk index的请求，`Master`返回相应的chunk handle和chunk locations。（`Client`使用file name和chunk index作为key缓存这些信息）
2. `Client`向其中一个replica（最近的一个）发送请求，请求内容有chunk handle和byte range（在`Client`的缓存过期之前，不用再和`Master`进行交互）
3. 特别的，`Client`通常会在同一个请求中请求多个块数据

## In paper

### Metadata

- Original Paper Links: [The Google File System](https://pdos.csail.mit.edu/6.824/papers/gfs.pdf)

- The file and chunk namespaces
- The mapping from files to chunks
- The locations of each chunk's replicas
都存在主内存里（可以快速实现周期性扫描，以实现GC、故障备份迁移、负载均衡等功能）；
命名空间和映射也存储在`Master`的本地磁盘和remote备份的操作日志中，以实现持久化；
`Master`不会持久化locations，而是在启动的时候询问每个chunkserver，然后保持最新状态（比持久化在`Master`中要方便，因为大量chunkserver的join、leave、crush等问题将影响chunkserver和`Master`状态的同步）（或者说chunkserver对他自己磁盘上的存储内容拥有决定权，而不应该在`Master`上维护）。

### Big storage (Why hard?)

1. 最初的述求：Performance性能
2. Sharding分片：将数据分割放到大量的服务器上，这样就可以并行的从多台服务器读取数据
3. 发生故障需要容错，fault tolerance自动化修复
4. 复制副本replication
5. 副本的一致性问题inconsistency，额外的操作，导致性能降低

### Bad design for strong consistency

两个replication之间的write可能不同
![Bad design][2]

3.4 GFS goal

- Big, Fast
  - 大容量和快速
- Global
  - 可以通过申请权限来查看已有的数据
- Sharding
  - 分片存储
- Automatic recovery
  - 故障的自动回复
- Single data center
  - 并不是一个目标，是设计成了单个数据中心
- Internal use
  - google内部使用，并未完全开放（部分服务对外）
- Big sequential access
  - 针对的是大型文件的存储，所以是顺序的，并不像小型的随机访问存储

### `Master` Node

- `Master` server存储文件和chunk server的信息，chunk server真正存储数据
- 存储两个表单（内存里）：
  - 第一个是文件名到Chunk ID或者Chunk Handle数组的对应。NV（non-volatile, 非易失），这个标记表示对应的数据会写入到磁盘上。
  - 第二个表单记录了Chunk ID到Chunk数据的对应关系。这里的数据又包括了：
    - 每个Chunk存储在哪些服务器上，所以这部分是Chunk服务器的列表（V）
    - 每个Chunk当前的版本号，所以`Master`节点必须记住每个Chunk对应的版本号。（NV）
    - 所有对于Chunk的写操作都必须在主Chunk（Primary Chunk）上顺序处理，主Chunk是Chunk的多个副本之一。所以，`Master`节点必须记住哪个Chunk服务器持有主Chunk。（V）
    - 并且，主Chunk只能在特定的租约时间内担任主Chunk，所以，`Master`节点要记住主Chunk的租约过期时间。（V）
- 在磁盘上存Log、Check Point；写同时写内存和磁盘，读只从内存里
- 为什么用Log而不是database？
  - 在磁盘中维护log而不是数据库的原因是：数据库本质上来说是某种B树（b-tree）或者hash table，相比之下，追加log会非常的高效，因为你可以将最近的多个log记录一次性的写入磁盘。因为这些数据都是向同一个地址追加，这样只需要等待磁盘的磁碟旋转一次。而对于B树来说，每一份数据都需要在磁盘中随机找个位置写入。所以使用Log可以使得磁盘写入更快一些。
- 当`Master`节点故障重启，并重建它的状态，你不会想要从log的最开始重建状态，因为log的最开始可能是几年之前，所以`Master`节点会在磁盘中创建一些checkpoint点，这可能要花费几秒甚至一分钟。这样`Master`节点重启时，会从log中的最近一个checkpoint开始恢复，再逐条执行从Checkpoint开始的log，最后恢复自己的状态。

### Read File

1. `Client`将offset（相较于文件头部的偏移位置，读取部分文件）和file name发给`Master`
2. `Master`从file表单中查找文件名，得到chunk ID的数组，再除以64MB得到对应的chunk ID，再从Chunk表单中找服务器列表，响应给`Client`
3. `Client`从这些Chunk（Google的数据中心中，IP地址是连续的，所以可以从IP地址的差异判断网络位置的远近）服务器中挑选一个最近的来读取数据，并将读请求发送到那个服务器。因为客户端每次可能只读取1MB或者64KB数据，所以`Client`可能会连续多次读取同一个Chunk的不同位置。所以`Client`会缓存Chunk和服务器的对应关系，这样，当再次读取相同Chunk数据时，就不用一次次的去向`Master`请求相同的信息。
4. `Client`与选出的服务器通信，将Chunk Handle和偏移量发送给那个服务器。Chunk服务器会在本地的硬盘上，将每个Chunk存储成独立的Linux文件，并通过普通的Linux文件系统管理。并且可以推测，Chunk文件会按照Handle（也就是ID）命名。所以，Chunk服务器需要做的就是根据文件名找到对应的Chunk文件，之后从文件中读取对应的数据段，并将数据返回给客户端。

### Write File

Read是从随机的最新副本读取，而Write是必须写到Primary chunk，但是有可能主副本不存在，这时`Master`会找出所有存有Chunk最新副本的Chunk服务器，挑出一个作为Primary，其他的作为Secondary，然后增加版本号，同时也写入磁盘。（最新版本号是NV的，所以服务器宕机后可以从磁盘中恢复过来）。然后`Master`会将这些信息给对应的服务器，包括新的版本号，服务器也会把版本号存在磁盘里。

- 为什么不用最大的版本号当作最新的？
  - 当`Master`重启时，无论如何都需要与所有的Chunk服务器进行通信，因为`Master`需要确定哪个Chunk服务器存了哪个Chunk。你可能会想到，`Master`可以将所有Chunk服务器上的Chunk版本号汇总，找出里面的最大值作为最新的版本号。如果所有持有Chunk的服务器都响应了，那么这种方法是没有问题的。但是存在一种风险，当`Master`节点重启时，可能部分Chunk服务器离线或者失联或者自己也在重启，从而不能响应`Master`节点的请求。所以，`Master`节点可能只能获取到持有旧副本的Chunk服务器的响应，而持有最新副本的Chunk服务器还没有完成重启，或者还是离线状态（这个时候`Master`能找到的Chunk最大版本明显不对）。
- 如果找不到最新版本号的chunk会怎么样？
  `Master`节点会定期与Chunk服务器交互来查询它们持有什么样版本的Chunk。有两种可能：要么`Master`会等待，并不响应客户端的请求；要么会返回给客户端说，我现在还不知道Chunk在哪，过会再重试吧。比如说机房电源故障了，所有的服务器都崩溃了，我们正在缓慢的重启。`Master`节点和一些Chunk服务器可能可以先启动起来，一些Chunk服务器可能要5分钟以后才能重启，这种场景下，我们需要等待，甚至可能是永远等待，因为你不会想使用Chunk的旧数据。所以，总的来说，在重启时，因为`Master`从磁盘存储的数据知道Chunk对应的最新版本，`Master`节点会整合具有最新版本Chunk的服务器。每个Chunk服务器会记住本地存储Chunk对应的版本号，当Chunk服务器向`Master`汇报时，就可以说，我有这个Chunk的这个版本。而`Master`节点就可以忽略哪些版本号与已知版本不匹配的Chunk服务器。
- 如果chunk的版本号大于`Master`的版本号怎么办？
  - `Master`在向磁盘写入版本号前崩溃了，但分配了新的Primary，会用这个更高的版本号作为最新版本号。可能是因为发送消息后崩溃了，所以最好让pri和sec返回确认消息之后再写入磁盘。
- Primary如果同时收到大量`Client`的并发请求，会排序请求。确定Primary和Secondary chunk后，`Client`将数据发送给这些服务器，收到后写入临时位置，所有服务器都返回确认后，`Client`再向Primary发送确认追加请求，然后分发请求。
- 对于Secondary chunk来说，不一定会执行成功，可能因为磁盘不足或者故障或者丢包；只有所有chunk都写入成功才会返回yes给`Client`；否则将返回no然后重新发起整个流程。
- 什么时候版本号会改变？
  - 只在`Master`认为Chunk没有Primary时才会增加。
- `Master`发现Primary挂了怎么办？
  - `Master`会定期Ping Primary，没有响应认为将要重新指定一个Primary（但这不一定对，可能只是因为没有Ping通而不是挂了），这将导致Primary还可以和`Client`通信，此时指定新的Primary将同时存在两个，导致脑裂（split-brain）
  - 解决方案是使用租约，租约到期后更新Primary，若Ping不通，将不会重新指定Primary，而是等待上一个租约到期

### Consistency

1. 第一个`Client`对三个副本追加数据A
2. 第二个`Client`追加数据B，P和S1成功了，S2失败
3. 第三个`Client`追加数据C
4. 第二个`Client`重试追加B，成功了
5. 第四个`Client`追加D，但是S1失败了，并且`Client`再重试成功前发送故障，导致S1中将不存在D
![GFS Consistency][3]

- 在这里的GFS是弱一致性的，副本是不一致的，没有做到精确同步；并且根据偏移量然后执行追加数据的请求，这样就可以实现并行（当然也可以撤销所有部分成功的操作，这导致更复杂且更慢，需要平衡性能）
- 缺陷：
  - 用内存来维护Chunk表单，随着应用越来越多，尽管可以增加内存，但迟早消耗完
  - 单`Master`节点可能来不及处理非常高的并发请求，并且其还要承担一些写入磁盘的任务
  - `Master`节点的故障处理是人工干预的，需要自修复才比较好

<!-- markdownlint-disable-file MD025 MD033 -->
[1]: https://www.lingzhicheng.cn/usr/file/picture/MIT/GFS01.jpg
[2]: https://www.lingzhicheng.cn/usr/file/picture/MIT/GFS02.png
[3]: https://www.lingzhicheng.cn/usr/file/picture/MIT/GFS03.png
