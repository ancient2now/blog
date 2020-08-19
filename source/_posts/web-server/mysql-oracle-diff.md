---
title: Oracle 与 MySQL 的差异
date: 2020-08-19 19:38:57
tags: [MySQL, Oracle]
categories: Web Server
---

有个项目中涉及到将数据存储从 Oracle 迁移到 MySQL ，有些语法的改动，记录下收藏的语法差异，之后补充 PostgreSQL 的语法差异。

<!--more-->

## dual特殊表

Oracle:

```sql
select 1 from dual
```

MySQL:

```sql
select 1
```

## null | 空字符串 | 0

> 对于 Oracle 而言，空字符串与 NULL 一样。对于 MySQL 底层存储而言，没有空字符串，只有 NULL。

### 1. substr

substr 在 Oracle 与 MySQL 中意义基本一致，除了 位置 参数：

对于位置 0，Oracle 认为与 1 一样，而 MySQL 返回 空字符串

另外，对于空字符串，Oracle 与 NULL 一样

| Oracle | MySQL |
|-|-|
| substr(x, 0)   | nullif(substr(x, 1), '')  | 或者直接把原始版本也改为 0 |
| substr(x, pos) | nullif(substr(x, pos), '')| |
| substr(x, pos, length) | nullif(substr(x, pos, length), '') | |

### 2. trim

Oracle 与 MySQL 的 trim 一样，但是需要注意空字符串问题，所以可以加入一个 nullif。

Oracle:

```sql
trim(x)
```

MySQL:

```sql
nullif(trim(x), '')
```

## nvl | nvl2 | ifnull

`NVL(expr1, expr2)`：如果 expr1 不为 null，返回 expr1；否则返回 expr2。

`NVL2(expr1, expr2, expr3)`：如果 expr1 不为 null，返回 expr2；否则返回 expr3。

| Oracle | Mysql | 备注 |
|-|-|-|
| nvl(a, b)     | ifnull(a, b)            | coalesce 返回第一个不为空的值，可以代替 nvl 及 isnull |
| nvl2(a, b, c) | if(a is not null, b, c) | |

## 排序的字段有null值

Oracle：

null值默认是最后，可以使用`nulls first`和`nulls last`来控制，mysql没有这两个语法

```sql
select *
from company
order by add_time desc nulls last
```

MySQL：

mysql可以使用`isnull`将null值和非null值分开来，再实现排序。

**注意:** 对于数字型，可以使用 MySQL 特定语法，也就是`null`比任何数字都小。

```sql
select *
from company
order by isnull(add_time), add_time desc
```

## 日期操作

当前时间，oracle使用`sysdate`,mysql使用`now()`

### 1. sysdate +/- 数字

加减数字，这个数字默认单位是 天，所以，实际使用时，可能会遇到以下值：

* 1 / 24，一天 24 小时，所以这是小时
* 1 / 1440，一天 1440 分钟，所以这是分钟
* 其它值，请根据实际值转换成 小时，分钟 等其它值

Oracle：

```sql
SELECT *
FROM company
WHERE add_time > SYSDATE - 5 / 1440;
```

MySQL：

5 / 1440，可以看出来是 5 分钟，所以使用 interval 5 minute。

示例使用interval -5 minute，及 date_add，建议使用 date_sub 代替 date_add 与 负数。

```sql
SELECT *
FROM company
WHERE add_time > date_add(now(), interval - 5 minute)
```

### 2. sysdate - date 或 date - sysdate

oracle：

```sql
select (expire_time - sysdate) > 60
```

mysql：

```sql
select DATEDIFF(expire_time, now()) > 60
```

### 3. add_months

`add_months` 可以直接使用 `date_add` / `date_sub`，但是在月末时，Oracle 与 MySQL 有差异。

* 月末，Oracle 总是返回月末值，如 2020-04-30 加减 1 个月，分别是 2020-05-31 及 2020-03-31
* 而 MySQL 就是直接 加减 1 个月，只有超出时，才调整。

### 4. 字符串转成日期

Oracle:

```sql
to_date(?, 'yyyy-mm-dd hh24:mi:ss')
```

MySQL:

```sql
str_to_date(?, '%Y-%m-%d %H:%i:%s')
```

常见格式对照表：

| Oracle (不区分大小写) | 解释 | MySQL (区分大小写) | 备注 |
|-|-|-|-|
| YYYY | 4位年 | %Y | 还有SYYYY，公元前使用 负数 |
| YY   | 2位年 | %y | |
| MM   | 月    | %m | |
| DD   | 日    | %d | |
| HH24 | 时（0-23）| %H | |
| HH   | 时（1-12）| %h| HH12 与 HH 一样 |
| MI   | 分（0-59）| %i| |
| SS   | 秒（0-59）| %s| |

### 小总结

* sysdate + 数字，使用 date_add
* sysdate - 数字，使用 date_sub
* sysdate - date 或 date - sysdate，使用 datediff

## to_char

oracle有几下三种格式

### 1. to_char(character)

对应mysql,使用`cast x as char` 或者 `convert`

### 2. to_char(datetime [, fmt])

没有 fmt 的话，使用默认格式,如果有 fmt 的话，使用 `date_format`

### 3. to_char(number)

没有 fmt 的话，就是把数字转成字符，使用 `cast x as char` 或者  `convert(x, char)`，也可以使用 `concat(x)` [不建议],有 fmt 的话，转成特定格式

示例：

Oracle：

```sql
to_char(expire_time, 'yyyy-MM-dd')
```

MySQL:

```sql
date_format(report_end_date, '%Y-%m-%d')
```

## to_number

Oracle：

```sql
to_number(x)
```

MySQL:

```sql
cast(x as signed)
```

## to_date

看[日期操作](#日期操作)章节

## decode

Oracle 和 MySQL 都有decode函数，但是含义完全不一样。

在 Oracle 下，decode 类似 switch-case，如:

```sql
decode(a, 1, 2, 3, 4, 5)
```

意思就是 a 为 1 时，返回 2；为 3 时，返回 4；其它返回 5。

对应Oracle的decode功能，MySQL可以使用 case-when。有两种风格：

```sql
select case 'a' when 1 then 2 when 3 then 4 else 5 end;
```

或者

```sql
select case when 'a' = 1 then 2 when 'a' = 3 then 4 else 5 end;
```

如果涉及 null，需要使用 is null 判断，只能使用第二个风格；如果条件较少，直接使用 if 即可。

## || -> concat_ws

Oracle 中，空字符串与 NULL 等价；

Oracle 中的 || 等同于 concat，但略有差异，主要是 null。

在 Oracle 中，字符串 与 null 拼接，不会返回 null；而 MySQL 返回 null。

```sql
-- Oracle:
select 'a' || null from dual;
-- MySQL:
select concat('a', null);
```

使用 concat_ws ，会跳过 null 值。

```sql
select concat_ws('', 'a', null);
```

最佳方案：

* 通用情况：`a || b` => `nullif(concat_ws('', a, b), '')`
* 特殊情况：`a || ''` => `nullif(concat_ws('', a), '')`，也就是接着的字符串在 MySQL 中是空字符串，可以直接去掉
* 如果确定没有 null 值（比如作为查询参数），可以直接使用 concat函数

测试用例:

|Oracle|Mysql|结果|
|-|-|-|
| 'a' &#124;&#124; 'b'   | nullif(concat_ws('', 'a', 'b'), '')  | 'ab' |
| 'a' &#124;&#124; null  | nullif(concat_ws('', 'a', null), '') | 'a'  |
| null &#124;&#124; 'b'  | nullif(concat_ws('', null, 'b'), '') | 'b'  |
| null &#124;&#124; null | nullif(concat_ws('', null, null), '')| null |

特殊场景：

|Oracle|Mysql|
|-|-|
| a &#124;&#124; '' | concat_ws('', a) |

解释一下：上面的特殊场景可以理解为，如果 a 不是 null，返回 a；否则返回空字符串

## with语句

MySQL 8.0 才支持 with 语句。

## (+) / left join

在 Oracle 中，可能使用 comma join，然后在 where 中使用 (+) 表示实际的 join。 加号在哪边，哪边就可能有 null，然后对应使用 left / right join

mysql直接使用join关键字

## 窗口函数 row_number / rank

MySQL 5.7 不支持窗口函数，且性能也并不太好。窗口函数多用于实现 分组 TOP 1 及 TOP N 问题。

MySQL8已经支持

## length | char_length

> Oracle 中，空字符串与 NULL 等价

Oracle 中  length 表示 字符串 的长度，与 MySQL 的 char_length 等价，但是对于 空字符串 略有差异。

| 字符串 | Oracle | MySQL |
|-|-|-|
| null | null | null |
| (空) | null | 0    |

## instr | locate

Oracle 的 instr 可能支持多个参数，如：

* instr(string, substring)
* instr(string, substring, postion)
* instr(string, substring, position, occurrence)

| Oracle | MySQL | 备注 |
|-|-|-|
| instr(string, substring) | instr(string, substring) 或 locate(substring, string) | MySQL推荐使用 instr |
| instr(string, substring, position) | locate(substring, string, position) | |
| instr(string, substring, position, occurrence) | - | |

## trunc(date), trunc(number)

| Oracle | MySQL | 备注 |
|-|-|-|
| trunc(数字) | truncate(数字) | |
| trunc(数字，小数位数) | truncate(数字，小数位数) | 小数位数可为负 |
| trunc(日期) | date(日期) | |
| trunc(日期，格式) | date_format(日期，格式) + cast | date_format 返回字符格式 |

一些常见的日期操作场景，如下：

| 说明 | Oracle (不区分大小写) | MySQL (区分大小写) | 备注 |
|-|-|-|-|
| 到月 | TRUNC(x, 'MM') | cast(date_format(x, '%Y-%m-01') as date) | 非返回数据可以略过 cast |
| 到日 | TRUNC(x) 或 TRUNC(x, 'DD') | date(x) | |
| 到时 | TRUNC(x, 'HH') | cast(date_format(x, '%Y-%m-%d %H:00:00') as datetime) | 非返回数据可以略过 cast |
| 到分 | TRUNC(x, 'MI') | cast(date_format(x, '%Y-%m-%d %H:%i:00') as datetime) | 非返回数据可以略过 cast |

## 正则匹配

Oracle:

```sql
select *
from company
where regexp_like(KEYWORD, '^[0-9I]');
```

MySQL:

```sql
select *
from company
where KEYWORD regexp '^[0-9Ii]';
```

## 随机排序

* `dbms_random.vaule`：生成一个指定范围的38位随机小数（小数点后38位），若不指定范围则默认为范围为[0,1)的随机数。
* `dbms_random.random`：生成一个从[-2^31, 2^31)的整数值，注意，区间为左闭右开。
* `dbms_random.normal`：生成一个符合正态分布的随机数，此正态分布标准偏差为1，期望值为0。这个函数返回的数值中有68%是介于-1与+1之间， 95%介于-2与+2之间，99%介于-3与+3之间。

Oracle:

```sql
select *
from company
where address='china'
order by dbms_random.value
```

MySQL:

```sql
select *
from company
where address='china'
order by RAND()
```
