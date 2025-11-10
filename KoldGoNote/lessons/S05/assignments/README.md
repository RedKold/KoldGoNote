# FBDP-assignment5
This is the repository of my Hadoop MapReduce assignment, assigned by course FBDP, 2025 Spring.
You can read the file `assignment.md` for detail.

## Tree

```
├── assignment.md
├── output
│   ├── negative-r-00000
│   └── positive-r-00000
├── pom.xml
├── README.md
├── src
│   ├── java
│   ├── main
│   │   └── java
│   │       ├── org
│   │       │   └── example
│   │       └── swc
│   │           └── SentimentWordCount.java
│   └── test
│       └── java
├── stock_data.csv
└── stop-word-list.txt
```

- This is the tree structure of my project. I have prepared a pom.xml file, you can use it to manage java package.

## Usage

### Compile & Package

```bash
mvn clean \
mvn compile
```
After that, you can get the jar file for the hadoop to run by 

```bash
mvn package
```

It will generate a jar file in `/target`.

### Hadoop command to run

You can enter this command to run it as a mapreduce program. Be sure you have started your hdfs and yarn service, and input files have been upload to hdfs.

```bash
hadoop jar <your jar file> swc.SentimentWordCount \
    <your stock_data.csv> \
    <your output path> \
    <your stop-word-list.txt>
```

## Output

This program will generate two output file:
`negative-r-00000`  and `positive-r-00000`. each contains top 100 most frequent words.


## 设计

### 基础Java 工具
- `Java BufferedReader`
	- `java nio`
	- 读取停词列表
	- `readLine()`：从输入流读取一行文本直到遇到 `\n, \r, \r\n` 。

- `Java StringTokenizer`
	- `java.util` 
	-  **`StringTokenizer (String str, String delim, boolean returnDelims)`** ：构造一个用来解析 str 的 StringTokenizer 对象，并提供一个指定的分隔符，同时，指定是否返回分隔符。

对于我们的数据，忽略大小写，忽略标点符号，忽略数字可以用正则表达式 regex 实现
```regex
// 忽略大小写，忽略标点符号，忽略数字
String token = itr.nextToken().replaceAll["^a-zA-Z]", "");
```

### Mapper
我们的 `Mapper` 应当能够读取 `stock_data.csv` 的内容，按照规定在 `Context` 操作，将 `<label\tword>, 1` 这样的单位词频信息交给 `Reducer`

对于标签的获取，有一个取巧的办法：**直接读取最后一个逗号分隔的字符**。
伪代码设计为：
```
Mapper 输入: <key, value>  // key是行号或偏移量，value是一行文本
Mapper 输出: <sentiment\tword, 1>  // sentiment是标签("1"或"-1")，word是单词

初始化:
    stopWords = 读取 stop-word-list.txt 并存入集合

map(key, value):
    line = value.toLowerCase()
    
    # 获取情感标签
    lastComma = line.lastIndexOf(',')
    if lastComma == -1:
        return  # 异常行跳过
    sentiment = line.substring(lastComma + 1).trim()
    if sentiment 不是 "1" 或 "-1":
        return

    # 处理文本内容
    textPart = line.substring(0, lastComma)
    tokens = 拆分 textPart 为单词 (用空格或其他分隔符)
    for token in tokens:
        token = 去掉 token 中非字母字符
        if token 为空:
            continue
        if token 在 stopWords 中:
            continue

        outKey = sentiment + "\t" + token
        输出 <outKey, 1>

```


### Reducer
保持 `top 100` ，排序是不值当的。我们可以维护一个优先队列，容量超过 `100` 时候，移除 `pq.min` 即可。


```
Reducer 输入: <sentiment\tword, [1, 1, 1, ...]>
Reducer 输出: <sentiment\tword, count>

初始化:
    N = 100  # Top N
    topMap = Map<String, PriorityQueue<WordCount>>()  # key: sentiment, value: 小顶堆，存 WordCount

reduce(key, values):
    # key = sentiment\tword
    # values = 一系列 1
    sum = 0
    for val in values:
        sum += val

    split key into sentiment, word
    if sentiment not in topMap:
        topMap[sentiment] = new PriorityQueue(按 count 升序)  # 小顶堆
    
    pq = topMap[sentiment]
    pq.offer(WordCount(word, sum))
    
    # 保持堆大小不超过 N
    if pq.size() > N:
        pq.poll()  # 弹出最小的 count

cleanup():
    # reduce 完所有 key 后，输出 Top N
    for sentiment in topMap:
        pq = topMap[sentiment]
        list = convert pq to list
        sort list by count 降序
        for wc in list:
            输出 <sentiment\twc.word, wc.count>

```

##### cleanup
这里额外注意一个 `cleanup` 方法。
`cleanup` 是 `Reducer` 的一个生命周期方法，整个 Reduce 任务处理完所有 `key-value` 之后被调用一次。
我们这里需要重写 `cleanup`：
- **能够排序输出结果**
- **能够实现多文件输出**（**分别输出负面和正面文件**）
	- 可利用 hadoop 的 `import org.apache.hadoop.mapreduce.lib.output.MultipleOutputs;`
## 代码

可阅读 `gitlab` [仓库](https://git.nju.edu.cn/Red_Kold/fbdp-assignment4)。

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251021112623.png)

## 测试
首先，上传文件到 `HDFS` 文件系统
```
hadoop jar sentiment-wordcount.jar swc.SentimentWordCount \
    /user/as5/input/stock_data.csv \
    /user/as5/output \
    /user/as5/stop-word-list.txt
```

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251021150931.png)


![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251021145224.png)
这次 `Failed`，读报错信息，发现是没有设置好 `named ouptuts`
在 `main` 函数 增加如下代码：
```java
// 注册 named outputs
    MultipleOutputs.addNamedOutput(job, "positive",
            org.apache.hadoop.mapreduce.lib.output.TextOutputFormat.class,
            Text.class, IntWritable.class);
    MultipleOutputs.addNamedOutput(job, "negative",
            org.apache.hadoop.mapreduce.lib.output.TextOutputFormat.class,
            Text.class, IntWritable.class);
```

- 修正后，可以通过
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251021151028.png)

### 查看结果

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251021151101.png)

- 查看输出目录
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251021151257.png)


用 `Vim` 查看输出
- 负面词 top `100`
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251021151510.png)
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251021151521.png)
		- 可见确实有 100 行。
- 正面词 `top 100`
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251021151639.png)


## 性能分析
**优点**
- **维护优先队列：** 我的程序通过维护 `TOP N` 优先队列，在最后 `cleanup` 阶段再归并各个 `TOP N` 并排序，起到了一个提高性能的效果。
	- 但是在 `Reducer` 量比较大的情况，归并的代价仍然较大。
- **使用多个输出文件**：能直接分开正面和负面输出，方便后续分析和存储。

**缺点**
- **停词文件每个 Mapper 单独加载**：如果停词列表很大，多个 Mapper 节点重复读取会浪费内存和 I/O，适合用 Hadoop 的 DistributedCache（或 HDFS 广播方式）统一分发。


## 收获

本次作业，从零开始掌握了一个 `hadoop mapreduce` 程序的编码、编译、打包、执行的全周期过程，收获良多。

之前写 `C` 程序多一些，这次 `Java` 实践包括鼓捣 `Unity C#`，让我体会到强**面向对象**语言的魅力和便捷。通过对 `mapreduce` 标准 `mapper` 和 `reducer` 的 `Override`，居然就能便捷的完成我们的目标。而 `cleanup` 等方法的 `Override` 就可以完成排序输出，实在是提供了一种一种可控制的方便的变化。继承多态之美，就在其中。
