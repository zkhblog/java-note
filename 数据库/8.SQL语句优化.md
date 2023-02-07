```
https://mp.weixin.qq.com/s/mhW_y_tdW2_EGrpCohMAhQ
https://mp.weixin.qq.com/s/cOvmXQYqe0CyAHjo-3-BjA
https://mp.weixin.qq.com/s/veLas1JX8kN0BXfKv2BXqw
https://www.cnblogs.com/qluzzh/p/10782993.html
```

1、对查询进行优化，应尽量避免全表扫描，首先应考虑在 WHERE 及 ORDER BY 涉及的列上建立索引。  
2、应尽量避免在 WHERE 子句中对字段进行 NULL 值判断，创建表时 NULL 是默认值，但大多数时候应该使用 NOT NULL，或者使用一个特殊的值，如 0，-1 作为默认值。  
3、应尽量避免在 WHERE 子句中使用 != 或 <> 操作符。MySQL 只有对以下操作符才使用索引：<，<=，=，>，>=，BETWEEN，IN，以及某些时候的 LIKE。  
4、应尽量避免在 WHERE 子句中使用 OR 来连接条件，否则将导致引擎放弃使用索引而进行全表扫描， 可以使用 UNION 合并查询：select id from t where num=10 union all select id
from t where num=20。  
5、IN 和 NOT IN 也要慎用，否则会导致全表扫描。对于连续的数值，能用 BETWEEN 就不要用 IN：select id from t where num between 1 and 3。  
6、下面的查询也将导致全表扫描：select id from t where name like‘%abc%’ 或者select id from t where name like‘%abc’若要提高效率，可以考虑全文检索。而select
id from t where name like‘abc%’才用到索引。  
7、如果在 WHERE 子句中使用参数，也会导致全表扫描。  
8、应尽量避免在 WHERE 子句中对字段进行表达式操作，应尽量避免在 WHERE 子句中对字段进行函数操作。  
9、很多时候用 EXISTS 代替 IN 是一个好的选择：select num from a where num in(select num from b)。用下面的语句替换：select num from a where exists(
select 1 from b where num=a.num)。  
10、索引固然可以提高相应的 SELECT 的效率，但同时也降低了 INSERT 及 UPDATE 的效。因为 INSERT 或 UPDATE 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况而定。一个表的索引数最好不要超过 6
个，若太多则应考虑一些不常使用到的列上建的索引是否有必要。  
11、应尽可能的避免更新 clustered 索引数据列， 因为 clustered 索引数据列的顺序就是表记录的物理存储顺序，一旦该列值改变将导致整个表记录的顺序的调整，会耗费相当大的资源。若应用系统需要频繁更新 clustered
索引数据列，那么需要考虑是否应将该索引建为 clustered 索引。  
12、尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。  
13、尽可能的使用 varchar, nvarchar 代替 char, nchar。因为首先变长字段存储空间小，可以节省存储空间，其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些。  
14、最好不要使用返回所有：select from t ，用具体的字段列表代替 “*”，不要返回用不到的任何字段。  
15、尽量避免向客户端返回大数据量，若数据量过大，应该考虑相应需求是否合理。  
16、使用表的别名（Alias）：当在 SQL 语句中连接多个表时，请使用表的别名并把别名前缀于每个 Column 上。这样一来，就可以减少解析的时间并减少那些由 Column 歧义引起的语法错误。  
17、使用“临时表”暂存中间结果 ：
> 简化 SQL 语句的重要方法就是采用临时表暂存中间结果。但是临时表的好处远远不止这些，将临时结果暂存在临时表，后面的查询就在 tempdb 中了，这可以避免程序中多次扫描主表，也大大减少了程序执行中“共享锁”阻塞“更新锁”，减少了阻塞，提高了并发性能。

18、一些 SQL 查询语句应加上 nolock，读、写是会相互阻塞的，为了提高并发性能。对于一些查询，可以加上 nolock，这样读的时候可以允许写，但缺点是可能读到未提交的脏数据。
> 使用 nolock 有3条原则：  
①查询的结果用于“插、删、改”的不能加 nolock；  
②查询的表属于频繁发生页分裂的，慎用 nolock；  
③使用临时表一样可以保存“数据前影”，起到类似 Oracle 的 undo 表空间的功能，能采用临时表提高并发性能的，不要用 nolock。

19、常见的简化规则如下：
> 不要有超过 5 个以上的表连接（JOIN），考虑使用临时表或表变量存放中间结果。少用子查询，视图嵌套不要过深，一般视图嵌套不要超过 2 个为宜。

20、将需要查询的结果预先计算好放在表中，查询的时候再Select。这在SQL7.0以前是最重要的手段，例如医院的住院费计算。 21、用 OR 的字句可以分解成多个查询，并且通过 UNION
连接多个查询。他们的速度只同是否使用索引有关，如果查询需要用到联合索引，用 UNION all 执行的效率更高。多个 OR 的字句没有用到索引，改写成 UNION 的形式再试图与索引匹配。一个关键的问题是否用到索引。  
22、在IN后面值的列表中，将出现最频繁的值放在最前面，出现得最少的放在最后面，减少判断的次数。  
23、尽量将数据的处理工作放在服务器上，减少网络的开销，如使用存储过程。
> 存储过程是编译好、优化过、并且被组织到一个执行规划里、且存储在数据库中的 SQL 语句，是控制流语言的集合，速度当然快。反复执行的动态 SQL，可以使用临时存储过程，该过程（临时表）被放在 Tempdb 中。

24、当服务器的内存够多时，配制线程数量 = 最大连接数+5，这样能发挥最大的效率；否则使用配制线程数量< 最大连接数，启用 SQL SERVER 的线程池来解决，如果还是数量 = 最大连接数+5，严重的损害服务器的性能。
25、查询的关联同写的顺序：

```
select a.personMemberID, * from chineseresume a,personmember b where personMemberID = b.referenceid and a.personMemberID = 'JCNPRH39681' （A = B, B = '号码'） 
select a.personMemberID, * from chineseresume a,personmember b where a.personMemberID = b.referenceid and a.personMemberID = 'JCNPRH39681' and b.referenceid = 'JCNPRH39681' （A = B, B = '号码', A = '号码'） 
select a.personMemberID, * from chineseresume a,personmember b where b.referenceid = 'JCNPRH39681' and a.personMemberID = 'JCNPRH39681' （B = '号码', A = '号码'）
```

26、尽量使用 EXISTS 代替 select count(1) 来判断是否存在记录。count 函数只有在统计表中所有行数时使用，而且 count(1) 比 count(*) 更有效率。  
27、尽量使用 “>=”，不要使用 “>”。  
28、索引的使用规范：
> 索引的创建要与应用结合考虑，建议大的 OLTP 表不要超过 6 个索引；  
尽可能的使用索引字段作为查询条件，尤其是聚簇索引，必要时可以通过 index index_name 来强制指定索引；  
避免对大表查询时进行 table scan，必要时考虑新建索引；  
在使用索引字段作为条件时，如果该索引是联合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使用；  
要注意索引的维护，周期性重建索引，重新编译存储过程。

29、下列 SQL 条件语句中的列都建有恰当的索引，但执行速度却非常慢：



