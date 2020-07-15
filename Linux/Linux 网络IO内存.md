
## IO
linux系统的输入输出（I/O）性能和状态，可以通过sysstat命令包中的iostat命令来进行实时的监控查看,这里介绍iostat命令的一些常用操作以便用于输出CPU和磁盘I/O相关的统计信息。

命令格式:
iostat [ -c ] [ -d ] [ -h ] [ -N ] [ -k | -m ] [ -t ] [ -V ] [ -x ] [ -z ] [ device [...] | ALL ] [ -p [ device [,...] | ALL ] ] [ interval [ count ] ]iostat


```
iostat -d -x -k 1 3

-x 必加项：man中解释显示出 extended report。不用管，就知道-x 会显示 await  svctm  %util等等，不加不显示就完了

-d，非必加项 ：只显示 硬盘的相关内容（如果没有-d 会多出一项avg-cpu，也有一堆列）

-k，非必加项 ：将关于吞吐的两个指标 用Kb单位显示，就是下面案例中的 第5、6项（rkB wkB）

1 3 ，每一秒 显示一次。总共显示3次。很好理解吧，不理解随便改改 就知道啥意思了

```

```
各个参数的说明

 -c 仅显示CPU统计信息.与-d选项互斥.
 -d 仅显示磁盘统计信息.与-c选项互斥.
 -k 以K为单位显示每秒的磁盘请求数,默认单位块.
 -p device | ALL
  与-x选项互斥,用于显示块设备及系统分区的统计信息.也可以在-p后指定一个设备名,如:
  # iostat -p hda
  或显示所有设备
  # iostat -p ALL
 -t    在输出数据时,打印搜集数据的时间.
 -V    打印版本号和帮助信息.
 -x    输出扩展信息.iostat的简单使用
```

```
解释一下各个输出项的含义：
avg-cpu段:
%user: 在用户级别运行所使用的CPU的百分比.
%nice: nice操作所使用的CPU的百分比.
%sys: 在系统级别(kernel)运行所使用CPU的百分比.
%iowait: CPU等待硬件I/O时,所占用CPU百分比.
%idle: CPU空闲时间的百分比.
Device段:
tps: 每秒钟发送到的I/O请求数.
Blk_read /s: 每秒读取的block数.
Blk_wrtn/s: 每秒写入的block数.
Blk_read:   读入的block总数.
Blk_wrtn:  写入的block总数.入门使用
```

```
Linux 2.6.18-92.el5xen    02/03/2009
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           1.10    0.00    4.82   39.54    0.07   54.46
Device:         rrqm/s   wrqm/s   r/s   w/s   rsec/s   wsec/s avgrq-sz avgqu-sz   await  svctm  %util
sda               0.00     3.50  0.40  2.50     5.60    48.00    18.48     0.00    0.97   0.97   0.28
sdb               0.00     0.00  0.00  0.00     0.00     0.00     0.00     0.00    0.00   0.00   0.00
sdc               0.00     0.00  0.00  0.00     0.00     0.00     0.00     0.00    0.00   0.00   0.00
sdd               0.00     0.00  0.00  0.00     0.00     0.00     0.00     0.00    0.00   0.00   0.00
sde               0.00     0.10  0.30  0.20     2.40     2.40     9.60     0.00    1.60   1.60   0.08
sdf              17.40     0.50 102.00  0.20 12095.20     5.60   118.40     0.70    6.81   2.09  21.36
sdg             232.40     1.90 379.70  0.50 76451.20    19.20   201.13     4.94   13.78   2.45  93.16
```
```
rrqm/s:   每秒进行 merge 的读操作数目。即 delta(rmerge)/s
wrqm/s:  每秒进行 merge 的写操作数目。即 delta(wmerge)/s
r/s:           每秒完成的读 I/O 设备次数。即 delta(rio)/s
w/s:         每秒完成的写 I/O 设备次数。即 delta(wio)/s
rsec/s:    每秒读扇区数。即 delta(rsect)/s
wsec/s:  每秒写扇区数。即 delta(wsect)/s
rkB/s:      每秒读K字节数。是 rsect/s 的一半，因为每扇区大小为512字节。(需要计算)
wkB/s:    每秒写K字节数。是 wsect/s 的一半。(需要计算)
avgrq-sz: 平均每次设备I/O操作的数据大小 (扇区)。delta(rsect+wsect)/delta(rio+wio)
avgqu-sz: 平均I/O队列长度。即 delta(aveq)/s/1000 (因为aveq的单位为毫秒)。
await:    平均每次设备I/O操作的等待时间 (毫秒)。即 delta(ruse+wuse)/delta(rio+wio)
svctm:   平均每次设备I/O操作的服务时间 (毫秒)。即 delta(use)/delta(rio+wio)
%util:      一秒中有百分之多少的时间用于 I/O 操作，或者说一秒中有多少时间 I/O 队列是非空的。即 delta(use)/s/1000 (因为use的单位为毫秒)
```

- 习惯性第一眼先看 %util，如果不高，其他值就不看了

- 如果%util存在问题，第二眼就去看svctm列，看这个值高不高。一般最多也就在15左右。

- svctm和磁盘性能息息相关，除了硬件性能，如果IOPS（tps）增加也会增加该速度 

- 所以这里我们要将svctm和await一起看。如果 svctm 比较接近 await，说明 I/O 几乎没有在队列中等待的时间   

- avgqu-sz 注意这个，如果平均超过2  ，说明IO系统有压力了



```
总结一下：

指标1 IOPS                 ：r/s+w/s=IOPS（tps)

指标2 吞吐量               ：rkB/s,wkB/s。 加在一起就是当前每秒吞吐量

指标3平均I/O数据大小：avgrq-sz

指标4 百分比               ： util%

指标5 服务时间           ：svctm

指标6 IO等待队列长度：avgqu-sz

指标7 等待时间           ：await
```



- 如果 %util 接近 100%，说明产生的I/O请求太多，I/O系统已经满负荷，该磁盘
可能存在瓶颈。
idle小于70% IO压力就较大了,一般读取速度有较多的wait.

- 同时可以结合vmstat 查看查看bi参数(等待资源的进程数)和wa参数(IO等待所占用的CPU时间的百分比,高过30%时IO压力高)

另外还可以参考

一般:
svctm < await (因为同时等待的请求的等待时间被重复计算了)，
svctm的大小一般和磁盘性能有关:CPU/内存的负荷也会对其有影响，请求过多也会间接导致 svctm 的增加。

await: await的大小一般取决于服务时间(svctm) 以及 I/O 队列的长度和 I/O 请求的发出模式。

- 如果 svctm 比较接近 await，说明I/O 几乎没有等待时间；
- 如果 await 远大于 svctm，说明 I/O队列太长，应用得到的响应时间变慢，
- 如果响应时间超过了用户可以容许的范围，这时可以考虑更换更快的磁盘，调整内核 elevator算法，优化应用，或者升级 CPU。

队列长度(avgqu-sz)也可作为衡量系统 I/O 负荷的指标，但由于 avgqu-sz 是按照单位时间的平均值，所以不能反映瞬间的 I/O 洪水

## 内存
在linux下，使用top，vmstat,free等命令查看系统或者进程的内存使用情况时，经常看到buff/cache memeory，swap，avail Mem等。


1. top命令:top命令是Linux下常用的性能分析工具，能够实时显示系统中各个进程的资源占用状况，类似于Windows的任务管理器。运行 top 命令后，CPU 使用状态会以全屏的方式显示，并且会处在对话的模式 -- 用基于 top 的命令，可以控制显示方式等等。退出 top 的命令为 q （在 top 运行中敲 q 键一次）。

2. free [option] 


```
-b 　# 以Byte为单位显示内存使用情况
-k 　# 以KB为单位显示内存使用情况 
-m 　# 以MB为单位显示内存使用情况
-g   # 以GB为单位显示内存使用情况
-h   # 自动转换单位（最常用）
-o 　# 不显示缓冲区调节列 
-s <间隔秒数> 　# 持续观察内存使用状况 
-t 　# 显示内存总和列 
-V 　# 显示版本信息
```
在终端输入free。结果如下：

```
[@bjzw_106_203 ~]# free
             total       used       free     shared    buffers     cached
Mem:       8182340    7909480     272860          0     463820    5228244
-/+ buffers/cache:    2217416    5964924
Swap:      1048568       2612    1045956



Mem ：表示物理内存的统计（系统已使用、空闲的内存）。
-/+ buffers/cache： 应用程序已使用的、空闲的物理内存。
Swap：交换分区的内存统计。
total：表示物理内存总量(total = used + free)
used：表示总计分配给缓存（包含buffers 与cache ）使用的数量，但其中可能部分缓存并未实际使用。
free：未被分配的内存。
shared：多个进程共享的内存总额。
buffers：系统分配但未被使用的buffers 数量。
cached：系统分配但未被使用的cache 数量。
```

## 网络

netstat 查看网络状况

netstat命令用来打印网络连接状况、系统所开放端口、路由表等信息

常用的关于netstat的命令就是这个 netstat -lnp （打印当前系统启动哪些端口）以及 netstat -an （打印网络连接状况）

netstat命令是一个监控TCP/IP网络的非常有用的工具，它可以显示路由表、实际的网络连接以及每一个网络接口设备的，netstat命令的功能是显示网络连接、路由表和网络接口信息，可以让用户得知目前都有哪些网络连接正在运作。
```
      该命令的一般格式为：
      netstat [选项]
      命令中各选项的含义如下：
      -a 显示所有socket，包括正在监听的。
      -c 每隔1秒就重新显示一遍，直到用户中断它。
      -i 显示所有网络接口的信息，格式同“ifconfig -e”。
      -n 以网络IP地址代替名称，显示出网络连接情形。
      -r 显示核心路由表，格式同“route -e”。
      -t 显示TCP协议的连接情况。
      -u 显示UDP协议的连接情况。
      -v 显示正在进行的工作。

```




```
1,ifconfig   
   查看当前网卡的配置

2,ping ip地址
   这个不必细说了

3,mtr ip地址
   这个是traceroute和ping的结合体

4,traceroute ip地址
   追踪到指定ip地址所经过的路由

5,route
　查看本机的静态路由表
```

