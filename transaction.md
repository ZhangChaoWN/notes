# 数据库事务笔记

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
