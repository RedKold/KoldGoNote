---
本科课程: FBDP
completed: "true"
date: 2025-10-30
author: 231275036-朱晗
---

# Lab2 - O2O 优惠券使用情况分析

## 项目简介

本项目使用 Hadoop MapReduce 对线下优惠券使用数据进行统计分析，探索影响优惠券使用行为的各种因素。
## 项目结构

```
lab2/
├── pom.xml                              # Maven配置文件
├── README.md                            # 项目说明文档
├── dataset/                             # 数据目录
│   ├── ccf_offline_stage1_train.csv     # 线下训练数据
│   ├── ccf_online_stage1_train.csv      # 线上训练数据
│   └── ...
└── src/main/java/lab2/
    ├── TaskRunner.java                  # 任务运行器
    ├── Task1MerchantCouponUsage.java    # 任务1：商家优惠券使用统计
    ├── Task2MerchantDistanceAnalysis.java  # 任务2：商家距离分析
    ├── Task3CouponConsumptionInterval.java # 任务3：优惠券使用间隔
    └── Task4DiscountRateAnalysis.java   # 任务4：折扣率影响分析
└── run_all_tasks.sh					 # 运行所有任务的shell脚本
```

请保证你在 `hadoop 3.4.2` 环境下运行此项目，以避免未知的问题。
本作业项目已经在 `ubuntu 22.04 - arm64` 下测试
## 数据说明

### 字段描述

| 字段名 | 含义 | 示例 |
|--------|------|------|
| User_id | 用户标识符 | 2166529 |
| Merchant_id | 商家标识符 | 7113 |
| Coupon_id | 优惠券标识符 | 6928 |
| Discount_rate | 折扣率 | 200:20:00 (满 200 减 20) |
| Distance | 用户与商家距离 (×500m) | 5 (2.5km) |
| Date_received | 优惠券领取日期 | 20160727 |
| Date | 优惠券使用日期 | NULL |

### 数据判断逻辑

- **负样本**：`Coupon_id != null && Date == null` - 领取了优惠券但未使用
- **普通消费**：`Coupon_id == null` - 未领取优惠券，直接消费
- **正样本**：`Coupon_id != null && Date != null` - 领取了优惠券并使用

## 任务说明

### 任务 1：商家优惠券使用统计

**目标**：统计每个商家的优惠券使用情况

**输出格式**：`<Merchant_id> TAB <负样本数> TAB <普通消费数> TAB <正样本数>`

**运行命令**：
```bash
hadoop jar coupon-analysis-1.0-SNAPSHOT.jar lab2.Task1MerchantCouponUsage \
  /input/dataset/ccf_offline_stage1_train.csv /output/task1
```


#### 设计思路
主要就是根据上述数据判断逻辑，清点个数即可。本质仍然是一个 `wordcount` 类任务。

伪代码如下

```
/**
 *	Mapper
 */
 void map():
 	if(领取了消费券):
 		dataType="normal"
	else:
		if(data is null):
			// not use
			dataType = "negative"
		else:
			dataType = "positive"
	emit the <merchantId : dataType> to Reducers
/**
 *	Reducer	
 */				
 void reduce():
 	// 统计数量
	for (Text value : values) {
		String type = value.toString();
		if ("negative".equals(type)) {
			negativeCount++;
		} else if ("normal".equals(type)) {
			normalCount++;
		} else if ("positive".equals(type)) {
			positiveCount++;
		}
	}
```

### 任务 2：商家距离活跃消费者统计

**目标**：统计每个商家不同距离的活跃消费者人数

**输出格式**：`<Merchant_id> TAB <距离为x的消费者人数>`

**说明**：Distance 字段含义
- `null`: 无距离信息
- `0`: 小于 500 米
- `1-10`: 表示实际距离为 x×500 米
- `10`: 大于等于 5 公里

**运行命令**：
```bash
hadoop jar coupon-analysis-1.0-SNAPSHOT.jar lab2.Task2MerchantDistanceAnalysis \
  /input/dataset/ccf_offline_stage1_train.csv /output/task2
```

#### 设计思路
首先要确保字段完整。
我们需要的字段，在 `csv` 表中：
```java
String userId = fields[0].trim();
String merchantId = fields[1].trim();
String distance = fields[4].trim();
```

然后 `Mapper` 把 `< <merchantId> + "_" + <distance> , userId>` 这样的组发送给 `Reducer`

> [!Note] 
> 用 `_`, `#` 等字符分割信息，到 Reducer 再 `parse` 解析，是一个常用的好办的方法



### 任务 3：优惠券使用时间间隔统计

**目标**：统计优惠券从领取到使用的平均间隔时间

**输出格式**：`<Coupon_id> TAB <平均消费间隔>`（仅输出使用次数>总使用次数 1%的优惠券）

**运行命令**：
```bash
hadoop jar coupon-analysis-1.0-SNAPSHOT.jar lab2.Task3CouponConsumptionInterval \
  /input/dataset/ccf_offline_stage1_train.csv /output/task3
```

#### 设计思路
和任务 2 类似。



### 任务 4：折扣率影响分析（自选任务）

**目标**：分析折扣率对优惠券使用行为的影响

**分析维度**：
1. 不同折扣率下优惠券的使用率（领取数 vs 使用数）
2. 不同折扣率下优惠券的平均使用时间
3. 不同折扣类型（满减 vs 折扣）的使用情况

**Discount_rate 字段格式**：
- 小数 (0-1)：直接折扣率，如 `0.8` 表示 8 折
- `x:y:z`：满减方案，如 `200:50:00` 表示满 200 减 50
- `"fixed"`：固定面值

**运行命令**：
```bash
hadoop jar coupon-analysis-1.0-SNAPSHOT.jar lab2.Task4DiscountRateAnalysis \
  /input/dataset/ccf_offline_stage1_train.csv /output/task4
```

本部分文件可以阅读 
[assignment4-output](https://git.nju.edu.cn/Red_Kold/fbdp-lab2/-/blob/main/output/task4/part-r-00000)

#### 详细分析

- **对于折扣**

| **折扣率** | **优惠程度 (越低越优惠)** | **领取数** | **使用数** | **使用率**    | **平均使用间隔 (天)** |
| ------- | ---------------- | ------- | ------- | ---------- | -------------- |
| 0.2     | **8 折**          | 59      | 3       | 5.08%      | 0.00           |
| 0.5     | **5 折**          | 104     | 16      | 15.38%     | 1.00           |
| 0.6     | **4 折**          | 41      | 1       | 2.44%      | 0.00           |
| 0.7     | **3 折**          | 29      | 2       | 6.90%      | 0.00           |
| 0.75    | **2.5 折**        | 75      | 2       | 2.67%      | 3.50           |
| 0.8     | **2 折**          | 1992    | 377     | **18.93%** | 0.46           |
| 0.85    | **1.5 折**        | 379     | 22      | 5.80%      | 0.18           |
| 0.9     | **1 折**          | 4791    | 358     | 7.47%      | 0.39           |
| 0.95    | **0.5 折**        | 12204   | 1435    | 11.76%     | 0.35           |
- 基本上看，**优惠程度越高**，大家就更喜欢领取和使用。且平均使用间隔也更少。值得注意的是，`2` 折的使用率最高，**这可能是由于更高折扣的使用条件比较苛刻**。

- **对于满减**

| **门槛 (x)**        | **优惠面值 (y)** | **领取数** | **使用数** | **使用率**    | **平均使用间隔 (天)** |
| ----------------- | ------------ | ------- | ------- | ---------- | -------------- |
| **低门槛 (5, 10)**   |              |         |         |            |                |
| 5                 | 1            | 1496    | 401     | **26.80%** | 7.77           |
| 10                | 1            | 10667   | 2554    | **23.94%** | 7.67           |
| 10                | 5            | 15654   | 1923    | 12.28%     | 7.37           |
| **中低门槛 (20, 30)** |              |         |         |            |                |
| 20                | 1            | 30630   | 4827    | 15.76%     | 8.64           |
| 20                | 5            | 54171   | 7430    | 13.72%     | 6.10           |
| 30                | 5            | 161932  | 13966   | 8.62%      | 7.61           |
| **中门槛 (50, 100)** |              |         |         |            |                |
| 100               | 50           | 1026    | 147     | 14.33%     | 11.91          |
| **高门槛 (200 及以上)** |              |         |         |            |                |
| 200               | 100          | 8       | 2       | **25.00%** | 13.00          |
| 150               | 5            | 6       | 2       | **33.33%** | 12.00          |
- 满减使用率最高的是两种情形：
	- **低门槛**，`5,10`，平均使用间隔也最短。对于这类便宜的物件，消费者往往抱有随手就买的心理。
	- **高门槛**，其中 `150` 使用率最高，但是其样本较少，可能参考性较低。
- 其中中门槛的领取数绝对领先。


对于 **满减** 和 **折扣**。


## 编译与运行

### 1. 编译项目

```bash
mvn clean package
```

编译成功后，会在 `target/` 目录下生成 `coupon-analysis-1.0-SNAPSHOT.jar` 文件。
**此 jar 包文件**可以在 `gitlab` 仓库下载

### 2. 上传数据到 HDFS

```bash
# 创建输入目录
hdfs dfs -mkdir -p /input/dataset

# 上传数据文件
hdfs dfs -put dataset/ccf_offline_stage1_train.csv /input/dataset/
hdfs dfs -put dataset/ccf_online_stage1_train.csv /input/dataset/

# 创建输出目录
hdfs dfs -mkdir -p /output
```

### 3. 运行任务

#### 方式 0: 使用 `run_all_tasks.sh` 脚本

本代码仓库中实现了一个便捷实现
- 清理输出目录
- 运行四个任务
- 反馈运行情况
的 `shell` 脚本。

#### 方式 1：使用 TaskRunner（推荐）

```bash
hadoop jar target/coupon-analysis-1.0-SNAPSHOT.jar \
  1 /input/dataset/ccf_offline_stage1_train.csv /output/task1
```


其中任务编号：
- `1` - 任务 1
- `2` - 任务 2
- `3` - 任务 3
- `4` - 任务 4

#### 方式 2：直接运行单个任务

```bash
# 任务1
hadoop jar target/coupon-analysis-1.0-SNAPSHOT.jar lab2.Task1MerchantCouponUsage \
  /input/dataset/ccf_offline_stage1_train.csv /output/task1

# 任务2
hadoop jar target/coupon-analysis-1.0-SNAPSHOT.jar lab2.Task2MerchantDistanceAnalysis \
  /input/dataset/ccf_offline_stage1_train.csv /output/task2

# 任务3
hadoop jar target/coupon-analysis-1.0-SNAPSHOT.jar lab2.Task3CouponConsumptionInterval \
  /input/dataset/ccf_offline_stage1_train.csv /output/task3

# 任务4
hadoop jar target/coupon-analysis-1.0-SNAPSHOT.jar lab2.Task4DiscountRateAnalysis \
  /input/dataset/ccf_offline_stage1_train.csv /output/task4
```

### 4. 查看结果

```bash
# 查看任务输出
hdfs dfs -ls /output/task1/
hdfs dfs -cat /output/task1/part-r-00000 | head -20

# 下载结果到本地
hdfs dfs -get /output/task1 /local/output/
```

![image.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/20251030144236.png)

访问 JobHistory，可见任务都已实现完成。


---
**其他结果输出**

`task1`
```
// 1-20 lines
1	0	12	0
100	1	1	0
1000	0	5	0
1001	7	20	14
1002	5	8	0
1003	0	3	0
1004	82	281	1
1005	30	19	1
1007	3	22	0
1008	0	11	0
1009	0	1	0
101	0	4	0
1010	3	10	2
1011	63	59	6
1013	29	64	5
1015	19	16	1
1016	10	5	1
1017	8	9	0
1018	4	6	5
1019	0	4	0
```

`task2`
```
1000	距离为0	1
1001	距离为0	4
1001	距离为10	2
1001	距离为3	1
1001	距离为5	1
1001	距离为7	1
1002	距离为0	3
1003	距离为0	1
1003	距离为1	1
1004	距离为0	30
1004	距离为1	5
1004	距离为10	1
1004	距离为2	2
1004	距离为5	2
1004	距离为7	2
1005	距离为0	9
1005	距离为1	4
1005	距离为10	3
1005	距离为2	3
1005	距离为3	1
```

`task3`

> [!Note] 输出的 `_` 的说明
> 这里我使用 `_` 分隔了 `key` 和 `使用次数`，以帮助后面用。
> 使用次数的实现上: `String outputKey = key.toString() + "_" + intervals.size();`


```
1_1	11.0
10_13	4.6923076923076925
100_1	7.0
1000_3	4.0
10000_9	4.888888888888889
10001_1	0.0
10002_2	2.5
10004_6	2.5
10005_1	1.0
10006_1	8.0
10009_1	6.0
10012_1	7.0
10014_2	4.5
10017_1	25.0
1002_1	0.0
10021_2	6.0
10024_6	11.333333333333334
10027_2	1.0
10028_10	14.3
10029_1	5.0
```

## 依赖说明

项目使用以下主要依赖：

- **Hadoop 3.3.1** - Hadoop MapReduce 框架

## 代码特点

1. **完整的注释**：所有类和方法都有详细的中文注释说明
2. **规范的输出格式**：严格按照题目要求的输出格式
3. **错误处理**：对空值和异常情况进行适当处理
4. **数据去重**：任务 2 中统计活跃消费者时进行去重处理

## 注意事项

1. **时间间隔计算**：使用 `yyyyMMdd` 格式解析日期，计算天数差
2. **数据字段判断**：使用字符串 `"null"` 表示空值
3. **CSV 解析**：使用简单的 `split(",")` 方法，假设数据中不包含逗号
4. **HDFS 路径**：运行前确保数据文件已上传到 HDFS




