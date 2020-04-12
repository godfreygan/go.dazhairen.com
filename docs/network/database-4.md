# NoSQL数据库

NoSQL（Not Only SQL），指的是非关系型的数据库。
Web2.0开始，传统的关系型数据库在应付Web2.0网站，特别是超大规模和高并发的动态网站时显得力不从心，暴露了很多难以克服的问题。  
而非关系型的数据库则由于其本身的特点得到了非常迅速的发展。

目前流行的NoSQL主要有 Redis、MongoDB、Cassandra和Membase等。这些数据库都有高性能、高并发读写等特点。


## Redis

Redis 是一个键值对存储系统，和 Memcached 类似，它支持存储的value类型相对更多，包括字符串（string）、链表（list）、哈希（hash）、集合（set）和有序集合（zset）。


Go语言目前支持的Redis的驱动如下：

- https://github.com/garyburd/redisgo

- https://github.com/go-redis/redis

- https://github.com/hoisie/redis

- https://github.com/alphazero/Go-Redis

- https://github.com/simonz05/godis


## MongoDB

MongoDB 是一个高性能、开源、无模式的文档型数据库，是一个介于关系型数据库和非关系型数据库之间的产品，是非关系型数据库当中功能最丰富的。

它支持的数据结构非常松散，采用的是类似json的bjson格式来存储数据的，因此可以存储比较复杂的数据类型。

MongoDB 最大的特点是它支持的查询语句非常强大，其语法有点类似于面向对象的查询语言，几乎可以实现类似关系型数据库单表查询的绝大部分功能，而且还支持对数据建立索引。
