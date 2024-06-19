# Linux常见命令

## 1、统计某一个文件中的 牛客网 出现了几次

```shell
grep "牛客网" 文件名 | wc -l
```

## 2、crontab命令

```shell
* * * * * command
分钟 小时 天 月 星期 具体命令
```

## 3、开启/关闭命令进程

```shell
jobs //查看当前有哪些命令进程
bg %jobnumber //将一个前台命令进程放到后台执行
fg %jobnumber //将一个后台命令进程放到前台执行
```

## 4、给某个文件增加：所有人可执行，同组可写的权限

```shell
chmod a+x g+w filename
```

权限对应数字：

```
u-g-o：用户-组内用户-其他用户
r-w-x：用4-2-1表示。
组外成员o的权限为只读：r-- = 4
所有者a有全部权限：rwx; = 7
组内g的权限为读与写:rw- = 6
```

## 5、合并两个文件

```shell
cat file1 file2 > newfile
```

6、top查看内存占用

```shell
# 查看某个进程id
ps -ef | grep nginx
# 查看指定进程id信息，cpu占用，内存占用
top -p xxxxxx
PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
1172601 root      20   0   97688   7100   1788 S   0.0   0.4   0:00.02 nginx
# 也可以查看指定进程id信息，cpu占用，内存占用
ps -aux | grep nginx
root     1172601  0.0  0.3  97688  7100 ?        S    Aug23   0:00 nginx: master process /usr/sbin/nginx
33       1172604  0.0 23.7 532116 441436 ?       S    Aug23   0:00 nginx: worker process
# 0.0 0.3是cpu和内存占用，7100是内存消耗，7100k=7m，441436k=440m
```

## 7、for循环打印指定字符串

```shell
#!/bin/bash
for i in $(seq 1 98)
do
echo "- http://106.14.127.79:8848/images/img/img("$i").webp"
done
```

## 8、grep查找文件内容

```shell
在查找字符串时，有时需要指定文件后缀方式查找，命令如下：
grep -rn --include="*.properties" 'tableIsSharding' ./  
```

## 9、使用less命令检索文件内容

```shell
less
less 工具也是对文件或其它输出进行分页显示的工具，应该说是linux正统查看文件内容的工具，功能极其强大。less 的用法比起 more 更加的有弹性。 在 more 的时候，我们并没有办法向前面翻， 只能往后面看，但若使用了 less 时，就可以使用 [pageup] [pagedown] 等按 键的功能来往前往后翻看文件，更容易用来查看一个文件的内容！除此之外，在 less 里头可以拥有更多的搜索功能，不止可以向下搜，也可以向上搜。

-m  显示类似more命令的百分比
-N  显示每行的行号
/字符串：向下搜索“字符串”的功能
?字符串：向上搜索“字符串”的功能
n：重复前一个搜索（与 / 或 ? 有关）
N：反向重复前一个搜索（与 / 或 ? 有关）
b  向前翻一页
d  向后翻半页
Q  退出less 命令
G - 移动到最后一行
g - 移动到第一行

我经常使用 less -mN 来查看文件，然后使用上面命令基本够用
```

## 10、端口占用

```shell
netstat -tunlp |grep 8080
-t 仅显示tcp相关选项
-u 仅显示udp相关选项
-n 能显示数字的全部转化为数字
-l 仅列出在Listen(监听)的服务状态
-p 显示建立相关链接的程序名
```

## 11、进程杀死

```shell
kill -9 xxx
-9 强制杀死进程
-15 正常终止进程（默认）
```

## 12、文件名查找

```shell
find 路径 -type 类型 -name "文件名"
类型：普通文件 f、目录d、符号链接l、块设备文件b、字符设备文件c、socket文件s、管道文件p
```

## 13、top命令查看系统负载

### 统计信息

第一行：

top - 17:22:00 up 772 days, 23:58,  2 users,  load average: 0.30, 0.19, 0.06

- up：机器运行了多长时间。
- users：当前登录用户数。
- load average：系统负载，即任务队列的平均长度。三个数值分别为 1分钟、5分钟、15分钟前到现在的平均值。

第二行：

Tasks: 140 total,   1 running, 139 sleeping,   0 stopped,   0 zombie

- Tasks：当前有多少进程。
- running：正在运行的进程数。
- sleeping：正在休眠的进程数。
- stopped：停止的进程数。
- zombie：僵尸进程数。

这里running越多，服务器自然压力就越大。

第三行：

%Cpu(s):  2.0 us,  2.0 sy,  0.0 ni, 95.6 id,  0.0 wa,  0.3 hi,  0.0 si,  0.0 st

- us：用户空间占CPU的百分比（像shell程序、各种语言的编译器、各种应用、web服务器和各种桌面应用都算是运行在用户地址空间的进程，这些程序如果不是处于idle状态，那么绝大多数的CPU时间都是运行在用户态）。  

- sy：内核空间占CPU的百分比（所有进程要使用的系统资源都是由Linux内核处理的，对于操作系统的设计来说，消耗在内核态的时间应该是越少越好，在实践中有一类典型的情况会使sy变大，那就是大量的IO操作，因此在调查IO相关的问题时需要着重关注它）。  

- ni：用户进程空间改变过优先级（ni是nice的缩写，可以通过nice值调整进程用户态的优先级，这里显示的ni表示调整过nice值的进程消耗掉的CPU时间，如果系统中没有进程被调整过nice值，那么ni就显示为0）。  

- id：空闲CPU占用率。  

- wa：等待输入输出的CPU时间百分比（和CPU的处理速度相比，磁盘IO操作是非常慢的，有很多这样的操作，比如，CPU在启动一个磁盘读写操作后，需要等待磁盘读写操作的结果。在磁盘读写操作完成前，CPU只能处于空闲状态。Linux系统在计算系统平均负载时会把CPU等待IO操作的时间也计算进去，所以在我们看到系统平均负载过高时，可以通过wa来判断系统的性能瓶颈是不是过多的IO操作造成的）。  

- hi：硬中断占用百分比【硬中断是硬盘、网卡等硬件设备发送给CPU的中断消息，当CPU收到中断消息后需要进行适当的处理(消耗CPU时间)】。  

- si：软中断占用百分比（软中断是由程序发出的中断，最终也会执行相应的处理程序，消耗CPU时间）。  

- st：steal time。

第四行：

MiB Mem :   1818.6 total,     94.9 free,   1181.8 used,    542.0 buff/cache

- total：物理内存总量。
- free：空闲内存量。
- used：使用的内存量。
- buffer/cache：用作内核缓存的内存量。

第五行：

MiB Swap:      0.0 total,      0.0 free,      0.0 used.    459.7 avail Mem

- total：交换区内存总量。
- free：空闲交换区总量。
- used：使用的交换区总量。
- buffer/cache：缓冲的交换区总量。

第四第五行分别是内存信息和swap信息，所有程序的运行都是在内存中进行的，所以内存的性能对与服务器来说非常重要。不过当内存的free变少的时候，其实我们并不需要太紧张。真正需要看的是Swap中的used信息。

Swap分区是由硬盘提供的交换区，当物理内存不够用的时候，操作系统才会把暂时不用的数据放到Swap中。所以当这个数值变高的时候，说明内存是真的不够用了。

### 进程信息

```shell
    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
 723317 root      10 -10  259680  44268  19220 S   1.3   2.4   0:56.94 AliYunDunMonito
 723306 root      10 -10  132984  13888  11768 S   0.7   0.7   0:26.63 AliYunDun
    826 root      20   0  216332   5148   2984 S   0.3   0.3  50:57.75 sssd_nss
```

```text
PID  进程id
USER  进程所有者的用户名
PR       优先级
NI    nice值，负值表示高优先级，正值表示低优先级
VIRT  进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
RES    进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
SHR    共享内存大小，单位kb
S    进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程
%CPU  上次更新到现在的CPU时间占用百分比
%MEM  进程使用的物理内存百分比
TIME+  进程使用的CPU时间总计，单位1/100秒
COMMAND  命令名/命令行
```

**默认情况下仅显示比较重要的 PID、USER、PR、NI、VIRT、RES、SHR、S、%CPU、%MEM、TIME+、COMMAND 列，还有一些参数，例如：**

```text
PPID  父进程id
GROUP   进程所有者的组名
SWAP:  进程使用的虚拟内存中被换出的大小
CODE  可执行代码占用的物理内存大小，单位kb
DATA  可执行代码以外的部分(数据段+栈)占用的物理内存大小，单位kb
nFLT  页面错误次数
nDRT  最后一次写入到现在，被修改过的页面数。
WCHAN  若该进程在睡眠，则显示睡眠中的系统函数名
Flags  任务标志
```

默认进入top时，各进程是按照CPU的占用量来排序的。

敲top后，按键盘数字“1”可以监控每个逻辑CPU的状况

敲top后，输入u，然后输入用户名，则可以查看相应的用户进程

敲top后，输入shift+m，进程列表将根据内存占用排序。

top -p xxxxxx，将查询某个进程xxxxxx的信息。





## 14、在Centos中修改默认JDK版本

```shell
[root@localhost jdk21]# java -version
java version "17.0.11" 2024-04-16 LTS
Java(TM) SE Runtime Environment (build 17.0.11+7-LTS-207)
Java HotSpot(TM) 64-Bit Server VM (build 17.0.11+7-LTS-207, mixed mode, sharing)
[root@localhost jdk21]# alternatives --config java

共有 4 个提供“java”的程序。

  选项    命令
-----------------------------------------------
   1           java-1.7.0-openjdk.x86_64 (/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.261-2.6.22.2.el7_8.x86_64/jre/bin/java)
   2           java-1.8.0-openjdk.x86_64 (/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.402.b06-1.el7_9.x86_64/jre/bin/java)
*+ 3           /usr/lib/jvm/jdk-17-oracle-x64/bin/java
   4           /usr/lib/jvm/java-21-amazon-corretto/bin/java

按 Enter 保留当前选项[+]，或者键入选项编号：
[root@localhost jdk21]# alternatives --config java

共有 4 个提供“java”的程序。

  选项    命令
-----------------------------------------------
   1           java-1.7.0-openjdk.x86_64 (/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.261-2.6.22.2.el7_8.x86_64/jre/bin/java)
   2           java-1.8.0-openjdk.x86_64 (/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.402.b06-1.el7_9.x86_64/jre/bin/java)
*+ 3           /usr/lib/jvm/jdk-17-oracle-x64/bin/java
   4           /usr/lib/jvm/java-21-amazon-corretto/bin/java

按 Enter 保留当前选项[+]，或者键入选项编号：4
[root@localhost jdk21]# java -version
openjdk version "21.0.3" 2024-04-16 LTS
OpenJDK Runtime Environment Corretto-21.0.3.9.1 (build 21.0.3+9-LTS)
OpenJDK 64-Bit Server VM Corretto-21.0.3.9.1 (build 21.0.3+9-LTS, mixed mode, sharing)
```