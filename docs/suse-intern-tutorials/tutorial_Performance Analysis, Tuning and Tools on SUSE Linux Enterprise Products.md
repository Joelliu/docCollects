# 针对SUSE Linux Enterprise的性能分析，调优和工具

SUSE Linux Enterprise

文档原作者： Macro Varlese, 软件工程师, SUSE

本文档介绍了如何配置和优化基于SUSE Linux Enterprise的系统，以获得最佳性能。并涵盖从BIOS设置到内核参数的多层次，教您修改何处以及如何修改。

另一方面，本文档并不会介绍完全绕过Linux内核堆栈并达到高吞吐量的网络解决方案（例如数据平面开发工具包 Data Plane Development Kit，简称DPDK）。本文档只关注标准的Linux内核基础架构。

## 1.0 介绍

随着计算机体系结构的发展，计算机性能已经达到了几年前难以想象的结果。然而，现代计算机体系结构的复杂性也导致要求终端用户和开发人员知道如何编写代码，如何为特定架构配置和部署软件以实现充分利用。

本文档关注SUSE Linux Enterprise系统的微调，包括SUSE Linux Enterprise软件产品提供的设置和可配置参数，网络接口（Network Interface Card, NIC）设置以及适用于大多数硬件的BIOS设置。

性能调优并不容易，给出普适的建议也比较困难。本文档试图深入了解会影响整体系统性能（吞吐量与延迟）的Linux内核配置，并给出一些可以使用的参数值示例。不过，对于不同的配置文件，这些值还是需要认真考虑，而不必作为要使用的唯一值。

本文档并不提供用于性能调优的通用经验法则（或值），文中所讲到的参数的最佳调优仍然需要在深入了解工作负载和硬件的基础上做出决定。

## 2.0 BIOS设置

BIOS是调优的基础，会对整个系统的性能产生影响。

BIOS能够控制CPU和RAM等组件运行的电压（以及频率）。此外，它还允许您启用或禁用特定的CPU功能，这些功能会对系统性能和功耗产生影响。

首先要知道的是CPU在运行时可处的状态（或模式），包括C状态和P状态。C状态是指空闲状态，而P状态是指操作状态。

除了C0状态是CPU实际工作时的唯一状态外，所有其他C状态都是空闲状态。C状态背后的基本思想是当CPU没有做任何有用的工作时最好将其关闭。这有助于降低功耗，对于CPU这样的电子元件来说也就意味着延长了其使用寿命。

P状态控制CPU运行时的操作状态。事实上，即使CPU处于C0状态，这也不意味着它就需要以最大速度运行。一个基本的实例就是电池模式下的笔记本电脑，此时CPU会进入更高的P状态，从而降低CPU的运行频率，并最大限度地降低功耗。

本文档并不涉及每个C/P状态的细节，如需详细信息可参考以下链接：

- http://www.hardwaresecrets.com/everything-you-need-to-know-about-the-cpu-c-states-power-saving-modes/
- https://software.intel.com/en-us/blogs/2008/03/12/c-states-and-p-states-are-very-different
- https://haypo.github.io/intel-cpus.html
- https://github.com/HewlettPackard/LinuxKI/wiki/Power-vs-Performance
- https://people.cs.pitt.edu/~kirk/cs3150spring2010/ShiminChen.pptx

是否需要启用或禁用CP状态以获得更高的吞吐量或低延迟很大程度上取决于用例。例如，在某些超低延迟的应用中，最好是禁用CPU的C状态，因为当CPU一直处于C0状态时，这样没有恢复工作的额外成本。

类似的，对于某些需要预测CPU在给定时间内执行工作量的用例，最好是将CPU频率设置为一个特定值（例如3Ghz），并仍然允许
睿频加速。

### 2.1 Cpupower工具

使用cpupower工具读取支持的CPU频率并进行相应设置。要安装该工具请在cpupower中运行zypper命令。

例如，如果运行 # cpupower frequency-info 命令，可以从输出中得到一些重要信息：

```shell
hardware limits: 1.20 GHz - 2.20 GHz
```

这表示了CPU支持的频率范围。

```shell
available frequency steps:  2.20 GHz, 2.10 GHz, 2.00 GHz, 1.90 GHz, 1.80 GHz,
1.70 GHz, 1.60 GHz, 1.50 GHz, 1.40 GHz, 1.30 GHz, 1.20 GHz
```

这表示了手动设置频率时可以设置的值。


```shell
available cpufreq governors: userspace ondemand performance
```

这表示内核支持的可用调控器：

- userspace 表示允许手动设置频率
- ondemand 表示允许CPU根据工作负载以不同速度运行
- performance 表示将CPU频率设置为允许的最大值

```shell
current CPU frequency: 1.70 GHz (asserted by call to hardware)
```

这表示了当前CPU的运行频率

```shell
boost state support:
       Supported: yes
       Active: no/
```

这表示CPU是否支持睿频加速以及其是否启用。

```shell
注意事项： 禁用P状态

如果P状态被禁用那么将自动不支持睿频加速。这意味着上面的‘Supported’这一项将始终为否，并因此无法启动。

类似的，当P状态启用并由intel_pstate驱动（Intel CPU）时，userspace管理器将不被支持。这意味着无法手动设置特定频率。目前，intel_pstate驱动仅支持performance和ondemand这两个管理器。

要在Intel平台上禁用P状态，只需将intel_pstate=disable添加到内核引导参数中。
```

当支持睿频加速时，可以通过运行以下命令启用它：

\# echo 0 > /sys/devices/system/cpu/intel_pstate/no_turbo

或者通过运行此命令禁用它：

\# echo 1 > /sys/devices/system/cpu/intel_pstate/no_turbo

要设置特定的管理器，请运行以下命令之一：

\# cpupower frequency-set -g userspace

(将管理器设置为 userspace)

\# cpupower frequency-set -g ondemand

(将管理器设置为 ondemand)

\# cpupower frequency-set -g performance

(将管理器设置为 performance)

如果要将CPU频率设置为特定速度，请运行以下命令：

\# cpupower frequency-set -f [FREQUENCY]

将 [FREQUENCY] 替换为 cpupower frequency-info 返回输出的可用频率中的之一。

## 3.0 内核调优

Linux内核提供了许多参数，可通过sysctl接口或proc文件系统进行调整。下面的章节将介绍那些能对整体系统性能产生直接影响的设置以及可被用于特定配置（例如高吞吐量与低延迟）的值。

### 3.1 I/O 调度程序调优

对I/O性能有直接影响的第一个设置是为设备选择的I/O调度程序。我们可以为每个设备选择相应的I/O调度程序，这意味着Linux内核允许为不同的设备使用不同的调度策略。这对于不同硬盘驱动器执行不同任务的系统来说会很方便，因为这种情况下采用不同的策略是有意义的。

要检索或更改I/O调度程序的值，可以访问/sys/block/sda/queue/scheduler中的文件。

在基于SUSE Linux Enterprise的发行版中，您可以从以下三种不同的调度算法中选择一种并分配给每个设备：noop，cfq和deadline。

完全公平队列（Complete Fair Queuing, CFQ）是一个面向公平的调度程序，也是内核使用的默认算法。该算法基于对于硬盘执行I/O操作的时间切片的使用。

要启用CFQ调度程序请运行以下命令：

```shell
echo cfq > /sys/block/sda/queue/scheduler
```

DEADLINE算法是一种面向延迟的I/O调度程序，其中每个请求都被分配一个目标截止期限。在多个线程执行读取或写入操作的情况下，该算法在不要求公平性的情况下能提供更大的吞吐量。

要启用DEADLINE调度程序请运行以下命令：

```shell
echo deadline > /sys/block/sda/queue/scheduler
```

NOOP算法是三者中最简单的算法。它执行任意发送到调度程序的I/O命令，无需任何复杂的调度。我们建议在那些存储设备可以自己执行调度的系统上使用这种算法，因为它避免了存储设备和试图执行任何调度的CPU之间的竞争。此外也在虚拟机中使用，因为虚拟机无法直接访问由虚拟机管理程序虚化的存储设备。

要启用NOOP调度程序请运行以下命令：

```shell
echo noop > /sys/block/sda/queue/scheduler
```

### 3.2 任务调度程序调优

Linux内核任务调度程序的基本配置已经在内核配置和编译时完成。本文档不会涉及到这些细节，而是主要介绍一些对数据包处理过程中所涉及的系统吞吐量或延迟产生影响的sysctl设置。

默认的Linux内核调度程序是完全公平调度程序 (Complete Fair Scheduler, CFS) ，其主要考虑累积虚拟运行时间  (vruntime)。当需要选择新任务时，它始终选择有最小累积vruntime值的任务。

以下是一些可为正在运行的进程分配的调度策略：

- **SCHED_OTHER** 是Linux默认的调度策略。
- **SCHED_FIFO** 使用先进先出算法，通常用于一些时间关键型应用。
- **SCHED_RR** 类似SCHED_FIFO策略，但其是使用Round Robin算法实现的。
- **SCHED_BATCH** 专为CPU密集型应用设计，这类应用可能需要占用CPU更长的时间。
- **SCHED_IDLE** 专为低优先级任务设计，这些任务可能很少运行或者对时间方面的要求不高。
- **SCHED_DEADLINE** 旨在给定期限内完成相应任务，与I/O调度程序非常相似。

通过使用chrt工具（由util-linux软件包提供）我们可以为进程分配不同的策略，还可以检索关于正在运行的进程以及受支持的策略的优先级的相关信息。

在下面的示例中，您可以检索各种调度策略的有效优先级：

```shell
# chrt -m
SCHED_SCHED_OTHER min/max priority	: 0/0
SCHED_SCHED_FIFO min/max priority	: 1/99
SCHED_SCHED_RR min/max priority	: 1/99
SCHED_SCHED_BATCH min/max priority	: 0/0
SCHED_SCHED_IDLE min/max priority	: 0/0
SCHED_SCHED_DEADLINE min/max priority	: 0/0 
```
根据上述优先级，您可以设置，例如，采用SCHED_FIFO策略且优先级为1的进程：

```shell
# chrt -f -p 1 <PID>
```

或者可以设置优先级为0的SCHED_BATCH策略：

```shell
# chrt -b -p 0 <PID>
```

以下sysctl设置会对吞吐量和延迟产生直接影响：


- kernel.sched_min_granularity_ns 表示CPU绑定任务（CPU bound tasks）的~~最小抢占粒度~~（minimal preemption granularity），详情可参阅sched_latency_ns。默认值为4000000纳秒。
- kernel.sched_wakeup_granularity_ns 表示~~唤醒抢占粒度~~（wake-up preemption granularity）。增加此变量可减少唤醒抢占，减少计算绑定任务的干扰。降低它则可以改善延迟关键任务的唤醒延迟和吞吐量，特别是当~~短占空比负载组件~~（short duty cycle load component）必须与CPU绑定组件竞争时。默认值为2500000纳秒。
- kernel.sched_migration_cost_ns 是上次执行完成迁移决策中‘缓存热点’任务后的时间。热门任务不太可能迁移到另一个CPU，因此增加此变量可减少任务迁移。默认值为500000纳秒。如果在有可运行进程时CPU空闲时间高于预期，可以尝试减少此值。如果任务经常在CPU或节点间~~反弹~~，可以尝试增加此值。
- kernel.numa_balancing 是一个布尔标志，用于表示进程/线程自动化NUMA平衡（automatic NUMA balancing）是否启用。自动NUMA平衡使用了多种算法和数据结构，且只在系统启动自动NUMA平衡时使用。

以下是针对不同性能配置文件中上述三个值的比较示例。

![表1 内核调优 - 对比](../images/suse-intern-tutorials/表1内核调优对比_1540198993_28750.png)

### 3.3 内存管理调优

Linux内核会将磁盘写入逐步放到内存中，并且随着时间推移异步地将它们刷新到磁盘。此外，还有可能会有很多的I/O操作会淹没缓存。Linux内核允许你通过使用sysctl命令来选择将多少数据在被交换到磁盘之前保留到RAM中，也可以调整各种其他如下所述的设置。

- vm.dirty_ratio 是在所有内容必须提交到磁盘前可用脏页填充的系统内存的绝对最大值（这里以百分比表示）。此时，所有新的I/O操作都会被阻塞，直到脏页都被写入到磁盘为止。这也通常是I/O操作长停顿的原因，但它可以防止过多的数据不安全地缓存在内存中。（优先选择vm.dirty_bytes）

- vm.dirty_bytes 是磁盘写入进程自身开始写回时的脏内存量。

```shell
注意：dirty_bytes 和 dirty_ratio

dirty_bytes 是 dirty_ratio 的对立变量，一次只能指定其中一个。当写入一个sysctl时，会立即考虑脏内存限制，而另一个在读取时显示为0。dirty_bytes允许的最小值是两页(以字节为单位)。任何小于此限制的值会被忽略，并保留原先的配置。
```

- vm.dirty_background_ratio 是在 pdflush/flush/kdmflush后台进程将其写入磁盘之前可以用“脏”页填充的系统内存的百分比。“脏”页面是仍需要写入磁盘的内存页面。例如，如果将此值设置为10（表示10％），并且您的服务器具有256 GB内存，那么在完成某些操作之前可以将25.6 GB的数据放在RAM中（最好使用vm.dirty_background_bytes）。

- vm.dirty_background_bytes 是后台内核刷新线程将开始回写的脏内存量。此设置与dirty_background_ratio相对立，一次只能指定其中一个。当写入一个sysctl时，会立即将其考虑在内以评估脏内存限制，另一个在读取时显示为0。在某些情况下，这是一个更好更安全的设置，因为它可以更好地调整内存量（例如，256 GB的1％= 2.56 GB对于某些情况可能已经太多了）。

- vm.swappiness: 内核缓冲区总是保留在主内存中，因为必须如此。而应用程序和缓存不需要留在RAM中，缓存可以删除，应用可以分页到交换文件中。 不过删除缓存意味着可能的性能损失，分页应用程序也是如此。此参数可帮助内核决定要执行的操作。通过将其设置为最大值100，内核将非常积极地进行交换。通过将其设置为0，内核将仅在防止出现内存不足的情况下交换。默认值为60，这意味着将进行一些交换。

以下是针对不同性能配置文件中上述三个值的比较示例。

![表2 内存管理调优 - 比较](../images/suse-intern-tutorials/表2内存管理调优比较_1540645820_8627.png)


### 3.4 网络堆栈调优

Linux内核允许修改影响网络堆栈的几个参数。 从内核2.6.17开始，网络堆栈支持完整的TCP自动调整，允许在最小值和最大值之间自动调整缓冲区大小。

本章将介绍一些可以增强Linux内核网络堆栈吞吐量和延迟的设置。，这些设置可通过sysctl接口进行配置。

#### net.ipv4.

- tcp_fastopen 是启用或禁用RFC7413的设置，允许以开放的SYN数据包形式发送和接收数据。启用此选项将保留有效负载传输的初始握手数据包，可以最大化网络带宽使用。

- 启用tcp_lowlatency（值设置为1）指示Linux内核做出倾向低延迟的决策。默认情况下，此设置被禁用（值设置为0）。建议在倾向低延迟的配置文件中启用此选项。

- 启用tcp_sack时允许选择确认。默认情况下，它被禁用（值设置为0）。建议启用此选项以提高性能。

- tcp_rmem是三个值的元组，表示TCP套接字使用的接收缓冲区的最小值，默认值和最大值。在适当内存压力下也会为每个TCP套接字设置。此元组中的默认值将覆盖参数net.core.rmem_default设置的值。

- tcp_wmem是三个值的元组，表示TCP套接字使用的发送缓冲区的最小值，默认值和最大值。每个TCP套接字都可以使用。此元组中的默认值将覆盖参数net.core.wmem_default设置的值。

-  ip_local_port_range定义TCP和UDP用于选择本地端口的本地端口范围。第一个数字是第一个本地端口号，第二个是最后一个本地端口号。

- tcp_max_syn_backlog表示尚未收到来自连接客户端确认消息的最大记忆连接请求数。低内存机器的最小值为128，并且它将与机器的内存成比例地增加。如果服务器出现过载，请尝试增加此数量。

-  tcp_syn_retries是未收到响应时重试SYN的次数。较低的值意味着较少的内存使用并减少SYN泛洪攻击的影响，但在有损网络上，最好选择一个大于5的值。

- tcp_tw_reuse 允许在协议安全的情况下为新连接重复使用TIME_WAIT状态下的套接字。它通常是tcp_tw_recycle的更安全的替代方法，但默认情况下它被禁用（值设置为0）。对于运行Web服务器或数据库服务器（例如MySQL）等服务的服务器来说，这是一个有趣的设置，因为它允许服务器在接受新连接时更快地扩展（例如TCP SOCKET ACCEPT）。重用套接字可以非常有效地减少服务器负载。由于此设置与用例密切相关，因此应谨慎使用（启用）。

- tcp_tw_recycle 允许TIME_WAIT套接字的快速回收。默认值伟0（禁用）。某些sysctl文档错误地将默认值声明为已启用。 已知的情况是启用（值设置为1）时在负载均衡和故障转移的情况下会导致一些问题，主要发生在配置此设置的计算机是一台处于执行natting设备背后的服务器的情况下。 启用回收后，服务器无法区分同一NAT设备后面的不同客户端的新传入连接。 由于此设置与用例密切相关，因此应谨慎使用（启用）。

-  tcp_timestamps启用RFC1323中定义的时间戳。 它默认启用（值设置为1）。其会为每个连接使用随机偏移，而不是仅使用当前时间。

#### net.core.

- 当接口接收数据包的速度超过内核处理数据包的速度时，netdev_max_backlog是设置在INPUT端排队的最大数据包数。

- netdev_budget: 如果SoftIRQ运行的时间不够长，传入数据的速率可能超出内核消耗缓冲区的能力。此时，NIC缓冲区将溢出，流量将丢失。所以有时有必要增加允许SoftIRQ在CPU上运行的时间，这正是此参数的作用。默认值为300，这会使得SoftRQ进程在CPU结束其运行前~~从NIC消耗300条消息~~。

- somaxconn 描述了socket listen（）backlog的限制，在用户空间中称为SOMAXCONN。默认值设置为128。可参阅tcp_max_syn_backlog以了解TCP套接字的其他调整。

- busy_poll 表示~~轮询和选择~~(poll and select) 时的低延迟繁忙轮询的超时情况，以微秒单位近似等待事件的时间。建议值取决于轮询的套接字数量。几个套接字可以使用50的值，上百个可以使用100。其他情况可能需要使用epoll。

```shell
注意：套接字

只有设置了SO_BUSY_POLL的套接字才会被繁忙轮询，这意味着可以在这些套接字上有选择地设置SO_BUSY_POLL或全局设置net.busy_read，不过这也将增加用电量。默认情况下此为禁用（值设置为0）。
```

- busy_read 表示了套接字读取所需的低延迟繁忙轮询的超时设置，以微秒为单位近似等待设备队列上的数据包的时间。这将为SO_BUSY_POLL套接字选项设置默认值。对于每个套接字可以通过设置套接字选项SO_BUSY_POLL来设置或覆盖此值，这也是首选的启用方法。如果需要通过sysctl全局启用该功能，建议值为50。此项会将增加用电量。默认情况下为禁用（值设置为0）。

- rmem_max 表示最大接收套接字缓冲区大小（以字节为单位）。

- wmem_max 表示最大发送套接字缓冲区大小（以字节为单位）。

- rmem_default 表示套接字接收缓冲区的默认设置（以字节为单位）。

- wmem_default 表示套接字传输缓冲区的默认设置（以字节为单位）。

以上表格是不同性能配置文件中上述参数的可能配置的比较。

![表3 网络堆栈调优 - 比较](../images/suse-intern-tutorials/表3网络栈调优比较_1540646188_26732.png)

## 4.0 IRQ设置

正确的IRQ设置，可以对吞吐量和延迟性能产生重要影响，在多核架构和多线程应用程序中尤其如此。

要验证IRQ关联（IRQ affinitization），可以读取/proc/interrupts，从输出中可以看到相关的硬件，所有中断以及是哪块CPU在处理。

不同的硬件供应商会提供他们自己的支持脚本以考虑NUMA架构的同时高效地配置IRQ关联。

无论是使用供应商脚本还是自己手动IRQ核心关联，在Linux上执行的第一步都是通过运行以下命令来停止和禁用irqbalance服务：

\# systemctl disable irqbalance

\# systemctl stop irqbalance

建议使用NIC供应商提供的脚本，但如果无法使用或想要手动设置，可执行以下步骤：

- 1.找到连接到相关端口的处理器：

```shell
    # numactl --cpubind netdev:eth1 -s
```
举个例子，如下：

```
    physcpubind: 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71
    cpubind: 2
    nodebind: 2
```

这些值告诉我们端口是由NUMA架构中的#2节点管理的，所涉及到的核心数为从为8-71。

- 2. 找到每个处理器的位掩码：

    Math: 2^CORE_ID 并把结果转换为 HEX

- 3. 找到分配给端口的IRQ：

```shell
    # grep eth1 /proc/interrupts
```

在这里，对于64个可用队列，中断范围是从52到115。

- 4. 通过以下方式将SMP亲和性（SMP affinity）（步骤2中的计算值）输入到相应的IRQ条目中：

```shell
    # echo 10000 > /proc/irq/52/smp_affinity
    # echo 20000 > /proc/irq/53/smp_affinity
    […]
    # echo 40000000 > /proc/irq/114/smp_affinity
```

## 5.0 NIC设置

不同的网卡提供不同的特性，可以提高吞吐量，并减少由计算主机处理的网络流量的延迟。

使用ethtool工具可以启用和配置这些功能。

除了更高级的特性之外，还有其他对所有NIC都通用的功能，例如允许更大或更小的缓冲区来存储由NIC本身接收或传输的数据包。

接下来将首先介绍一些常用参数，然后介绍更多高级功能。

### 5.1 环缓冲区（Ring buffers）

每个NIC都配有一定内存，用于存储接收或要传输的网络数据包。

较大的缓冲区允许NIC在发出中断前存储更多的数据包，从而减少以特定速率丢失的数据包数量。

在触发中断前可以调整NIC要接收的（从网络中读取或传输到网络中）数据包的数量。此外，还可以在触发中断前控制接收到设置量后NIC应等待的时间。

例如，可以设置以下值：

```
Ethernet Link @ 10Gb/s
Minimum frame size: 84 bytes (worst case scenario)
Packet rate:
       10,000,000,000 b/s / (84 B * 8 b/B) = 14,880,960 packets/s (maximum rate)
       ~14,880 packets/ms (millisecond) 
       ~14 packets/us (microsecond)
Interrupt rate: 100us (microseconds)
Receive buffer size required: 1400 entries
```

在上面的示例中，可以看到NIC支持超过4096个RX和TX环条目，但当前之都设置为64。

要修改系统使用的值，可以使用带-G选项的ethtool命令。

如下：

```shell
# ethtool -G eth1 rx 4096
# ethtool -G eth1 tx 4096
# ethtool -g eth1
Ring parameters for eth1:
Pre-set maximums:
RX:		4096
RX Mini:	0
RX Jumbo:	0
TX:		4096
Current hardware settings:
RX:		4096
RX Mini:	0
RX Jumbo:	0
TX:		4096
```

将环形缓冲区大小设置为更到值允许NIC以特定速率接收或发送更多的数据包，从而提高网络吞吐量。虽然增加缓冲区大小可以对吞吐量产生积极影响，但对数据包延迟来说就是反作用，这是因为在网络堆栈处理前，数据包将在NIC内存中保留更长的时间。

为了在不增加网络吞吐量的同时不要过多地牺牲延迟，可以使用ethtool（选项-S）提供的统计信息来平衡吞吐量和延迟。

具体来说，在处理目标数据包速率时，从接收和发送环大小的默认值（例如10Gb/s）开始，然后查看ethtool -S命令提供的关于rx_dropped/tx_dropped计数器的信息，并逐步增加环缓冲区大小（以2倍速率）直到rx_dropped/tx_dropped计数器停止或达到要求。需要注意的是，并非所有场景都要求无数据包丢失。

### 5.2 中断合并（Interrupt Coalescing）

如之前提到的，NIC还允许如下配置：

- 在触发中断之前，在接收（rx帧）或发送（tx帧）中排队的数据包数量

- 在触发中断之前达到rx-frames / tx-frames的值后等待多长时间（rx-usecs / tx-usecs）

要微调这些参数仍然可以使用ethtool - S命令提供的统计信息。

但是，当需要更高的吞吐量并且NIC驱动程序正在使用NAPI时，rx-frames参数的值为64可以帮助提高吞吐量，因为在每次轮询时，驱动程序将最多轮询64个数据包。

要实现上述设置，请使用以下命令：

\ # ethtool -C eth1 rx-frames 64

\ # ethtool -C eth1 tx-frames 64

\ # ethtool -C eth1 tx-usecs 8

\ # ethtool -C eth1 rx-usecs 8

要验证是否已经设置完毕请使用以下命令：

\ # ethtool -c eth1

要为rx-frames / tx-frames和rx-usecs / tx-usecs使用自定义值，需要关闭动态中断自适应（DIA），DIA是允许NIC根据网络负载自动调整这些设置的一种特性。但并非所有NIC都能实现这样的功能; 有些需要特定的内核和驱动程序版本来支持它。

要为RX和TX配置DIA，请使用以下两个命令：

\ # ethtool -C eth1 adaptive-rx on

\ # ethtool -C eth1 adaptive-tx on

### 5.3 卸载功能（Offload Capabilities）

不同NIC供应商提供不同的卸载功能。 要检查NIC支持哪些功能，请使用命令ethtool -k DEVICE。 标记为[fixed]的功能无法更改，因为NIC（或驱动程序）可能未实现该功能（例如off [fixed]），或者它们是NIC正常工作所必需的功能（例如on [fixed]）。

输出示例：

```shell
# ethtool -k eth1
Features for eth1:
rx-checksumming: on
tx-checksumming: on
       tx-checksum-ipv4: on
       tx-checksum-ip-generic: off [fixed]
       tx-checksum-ipv6: on
       tx-checksum-fcoe-crc: on [fixed]
       tx-checksum-sctp: on
scatter-gather: on
       tx-scatter-gather: on
       tx-scatter-gather-fraglist: off [fixed]
tcp-segmentation-offload: on
       tx-tcp-segmentation: on
       tx-tcp-ecn-segmentation: off [fixed]
       tx-tcp6-segmentation: on
udp-fragmentation-offload: off [fixed]
generic-segmentation-offload: on
generic-receive-offload: on
large-receive-offload: on
rx-vlan-offload: on
tx-vlan-offload: on
ntuple-filters: off
receive-hashing: on
highdma: on [fixed]
rx-vlan-filter: on [fixed]
vlan-challenged: off [fixed]
tx-lockless: off [fixed]
netns-local: off [fixed]
tx-gso-robust: off [fixed]
tx-fcoe-segmentation: on [fixed]
tx-gre-segmentation: off [fixed]
tx-ipip-segmentation: off [fixed]
tx-sit-segmentation: off [fixed]
tx-udp_tnl-segmentation: off [fixed]
fcoe-mtu: off [fixed]
tx-nocache-copy: off
loopback: off [fixed]
rx-fcs: off [fixed]
rx-all: off
tx-vlan-stag-hw-insert: off [fixed]
rx-vlan-stag-hw-parse: off [fixed]
rx-vlan-stag-filter: off [fixed]
l2-fwd-offload: off
busy-poll: on [fixed]
hw-tc-offload: off
```

#### 校验和卸载（checksum offload）

Linux内核允许在NIC上配置接收和传输的校验和卸载。


标识接收校验和卸载的参数称为rx-checksumming，可以设置为on或off。

可以在下面看到子功能的完整列表：

```
tx-checksumming: on
       tx-checksum-ipv4: on
       tx-checksum-ip-generic: off
       tx-checksum-ipv6: on
       tx-checksum-fcoe-crc: on
       tx-checksum-sctp: on
```

要启用或禁用任何允许的子参数，只需将子参数名称传递给-K选项即可。 以下命令中给出一个示例：

\# ethtool -K eth1 tx-checksum-ipv4 off

#### 分段卸载（segmentation offload）

要通过特定网络发送数据包，必须符合该网络的MSS和MTU。 任何应用程序都应该从它运行的实际网络中抽象出来， 这增强了其可移植性和易维护性，也使得由内核来负责在通过网络发送数据之前将数据分段为多个数据包。

为了释放CPU周期并使内核尽可能使用缓冲区，大多数NIC使用所谓的GSO（Generic Segmentation Offload，通用分段卸载）和TSO（TCP Segmentation Offload，TCP分段卸载）来自行执行调整大小和重新打包。

要启用或禁用GSO或TSO，请使用以下命令：

\# ethtool -K eth1 gso on

\# ethtool -K eth1 gso off

\# ethtool -K eth1 tso on

\# ethtool -K eth1 tso off

要禁用TCP分段卸载，还需要禁用通用分段卸载， 否则，任何TCP流量都将被视为通用。

另一方面，可以在禁用GSO时启用TSO。 在这种情况下，只有TCP流量将被卸载到NIC进行分段， Linux内核网络堆栈将处理（用于分段的）任何其他协议。

#### 接收卸载（receive offload）

为了最小化每个数据包的开销，Linux内核实现了所谓的大接收卸载（LRO）和通用接收卸载（GRO）。不过，已经证明LRO在某些情况下会失效，因此建议禁用它。

GRO相反则实现了一种更好的合并接收数据包的技术：MAC头必须相同，只有少数TCP或IP头可以不同。严格限制可能不同的标头集：校验和必然会不同，IP ID字段允许增加。甚至TCP时间戳也必须相同，这比它看起来的限制要少；时间戳是一个相对低分辨率的字段，因此很多数据包具有相同的时间戳并不奇怪。由于这些限制，合并的数据包可以无损地重新分类。作为额外的好处，GSO还可用于执行重新分类。 GRO的另一个重要方面是LRO不仅限于TCP / IPv4。 GRO从内核2.6.29开始合并，并可以支持各种10G驱动程序（另请参阅https://lwn.net/Articles/358910/]

\# ethtool -K eth1 gro on

\# ethtool -K eth1 gro off

#### VLAN 卸载（VLAN offload)

如今，大多数NIC都支持接收和发送路径的VLAN卸载。此功能允许在接收或传输时从数据包中添加或剥离VLAN标记。

默认情况下，大多数驱动程序启用此功能，但如果需要禁用它，可运行如下命令：

\# ethtool -K [DEVICE] rxvlan off

\# ethtool -K [DEVICE] txvlan off 

#### 隧道（无状态）卸载 (tunnels (stateless) offload)

每个用于虚拟网络的隧道协议都会在原始数据包（例如VxLAN包）周围封装UDP报头，相当于添加了额外的一层。由于需要为每个数据包添加和删除这个额外的层，CPU需要执行更多工作来接收和发送每个数据包。由于CPU忙于这些“新”步骤，覆盖网络的系统吞吐量和延迟方面的性能相比扁平网络要更差。

较新的NIC实现隧道分段卸载，为覆盖网络实现可用于TCP的相同概念（TCP分段卸载）。

此功能可将大型传输数据包的分段卸载到NIC硬件。例如，您可能有9000字节的内部有效负载，而您仍然需要遵守1500的最大MTU。在将数据包传输到网络之前，NIC会执行将有效负载分段为多个数据包（例如，VXLAN封装）的操作。

要启用或禁用此功能，请运行以下命令：

\# ethtool -K [DEVICE] tx-udp_tnl-segmentation <off>

\# ethtool -K [DEVICE] tx-udp_tnl-segmentation <on>

在某些NIC中可以找到的另一个功能是内部数据包校验和卸载。 启用此功能后，可以将已封装数据包的校验和计算卸载到硬件。

要启用或禁用此功能，请运行以下命令：

\# ethtool -K [DEVICE] tx-udp_tnl-csum-segmentation <off>

\# ethtool -K [DEVICE] tx-udp_tnl-csum-segmentation <on> 

####  哈希和数据包引导卸载（hashing and packet steering offload）

现在的NIC通常具有多个硬件队列，其中数据包可以放置在接收侧或发送侧。

这种硬件功能在多核架构中具有很有优势，因为每个队列都可以被分配一个特定的IRQ（请参阅IRQ配置）。因此，每个中断都可以有一个特定核心固定和处理。

同样，NIC允许将符合某些条件的特定流引导到特定的硬件队列，从而引导到可能的特定的CPU核心。这与RSS在软件领域中的作用很相似，但它同时还具有由硬件执行的额外优势。CPU可以从散列/分类数据包，引导它们到正确软件队列中解放出来。

与数据流的散列和引导相关的两个ethtool参数是rxhash和ntuple。

rxhash是一个非常基本的参数，可以使用以下的命令启用或禁用：

\# ethtool -K \[DEVICE\] rxhash on

\# ethtool -K \[DEVICE\] rxhash off

ntuple参数是一个更复杂的参数，它允许通过在数据包本身的各个字段上配置匹配条件来指定您感兴趣的流。下面给出一些例子。

例如，要将TCP流从192.168.10.10引导到192.168.10.20并赋给队列号3，可以运行以下命令：

\# ethtool -N flow-type tcp4 src-ip 192.168.10.10 dst-ip 192.168.10.20 action 3

如果用于参数操作的值为-1，则NIC将丢弃收到的数据包。

这种方法还可以适配不同协议。 不过一些要配置的参数仅适用于某些协议（例如，proto仅适用于flow-type ether，而l4proto仅适用于flow-type ip4）。 要查看支持的参数和有效值的完整列表，请参阅ethtool手册页（[http://man.he.net/man8/ethtool](http://man.he.net/man8/ethtool)）及NIC供应商文档。

要显示当前应用于接口的过滤器，请使用命令ethtool --show-ntuple。

## 6.0 参考

有关更多详细信息和参考，请查看以下文章：

- [Hardware Secrets, "Everything You Need to Know About the CPU C-States Power Saving Modes"](http://www.hardwaresecretscom/everything-you-need-to-know-about-the-cpu-c-states-power-saving-modes/)

- [Intel Developer Zone, "C-states and P-states are very different"](https://software.intel.com/en-us/blogs/2008/03/12/c-states-and-p-states-are-very-different)

- [GitHub, Haypo Blog, "Intel CPUs: P-state, C-state, Turbo Boost, CPU frequency, etc."](https://haypo.github.io/intel-cpus.html)

- [GitHub, HewlettPackard/LinuxKl, "Power Savings vs. Performance on Linux"](https://github.com/HewlettPackard/LinuxKI/wiki/Power-vs-Performance)

- [Presentation, Shimin Chen, Intel Labs Pittsburgh, "Power Management Features in Intel Processors"](https://people.cs.pitt.edu/~kirk/cs3150spring2010/ShiminChen.pptx)

- [Cisco, "Bandwidth, Packets Per Second, and Other Network Performance Metrics"](https://www.cisco.com/c/en/us/about/security-center/network-performance-metrics.html)

- [Kernel documentation](https://www.kernel.org/doc/Documentation/networking/ip-sysctl.txt)

- [The Lone Sysadmin, "Better Linux Disk Caching & Performance with vm.dirty_ratio & vm.dirty_background_ratio"](https://lonesysadmin.net/2013/12/22/better-linux-disk-caching-performance-vm-dirty_ratio/)

- [openSUSE Leap 42.3 System Analysis and Tuning Guide, Part V, Chapter 13 Tuning the Task Scheduler](https://doc.opensuse.org/documentation/leap/tuning/html/book.sle.tuning/cha.tuning.taskscheduler.html)

## 7.0 法律声明

版权所有 ©2006-2018 SUSE LLC和贡献者保留一切权利。

根据GNU自由文档授权1.2版或（根据您的选择）1.3版的相关条款，允许复制，分发和/或修改本文档；不可变的是此项版权声明和许可部分。许可证版本1.2的副本包含在标题为GNU自由文档授权的部分中。

SUSE，SUSE图标和YaST是SUSE LLC在美国和其他国家/地区的注册商标。有关SUSE商标，请访问http://www.suse.com/company/legal/获取更多信息。 Linux是Linus Torvalds的注册商标。本文档中提及的所有其他名称或商标可能是其各自所有者的商标或注册商标。

本文是名为“SUSE Best Practices”系列文档的一部分。该系列中的文件由SUSE的员工和第三方自愿提供。

本文档中的所有信息都经过精心编制，并且非常注重细节。但是，这并不能保证完全准确。

因此，我们需要明确声明，SUSE LLC及其关联公司，作者和翻译人员均不对可能的错误或其后果承担责任。下面我们提请您注意文章发布所遵循的许可。

## 8.0 GNU自由文档授权

著作权所有 (C) 2000, 2001, 2002 Free Software Foundation, Inc. 51 Franklin St, Fifth Floor, Boston, MA 02110-1301 USA。允许每个人复制和发布本授权文件的完整副本，但不允许对它进行任何修改。

0. 导言

本授权的目的在于作为一种手册、教科书或其它的具有功能性的有用文件获得自由：确保每一个人都具有复制和重新发布它的有效自由，而不论是否作出修改，也不论其是否具有商业行为。其次，本授权保存了作者以及出版者由于他们的工作而 得到 名誉的方式，同时也不被认为应该对其他人所作出的修改而担负责任。

本授权是一种’copyleft’，这表示文件的衍生作品本身必须具有相同的自由涵义。它补足了GNU 公共通用授权-- 一种为了自由软件而设计的’copyleft’授权。

我们设计了本授权是为了将它使用到自由软件的使用手册上，因为自由软件需要自由的文档：一种自由的程序应该提供与此软件具有相同的自由的使用手册。但是本授权并不被限制在软件使用手册的应用上；它可以被用于任何以文字作基础的作品，而不论其主题内容，或者它是否是一个被出版的印刷书籍。我们建议本授权主要应用在以使用说明或提供参考作为目的的作品上。

1. 效力与定义

本授权的效力在于任何媒体中的任何的使用手册或其它作品，只要其中包含由版权所有人所指定的声明，说明它可以在本授权的条款下被发布。这样的一份声明提供了全球范围内的,免版税的和没有期限的许可，在此所陈述条件下使用那个作品。以下所称的文件，指的是任何像这样的使用手册或作品。公众中的任何成员都是被许可人，并且称作为你。如果你以一种需要在版权法下取得允许的方式进行复制、修改或发布作品，你就接受了这项许可。

“修改版本”指的是任何包含文件或是它的其中一部份，不论是逐字的复制或是经过修正，或翻译成其它语言的任何作品。

“次要章节”是一个具名的附录，或是文件的本文之前内容的章节，专门用来处理文件的出版者或作者，与文件整体主题(或其它相关内容)的关系，并且不包含任何可以直接落入那个整体主题的内容。(因此，如果文件的部分内容是作为数学教科书，那么其次要章节就可以不用来解释任何数学。)它的关系可以是与主题相关的历史连接，或是与其相关的法律、商业、哲学、伦理道德或政治立场。

“不变章节”是标题已被指定的某些次要章节，在一个声明了是以本授权加以发行的文件中，依此作为不变章节。如果一个章节并不符合上述有关于次要的定义时，则它并不允许被指定为不变。文件可以不包含不变章节。如果文件并没有指出任何不变章节，那么就是没有。

“封面文字”是某些被加以列出的简短文字段落，在一个声明了是以本授权加以发行的文件中，依此作为前封面文字或后封面文字。前封面文字最多可以包含 5 个单词，后封面文字最多可以包含 25 个单词。

文件的”透明”拷贝指的是一份机器可读的拷贝，它以一种一般公众可以取得其规格说明的格式来表现，适合于直接用一般文字编辑器、一般点阵图像程序用于由图元像素构成的影像或一些可以广泛取得的绘图程序用于由向量绘制的图形直接地进行修订；并且适合于输入到文字格式化程式，或是可以自动地转换到适合于输入到文字格式化程序的各种格式。一份以透明以外的档案格式所构成的拷贝，其标记或缺少标记，若是被安排成用来挫折或是打消读者进行其后续的修改，则此拷贝并非透明。一种影像格式，如果仅仅是用来充斥文本的资料量时，就不是透明的。一个不是透明的拷贝被称为混浊。

透明拷贝适合格式的例子包括有：没有标记的纯 ASCII、Texinfo 输入格式、LaTeX 输入格式、使用可以公开取得其 DTD 的 SGML 或 XML、合乎标准的简单 HTML、PostScript 或 PDF。透明影像格式的例子有 PNG、XCF 和 JPG 。混浊格式包括只能够以私人文书处理器阅读以及编辑的私人格式、DTD 以及或处理工具不能够一般地加以取得的 SGML 或 XML、以及由某些文书处理器只是为了输出的目的而做出的，由机器制作的 HTML、PostScript 或 PDF 。

标题页对一本印刷书籍来说，指的是标题页本身，以及所需要用来容纳本授权必须出现在标题页的易读内容的，如此的接续数页。对于并没有任何如此页面的作品的某些格式，标题页指的是本文主体开始之前作品标题最显着位置的文字。

出版者指的是把那些文本副本发布给公众的任何人或实体。

一个标题为 XYZ的章节指的是文件的一个具名的次要单元，其标题精确地为 XYZ 或是将 XYZ 包含在跟着翻译为其它语言的 XYZ 文字后面的括号内 -- 这里 XYZ 代表的是名称于下提及的特定章节，像是感谢、贡献、背书或历史。当你修改文件时，给像这样子的章节保存其标题指的是，它保持为一个根据这个定义的标题为 XYZ的章节。

文件可以在用来陈述本授权效力及于文件的声明后，包括担保放弃。这些担保放弃被考虑为以提及的方式，包括在本授权中，但是只被看作为放弃担保之用：任何这些担保放弃可能会有的其它暗示都是无效的，并且也对本授权的含义没有影响。

以下是有关复制、发布及修改的明确条款及条件。

2. 逐字的复制

你可以复制或发布文件于任何媒体，而不论其是否具有商业买卖行为，其条件为具有本授权、版权声明和许可声明，说明本授权效力于文件的所有重制拷贝，并且你没有增加任何其它条件到本授权的条件中。你不可以使用技术手段，来妨碍或控制你所制作或发布的拷贝阅读或进一步的发布。然而，你可以接受补偿以作为拷贝的交换。如果你发布了数量足够大的拷贝，你也必须遵循第三条的条件。

你也可以在上述的相同条件下借出拷贝，并且你可以公开地陈列拷贝。

3. 大量地复制

如果你出版文件的印刷拷贝或者通常具有印刷封面的媒体的拷贝，数量上超过一百个单位，而且文件许可声明要求有封面文字，那么你必须将这些拷贝附上清楚且易读的文字：前封面文字于前封面上、后封面文字于后封面上。这两种封面必须清楚易读地辨认出，你是这些拷贝的出版者。前封面文字必须展示完整的标题，而标题的文字应当同等地显著可见。你可以增加额外的内容于封面上。仅在封面作出改变的复制，只要它们保存了文件的标题，并且满足了这些条件，可以在其它方面被看作为逐字的复制。

如果对于任意一个封面所需要的文字，数量过于庞大以至于不能符合易读的原则，你应该在实际封面的最前面列出所能符合易读原则的内容，然后将剩下的接续在相邻的页面。

如果你出版或发布数量超过一百个单位文件的混浊拷贝，你必须与此混浊拷贝一起包含一份机器可读的透明拷贝，或是与一份混浊拷贝一起或其陈述一个电脑网络位址，使一般的网络使用公众具有存取权，可以使用公开标准的网络协定，下载一份文件的完全透明拷贝，此拷贝中并且没有增加额外的内容。如果你使用后面的选项，当你开始大量地发布混浊拷贝时，你必须采取合理的审慎步骤，以保证这个透明拷贝将会在发布的一开始就保持可供存取，直到你最后一次发布那个发行版的一份混浊拷贝给公众后，至少一年为止。以保证这个透明拷贝，将会在所陈述的位址保持如此的可存取性，直到你最后一次发布那个发行版直接或经由你的代理商或零售商的一份混浊拷贝给公众后，至少一年为止。

你被要求，但不是必须，在重新发布任何大数量的拷贝之前与文件的作者联络，给予他们提供你一份文件的更新版本的机会。

4. 修改

你可以在上述第二条和第三条的条件下，复制和发布文件的修改版本，其条件为你要精确地在本授权下发布修改版本，且修改版本补足了文件的角色，从而允许修改版本的发布和修改权利给任何拥有它拷贝的人。另外，你必须在修改版本中做这些事：

第一款、 在标题页或在封面上使用，如果有与先前版本不同的文件，应该被列在文件的历史章节不同的标题。如果版本的原始出版者允许，你可以使用与某一个先前版本相同的标题。

第二款、 在修改版本的标题页上列出担负作者权的一个或多个人或实体作为作者，并且列出至少五位文件的主要作者。如果少于五位，则列出全部的主要作者，除非他们免除了你这个要求。

第三款、 在标题页陈述修改版本的出版者的名称作为出版者。

第四款、 保存文件的所有版权声明。

第五款、 为你的修改增加一个与其它版权声明相邻的适当的版权声明。

第六款、 在版权声明后面，以授权附录所显示的形式，包括一个给予公众在本授权条款下使用修改版本的许可声明。

第七款、 在那个许可声明中保存恒常章节和文件许可声明中必要封面文字的全部列表。

第八款、 包括一个未被改变的本授权的副本。

第九款、 保存标题为历史的章节和其标题，并且增加一项至少陈述如同在标题页中所给的修改版本的标题、年份、新作者和出版者。如果在文件中没有标题为历史的章节，则制作出一个陈述如同在它的标题页中所给的文件的标题、年份、新作者和出版者，然后增加一项描述修改版本如前面句子所陈述的情形。

第十款、 如果有的话，保存在文件中为了给公众存取文件的透明拷贝，而给予的网络位址，以及同样地在文件中为了它所根据的先前版本，而给予的网络位址。这些可以被放置在历史章节。你可以省略一个在文件本身之前，已经至少出版了四年的作品的网络位址，或是如果它所参照的那个版本的原始出版者给予允许的情形下也可以省略它。

第十一款、 在任何标题为感谢或贡献的章节，保存章节的标题，并且在那章节保存到那时候为止，每一个贡献者的感谢以及或贡献的所有声色。

第十二款、 保存文件的所有恒常章节，于其文字以及标题皆不得变更。章节号码或其同等物并不被认为是章节标题的一部份。

第十三款、 删除任何标题为背书的章节。这样子的章节不可以被包括在修改版本中。

第十四款、 不要重新命名任何现存的章节，而使其标题为背书，或造成与任何恒常章节相冲突的标题。

第十五款、 保存任何的担保放弃。

如果修改版本包括新的本文之前内容的章节，或合乎作为次要章节的附录，并且没有包含复制自文件的内容，则你具有选择可以指定一些或全部这些章节为恒常的。要这样做，将它们的标题增加到在修改版本许可声明中的恒常章节列表中。这些标题必须可以和任何其它章节标题加以区别。

你可以增加一个标题为背书的章节，其条件为它仅只包含由许多团体所提供的你的修改版本的背书 -- 举例来说，同侪评审的说明，或本文已经被一个机构认可为一个标准的权威定义。

你可以增加一个作为前封面文字的最多五个字的段落，以及一个作为后封面文字的最多二十五个字的段落，到修改版本的封面文字列表的后面。前封面文字和后封面文字都只能有一个段落，可以经由任何一个实体，或经由任何一个实体所作出的安排而被加入。如果文件已经在同样的封面包括了封面文字 -- 先前由你或由你所代表的相同实体所作出的安排而加入，则你不可以增加另外一个；但是你可以在先前出版者的明确允许下替换掉旧的。 文件的作者和出版者并不由此授权，而给予允许使用他们的名字以为了或经由声称或暗示任何修改版本背书为自己所应得的方式而获得名声的权利。

5. 组合文件

你可以在上述第四条的条款中对于修改版本的定义之下，将文件与其它在本授权下发行的文件组合起来，其条件是你要在组合品中，包括所有原始文件的所有恒常章节，不做修改，同时在组合作品的许可声明中将它们全部列为恒常章节，并且你要保存它们所有的担保放弃。

组合作品只需包含本授权的一份副本，并且重复的恒常章节可以仅以单一个拷贝来取代。如果名称重复但内容不同的恒常章节，则将任此章节的标题，以在它的后面增加的方式加以独特化，如果已知的话，于括号中指出那个章节的原始作者或出版者的名称，或是指定一个独特的号码。在此组合作品许可声明中恒常章节的列表中，对其章节标题也作出相同的调整。

在组合品中，你必须组合在不同原始文件中，标题为历史的任何章节，形成一个标题为历史的章节；同样组合任意标题为感谢或贡献的章节。你必须删除标题为背书的所有章节。

6. 文件的收集

你可以制作含有文件以及其它以本授权发行文件的收集品，并且将本授权对不同文件中的个别副本，以单一个包括在收集品的副本取代，其条件是你要遵循在其它方面，给予一个文件逐字复制的允许本授权的规则。

你可以从这样的一个收集品中抽取出一份单一的文件，并且在本授权下将它单独地发布，其条件是你要在抽取出的文件中插入本授权的一份副本，并且在关于那份文件的逐字的复制的所有其它方面，遵循本授权。

7. 独立作品的聚集

一个文件的编辑物，其中或附加于储存物或发布媒体的一册的，具有其它分别且独立的文件或作品的衍生品，如果经由编辑而产生的版权，并没有用来限制此编辑物使用者的法律权力，而超过了个别的作品所允许的，则被称为一个聚集品。当文件中包括一个聚集品，本授权的效力并不仅在于此聚集品中的，于其本身并非文件的衍生作品的其它作品。 如果第三条的封面文字要求效力于这些文件的拷贝，并且文件的篇幅少于整个聚集品的一半，则文件的封面文字可以被放在只围绕着文件，并于聚集品内部的封面或是电子的封面同等物上，如果文件是以电子的形式出现的话。否则它们必须出现在绕着整个聚集品的印刷封面上。

8. 翻译

翻译被认为一种修改，因此你可以在第四条的条款下发布文件的翻译。用翻译更换恒常章节需要取得版权所有者的特别允许，但是你可以包括部份或所有恒常章节的翻译，使其附加到这些恒常章节的原始版本之中。你可以包括本授权、文件中的所有许可声明和任何的担保放弃的翻译，其条件为你也必须包括本授权的原始英文版本，以及这些声明与放弃的原始版本。如果发生翻译与本授权、声明或放弃的原始版本有任何的不同意时，将以原始版本为准。

如果在文件中的章节被标题为感谢、贡献或历史，则保存标题第一条的必要条件第四条，典型上将会需要去更动实际的标题。

9. 终止

你不可以复制、修改、在本授权下再设定额外条件的次授权或发布文件，除非明白地表示是在本授权所规范的条件下进行。任何其它的复制、修改、在本授权下再设定额外条件的次授权、或发布文件的意图都是无效的，并且将会自动地终止你在本授权下所被保障的权利。

然而，如果你终止所有违反本授权的行为，特定版权所有人会暂时恢复你的授权直到此版权所有者明确并最终地终止你的授权。或者特定版权所有人永久地恢复你的授权如果此版权所有人在停止违反授权后的60天内没有通过合理的方式通知你违反授权。

此外，此版权所有者用一些合理的方式通知你违反了授权规定，也是你第一次从此版权所有者收到违反授权的通知，并且你在收到通知后的30天内终止了这种行为，那么此版权所有者会永久地恢复你的授权。

你的权利在此章节的终止并不代表终止得到你的拷贝和权利的当事人的授权。如果你的权利被终止而没有被永久性地恢复，那么你将没有任何权利去使用此章节的全部或部分资料和资料的拷贝。

10. 本授权的未来改版

自由软件基金会可能偶尔会出版自由文件授权的新修订过的版本。这种新版本在精神上将会类似于现在的版本，但在细节上可能会有不同，以对应新的问题或相关的事。请见 http://www.gnu.org/copyleft/。

本授权的任何版本都被指定一个可供区别的版本号码。如果文件指定一个效力于它的特定号码版本的本授权或任何以后的版本，你就具有选择遵循指定的版本，或任何已经由自由软件基金会出版的后来版本并且不是草稿的条款和条件。如果文件并没有指定一个本授权的版本号码，你就可以选择任何一个曾由自由软件基金会所出版不是草稿的版本。如果文件指定一个代理能够决定本授权未来的那一种版本可以用，而且还指定代理公开声明如果你接受一种版本你将会永久地被授权为文本选择此版本的权利。 11.重新授权

“MMC 网站”是指任何发布有著作版权作品的网站服务器，也为任何人提供卓越的设施去编辑一些作品。任何人都可编辑的一种公众维基就是这种服务器的一个例子。包含在这个网站的”MMC”是指任何一套在MMC网站上发布的具有著作版权的作品。

“CC-BY-SA”是指 Creative Commons Attribution-Share Alike 3.0 授权，它是被“知识共享组织” 颁布的，“知识共享组织” 是一家非赢利性的，在圣弗朗西斯科，加利福尼亚具有重要的商业地位的组织。 而且未来的“copyleft”版本的授权也是被同一个组织发布的。

“合并”是指以整体或作为另一个文本的部分发布或重新发布一个文本。

MMC有重新授权的资格，如果它是在MMC下授权；或者所有的作品首次发布并非在MMC下授权，后来以整体或部分合并到MMC下，它们没有覆盖性文本或不变的章节并且在2008年11月1日之前合并的。

那么MMC网站的操作者会在2009年8月1日之前的任何时间在同一网站重新发布包含在这一网站的MMC是经过CC-BY-SA 授权的，只要那个MMC有资格重新授权。

授权附录：如何使用本授权用于你的文件

为使用本授权成为你撰写成的一份文件，必须在文件中包括本授权的一份复本，以及标题页的后面包括许可声明：

Copyright (c) YEAR YOUR NAME. Permission is granted to copy, distribute and/or modify this document under the terms of the GNU Free Documentation License, Version 1.2 or any later version published by the Free Software Foundation; with no Invariant Sections, no Front-Cover Texts, and no Back-Cover Texts. A copy of the license is included in the section entitled "GNU Free Documentation License".

如果你有恒常章节、前封面文字和后封面文字，请将with... Texts这一行以这些文本取代： with the Invariant Sections being LIST THEIR TITLES, with the Front-Cover Texts being LIST, and with the Back-Cover Texts being LIST.

如果你有不具封面文字的恒常章节，或一些其它这三者的组合，将可选择的二项合并以符合实际情形。

如果你的文件中包含有并非微不足道的程序码范例，我们建议这些范例平行地在你的自由软件授权选择下，比如以 GNU General Public License 的自由软件授权来发布，从而允许它们作为自由软件而使用。
