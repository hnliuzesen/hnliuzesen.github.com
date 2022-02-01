---
title: PostgreSQL将字符串类型的列转为数组
date: 2022-02-01 00:58:29
categories:
- Develop
- Database
tags:
- PostgreSQL
---

在学习 PostgreSQL 的时候，对于数组类型，之前是直接用了 varchar(2550) 来存储，想将其转换成 bigint[]，期间遇见了 DDL
语句处理已存在的数据无法使用 SELECT 和转换过程中遇见不符合规范的数据会报错之类的一些问题，这里是一些问题解决的方案，和更改类型前后的对比。

## 仅修改字段类型的语句
如果只是修改字段类型的话很简单，使用普通的 DDL 就行。参考 [ALTER TABLE](https://www.postgresql.org/docs/current/sql-altertable.html)
```PostgreSQL
ALTER TABLE <table>
    ALTER COLUMN <column> TYPE bigint[];
```

<!--more-->

## 修改字段类型同时处理可以强转的数据
但是之前的数据是 `[1,2,3]` 的形式，在转换的时候会提示

`ERROR: column "<column>" cannot be cast automatically to type bigint[]` `Hint: You might need to specify "USING <column>::bigint[]"`

```PostgreSQL
ALTER TABLE <table>
    ALTER COLUMN <column> TYPE bigint[] USING <column>::bigint[];
```

## 修改字段类型同时处理不能强转的数据
按提示修改后执行会再次报错

`ERROR: malformed array literal: "[1,2,3]"` `Detail: Missing "]" after array dimensions.`

因为 `[1,2,3]` 形式的数据无法自动转换为 bigint[] 类型，如果当初使用了 `{1,2,3}` 就可以顺利转换了。此时能想到的只有 2 种处理方案，先把所有的
`[]` 都用字符串函数替换成 `{}`，然后再执行修改语句。或者在执行修改语句的时候使用 `USING` 处理。第一种方法要对所有的数据处理两遍，同时如果处理的不是
bigint 类型的数据，而是字符串，本身就包含有 `[]{}` 的字符串数组，就会有问题。

这里思路是先能把查询结果展示为数组，然后放到 `USING` 语句中。经过尝试使用
`SELECT array_agg(x)::bigint[] || ARRAY []::bigint[] FROM <table>, jsonb_array_elements_text(<table>.<column>::jsonb) t(x);`
可以正确返回数组类型的查询结果 `{1,2,3}`，放入 `USING` 字句中

```PostgreSQL
ALTER TABLE <table>
    ALTER COLUMN <column> TYPE bigint[] USING (SELECT array_agg(x)::bigint[] || ARRAY []::bigint[]
                                               FROM jsonb_array_elements_text(<column>::jsonb) t(x));
```

此时报错 `ERROR: cannot use subquery in transform expression`，提示在类型转换中不能使用子查询，因为这里是先把 `varchar` 强转 `jsonb`
后通过 `jsonb_array_elements_text()` 展开成一组文本，然后再通过 `array_agg()` 聚集函数收集到数组中，如果出错就直接返回空数组。这里因为对
PostgreSQL 还是不够熟悉，不知道该怎么用不是查询的方式将这一列字段转换为数组类型，也没有深入研究，直接选择了把查询创建成函数来解决这个报错。

```PostgreSQL
-- 创建转换 varchar 数组为 bigint 数组的函数
CREATE OR REPLACE FUNCTION varchar_arr2bigint_arr(arg varchar)
    RETURNS bigint[]
    IMMUTABLE PARALLEL SAFE AS $$
        BEGIN
            RETURN (SELECT ARRAY(SELECT jsonb_array_elements_text(arg::jsonb)::bigint));
        EXCEPTION WHEN OTHERS THEN
            RETURN (SELECT ARRAY[]::bigint[]);
        END;
    $$ LANGUAGE plpgsql;

-- 在 ALTER TABLE 中使用函数来进行类型转换
ALTER TABLE <table>
    ALTER COLUMN <column> TYPE bigint[] USING varchar_arr2bigint_arr(<column>::jsonb);
```

这个函数主要内容就是 `SELECT ARRAY(SELECT jsonb_array_elements_text(arg::jsonb)::bigint)`，因为之前存储类型是 varchar
，所以有可能一些数据不是 `[1,2,3]` 这种类型的，这种情况下这条语句会抛异常，对于不合法的数据，直接返回一个空数组。

因为学艺不精，需要多一步创建函数的操作，我觉得肯定有不创建函数直接在 `USING` 中解决的方法，后面学习到了再补充。

## 转换为原生数组类型的好处
`varchar` 类型存储一个数组的字符串形式，可以正常使用，转换成原生数组类型有什么收益呢？

### 性能
首先第一反应就是原生类型会不会更快。通过 `EXPLAIN ANALYZE` 来分析前后查询包含指定值的 SQL，Planning Time 变化不大，Execution Time
基本都有 50% 左右的性能提升。

#### 修改前
{% asset_img before.webp "BEFORE" %}

#### 修改后
{% asset_img after.webp "AFTER" %}

### 查询便利
性能上 50% 的提升，其实只是从 0.238 ms 降到了 0.090 ms 之类的，实际一条语句快了不到 1 毫秒。但是查询语句前后的变化，就很有价值了。

#### 查询数组中含有指定值的 SQL

```PostgreSQL
-- 修改之前
SELECT *
FROM <table>
WHERE '1' = ANY (string_to_array(replace(btrim(<column>, '[''['', '']'']'), ' ', ''), ','));

-- 修改之后
SELECT *
FROM <table>
WHERE <column> @> ARRAY [1::bigint];
```

#### 查询数组中与查询条件有交集的 SQL

```PostgreSQL
-- 修改之前是下面对查询条件用 <foreach separator=“ OR ”> 来循环拼接
SELECT *
FROM <table>
WHERE <column>::jsonb @> '{1|2|3}'::jsonb;

-- 修改之后只需要下面一句
SELECT *
FROM <table>
WHERE <column> && ARRAY [1::bigint, 2::bigint, 3::bigint];
```

是不是 SQL 写起来变爽了？机器的 CPU 时间可比开发人员的工时便宜太多了。
