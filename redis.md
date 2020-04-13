# redis学习笔记

aliyun改造redis，使用了多线程模型
> 与传统的单线程Redis服务不同，云数据库Redis版的增强性能实例采用多线程模型，将单个节点的性能提高到原本的三倍左右。
来源：https://help.aliyun.com/document_detail/126397.html?spm=a2c4g.11186623.2.13.47703b77ukQ0Y9#concept-1305431

redis.io关于数据类型的描述： 
> Automatic creation and removal of keys
> So far in our examples we never had to create empty lists before pushing elements, or removing empty lists when they no longer have elements inside. It is Redis' responsibility to delete keys when lists are left empty, or to create an empty list if the key does not exist and we are trying to add elements to it, for example, with LPUSH.

> This is not specific to lists, it applies to all the Redis data types composed of multiple elements -- Streams, Sets, Sorted Sets and Hashes." 

开发者无须考虑删除空集合， redis 会自动删除空集合对应的key