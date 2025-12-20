
| 姓名  | 学号        |
| --- | --------- |
| 朱晗  | 231275036 |

## 任务 1 导入数据


```sql
CREATE EXTERNAL TABLE IF NOT EXISTS ccf_offline_stage1_train (
  user_id BIGINT,
  merchant_id BIGINT,
  coupon_id INT,
  discount_rate STRING,
  distance INT,
  date_received STRING,
  consume_date STRING
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
WITH SERDEPROPERTIES (
  "field.delim"=",",
  "serialization.null.format"="null",
  "skip.header.line.count"="1"  -- 明确告诉 SerDe 跳过第一行表头
)
STORED AS TEXTFILE
LOCATION '/user/hive/csv_data/offline';
```


创建 online 的同理

```sql
CREATE EXTERNAL TABLE IF NOT EXISTS ccf_online_stage1_train (
  user_id BIGINT,
  merchant_id BIGINT,
  action INT,
  coupon_id INT,
  discount_rate STRING,
  date_received STRING,
  consume_date STRING
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
WITH SERDEPROPERTIES (
  "field.delim"=",",
  "serialization.null.format"="null",
  "skip.header.line.count"="1"  -- 明确告诉 SerDe 跳过第一行表头
)
STORED AS TEXTFILE
LOCATION '/user/hive/csv_data/online';
```
我们第一行表头不是数据，而是字符串对表头的说明。在做的时候 hive 因为出现 string ->int 失败的错误。因此查阅[这篇教程](https://community.cloudera.com/t5/Support-Questions/ParseException-missing-EOF-error-message-with-Hive-Query/m-p/206332)得到：设定跳过第一行。

![image.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/20251127201530.png)

建表成功

查看数据
```sql
select * from ccf_offline_stage1_train limit 5;
```
![image.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/20251127200208.png)

已经正常加载了。
![image.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/20251127201720.png)

检查 `ccf_online_stage1_train` 同理
![image.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/20251127204343.png)


```sql
SELECT COUNT(*) FROM ccf_offline_stage1_train;
```

可以看到输入命令后，后台启动了一个 MapReduce 任务。

![image.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/20251127202021.png)

结果为 `1048575`，对照原始 `csv` 行数，说明数据集已经全部加载。

`online` 同理：
```sql
SELECT COUNT(*) FROM ccf_online_stage1_train;
```
![image.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/20251127202308.png)



## 任务 2 基本数据查询

### sub-task 1
1. 查询⽤户⾏为数量： 使⽤ ccf_online_stage1_train 统计三种⾏为（点击、购买、领取）的总次数，并按数量降序排列输出。
输出格式：

```
<⾏为> <总次数>
```

此信息在 action 列

```sql
SELECT
  -- 将 action 数值映射为行为名称（0=点击，1=购买，2=领取）
  CASE
    WHEN action = 0 THEN 'click'
    WHEN action = 1 THEN 'purchase'
    WHEN action = 2 THEN 'receive'
  END AS Action,
  -- 统计每种行为的总次数
  COUNT(*) AS ActionAmt 
FROM ccf_online_stage1_train
-- 按行为类型分组（本质是按 CASE 映射后的结果分组）
GROUP BY
  CASE
    WHEN action = 0 THEN 'click'
    WHEN action = 1 THEN 'purchase'
    WHEN action = 2 THEN 'receive'
  END
-- 按行为总数降序排列
ORDER BY ActionAmt DESC;
```


![image.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/20251127205552.png)

结果为：
```
click	860169
purchase	127878
receive	60528
```

根据 `excel` 对原始表做检验，答案正确
![image.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/20251127205915.png)

### sub-task 2

2. 查询指定商家优惠券使⽤情况： 使⽤ ccf_online_stage1_train 统计每个商家的优惠券使⽤情况，分为负样本、普通消费和正样本三种，并将结果存储在新表 online_consumption_table 中。
*注*：如果 Date=null & Coupon_id != null，该记录表示领取优惠券但没有使⽤，即负样本；如果 Date!=null & Coupon_id = null，则表示普通消费⽇期；如果 Date!=null & Coupon_id != null，则表示⽤优惠券消费⽇期，即正样本。

输出要求： 通过下⾯的指令得到所需要的结果。

```sql
SELECT * FROM online_consumption_table LIMIT 20;
```

输出格式：

```
<Mechant_id> <负样本数量> <普通消费数量> <正样本数量>
```


```sql
-- 创建新表 online_consumption_table 存储结果
CREATE TABLE IF NOT EXISTS online_consumption_table (
  merchant_id BIGINT,        -- 商家ID（与原表字段类型一致）
  negtive_amt INT,
  normal_amt INT,
  positive_amt INT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'  -- 制表符分隔，便于查看
STORED AS TEXTFILE;

-- 插入统计结果到新表
INSERT INTO TABLE online_consumption_table
SELECT
  merchant_id,
  -- 统计负样本数量（领券未使用）
  SUM(CASE WHEN coupon_id IS NOT NULL AND consume_date IS NULL THEN 1 ELSE 0 END) AS negtive_amt,
  -- 统计普通消费数量（无券消费）
  SUM(CASE WHEN coupon_id IS NULL AND consume_date IS NOT NULL THEN 1 ELSE 0 END) AS normal_amt,
  -- 统计正样本数量（用券消费）
  SUM(CASE WHEN Coupon_id IS NOT NULL AND consume_date IS NOT NULL THEN 1 ELSE 0 END) AS positive_amt
FROM ccf_online_stage1_train
-- 按商家ID分组
GROUP BY merchant_id;
```


![image.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/20251127212831.png)


## 任务 3 数据聚合分析

### sub-task 1
对每个商家与周边消费者的距离进⾏统计，给出不同距离的活跃消费者⼈数

消费者需要用 `DISTINCT` 去重
```sql
SELECT
 merchant_id,
 COALESCE(distance, 'NULL') AS Distance,
 COUNT(DISTINCT user_id) AS customer_amt
FROM ccf_offline_stage1_train
GROUP BY merchant_id, distance
;
```


输出如下：

![image.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/20251127211129.png)

### sub-task 2

商家正样本⽐例统计： 根据online_consumption_table 表中数据，按正样本⽐例对商家排序，给出正样本⽐例最⾼的前⼗个商家。

```sql
SELECT
  merchant_id,
  -- 计算正样本比例（保留3位小数）
  ROUND(
    CASE WHEN (negtive_amt + normal_amt + positive_amt) = 0 THEN 0  -- 分母为0时比例设为0
         ELSE positive_amt / (negtive_amt + normal_amt + positive_amt)  -- 正常计算比例
    END,
    3
  ) AS positive_ratio,
  -- 顺带输出各样本数量（可选，便于验证）
  positive_amt,
  negtive_amt,
  normal_amt
FROM online_consumption_table
-- 按正样本比例降序排列
ORDER BY positive_ratio DESC
-- 取前10个商家
LIMIT 10;
```

![image.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/20251127212909.png)
- 结果如图所示


## 任务 4 复杂查询与分析

### sub-task 1

我的 Hive 在翻译成 MapReduce 程序时，尝试使用 `MapJoin`，但是我没有安装这个库。

通过如下配置避免优化
```sql
SET hive.auto.convert.join=false;
SET hive.optimize.autojoin=false;
```


```sql
SELECT
    t_interval.Coupon_id, -- 修正：使用 t_interval
    t_count.use_count AS use_count_total, -- 修正：使用 t_count
    ROUND(AVG(t_interval.consume_interval), 1) AS avg_interval_days -- 修正：使用 t_interval
FROM (
    -- Step 1: 计算每条记录的消费间隔
    SELECT
        Coupon_id,
        DATEDIFF(
            FROM_UNIXTIME(UNIX_TIMESTAMP(consume_date, 'yyyyMMdd'), 'yyyy-MM-dd'),
            FROM_UNIXTIME(UNIX_TIMESTAMP(date_received, 'yyyyMMdd'), 'yyyy-MM-dd')
        ) AS consume_interval
    FROM ccf_offline_stage1_train
    WHERE
        Coupon_id IS NOT NULL 
        AND date_received IS NOT NULL 
        AND consume_date IS NOT NULL
) t_interval
-- JOIN 统计每张优惠券的总使用次数
JOIN (
    -- Step 2: 统计每张优惠券的总使用次数 (use_count)
    SELECT
        Coupon_id,
        COUNT(*) AS use_count
    FROM ccf_offline_stage1_train
    WHERE
        Coupon_id IS NOT NULL 
        AND date_received IS NOT NULL 
        AND consume_date IS NOT NULL
    GROUP BY Coupon_id
) t_count ON t_interval.Coupon_id = t_count.Coupon_id
-- CROSS JOIN 阈值
CROSS JOIN (
    -- Step 3: 计算总使用次数和1%的阈值 (threshold)
    SELECT
        ROUND(SUM(total_use) * 0.01, 0) AS threshold
    FROM (
        -- 计算所有有效使用记录的总数
        SELECT
            COUNT(*) AS total_use
        FROM ccf_offline_stage1_train
        WHERE
            Coupon_id IS NOT NULL
            AND date_received IS NOT NULL
            AND consume_date IS NOT NULL
        GROUP BY Coupon_id -- 聚合是为了得到每张优惠券的使用次数，SUM在外层实现总数
    ) t_total
) t_threshold
WHERE
    t_count.use_count > t_threshold.threshold -- 筛选：使用次数 > 阈值
-- 最终按优惠券ID和使用次数分组，计算平均间隔
GROUP BY 
    t_interval.Coupon_id, 
    t_count.use_count
ORDER BY 
    t_count.use_count DESC;
```

![image.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/20251127222342.png)


输出结果如图所示：

整理成表格：

|**优惠券 ID (coupon_id)**|**总使用次数 (use_count_total)**|**平均消费间隔（天） (avg_interval_days)**|
|---|---|---|
|**11539**|**1846**|**8.6**|
|**10323**|**1392**|**5.0**|
|**111**|**1063**|**11.2**|
|**12034**|**1019**|**8.5**|
|**5686**|**995**|**5.0**|
|2418|848|9.2|
|4773|807|8.8|
|9776|535|12.3|
|7751|455|11.2|

### sub-task 2


```sql
-- 强制避免 MapJoin，确保兼容性
SET hive.auto.convert.join=false;
SET hive.optimize.autojoin=false;

SELECT
    t_use_rate.Coupon_id,
    t_use_rate.use_rate,
    t_use_count.use_count,
    -- Step 5: Format Discount Rate
    CASE
        -- Full Reduction: (Original Price - Discounted Price) / Original Price
        WHEN t_use_rate.discount_type = 'FULL_REDUCTION' THEN 
             CAST(ROUND((t_use_rate.discount_off / t_use_rate.discount_full) * 100, 2) AS STRING) || '%'
        -- Discount: (1 - z) * 100%
        WHEN t_use_rate.discount_type = 'DISCOUNT' THEN 
             CAST(ROUND((1 - t_use_rate.discount_z) * 100, 2) AS STRING) || '%'
        ELSE 'N/A'
    END AS discount_pct,
    t_use_rate.discount_type
FROM (
    -- Step 3 & 4: Calculate Use Rate and Parse Discount Parameters
    SELECT
        t_all.Coupon_id,
        CAST(SUM(t_all.is_used) AS DOUBLE) / CAST(COUNT(t_all.Coupon_id) AS DOUBLE) AS use_rate,
        -- Parse Discount Parameters
        -- Full Reduction (x:y)
        CAST(SPLIT(t_all.discount_rate, ':')[0] AS DOUBLE) AS discount_full, -- x (full price)
        CAST(SPLIT(t_all.discount_rate, ':')[1] AS DOUBLE) AS discount_off,  -- y (off price)
        -- Discount (z)
        CAST(t_all.discount_rate AS DOUBLE) AS discount_z,
        -- Determine Discount Type
        CASE
            WHEN t_all.discount_rate LIKE '%:%' THEN 'FULL_REDUCTION'
            ELSE 'DISCOUNT'
        END AS discount_type
    FROM (
        -- Step 1 & 2: Filter Valid Received Records and Mark Usage
        SELECT
            Coupon_id,
            discount_rate,
            -- is_used: 1=Used (Positive Sample), 0=Unused (Negative Sample)
            CASE WHEN consume_date IS NOT NULL THEN 1 ELSE 0 END AS is_used
        FROM ccf_offline_stage1_train
        WHERE
            -- Only count valid received coupons
            Coupon_id IS NOT NULL
            AND date_received IS NOT NULL
    ) t_all
    GROUP BY t_all.Coupon_id, t_all.discount_rate
) t_use_rate
-- Join with t_use_count (Total Times Used)
JOIN (
    -- Step 6: Calculate Actual Use Count (Positive Samples)
    SELECT
        Coupon_id,
        COUNT(*) AS use_count
    FROM ccf_offline_stage1_train
    WHERE
        Coupon_id IS NOT NULL
        AND date_received IS NOT NULL
        AND consume_date IS NOT NULL
    GROUP BY Coupon_id
) t_use_count ON t_use_rate.Coupon_id = t_use_count.Coupon_id
-- Final Sort and Limit
ORDER BY t_use_rate.use_rate DESC
LIMIT 10;
```

折扣方式我们有两种
- `DISCOUNT`: 打折扣
- `FULL_REDUCTION`: 满减
![image.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/20251127224509.png)


## 感受
在本次实验，我亲自体验了 Hive 查询指令翻译成 MapReduce 任务执行的过程，这是之前串行数据库所没有的体验，使我对 Hadoop 家族有了更深的理解。
我的数据库选型是默认的 Derby，同时也可以选择 SQL 等，但是 Hive 使用的 HiveQL 语言不关心具体的数据库选型，**这正是抽象和封装的魅力**，为程序员带来了方便。


同时，对于实验 2 用手动编写 MapReduce 任务实现的 wordcount，hive 显然更简单，体现了自底向上逐渐封装简化的妙处，也体现了 MapReduce 计算框架的**生命力。**

