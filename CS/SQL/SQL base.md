## 基础语法
你可以通过 `WITH ... AS` 创建临时的子表
## 条件聚合

在 SQL 中，将 `SUM` 和 `CASE WHEN` 结合使用是一种极其强大的技术，被称为 **“条件聚合” (Conditional Aggregation)**。

在你的优惠券任务中，它能让你在**一次扫描**中，把原本分散在多行的数据，“分类统计”到不同的列里。

---

### 1. `CASE WHEN` 基础

它相当于编程语言里的 `if...else`。

SQL

```
CASE 
    WHEN 条件1 THEN 结果1
    WHEN 条件2 THEN 2
    ELSE 默认结果 
END
```

### 2. `SUM(CASE ...)` 的核心逻辑

如果你直接用 `COUNT(*)`，它会数出所有行。但如果你想“只数满足某个条件的行”，逻辑是这样的：

1. **判定**：如果符合条件，给它记为 `1`。
    
2. **忽略**：如果不符合条件，给它记为 `0`。
    
3. **求和**：把这些 `1` 和 `0` 全部加起来。
    

**结果：** 得到的总和就是符合该特定条件的记录总数。

---

### 3. 你的任务示例：计算三旬分布

假设你有一张表 `cleaned_data`，里面有 `Coupon_id` 和 `day`（天数）。

SQL

```
SELECT 
    Coupon_id,
    -- 如果天数在1-10，产生个1，否则0。SUM加起来就是上旬总数。
    SUM(CASE WHEN day BETWEEN 1 AND 10 THEN 1 ELSE 0 END) AS early_count,
    
    -- 如果天数在11-20，记为1。
    SUM(CASE WHEN day BETWEEN 11 AND 20 THEN 1 ELSE 0 END) AS mid_count,
    
    -- 如果天数 > 20，记为1。
    SUM(CASE WHEN day > 20 THEN 1 ELSE 0 END) AS late_count
FROM cleaned_data
GROUP BY Coupon_id
```

---

### 4. 为什么要用 `SUM` 而不是 `COUNT`？

这是一个常见的坑：

- **`SUM(CASE WHEN condition THEN 1 ELSE 0 END)`**：永远正确。
    
- **`COUNT(CASE WHEN condition THEN 1 ELSE 0 END)`**：**错误！** 因为 `COUNT` 会数出所有非空的值。即使是 `0`，也是一个非空值，所以 `COUNT` 会把不符合条件的行也数进去。
    
- 如果你非要用 `COUNT`，必须写成：`COUNT(CASE WHEN condition THEN 1 ELSE NULL END)`，因为 `COUNT` 不数 `NULL`。


## GROUP BY 语句

GROUP BY 语句用于结合聚合函数，根据一个或多个列对结果集进行分组。

### SQL GROUP BY 语法

```sql
SELECT column_name, aggregate_function(column_name)  
FROM table_name  
WHERE column_name operator value  
GROUP BY column_name;
```

# SQL ROUND ()
## ROUND() 函数

ROUND() 函数用于把数值字段舍入为指定的小数位数。

### SQL ROUND() 语法

SELECT ROUND(column_name,decimals) FROM TABLE_NAME;  

  

|参数|描述|
|:--|:--|
|column_name|必需。要舍入的字段。|
|decimals|可选。规定要返回的小数位数。|

  

---

  

---

## SQL ROUND() 实例

**ROUND(X)：** 返回参数X的四舍五入的一个整数。

## 实例

mysql> SELECT ROUND(-1.23);  
        -> -1  
mysql> SELECT ROUND(-1.58);  
        -> -2  
mysql> SELECT ROUND(1.58);  
        -> 2  

**ROUND(X,D)：** 返回参数X的四舍五入的有 D 位小数的一个数字。如果D为0，结果将没有小数点或小数部分。

## 实例

```sql
mysql> SELECT ROUND(1.298, 1);  
        -> 1.3  
mysql> SELECT ROUND(1.298, 0);  
        -> 1  
```

注意：ROUND 返回值被变换为一个BIGINT!

# SQL ORDER BY 关键字

---

ORDER BY 关键字用于对结果集进行排序。

---

## SQL ORDER BY 关键字

ORDER BY 关键字用于对结果集按照一个列或者多个列进行排序。

ORDER BY 关键字默认按照升序对记录进行排序。如果需要按照降序对记录进行排序，您可以使用 `DESC `关键字。

### SQL ORDER BY 语法

SELECT column1, column2, ...
FROM table_name
ORDER BY column1, column2, ... ASC|DESC;

- **column1, column2, ...**：要排序的字段名称，可以为多个字段。
- **ASC**：表示按升序排序。
- **DESC**：表示按降序排序。