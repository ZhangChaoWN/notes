# 数据库

## 关系型数据库概念的提出

根据维基百科，relational database 这个术语是1970年 IBM 的 E. F. Codd 在论文他的论文《a relational model of data for large shared data banks 中提出的》。

下载地址：https://www.seas.upenn.edu/~zives/03f/cis550/codd.pdf

## 事务

标准规范：MySQL 5.7 和Postgre SQL 都引用到 ISO/IEC 9075:1992，数据库的实现大多会遵循ISO的标准，ISO标准发布后是要付费购买的，出于学习目的，可以选择一种经济的途径来阅读标准文档--阅读读不需要付费的公开发布的标准草案，这个链接是SQL标准草案的一个版本<a href="http://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt">ISO SQL 92 standard</a></p>

[这里可以下载part 1 framework](http://standards.iso.org/ittf/PubliclyAvailableStandards/c053681_ISO_IEC_9075-1_2011.zip)

ISO SQL 1992 标准提到：
<blockquote>
    <pre>
An SQL-transaction has an isolation level that is READ UNCOMMITTED,
READ COMMITTED, REPEATABLE READ, or SERIALIZABLE. The isolation
level of an SQL-transaction defines the degree to which the opera-
tions on SQL-data or schemas in that SQL-transaction are affected
by the effects of and can affect operations on SQL-data or schemas
in concurrent SQL-transactions. The isolation level of a SQL-
transaction is SERIALIZABLE by default. The level can be explicitly
set by the &lt;set transaction statement&gt;.
    </pre>
</blockquote>
标准规范中，默认隔离级别是SERIALIZABLE，实际上，MySQL和postgreSQL默认的隔离级别都是reapteable read。（问题：为什么？是不是为了性能？）

历史： [A History of Transaction Histories](https://ristret.com/s/f643zk/history_transaction_histories")

问题：这篇论文：<a href="https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tr-95-51.pdf">A Critique of ANSI SQL Isolation Levels</a> 对ANSI SQL的隔离级别提出了一些问题，目前，主流的数据库有尝试解决论文中提出的问题吗？如何解决的？

问题：数据库是如何是现原子性与持久性的？机器断电之后，会不会出现，buffer没有写入存储器的情况？搜索引擎上搜到了stackover flow上的这个问题。<a href="https://stackoverflow.com/questions/2009063/are-disk-sector-writes-atomic">Are disk sector writes atomic?</a>

上面那个stackoverflow回答中提到了这个postgresql 邮件列表的<a href="https://www.postgresql.org/message-id/flat/4AFD5517.8030207%40shopzeus.com#4AFD5517.8030207@shopzeus.com">讨论</a>

市面上有很多的HDD硬盘带有缓存功能，所以，这种HDD会把操作系统发送给HDD的内容缓存起来，并不会真的写入的磁盘中去。我记得在哪里看到一种说法，硬盘中有电容，这个电容的作用时，即使计算机断电，这个电容中也有足够的电能把缓存中的内容写入到磁盘中去。

<ul>
    <li><a href="https://modern-sql.com/standard">modern-sql.com/standard</a></li>
    <li><a href="https://www.wiscorp.com/SQLStandards.html">www.wiscorp.com/SQLStandards.html</a></li>
    <li><a href="https://ristret.com/s/f643zk/history_transaction_histories"> A History of Transaction Histories</a></li>
    <li><a href="http://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt">ISO SQL 92 standard</a></li>
    <li>
        <a href="https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tr-95-51.pdf">
            A Critique of ANSI SQL Isolation Levels
        </a>
    </li>
    <li>
        <a href="https://stackoverflow.com/questions/2009063/are-disk-sector-writes-atomic">Are disk sector writes atomic?</a>
    </li>
    <li>
        <a href="https://www.postgresql.org/docs/current/static/transaction-iso.html">PostgreSQL文档</a>
    </li>
    <li>
        <a href="https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.html">
            InnoDb文档
        </a>
    </li>
</ul>

## 问题：Mysql创建索引是否会锁表？是否会影响业务？
参考资料:
https://dev.mysql.com/doc/refman/5.6/en/innodb-online-ddl-operations.html

mysql文档中的描述：
> Creating or adding a secondary index
> CREATE INDEX name ON table (col_list);
> ALTER TABLE tbl_name ADD INDEX name (col_list);
> The table remains available for read and write operations while the index is being created. The CREATE INDEX statement only finishes after all transactions that are accessing the table are completed, so that the initial state of the index reflects the most recent contents of the table.
来源：https://dev.mysql.com/doc/refman/5.6/en/innodb-online-ddl-operations.html

根据mysql 文档的说明，创建“secondary index”的时候，表数据可读可写不会阻塞。
问题：什么是secondary index？根据文档中的描述，secondary index的意思好像就是非主键的index，需要进一步查资料确认。

> InnoDB tables always have a clustered index representing the primary key. They can also have one or more secondary indexes defined on one or more columns. Depending on their structure, secondary indexes can be classified as partial, column, or composite indexes.
来源： https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_index

> * When you define a PRIMARY KEY on your table, InnoDB uses it as the clustered index. Define a primary key for each table that you create. If there is no logical unique and non-null column or set of columns, add a new auto-increment column, whose values are filled in automatically.
> * If you do not define a PRIMARY KEY for your table, MySQL locates the first UNIQUE index where all the key columns are NOT NULL and InnoDB uses it as the clustered index.
> * If the table has no PRIMARY KEY or suitable UNIQUE index, InnoDB internally generates a hidden clustered index named GEN_CLUST_INDEX on a synthetic column containing row ID values. The rows are ordered by the ID that InnoDB assigns to the rows in such a table. The row ID is a 6-byte field that increases monotonically as new rows are inserted. Thus, the rows ordered by the row ID are physically in insertion order.
...
> All indexes other than the clustered index are known as secondary indexes.
来源：https://dev.mysql.com/doc/refman/8.0/en/innodb-index-types.html

非cluster key的索引都是secondary key.如果有主键，mysql把主键作为clustered index, 如果没有主键，mysql会按照规则选择一个索引或生成一个隐藏的索引作为clustered key。

## Cardinality

> An estimate of the number of unique values in the index. To update this number, run ANALYZE TABLE or (for MyISAM tables) myisamchk -a.

> Cardinality is counted based on statistics stored as integers, so the value is not necessarily exact even for small tables. The higher the cardinality, the greater the chance that MySQL uses the index when doing joins.

## mysql 缓存
mysql其实有缓存，测试一条语句的性能的时候，有可能测的是缓存。
url: https://dev.mysql.com/doc/refman/5.6/en/query-cache.html

## mysql 优化

mysql优化文档： url: https://dev.mysql.com/doc/refman/5.7/en/optimization.html

### 通过limit关键字分页，靠后的页数查询存在的问题

B+树查找时间复杂度是O(log(n))。如果一个查询语句where条件走了B+树索引，explain结果中的rows会非常小。
但是如果where条件中没有走索引，或者按索引字段筛选后结果集依然很大，explain的rows会很大。这种情况下如果使用了limit关键字分页，而且order by字段走了索引，则分页越靠前查询越快，分页越靠后查询越慢，比如说limit 8000, 200, explain rows可能是扫描8200行。

## EAV 模型

EAV模型：RDBMS中，一种表结构设计模式，也是一种数据模型，通过三个字段：Entity ID, Attribute name, value存储一些程序设计开发阶段没法确定属性名称的属性值。

优点：
表结构稳定，数据的key非常多变且无法固定的情况下，不需要频繁维护表结构。

缺点：
数据量大的情况下，也许不容易做性能优化（如果按多个key筛选、排序、怎么加索引是个问题，属性表中的字段与主表中的字段如何加索引，也是个问题。）
数据模型不直观，表结构不是很符合开发者习惯，编写查询语句也许相对更难一些，程序逻辑的理解成本也许相对更高一些。

总结：这些扩展属性如果不需要作为查询条件，不需要作为排序条件，仅仅作为查询结果能被关联出来就可以的情况下，可以考虑使用这种模式，但是表结构一旦确定，以后想要改的时候，成本可能会很高，很多sql需要重写。所以应当慎重考虑这种模型。

另外数据量非常小，且很肯定未来不会增长，或者做好了数据增长后重新设计的准备，也可以考虑采用这种模型。

这种参数作为查询条件，如果能保证所有的查询都可以通过主表中的字段把结果缩小至很小的范围，可以考虑使用这种模型。
