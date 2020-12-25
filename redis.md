# Redis
### 持久化
* RDB (Redis DataBase)
> 通过时段性的保留快照数据，保证当数据丢失时，可以通过快照文件来恢复。

优点：数据恢复速度快

缺点：保存快照的时间间隔中容易丢失数据,备份时需要将数据写入临时文件，占用大量内存
* AOF (append only file)
> 记录每次数据的写操作，保存在aof文件中，重启redis服务之后重新执行以上操作

优点：数据一致性高

缺点：随着操作的不断增加，aof文件会变得越来越大，数据恢复的速度也会明显下降
(重写机制:当aof超过设置的大小时，会压缩文件内容。例如:set test 123和set test 1234可以合并为一条)

[参考资料](https://www.cnblogs.com/itdragon/p/7906481.html)

### 过期键的删除策略
* 定时删除
> 在创建键的时候生成一个timer定时器，当时限到达的时候定时器直接删除该键。

优点：减少内存占用

缺点：占用cpu性能
* 惰性删除
> 在键进行读操作的时候再进行判断是否过期，过期则删除。

优点：减少cpu负担

缺点：占用更多内存

* 定期删除
> 在某个时间段过后监测过期键并进行删除。

特点：属于以上两种策略的折中方案，根据对于时间段的设置和删除键时的持续时间，可以根据实际情况来倾向于保证cpu或内存中的某一方。

	redis使用的过期键值删除策略是：惰性删除加上定期删除，两者配合使用。

[参考资料1](https://blog.csdn.net/ThinkWon/article/details/101522970)

[参考资料2](https://www.cnblogs.com/lukexwang/p/4694094.html)

### 内存淘汰策略
> 当占用的内存超过指定的内存时，已经无法再进行占用内存的写操作（非删除类型的写操作），这时候需要对内存进行淘汰策略以保证接下来的操作。（可占用内存的大小可在redis.conf中的maxmemory进行设置 ）

* noeviction: 不删除策略, 达到最大内存限制时, 如果需要更多内存, 直接返回错误信息。（默认）
* volatile-lru：从已设置过期时间的数据集中挑选最近最少使用的数据淘汰。
* volatile-ttl：从已设置过期时间的数据集中挑选将要过期的数据淘汰。
* volatile-random：从已设置过期时间的数据集中任意选择数据淘汰。
* allkeys-lru：从数据集中挑选最近最少使用的数据淘汰
* allkeys-lfu：从数据集中挑选使用频率最低的数据淘汰。
* volatile-lfu：从已设置过期时间的数据集挑选使用频率最低的数据淘汰。（redis 5）
* allkeys-random：从数据集(server.db[i].dict)中任意选择数据淘汰。（redis 5）

>FIFO：First In First Out，先进先出。判断被存储的时间，离目前最远的数据优先被淘汰。
>LRU：Least Recently Used，最近最少使用。判断最近被使用的时间，目前最远的数据优先被淘汰。
>LFU：Least Frequently Used，最不经常使用。在一段时间内，数据被使用次数最少的，优先被淘汰。

[参考资料1](https://blog.csdn.net/ligupeng7929/article/details/79603060)

[参考资料2](https://blog.csdn.net/zhangchaoyang/article/details/109649331)
