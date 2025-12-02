## 作业清单
1. 简述Hive和RDBMS的区别  

2. 简述Hive和HBase的区别

3. 简述Hive的体系结构和各组成模块的作用

4. 简述Hive的数据模型


### 简述 Hive 和 RDBMS 的区别

- Hive 主要用于批量数据分析处理（OLAP）。RDBMS 主要用于事务型数据处理（OLTP）
- Hive 使用的是 Hadoop 的 HDFS，容易拓展自己的存储能力和计算能力；RDBMS 用的是服务器本地的文件系统
- RDBMS 使用**标准的 SQL 查询语言**，查询通常是**在单机或者集群模式**下由 SQL 引擎直接执行，查询响应速度快；Hive 提供和 SQL 类似的查询语言 HiveQL，但底层会将查询翻译成 **MapReduce、Tez 或 Spark** 作业在**分布式环境**下执行，适合高延迟的批处理分析。

- Hive 的数据加载模式是**读时模式**（快），RDBMS 是写时模式（慢）
- Hive 数据插入支持批量导入/单条插入，RDBMS 支持单条或者批量导入


### 2. 简述 Hive 和 HBase 的区别
HBase 主要解决实时数据查询问题，Hive 主要解决数据处理和计算问题
- HBase 主要针对 OLTP 应用。实时性 随机访问
- Hive 主要针对 OLAP 应用。批处理，离线数据分析

- 从数据存储和结构来看
	- HBase：基于 Hadoop 的 NoSQL 数据库，键值对存储数据。列族为单位进行存储，支持稀疏数据模型，擅长处理半结构化或非结构化的数据。 **实时访问、数据存储在 HDFS**中并以列为单位存储
	- Hive：基于 Hadoop 的数据仓库工具，支持结构化数据存储，数据以表的形式存储在 HDFS 中。Hive 的数据可以是 CSV, Parquet、ORC 等格式，通常以批量方式写入和读取，适合大规模数据分析
- 查询语言和访问方式
	- HBase：不支持 SQL 语法，采用基于 Java 的 API 进行数据操作。数据查询方式更接近 NoSQL 数据库，通过键值对或行键查找数据，支持快速随机读写
	- Hive：提供类似 SQL 的查询语言 HiveQL，允许用户使用 SQL 语法进行查询。但是 Hive 会转化为 MapReduce、Tez 或者 Spark 执行。
- 实时访问和延迟
	- HBase：低延迟的随机读写访问。适合实时处理和快速查询
	- Hive：高延迟的数据批处理。适合离线数据分析，尤其是处理大量历史数据或者生成定期报表。**类似于一个数据仓库**
- 事务支持
	- HBase：不支持传统意义的 ACID 事务，但提供基本的一致性支持，允许行级的原子操作，适合简单的插入和更新操作
	- Hive：从 3. x 开始支持基本的 ACID。主要用于数据仓库和批处理，通常不用于实时写入和更新
- 数据模型和索引
	- HBase: column family 列族 - key-value 键值对模型。大规模表存储。
		- 没有传统的索引结构，主要依赖于行键和列键
	- Hive：传统关系型数据库的表、行、列的数据模型，支持简单索引。可以通过分区和分桶加速查询。适合结构化数据分析，不适合频繁的小规模更新和插入操作
- 执行引擎和计算模型
	- HBase：没有特定的计算引擎，它的重点在于快速的数据存取，主要通过直接访问API 操作数据，而不涉及MapReduce 等批处理框架
	- Hive: Hive 本身不存储数据，主要作为数据查询和分析工具，将查询翻译成MapReduce、Tez 或Spark 作业并在分布式计算集群中执行，适合大规模的批量计算和数据分析任务。

### 3. 简述 Hive 的体系结构和各组成模块的作用
- 体系结构图：
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251124172012.png)
- Hive 是构建在整个 hadoop 之上的，主要由 Driver（Complier、 Optimizer、Executor）、Metastore、客户端（CLI、HWI、ThriftServers（JDBC、ODBC））组成
- **Driver**：Hive 的核心。包括 Compiler, Optimizer, 和 Executor
	- 解析 HQL 预计、编译优化、生成执行计划、然后调用 MapReduce 计算框架
- **Metastore**组件：数据服务组件，用以存储 Hive 的元数据：存储操作的数据对象的格式信息，在 HDFS 的存储位置信息以及其他的元数据。
- **CLI**：Command Line Interface 命令行接口
- **ThriftServers**
	- 提供 JDBC 和 ODBC 接入的能力，用来进行可拓展且跨语言的服务的开发，Hive 集成了该服务。
- **Hive WEB Interface(HWI)**：Hive 客户端提供了通过网页的方式访问 hive 所提供的服务。这个接口对应 hive 的 HWI。

- **HiveQL**：Hive 的数据查询语言，类似于 SQL。提供了这个数据查询语言与用户接口，包括一个 shell 的接口。


### 4. 简述 Hive 的数据模型

- Hive 的数据模型是一个 `Table -> Partitions -> Buckets` 的多级结构
	 - ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251124173236.png)
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251124173253.png)

- Tables: Hive 的数据模型由数据表组成。数据表中的列有类型。(int, float, string...) 也可以是复合的类型
	- 表和关系数据库的表类似。表的元数据描述了数据的布局。可以对表执行过滤、关联、合并等操作
- Partitions：数据表可以按照一定的规则进行划分 Partition
	- 为了提高效率。Hive 提供了表分区机制。分区表基于分区键把具有相同分区键的数据存储在一个目录下。查询某一个分区的数据的时候，只需要查询对应目录下的数据。
- Buckets：数据存储的桶。在一定范围内的数据按照 Hash 的方式进行划分。
	- Hive 可以对每一个表或者是分区，进一步组织成桶。
	- 桶是针对表的某一列进行分桶，Hive 采用对表的列值进行 hash 计算，然后除以桶的个数求余的方法决定该条记录放在哪个桶中。分桶的好处是可以获得更高的查询处理效率，使取样更高效。
	- 每个桶只是表目录或者分区目录下的一个文件
- 元数据存储: metastore
	- 包括服务和后台的数据存储‘
	- Embedded metastore: 默认使用内嵌的 Derby 数据库实例
	- Local metastore: 可以使用运行在一个进程中的 metastore 的进程来访问独立的数据库，可通过 JDBC 进行设置具体的数据库访问
	- Remote metastore: 一个或者多个 metastore 服务器和 Hive 服务运行在不同的进程内


![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251124173718.png)



