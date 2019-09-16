# [MySql 执行计划解读](https://www.cnblogs.com/LQBlog/p/10723158.html)

**文章主目录**

*   [ 说明](https://www.cnblogs.com/LQBlog/p/10723158.html#_label0)
*   [mysql执行计划表结构](https://www.cnblogs.com/LQBlog/p/10723158.html#_label1)
*   [各个字段详解](https://www.cnblogs.com/LQBlog/p/10723158.html#_label2)
*   [如何查找mysql中的慢sql](https://www.cnblogs.com/LQBlog/p/10723158.html#_label3)

#  说明

解读执行计划对于慢sql的分析和调优有很大帮助,同时在解读的过程中也能知道如何规避慢sql

# mysql执行计划表结构

![](https://img2018.cnblogs.com/blog/779774/201904/779774-20190417104142787-1412321483.png)

# 各个字段详解

## 测试表结构说明

sl\_sales\_bill\_copy1 订单抬头表

sl\_sales\_bill\_copy1订单行项目表

order\_status 订单状态变动表

## id

执行顺序 值越大的优先执行 如果相同则根据顺序来定

#查出订单状态存在1010的订单的所有行项目信息
EXPLAIN select \* from sl\_sales\_bill\_copy1 lb join sl\_sales\_bill\_head\_copy1 lh on lh.SALES\_BILL\_NO \= lb.SALES\_BILL\_NO where lb.SALES\_BILL\_NO in(select ls.SALES\_BILL\_NO from order\_status ls where ls.status\_code\=1010)

![](https://img2018.cnblogs.com/blog/779774/201904/779774-20190417104847603-759623762.png)

个人理解:先查询ls保存为临时表 然后通过lb为驱动表关联查询非驱动表lh 得出 subquery2临时表 然后作为驱动表 去非驱动表ls查询数据

## select\_type

查询的类型，主要是用于区分普通查询、联合查询、子查询等复杂的查询

1、SIMPLE：简单的select查询，查询中不包含子查询或者union
2、PRIMARY：查询中包含任何复杂的子部分，最外层查询则被标记为primary
3、SUBQUERY：在select 或 where列表中包含了子查询
4、DERIVED：在from列表中包含的子查询被标记为derived（衍生），mysql或递归执行这些子查询，把结果放在零时表里
5、UNION：若第二个select出现在union之后，则被标记为union；若union包含在from子句的子查询中，外层select将被标记为derived
6、UNION RESULT：从union表获取结果的select

## type

查询的指标 结果从优到劣排序为

system > const > eq\_ref > ref > fulltext > ref\_or\_null > index\_merge > unique\_subquery > index\_subquery > range > index > ALL

**一般来说，好的sql查询至少达到range级别，最好能达到ref**

**system:**

const的特例平时无法重现 可以忽略

**const:**

表示通过扫描索引 扫描一行就找到了数据 const用于比较primary key 或者 unique索引。因为只需匹配一行数据，所有很快。如果将主键置于where列表中，mysql就能将该查询转换为一个const

![](https://img2018.cnblogs.com/blog/779774/201904/779774-20190417111206816-1072418733.png)

## eq\_ref

非驱动表关联字段为主键或者唯一索引查找时

lb.id为主键索引

![](https://img2018.cnblogs.com/blog/779774/201904/779774-20190417111709162-666810092.png)

如果where条件中查询非驱动表为非唯一索引或者主键索引则会降为ref

## ref

非驱动表关联字段为非唯一索引

lh.SALES\_BILL\_NO为非唯一索引

![](https://img2018.cnblogs.com/blog/779774/201904/779774-20190417112237222-1269923792.png)

## range

表示范围查询 常见于between 和>, >=,<, <= 前提是字段有建立btree索引

 count未建立索引执行计划

![](https://img2018.cnblogs.com/blog/779774/201904/779774-20190417112531959-958681177.png)

count建立索引后

![](https://img2018.cnblogs.com/blog/779774/201904/779774-20190417112637860-1339170264.png)

## index

与ALL类似 只是index全表扫描扫描的是索引页而不是数据行

![](https://img2018.cnblogs.com/blog/779774/201904/779774-20190417112859301-1548747561.png)

因为我们id做了索引 所以只需要去索引页里面取出所有id数据就好了

**ALL
**

全表扫描

## possible\_keys

指出MySQL能使用哪个索引在表中找到行，查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询使用

![](https://img2018.cnblogs.com/blog/779774/201904/779774-20190417142623552-1364834520.png)

## key

实际使用的索引，如果为NULL，则没有使用索引

## key\_len

表示索引中使用的字节数，查询中使用的索引的长度（最大可能长度），并非实际使用长度，理论上长度越短越好。key\_len是根据表定义计算而得的，不是通过表内检索出的

## ref

显示索引的那一列被使用了，如果可能，是一个常量const。

## rows

根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数

## **Extra**

包含不适合在其他列中显示但十分重要的额外信息

### Using index

表示使用了覆盖索引,覆盖索引:表示索引包含了返回和查询的所有列 而不需要读取文件

![](https://img2018.cnblogs.com/blog/779774/201904/779774-20190417143910205-58594453.png)

 如果查询和返回返回非索引字段

![](https://img2018.cnblogs.com/blog/779774/201904/779774-20190417144031153-544179431.png)

![](https://img2018.cnblogs.com/blog/779774/201904/779774-20190417144106736-782814594.png)

### Using where

Using where的作用只是提醒我们MySQL将用where子句来过滤结果集

![](https://img2018.cnblogs.com/blog/779774/201904/779774-20190417144401929-785676684.png)

### Using temporary

表示mysql需要临时表转存数据 常见于 group by使用非索引字段

![](https://img2018.cnblogs.com/blog/779774/201904/779774-20190417150354075-1080846679.png)

表示使用了非索引字段排序

### Using filesort

![](https://img2018.cnblogs.com/blog/779774/201904/779774-20190417150932707-815483741.png)

# 如何查找mysql中的慢sql

1.查看mysql是否开启mansql记录日志

show variables like 'slow\_query\_log';

2.慢sql记录时间

show variables like 'long\_query\_time';

3.设置记录mysql为打开状态

`set global slow_query_log='ON';OFF为关闭`

`设置超过一秒的sql都将记录`

`set global long_query_time=1
`

设置记录文件

set global slow\_query\_log\_file='/var/lib/mysql/test\_1116.log';

查看记录文件

show variables like 'slow\_query\_log\_file';

## 实际优化例子

## 需求

sl\_sales\_bill 订单抬头表

sl\_sales\_bill\_head 订单行项目表

order\_status 订单状态变动表(最新的为当前订单状态)

需求:查找开单未出库的指定产品的下单数量 订单状态为'1010', '1020', '1030', '1040', '1050'

## 3种查询重复数据取最新的一条的方式

## 方式1

EXPLAIN  SELECT \* FROM (SELECT \* FROM order\_status os ORDER BY os.CREATED\_DATE DESC) tab  GROUP BY  tab.SALES\_BILL\_NO

先不说这种方式性能如何 我当前版本sql必须在内部加limit 过滤一批数据才支持 个人觉得这种方式应该适合在一个表里面找一批数据的重复数据取最新 同时这一批数据知道条数 同时数量较小

SELECT \* FROM (SELECT \* FROM order\_status os ORDER BY os.CREATED\_DATE DESC  limit 1,10000) tab where tab.SALES\_BILL\_NO\='HP20190413002350' GROUP BY  tab.SALES\_BILL\_NO

## 方式2

select o.\* from(select SALES\_BILL\_NO,max(CREATED\_DATE) CREATED\_DATE from order\_status os where os.REGION\_CODE\='CB407'
group by SALES\_BILL\_NO)tab join order\_status o on o.SALES\_BILL\_NO\=tab.SALES\_BILL\_NO and o.CREATED\_DATE\=tab.CREATED\_DATE

要求日期同一批次数据 日期不能相同 最好是精确到毫秒6位   比如一个订单的订单状态变更记录的创建日期必须不等

### 方式3

select \* from order\_status o where    not EXISTS (select os.id from order\_status os where os.SALES\_BILL\_NO\=o.SALES\_BILL\_NO and os.CREATED\_DATE\>o.CREATED\_DATE)

通过not exists

## 现象有索引

orderStatus

![](https://img2018.cnblogs.com/blog/779774/201904/779774-20190418105725230-653604905.png)

 sl\_sales\_bill\_head

![](https://img2018.cnblogs.com/blog/779774/201904/779774-20190418105839604-142534691.png)

## 优化前sql

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

EXPLAIN select b.CHARACTER\_ID,s.FACTORY\_ID,sum(b.COUNT) as sum\_count from sl\_sales\_bill  b join sl\_sales\_bill\_head s on b.SALES\_BILL\_NO\=s.SALES\_BILL\_NO join (select o.\* from(select SALES\_BILL\_NO,max(CREATED\_DATE) CREATED\_DATE from order\_status os where os.REGION\_CODE\='CB407'
            group by SALES\_BILL\_NO)tab join order\_status o on o.SALES\_BILL\_NO\=tab.SALES\_BILL\_NO and o.CREATED\_DATE\=tab.CREATED\_DATE
            ) o on o.SALES\_BILL\_NO\=s.SALES\_BILL\_NO where b.CHARACTER\_ID in('754') and s.REGION\_CODE\='CB407'  and  (b.DEL\_FLAG\='0' or b.DEL\_FLAG is null or b.DEL\_FLAG\='') and o.STATUS\_CODE in('1010', '1020', '1030', '1040', '1050') group by CHARACTER\_ID,s.FACTORY\_ID

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

![](https://img2018.cnblogs.com/blog/779774/201904/779774-20190418105348439-1883570513.png)

通过type指标可以发现 伴随数据量的增大 性能会越来越慢

通过上面指标发现会随着order\_sataus表数据增大 sql性能会越慢  (依据是join匹配原理)

## 优化后

采用方式3

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

EXPLAIN select b.CHARACTER\_ID,s.FACTORY\_ID,sum(b.COUNT) as sum\_count from sl\_sales\_bill  b join sl\_sales\_bill\_head s on b.SALES\_BILL\_NO\=s.SALES\_BILL\_NO join order\_status o on o.SALES\_BILL\_NO\=s.SALES\_BILL\_NO where b.CHARACTER\_ID in('754') and o.STATUS\_CODE in('1010', '1020', '1030', '1040', '1050') and s.REGION\_CODE\='CB407'
                        and  (b.DEL\_FLAG\='0' or b.DEL\_FLAG is null or b.DEL\_FLAG\='') and  not EXISTS (select os.id from order\_status os where os.SALES\_BILL\_NO\=o.SALES\_BILL\_NO and os.CREATED\_DATE\>o.CREATED\_DATE) group by CHARACTER\_ID,s.FACTORY\_ID

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

![](https://img2018.cnblogs.com/blog/779774/201904/779774-20190418105628788-1337921566.png)

可以发现性能大大的提高了