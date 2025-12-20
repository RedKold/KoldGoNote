---
本科课程: FBDP
date: 2025-12-04
author: 231275036-朱晗
completed: "true"
---

作业清单
1. 简述Spark的技术特点

2. 简述Spark的基本构架组成

3. 简述Spark RDD的调度过程

4. 简述Spark的程序执行过程

### 1 . 简述Spark的技术特点
- **技术特点**
	- RDD：**弹性分布式数据集**： 最核心的数据抽象
	- Transformation & Action: Spark 通过 RDD 的两种不同类型的运算实现了惰性计算。
	- Lineage：通过血统关系 Lineage 记录一个 RDD 如果通过其他 RDD 转换过来。保证可以根据父系从新计算，**鲁棒性提升**
	- Spark 调度：事件驱动的 Scala 库 Akka 完成。复用线程池取代 MapReduce 进程或者线程启动和切换的开销
- API: Scala API, also Java, Python 等的支持。
- Spark 生态
	- Spark SQL
	- Spark Streaming
	- GrpahX
	- 适合应用不同的计算模式和计算任务
- Spark 部署
	- Standalone, YARN, K8S
	- 可以部署在多种底层平台上
- **适合**需要**多次操作特定数据集**的应用场合。
- **不适合**异步细粒度更新状态的应用



Spark 是一种为大规模数据处理而设计的快速通用的分布式计算引擎，适合于完成一些迭代式、关系查询、流式处理等计算密集型任务

### 2. 简述 Spark 的基本构架组成

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251204113450.png)
- **Application**: 基于 Spark 的用户程序
	- 由集群上的一个驱动 (Driver) 程序和多个执行器 (Executor) 组成。应用程序入口为用户定义的 main 方法
- **SparkContext**：Spark 所有功能的主要入口点。
	- 用户逻辑和 Spark 集群的主要的交互接口。通过 SparkContext 连接到 ClusterManager, 交互 Master 节点
	- 也能将应用程序用到的 JAR 包或 Python 文件发送到多个执行器 (Executor) 节点上
- **Cluster Manager**: 集群管理器
	- 存在于 Master 进程中，主要用来对应用程序申请的资源进行管理
- **Worker Node**
	- Any note can be executed in Spark cluster
	- **最重要角色**
- **Task**:
	- SparkContext发送到 Executor 节点执行的一个工作单元
- **Driver**
	- **驱动器节点**。运行 Application 的，由 `main()` 创建 SparkContext 的进程。Driver 节点页负责提交 Job，并将 Job 转化为 Task，在各个 Executor 进程中协调 Task 的调度。Driver 节点可以不运行于集群节点机器上
	- **最重要角色**：执行的起点，负责调度
- **Executor**
	- 执行器节点。在 Work Node 上位 Application 启动的进程。可以运行 Task 并将数据保存在内存或磁盘存储，也能将结果返回给 Driver

![](https://kold.oss-cn-shanghai.aliyuncs.com/20251204114930.png)


### 3. 简述 Spark RDD 的调度过程
- **调度过程**
	- 创建 RDD 生成 DAG，由 DAGScheduler 分解 DAG 为包含多个 Task (that is TaskSet) 的 Stages
	- 再将 TaskSet 发送至 TaskScheduler
	- TaskScheduler 调度每个 Task，分配到 Worker 节点上执行
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251204122318.png)
- Spark 调度器
	- 主要两种
		- DAG Scheduler
		- Task Scheduler
	- DAG Scheduler: 
		- 执行 Action 操作的时候，根据 RDD 的依赖关系将一个 Job 分解位多个 Stage。每个 Stage 包含多个 Task，每个 Task 处理 RDD 中的一个 Partition
		- 将 Stage 抽象为任务集 (`TaskSet`) 交给 `TaskScheduler` 进行进一步调度
	- TaskScheduler:
		- FIFO
			- First-In-First-Out
		- FAIR
			- 任务分组到池，每个池不同的调度权重

### 4. 简述 Spark 的程序执行过程

- **程序提交**：用户编写的Spark程序提交到相应的Spark运行框架中
- **创建SparkContext**：Spark程序启动时，创建一个SparkContext对象为本次程序的运行环境。
- **集群资源连接**：SparkContext会与集群管理器进行通信，确定程序的资源配置使用情况
- **获取Executor节点**：连接集群资源节点成功后，SparkContext会在集群中处于**活动**并且**可用状态**的节点上启动Executor进程 (Spark 准备运行你的程序并且确定数据存储)
- **代码分发**：Spark 分发程序代码到各个节点
- **任务执行**：最终，SparkContext 发送 tasks到不同的Executor执行。

从 RDD 的转换和存储角度来看：
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251204123509.png)

一个作业就是一张 RDD  Lineage 图，也是一个 DAG 图。可以追踪数据的父系关系

- Job 和 Stage 是针对一个 RDD 执行过程的划分
- Tasks 具体到了 RDD 中每个分区的执行

