# 1.1 Redis

Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。 它支持多种类型的数据结构，如 [字符串（strings）](http://www.redis.cn/topics/data-types-intro.html#strings)， [散列（hashes）](http://www.redis.cn/topics/data-types-intro.html#hashes)， [列表（lists）](http://www.redis.cn/topics/data-types-intro.html#lists)， [集合（sets）](http://www.redis.cn/topics/data-types-intro.html#sets)， [有序集合（sorted sets）](http://www.redis.cn/topics/data-types-intro.html#sorted-sets) 与范围查询， [bitmaps](http://www.redis.cn/topics/data-types-intro.html#bitmaps)， [hyperloglogs](http://www.redis.cn/topics/data-types-intro.html#hyperloglogs) 和 [地理空间（geospatial）](http://www.redis.cn/commands/geoadd.html) 索引半径查询。 Redis 内置了 [复制（replication）](http://www.redis.cn/topics/replication.html)，[LUA脚本（Lua scripting）](http://www.redis.cn/commands/eval.html)， [LRU驱动事件（LRU eviction）](http://www.redis.cn/topics/lru-cache.html)，[事务（transactions）](http://www.redis.cn/topics/transactions.html) 和不同级别的 [磁盘持久化（persistence）](http://www.redis.cn/topics/persistence.html)， 并通过 [Redis哨兵（Sentinel）](http://www.redis.cn/topics/sentinel.html)和自动 [分区（Cluster）](http://www.redis.cn/topics/cluster-tutorial.html)提供高可用性（high availability）。

# 2.关系型数据库与非关系型数据库

## 2.1.比较

| 内容       | 关系型数据库                   | 非关系型数据库                   |
| ---------- | ------------------------------ | -------------------------------- |
| 成本       | 有些需要收费                   | 基本都是开源                     |
| 查询数据   | 存储存于磁盘中，速度慢         | 数据存于缓存中，速度快           |
| 存储格式   | 只支持基础类型                 | K-V文档，图片等                  |
| 扩展性     | 有多表查询机制，扩展困难       | 数据之间没有耦合，容易扩展       |
| 持久性     | 适用持久存储、海量存储         | 不适用持久存储、海量存储         |
| 数据一致性 | 事务能力强，强调数据的强一致性 | 事务能力弱，强调数据的最终一致性 |

# 3.Rdeis-cli操作Rdeis

## 3.1 Redis-cli连接Redis

-h:用于指定ip

-p:用于指定端口

-a用于指定认证密码

![WeChatdcbe336fe431c84a0467b2f2e9a743f0](/Users/apple/Library/Containers/com.tencent.xinWeChat/Data/Library/Caches/com.tencent.xinWeChat/2.0b4.0.9/1f989e79eb4808b23080e898a2a556e4/dragImgTmp/WeChatdcbe336fe431c84a0467b2f2e9a743f0.png)

