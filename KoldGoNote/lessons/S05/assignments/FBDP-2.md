姓名：朱晗
学号：`231275036`

## 1. 简述并行和并发的区别
并行 (Parallelism) 是同一物理时间时刻，有多个任务（task）在同时执行。

并发 (Concurrency) 指的是同一时间端内，有多个任务都在进行中，但是不一定同一时刻执行着。并发任务共享同一个处理单元，切换速度很快。事实上每一个微小瞬间只有一个任务在被处理。

## 2. 并行计算按照系统类型划分，可以分为哪几种？简述每一种系统类型的特点。

按系统类型：
- 多核/众核并行计算系统 MC
- 对称多处理系统 SWP
	- 多个相同处理器类型通过总线 bus 连接并共享存储器
- 大规模并行处理 MPP (Massive Parallel Processing)
	- 专用**内联网**连接一组处理器形成的一个计算系统
- 集群 Cluster
	- 网络连接的**一组商品计算机构成的计算系统**
- 网格 Grid
	- 用网格连接**远距离**分布的一组异构计算机构成的计算系统
- 云 Cloud
	- 通过互联网按需访问计算资源

## 3. MPI点对点通信和节点集合通信分别提供哪几类接口？
- **点对点**
	- 同步通信：阻塞
		- MPI_Send
		- MPI_Recv
	- 异步通信
		- MPI_ISend
		- MPI_IRecv
		- MPI_Wait
		- MPI_Test
- **节点集合通信**
	- 同步 Barrier
		- MPI_Barrier
	- 数据移动 Data movement
		- MPI_BCAST
		- MPI_GATHER
		- MPI_SCATTER
	- 数据规约
		- MPI_Reduce
		- 其内部也有不少别的 interface。
## 4. 尝试安装并运行MPICH，编译examples里自带的cpi.c代码并执行，给出运行结果截图。
![image.png|500](https://kold.oss-cn-shanghai.aliyuncs.com/20250911203655.png)
