---
layout: post
title: PostgreSQL一些记录
categories: sql
tag: tips
---

# 记录一些 PostgreSQL的基本操作

### 数据库连接

```
# psql -h 127.0.0.1 -p 45432 -U acunetix -W wvs
口令：

```

### 查看表结构

```
wvs-# \dt
                        关联列表
 架构模式 |           名称           |  类型  |  拥有者  
----------+--------------------------+--------+----------
 public   | admin_records            | 数据表 | acunetix
 public   | events                   | 数据表 | acunetix
.....
```

### 导出表 csv

参考 
> https://segmentfault.com/a/1190000008328676

```
COPY admin_records
TO '/path/to/output.csv'
WITH csv header;


# 指定属性
COPY products (name, price)
TO '/path/to/output.csv'
WITH csv header;

# 查询导出
COPY (
  SELECT name, category_name
  FROM products
  LEFT JOIN categories ON categories.id = products.category_id
)
TO '/path/to/output.csv'
WITH csv header;
```

### csv 导入表

```
COPY products
FROM '/path/to/input.csv'
WITH csv header;
```
