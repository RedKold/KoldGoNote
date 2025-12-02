# 作业

- [[ics-1]]
- [[ics-2]]
- [[ics-3]]
- [[ics-5]]
- [[ics-6]]

# pa
- [[ics-pa1-report]]
- [[ics-pa1 note]]
- [[ics-pa2 report]]
- [[ics-pa2 note]]
- [[ics-pa3 note]] 

# 授课
## [[ics - section 2 ：数据的机器级表示与处理]]
## [[ics - section 3 程序的转换和指令系统]]
## [[ics - section 4 程序的机器级表示]]

- from c to object code
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251108123433.png)

##  [[ics - section 5 程序的链接和加载执行]]

## ics - section 6 内存层次结构 memory hierarchy

### 存储技术和趋势 storage techonologies and trends
 
#### random-access memory (ram)
- key feature:
	- basic storage unit is normally a **cell** (one bit per cell)(存储 0 或 1 的记忆单元：**cell**)
	- multiple ram chips from a memory
- **ram** comes in two varieties
	- sram (static ram)
	- dram (dynamic ram)

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251115164716.png)

- **sram**:
	- application: cache memories
	- maybe needs edc ( error detection and correction)



#### nonvolatile memories (非易失性存储器)
- dram and sram are volatile memoreis
	- lose information if powered **off**
- nonvolatile memories retain value even if powered off
	- **rom**(read-only memory)
	- **prom**(programmable rom)
	- **eprom**(eraseable prom)
	- **eeprom**(electrically eraseable prom)
	- **flash memory**
- **uses**
	- frimwares. (bios, controllers for disks, network card)
	- solid state disks.



#### traditiona**l bus structure** connecting cpu and memory 总线
- a **bus**(总线) is a collection of parallel wires that carry address, data, and control signals.
-  buses are typically shared by multiple **devices**.

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251115165221.png)


but, it takes **time** to transport data on **bus** between cpu to different storage unit. (**记忆单元**) for multiple data use-situation, we need to setup a storage hierarchy


#### i/o bus
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251115170325.png)

bus as this sort of a single set of wires. ( 总线是一组电子线路)
where each wires carry a bit.


#### solid state disks (ssds)
**固态硬盘**
- from the view of cpu, ssds is just like a rotating **disk**. it has the same socket plug
- but, from the physical structure, ssds is a **set** of **flash memory**
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251115170838.png)
- between i/o bus and flash memory layer, there is a flash translation **layer**. it's like a control layer

#### **locality**
程序有个属性：**局部性**
- principle of locality:
	- programs tend to use data nad instructions with addresses near or equal to those they have used recently


- temporal **locality** **时间局部性**
	- recently referenced items are likely to be **referenced** again in the **near future**
- spatial locality **空间局部性**
	- items with nearby addresses tend to be referenced close together in time


> [!example] locality example
> ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251115172230.png)
> ```c
> sum = 0;
> for (i=0; i<n; i++)
> 	sum += a[i]
> return sum;
> ```
> 对数组的访问具有：
> - **时间局部性**：我们对某元素的访问，很可能在将来被访问 (`sum`)
> - **空间局部性**：我们接连 (in sucession) 访问数组的元素


> [!example] use locality to impore your program's performance
> ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251115172932.png)
> this function visit the **array** by **rows** , then by **cols**. it's a good way.
> because the array storage pattern is **rows** first. so when rows `i` is fixed, `a[i][j]` is trying to access a series of nearby address. it's spatial **locality**
> while for **tempory locality**, we access `sum` each iteration. it's a tempory locality example. 

> [!summary] locality and how you visit a 2-dimension array
> 





insighted by **locality**, **cache** is created.
- store the high-access-frequency element, in storage unit with higher efficiency.
- a balance.
	- fast storage technologies cost more per byte, have less capacity, and require more power (heat!)
	- the gap between cpu and main memory speed is widening
	- well-written programs tend to exhibit **good locality**
**they suggest an approach for organizing memory and strorage systems known as a memory hirerachy**
- **存储器层次结构**

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251115175314.png)
 
- remote secondary storage
	- web servers as **google** or **amazon**

---

### caches
- ***cache***: a smaller, faster storage device that acts as a **staging area** for a subset of the data in a **larger, slower device.**
- **基本思想**：
	- for each `k`, the faster, smaller device at level `k`  serves as a **cache** for teh larger, slower device at level `k+1`.
	- why do memory hierarchies work?
		- because of [[#**locality**|locality]], programs tend to access the data at level `k` more often than they access the data at level ` k+1 `
		- 所以更高层级的可以更快、更贵。更低层级的可以更慢，更便宜。**economic**
**层次结构**创建了一个巨大的**存储池**。存储池的总量是最底层的容量。



#### types of cache misses

- **cold (compulsory) miss**
	- cold misses occur because the cache is **empty**
- **conflict miss** 
	- **哈希表映射不够用了**
	- conflict misses occur when the level k cache is large enough, but multiple data objects all map to the same level k block
- capacity miss
	- occurs when the set of active cache blocks (**working set**) is larger than the **cache**. （缓存不够存块）

#### cache memory
- cache memories are **small**, **fast** sram-based memories managed automatically in hardware
	- hold frequently accessed blocks of main memory
- cpu looks first for data in cache
- typical system structure



address of word:
```
address of word:

| t bits	| s bits	 | b bits	 	|
| tags		| set index	 | block offset |
```


确定 set **的过程**:
- set **的位数**取决于一个 cache 能装多少块
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251117153602.png)

- locate set
- check if any line in set has matching ***tag***
- yes + line valid: ***hit***
- locate data starting at ***offset***


#### direct mapped cache (e=1) 直接映射 cache
direct mapped: one line per set 
assume: cache block size 8 bytes

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251117154545.png)
- 直接映射方式类似一个哈希表
- if tag match: assume yes = **hit**
- if tag not match: old line is evicted and replaced

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251117172236.png)


#### e-way set associative cache (2 路组相连缓存)
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251117155430.png)

- 组（**set**）：缓存被划分为 `s` 个组（set）, 每个组包含 `e` 个缓存行 (cache line)。在 2 路组相联缓存中，`e=2`, 即 **每组包含 2**条缓存行（路）
- 缓存行 (cache line) 缓存中的基本存储单元，每行包含：
	- 有效位（valid bit）
	- 标记位 (tag bit)
	- 数据块 (data block)
当cpu发出一个**内存地址** $a$ 时，这个地址会被逻辑上分成三个部分：
$$\text{内存地址 } a = [\underbrace{\text{tag}}_{\text{标记 } t \text{ 位}} \mid \underbrace{\text{set index}}_{\text{组索引 } s \text{ 位}} \mid \underbrace{\text{block offset}}_{\text{块偏移 } b \text{ 位}}]$$
1. **块偏移（block offset, $b$ 位）：** 用于在数据块内定位所需的特定字节。$b = 2^b$ 字节是块大小。
    
2. **组索引（set index, $s$ 位）：** 用于确定该地址应映射到缓存中的哪一个**组**。$s = 2^s$ 是组的数量。
    
3. **标记（tag, $t$ 位）：** 用于在选定的组中，唯一标识**具体**是哪个主存块的数据存放在了该缓存行中。


#### 缓存读取（cache read）过程

当cpu请求读取内存地址 $a$ 处的字时，缓存会执行以下步骤：

1. **组选择（set selection）：**
    
    - 缓存使用地址中的 $s$ 位**组索引**来定位 $s$ 个组中的唯一一个**目标组**。
        
2. **行匹配（line matching）：**
    
    - 在选定的目标组内，缓存会**并行地**检查该组中的所有 $e=2$ 条缓存行。
        
    - 对于每一行，检查两个条件：
        
        - **有效位（valid bit）** 是否为 1（表示该行是有效的）。
            
        - 行中的**标记位（tag）** 是否与地址中的 $t$ 位**标记**相匹配。
            
3. **命中（hit）或不命中（miss）：**
    
    - **命中（cache hit）：** 如果找到了一个有效位为 1 且标记匹配的缓存行，则缓存命中。它使用**块偏移** $b$ 位从该行的数据块中取出所需的数据，并返回给cpu。
        
    - **不命中（cache miss）：** 如果目标组中的所有 $e=2$ 条缓存行都没有满足匹配条件的，则缓存不命中。
        
4. **不命中处理（miss handling）：**
    
    - 发生不命中时，缓存必须从下一级存储器（通常是主存）取出整个数据块，并将其存入选定的组中。
        
    - **行替换（line replacement）：**
        
        - 由于每组只有 $e=2$ 个位置，如果该组中有一条缓存行是无效的（valid bit = 0），则可以直接将新数据存入该空闲行。
            
        - 如果两行都有效（valid bit = 1），则必须选择其中一行替换掉。这时会使用**替换策略**（replacement policy），最常见的是 **lru (least recently used，最近最少使用)** 策略，即替换掉组中最近最久未使用的缓存行。


#### 缓存写过程 (cache write)
**写策略**
- 通写法 (write through)(全写法、直写法或写直达法)
	- write immediately to memory
	- 如果写命中，则同时写 cache 和主存
	- 如果写缺失，则先写主存，并有处理方式：
		- 写分配法 (write-allocate)
			- load into cache, update line in cache
				- good if more writes to the location follow
		- 非写分配法 (no-write-allocate)
			- writes straight to memory, does not load into cache
- 回写法 (write back)（一次性写，写回法）
	- defer write to memory until replacement of line
		- need a dirty bit (line different from memory or not)
		- **关联一个修改位**(dirty bit)。向 cache 装入新主存块，清 0. 
			- cpu 写入 cache 行，置 1. 
			- 替换 cache 行时候检查 dirty bit，
				- 如果为 1，需要 **write back** to main memory. 
				- if 0, no need to write back. (之前没有被修改过，可以直接替换！)
	- 若写**命中**，则只将**内容存入 cache 而不写入主存；**
	- 若写缺失，则分配一个 cache 行并**装入主存块**（此时更新），然后更新该行的内容。
	- 回写法的主要目的是减少 cpu 与主存之间的通信量（bus traffic）。其工作流程依赖于一个关键的标志位：**修改位（dirty bit）**。
	- **驱逐/替换（eviction）：** 当缓存满了，需要腾出位置给新数据时：
	    - 如果被踢出的块 **dirty bit = 0**（未被修改）：直接覆盖，无需任何操作。
	    - 如果被踢出的块 **dirty bit = 1**（已被修改）：先将该块的内容写回主存，然后再覆盖。


![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251126152835.png)



## ics - section 7 虚拟存储器
**可寻址的地址空间** **是一种虚拟内存**
我们希望可以解释访存指令 `mov( , , ,)` 在机器放生的一切 
### 分页 paging

- 基本思想
	- **内存**被分成固定长且比较小的存储块（页框、实页、物理页）
	- 每个进程也被划分为固定长的程序块（页，虚页、逻辑页）
	- **程序块**可装入主存页框中
	- 无需用连续页框存放一个进程
	- os 为每个进程生成一个页表
	- 通过**页表**(page table) 实现***逻辑地址到物理地址转换***
- **逻辑地址** (logical address)
	- 程序中指令所用地址（进程所在地址空间），也称为**虚拟地址**(virtual address, **va**)
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


### linux 的存储保护机制

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201110204.png)


## ics - section 8 异常处理控制流

### 进程与进程的上下文切换

**cpu**所执行的***指令的地址序列*** 称为 cpu 的控制流
- 正常控制流
- 异常控制流 (exception control of flow, ecf)
	- 形成原因：
	- 内部异常 (hardware)
	- 外部中断 (hardware)
	- 进程的上下文切换（操作系统）
	- 一个进程直接发送信号给 lingerie 进程（应用软件层）
#### 程序和进程
- **program**: 静态过程
- **process**: 程序的一个运行过程。确切的说，是一个具有独立功能的程序关于某个数据集合的一次运行活动，因此具有 **动态含义**。同一个程序处理不同的数据就是不同的进程


对于单处理器系统，进程会轮流使用 **处理器**。处理器的物理控制流，**由不同逻辑控制流交叉组成**。

- **并发**: concurrency
	- 不同进程的逻辑控制流在时间上交错或重叠的情况

#### 进程与上下文切换 context switching
os 通过**处理器调度**让处理器轮流执行多个进程。实现不同进程中指令交替执行的机制。称为**进程的上下文切换**。(context switching)



