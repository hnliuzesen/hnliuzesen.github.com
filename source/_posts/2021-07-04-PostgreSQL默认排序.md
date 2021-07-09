---
title: PostgreSQL 默认排序
date: 2021-07-04 21:04:28
categories:
- Develop
- Database
tags:
- PostgreSQL
---

刚刚接触到号称功能最强大的开源数据库 PostgreSQL，发现在不加 ORDER BY 的情况下每次查询返回的结果顺序都不一致，推测以下两种情况可能会导致。

## JOIN 导致结果顺序随机
第一个遇见的每次查询结果顺序都不一致的 SQL 大致如下：
```PostgreSQL
SELECT a.id, b.id, c.id
FROM table_a a
         INNER JOIN table_b b on a.b_id = b.id
         INNER JOIN table_c c on a.c_id = c.id
LIMIT 20
```

<!--more-->

开始以为是 LIMIT 导致的，因为最早发现排序
差异的时候是 LIMIT 10 和 LIMIT 1000 的前几条顺序不一样，后来换了 pgAdmin 4 后发现，不改变 LIMIT 
的大小，每次点执行，顺序也都会有变化。通过 pgAdmin 4 里面的查询分析，发现先是使用了索引，后进行了 join。

{% asset_img "explain-of-join.webp" %}

记得之前在网上看到 MySQL 在 join 的时候，如果是用了 Block Nested Loop Join，则会生成一个 join buffer 作为缓冲区，然后批量匹配进行 
join。既然是批量执行，那顺序自然得不到保证，不过不能确定 PostgreSQL 是不是和 MySQL 
的原理一致，最后在官方文档中找到了相关答案。在 [7.5. Sorting Rows](https://www.postgresql.org/docs/current/queries-order.html#:~:text=The%20actual%20order%20in%20that%20case%20will%20depend%20on%20the%20scan%20and%20join%20plan%20types%20and%20the%20order%20on%20disk%2C%20but%20it%20must%20not%20be%20relied%20on.) 
中提到，depend on the scan and join plan types and the order on disk，取决于扫描、join 
和数据在磁盘上的顺序，这中间没有操作过数据，在磁盘上的顺序应该是没有变，剩下最大的可能就是 join 导致的。通过粗略查看 pg join 
的源码 [nodeHashjoin.c](https://doxygen.postgresql.org/nodeHashjoin_8c_source.html) 发现 Hash joins can participate in 
parallel query execution in several ways。意思是 hash join 会并行执行。后面没有再继续追查这个 nodeHashjoin 是否就是上面 SQL 
使用的 join。

## LIMIT 影响结果顺序
另一个查询不一致的情况并没有使用 join，只是一个简单的单表查询，如果 SQL 不变顺序不变，但是改变了 LIMIT 
的值之后，顺序就会发生变化，经过多次测试，发现变化的阈值是 14，查询 13 条数据和查询 14 条数据的排序必定会不一样。在网上搜索之后还是找到文档 
[7.6. LIMIT and OFFSET](https://www.postgresql.org/docs/8.1/queries-limit.html#:~:text=The%20query%20optimizer%20takes%20LIMIT%20into%20account%20when%20generating%20a%20query%20plan)，
The query optimizer takes LIMIT into account when generating a query plan，意思是 pg 的查询优化器会通过 LIMIT 
的值来生成查询计划，看到这句话就有方向了，直接对两条语句进行 EXPLAIN 分析查询计划得知，在数据量少的时候是使用的索引，在数据量大的时候，使用的 
bitmap，导致排序表现不一致。

{% asset_img explain-limit-13.webp "LIMIT 13" %}

{% asset_img explain-limit-14.webp "LIMIT 14" %}