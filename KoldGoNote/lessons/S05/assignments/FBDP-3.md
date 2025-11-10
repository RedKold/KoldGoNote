姓名：朱晗
学号：`231275036`
### 1. MapReduce提供了一个统一的计算框架，请简述该框架的主要功能。
Map 和 Reduce 两个抽象的接口
`map: (k1,v1) -> [(k2;v2)]`
- input:
	- key-value pair data
- process:
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

### 2 . 简述Google MapReduce并行处理的基本过程。
Google MapReduce 的并行处理大概可以按这个流程图来理解：
![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20250911144518.png)
基本过程：
- User Program fork 到各 worker 节点和 master 节点，数据 split 成块
- master 节点根据资源调度情况，将任务分发给 map 节点。同时寻找配备 reduce 的节点
- map 节点对数据进行进行处理，并进行数据整理工作 (sorting? combining?)。告知 master 完成。（`local write`）
- 在 map 节点完成后，master 调度 reduce 来汇总结果。`remote read`
- Reduce 节点计算结果汇总输出到一个结果文件，**即获得整个处理结果** 

### 3 . 简述GFS的基本架构和数据访问过程。
- **GFS**的基本架构：
	- ![image.png|500](https://kold.oss-cn-shanghai.aliyuncs.com/20250911152621.png)

- 三个核心部件：
	- `GFS Master`
		- 保存三种元数据
		- Name space
		- 映射表 (Chunk -> filename)
		- chunk 副本的位置信息。
	- `GFS chunkserver`
		- 保存大量实际数据的服务器
		- 每个数据块会在 3 个不同的地方复制副本。
	- `GFS client`

- **数据访问过程**
	- Before program run, data has been store in **GFS** file system. When program run, the **application** (known as GFS client) will tell GFS server filename or chunk index which he want to visit.
	- GFS server will search and locate the file or chunk by filename or index, and find **Chunk Server** who store these data. then **return location info** back to application
	- **App** directly visit ChunkServer based on location information provided by **GFS Server**
	- App get the data chunks, then start compute process.
- **并发访问**。
- 具体数据不经过 GFS Master
### 4 . 简述BigTable的数据模型和数据存储格式。

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
