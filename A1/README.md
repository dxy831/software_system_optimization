# 上机作业 A1：初试环境和工具

学号：10234500007  
姓名：丁熙妍

## 系统信息

- 操作系统：Ubuntu 24.04 LTS，运行在 Windows 宿主机的 WSL2 中。
- CPU：当前 WSL2 报告 AMD Ryzen 9 8940HX with Radeon Graphics，12 个 Core、每 Core 2 个硬件线程，共 24 个逻辑 CPU。这是虚拟机可见的拓扑。
- 内存：当前 WSL2 可见内存约 9.7 GiB。

## 1. 环境搭建和工具安装

| 项目 | 当前实际结果 | 课程要求 |
| --- | --- | --- |
| Ubuntu / Kernel | Ubuntu 24.04 LTS / Linux 6.6 | Ubuntu 20.04 LTS / 5.4+ |
| GCC / Clang | 13.3 / 18.1 | 9.3+ / 10.0+ |
| Python / Java | 3.12 / OpenJDK 21.0 | 3.8+ / 11+ |
| Valgrind / perf | 3.22 / 6.6 | 3.17+ / 5.4+ |
| OpenCilk | 已安装；所附 OpenCilk 编译器为 Clang 19.1，`-fopencilk` 编译检查通过 | 1.0+ |

OpenCilk 一行的 19.1 是其编译器所报告的 Clang 版本，安装内容未单独给出 OpenCilk 发行版号；`-fopencilk` 已成功编译。相关命令的 `man` 页面已查阅。WSL2 的硬件接口限制在相应题目说明。

## 2. 常用工具命令操作练习

### (1) `uname -a`

**a.** 输出依次包含内核名称、主机名、内核发行号、构建信息、架构和操作系统名称。本次输出为 `Linux ddd 6.6.87.2-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Thu Jun 5 18:30:46 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux`。

**b.** Linux 内核发行号为 `6.6.87.2-microsoft-standard-WSL2`；指令集架构为 `x86_64`。

![uname 实际输出](images/01-uname-full.png)

### (2) `cat /etc/os-release`

**a.** 本机发行版名称 `NAME=Ubuntu`，完整名称 `PRETTY_NAME=Ubuntu 24.04.5 LTS`，版本标识 `VERSION_ID=24.04`，完整版本 `VERSION=24.04.5 LTS (Noble Numbat)`，代号 `VERSION_CODENAME=noble`。`ID=ubuntu` 是发行版标识，`ID_LIKE=debian` 表示其所属发行版家族；文件还给出支持、报错等网址。这些是 Ubuntu 发行版信息，不是内核发行号。

![/etc/os-release 实际输出](images/02-os-release.png)

### (3) `sysctl -a`

**a.** `sysctl` 读取或设置内核运行参数；`-a` 列出当前可见的全部参数及其值。这里只读取，未修改参数。

**b.** 大量 `sysctl` 键映射到 `/proc/sys` 中的文件，例如 `kernel.ostype` 对应 `/proc/sys/kernel/ostype`。点分隔键名对应目录层级。

**c.** 实际得到 `kernel.ostype = Linux`、`kernel.osrelease = 6.6.87.2-microsoft-standard-WSL2`，与 `uname -a` 中的内核名称和发行号一致。`/etc/os-release` 中的 Ubuntu 24.04.5 是用户空间发行版版本，不能拿它与内核发行号作字面相等比较。

**d.** 本机存在 `kernel.perf_event_max_sample_rate = 100000`，它限制 perf 采样事件允许的最大采样频率；`kernel.sched_rr_timeslice_ms = 100`，它规定 SCHED_RR 实时调度策略的轮转时间片，单位毫秒。两个参数均先在实际 `sysctl -a` 输出中确认。

![sysctl 输出节选](images/03-sysctl.png)

![内核参数实际值](images/03-kernel-keys.png)

![perf 参数实际值](images/03-kernel-perf.png)

![调度参数实际值](images/03-kernel-sched.png)

### (4) `lscpu`

**a.** 当前 WSL2 输出的型号是 AMD Ryzen 9 8940HX with Radeon Graphics；`Socket(s)=1`、`Core(s) per socket=12`、`Thread(s) per core=2`，因此可见 12 个 Core、24 个逻辑 CPU，存在每核两个硬件线程。`lscpu` 未显示基准、最大和最小频率，因此此处不列具体值。缓存行显示 L1d 384 KiB（12 个实例，约每核 32 KiB）、L1i 384 KiB（12 个实例，约每核 32 KiB）、L2 12 MiB（12 个实例，约每核 1 MiB）、L3 32 MiB（1 个实例）。前三项是实例容量的汇总。

**b.** `Byte Order: Little Endian`，本机为小端序。网络协议通常使用大端的网络字节序，这是大端序的应用场景。

**c.** `Address sizes: 48 bits physical, 48 bits virtual`：前者是当前环境报告的物理地址宽度，后者是虚拟地址宽度。64 位处理器表示寄存器和基本指针架构为 64 位，不代表硬件实现或虚拟环境暴露的物理、虚拟地址宽度都必须是 64 位；本次两个值实际都是 48 位。

![lscpu 型号、线程与地址宽度](images/04-lscpu-full.png)

![lscpu 缓存与 NUMA 字段](images/04-lscpu-cache.png)

### (5) `dmidecode`

**a.** `dmidecode` 读取 SMBIOS/DMI 固件表，列出硬件配置，而不是凭型号推断硬件。

**b.** 在直接暴露相应信息的机器上，内存条条目通常可给出插槽、容量、类型、速率、制造商、序列号等。本机普通用户执行提示不能读取 `/dev/mem`；以 root 再次执行得到 `No SMBIOS nor DMI entry point found`。当前 WSL2 未提供内存条 DMI 信息。

![普通用户执行 dmidecode](images/05-dmidecode.png)

![root 执行 dmidecode 的实际结果](images/05-dmidecode-root.png)


### (6) `numactl -H` 与 `numactl --show`

**a.** `numactl` 查看或设置进程的 NUMA 绑定和内存策略；`-H`（hardware）列出当前可见的 NUMA 硬件拓扑。

**b.** `numactl -H` 显示 `available: 1 nodes (0)`，节点 0 有逻辑 CPU 0–23；采样时节点内存约 9942 MB。

**c.** `node distances` 是节点间访问成本的相对值，通常本地节点值较小。本次只有节点 0，到自身的距离是 10，因而没有可比较的远端节点。多节点机器上的跨节点访存密集型数据库、内存密集型并行计算等可能受距离影响。

**d.** `numactl --show` 显示当前进程的策略和绑定：本次 `policy: default`、`preferred node: current`、`physcpubind: 0 ... 23`、`cpubind: 0`、`nodebind: 0`、`membind: 0`。其中 `physcpubind` 是可用 CPU 列表，其余绑定字段按工具当前输出理解。`-H` 描述系统可见节点及距离，`--show` 描述执行该命令的进程当前使用的策略和绑定。

![NUMA 节点与距离](images/06-numactl-h.png)

![当前 NUMA 策略](images/06-numactl-show.png)

### (7) `free -h`

**a.** `Mem` 行描述内存总量、已用、空闲、共享、缓存和可用量；截图样本约为总量 9.7 GiB、已用 639 MiB、空闲 8.7 GiB、`buff/cache` 600 MiB、`available` 9.1 GiB。`available` 估计无需大量交换即可供新程序使用的内存，不能只看 `free`。`Swap` 行描述交换空间，本次总量 8.0 GiB、已用 0 B、空闲 8.0 GiB。数值会随运行时变化。

**b.** `GiB = 2^30` 字节，`GB = 10^9` 字节；二者数值基准不同。

![free -h 实际输出](images/07-free.png)

### (8) `ps -aux`

**a.** 实际表头有 `USER`（用户）、`PID`（进程号）、`%CPU`（进程 CPU 百分比）、`%MEM`（进程驻留内存相对物理内存的比例）、`VSZ`（虚拟地址空间大小，KiB）、`RSS`（驻留物理内存，KiB）、`TTY`（关联终端，`?` 表示无终端）、`STAT`（状态及附加标记）、`START`（启动时间）、`TIME`（累计 CPU 时间）、`COMMAND`（启动命令）。例如本次样本中 PID 1 是 `/sbin/init`。`ps` 给的是执行时的进程快照。

![ps -aux 输出节选](images/08-ps.png)

### (9) `top` 与 `htop`

**a.** `top` 首行显示当前时间、运行时长、登录用户数和最近 1、5、15 分钟 load average；`Tasks` 显示进程总数及运行、休眠、停止、僵尸数。`%Cpu(s)` 中 `us` 是用户态、`sy` 是内核态、`ni` 是调整过 nice 值的用户态进程占用、`id` 是空闲、`wa` 是 I/O 等待，`hi/si` 是硬/软中断，`st` 是虚拟机被宿主机占用的时间。`MiB Mem`、`MiB Swap` 分别显示内存和交换空间的总量、空闲及使用情况。进程表中的 PID、USER、PR、NI、VIRT、RES、SHR、S、`%CPU`、`%MEM`、`TIME+`、COMMAND 分别对应进程标识、用户、优先级、nice 值、虚拟/驻留/共享内存、状态、CPU/内存占比、累计 CPU 时间和命令。

**b.** 两者都实时刷新。`htop` 直接显示各逻辑 CPU 的条形图，并提供明显的搜索、排序、树形视图和功能键操作；`top` 以文字汇总和进程表为主，也可交互调整显示。两者均已实际进入交互界面。交互程序会覆盖原有提示符，因此分别保存输入命令时和运行中的画面。

![top 命令输出](images/09-top-batch.png)

![在终端输入 top](images/09-top-entered.png)

![top 交互界面](images/09-top-interactive.png)

![在终端输入 htop](images/09-htop-entered.png)

![htop 交互界面](images/09-htop-interactive.png)

### (10)–(13) 性能统计命令

（10）`vmstat 1`

![vmstat 采样](images/10-vmstat.png)

（11）`mpstat -P ALL 1`

![mpstat 采样](images/11-mpstat.png)

（12）`pidstat 1`

![pidstat 采样](images/12-pidstat.png)

（13）`iostat -xz 1`

![iostat 采样](images/13-iostat.png)

**a.** 四条命令中的 `1` 均表示每隔 1 秒采样并刷新一次。未指定次数时持续输出，可用 Ctrl+C 结束；`vmstat`、`iostat` 的首批统计可能是自启动以来的平均值，随后才是相邻采样间隔的数据。

**b.** `vmstat` 统计可运行/阻塞进程、内存、交换、块 I/O、系统中断与上下文切换，以及 CPU 时间分布。`mpstat` 统计整体及每个逻辑 CPU 的用户态、系统态、I/O 等待和空闲等比例；本机样本显示 24 个逻辑 CPU。`pidstat` 统计进程或任务在采样间隔内的 CPU 使用，包括 `%usr`、`%system` 和运行所在 CPU 等字段。`iostat` 统计 CPU 总览和块设备的吞吐、请求率、等待时间、队列和设备利用率；本机可见 `sda`–`sdd` 等 WSL2 设备。

**c.** `ps -aux` 的 `%CPU` 是快照中按进程累计 CPU 时间和存活时长计算的占比；`pidstat` 的 `%CPU` 反映采样间隔内某进程的 CPU 使用；`top/htop` 按刷新周期更新进程 CPU 使用率，界面还提供整机或各核负载。因此同一进程在不同时间、统计窗口和多核显示模式下数值可能不同；`top` 顶部的整机 `%Cpu(s)` 与单个进程的 `%CPU` 观察对象不同。

### (14) `sar -n DEV 1`

**a.** `1` 表示每隔 1 秒输出一轮网络设备统计。

**b.** `-n DEV` 按网络接口给出收发包速率 `rxpck/s`、`txpck/s`，收发数据速率 `rxkB/s`、`txkB/s`，压缩包速率 `rxcmp/s`、`txcmp/s`，接收组播包速率 `rxmcst/s` 和接口利用率 `%ifutil`。本次可见 `lo`、`eth0`、`docker0`；所截图的采样轮次这些指标均为 0，表示该短时间窗口内没有记录到对应流量。

![sar 网络设备采样](images/14-sar.png)

### (15) `uptime`

**①** 截图样本为 `12:23:58 up 11 min, 1 user, load average: 0.16, 0.05, 0.01`，依次是当前时间、运行时长、登录用户数和三个平均负载值。

**②** 三个值依次是最近 1、5、15 分钟的平均负载。它反映等待运行或处于不可中断等待状态的任务数量趋势，不是 CPU 利用率百分比；判断是否拥塞还需结合本次可见的 24 个逻辑 CPU 和任务状态。

![uptime 实际输出](images/15-uptime.png)

### (16) `/proc/interrupts` 与 `/proc/softirqs`

**a.** `/proc/interrupts` 按 CPU 列出各 IRQ/中断类别的累计计数及可见设备或说明；`/proc/softirqs` 按 CPU 列出 TIMER、NET_RX、SCHED、RCU 等软中断类别的累计计数。二者是计数器快照，并非每秒发生率。

**b.** 保存的较晚一次快照中，带数字编号的设备 IRQ 里 `25: virtio0-virtqueues` 合计 4223 次，为这组设备 IRQ 中最高；内核列出的其他中断类别中，`CAL`（Function call interrupts）为 269027 次、`HVS`（Hyper-V stimer0）为 216119 次。软中断类别中 `RCU` 合计 122262 次，高于 `SCHED` 的 99640 次和 `TASKLET` 的 95092 次。内核 RCU、调度与 WSL2 虚拟设备活动可能影响这些计数；这是当前虚拟环境的累计快照。下方终端截图采集较早，因此计数较小。

![硬中断计数节选](images/16-interrupts.png)

![软中断计数节选](images/16-softirqs.png)

### (17) `lstopo --of svg > topo.svg`

生成的 [topo.svg](topo.svg) 已实际打开。图中有一个 Machine、一个 Package、一个 NUMA Node；该 Package 下显示共享的 32 MB L3、12 个 Core，每 Core 两个 PU，总计 24 个 PU。各 Core 对应 1024 KB L2、32 KB L1d、32 KB L1i；图上还列出 `eth0` 和若干虚拟块设备。该图描述 WSL2 当前可见的拓扑。

![生成 topo.svg 的命令](images/17-lstopo.png)

![lstopo 生成的拓扑图](topo.svg)

![在浏览器中打开 topo.svg](images/17-topo-open.png)

### (18) Git 命令练习

本题在 WSL2 中的独立本地仓库完成。

**① 用户名和邮箱。** 初始化仓库后执行 `git config user.name "丁熙妍"`、`git config user.email "10234500007@stu.ecnu.edu.cn"`，并用 `git config --get` 核对；只设置该练习仓库的本地配置。

![Git 用户配置](images/18-git-config.png)

**② 初始化和分支。** 执行 `git init` 后，`git branch --show-current` 实际输出 `master`。若要为今后的新仓库修改默认分支，可用 `git config --global init.defaultBranch main`；若要重命名当前分支，可用 `git branch -m main`。本次只是回答这两条命令，没有无关修改全局设置或强行重命名练习仓库。

![仓库分支复核](images/18-git-init.png)

**③ 提交。** 创建但未暂存 `README.md` 时直接执行 `git commit -m "docs: add practice readme"`，Git 拒绝提交并提示有未跟踪文件、没有可提交的改动。随后执行 `git add README.md` 和同一条 `git commit -m ...`，创建了真实提交。

![用未跟踪测试文件再次验证提交行为](images/18-git-commit-before.png)

![首次提交记录](images/18-git-commit-after.png)

**④ 图片忽略与取消跟踪。** 复制一张系统提供的无隐私 PNG 为 `practice.png`，执行 `git add practice.png`、`git commit -m "chore: add practice image"`。随后把 `practice.png` 写入 `.gitignore`，执行 `git rm --cached practice.png`、`git add .gitignore` 和 `git commit -m "chore: ignore practice image"`。最终 `git ls-files` 不再列出图片，`git check-ignore -v practice.png` 命中规则，而本地文件仍存在。`.gitignore` 不会自动取消已跟踪文件，`git rm --cached` 只从索引移除，不删除工作区文件。

![图片提交记录](images/18-git-image.png)

![忽略规则提交记录](images/18-git-ignore.png)

![最终跟踪文件与本地图片](images/18-git-final.png)

![最终工作区状态](images/18-git-clean.png)

**⑤ 语义化提交。** 阅读 [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/#summary) 后，我的理解是：提交标题以 `type: description`（可选 scope）说明变更性质，例如 `docs: add practice readme` 表示文档变更，`fix: correct matrix size` 表示修复，`feat:` 表示新功能；破坏兼容性的变更需要显式标记。这样历史更容易阅读，也便于自动生成变更记录。练习仓库中的三条提交实际使用了 `docs:`、`chore:`。

**⑥ 合并方式。** `git merge` 把目标分支的历史并入当前分支，存在分叉时通常生成 merge commit，保留原提交关系；`git rebase` 把当前分支提交依次重新应用到新基点，历史看起来较直，但重新创建的提交 SHA 会变化。已被其他人使用的共享提交若被随意 rebase，其他人的分支和引用会难以对齐，因此不应任意重写共享历史。

## 3. MIT 6.172 Homework 1: Getting Started

使用老师给出的 `c-primer` 和 `matrix-multiply` starter code。下文记录本机实际运行结果。

### Write-up 2

原始 `pointer.c` 执行 `make pointer` 时，编译器对修改只读目标及重新赋值 const 指针的语句报错。逐项回答源码注释：

1. `argv` 在 `main` 的参数中等效为 `char **`，指向各命令行参数字符串指针。
2. `char d = *pc` 取得字符串 `"6.172"` 的首字符，输出 `char d = 6`。
3. `pcp = argv` 合法，因为两者都是 `char **`。
4. `pcc2` 的类型是 `char const *`，与 `const char *` 等价：指向不可经该指针修改的字符，指针自身可重新赋值。
5. `*pcc = '7'` 非法，因为 `pcc` 指向 const char。
6. `pcc = *pcp` 合法：右侧为 `char *`，可赋给指向 const char 的可变指针。
7. `pcc = argv[0]` 同理合法。
8. `cp = *pcp` 非法，因为 `cp` 是 `char * const`，指针自身不可重新赋值。
9. `cp = *argv` 同理非法。
10. `*cp = '!'` 合法，因为 `cp` 指向的字符不是 const。
11. `cpc = *pcp` 非法，因为 `cpc` 是 `const char * const`，指针自身不可重新赋值。
12. `cpc = argv[0]` 同理非法。
13. `*cpc = '@'` 非法，因为 `cpc` 指向 const char。

按 Exercise 注释六条非法赋值后，重新编译成功；剩余未使用变量警告不影响编译。

![原始 pointer.c 编译错误前半](images/writeup2-pointer-error-head.png)

![原始 pointer.c 编译错误后半](images/writeup2-pointer-error-tail.png)

![注释非法语句后重新编译 pointer](images/writeup2-pointer-fixed.png)

![运行修复后的 pointer](images/writeup2-pointer-run.png)

### Write-up 3

在 `sizes.c` 中对原题所列类型同时输出类型本身及其指针的 `sizeof`。本机 `int` 为 4 字节、`__int128` 为 16 字节，截图列出全部类型及指针的实测值。数组 `x` 用 `sizeof(x)` 测得整体 20 字节，用 `sizeof(&x)` 测得指向整个数组的指针 8 字节；`student` 对象及其指针均为 8 字节。

![类型及指针大小：前半](images/writeup3-sizes-part1.png)

![类型及指针大小：后半](images/writeup3-sizes-part2.png)

### Write-up 4

原 `swap(int i, int j)` 只交换函数内的值拷贝，不能改变 `main` 的 `k`、`m`。将参数改为指针，并在 `main` 中传入变量地址。修改后的代码见截图。

![修改后的 swap.c 代码](images/writeup4-swap-code.png)

`./swap` 输出 `k = 2, m = 1`。用 Python 3 兼容副本运行老师提供的检查，最终输出 `LGTM`。

![修改后的交换结果](images/writeup4-swap.png)

![verifier 最终结果](images/writeup4-verifier.png)

### Write-up 5

将 `matrix-multiply/Makefile` 的普通构建优化级别由 `-O1` 改为 `-O3`。执行 `make clean; make` 时，先删除旧目标文件和二进制文件，再以 `clang -O3 -DNDEBUG ...` 编译 `testbed.c`、`matrix_multiply.c` 并链接为 `matrix_multiply`。实际命令和输出见截图。

![O3 构建实际输出](images/writeup5-build.png)

### Write-up 6

执行 `make clean; make ASAN=1; ./matrix_multiply` 后，LeakSanitizer 报告 `make_matrix` 中的内存泄漏：直接泄漏 48 字节，间接泄漏 288 字节，合计 336 字节、18 次分配。具体调用位置和完整报告见截图。

![ASan/LeakSanitizer 实际报告](images/writeup6-asan.png)

### Write-up 7

修复后执行 `./matrix_multiply -p`，截图给出两个输入矩阵、4×4 结果矩阵和完整程序输出；逐项计算结果一致。

![修复后的矩阵乘法输出](images/writeup7-result.png)

### Write-up 8

在使用完矩阵并输出结果后，调用现有的 `free_matrix()` 释放 A、B、C。最终执行 `valgrind --leak-check=full ./matrix_multiply -p`，报告显示 `in use at exit: 0 bytes in 0 blocks`、`All heap blocks were freed`、`ERROR SUMMARY: 0 errors`。最终运行的结果见截图。

![最终 Valgrind 结果](images/writeup8-valgrind-final.png)
