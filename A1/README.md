# A1：初试环境和工具

学号：10234500007　姓名：丁熙妍

## 1. 环境搭建和工具安装

操作系统：Ubuntu 24.04 LTS，运行于 Windows 宿主机的 WSL2。  
CPU：AMD Ryzen 9 8940HX with Radeon Graphics。  
内存：WSL2 当前可见约 9.7 GiB。

| 工具 | 本机版本或检查结果 |
| --- | --- |
| Linux 内核 | 6.6 |
| GCC / Clang | 13.3 / 18.1 |
| Python / OpenJDK | 3.12 / 21.0 |
| Valgrind / perf | 3.22 / 6.6 |
| OpenCilk | 所附编译器报告 Clang 19.1；`-fopencilk` 编译检查通过，OpenCilk 发行版号未单独确认 |

## 2. 常用工具命令操作练习

### (1) `uname -a`

**a.** 输出包含内核名称、主机名、内核发行号、构建信息、机器架构、处理器类型、硬件平台和操作系统名称。其中 `Linux` 是内核名称，`ddd` 是主机名；`SMP` 和 `PREEMPT_DYNAMIC` 属于内核构建信息。这条命令主要说明正在运行的内核和平台，Ubuntu 的发行版信息需要另看 `/etc/os-release`。

**b.** 本次内核发行号为 `6.6.87.2-microsoft-standard-WSL2`，指令集架构为 `x86_64`。

![uname 输出](images/01-uname-full.png)

### (2) `cat /etc/os-release`

**a.** 文件记录 Ubuntu 的发行版信息。`NAME` 和 `PRETTY_NAME` 给出名称，`VERSION_ID` 和 `VERSION` 给出版本，`VERSION_CODENAME` 给出发行代号；本次分别可以读到 Ubuntu、24.04.5 LTS 和 `noble`。`ID=ubuntu` 用于标识发行版，`ID_LIKE=debian` 表示其所属发行版家族，其余网址用于获取帮助或报告问题。

这里的 Ubuntu 版本与上一题的 Linux 内核版本不是同一层信息，因此两个版本号不同并不矛盾。

![/etc/os-release 输出](images/02-os-release.png)

### (3) `sysctl -a`

**a.** `sysctl` 用于读取或设置内核运行参数，`-a` 列出当前可见的参数及其值。

![sysctl 输出节选](images/03-sysctl.png)

**b.** 参数对应 `/proc/sys` 下的文件。例如 `kernel.ostype` 对应 `/proc/sys/kernel/ostype`，键名中的点对应目录分隔。这两种方式是在查看同一个内核参数，而不是两套独立配置。

**c.** `kernel.ostype=Linux`、`kernel.osrelease=6.6.87.2-microsoft-standard-WSL2`，与 `uname -a` 中的内核名称和发行号一致。`/etc/os-release` 记录的是 Ubuntu 发行版信息，不能直接把其中的 24.04.5 与内核发行号比较。

![内核名称和发行号](images/03-kernel-keys.png)

**d.** 选择以下两个参数：

`kernel.perf_event_max_sample_rate=100000` 表示 perf 采样事件允许的最大采样频率。这是采样频率的上限，不表示当前已经按该频率采样。

`kernel.sched_rr_timeslice_ms=100` 表示 SCHED_RR 实时调度策略的轮转时间片为 100 毫秒。它针对该实时调度策略，不是所有普通进程统一使用的时间片。

![perf 参数](images/03-kernel-perf.png)

![调度参数](images/03-kernel-sched.png)

### (4) `lscpu`

**a.** 型号为 AMD Ryzen 9 8940HX with Radeon Graphics。WSL2 报告 1 个插槽、每插槽 12 个 Core、每 Core 2 个硬件线程，共 24 个逻辑 CPU。这里能确认的是 WSL2 可见的拓扑，不能直接把它当成宿主机的完整物理拓扑。基准、最大和最小频率均未在本次输出中显示。

缓存显示为 L1d 384 KiB、L1i 384 KiB、L2 12 MiB，三者各有 12 个实例；对应每个实例分别为 32 KiB、32 KiB、1 MiB。L3 为 32 MiB，共 1 个实例。因此，不能把 L1d 的 384 KiB 误当成单核 L1d 容量。

![CPU 型号、线程与地址宽度](images/04-lscpu-full.png)

![CPU 缓存](images/04-lscpu-cache.png)

**b.** 本机为小端序。网络字节序采用大端表示，是大端序的一个应用场景。

**c.** `physical` 表示物理地址宽度，`virtual` 表示虚拟地址宽度，本次均为 48 位。64 位架构并不要求物理地址和虚拟地址都用满 64 位，实际支持的地址位数取决于处理器实现和虚拟环境。指针占多少字节与其中有多少有效地址位，是两个不同的问题。

### (5) `dmidecode`

**a.** `dmidecode` 读取并解码 SMBIOS/DMI 固件表，用于查看硬件配置。

**b.** 内存条条目通常能给出插槽、容量、类型、速率、制造商、序列号等信息。

本次普通用户执行时提示无法读取 `/dev/mem`，这时还不能排除权限不足。以 root 再次执行后，提示变为 `No SMBIOS nor DMI entry point found`。因此，本次 WSL2 没有提供可用的 SMBIOS/DMI 入口，无法据此列出宿主机内存条的具体信息。

![普通用户执行 dmidecode](images/05-dmidecode.png)

![root 执行 dmidecode](images/05-dmidecode-root.png)

### (6) `numactl -H` 与 `numactl --show`

**a.** `numactl` 用于查看或设置进程的 NUMA 策略和绑定。`-H` 是 `--hardware`，显示系统可见的 NUMA 节点、CPU、内存和节点距离。

**b.** 本次只有 1 个节点，即节点 0；其中包含逻辑 CPU 0–23。

**c.** `node distances` 表示节点间访问成本的相对值。本次节点 0 到自身的距离为 10，没有远端节点可供比较。在多节点机器上，数据库和内存访问密集的并行程序可能因跨节点访存而受到影响。当前只有一个可见节点，不能据此比较本地和远端访存的性能差别。

![NUMA 节点与距离](images/06-numactl-h.png)

**d.** `numactl --show` 查看执行该命令的进程当前使用的 NUMA 策略和绑定。

本次 `policy: default` 表示使用默认内存策略，`preferred node: current` 显示当前节点。`physcpubind: 0 ... 23` 给出进程可使用的逻辑 CPU；`cpubind`、`nodebind` 显示的 CPU 节点均为 0，`membind` 显示的内存节点范围也只有 0。节点字段为 0 不表示只允许使用编号为 0 的逻辑 CPU。

两条命令的区别在于观察对象：`-H` 回答系统有哪些节点和资源，`--show` 回答当前进程可以使用哪些资源、采用什么策略。

![当前 NUMA 策略](images/06-numactl-show.png)

### (7) `free -h`

**a.** `Mem` 行描述内存：`total` 是总量，`used` 是已用量，`free` 是未使用量，`shared` 是共享内存占用，`buff/cache` 是缓冲和缓存，`available` 是估计可供新程序使用的内存。本次可见总量约 9.7 GiB，`available` 约 9.1 GiB。

判断还能使用多少内存时，不能只看 `free`，因为一部分缓存可以回收，`available` 更适合回答这个问题。`Swap` 行描述交换空间，本次总量 8.0 GiB，已用 0 B，表示采样时尚未使用交换空间。

![free -h 输出](images/07-free.png)

**b.** `GiB = 2^30` 字节，`GB = 10^9` 字节。同一容量用这两种单位表示时，数字不同，并不代表容量发生了变化。

### (8) `ps -aux`

**a.** `USER` 是进程用户，`PID` 是进程号；`%CPU` 表示 CPU 使用比例，`%MEM` 表示驻留内存占系统物理内存的比例。`VSZ` 是虚拟内存大小，`RSS` 是驻留物理内存大小，单位均为 KiB。

`TTY` 是关联终端，`?` 表示无关联终端；`STAT` 是进程状态及附加标记，`START` 是启动时间，`TIME` 是累计使用的 CPU 时间，`COMMAND` 是启动命令。`TIME` 不等于进程已经存活的时间，因为进程等待或休眠时并不会持续消耗 CPU。

`ps` 给出执行命令时的进程快照，不会像 `top` 一样持续刷新。

![ps -aux 输出节选](images/08-ps.png)

### (9) `top` 与 `htop`

**a.** `top` 顶部首行给出当前时间、系统运行时长、登录用户数和最近 1、5、15 分钟的平均负载。`Tasks` 给出进程总数及运行、休眠、停止、僵尸进程数。

`%Cpu(s)` 中，`us` 是普通用户态时间，`sy` 是内核态时间，`ni` 是调整过 nice 值的用户态任务占用，`id` 是空闲时间，`wa` 是 I/O 等待时间，`hi`、`si` 分别是硬中断和软中断处理时间，`st` 是虚拟 CPU 未获得宿主机 CPU 执行机会的时间比例。`MiB Mem` 和 `MiB Swap` 分别显示内存及交换空间的总量和使用情况，内存部分还包括缓冲、缓存及可用量。

进程表中，`PID`、`USER` 是进程号和用户；`PR`、`NI` 是调度优先级和 nice 值；`VIRT`、`RES`、`SHR` 是虚拟、驻留和共享内存；`S` 是状态；`%CPU`、`%MEM` 是 CPU 和内存占比；`TIME+` 是累计 CPU 时间；`COMMAND` 是命令。顶部 CPU 汇总反映整体使用情况，进程行反映单个进程，不能把两者直接当成同一个百分比。

![top 交互界面](images/09-top-interactive.png)

**b.** 两者都能持续查看进程和资源使用情况。`htop` 用条形图展示各逻辑 CPU，搜索、排序、树形视图和进程选择更直观；`top` 以系统摘要和进程表为主，也支持交互操作。前者更方便同时观察多核和多个进程，后者便于查看紧凑的系统概况。

![htop 交互界面](images/09-htop-interactive.png)

### (10)–(13) 性能统计命令

(10) `vmstat 1`

![vmstat 采样](images/10-vmstat.png)

(11) `mpstat -P ALL 1`

![mpstat 采样](images/11-mpstat.png)

(12) `pidstat 1`

![pidstat 采样](images/12-pidstat.png)

(13) `iostat -xz 1`

![iostat 采样](images/13-iostat.png)

**a.** 参数 `1` 表示统计更新的间隔为 1 秒，不是只输出一次。`vmstat` 首份报告中的 CPU 等统计、`iostat` 的首份报告反映自启动以来的情况，后续报告反映相邻采样间隔；`vmstat` 的进程和内存数据则始终是即时状态。

**b.** `vmstat` 从整体上观察进程、内存、交换、块 I/O、系统活动和 CPU；`mpstat -P ALL` 将 CPU 使用情况细分到各逻辑 CPU；`pidstat` 按进程或任务统计 CPU 使用；`iostat -xz` 查看块设备吞吐、等待时间、队列和利用率等扩展统计。

它们关注的层次不同：查看系统整体状态用 `vmstat`，区分哪个逻辑 CPU 忙用 `mpstat`，查哪个进程占用 CPU 用 `pidstat`，分析块设备 I/O 用 `iostat`。

**c.** `ps -aux` 的 `%CPU` 按进程累计 CPU 时间与存活时长计算；`pidstat 1` 反映采样间隔内的 CPU 使用，`top/htop` 则按刷新周期更新。因此，一个进程刚开始繁忙时，短时采样可能已经显示较高占用，而 `ps` 的长期平均仍较低。比较结果时，需要对应同一进程、相近时间窗口，并注意整机、单核和单进程百分比的区别。

### (14) `sar -n DEV 1`

**a.** `1` 表示每隔 1 秒输出一轮网络设备统计。

**b.** `-n DEV` 按网络接口统计收发情况：`rxpck/s`、`txpck/s` 是收发包速率，`rxkB/s`、`txkB/s` 是收发数据速率，`rxcmp/s`、`txcmp/s` 是压缩包速率，`rxmcst/s` 是接收组播包速率，`%ifutil` 是接口利用率。

本次显示 `lo`、`eth0`、`docker0`，截图中的采样轮次各项均为 0。这只能说明该短时间窗口内没有记录到对应流量，不能说明这些接口始终没有通信。

![sar 网络设备采样](images/14-sar.png)

### (15) `uptime`

**①** 输出依次为当前时间、系统运行时长、登录用户数和三个平均负载值。本次采样时系统已运行 11 分钟，有 1 个登录用户，三个负载值为 0.16、0.05、0.01。

![uptime 输出](images/15-uptime.png)

**②** 三个值分别对应最近 1、5、15 分钟，反映可运行及不可中断等待任务的平均数量。它以任务数计量，不是 CPU 利用率百分比。即使 CPU 没有满载，处于不可中断等待的任务也会影响负载，因此判断系统是否繁忙还应结合 CPU 和 I/O 状态。

### (16) `/proc/interrupts` 与 `/proc/softirqs`

**a.** `/proc/interrupts` 按 CPU 列出 IRQ 及其他中断类别的累计计数和相关说明；`/proc/softirqs` 按 CPU 列出 RCU、SCHED、TIMER、NET_RX 等软中断类别的累计计数。这些是累计次数，不是每秒发生率。

**b.** 对保存的 12:49:55 完整快照按 CPU 求和，带数字编号的设备 IRQ 中，`25: virtio0-virtqueues` 最多，为 4223 次。把其他中断类别也计入时，`CAL` 为 269027 次，高于 `HVS` 的 216119 次。软中断中 `RCU` 最多，为 122262 次，其次是 `SCHED` 的 99640 次和 `TASKLET` 的 95092 次。

`CAL` 对应核间函数调用，`HVS` 对应 Hyper-V 计时器，`RCU` 与内核同步维护有关。这些计数可能受到内核调度、同步和虚拟设备活动影响，不能只根据一次累计快照确定唯一原因。这里统计的是 WSL2 可见的中断，也不代表宿主机全部物理设备的中断情况。下方截图是较早时刻的输出节选，与上述完整快照的计数不同。

![硬中断计数节选](images/16-interrupts.png)

![软中断计数节选](images/16-softirqs.png)

### (17) `lstopo --of svg > topo.svg`

[生成的拓扑图](topo.svg) 中有一个 Machine、一个 Package 和一个 NUMA Node。Package 下有 12 个 Core，每个 Core 对应两个 PU，共 24 个 PU；PU 对应系统可调度的逻辑 CPU，而不是额外的物理核。

图中每个 Core 对应 1024 KB L2、32 KB L1d 和 32 KB L1i，12 个 Core 上方是共享的 32 MB L3，还列出了 `eth0` 和虚拟块设备。它与 `lscpu` 展示的是同一套 WSL2 可见拓扑，图形更容易看清资源的从属和共享关系。

![lstopo 拓扑图](topo.svg)

### (18) Git 命令练习

**① 用户名和邮箱。** 在初始化后的本地练习仓库中设置：

```bash
git config user.name "丁熙妍"
git config user.email "10234500007@stu.ecnu.edu.cn"
```

这两条命令只设置当前仓库的提交身份，不会修改其他仓库的配置。

![Git 用户配置](images/18-git-config.png)

**② 初始化和分支。** 执行 `git init` 后，`git branch --show-current` 显示 `master`。设置以后新建仓库的默认分支名称，可以使用：

```bash
git config --global init.defaultBranch main
```

重命名当前仓库分支，可以使用：

```bash
git branch -m main
```

前者影响以后新建的仓库，后者修改当前分支的名称，作用范围不同。

![仓库分支复核](images/18-git-init.png)

**③ 提交。** 存在未跟踪文件、但尚未暂存任何改动时，`git commit` 提示没有可提交的改动。将 `README.md` 加入暂存区后再提交：

```bash
git add README.md
git commit -m "docs: add practice readme"
```

关键不是文件是否已经保存在目录里，而是要提交的改动是否进入暂存区。

![未暂存文件时的提交结果](images/18-git-commit-before.png)

![首次提交记录](images/18-git-commit-after.png)

**④ 图片忽略与取消跟踪。** 先将测试图片加入版本控制并提交：

```bash
git add practice.png
git commit -m "chore: add practice image"
```

![图片提交记录](images/18-git-image.png)

随后在 `.gitignore` 中写入 `practice.png`，再执行：

```bash
git rm --cached practice.png
git add .gitignore
git commit -m "chore: ignore practice image"
```

![忽略规则提交记录](images/18-git-ignore.png)

最终 `git ls-files` 中不再有该图片，`git check-ignore -v practice.png` 命中忽略规则，本地图片仍然存在。`.gitignore` 不会自动取消已经跟踪的文件；`git rm --cached` 移除的是索引中的条目，不是工作区文件，也不会删除历史提交中的图片。

![最终跟踪文件与本地图片](images/18-git-final.png)

**⑤ 语义化提交。** [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/#summary) 用 `type: description` 的形式说明变更性质，也可以添加 scope。本次的 `docs: add practice readme` 表示文档修改，图片追踪相关提交使用 `chore:`；`fix:` 用于修复，`feat:` 用于新增功能，破坏兼容性的变更需要明确标记。

相比只写“update”，这样的标题能直接看出提交在做什么。不过类型只是分类，后面的描述仍要写清实际改动。

**⑥ 合并方式。** `git merge` 合并两个分支的历史，存在分叉时通常生成 merge commit，保留原有提交关系；`git rebase` 将一组提交重新应用到新的基点，通常得到更线性的历史，但重新创建的提交 SHA 会变化。

因此，两者不只是历史图形不同。对于已经被他人使用的共享提交，rebase 会改变他们所依赖的历史，需要额外协调；不能仅为了让历史看起来更直，就随意重写共享分支。

## 3. MIT 6.172 Homework 1: Getting Started

### Write-up 2

按 `pointer.c` 中注释的顺序回答：

1. `argv` 在函数参数中等效为 `char **`，指向各命令行参数字符串的指针。
2. `char d = *pc` 取出字符串 `"6.172"` 的首字符，因此输出 `char d = 6`。
3. `pcp = argv` 合法，因为两者类型都是 `char **`。
4. `pcc2` 是 `char const *`，与 `const char *` 等价：不能通过它修改字符，但可以改变指针的指向。
5. `*pcc = '7'` 非法，因为它试图通过指向 const char 的指针修改字符。
6. `pcc = *pcp` 合法，右侧是 `char *`，可以赋给 `const char *`；这里增加了访问限制，没有去掉 const。
7. `pcc = argv[0]` 合法，原因同上。
8. `cp = *pcp` 非法，因为 `cp` 是 `char * const`，指针本身不能重新赋值。
9. `cp = *argv` 非法，同样是在修改 const 指针本身。
10. `*cp = '!'` 合法，`cp` 的指向固定，但它指向的字符可以修改。
11. `cpc = *pcp` 非法，因为 `cpc` 的指向不能改变。
12. `cpc = argv[0]` 非法，原因同上。
13. `*cpc = '@'` 非法，因为也不能通过 `cpc` 修改所指字符。

判断这些语句时，要区分赋值目标是“指针本身”还是“指针指向的字符”。注释六条非法赋值后，`make pointer` 编译成功。

![注释非法语句后重新编译 pointer](images/writeup2-pointer-fixed.png)

### Write-up 3

`sizes.c` 同时输出题目所列类型和相应指针的 `sizeof`。本次各类型本身的大小不同，但所测对象指针均为 8 字节。指针大小反映的是存放地址所需的空间，不是被指向对象的大小。

例如，`x` 是包含 5 个 `int` 的数组，`sizeof(x)` 为 20 字节；`sizeof(&x)` 测量指向整个数组的指针，为 8 字节。`student` 对象和它的指针本次都是 8 字节，但两次测量的对象仍然不同。

![类型及指针大小：前半](images/writeup3-sizes-part1.png)

![类型及指针大小：后半](images/writeup3-sizes-part2.png)

### Write-up 4

原来的 `swap(int i, int j)` 只交换形参的值，不会改变 `main` 中的 `k` 和 `m`。修改后传入 `&k`、`&m`，函数通过 `*i`、`*j` 访问并修改原变量。这里仍然是值传递，只是传递的值变成了地址。

```c
// Copyright (c) 2012 MIT License by 6.172 Staff

#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>

void swap(int *i, int *j) {
  int temp = *i;
  *i = *j;
  *j = temp;
}

int main() {
  int k = 1;
  int m = 2;
  swap(&k, &m);
  // What does this print?
  printf("k = %d, m = %d\n", k, m);

  return 0;
}
```

编译运行后输出 `k = 2, m = 1`，两个变量的值已交换。

![修改后的交换结果](images/writeup4-swap.png)

使用老师检查脚本的 Python 3 兼容副本验证，最终输出 `LGTM`，其中包括 `sizes.c` 和 `swap.c` 的结果检查。

![verifier 最终结果](images/writeup4-verifier.png)

### Write-up 5

将 Makefile 中普通构建的优化选项改为：

```makefile
CFLAGS_RELEASE := -O3 -DNDEBUG
```

执行 `make clean; make` 后，先删除旧目标文件和可执行文件，再以 `clang -O3 -DNDEBUG ...` 编译两个源文件并链接生成 `matrix_multiply`。清理旧文件是为了让修改后的编译选项作用于重新生成的程序；输出中的 `-O3` 说明该构建配置已经生效。这里确认的是编译选项生效，不能仅凭 `-O3` 判断实际加速幅度。

![O3 构建输出](images/writeup5-build.png)

### Write-up 6

修复矩阵维度、尚未补充释放操作时，执行 `make clean; make ASAN=1; ./matrix_multiply`。LeakSanitizer 报告 `make_matrix` 中的分配发生泄漏：直接泄漏 48 字节，间接泄漏 288 字节，合计 336 字节、18 次分配。

直接泄漏对应矩阵结构体，间接泄漏对应其关联的行指针数组和各行数据。程序能够运行结束，并不表示分配的内存已经正确释放。这里报告的是泄漏；矩阵元素未初始化的问题是在后续 Valgrind 检查中发现的。

![ASan/LeakSanitizer 报告](images/writeup6-asan.png)

### Write-up 7

矩阵部分有两个影响正确性的问题。首先，原来的 A 为 4×5，B 为 4×4，A 的列数与 B 的行数不一致；修正为 4×4 后，程序不再因维度不匹配而崩溃。其次，乘法循环使用 `C->values[i][j] += ...`，C 的元素必须先为 0，否则未初始化的值会参与累加。

将矩阵行分配改为 `calloc(cols, sizeof(int))` 后，各元素从 0 开始，保留原来的乘法循环即可得到正确结果。这个修改解决的是初始化问题，不是改变矩阵乘法算法。

运行 `./matrix_multiply -p` 后，输出与矩阵乘法定义一致。例如左上角元素为 `3×1 + 7×5 + 8×0 + 1×9 = 47`，与程序输出相同；其余元素也与逐项计算结果一致。

![修复后的矩阵乘法输出](images/writeup7-result.png)

### Write-up 8

一个矩阵的结构体、行指针数组和各行数据分别分配，因此只释放最外层结构体还不够。已有的 `free_matrix()` 会依次释放各行数据、行指针数组和结构体，在使用完 A、B、C 后分别调用即可。

再次执行 `valgrind --leak-check=full ./matrix_multiply -p`，退出时剩余内存为 `0 bytes in 0 blocks`，并显示 `All heap blocks were freed` 和 `ERROR SUMMARY: 0 errors`。说明本次执行中没有检测到内存错误和泄漏。

![最终 Valgrind 结果](images/writeup8-valgrind-final.png)

