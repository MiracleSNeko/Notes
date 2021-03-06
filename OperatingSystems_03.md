# 操作系统

> 王道考研操作系统教程：BV1YE411D7nH

## 3. 内存管理

### 3.1 内存的基础概念

- 内存是用于存放数据的硬件，程序执行前需要先放入内存才能被 CPU 处理

- 编译生成的指令中，一般使用的是逻辑地址（相对地址），实际放入内存时再通过偏移访问物理地址（绝对地址）

- 程序从源代码到放入内存运行大概有以下步骤

    - 编译：将用户源代码编译成若干个目标模块（每个模块有自己的逻辑地址）

    - 链接：将目标模块链接成装入模块（有一个完整的逻辑地址空间）

        - 静态链接：在程序运行之前，先将各个目标模块和他们所需的库函数连接成一个完整的可执行文件，之后不再拆开

        - 装入时动态链接：将各目标模块装入内存时，边装入边链接的链接方式

        - 运行时动态链接：在程序执行中需要该目标模块时，才对其进行链接

    - 装入：将装入模块放在内存中（此时使用物理地址）

- 逻辑地址到物理地址的转换发生在装入阶段

    - 绝对装入：在编译时，如果知道程序将放到内存中的哪个位置，编译程序将产生绝对地址的目标代码。装入程序按照装入模块中的地址，将程序和数据装入内存。只适用于单道程序环境

    - 静态重定位（可重定位装入）：编译链接后的装入模块地址都是从 0 开始，装入程序负责对地址重定位，将逻辑地址转换为物理地址。在作业装入内存时必须分配其要求的全部内存空间，运行期不能再移动或申请内存。多用于多道批处理系统

    - 动态重定位（动态运行时装入）：编译链接后的装入模块地址从 0 开始，并且装入模块不负责将逻辑地址转换为物理地址，而是把转换推迟到程序执行时，由重定位寄存器完成转换（重定位寄存器存放装入模块的起始位置，CPU 将目标逻辑地址和重定位寄存器相加）。采用动态重定位允许程序在内存中发生移动和动态申请内存。多用于现代操作系统

### 3.2 内存管理的概念

- 操作系统负责内存空间的分配和回收

- 操作系统需要提供内存空间扩充方法

    - 覆盖技术

        - 将程序分为多个段，常用的段常驻内存，不常用的段在需要时调入内存

        - 内存中划分为一个“固定区”和多个“覆盖区”，常驻段放入固定区中，除非运行结束否则调入后就不再调出；不常用段放在覆盖区中，需要时调入，不需要时调出

        - 必须由程序员声明覆盖结构，操作系统自动完成覆盖。缺点：对用户不透明，增加编程负担

    - 交换技术

        - 在内存空间紧张时，操作系统在内存与磁盘之间动态调度，将某些进程暂时换出外存，外存中具备运行条件的进程换入内存

        - 引入挂起原语（suspend）后，将暂时换出外存等待的状态称为挂起态

        - 具有兑换功能的操作系统中，通常将磁盘空间分为文件区和对换区两部分，文件区主要用于存放文件，追求储存空间的利用率；对换区放置被换出的进程数据，主要追求换入换出速度

        - 交换发生在许多进程运行且内存吃紧时，而系统负荷降低就降低，如经常发生缺页时换出，缺页率明显下降后换入

- 操作系统需要提供地址转换功能，用于从逻辑地址转换到物理地址

    - 实现方式即上节中的三种装入

- 操作系统需要提供内存保护功能保证各个进程只能访问自己的内存空间

    - 设置上下限寄存器用来检查是否越界

    - 设置基址寄存器和界地址（限长）寄存器

### 3.3 内存空间的分配回收

#### 3.3.1 连续分配

- 连续分配指为用户进程分配的必须是一个连续的内存空间

- 单一连续分配

    - 将内存分为系统区和用户区，系统区位于低地址部分，用于存放操作系统相关数据，用户区存放用户进程相关数据

    - 只能有一道用户程序，用户程序独占整个用户区空间

    - 优点：实现简单，无外部碎片；可以采用覆盖技术扩充内存；不一定需要采取内存保护

    - 缺点：只能用于单用户、单任务的操作系统，有内部碎片（分配给某进程的内存区域有些部分没用上），储存器利用率极低

- 固定分区方式

    - 将整个用户空间划分为若干个固定大小的分区，在每个分区中只装入一道作业

    - 可分为分区大小相等和分区大小不等两种

    - 操作系统需要建立一个分区说明表，记录每个分区的分区号、大小、起始地址和分配状态等

    - 优点：实现简单，无外部碎片

    - 缺点：用户程序太大时可能需要覆盖，会产生内部碎片，内存利用率低

- 动态分区分配（可变分区分配）

    - 在进程装入内存时，根据进程的大小动态建立分区，并使得分区大小正好适合进程需要

    - 内存使用情况记录

        - 空闲分区表

            - 每个表项对应一个空闲分区

        - 空闲分区链

            - 每个分区起始和末尾设置指针

            - 起始部分可以记录额外信息
    
    - 动态内存分配算法

        - 首次适应

            - 每次从低地址开始查找，找到第一个能满足大小的空闲分区

        - 最佳适应

            - 优先使用更小的空闲区，保证当“大进程”到来时能有连续的大片空间

            - 缺点：会导致留下更多更小的外部碎片

        - 最坏适应（最大适应）

            - 优先利用最大的连续空闲区域，这样分配后剩余的空闲去就不会太小

            - 空闲分区按照容量递减次序链接，每次分配时找到大小能满足要求的第一个空闲分区

            - 缺点：大空闲分区会被快速用完，如果之后有“大进程”到达，会导致饥饿

        - 邻近适应

            - 空闲分区议地址递增的顺序排列（可排成循环链表），每次分配内存时从上次查找结束的位置开始，找到大小能满足要求的第一个空闲分区

            - 优点：会更有可能用到第地址部分的小分区，也会更有可能保留高地址部分的大分区

            - 缺点：低地址和高地址部分的空闲分区都有相同概率被使用，会导致高地址部分的大分区更可能被划分为小分区

    - 外部碎片可以通过拼凑技术，通过移动来合并外部碎片

#### 3.3.2 非连续分配

- 基本分页储存管理

    - 基本分页储存管理的思想：把内存分为一个个相等的小分区，再按分区大小把进程拆分成一个个小部分

    - 将内存空间分为一个个大小相等的分区，每个分区称为一个页框（页帧、内存块）

    - 将用户进程的地址空间分为与页框大小相等的一个个区域，称为页或页面。每个页面有一个编号，称为页号（从 0 开始）

    - 操作系统议页框为单位分配内存空间，进程的每个页面放入一个页框中

    - 地址转换：
      
        - 算出逻辑地址对应的页号，页号 = 逻辑地址 / 页面长度 （取整除法）

        - 找到该页号对应页面在内存中的起始地址，页内偏移量 = 逻辑地址 % 页面长度

        - 计算逻辑地址在页面内的偏移量，则物理地址 = 页面起始地址 + 页内偏移量

    - 操作系统需要用某种数据结构记录进程各个页面的起始位置

    - 为方便计算页号和页内偏移量，操作系统一般会将页面大小设置为 2 的整数幂。以 16 bit 表示逻辑地址，页面大小为 $2^{12}$ = 4096 Bit 举例

        - 零号页面逻辑空间地址是 0 到 4095 ，即

            0000,0000,0000,0000 - 0000,1111,1111,1111

            其他页面可以类推，更改高四位即可

        - 逻辑地址 2 的二进制是 0000,0000,0000,0010 ，若 0 号页在内存中起始地址为 X ，则逻辑地址 2 对应物理地址 X + 0000,0000,0010 。逻辑地址 4097 的二进制是 0001,0000,0000,0001 ，若 1 号页在内存中的起始位置为 Y ，则逻辑地址 4097 对应物理地址 Y + 0000,0000,0001

        - 二进制表示下，可以用掩码直接计算页号和偏移量，即分页储存管理的逻辑地址结构由高位的页号和低位的页内偏移量组成

    - 页表

        - 为了让进程知道每个页面在内存中存放的位置，操作系统为每个进程建立一张页表

        - 页表记录了页号和块号的对应关系。M 号内存块的起始地址就是 M * 块大小

        - 每个页表项的长度相同，页号是“隐含”的（通过页表起始地址和页表项长度计算）

- 基本地址变换机构

    - 基本地址变换机构用于实现从逻辑地址到物理地址的硬件转换

    - 系统中设置页表寄存器 PTR ，存放页表在内存中的起始地址 F 和页表长度 M（页表长度是页表项的个数）

    - 进程未执行时，页表的起始地址和页表长度存放在 PCB 中，当进程被调度时移入 PTR

    - 进程访问内存过程：

        - 将 F 和 M 装载到 PTR

        - 从逻辑地址读取页号 P 和页内偏移量 W

        - 检查 P 是否不小于 M，若是，发生越界中断；若否，根据 F、M、P 计算出页表始址 b 。注意：页号从 0 开始，而页表长度至少为 1 ，因此 P=M 也会越界

        - b:W 即目标物理地址

- 具有快表的地址变换机构

    - 局部性原理   

        - 时间局部性：如果执行了程序中的某条指令，那么不久之后这条指令可能被再次执行；如果某个数据被访问过，那么不久之后该数据很可能被再次访问（因为程序中存在大量循环）

        - 空间局部性：一旦程序访问了某个储存单元，那么不久之后其附近的储存单元也有可能被访问（因为很多数据在内存中连续存放）

    - 快表又称联想寄存器 TLB ，是一种访问速度比内存快很多的高速缓冲储存器，用来存放过当前访问的若干页表项，以加速地址变换的过程。与之相对，称页表为慢表

    - 在越界检查后，如果 P 命中快表中的页号-内存块号键值对，则直接返回页号，否则计算并查找页表（慢表）

    - 若快表已满，需要按一定的算法对旧表项进行替换

- 两级页表

    - 单级页表的缺点

        - 根据页号查询页表的方法，要求所有的页表项都连续存放才可以用页表基址加偏移的方法找到页表项

        - 很多时候进程在一段时间内只需要访问某几个页面就可以正常运行，不需要整个页表常驻内存（虚拟内存技术）

    - 可以将长页表进行分组，使得每个内存块刚好可以放入一个分组。对离散分配的页表再建立一张页表，称为页目录表（外层/顶层页表）

        假设内存块的大小为 4Kb (0000,0000,0000-1111,1111,1111)。如果将二级页表拆分装入内存，即每个页表块保存 1024（00,0000,0000-11,1111,1111）个块号并装入内存，再用页目录表保存至多 1024 个二级页表块，则逻辑地址结构为

        | **31-22** | **21-12** | **11-0** |
        |:------:|:------:|:------:|
        | 一级页号 P1 | 二级页号 P2 | 页内偏移 W |

        在转换时，用掩码分别读出 P1、P2、W ，根据 P1 和页目录表起始地址读出 P1 号二级页表块所在内存块，再通过二级页表块和 P2 读出物理地址对应内存块号 b ，最终得出物理地址 b:W

    - 若采用多级页表机制，则各级页表的大小不能超过一个页面

- 基本分段储存管理

    - 进程的地址空间按照程序自身的逻辑关系分为若干个段，每个段内从 0 开始编址。内存分配时以段为单位分配，每个段在内存中占有连续空间，但是段之间可以不相邻

    - 分段管理的逻辑地址分为高位的段号和低位的段内偏移量

    - 每个进程建立一张段映射表，简称段表，存放段长和基址。各个段表项大小相同。系统设置一个段表寄存器，负责从 PCB 中读取段表始址 F 和段表长度 M ，寻址操作类似基本分页储存管理

    - 分段和分页的区别

        - 页是信息的物理单位，分页的目的是实现离散分配，提高内存利用率。分页管理是系统行为，对用户不可见

        - 页的大小由系统决定且固定，分页的地址空间逻辑上是一维的

        - 段是信息的逻辑单位，分段的目的是满足用户需求，一个段通常包含一组属于一个逻辑模块的信息。分段对用户可见，用户需要显式给出段名

        - 段的大小取决于用户程序，分段的地址空间逻辑上是二维的

        - 分段比分页更容易实现信息的共享和保护

- 段页式管理

    - 先将进程按逻辑模块分段，再将各个段分页装入内存块中

    - 段页式系统的逻辑地址结构如下

        | **31-16** | **15-12** | **11-0** |
        |:------:|:------:|:------:|
        | 段号 S | 页号 P | 页内偏移 W |

        段号位数决定段个数，页号位数决定每个段最大页个数。寻址过程中对 S 和 P 都需要进行越界检查

    - 可引入段页号为关键字的快表辅助查询

### 3.4 虚拟内存

#### 3.4.1 基本概念

- 传统储存管理要求作业必须一次性装入内存后才能开始运行，这会造成两个问题

    - 作业很大时，不能全部装入内存，导致大作业无法运行

    - 当大量作业要求运行时，由于内存无法容纳所有作业，因此只有少量作业能够运行，导致多道并发度下降

- 一旦作业被装入内存，就会一直驻留内存直到作业结束。主要问题是很多暂时用不到的数据也会长期占用内存，导致空间浪费

- 基于局部性原理，在程序装入时，可以讲程序中很快会用到的部分装入内存，暂时用不到的部分留在外存。程序执行过程中，当所访问的信息不存在时，由操作系统负责将所需信息从外存调入内存，然后继续执行

- 如果内存空间不够，由操作系统负责将内存中暂时不用的部分换出到外存。在操作系统的管理下，用户看来似乎有一个比实际内存大得多的内存，这就是虚拟内存

- 虚拟内存的最大容量是由计算机的地址结构（ CPU 寻址范围）确定的，虚拟内存的实际容量是 CPU 寻址范围和内外存容量和的较小值

- 虚拟内存的特征

    - 多次性：无需在作业开始时一次性全部装入内存，而是允许分成多次调入内存

    - 对换性：在作业运行时无需一直常驻内存，而是允许在作业运行过程中将作业换入换出

    - 虚拟性：从逻辑上扩充了内存的容量

#### 3.4.2 请求分页管理

- 虚拟内存需要建立在离散分配的内存管理方式上。当所访问的信息不在内存时，由操作系统提供调页/调段，将程序从外存换入内存；当内存空间不足时，由操作系统提供页置换/段置换，将内存中暂时用不到的信息换入外存

- 请求分页储存管理的页表除了内存块号，还有以下信息

    - 状态位：是否已经调入内存

    - 访问字段：可记录最近被访问几次，或记录上次访问的时间，供置换算法选择换出页面

    - 修改位：是否被修改。没有修改的块调出时不需要修改外存中的备份

    - 外存地址：在外存中的存放位置

- 缺页中断

    - 在请求分页系统中，每当要访问的页面不存在时便产生一个缺页中断，然后由操作系统的缺页中断处理程序处理中断。此时缺页的进程被阻塞，放入阻塞队列。调页完成后再将其唤醒，放入就绪队列

    - 如果内存中有空闲块，则为进程分配一个空闲块并装入所缺页，修改相应的页表项。如果没有空闲块，则由页面置换算法选择一个页面淘汰

    - 缺页中断是内中断，一条指令执行期间可能会有多次缺页中断

- 地址变换机构

    - 先检查越界异常，再检查快表

    - 若快表未命中，在请求页表中找到对应表项后，若对应页面未调入内存，则产生缺页中断，由操作系统进行处理

    - 快表中的页面一定在内存中。若快表中的页面被换出，快表中的表项也要删除

    - 访问位和修改位需要更改

    - 也可以在调入后把表项装入快表，这样可以直接命中

#### 3.4.3 页面置换算法

- 最佳置换 OPT：每次淘汰以后永不使用或最长时间内不再使用的页面

    - 无法实现，因为不知道页面访问序列

- 先进先出 FIFO：每次淘汰最早进入的页面

    - 会产生 Belady 异常：当为进程分配的物理块数增大时，缺页率不减反增

    - 虽然实现简单，但与进程实际运行时的规律不符，算法性能差

- 最近最久未使用置换 LRU

    - 淘汰最近最久未使用的页面

    - 用访问字段记录上次访问以来经历时间，淘汰时选取最大的

        > 可以使用哈希链表实现

- 时钟置换 CLOCK （最近未用 NRU ）

    - 为每个页面设置一个访问位，再将内存中所有页面链成一个循环队列。当某页被访问时，访问位置 1

    - 需要淘汰页面时，检查访问位，找到并换出访问位为 0 的。如果第一次扫描的结果都是 1 ，则将页面的访问位全部置 0，再进行第二轮扫描。简单的 CLOCK 算法最多扫描两轮

- 改进的 CLOCK

    - 增加修改位，用来表示页面是否在内存中被修改

    - 在淘汰时，优先淘汰没有被修改的页面以避免 I/O 操作

    - 第一轮扫描（0，0）页面；若失败，扫描第一个（0，1）页面并将所有扫描过的页面访问位置 0；若第二轮扫描失败，重新扫描第一个（0，0）页面；若第三轮失败，重新扫描第一个（0，1）页面。改进的 CLOCK 算法最多扫描四轮

#### 3.4.4 页面分配策略

- 驻留集：请求分页储存管理中给进程分配的物理块的集合，一般小于进程总大小

- 驻留集过小会频繁缺页，过大会导致并发度下降。有两种分配策略

    - 固定分配：为每个进程分配一组固定数目的物理块，运行期间不再改变

    - 可变分配：运行期间适当增减物理块数目

- 置换的两种策略

    - 局部置换：发生缺页时只选择自己的物理块进行置换

    - 全局置换：可以将操作系统保留的空闲物理块分配给缺页进程，也可以将进程持有的物理块置换到外存，再分配给缺页进程

- 实际的页面分配可以组合上述的四种策略，较为理想的是可变分配局部置换，为频繁缺页的进程多分配物理块，对缺页率低的进程适当减少分配的物理块

- 何时调入

    - 预调页策略：主要用于进程的首次调入，由用户指定应该先调入哪部分

    - 请求调页策略：进程运行期间发生缺页时才将所缺页面调入内存， I/O 开销较大

- 从何处调入

    - 系统拥有足够的对换区空间时：页面的调入调出都发生在内存与对换区之间。在进程运行前，需要将进程相关数据从文件复制到对换区

    - 系统对换区空间不足时：不会被修改的数据直接从文件区调入，换出时无需写回磁盘；可能被修改的文件换出时写入对换区，下次需要时从对换区调入

    - UNIX 方式：运行前有关数据全部放在文件区。若使用过的页面换出，则写回对换区

- 抖动（颠簸）：刚刚换出的页面马上又要换入内存。产生抖动的主要原因是分配给进程的物理块不够

- 工作集：在某段时间间隔内，进程实际访问的页面的集合。操作系统会根据“窗口尺寸”计算工作集

- 工作集大小可能小于窗口尺寸，操作系统可以统计进程工作集的大小，并根据工作集大小分配内存块。一般驻留集大小不能小于工作集大小，否则进程运行过程中将频繁缺页

- 可以根据近期访问的工作集设计页面置换算法



#

- 上一节：[进程与线程](./OperatingSystems_02.md)
- 下一节：[文件管理](./OperatingSystems_04.md)