---
title: MapReduce in Distributed Systems
date: 2024-04-14
tags: 
    - Distributed Systems
    - LinkedList
    - 总结
categories: Distributed Systems
copyright: true
---

:pushpin:

- MIT 6.824 Distributed Systems Lab-1-Golang
:dash::clock1030::sleeping:
<!--more-->

# MIT 6.824 Distributed Systems

# Lecture 01 - Introduction

## Links

- [MIT 6.5840 Schedule: Spring 2024][1]
- [MIT 6.824 Schedule: Spring 2020][2]
- [Youtube 2020][3]
- BiliBili Translated:
  - [https://www.bilibili.com/video/BV1x7411M7Sf/?spm_id_from=333.999.0.0&vd_source=2c248c46a7ba66244aa5b9fb5b07d4fc](https://www.bilibili.com/video/BV1x7411M7Sf/?spm_id_from=333.999.0.0&vd_source=2c248c46a7ba66244aa5b9fb5b07d4fc)
  - [https://www.bilibili.com/video/BV1R7411t71W/?spm_id_from=333.1007.top_right_bar_window_default_collection.content.click&vd_source=2c248c46a7ba66244aa5b9fb5b07d4fc](https://www.bilibili.com/video/BV1R7411t71W/?spm_id_from=333.1007.top_right_bar_window_default_collection.content.click&vd_source=2c248c46a7ba66244aa5b9fb5b07d4fc)

## MapReduce 基本工作方式

![Read MapReduce(2024) png][4]

[Read MapReduce(2024)][5]

**Map：** 接受一个键值对(key-value pair)，产生一组中间键值对。MapReduce 会将 Map 函数产生的中间键值对传递给一个 Reduce 函数。

**Reduce：** 接受一个键以及相关的一组值，将这组值进行合并产生一组规模更小的值(通常只有一个或零个值)。

在实现的过程中，Reduce 函数使用 Iterator 读取中间结果，为了防止值过多，而无法全部放到内存中。

**Types:**

`map    (k1,v1)       -> list(k2,v2)`

`reduce (k2,list(v2)) -> list(v2)`

词频统计伪代码：

```cpp
map(String key, String value):
    // key: document name
    // value: document contents
    for each word w in value:
        EmitIntermediate(w, "1");

reduce(String key, Iterator values):
    // key: a word
    // values: a list of counts
    int result = 0;
    for each v in values:
        result += ParseInt(v);
    Emit(AsString(result));
```

### Process

1. 用户程序将输入文件切分为 M 个 16～64MB 之间的 split，并在集群中创建程序副本。
2. 程序副本分为 1 个 Master 和多个 Worker。Master 选取空闲的 Worker，并为其分配 Map 任务或 Reduce 任务。Master 存储任务的状态(空闲，工作中，完成)和 worker 机器(非空闲任务的机器)的标识。
3. Mapper 读取对应的 split 并**调用用户定义的 map 函数**解析为键值对，将输出的中间结果键值对存储到内存中。
4. Mapper 将缓存的键值对通过 Partition 函数(通常为 hash(key) mod R)划分为 R 个部分，并周期性地写入本地磁盘，同时将本地磁盘上中间结果的位置信息发送给 Master。
5. Master 将收到的中间结果的位置信息发送给 Reducer，Reducer 通过 RPC 读取存储在 Mapper 本地磁盘上的中间结果。在读取完毕后，<u>Reducer 会根据 key 对中间结果进行排序</u>。
6. 对每一个 key 和其对应的一组值，**调用用户定义的 Reduce 函数**进行处理，输出结果存储在对应的 Reduce Partition 文件中。
7. 当所有的 Map 任务和 Reduce 任务完成后，Master 通知用户程序。

### 容错机制

- worker 失效

Master 会**周期性地 ping** 每一个 Worker，如果某个 Worker 在一段时间内没有响应，Master 就会认为这个 Worker 已经不可用，对其上分配的任务**重新分配**给其他 Worker。

- master 失效

Master 会周期性地将集群的当前状态作为 **checkpoint** 写入到磁盘中。Master 发生故障后，重新启动 Master 即可利用存储在磁盘中的最近的 checkpoint 进行**恢复**。

### 数据存储机制

由于网络带宽是一种相当匮乏的计算资源，MapReduce 将数据存储在本地磁盘，由 GFS 进行管理。GFS 把每一份文件划分为多个 64MB 的 block，通常情况下，每个 block 会在不同的机器上保存三份副本。Master 在调度 Map 任务时会考虑输入数据的位置信息，采取**就近原则**，即尽量将 Map 任务**调度到包含相关输入数据副本的机器上执行**。因而大部分输入数据可以从本地读取，消耗非常少的网络带宽。

### 负载均衡机制

Master 将 Map 任务和 Reduce 任务划分为 M 和 R 个片段，**M 和 R 的数量应远大于集群中 Worker 的数量**。每个 Worker 执行大量不同的任务有助于提高集群的动态负载均衡能力，并且加快故障恢复的速度。

### 备份任务机制

如果集群中某个 Worker 花了很长时间才完成最后几个 Map 任务或 Reduce 任务，导致 MapReduce 总执行时间延长，这样的 Worker 被称为落后者(Straggler)。MapReduce 提供了备份任务机制来缓解这种情况。当 MapReduce 快要完成时，**Master 为剩下的正在运行的任务启动备份任务，将其分配给其他的空闲 Worker 来执行**，并在其中一个 Worker 完成后将该任务视作已完成。通过备份任务机制，大大减少了 MapReduce 的执行时间。

## Lab1: MapReduce

Lab source: [https://pdos.csail.mit.edu/6.824/labs/lab-mr.html](https://pdos.csail.mit.edu/6.824/labs/lab-mr.html)

- System `(hostnamectl)`

  - Static hostname: ubuntu
  - Icon name: computer-vm
  - Chassis: vm
  - Virtualization: vmware
  - Operating System: Ubuntu 18.04.6 LTS
  - Kernel: Linux 5.4.0-144-generic
  - Architecture: x86-64
- Goland version `(go version)`

  - go version go1.19.5 linux/amd64
- GCC `(gcc -version)`

  - gcc (Ubuntu 7.5.0-3ubuntu1~18.04) 7.5.0

### Remote debug

```bash
go install github.com/go-delve/delve/cmd/dlv@latest
```

### Sequential and Settings

```bash
$ cd ~/6.5840
$ cd src/main
$ go build -buildmode=plugin ../mrapps/wc.go
$ rm mr-out*
$ go run mrsequential.go wc.so pg*.txt
$ more mr-out-0
A 509
ABOUT 2
ACT 8
...
```

运行 `go build -buildmode=plugin ../mrapps/wc.go` 命令将生成一个插件文件，即构建 MR APP 的动态链接库。在这种情况下，生成的文件将具有 `.so` 扩展名，表示它是一个共享对象文件（shared object），用于动态加载到其他程序中。

通常情况下，生成的文件名将与源文件的包名相同，只是扩展名为 `.so`，例如 `wc.so`。当然，在 windows 下是不显示的，即使同步了双端代码，如果在 win 下生成将用 `.dll` 表示动态链接库。

文件 `wc.go` 以及 `mrapps` 目录下的其它几个文件，都定义了名为 `map, reduce` 的函数，这两个函数在 `mrsequential.go` 中加载并调用。给 `mrsequential` 绑定不同的 `*.so` 文件，也就会加载不同的 `map, reduce` 函数。如此实现某种程度上的**动态绑定。**

#### 可能会报错

```bash
# runtime/cgo
cgo: C compiler "gcc" not found: exec: "gcc": executable file not found in $PATH
```

It seems like the error message you’re encountering is related to the absence of the C compiler “gcc” in your system’s PATH variable.

To resolve this issue, you will need to ensure that the GNU Compiler Collection (GCC) is installed on your system and that its location is included in the PATH environment variable.

Here are the general steps to address this issue:

1. **Install GCC:**
   If you don’t have GCC installed, you can typically install it using your system’s package manager. For example, on a Debian-based system, you can use:

```bash
sudo apt-get update  
sudo apt-get install build-essential
```

1. **Add GCC to PATH:**
   Once installed, you should ensure that the directory containing the GCC executable is included in your PATH environment variable. This can typically be achieved by adding a line to your shell configuration file (e.g., **`****.bashrc****`**, **`****.zshrc****`**, etc.):

```bash
export PATH=$PATH:/path/to/gcc/directory
```

1. **Verify Installation:**
   After making these changes, open a new terminal or run **source**on the configuration file to apply the changes. You can then verify that GCC is accessible by running **gcc --version**.

#### 配置优化

- 原始命令

```bash
$ cd ~/6.5840
$ cd src/main
$ go build -buildmode=plugin ../mrapps/wc.go
$ rm mr-out*
$ go run mrsequential.go wc.so pg*.txt
$ more mr-out-0
A 509
ABOUT 2
ACT 8
...
```

- `.vscode/launch.json`

> 注意：这里的"args"貌似不支持通配符 `*`，所以需要手动把所有文件全部加进去，<u>或许可能有其他办法。</u>

```json
"args": ["wc.so", "pg-being_ernest.txt", "pg-dorian_gray.txt", "pg-frankenstein.txt", "pg-grimm.txt", "pg-huckleberry_finn.txt", "pg-metamorphosis.txt", "pg-sherlock_holmes.txt", "pg-tom_sawyer.txt"],
```

```json
{
    _// Use IntelliSense to learn about possible attributes._
    _// Hover to view descriptions of existing attributes._
    _// For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387_
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Run mrcoordinator.go",
            "type": "go",
            "request": "launch",
            "mode": "auto",
            "args": ["pg-being_ernest.txt", "pg-dorian_gray.txt", "pg-frankenstein.txt", "pg-grimm.txt", "pg-huckleberry_finn.txt", "pg-metamorphosis.txt", "pg-sherlock_holmes.txt", "pg-tom_sawyer.txt"],
            "program": "${workspaceFolder}/src/main/mrcoordinator.go",
            "cwd": "${workspaceFolder}/src/main"
        },
        {
            "name": "Run mrworker.go",
            "type": "go",
            "request": "launch",
            "mode": "auto",
            "args": ["wc.so"],
            "program": "${workspaceFolder}/src/main/mrworker.go",
            "cwd": "${workspaceFolder}/src/main",
            "preLaunchTask": "build wc && temp dir"
        },
        {
            "name": "Run mrsequential.go with wc.so",
            "type": "go",
            "request": "launch",
            "mode": "auto",
            "program": "${workspaceFolder}/src/main/mrsequential.go",
            "env": {},
            "args": ["wc.so", "pg-being_ernest.txt", "pg-dorian_gray.txt", "pg-frankenstein.txt", "pg-grimm.txt", "pg-huckleberry_finn.txt", "pg-metamorphosis.txt", "pg-sherlock_holmes.txt", "pg-tom_sawyer.txt"],
            "preLaunchTask": "build wc"
        }
    ]
}
```

- `.vscode/tasks.json`

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "build wc",
            "type": "shell",
            "command": "bash",
            "args": ["${workspaceFolder}/src/main/build-wc.sh"],
            "problemMatcher": [],  
            "group": {  
              "kind": "build",  
              "isDefault": true  
            }, 
            "presentation": {  
              "reveal": "silent"  
            }
        },
        {
            "label": "build temp dir",
            "type": "shell",
            "command": "bash",
            "args": ["${workspaceFolder}/src/main/build-temp-dir.sh"],
            "problemMatcher": [],  
            "group": {  
              "kind": "build",  
              "isDefault": true  
            },  
            "presentation": {  
              "reveal": "silent"  
            }
        },
        {
            "label": "build wc && temp dir",
            "dependsOn": ["build wc", "build temp dir"]
        }
    ]
}
```

- `6.8540/src/main/build-wc.sh`

> 注意：如果出现 `cannot load plugin wc.so`，需要加上 `-gcflags="all=-N -l"`，以禁用优化和内联，但这不一定是必要的，如果直接在 Terminal 进行动态库的生成，则不需要。

```bash
echo "Building wc plugin..."
cd src/main
go build -buildmode=plugin -gcflags="all=-N -l" ../mrapps/wc.go
echo "wc plugin built"
rm -f mr-*
echo "deleted old mr-* files"
```

## Our tasks

> Your job is to implement a distributed MapReduce, consisting of two programs, the coordinator and the worker. There will be just one coordinator process, and one or more worker processes executing in parallel. In a real system the workers would run on a bunch of different machines, but for this lab you'll run them all on a single machine. The workers will talk to the coordinator via RPC. Each worker process will, in a loop, ask the coordinator for a task, read the task's input from one or more files, execute the task, write the task's output to one or more files, and again ask the coordinator for a new task. The coordinator should notice if a worker hasn't completed its task in a reasonable amount of time (for this lab, use ten seconds), and give the same task to a different worker.

- 主体：完成一个分布式的 MapReduce，由一个 coordinator 进程（在之前的版本中用 master 称呼）和多个并行的 worker 进程组成。现实中真实的分布式系统通常运行在一堆不同的设备上，但在这里只用一台计算机，worker 通过 RPC 和 coordinator 通信。每一个 worker 进程将循环地向 coordinator 请求 task，从 task 的输入中读取一个或多个文件，执行程序然很将结果输出到一个或多个文件，然后再次请求新的 task。如果一个 worker 长时间没有完成 task（这里设置为 10s），coordinator 将通知另一个 worker 来执行这个 task。
- 注意：`main/mrcoordinator.go` 和 `main/mrworker.go` 是 test 的入口，不要求改。需要完成的在 `mr/coordinator.go`、`mr/worker.go` 和 `mr/rpc.go`。
- 大致的测试步骤：

  - 构建中间件用于 word-count，这里和上面类似，后续可以附带在启动 mrworker 的任务中。

    ```bash
    go build -buildmode=plugin ../mrapps/wc.go
    ```

  - 启动mrcoordinator，输入是所有txt文件，每一个文件代表一个split，同样的这里也要注意通配符。

    ```bash
    rm mr-out*
    go run mrcoordinator.go pg-*.txt
    ```

  - 启动多个mrworker

    ```bash
    go run mrworker.go wc.so
    ```

  - 按顺序排序后的词频统计应当和序列化单机的结果保持一致，用main/test-mr.sh进行测试，将测试输出是否保持正确，是否具有修复崩溃worker的能力

    ```bash
    $ cat mr-out-* | sort | more
    A 509
    ABOUT 2
    ACT 8
    ...
    ```

  - 结果将生成多个mr-out-X文件
  - 可能会出现RPC的报错rpc.Register: method "Done" has 1 input parameters; needs exactly three，Done()没有通过RPC，先忽略

### Rules

- map 阶段应当将中间态的 keys 划分到 `nReduce` 个桶进行 reduce 任务，这个数量就是 reduce 任务的数量，这个参数是从 `main/mrcoordinator.go` 传入到 `mr/coordinator.go` 的 `MakeCoordinator()` 函数的。为了减少任务的消耗，每一个 mapper 应当创建 `nReduce` 个中间文件。
- worker 应当将第 X 个 reduce 任务的输出放到 `mr-out-X`。
- `mr-out-X` 的输出格式，每一行包括一个 Reduce 函数的输出，以键值对的形式表示，用 `"%v %v"` 进行格式化，参照 `main/mrsequential.go`。格式错误可能导致测试不通过。
- worker 应当将 Map 的输出文件放在当前目录下，以便它作为 Reduce 任务的输入。
- 当 MapReduce 任务完全结束时，`mr/coordinator.go` 的 `Done()` 方法应当返回 `true`，此时 `main/mrcoordinator.go` 将退出。
- worker 也应当退出。一个简单的实现方法可以是使用 `call()` 的返回值，如果 worker 联系不上 coordinator，则说明任务已经完成，coordinator 已经退出，此时 worker 也可以退出。

### Hints

- 相关 Golang 的知识 [https://pdos.csail.mit.edu/6.824/labs/guidance.html](https://pdos.csail.mit.edu/6.824/labs/guidance.html)
- 一个入手的方法就是，设计 `mr/worker.go` 的 `Worker()` 函数，通过 RPC 向 coordinator 请求一个任务，然后设计 coordinator 的响应（尚未进行 map 任务的文件名），再设计 worker 如何读取这些文件，调用 Map 函数（类似于 `mrsequential.go`）。
- Map 和 Reduce 函数是通过 go 的 plugin 进行动态加载的，后缀是 `.so` 的文件。
- 如果修改了 `mr/` 目录下的任何东西，最好都重新加载一下 plugin，也就是上面提到的放到 tasks 里面作为启动前的任务。
- 当前的 lab 是基于一台机器进行文件操作的，如果是多台机器则需要全局的文件系统例如 GFS，在此并未考虑。
- 中间文件的命名，可以是 `mr-X-Y`，其中 X 是 Map 任务的编号，Y 是 reduce 任务的编号。
- map 任务存储的中间态键值对格式应当可以直接被 reduce 任务读取，一个可行的办法是使用 go 的 `encoding/json` 包：

    ```go
    // write
    enc := json.NewEncoder(file)
    for _, kv := ... {
    err := enc.Encode(&kv)
    
    // read
    dec := json.NewDecoder(file)
    for {
        var kv KeyValue
        if err := dec.Decode(&kv); err != nil {
            break
        }
        kva = append(kva, kv)
    }
    ```

- map部分可以用`worker.go`里的`ihash(key)`函数通过给定的key来选择reduce任务。

- 可以参考部分`mrsequential.go`中的代码，是非常有用的。

- coordinator是一个RPC服务，可能会有并发的问题，应当给一些数据加锁。

- 使用`go run-race`来引入go的竞态检测器，`test-mr.sh`开头有一段如何使用的说明，测试不会使用竞态检测器。不过，如果代码存在竞态，测试时会失败。

- workers有时候需要等待，例如reduces需要再最后一个map完成再工作。一个解决方案是workers周期性地向coordinator请求work，使用`time.Sleep()`；另一个方法是再relevant RPC handler中使用`time.Sleep()`或者`sync.Cond`进行等待。每个RPC都有自己的线程，不会影响其他RPC。

- coordinator无法区分宕机、卡住以及运行非常缓慢的workers。最好是让coordinator等待一定的时间（在这里是10s），超时则判定为死亡（当然有可能没有），然后把这个任务分发给其他worker。

- 如果要完成Section3.6中的Backup任务，将测试代码完成：worker没有崩溃的时候不会去完成其他任务，备份任务仅在10s后才进行。

- 可以使用`mrapps/crash.go`作为plugin进行崩溃恢复。

- MapReduce论文中提到，为了确保发生崩溃的时候没有人注意到这部分文件，可以采用临时文件以及完全写入后再进行重命名的方法。可以用`ioutil.TempFile`或者`os.CreateTemp`进行临时文件的创建，用`os.Rename`进行重命名

- `test-mr.sh`在子目录 `mr-tmp`中运行其所有进程，因此如果出现问题可以看那里。 可以临时修改`test-mr.sh` 以在测试失败后退出，这样脚本就不会继续测试（并覆盖输出文件）。

- `test-mr-many.sh`是连续运行很多次`test-mr.sh`，以便找到概率性的错误。不能并行多个`test-mr.sh`实例，因为coordinator用了同一个socker会导致冲突。

- GO RPC仅发送以大写字母开头的结构体字段，子结构必须由大写的字段名称。

- 调用RPC的`call()`方法时，返回结构应当包含所有默认值。RPC 调用应该如下所示：在调用之前不设置任何回复字段。 如果您传递具有非默认字段的回复结构，RPC 系统可能会默默地返回不正确的值。
  
    ```go
    reply := SomeType{}
    call(..., &reply)
    ```

### Challenges

- Implement your own MapReduce application (see examples in mrapps/*), e.g., Distributed Grep (Section 2.3 of the MapReduce paper).实现论文中 2.3 节的 MapReduce 应用。
- Get your MapReduce coordinator and workers to run on separate machines, as they would in practice. You will need to set up your RPCs to communicate over TCP/IP instead of Unix sockets (see the commented out line in Coordinator.server()), and read/write files using a shared file system. For example, you can ssh into multiple [Athena cluster](http://kb.mit.edu/confluence/display/istcontrib/Getting+Started+with+Athena) machines at MIT, which use [AFS](http://kb.mit.edu/confluence/display/istcontrib/AFS+at+MIT+-+An+Introduction) to share files; or you could rent a couple AWS instances and use [S3](https://aws.amazon.com/s3/) for storage.把 coordinator 和 workers 部署在不同的服务器上，改用 TCP/IP 取代 sockets 通信，并且使用 SFS 共享文件系统。

### 正式开始 Coding

#### worker 请求任务，coordinator 分配任务，worker 收到后打印

> 暂时只关注 coordinator 和 worker 的交互逻辑

- `wc.go` 的逻辑

 ```go
// 输入是文件名和内容，输出是结构体切片存放kva
func Map(filename string, contents string) []mr.KeyValue {
    // function to detect word separators.
    // 匿名函数，接收一个rune，返回一个bool，用于判断是否是字母，不是字母则是true
    ff := func(r rune) bool { return !unicode.IsLetter(r) }

    // split contents into an array of words.
    // 根据不是字母的点对contents进行拆分，也就是空格和其他，将得到单词的slice
    words := strings.FieldsFunc(contents, ff)

    kva := []mr.KeyValue{}
    // 遍历words切片，单词是key，value是"1"；重复的key在这时不会合并
    for _, w := range words {
        kv := mr.KeyValue{w, "1"}
        kva = append(kva, kv)
    }
    return kva
}

func Reduce(key string, values []string) string {
    // return the number of occurrences of this word.
    // values切片的长度就是个数相加，实现了map中同key相加的效果
    return strconv.Itoa(len(values))
} 
```

- `mrsequential.go`的逻辑
  - 判定输入的参数`os.Args`
  - 用`loadPlugin(os.Args[1])`读取`mapf`和`reducef`两个以动态链接加载的函数（用1是因为0是文件路径 **/home/boom/go/src/6.5840/src/main/__debug_bin710355273** `wc.so pg-being_ernest.txt pg-dorian_gray.txt pg-frankenstein.txt pg-grimm.txt pg-huckleberry_finn.txt pg-metamorphosis.txt pg-sherlock_holmes.txt pg-tom_sawyer.txt`，1才是`wc.so`）
    - `p, err := plugin.Open(filename)`加载插件
    - `xf, err := p.Lookup("Map")和p.Lookup("Reduce")`检索符号（这里的符号是函数）
    - `f :=  xf.(func())`调用函数
  - 在worker中KeyValue的结构体，用这个结构体定义未初始化的切片`type ByKey []mr.KeyValue`用于`intermediate := []mr.KeyValue{}`的排序
  
    ```go
    // struct is in 6.5840/mr
    type KeyValue struct {
        Key   string
        Value string
    }

    // for sorting by key.
    type ByKey []mr.KeyValue

    // for sorting by key.
    func (a ByKey) Len() int           { return len(a) }
    func (a ByKey) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }
    // 实现了sort接口的三个方法以完成排序，按照键的字母顺序排序a-z
    func (a ByKey) Less(i, j int) bool { return a[i].Key < a[j].Key }
    ```

  - 读取后续各个参数，open、read、close各个file，依次把filename和content进行map，再存入切片

    ```go
    for _, filename := range os.Args[2:] {
        //...
        file, err := os.Open(filename)
        content, err := ioutil.ReadAll(file) // 在go1.16之后用io.ReadAll()
        file.Close()
        kva := mapf(filename, string(content)) // 调用mapf获取当前文档的所有分词kv的切片
        intermediate = append(intermediate, kva...) // 追加到中间态的切片中暂存
    }
    ```

  - 排序切片，创建文件

    ```go
    sort.Sort(ByKey(intermediate))

    oname := "mr-out-0"
    ofile, _ := os.Create(oname)
    ```

  - reduce，然后关闭文件

    ```go
    i := 0
        for i < len(intermediate) {
            j := i + 1
            // 找到所有相同key的下标范围[i,j)
            for j < len(intermediate) && intermediate[j].Key == intermediate[i].Key {
                j++
            }
            // 暂存这些同key的所有value
            values := []string{}
            for k := i; k < j; k++ {
                values = append(values, intermediate[k].Value)
            }
            // 进行reduce
            output := reducef(intermediate[i].Key, values)
            // 写入上面创建的ofile文件，注意要按照这个格式
            // this is the correct format for each line of Reduce output.
            fmt.Fprintf(ofile, "%v %v\n", intermediate[i].Key, output)
            // 更新i，找下一组key
            i = j
        }

        ofile.Close()
    ```

#### 入口

- 单个终端启动`main/mrcoordinator.go`
  - `m := mr.MakeCoordinator(os.Args[1:], 10)`带Args读入，这里的Args是所有的txt文档；和`nReduce=10`（对于`map`任务的数量，**文件个数**就是`map`个数，**文件序号**就是`map`序号）
    - 由`mr.MakeCoordinator（）`初始化`Coordinator`
    - `c.server()`启动一个线程监听来自`mr/worker.go`的RPC请求
    - `go c.loopRemoveTimeoutTasks()`启动另一个线程循环移除过期的任务
  - 周期性调用`Done()`询问是否结束，如果返回`false`则在延迟1s后结束进程
  - 把所有`files`放入`c.unIssueMapTasks`队列

- 多个终端启动`main/mrworker.go`
  - 加载一个启动参数`wc.so`动态插件，`mapf, reducef := loadPlugin(os.Args[1])`
  - `mr.Worker(mapf, reducef)`
  - `Worker`结构：两个函数，任务属性、是否全部结束、Id

    ```go
    type AWorker struct {
        mapf    func(string, string) []KeyValue
        reducef func(string, []string) string

        // false for map, true for reduce
        mapOrReduce bool

        // false for not done, true for done and must exit
        allDone bool

        workerId int
    }
    ```

  - 判定`allDone`，进行`worker.process()`
  - 判定`mapOrReduce`，选择`map`任务或者`reduce`任务
    - 如果是`map`任务
      - 请求任务`reply := worker.askMapTask()`
      - 处理`reply`，如果为`nil`表示`map`任务已经处理完成，切换到`reduce`任务`worker.mapOrReduce = true`；如果不为`nil`，若`reply.FileId==-1`说明任务分配完了但是还没处理完，没有`map`任务可以给当前`worker`处理，什么都不做，随机等待一小段时间后再返回，重新`process`请求，否则就正常处理`worker.executeMap(reply)`
    - 如果是`reduce`任务
      - 请求任务`reply := worker.askReduceTask()`
      - 处理`reply`，如果为`nil`则表示全部结束， `worker.allDone = true`后返回，worker进程会退出；如果不为`nil`，若`reply.RIndex==-1`说明reduce任务已分配完但没处理完，否则就正常处理`worker.executeReduce(reply)`

#### 任务的请求和处理机制

- `reply := worker.askMapTask()`

```go
func (worker *AWorker) askMapTask() *MapTaskReply {
    // declare an argument structure and reply structure.
    args := MapTaskArgs{WorkerId: worker.workerId}
    reply := MapTaskReply{}

    worker.logPrintf("Asking map task...\n")
    ok := call("Coordinator.GiveMapTask", &args, &reply)
    if !ok {
        worker.logPrintf("askMapTask failed\n")
    }
    
    worker.workerId = reply.WorkerId
    
    if reply.FileId == -1 {
        if reply.AllDone {
            worker.logPrintf("All map tasks are done\n")
            return nil
        } else {
            worker.logPrintf("No map task available\n")
            return &reply
        }
    }
    worker.logPrintf("Got map task %v on file %v\n", reply.FileId, reply.FileName)
    return &reply
}
```

- - 请求主体的结构：`worker *AWorker`
  - 请求参数`args`结构：

    ``` go
    type MapTaskArgs struct {
        // give -1 if no task
        WorkerId int
    }
    ```

    - 返回参数`reply`结构：

    ```go
    type MapTaskReply struct {
        // worker pass the file name to the os to read
        FileName string

        // marks a unique file id for map task, give -1 if no more file
        FileId int

        // for reduce task
        NReduce int

        // assign the worker id to the task
        WorkerId int

        // assign where this kind of tasks are all done
        // if not or FileId is -1, the worker should wait
        AllDone bool
    }
    ```

  - 任务分发：`func (c *Coordinator) GiveMapTask(args *MapTaskArgs, reply *MapTaskReply) error{}`
    - 分发任务WorkerId为当前Coordinator的`c.curWorkerId`，此后`curWorkerId++`，如果WorkerId存在则不改变。
    - 通过`c.issuedMapMutex.Lock()`获取Map任务的互斥锁`issuedMapMutex  sync.Mutex`
    - 判定`Coordinator`的`c.mapDone`字段，map任务结束则直接释放锁、`reply.FileId == -1`，`reply.AllDone`设为true，然后返回nil
    - 判定没有分配的map任务队列和处理中的map任务队列是否都为空，为空则和上一条一样，附加设置`c.mapDone`为`true`，以及按照`c.nReduce`放入reduce的未分配队列
    - 从未分配map任务队列中`PopBack()`拿到fileId（按照文件数量排列`0-len(files)-1`)，以此也可以拿到文件名并写入reply，并且给`MapTaskState`设置开始时间，为后续超时做准备。

    ```go
    type MapTaskState struct {
        beginSecond int64
        workerId    int
        fileId      int
    }
    ```

    - 拿出的fileId放入已分配正在处理的map任务队列MapSet（实际上是使用map结构，并且统计了count）
    - 处理相应的返回值

- `worker.executeMap(reply)`

```go
func (worker *AWorker) executeMap(reply *MapTaskReply) {
    // execute map task
    intermediate := makeIntermediateFromFile(reply.FileName, worker.mapf)
    // write intermediate to files
    worker.writeToFiles(reply.FileId, reply.NReduce, intermediate)
    worker.joinMapTask(reply.FileId)
}
```

- - `intermediate := makeIntermediateFromFile(reply.FileName, worker.mapf)`主要用`mapf`打开文件然后文档，生成键值对，返回一个中间态的`[]KeyValue`切片
  - `worker.writeToFiles(reply.FileId, reply.NReduce, intermediate)`写入到中间文件，求hash后取`mod nReduce`得到分给哪一个reduce任务，因为reduce任务数量是知道的
  - 用`encoding/json`包存储中间态文件，可以直接被reduce任务读取
  - 请求`call("Coordinator.JoinMapTask", &args, &reply)`
    - 拿到已分配map任务队列的锁
    - 检查是否因为超时而导致当前FileId的任务已经从已分配map任务队列中移除
      - 已经被移除，解锁、打印信息
      - 未被移除，移除、解锁，打印信息
      - 未被移除且超时，放入未分配队列

- `reply := worker.askReduceTask()`

```go
func (worker *AWorker) askReduceTask() *ReduceTaskReply {
    // declare an argument structure and reply structure.
    args := ReduceTaskArgs{WorkerId: worker.workerId}
    reply := ReduceTaskReply{}

    ok := call("Coordinator.GiveReduceTask", &args, &reply)
    if !ok {
        worker.logPrintf("askReduceTask failed\n")
    }

    worker.workerId = reply.WorkerId

    if reply.RIndex == -1 {
        if reply.AllDone {
            worker.logPrintf("All reduce tasks are done, need to terminate this worker\n")
            return nil
        } else {
            worker.logPrintf("No reduce task available\n")
            return &reply
        }
    }
    worker.logPrintf("Got reduce task %v\n", reply.RIndex)
    return &reply
}
```

- - 请求主体的结构：`worker *AWorker`
  - 请求参数`args`结构：

    ```go
    type ReduceTaskArgs struct {
        // give -1 if no task
        WorkerId int
    }
    ```

  - 返回参数`reply`结构：

    ```go
    type ReduceTaskReply struct {
        RIndex    int
        NReduce   int
        FileCount int
        WorkerId  int
        AllDone   bool
    }
    ```

- 任务分发`func (c *Coordinator) GiveReduceTask(args *ReduceTaskArgs, reply *ReduceTaskReply) error {}`
  - 分发任务WorkerId为当前Coordinator的`c.curWorkerId`，此后`curWorkerId++`，如果WorkerId存在则不改变，这里用的是Map任务的同一套编号，也就是说不会出现map和reduce的WorkerId相同。
  - 通过`c.issuedReduceMutex.Lock()`获取Reduce任务的互斥锁`issuedReducepMutex  sync.Mutex`
  - 判定`Coordinator`的`c.allDone`字段，Reduce任务结束则直接释放锁、`reply.AllDone`设为true，`reply.RIndex`设为-1，然后返回nil
  - 判定没有分配的Reduce任务队列和处理中的Reduce任务队列是否都为空，为空则和上一条一样，附加设置`c.AllDone`为`true`，先比较请求Map，这里多设置一个参数`c.AllDone`
  - 从未分配Redduce任务队列中`PopBack()`拿到rIndex（按照NReduce数量排列`0-NReduce-1`)并写入reply，并且给`ReduceTaskState`设置开始时间，为后续超时做准备。

- `worker.executeReduce(reply)`

```go
func (worker *AWorker) executeReduce(reply *ReduceTaskReply) {
    outName := fmt.Sprintf("mr-out-%v", reply.RIndex)
    intermediate := make([]KeyValue, 0)
    for i := 0; i < reply.FileCount; i++ {
        intermediate = append(intermediate, readIntermediateFiles(i, reply.RIndex)...)
    }
    worker.logPrintf("Total intermediate size: %v\n", len(intermediate))
    tempfile, err := os.CreateTemp(".", "mrtemp")
    if err != nil {
        worker.logPrintf("cannot create tempfile for %v\n", outName)
    }
    reduceKVSlice(intermediate, worker.reducef, tempfile)
    tempfile.Close()
    err = os.Rename(tempfile.Name(), outName)
    if err != nil {
        worker.logPrintf("cannot rename tempfile for %v\n", outName)
    }
    worker.joinReduceTask(reply.RIndex)
}
```

- - `outName := fmt.Sprintf("mr-out-%v", reply.RIndex)`，当前`Reduce`任务的输出
  - 从当前RIndex所属的各个中间态文件读取，汇合到intermediate

    ```go
    intermediate := make([]KeyValue, 0)
    for i := 0; i < reply.FileCount; i++ {
        worker.logPrintf("Reading intermediate files on cluster %v\n", i)
        intermediate = append(intermediate, readIntermediateFiles(i, reply.RIndex)...)
    }
    ```

- - 通过`reduceKVSlice`排序以及合并，然后重命名、输出
  - 请求`call("Coordinator.JoinMapTask", &args, &reply)`
    - 拿到已分配map任务队列的锁
    - 检查是否因为超时而导致当前FileId的任务已经从已分配map任务队列中移除
      - 已经被移除，解锁、打印信息
      - 未被移除，移除、解锁，打印信息
      - 未被移除且超时，放入未分配队列

#### 任务分配机制

> 使用**任务队列**，`coordinator` 维护了两组两个任务队列，未分配、已分配（正在运行），当 `worker` 请求到达时，给 `worker` 分配一个未分配的任务，并把这个任务标记为已分配

未分配任务使用有锁的双向队列，已分配任务使用无锁的 Map，操作时加锁，有待优化

#### 测试

```bash
bash test-mr.sh > test-mr.out
```

```
*** Starting wc test.
--- wc test: PASS
*** Starting indexer test.
--- indexer test: PASS
*** Starting map parallelism test.
--- map parallelism test: PASS
*** Starting reduce parallelism test.
--- reduce parallelism test: PASS
*** Starting job count test.
--- job count test: PASS
*** Starting early exit test.
--- early exit test: PASS
*** Starting crash test.
--- crash test: PASS
*** PASSED ALL TESTS
```

<!-- markdownlint-disable-file MD025 MD033 -->
[1]: https://pdos.csail.mit.edu/6.824/schedule.html
[2]: http://nil.csail.mit.edu/6.824/2020/schedule.html
[3]: https://www.youtube.com/watch?v=cQP8WApzIQQ&list=PLrw6a1wE39_tb2fErI4-WkMbsvGQk9_UB
[4]: https://www.lingzhicheng.cn/usr/file/picture/MIT/MapReduce.PNG
[5]: https://pdos.csail.mit.edu/6.824/papers/mapreduce.pdf
