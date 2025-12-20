## ics - section 7 虚拟存储器
**可寻址的地址空间** **是一种虚拟内存**
我们希望可以解释访存指令 `mov( , , ,)` 在机器放生的一切 

Why we need Virtual Memory?
- Uses main memory efficiently
	- Use DRAM as a cache for parts of a virtual address space
- Simplifies memory management
	- Each process gets the same uniform linear address space
- Isolates address spaces (创建受保护的私有地址空间，进程独立的)
	- One process can't interfere with another's memory
	- User program cannot access privileged kernel information and code

- Vm as a tool for **Caching****
### 分页 paging

- 基本思想
	- **内存**被分成固定长且比较小的存储块（页框、实页、物理页）
	- 每个进程也被划分为固定长的程序块（页，虚页、逻辑页）
	- **程序块**可装入主存页框中
	- 无需用连续页框存放一个进程
	- os 为**每个进程生成一个页表**
	- 通过 [**页表**(page table) ](#enabling-data-structure-page-table) 实现***逻辑地址到物理地址转换***
- **逻辑地址** (logical address)
	- 程序中指令所用地址（进程所在地址空间），也称为**虚拟地址**(virtual address, **va**)
	- 我们希望对于用户、程序来说，**地址是连续的**，**所以这就是线性地址**。
- **物理地址**(physical address, **pa**)
	- 存放指令或数据的实际主存地址，也称为**实地址**、**主存地址**

我们不需要将一个**进程**的全部装入内存，**根据局部性**，我们可以把活跃的页面调入主存，其余留在磁盘上。




![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201112457.png)

> [!note] **虚拟存储系统的实质**
> **主存**的容量受到限制；
> **主存**容量要求越来越大；-> *conflict*!
> 程序员在比实际主存空间大得多的逻辑地址空间编写程序。
> 程序执行的时候，把当前需要的程序段和相应的数据库调入主存，其他放在磁盘上。

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201113145.png)
- **实质**：通过页表建立虚拟空间和物理空间之间的映射
	- `虚拟空间 <-> 物理空间`


### linux 虚拟地址空间
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201113211.png)

- **内核空间**
	- 与进程相关、对每个进程都相同
- **用户栈**(user stack)
- 共享库 (shared libraries)
- 堆（heap）
- 可读写数据 (read/write data)
- 只读数据和代码 (read-only data/code)

**加载程序**的时候，我们不会真正从磁盘调入信息到主存，而是将虚拟页和磁盘上的数据/代码构建对应关系，**即存储器映射**。
这是 `mmap` 系统调用。

### 虚拟存储器管理
- 块大小多大？(here we call page 页)
- 主存和虚存如何分区管理？
- 页和页框如何映射？
- 逻辑地址和物理地址如何转换？
- 页表如何实现？页表项记录哪些信息？
- 如何加快访问页表的速度？
- 如果要找的内容不在主存，怎么办？
- 如何保护进程个字的存储区不被其他进程访问？

- 3 虚拟存储器实现方式
	- 分页式
	- 分段式
	- 段页式

- **主存--磁盘**层次存储
这个层次，比 `cache - 主存` 更大，我们需要更大的页！
- why?
	- if cache hit failed, we need to access memory
	- but if page hit failed, we need to access **disk**! cost highly
	- 通过软件处理缺页
	- 使用 write back 策略
		- 避免频繁的慢速磁盘访问操作
	- 地址转换用硬件实现
		- 加快指令执行 


### 页式虚拟存储器

**每个进程有一个页表**，每个虚拟页在页表中有一个对应的表项目
- **页表项**
	- 包括虚拟页的存放位置，装入位（valid），修改位（dirty）、使用位、访问权限位和禁止缓存位
- 页表项的**存放位置**字段，用于建立虚拟页和物理页框之间的映射
	- 虚拟页和物理页大小一样
	- 虚拟地址分位两个字段：
		- **高位字段**：虚拟页号（即虚页号，逻辑页号）
		- **低位字段**：页内偏移地址（**页内地址**）
	- 主存物理地址也是两个字段
		- **高位字段**：物理页号
		- **低位字段**：页内偏移地址
	- 二者页内偏移地址 `offset` 是一样的

| **字段名称**  | **英文缩写** | **线性地址位数**       | **作用**                                      |
| --------- | -------- | ---------------- | ------------------------------------------- |
| **页目录索引** | PDI      | VA[31:22] (10 位) | 用于查找页目录表 (Page Directory) 中的**页目录项 (PDE)**。 |
| **页表索引**  | PTI      | VA[21:12] (10 位) | 用于查找页表 (Page Table) 中的**页表项 (PTE)**。        |
| **页内偏移量** | Offset   | VA[11:0] (12 位)  | 用于定位页框 (Page Frame) 内的**具体字节**。             |

### 信息访问的异常情况 (exception)
- **缺页**(page fault)
	- **产生条件**：if `valid = 0`
	- **相应处理**：
		- 从磁盘读页面到主存，若主存没有空间，则从主存选择一页替换到磁盘上
		- 替换算法类似于 cache ，回写法。淘汰时，根据 `dirty` 位确定是否写磁盘
	- **当前**指令执行阻塞，进程挂起，处理结束后回到原指令执行
- **保护违例**（protection_violation_fault）
	- **产生条件**：当 access rights（存取权限）与所指定的具体操作不相符时
	- **相应处理**：在屏幕上显示“内存保护错”或“访问违例”信息
	- 当前指令执行阻塞，当前进程被终止
	- `access right:`
		- `r = read-only, r/w = read/write, x = execute only`

| **字段**     | **名称（英文）**                           | **位数** | **意义**                                               | **典型值（代码页/第一次执行）** |
| ---------- | ------------------------------------ | ------ | ---------------------------------------------------- | ------------------ |
| **P**      | **P**resent Bit (存在位)                | 1      | **指示该页是否在主存中。**                                      | $\text{P}=1$       |
|            |                                      |        | - $\text{P}=1$: 页框在物理内存中，地址转换可以继续。                   |                    |
|            |                                      |        | - $\text{P}=0$: 页框不在物理内存中（可能在磁盘上），将触发**缺页中断**。       |                    |
| **R/W**    | **R**ead/**W**rite Bit (读/写位)        | 1      | **指示该页是否允许写入。**                                      | $\text{R/W}=1$     |
|            |                                      |        | - $\text{R/W}=1$: 允许读和写（或执行，结合 $\text{U/S}$）。        |                    |
|            |                                      |        | - $\text{R/W}=0$: 只读。                                |                    |
| **U/S**    | **U**ser/**S**upervisor Bit (用户/管理位) | 1      | **指示哪个特权级可以访问该页。**                                   | $\text{U/S}=1$     |
|            |                                      |        | - $\text{U/S}=1$: **用户级（CPL=3）**可访问。                 |                    |
|            |                                      |        | - $\text{U/S}=0$: 仅**管理级（CPL=0, 1, 2）**可访问。          |                    |
| **A**      | **A**ccessed Bit (访问位)               | 1      | **指示该页自上次重置以来是否被访问过。**                               | $\text{A}=1$       |
|            |                                      |        | - 每当 MMU 读取或写入该页时，硬件自动将其设置为 1。主要用于操作系统进行页置换时的统计。     |                    |
| **D**      | **D**irty Bit (脏位)                   | 1      | **指示该页自加载以来是否被写入（修改）过。**                             | $\text{D}=0$       |
|            |                                      |        | - 仅当 R/W=1 且页被写入时，硬件才将其设置为 1。主要用于判断页置换时是否需要将页内容写回磁盘。 |                    |
| **高 20 位** | 页框号 (PFN)                            | 20     | **页在物理内存中的起始地址**（需要乘以 $4 \text{KB}$）。                |                    |

### linux 的存储保护机制

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201110204.png)


### DRAM Cache Organization
- DRAM cache organization driven by the enormous miss penalty
	- DRAM is about *10x* slower than SRAM
	- Disk is about *10,000x* slower than DRAM

- 以 Cache 视角来看：虚拟存储是完全联合的 (full associative)
	- Any VP can be placed in any PP
	- Requires a "large" mapping function - different from cache memories
- Highly sophisticated, expensive replacement algorithms
- Write-**back** rather than write-through
	- 都是为了减少成本。因为 access disk is very slow!

我们使用的数据结构： 页表

---
#### Enabling Data Structure Page table
A *page table* is an array of page table entries (PTEs) that maps virtual pages to physical pages
- Per-process kernel data structure in **DRAM**


#### Page Hit
类似于缓存
- ***Page Hit***: reference to **VM** word  that is in physical memory (DRAM cache hit)

#### Page Fault
- ***Page Fault***
	- reference to VM word that is not in physical memory (DRAM cache **miss**)
- 和缓存的 miss 相当类似
- 对未在虚拟表中缓存的物理地址引用

**处理**-Handling Page **Fault**
- Page miss causes **page fault**
- Page fault handler selects a **victim** to be evicted (here VP 4)
	- 我们找一个表项，替换成这个新的地址条目
	- 这里涉及到切换进程进行处理
- Offending instruction is restarted: ***page hit!***


**我们**通过局部性 (Locality) 的应用，让这一套方案获得比较好的效果
我们常访问的，实际是一套**工作集**
- At any point in time, programs tend to access a set of active virtual pages called the *working set*
	- Programs with better temporal locality will have smaller working sets

- If (working set size < main memory size)
	- Good performance for one process after **compulsory** (必须的)misses
	- 除了必要的第一次 miss，具有良好的表现。各项目不冲突。hit rate is high
- If (SUM (working set sizes) > main memory size)
	- **Thrashing***: Performance meltdown where pages are swapped in and out continuously
	- 工作集总大小如果大于主存大小，则..
### VM as a Tool for Memory Management
- Key idea: each process has its own **virtual address space**(linear address: 线性地址，虚拟地址)
	- It can view memory as a simple **linear** array
	- Mapping function **scatters** addresses through physical memory
		- Well-chosen **mappings** can improve locality
我们可以为程序员提供一个**更简单的视图**，程序员有一个更顶层的使用内存的**封装**, 利用虚拟内存

- Simplifying memory **allocation**
	- Each virtual page can be mapped to any physical page
	- A virtual page can be stored in different physical pages at different times



### 控制寄存器 CR
控制寄存器保存机器的各种控制和状态信息，它们将影响系统所有任务的运行，操作系统进行任务控制或存储管理时使用这些控制和状态信息。 

**CR0：控制寄存器**
- PE: 1-保护模式；0-实地址模式。
	- **Protection Enable**
- PG：1-启用分页；0-禁止分页，此时**线性地址被直接作为物理地址**使用。若要启用分页机制，则 PE 和 PG 都要置 1。
	- **Paging Enable**
- 任务切换位 TS：任务切换时将其置 1，切换完毕则清 0，可用 CLTS 指令将其清 0。
- 对齐屏蔽位 AM。
- cache 功能控制位 NW（（Not Write-through）和 CD（Cache Disable）。只有当 NW 和 CD 均为 0 时，cache 才能工作。

**CR2：页故障（page fault）线性地址寄存器**
- 存放引起页故障的线性地址。只有在 CR0 中的 PG=1 时，CR2 才有效。
**CR3：页目录基址寄存器** 
- 保存页目录表的起始物理地址。只有 CR0 中的 PG=1 时，CR3 才有效。

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251213120048.png)

### **TLB**
**TLB 的性质：** TLB (Translation Lookaside Buffer) 是 CPU 内部的一块高速缓存，用于存储**最近使用过的虚拟页号 (VPN)** 到 **物理页框号 (PFN)** 的映射关系。
