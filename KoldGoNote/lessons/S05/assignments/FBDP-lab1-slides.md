本 Slides 基于 Obsidian 幻灯片插件制作，需要用 Obsidian 放映。
## 实验概况

---

- 设备：`Macbook Pro M4, ram: 24g`
- `docker` + `ubuntu 22.04`
- `hadoop-3.4.2-aarch64` 
- 此报告已经发布在我的个人博客，[金融大数据处理技术(FBDP) lab1 日志 | RedKold的小站](https://redkold.github.io/2025/10/10/FBDP-lab1/)，欢迎评论指正问题。

---
## 任务 1
---


### 环境准备
---

我们先用 `docker` 部署一个单机伪分布模式。
- 在终端 (我使用 `iTerm2 -zsh`) 输入 ` docker exec -it <container_id> /bin/bash ` 进入容器
- `sudo apt install` 安装 `vim` 以修改设置
- `docker cp` 命令将宿主机 `Mac` 下载好的 `Hadoop` 文件传输，解压，配置环境变量。
	- `vi ~/.bashrc`
---

```bash
export HADOOP_HOME=/usr/local/hadoop
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export JAVA_HOME=/usr/bin/java
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```
- `source ~/.bashrc`
---

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251009201548.png)
- 该图片说明 hadoop 已经配置完毕环境。
---

### 配置 Hadoop
该配置发生在 `etc/hadoop` 目录。参照老师的讲义。

---

对于各文件的意义，可参考：
- `core-site.xml`：核心配置文件，定义了集群是分布式，还是本机运行
- `hdfs-site.xml`：分布式文件系统的核心配置，决定了数据存放路径，数据的副本，数据的block块大小等等
- `mapred-site.xml`：定义了mapreduce运行的一些参数
- `yarn-site.xml`：定义yarn集群
- `workers`：定义了从节点是哪些机器datanode，nodemanager运行在哪些机器上
- `hadoop-env.sh`：配置jdk的home路径

---

- `hadoop-env.sh`
	- 配置 `export JAVA_HOME`
---

- `core-site.xml`
	- 配置 HDFS 的 `NameNode` 地址端口
```xml
<configuration>
    <property>
        <name>fs.default.name</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/hadoop3/hadoop/tmp</value>
    </property>
</configuration>
```
---
- `hdfs-site.xml`
	- 配置 HDFS 副本数量和数据存储目录
```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/home/hadoop3/hadoop/hdfs/name</value>
    </property>
    <property>
        <name>dfs.namenode.data.dir</name>
        <value>/home/hadoop3/hadoop/hdfs/data</value>
    </property>
</configuration>
```
---
- `mapred-site.xml`
	- 配置 MapReduce 框架，使用 yarn
---
- `yarn-site.xml`
	- **重要**：配置 Yarn 的 `ResourceManager` 地址
```xml
<configuration>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>master</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```
---
- `workers`
	- 指定主机名（单机伪分布填写本机）
---
### 配置无密码登录 SSH
---

首先安装 `ssh` 服务
- `ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa`
- 然后复制该密钥 `cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys `
---
### 开始测试
---
- 启动 HDFS 服务
```bash
root@fcda3e2ea909:/usr/local/hadoop# start-dfs.sh
Starting namenodes on [localhost]
Starting datanodes
Starting secondary namenodes [fcda3e2ea909]
```
---
- 启动 YARN 服务
```bash
start-yarn.sh
Starting resourcemanager
Starting nodemanagers
```
---
- `jps` 命令查看进程是否都启动
```
root@fcda3e2ea909:/usr/local/hadoop# jps
595 NameNode
1171 ResourceManager
939 SecondaryNameNode
748 DataNode
1485 NodeManager
1662 Jps
```
---
- 使用 `docker cp` 复制目标 `example.txt` 文件到 `docker` 主体上 
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251010110853.png)
- 然后，上传该文件到 HDFS
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251010111238.png)
---

#### 实验数据说明
---
本试验运行 `wordcount` ，输入为一段 `txt` 文本，其内容为：

```
Hadoop hadoop mapreduce zookeeper
Zookeeper HBase HBase hive spark
hadoop Spark Hive MapReduce mapreduce
Hive Spark spark zookeeper hadoop MapReduce
```
---

#### 运行程序
---
```
yarn jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar wordcount /user/lab1/input /user/lab1/output
```
---

输入后，机器开始处理。
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251010112016.png)

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251010112453.png)

---

##### **Job Counters（作业总体资源使用）**
---
-  `Launched map tasks=1 Launched reduce tasks=1 Data-local map tasks=1`
	- 任务共启动了一个 Map、一个 Reduce，都运行在数据本地节点（data-local），性能最佳。
-  `Total time spent by all map tasks (ms)=836 Total vcore-milliseconds taken by all map tasks=836`
	- Map 阶段用时约 836ms，Reduce 阶段用时约 949ms。说明任务轻量。

---

#####  **Map-Reduce Framework（任务统计）**

```
Map input records=4 Map output records=20 Combine input records=20 Combine output records=11 Reduce output records=11

Reduce shuffle bytes=154

Shuffle 阶段传输了 154 字节数据（Map 输出传给 Reduce 的中间数据量）。
```

---

### 查看结果
---
 **查看 HDFS 输出目录的内容：**
```
hdfs dfs -ls /user/lab1/output
```
看到一个名为 `_SUCCESS` 的空文件和一个名为 `part-r-00000` (或类似名称) 的文件，后者包含计数结果。

我们下载到本地打开
- ![](https://kold.oss-cn-shanghai.aliyuncs.com/20251010114217.png)
---

- ![image.png|200](https://kold.oss-cn-shanghai.aliyuncs.com/20251010114235.png)

- 结果正常。
---
- 登录 web 界面查看 `mapreduce` 工作节点的结果
- ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251010141806.png)

---

## 任务 2 - docker 集群
---
### 环境准备
---
我们先配置一个 **伪集群**（在任务 1 实际已经完成） ，运行成功后，在其基础上用 `docker` 构建 ` hadoop cluster`

在 `docker` 上配置好 `hadoop` 相关设置
- 在 `core-site.xml` 中修改 `NameNode` 为 `h01`
- 修改 `workers`，制作 `h01 h02 h03 h04` 的分布集群。
```
(base) kold@zhuhandeMacBook-Pro ~ % sudo docker commit -m "hadoop cluster making" -a "hadoop" 4ff9cb430765b08ce8f3c088cc370a0f349b6f4f3ec0cd0de46b3e3aead010b5 hadoop-cluster-image:v1
Password:
sha256:13772772a99999a42cab659e25a13823897182fce0c84c0ab2991cec98dc4b8d
(base) kold@zhuhandeMacBook-Pro ~ %
```
---
配置好 `ubuntu hadoop` 镜像，`docker commit` 来配置集群，封装成 `images`，并导出。



---

- 使用如下命令配置 `docker` 端口映射并启动
```bash
docker run -it \
		--network hadoop \
		-h "h01" \
		--name "h01" \
		-p 9870:9870 \
		-p 8088:8088 \
		-p 19888:19888\
		hadoop-cluster-image:v2 /bin/bash

```
---
- 启动成功，进入 web UI 页面查看
![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20250925163208.png)

---
### 问题和解决问题
---
#### Port Conflict
---
- 端口冲突：
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251010150422.png)
---
- 解决：
	- 启动的时候，`-p [x]8088:8088`，其中 `[x]` 为 `h[x]` 的序号。
	- `--network hadoop` 的作用是创建一个 docker 的桥接网络 (Bridge), 由于我们在[[#配置无密码登录 SSH|第一部分的SSH配置]]中已经完成了相关配置，这里不需要额外配置 ssh 了。（各个容器共用一套密钥）
```bash
docker run -it --network hadoop -h "h02" --name "h02" -p 29870:9870 -p 28088:8088 hadoop-cluster-image:v2 /bin/bash
docker run -it --network hadoop -h "h03" --name "h03" -p 39870:9870 -p 38088:8088 hadoop-cluster-image:v2 /bin/bash
docker run -it --network hadoop -h "h04" --name "h04" -p 49870:9870 -p 48088:8088 hadoop-cluster-image:v2 /bin/bash
```
---
- 经测试，`h1` 可以通过 `ssh` 登录到其他节点
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251010153710.png)
---
#### 格式化
---
由于导出容器是测试过单机伪分布的，其保存了一些 DataNode 的旧集群的**元数据**。
```bash
rm -rf /home/hadoop3/hadoop/tmp/dfs/data/*
```
- 清空之后，重启主节点，解决了问题
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251010154740.png)
---
### 启动容器集群
![image.png|500](https://kold.oss-cn-shanghai.aliyuncs.com/20251010151002.png)

---
使用 `hadoop dfsadmin -report` 查看主节点报告信息，可见启动成功。

```
root@h01:/usr/local/hadoop# hadoop dfsadmin -report
WARNING: Use of this script to execute dfsadmin is deprecated.
WARNING: Attempting to execute replacement "hdfs dfsadmin" instead.

Configured Capacity: 1456421953536 (1.32 TB)
Present Capacity: 1313848233984 (1.19 TB)
DFS Remaining: 1313848160256 (1.19 TB)
DFS Used: 73728 (72 KB)
DFS Used%: 0.00%
Replicated Blocks:
	Under replicated blocks: 0
	Blocks with corrupt replicas: 0
	Missing blocks: 0
	Missing blocks (with replication factor 1): 0
	Low redundancy blocks with highest priority to recover: 0
	Pending deletion blocks: 0
Erasure Coded Block Groups:
	Low redundancy block groups: 0
	Block groups with corrupt internal blocks: 0
	Missing block groups: 0
	Low redundancy blocks with highest priority to recover: 0
	Pending deletion blocks: 0

-------------------------------------------------
Live datanodes (3):

Name: 172.21.0.3:9866 (h02.hadoop)
Hostname: h02
Decommission Status : Normal
Configured Capacity: 485473984512 (452.13 GB)
DFS Used: 24576 (24 KB)
Non DFS Used: 22788616192 (21.22 GB)
DFS Remaining: 437949386752 (407.87 GB)
DFS Used%: 0.00%
DFS Remaining%: 90.21%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 0
Last contact: Fri Oct 10 07:48:48 GMT 2025
Last Block Report: Fri Oct 10 07:47:00 GMT 2025
Num of Blocks: 0


Name: 172.21.0.4:9866 (h03.hadoop)
Hostname: h03
Decommission Status : Normal
Configured Capacity: 485473984512 (452.13 GB)
DFS Used: 24576 (24 KB)
Non DFS Used: 22788616192 (21.22 GB)
DFS Remaining: 437949386752 (407.87 GB)
DFS Used%: 0.00%
DFS Remaining%: 90.21%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 0
Last contact: Fri Oct 10 07:48:48 GMT 2025
Last Block Report: Fri Oct 10 07:47:00 GMT 2025
Num of Blocks: 0


Name: 172.21.0.5:9866 (h04.hadoop)
Hostname: h04
Decommission Status : Normal
Configured Capacity: 485473984512 (452.13 GB)
DFS Used: 24576 (24 KB)
Non DFS Used: 22788616192 (21.22 GB)
DFS Remaining: 437949386752 (407.87 GB)
DFS Used%: 0.00%
DFS Remaining%: 90.21%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 0
Last contact: Fri Oct 10 07:48:48 GMT 2025
Last Block Report: Fri Oct 10 07:47:00 GMT 2025
Num of Blocks: 0
```

---

- 也可以在 `localhost:9870` 查看 web ui 的信息
- ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251010164419.png)
---
### 运行程序
---
- 首先，把输入文件（本实验要求 `$HADOOP_HOME/etc/hadoop/*.xml` ）传输到 ` hdfs `
	- `root@h01:/usr/local/hadoop/etc/hadoop# hdfs dfs -put ./*.xml /user/lab1_2/input`
- 然后，运行官方 `jar` 
```bash
yarn jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar grep /user/lab1_2/input /user/lab1_2/output 'dfs[a-z.]+'
```
---
### 查看结果
---
- 同理，下载下来，查看文件
```
root@h01:/usr/dev/lab1# hdfs dfs -ls /user/lab1_2/output
Found 2 items
-rw-r--r--   2 root supergroup          0 2025-10-10 08:41 /user/lab1_2/output/_SUCCESS
-rw-r--r--   2 root supergroup         77 2025-10-10 08:41 /user/lab1_2/output/part-r-00000
root@h01:/usr/dev/lab1# hdfs dfs -get /user/lab1_2/output/part-r-00000 ./lab1_2_output
root@h01:/usr/dev/lab1# vim lab1_2_output
```
---
- 得到
```
1       dfsadmin
1       dfs.replication
1       dfs.namenode.name.dir
1       dfs.namenode.data.dir
```
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251010164831.png)

---
- 去 `web ui` 查看结果
![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251010164502.png)


---
## 实验收获
---
本次实验，进一步熟悉了 `linux` 的命令行操作，熟悉了 `docker` 的进阶用法（以前只会简单应用安装，现在学习了导出自己写好的镜像），体会到了 `docker` 抽象的魅力和并行计算的乐趣。
期间遇到的问题，多数是 `环境配置` 问题，通过 `AI Agent` 和 `Google` 的帮助能得到比较好的解决。仰赖——

---
- 提问的智慧
- 对报错 `log` 的耐心阅读
- 老师讲义的帮助
- 无私的论坛奉献者
---
关于问题的解决，可参见[[#问题和解决问题]] 部分。

---
## 谢谢大家！
