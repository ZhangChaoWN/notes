# 分布式计算

## 什么是分布式系统

Roy Fielding 在他的论文 Architectural Styles and the Design of Network-based Software Architectures 中引用了 Tanenbaum and van Renesse 的说法

> Tanenbaum and van Renesse [127] make a distinction between distributed systems and network-based systems: a distributed system is one that looks to its users like an ordinary centralized system, but runs on multiple, independent CPUs.

Tanenbaum and van Renesse 认为用户看起来像是普通集中式系统，但是运行在多个相互独立的CPU上的系统，称为分布式系统。

## 两阶段提交（2PC）

[正确理解二阶段提交（Two-Phase Commit）](https://blog.csdn.net/lengxiao1993/article/details/88290514)

这篇博文通过银行转账的场景，介绍了 2PC 的运用，这篇博文所所介绍的 2PC ，是业务逻辑层需要写代码的 2PC，需要业务逻辑层的代码写清楚 prepare 逻辑，commit 逻辑，以及 rollback 逻辑，所以说并不是一种对于业务逻辑代码透明的技术。

**问题**：2PC 方案能否做到对业务逻辑层透明？

查阅资料：
- mysql 文档中的 [XA Transactions 章节](https://dev.mysql.com/doc/refman/8.0/en/xa.html)
- mysql XA Transaction 基于 The Open Group 的标准规范：[Distributed Transaction Processing: The XA Specification](https://pubs.opengroup.org/onlinepubs/009680699/toc.pdf)

**问题**：“XA”是什么意思?

查阅了一些资料：
- [Improvements to XA Support in MySQL 5.7](https://mysqlserverteam.com/improvements-to-xa-support-in-mysql-5-7/)
- [X/Open XA](
https://en.wikipedia.org/wiki/X/Open_XA)
- [Re: What does XA stand for](http://www.orafaq.com/usenet/comp.databases.oracle.misc/2004/04/23/0521.htm)
- [Understanding XA transactions](https://docs.microsoft.com/en-us/sql/connect/jdbc/understanding-xa-transactions?view=sql-server-ver15)

eXtended Architecture的缩写，是一个分布式事务架构，已被 open group 标准化。

## 阅读资料
- [《从 Paxos 到 ZooKeeper》](https://book.douban.com/subject/26292004/)倪超 著
- [Paxos Made Simple](https://lamport.azurewebsites.net/pubs/paxos-simple.pdf)
- rocket mq 事务消息：[Transaction example](https://rocketmq.apache.org/docs/transaction-example/)
