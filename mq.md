# 消息队列

## 资料

[设计(rocket mq git repo 中的文档) ](https://github.com/apache/rocketmq/blob/master/docs/cn/design.md)

## 顺序消息与阻塞
> 顺序消息的重试
> 对于顺序消息，当消费者消费消息失败后，消息队列 MQ 会自动不断地进行消息重试（每次间隔时间为 1 秒），这时，应用会出现消息消费被阻塞的情况。因此，建议您使用顺序消息时，务必保证应用能够及时监控并处理消费失败的情况，避免阻塞现象的发生。

上面这段话说明rocketmq 顺序消息 从语义上能保证消费的有序性，而不仅仅是消费者接收到消息的有序性。

## 分区顺序消息

> 阿里巴巴集团内部电商系统均使用分区顺序消息，既保证业务的顺序，同时又能保证业务的高性能。

https://help.aliyun.com/document_detail/49319.html?spm=5176.234368.1250548.2.187bdb25ajAhVl


## producer group
rocketmq source code git repo rocketmq/docs/en/Concept.md
> ## 9 Producer Group
> A collection of the same type of Producer, which sends the same type of messages with consistent logic. If a transaction message is sent and the original producer crashes after sending, the broker server will contact other producers in the same producer group to commit or rollback the transactional message.
