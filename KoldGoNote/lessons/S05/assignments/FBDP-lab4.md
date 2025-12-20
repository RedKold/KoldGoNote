## Environment Setup

- Use docker to setup `spark:latest`, with R, python, scala,  java support.
 ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251215154548.png)
 - Develop Environment: VS Code with docker plugin
 - Use Python Spark (`pyspark`)


- Prepare a `spark-submit` script:
```shell
bin/spark-submit \
  --master local[*] \
  --name "CouponStatisticsJob" \
  /opt/spark/work-dir/lab4/task1-1.py\
  /opt/spark/work-dir/lab4/data/ccf_online_stage1_train.csv
```

## Task1 Spark RDD Programming

### Sub task 1
1. 统计优惠券发放数量： 使⽤ ` ccf_online_stage1_train` 统计每种优惠券的被使⽤次数，并
按数量降序排列输出，完整结果以附件形式给出，实验报告中给出前⼗名优惠券的结果。
输出格式：
```
<Coupon_id> <总使⽤次数>
```

#### Idea 
仍然是一个 `word_count` 类似的问题
用 RDD 的思想来说，我们每次处理 ` ccf_online_stage1_train` 的一行，就是对这个单元 RDD 做操作。
我们可以先 `map` 其做 `split`，将 csv 的一行处理成不同的 token
然后做 `filter` 筛选出符合使用了优惠券 (That is `Action = 2`) 的数据，写入新的 RDD
然后，在新的 RDD 中 `map` 成 `key, 1`, 然后 `reduceByKey` 方法是 `add`（累加出现次数）

最后再排序即可

#### Code
```python
def coupon_count(spark, input_path):
    """
        Use RDD to count each coupon times being used
    """

    sc = spark.sparkContext

    # read the data
    try:
        raw_rdd = sc.textFile(input_path)
    except Exceptoin as e:
        print(f"ERROR: fail to read the data path{input_path}")
        return

    header = raw_rdd.first()
    print(f"header is {header}")
    data_rdd = raw_rdd.filter(lambda row: row!=header)

    # count

    coupon_counts_rdd = (
        data_rdd 
        .map(lambda line : line.split(','))
        # oupon_id != null && Date != null
        .filter(lambda fields:
            # action: 2 means buying
            fields[2].strip() == '2' and
            # Coupon_id
            fields[3].strip() != 'null')
        # start mapping
        .map(lambda fields : 
            (fields[3], 1))
        .reduceByKey(lambda a,b : a+b)
    )

    sorted_rdd = (
        # here we only need map the elements apart
        coupon_counts_rdd
        # map: (count, key)
        # now we use count as key, so it can be sort by key easily
        .map(lambda x : (x[1], x[0]))
        .sortByKey(ascending=False)
    )

    top10 = sorted_rdd.take(10)
```

#### Result
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251216212104.png)

完整请看附件 

### Sub task 2
查询指定商家优惠券使⽤情况： 使⽤ ccf_online_stage1_train 统计每个商家的优惠券使⽤情况，分为负样本、普通消费和正样本三种，按照 Mechant_id 升序排序并将结果存储在新表 online_consumption_table 中，实验报告中给出前⼗⾏结果。

*注：如果Date=null & Coupon_id != null，该记录表示领取优惠券但没有使⽤，即负样本；如果Date!=null &Coupon_id = null，则表示普通消费⽇期；如果Date!=null & Coupon_id != null，则表示⽤优惠券消费⽇期，即正样本。*

输出格式：
```
<Mechant_id> <负样本数量> <普通消费数量> <正样本数量>
```

#### Idea
类似的，我们这次仍然对 RDD 做操作
不同的是，**这次我们有三种样本要标记**。如何做 filter 呢？
我们引出一个函数，做标记，然后返回一个 `<Mechant_id> <negtive> <normal> <positive>` 的元组即可了


```shell
bin/spark-submit \
  --master local[*] \
  --name "CouponCheckJob" \
  /opt/spark/work-dir/lab4/task1-2.py\
  /opt/spark/work-dir/lab4/data/ccf_online_stage1_train.csv
```


#### Code
**仅展示核心逻辑**，详细代码  please read the src
```python

def coupon_check(spark, input_path):
	
	# some code...
	
	coupon_check_rdd = (
    	data_rdd 
    	.map(lambda line : line.split(','))
    	# start mapping
    	.map(lambda fields : 
    	    map_consume_type(fields))
    	# reduce: add up each type, return a tuple
    	.reduceByKey(lambda a,b : (a[0]+b[0], a[1]+b[1], a[2]+b[2]))
	)


def map_consume_type(fields):
    '''
    *注: 如果Date=null & Coupon_id != null,该记录表示领取优惠券但没有使⽤,即负样本;
    如果Date!=null &Coupon_id = null,则表示普通消费⽇期;
    如果Date!=null & Coupon_id != null,则表示⽤优惠券消费⽇期,即正样本。
    *
    '''
    merchant_id = fields[1]
    date = fields[6]
    coupon_id = fields[3]

    count = (0, 0, 0)
    # -1 0 1
    
    # negtive
    if date is "null" and coupon_id is not "null":
        count = (1, 0, 0)
    # normal
    elif date is not "null" and coupon_id is "null":
        count = (0, 1, 0)
    # positive
    elif date is not "null" and coupon_id is not "null":
        count = (0, 0, 1)

    return (merchant_id, count)
```



#### Result
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251216221652.png)

For detailed result please see output directory


## Task 2 Spark SQL Programming
In this task, we need to use Spark SQL to do the job.

[参考文档](https://spark.apache.org/docs/latest/sql-getting-started.html)

### Sub task1
优惠券使⽤时间分布统计： 根据 ccf_offline_stage1_train 表中数据，统计每⼀种优惠券被使⽤时间位于⼀个⽉的上中下旬。给出每⼀种优惠券被使⽤时间的分布。
输出格式：
```
<Coupon_id> <上旬被使⽤概率> <中旬被使⽤概率> <下旬被使⽤概率>
```

#### Idea

提交代码如下

```
bin/spark-submit \
  --master local[*] \
  --name "CouponMonthStatJob" \
  /opt/spark/work-dir/lab4/task2-1.py\
  /opt/spark/work-dir/lab4/data/ccf_offline_stage1_train.csv
```

首先 init the table:
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251218121553.png)
可以看到，if `Date is not 'null'`, then last 2 bit is month
```
[1,10] 上旬
[11,20] 中旬
[21, ] 下旬
```

构造 SQL 查询指令即可



我们用 `sql` 实际返回了一个 `dataframe`, 由于惰性计算机制，只有调用 save 或者 show 的时候才开始计算。

#### Code
此处展示 SQL 查询关键代码
```python
def coupon_month(spark, input_path):
    sc = spark.sparkContext
    df = spark.read.csv(input_path, header=True)

    df.show()


    df.createOrReplaceTempView("offline_train")
    # prepare a SQL query

    sql_qurey = """
    WITH used_coupons AS (
        SELECT 
            Coupon_id, 
            CAST(SUBSTRING(Date,7,2) AS INT) as day
        FROM 
            offline_train
        WHERE Date IS NOT NULL AND Date != 'null' AND Coupon_id != 'null'
    ),
    count_table AS (
        SELECT
            Coupon_id,
            COUNT(*) as total,
            SUM(
                CASE 
                    WHEN day BETWEEN 1 AND 10 THEN 1 ELSE 0
                END
            ) as early_count,
            SUM(
                CASE
                    WHEN day BETWEEN 11 AND 20 THEN 1 ELSE 0
                END
            ) as mid_count,
            SUM(
                CASE
                    WHEN day BETWEEN 21 AND 31 THEN 1 ELSE 0
                END
            ) as late_count
        FROM used_coupons
        GROUP BY Coupon_id
    )

    SELECT
        Coupon_id,
        ROUND(early_count / total, 2) as early_prob,
        ROUND(mid_count / total, 2) as mid_prob,
        ROUND(late_count / total, 2) as late_prob
    FROM count_table
    """

    res = spark.sql(sql_qurey)

    res.show(10)
```


#### Result

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251218131643.png)



### Sub task2
> [!Question] 商家正样本比例统计
  根据 online_consumption_table 表中数据，按正样本⽐例对商家排序，给出正样本⽐例最⾼的前⼗个商家。
输出格式：
> ```
> <Merchant_id> <正样本⽐例> <正样本数量> <总样本数量>
> ```
#### Idea
仍然用 SQL 处理。
先构建表，然后做一个子表统计正比例，最后排序输出即可。

- init the table
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251218133035.png)


- 如果Date!=null & Coupon_id != null，则表示⽤优惠券消费⽇期，即正样本。

启动命令：
```shell
bin/spark-submit \
  --master local[*] \
  --name "CouponPosiStat" \
  /opt/spark/work-dir/lab4/task2-2.py\
  /opt/spark/work-dir/lab4/data/ccf_online_stage1_train.csv
```
#### Code
框架完全类似，这里附上 SQL 查询代码

```sql
WITH count_table AS(
	SELECT 
		Merchant_id,
		COUNT(*) as total,
		SUM(
			CASE 
				WHEN Date != 'null' AND Coupon_id != 'null' THEN 1 ELSE 0
			END
		) as posi_count
	FROM online_train
	GROUP BY Merchant_id
)

--- caclulate the prob

SELECT
	Merchant_id,
	ROUND(posi_count/total,2) as posi_prob
FROM
	count_table
ORDER BY posi_prob|DESC
```
其中 `DESC` 是排序降序关键字

#### Result
取十行
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251218134753.png)


## Task 3 Spark MLlib Programming

本任务是[天池新人实战赛o2o优惠券使用预测任务](https://tianchi.aliyun.com/competition/entrance/231593/information)
可以尝试决策树模型或者Logistic回归


```shell
bin/spark-submit \
  --master local[*] \
  --name "CouponPredict" \
  /opt/spark/work-dir/lab4/task3-main.py
```


![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251218230648.png)
