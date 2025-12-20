## ics - section 6 内存层次结构 memory hierarchy

### 存储技术和趋势 storage technologies and trends
 
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