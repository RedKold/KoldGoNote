## L1

- 教学目标：
	- **大数据处理技术**、并行计算技术思想，并行计算系统基础架构
	- Hadoop、Spark
	- MapReduce 和 Spark 并行程序设计和基础算法
	- 课程实践，将大数据处理技术应用到金融领域的应用中。
- **课程性质**
	- Not a language lesson
		- but Python / **Java** / Scala needed
	- Not a data mining lesson
		- but will teach some important algorithm
	- Not a distributed Parallel Computing lesson
		- but will ask to operate distributed Parallel Computing system

- **课本**
	- 《深入理解大数据——大数据处理与编程实现》

- **考核方式**
	- 平时 10%
		- [[FBDP-1]] 
		- [[FBDP-2]]
		- [[FBDP-3]]
		- [[FBDP-4]]
		- [[FBDP-5]]
		- [[FBDP-6]]
		- [[FBDP-7]]
		- [[FBDP-8]]
	- 实验 **40%**
		- [[FBDP-lab1]]
		- [[FBDP-lab2]]
		- [[FBDP-lab3]]
	- 期末笔试 **50%**

- Data Science
	- 已经成为了科学研究的**第四范式**
	- 通过数据发现理论和规律。
- 计算机科学～关于算法的科学
- 数据科学～关于数据的科学
	- 存储 Storing ？
	- 计算 Processing？
	- 管理 Managing？
	- 分析 Analyzing？

什么是大数据？
- 数据存储访问能力大幅落后数据增长速度
- 传统关系数据库已经无法应用大数据的存储和处理


- 思维变革：
	- 大数据的简单算法 > 小数据的复杂算法

- 大数据的 5V 特征
	- Volume 大体量
	- Variety 多样性
	- Velocity 时效性
	- Veracity 准确性
	- Value 大价值

- **大数据的类型**
	- 结构特征
		- 结构化？非结构化？
	- 获取和处理方式
		- 动态/实时
		- 静态/非实时
	- 关联特征
		- 无关联/简单的数据
		- 复杂关联数据


## L2

- **数据平台技术演变**
	- GFS
	- MapReduce
	- BigTable
	- 2006: 开源 Hadoop
	- 2009： Spark
- ![image.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/20250828144420.png)


- **四大挑战**
	- **大数据管理**
		- 数据为中心的计算体系
	- **大数据处理**（批处理、流处理、图计算）
	- **大数据分析**（多源异构数据的可解释性分析）
		- 多模态、联邦学习、因果推断
	- **系统化大数据治理框架和关键技术**


- 商业价值：
	- 个性营销
	- 企业经营决策
	- 核心：**大数据**
		- 颠覆信息不对称问题
	- 大数据公司

- **金融数据**：**流数据**
	- 短时间快速处理
	- 逻辑关系紧密
	- 处理实时性要求高
	- 可展示性需求强
- **金融大数据**

- **提高计算机性能**
	- **提高处理器字长**
		- 32bit 64bit
- **提高集成度**
	- 摩尔定律。
- 流水线等微体系结构技术
	- RISC
	- 5 级流水线
	- 分支预测
	- 寄存器重命名

- **瓶颈**
	- 处理器的指令级并行度提升遇到瓶颈
	- 处理器速度和存储器速度差异越来越大
- 功耗+散热>>>芯片承受能力


- **多核发展成为趋势**
	- 2005, intel, 多核/众核并行计算
- **多核众核并行计算**
	- Huang's Law - GPU push the efficiency of AI double over year

---


- **并行计算技术的分类**
- 按并行类型
	- **Flynn's taxonomy**
		- SISD
			- 单指令单数据流
			- 传统单处理器串行处理
		- SIMD
			- 单指令多数据流
			- 向量机，信号处理系统
		- MISD
			- 多指令单数据流
			- 很少
		- MIMD
			- 多指令多数据流
			- 最常用。
		- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250901141635.png)
-  按并行类型
	- 位级并行
		- Bit-level Parallelism
	- 指令级并行
		- ILP: Instruction-Level Parallelism
	- 线程级并行
		- Thread-Level Parallelism
		- Data level:
			- **大数据块划分为小块**
		- Task Level
			- **大的计算任务划分为子任务**
- 按存储访问结构
	- 共享内存 Shared Memory (UMA)
		- Uniform Memory Access
	- 分布共享存储体系结构 - NUMA 结构
		- Non-Uniform Memory Access
		- 各个处理器有自己的**本地**存储器
		- 同时再共享一个全局的
	- 分布式内存 - NUMA 结构
		- 各个处理器使用自己独立的存储器
-	 ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250901142217.png)
- 按系统类型
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250901142326.png)
		- **MPP**并行的不一定是同一类型的处理器，彼此连接，最大特点事 **规模大**
		- 数据中心采用集群 Cluster
			- 网络连接
			- 物理距离靠近
			- 所谓刀片服务器
		- 网格  Grid
		- 云 Cloud
				- 通过互联网按需访问计算资源
	- **总结一下**
		- 从 MC 到 Cloud，**耦合度**越来越低，**可拓展性**越来越高，**系统规模**越来越大，**能耗**越来越高
		- **MC**
			- NOC（刀上网络）。混合式内存访问机制，功耗低
			- **SMP**独立的处理器和共享内存，**总线链接，运行一个操作系统**，定制成本高，难以扩充
			- MPP 独立的处理器、独立的内存、OS，专用的告诉内联网络，难以升级扩充，规模中等
			- **Cluster** 商品化的刀片服务器，最常用，扩展性强，规模可小可大
			- Grid 地理上广泛分布
			- **Cloud**互联网按需访问计算资源
	- **计算特征分类**
		- 数据密集型并行计算 Data-Intensive Parallel Computing
			- 大规模 Web 信息搜索
		- 计算密集型并行计算 Computation-Intensive Parallel Computing
			- 3D 建模渲染，科学计算
		- 混合型
			- 3D 电影渲染
	- **并行程序设计模型/方法分类**
		- 共享内存变量 (Shared Memory Variables)
			- 数据不一致冲突的解决办法
			- 同步控制机制：Pthread, OpenMP: 共享内存分发
				- remember you used pthread in cs network course
		- Message Passing
			- 分布式内存，分发数据/收集计算结果，需要用消息分发
		- MapReduce 方式
			- Google
			- 易于使用的设计方法
			
- 分布式数据与文件管理
	- **并行计算**：大规模集群，如何解决大数据块的划分、存储和访问管理。
	- 要求提供分布式数据和文件管理系统
		- Google GFS (Google File System)
		- Hadoop HDFS (Hadoop Distributed File System)


- **系统性能评估和程序并行度如何评估**？
	- 系统性性能
		- Benchmark 方法
		- TOP500 use
	- 程序并行度评估
		- 程序能得到多大并行加速依赖于该程序有多少可并行计算的比例。经典的程序并行加速评估公式 Amdahl **定律**
$$
S=\frac{1}{(1-P)+\frac{P}{N}}
$$
where $S$ is speed ration, $P$ is program parallelized  ratio, $N$ is amount of programmer


- **MPI**并行程序设计
	- Message Passing Interface
	- Message Passing based high performance parallel computing program interface
- **MPI** main function
	- All nodes run the same one program, but dealing **different** data
	- **Point-point communication**
	- **Collective communication**
		- one to all broadcast communication
		- multiple nodes compute  synchronized control （同步控制）
		- 对结果的规约 (Reduce) 计算功能
- MPI 并行程序设计接口
	- 初始化和结束
		- `MPI_Init`, `MPI_Finalize`
	 	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250901153354.png)
	
	- 通信组（Communicator）
		- **划分不同通信组**。
		- 最大的缺省通信组  `MPI_COMM_WORLD`
		- 总进程用 `MPI_Comm_Size` 确定
	- 进程标识
		- `MPI_Comm_Rank`
- **点对点通信**
	- **同步**：阻塞式。等待通信操作完成才返回
		- `MPI_Send`
		- `MPI_Recv`
	- **异步**：非阻塞


- **节点集合通信接口**
	- 同步 Barrier
		- `MPI_Barrier`
		- 设置同步障使所有进程的执行同时完成
	- 数据移动 Data movement
		- `MPI_BCAST`
			- one to all
		- `MPI_GATHER`
			- multiple process' s message gather to one process
		- `MPI_SCATTER`
			- one infomation cut into pieces 
	- 数据规约 Reduce
		- `MPI_REDUCE`


- 具体说说 `MPI_Reduce`
	- 将一组进程的数据按照指定的操作方式规约到一起并传送给一个进程
	- 求最大值、求和、求最小值，逻辑与、按位与... 最小值和位置



### MapReduce
- **问题与需求**
	- 巨量的 Web 文档建立索引的方法
- **解决方案
	- 分布式计算环境和框架。
- What is MapReduce
	- Google 发明的面向大规模海量数据处理的高性能并行计算平台和软件编程框架


#### 为什么分而治之？

- **什么样的计算任务可以进行并行化计算？**
	- 第一个重要问题：
		- 如何划分计算任务或者计算数据以便对划分的子任务或数据块同时进行计算
	- But some question can't divide
		- **Fibonacci**


#### Map 与 Reduce
借鉴了 **Lisp** 的设计思想
函数编程思想

总结什么是 Reduce：
**将一组进程的数据按照指定的操作方式规约到一起**，并传送给 **一个进程**

`MPI_MAX` 
`MPI_SUM`
`MPI_LAND`

## L3

### **典型流式大数据问题的特征**

- 大量数据记录/元素进行重复处理
- 对每个数据记录/元素作感兴趣的处理、获取感兴趣的中间结果信息
	- **MAP**
- 排序和整理中间结果以利于后续处理

- 收集整理中间结果
- 产生最终结果输出
	- **REDUCE**

### Map and Reduce Abstract Model
借鉴了函数式程序设计语言 Lisp 的思想。

Map 和 Reduce 两个抽象的接口

`map: (k1,v1) -> [(k2;v2)]`
- input:
	- key-value pair data
- process:G
	- 处理键值对，以另一种键值对形式输出中间结果
- output:
	- 键值对 `[(k2;v2)]` 表示的一组中间数据

`reduce: (k2;[v2])-> [(k3,v3)]`
- input:
	- map 输出的一组键值对将通过合并处理，**将同样主键下的**不同数值合并到一个列表 `[v2]` 种
- process
	- 对传入的中间结果列表数据进行处理整理，产生最终的结果输出 `[(k3;v3)]`
- output
	- `[(k3,v3)]`



### Map 和 Reduce 的并行计算模型

- 各个map函数对所划分的数据并行处理，从不同的输入数据产生不同的中间结果输出 
- 各个reduce也各自并行计算，各自负责处理不同的中间结果数据集合 
- 进行reduce处理之前，必须等到所有的map函数做完，因此，在进入reduce前需要有一个**同步障(barrier)；** 
	- 这个阶段也负责对map的中间结果数据进行收集整理(aggregation & shuffle)处理，以便reduce更有效地计算最终结果
- **最终汇总所有 reduce的输出结果即可获得最终结果**

- eg
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250908144432.png)
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250908144448.png)
### 如何提供统一的计算框架
- **主动需求和目标**
	- 实现自动化并行计算
	- 为程序员隐藏系统层细节
- **需要考虑的细节技术问题**
	- 如何管理和存储数据？如何划分数据？
	- 如何调度计算任务并分配 map 和 reduce 节点？
	- 如果节点间需要共享或交换数据怎么办？
	- 如何考虑数据通信和同步？
	- 如何掌控节点的执行完成情况？如何收集中间和最终的结果数据？
	- 节点失效如何处理? 如何恢复数据？如何恢复计算任务？
	- 节点扩充后如何保证原有程序仍有正常运行并保证系统性能提升？
- MapReduce 需要写盘，IO 开销很大！
	- Spark 优化了这一点

**计算框架**，可完成：
- 计算任务的划分和调度
- 数据的分布存储和划分
- 处理数据与计算任务的同步
- 结果数据的收集整理（sorting, combining, partitioning）
- 系统通信、负载平衡、计算性能优化处理
- 处理系统节点出错检测和失效恢复

- MapReduce **主要功能**
	- **任务调度**
		- job -> tasks
		- map and reduce
		- monitor the state of nodes.
		- barrier
	- 数据/**代码互相定位**
		- 基本原则：locality
		- 本地化数据处理。（减少数据通信）
- **出错处理**
- 分布式数据存储与文件管理
	- 多备份
- Combiner and Partitioner
	- 中间结果数据进入 reduce 节点前，需要合并 (combine)


- **顺序访问**和 **随机访问**性能上差异巨大

### Google 三驾马车

- The Google File System
	- SOSP2003
	- **GFS**
- **MapReduce**: simplified data processing on large clusters
	- OSDI2004
- **Bigtable**: a distributed storage system for structured data
	- OSDI2006


- MapReduce **并行处理的基本流程图**


## L4 - Google MapReduce 详解

### 并行处理的基本过程
![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20250911144518.png)


1. **一个待处理的大数据**，被划分为大小相同的数据块，**及与此对应的用户作业程序**(User Program)
2. 系统中有一个负责调度的**主节点**(Master)，以及数据 Map 和 Reduce **工作节点**(Worker)
3. 用户作业程序提交给主节点 (Master)
	1. 对于 MapReduce 程序，我们实现的程序逻辑实际的执行者是云端的各个设备
4. 主节点为作业程序寻找和配备可用的Map节点，并将程序传送给Map节点 
5. 主节点也为作业程序寻找和配备可用的 Reduce 节点，并将程序传送给 Reduce 节点。
6. 主节点启动每个 Map 节点执行程序，每个 Map 节点尽可能 **读取本地或本机架** 的数据进行计算 (不要跨机房、跨中心读)
	- **Locality**
7. 每个 Map 节点处理读取的数据块，并做一些数据整理工作 (combining, sorting)，并将中间结果存放在本地；同时通知主节点计算任务完成并告知中间结果数据存储位置
8. 主节点等所有 Map 节点计算完成后，开始启动 Reduce 节点运行；Reduce 节点从主节点所掌握的中间结果数据位置信息，远程读取这些数据 
	- **我们本地**数据中心存好**中间结果**数据，Reduce 远程读，
	- 这样可以保留：数据间的关系
	- 提高鲁棒性。如果 Reduce 节点坏了，不会影响数据安全，也不用前面的节点重新计算
9. Reduce 节点计算结果汇总输出到一个结果文件，**即获得整个处理结果**

### 失效处理

- IF 主节点失效
	- 主节点周期设置检查点 `checkpoint`，检查整个作业的运行情况。if failed, 可以从最近的有效检查点开始重新执行
	- 如果只有一个 Master，不太肯跟失败，如果失败就中止计算
- **工作节点失效**
	- 很普遍发生
	- 主节点周期行发送检测命令给工作节点，if not response, 认为失效。
	- 重新调度
- 


### 计算优化

- **问题**
	Reduce 节点必须等到所有 Map 节点计算结束之后才能开始执行！
	- 拖后腿问题
- **解决方案**
	- 冗余 Map：
	- 一个 Map 计算任务给多个节点同时做，取最快者的
- Google 测试：提高了 40%效率


### 数据分区解决数据相关性问题

- **问题**
	- 一个 Reduce 节点上的计算数据可能来自多个 Map 节点，因此为了在进入 Reduce 计算之前，需要把属于一个 Reduce 节点的数据归并到一起。
- 解决方案
	- 在 Map 阶段进行了 Combining 后，可以根据一定的策略对 Map 输出的中间结果进行分区 (Partitioning)，这样即可解决 Reduce 计算过程中的 **数据通信**
> 例如：有一个巨大的数组，其最终结果需要排序，每个Map节点数据处理好后，为了避 免在每个Reduce节点本地排序完成后还需要进行全局排序，我们可以使用一个分区策略 如:`(d%R)`，d为数据大小，R为Reduce节点的个数，则可根据数据的大小将其划分到指定 数据范围的Reduce节点上，每个Reduce将本地数据排好序后即为最终结果

### 分布式文件系统 GFS 的工作原理

Google GFS **的基本设计原则**
**GFS**将整个数据形成逻辑上整体的文件，尽管数据存储在物理上分布的每个节点上。

- 廉价本地磁盘分布存储
	- 各节点本地分布式存储数据。
- **多数据自动备份**解决可靠性
	- 采用廉价的普通磁盘，将磁盘数据出错视为常态，用 **自动多数据备份** 存储解决数据存储可靠性问题
- **为上层的 MapReduce**计算框架提供支撑


![image.png|500](https://kold.oss-cn-shanghai.aliyuncs.com/20250911152621.png)


#### 基本架构
- GFS **Master**
	- Master 存储了*三种元的基本架构数据*
		- Name Space 命名空间。存储整个分布式文件系统的目录结构
		- Chunk->Filename 映射表
		- Chunk copy 的位置信息。每一个 Chunk 有 3 个副本
- GFS ChunkServer
	- 用来保存大量实际数据的数据服务器
	- GFS中每个数据块划分缺省为64MB 
	- 每个数据块会分别在3个(缺省情况下)不同的地方复制副本；
	-  对每一个数据块，仅当3个副本都更新成功时，才认为数据保存成功。 
	- 当某个副本失效时，Master会自动将正确的副本数据进行复制以保证足够的副本数； 
	- GFS上存储的数据块副本，在物理上以一个本地的Linux操作系统的文件形式存储，每一个数据块再划分为64KB的子块，每个子块有一个32位的校验和，读数据时会检查校验和以保证使用未失效的数据

**具体来说**，GFS 访问具体数据不需要经过 GFS Master。
### BigTable

事实上不是一个数据库系统
- BACKGROUND
	- GFS is distributed file system, it's hard to store/visit **struct data**
	- Column family is the unit of access **control**
- Purpose:
	- store multiple type of data
	- really **busy** requests


#### models
- BigTable主要是一个分布式多维表，表中的数据通过：
	-  一个行关键字（row key） 
	- 一个列关键字（column key） 
	- 一个时间戳（timestamp） 进行索引和查询定位的。
- BigTable对存储在表中的数据不做任何解释，一律视为字节串，具体数据结构的实现由用户自行定义。
- BigTable查询模型 
	-  (row: string, column: string,time:int64)->结果数据字节串 
	- 支持查询、插入和删除操作
![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20250911154759.png)
- BigTable **数据存储格式**
	
- 数据模型
	- BigTable 主要是一个分布式多维表，表中的数据通过：
	-  一个行关键字（row key） 
	- 一个列关键字（column key） 
	- 一个时间戳（timestamp） 
	进行索引和查询定位
- 数据存储格式
	- 行 (row) 大小不超过 `64kb` 的任意字符串。 sorted by row keyword
	- 子表（Tablet) 水平方向分为多个小表
	- 列 (Column) 将列关键字组织为列族。每个列族中的数据属于同一类别。
	- 时间戳 (time stamp) 一个 URL 网页可能不断更新，Google 保存时间戳来区分不同时间的网页数据。
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250915151635.png)
	- 单元： Cell: the storage referenced by a particular row key, column key, and time stamp

## L5 - Hadoop
- Hadoop 是 Apache 软件基金会旗下的一个开源分布式计算平台
- 基于 Java 语言
- 核心：**HDFS**
- **行业大数据标准开源软件**


- DataNode
- 
 - HDFS **通信协议**
	 - HDFS is based on `TCP/IP`

- HDFS client （客户端）
	- 是一个库。暴露了部分 HDFS 文件接口
	- Java API
- **数据存储策略**
	- **第一个副本**：放置在上传文件的数据节点
	- **第二个副本**：放置在与第一个副本不同的机架的节点上
	- **第三个副本**：与第一个副本相同机架的其他节点上
	- **more**: random nodes

- HDFS read 读过程
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250915154245.png)
```java
import java.io.BufferedReader;
import java.io.InputStreamReader;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.fs.FSDataInputStream;

public
class ReadHdfsFile {
public
  static void main(String[] args) {
    try {
      Configuration conf = new Configuration();
 conf.set(“fs.defaultFS”,“hdfs://localhost:9000”);

	  conf.set("fs.hdfs.impl","org.apache.hadoop.hdfs.DistributedFileSystem");
 FileSystem fs = FileSystem.get(conf);
 Path file = new Path("test");
 FSDataInputStream getIt = fs.open(file);
 BufferedReader d = new BufferedReader(new InputStreamReader(getIt));
 String content = d.readLine();
 //读取文件一行 System.out.println(content);
 d.close();
 //关闭文件 fs.close();
 //关闭hdfs } catch (Exception e) { e.printStackTrace();
    }
  }
}
```


## L6
- HDFS **可靠性和出错恢复**
	- DataNode **check**
		- 心跳：` NameNode` check if **DataNote** is valid
		- if **invalid**, find a new node. re distribute the invalid node data
	- 数据一致性 Consistency
		- `checksum`
	- NameNode 元数据失效
		- Multiple Fslmage and Editlog
		- Checkpoint

- HDFS HA (High Availability) **解决单点故障问题**
	- HA:
		- 2 NameNode
			- Active
			- Standby
		- 状态同步两个节点。-> shared storage system
	- One NameNode crack, immediately switch to **standby** node
	- Zookeeper:
		- one NameNode is providing **service**
	- NameNode maintain the mapping info.
	- DataNode report the info for 2 NameNode
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250918152453.png)


- **Classic MapReduce Frame**
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250918152517.png)
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250918152530.png)
	- Job tracker is very **busy**
		- Assign task trackers
		- Coordinate map and reduce phases
		- Provide job progress info
- **Job tracker** is facing great **PRESSURE!!**

- Hadoop
	- 1.0
		- MapReduce
			- Resource Management
			- Data Processing
		- HDFS
			- Distributed File Storage
	- 2.0
		- Independent YARN (Yet Another Resource Negotiator)
		- 把 Resource Management (**RM**) 独立出来
			- **RM**全局管理所有应用程序计算资源的分配
			- **AM**负责相应的调度和协调
	- **一个应用程序**无非是 `DAG` or `排序` 等工作。
		- **YARN**架构思路：将 Job Tracker 三大功能拆分。
			- One Job Tracker, Three Function
				- Schedules Job submitted by clients
				- Keep track of live TaskTrackers and available map and reduce slots
				- Monitors jobs and tasks execution on the cluster
	- ![image.png|500](https://kold.oss-cn-shanghai.aliyuncs.com/20250918151904.png)
	- **YARN**的 MapReduce 架构
		- ![image.png|500](https://kold.oss-cn-shanghai.aliyuncs.com/20250918153349.png)


	- **YARN**
		- **一个集群多个框架** One Cluster Multi Frame
		- 由 YARN 为这些计算框架提供统一的资源调度管理服务，并且能够根据各种计算框架的负载需求，调整各自占用的资源，实现集群资源共享和资源弹性收缩。可以实现一个集群上的不同应用负载混搭，有效提高了集群的利用率。不同计算框架可以共享底层存储，避免了数据集跨集群移动。
		- **更高的集群利用率**
		- 新的 YARN：加入 `ApplicationMaster`，是一个可变更的部分。用户可以通过自己的编程模型编写自己的 `ApplicaitonMaster`，让更多的编程模型运行在 hadoop 集群了。
		- `JobTracker` 的很大负担（监控 Job 的 Tasks 运行情况）被下放到 `ApplicationMaster` 中
	- 


	- Hadoop MapReduce Working Process
	- ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20250918153035.png)
	- 

## L7

- you may rely on this to start your hadoop
```bash
yarn jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar wordcount /user/wordcount_input /user/wordcount_output
```


- aliyun source_list
```
deb http://mirrors.aliyun.com/ubuntu/ xenial main
deb-src http://mirrors.aliyun.com/ubuntu/ xenial main

deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main

deb http://mirrors.aliyun.com/ubuntu/ xenial universe
deb-src http://mirrors.aliyun.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates universe

deb http://mirrors.aliyun.com/ubuntu/ xenial-security main
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main
deb http://mirrors.aliyun.com/ubuntu/ xenial-security universe
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security universe
```


[[FBDP-lab1]]

[[FBDP-4]]


### MapReduce 来做一些经典算法的并行版本

#### 频繁项集

- `m` 个项的集合：$I=\{ I_{1},I_{2},\dots,I_{m} \}$
- `n` 个事务的数据库：$D=\{ T_{1},T_{2},.,..,T_{n} \}$，其中 $T_{i}$ 是 $I$ 的非空子集


**频繁项集挖掘**：将所有满足



## NoSQL

**一个事务数据库的性质**
- **A**
- **C**
- **I**
- **D**
**原子性（atomicity）、一致性 (consistency)、隔离性 (isolation)和持久性 (durability)**。


- **NoSQL** = **Not Only SQL**
	- 泛指非关系型数据库。
	- 放松了 ACID 事务处理特征和数据高度结构化的要求，简化设计，提高数据存储管理的灵活性，提高处理性能，支持良好的水平扩展。
- Why NoSQL raise
	- `One size fit all` model is hard to fit different situation
	- relation model, as a unified data model, used in data analysis and online business. But, one means high take-in-out, one imply low delay. The construct varies, one model to abstract is not enough.

**CAP 定理**
一个分布式系统不可能同时很好的满足
- **一致性**
- **可用性**
- **分区容错性**
需要取舍。

**最终一致性**



### HBase

- constructed on **HDFS**
- provide struct-ed or half-struct-ed visit method to HDFS

- Zoo **keeper**
	- 分布式协调服务器
	- at any time, the cluster only have **one** HBase Master
	- observe the state of ` region server`
	- store HBase entrance

- 可与 MapReduce 协同工作


## Hive

- Group by
	- 通常和聚合函数一起用

- **表的分桶**
	- **分桶是相对于分区进行更细粒度的划分**
	- 在分区数量过于庞大以至于可能导致文件系统崩溃时，就需要使用分桶来解决问题
	- 分桶将整个数据内容按照某列属性值的 `hash` 值进行群峰。
	- **分桶**同样应当在建表的时候就建立
```
CLUSTERED BY (Id) into 3 buckets
```


## Spark
Hadoop MapReduce 暴露了一些问题
- Berkley AMP 实验室 2009
- 通用内存并行计算框架
- 2010: open source

- Spark
	- Spark SQL
	- Spark Streaming
	- MLlib
	- GraphX
为什么会有 Spark？
- Resilient Distributed Datasets (RDDs)
	- **弹性分布式数据集** Resilient Distributed Datasets (RDDs)
	- 基于 RDD 之间的弹性关系


- Spark 调度器
	- 主要由两种
		- DAG Scheduler
		- Task Scheduler
	- DAG Scheduler: divide a Job into multiple Stage, based on rely-relation between RDDs
	- 将 Stage 抽象为任务集 (`TaskSet`) 交给 `TaskScheduler` 进行进一步调度

- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201141219.png)

-  ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201141208.png)
- Task Scheduler:
	- Task Scheduler 为每一个 TaskSet 进行任务调度。
	- Spark 任务调度
		- FIFO (FIrst-In-First-Out)
		- FAIR


- **执行过程**
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201141319.png)
	- 从 RDD 的转换和存储角度来看
		- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201141342.png)
		- 
	- 一个作业就是一张 RDD 世系 (Lineage) 图
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251201140944.png)

### 技术特点
- **技术特点**
	- RDD：弹性分布式数据集
	- Transformation & Action: Spark 通过 RDD 的两种不同类型的运算实现了惰性计算。
	- Lineage：通过血统关系 Lineage 记录一个 RDD 如果通过其他 RDD 转换过来。保证可以根据父系从新计算，鲁棒性 up
	- Spark 调度：事件驱动的 Scala 库 Akka 完成。复用线程池取代 MapReduce 进程或者线程启动和切换的开销
- API: Scala API, also Java, Python
- Spark 生态
	- Spark SQL
	- Spark Streaming
	- GrpahX
- Spark 部署
	- Standalone, YARN, K8S
- **适合**需要**多次操作特定数据集**的应用场合。
- **不适合**异步细粒度更新状态的应用



Spark是一种为大规模数据处理而设计的快速通用的 分布式计算引擎，适合于完成一些迭代式、关系查询、流式处理 等计算密集型任务





## 顾荣老师讲座 - Alluxio
高速跨平台大数据存储系统  Alluxio

All : 跨平台
lux：the unit of  illuminance
io: input & output


Alluxio **是世界上第一个** 以内存为中心的 (memory-centric ) 的虚拟的分布式存储系统。
Alluxio 介于计算框架和现有的存储系统之间

**起名字很重要**（前身 tachiyon）

系统框架和原理
- 整体架构
	- Master
		- manage all meta data
	- Worker
		- manage local memory, ssd and hdd

- Alluxio：文件数据按**块**处理（block）
	- file & block store in master
- Alluxio: 读写行为
	- 读写类型控制
	- ReadType, WriteType
	- explicitly 控制读写类型

- Alluxio 
	-  透明命名机制
	- 统一命名空间
		- 能够将多个数据源中的数据，挂载到 Alluxio 中
		- 类似于操作系统的不同磁盘管理
