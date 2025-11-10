### 简述HDFS中NameNode 、DataNode和SecondaryNamedNode的作用


| Number    | Name |
| --------- | ---- |
| 231275306 | 朱晗   |

#### NameNode

- NameNode 类似于 HDFS 集群的“管理者”。核心作用是**存储和管理文件系统的元数据**
- NameNode负责管理分布式文件系统的命名空间（Namespace），文件系统的元数据以两个核心的数据结构，即 `FsImage` 和 `EditLog` 的形式存储
	- `FsImage`: (File System Image) 存储文件系统元数据的完整快照 (**Snapshot**)
	- 它记录了 NameNode **在某一特定时间点**的文件系统的所有状态信息，包括：
		- 完整的目录树结构。
		- 文件的所有者、权限、副本数。
		- 每个文件对应的数据块 ID 列表。
	- `EditLog`
		- 记录 HDFS 所有操作事务的日志文件。
		- 实时更新。
		- 记录了从上次 `FsImage` 快照生成以来的所有更改
	- 二者共同负责文件系统元数据的管理。`FsImage` 完整但是大小太大，`EditLog` 灵活。体现了 HDFS 的设计智慧
#### DataNode
- DataNode 类似一个“工人”节点。负责数据的存储和检索。

- **数据存储**
    - DataNode负责存储来自HDFS文件的**数据块（data blocks）**。
    - Hadoop会将每个大文件切分成多个小的数据块，并将这些数据块分布到不同的DataNode上进行存储。
        
- **读写服务**
    - DataNode直接与客户端（比如运行MapReduce任务的程序）进行数据读写交互。
    - 当客户端需要读取一个文件时，**NameNode**（集群的管理者）会告诉它所需数据块在哪个DataNode上，然后客户端直接与该DataNode通信，读取数据。
    - 当客户端需要写入一个文件时，DataNode会根据NameNode的指令接收并存储数据块。
        
- **心跳报告与健康检查**
    - 每个DataNode都会定期（默认每3秒）向NameNode发送“心跳”信号。
    - 这个心跳信号是DataNode向NameNode报告自己还活着、运行正常的方式。
    - 心跳信号中还包含了该DataNode上所有已存储的数据块列表，这帮助NameNode更新和维护其元数据，确保数据块的可用性和冗余性。
        
- **数据块复制与故障恢复**
    - 如果一个DataNode上的数据块副本损坏或丢失（例如，因为磁盘故障或网络问题），DataNode会立即通知NameNode。
    - NameNode会启动复制过程，从其他健康的DataNode上复制一个数据块，以满足文件的冗余要求，确保数据不会丢失。

#### SecondaryNameNode
- SecondaryNameNode 是一个辅助性的守护进程。负责 `Checkpointing` 操作。
- 具体而言，是：1. 镜像备份；2. 日志与镜像的定期合并。
- 回想，NameNode 启动时候，它首先加载 FsImage 到内存，然后重放 EditLog 的操作来更新状态。随着集群运行，EditLog 越来越大。而 SecondaryNameNode 通过周期性检查点，来合并 EditLog 到 FsImage 来保持 EditLog 大小可控。

### 简述HDFS是如何应对NameNode 、DataNode和数据出错的
- **NameNode 出错**
	- HDFS 实现了备份机制，即核心元数据文件 `FsImage` 和 `EditLog` 定期备份到服务器 SecondaryNameNode
	- 当 NameNode 出错，可以通过 SecondaryNameNode 来回复
- **DataNode 出错**
	- DataNode 会定期发送“心跳”message 给 NameNode
	- 如果 DataNode 发生故障，NameNode 侦测到某些 DataNode 没有及时发送心跳信息，就标记它们为宕机。NameNode 会尝试指示其他健康的 DataNode 之间相互复制。将受影响的数据块创建新的副本。
- **数据出错**
	- HDFS 在 DataNode **存储数据块的时候也存储的校验和**
	- 客户端读取数据进行校验和验证，如果验证失败，客户端报告 NameNode
	- NameNode 尝试从健康的 DataNode 复制数据。

### 简述YARN的设计思路和作用

- YARN 的设计思路
	- 将原有的 JobTracker 的三大功能拆分。
	- **资源管理**和**任务调度/执行**解耦，形成一个通用的分布式资源管理平台。
	- **核心设计**
		- **统一资源管理**：将集群所有机器的 CPU、内存抽象，由 `ResourceManager` 进行管理和调度
		- **应用自治调度**：运行 `ApplicationMaster`
		- **节点级资源代理**：每台机器运行 `NodeManager`，处理来自 `ResourceManager` 和 `ApplicationMaster` 的命令
		- **容器化执行环境**：任务运行在容器中。容器由 `ResourceManager` 分配

- 作用：
	- **资源统一管理**：支持多种计算框架（MapReduce、Spark、Tez等）在同一集群上共享资源。
	- **提高扩展性**：通过AM机制，让不同类型的应用可以自定义调度和执行逻辑，增强系统通用性。
	- **提升集群利用率**：容器化资源分配避免了资源浪费，提高整体吞吐量。
	- **增强容错能力**：RM和AM相互配合，能对失败任务和节点进行恢复。

**总体而言**，YARN 作为 Hadoop 2.0 的更新，使得其从 MapReduce 引擎升级为通用的计算处理平台。更多的编程模型可以运行在同一个 Hadoop 集群中