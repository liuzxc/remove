---
layout: post
title:  Discord 如何存储数十亿消息数据
excerpt: Cassandra 在 Discord 中的应用
comments: true
share: true
categories: articles
---

原文链接: [How Discord Stores Billions of Messages](https://blog.discordapp.com/how-discord-stores-billions-of-messages-7fa6ec7ee4c7#.p2vea9li9)


Discord 的发展速度已经超出了我们的预期，越来越多的用户产生大量的聊天信息。7月份，一天的消息量
已经达到了 4000 万，12 月份的时候就到了 1 亿，在写这篇博客的时候每天的消息量已经超过了 1.2
亿。我们很早就决定永久存储聊天记录，用户可以在任何时间，任何设备上查看他们的消息。有如此多的
数据，而且还在不断的增长，并且要保证可用性，我们要如何实现呢？我们选择了 Cassandra！

## 我们在做什么

Discord 的早期版本是在 2015 年初花了两个月左右的时间完成的。在这种情况下，满足快速迭代的
最佳数据库就是 MongoDB。 Discord 的所有数据都是存储在一个单独的 MongoDB 副本集合里，这是
我们有意为之的，但是我们也有计划迁移到新的数据库（我知道由于 MongoDB 分片使用起来很复杂并且
有未知的不稳定因素，我们将不再使用它）。实际上这也是我们公司文化的一部分：快速构建以证明产品
特性，不过通常会使用比较粗暴的解决方案。

消息被存储在一个以 channel_id 和 created_at 字段为组合索引的 MongoDB 集合中。在 2015 年的
11 月份左右，消息数达到了 1 亿的存储量，此时我们预计到的问题发生了：RAM 没有足够的空间存储
数据和索引，延迟也变得不可预期，所以是时候迁移到一个新的数据库了。


## 选择正确的数据库

在选择一个新的数据库之前，我们必须理解当前系统的读写模式以及当前的解决方案为什么会产生这样的
问题。

* 很显然的是，我们的读操作非常的随机并且读写比例是 50/50；

* 语音聊天服务器几乎不发送文本消息，每隔几天才会产生一条文本消息，就算是一年也不太可能会产生
1000 条文本消息。问题在于即便是如此少的文本消息数据也很难提供给用户，就算只返回 50 条信息也可
能导致许多磁盘随机搜索从而导致磁盘缓存驱逐；

* 私有的文本聊天服务器会发送很多消息，一年时间内很容易达到 10 - 100 万条记录，用户请求的
数据通常是最近的，问题在于这些服务器的用户通常在 100 人以下，这些数据被请求的频率很低，因此
不太可能在磁盘缓存里获取到。

* 庞大的公共聊天服务器会发送大量的消息，数以千计的用户每天都会发送很多消息，所以一年很容易
就会产生几百万条消息。这类数据会在最近的一个小时里被频繁访问，因此数据通常会在磁盘缓存里。

* 我们知道在未来的一年里，我们会提供更多的方式去帮助用户实现随机读：有能力浏览关于你过去 30
 天内的数据，然后跳转到历史记录的某个点上，以及全文搜索。所有的这些都预示着更多的随机读！！


接下来定义我们的需求：

* Linear scalability（线性扩展）—— 我们希望推迟该方案，也不想手动重新分配（re-shard）数据。

* Automatic failover（自动故障转移）—— 我们喜欢晚上睡觉，让 Discord 可以尽可能的自我治愈。

* Low maintenance （维护成本低）—— 搭建好后就可以工作，随着数据的增长，我们只需要添加节点。

* Proven to work （被证明可用）—— 我们喜欢尝试新技术，但不要太新！

* Predictable performance （可预期的性能）—— 当 95% 的接口响应时间超过 80 ms 时，我们
应该收到警报，我们也不想用 Redis 或者 Memcached 缓存消息。

* Not a blob store （不是二进制对象存储）—— 如果我们不得不经常反序列化和追加 blob，每秒
成千上万的写消息将会很糟糕。

* Open source （开源）—— 我们想自己控制命运而不想依赖于一个第三方公司。

Cassandra 是唯一完全符合我们需求的数据库。我们可以随时添加节点去扩展它，也能容忍丢失部分节点
而不影响整个应用。像 Netflix 和 Apple 这样的大公司都有上千个 Cassandra 节点，相关数据被顺序
的存储在磁盘上，方便在集群上分发和查找。Cassandra 虽然由 DataStax 支持，但是依然是开源和社区
驱动的。

做出选择之后，我们需要证明它是可以按照我们预期工作的。

## 数据建模

向新手介绍 Cassandra 的最好方式就是 KKV 存储，前两个 K 构成了主键。第一个 K 是 partition key，
用于确定数据所在的节点以及在磁盘上的位置。一个分区会包含多行数据，一个分区内的一行被第二个 K （clustering key）
所标识。clustering key 是分区中的主键，它也决定了如何对行进行排序。你可以把分区看做是一个有序
字典，这些属性组合允许非常强大的数据建模。

还记得在 MongoDB 中消息是通过 channel_id 和 created_at 来进行索引的么？由于所有的查询都会基于
channel_id，索引 channel_id 可以作为 partition key，但 created_at 却不是一个好的 clustering key，
因为两条消息可能会有相同的创建时间。幸运的是， Discord 中的每一个 ID 都是一个 [Snowflake](https://blog.twitter.com/engineering/en_us/a/2010/announcing-snowflake.html)（按时间顺序可分类），我们可以使用它们来代替，所以主键就变成了 (channel_id, message_id)，
这意味着在加载通道时，我们可以准确地告诉 Cassandra 对消息的扫描范围。

下面是我们 messages 表的 schema（忽略了大概10个字段）。

```sql
CREATE TABLE messages (
  channel_id bigint,
  message_id bigint,
  author_id bigint,
  content text,
  PRIMARY KEY (channel_id, message_id)
) WITH CLUSTERING ORDER BY (message_id DESC);
```

Cassandra 的 schema 并不像传统的关系型数据库，很容易被修改但并不会影响性能，我们得到了最好的blob存储和关系存储。

当把消息数据导入 Cassadra 的时候，我们立刻看到了警告日志，告诉我们分区大小超过了 100 MB， 为什么
会这样呢？Cassandra 支持 2GB 的分区啊！很显然是是，尽管可以这样做，但并不意味这应该这样做。如果分区过大，
Cassandra 在集群扩张和数据压缩等情况下会导致巨大的 GC 压力，大的分区也意味着数据不能在集群内
很好的被分发。很明显，我们必须以某种方式限制分区的大小，因为单个的 Discord channel 可以存在多年，并且不断地增大。

我们决定按照时间存储消息。我们查看了 Discord 中最大的 channel，确定如果在一个 bucket 内储存大约10天的信息，
我们可以轻松地把分区保持在100MB以下。bucket 必须从 message_id 或时间戳中派生出来。

```python
DISCORD_EPOCH = 1420070400000
BUCKET_SIZE = 1000 * 60 * 60 * 24 * 10


def make_bucket(snowflake_id):
   if snowflake_id is None:
       timestamp = int(time.time() * 1000) - DISCORD_EPOCH
   else:
       # When a Snowflake is created it contains the number of
       # seconds since the DISCORD_EPOCH.
       timestamp = snowflake_id >> 22
   return int(timestamp / BUCKET_SIZE)


def make_buckets(start_id, end_id=None):
   return range(make_bucket(start_id), make_bucket(end_id) + 1)
```

Cassandra 的 partition keys 是可以组合的，因此我们新的主键变为 `((channel_id, bucket), message_id)`。

```sql
CREATE TABLE messages (
   channel_id bigint,
   bucket int,
   message_id bigint,
   author_id bigint,
   content text,
   PRIMARY KEY ((channel_id, bucket), message_id)
) WITH CLUSTERING ORDER BY (message_id DESC);
```

为了查询 channel 中的最近消息，我们生成了从当前时间到 channel_id 的一个 bucket range。
然后我们顺序的查询分区直到收集到足够多的消息。这种方法的缺点是很少有活跃的 Discords 需要查询
多个 bucket 来收集足够多的消息。在实践中，这已经被证明是很好的，因为在第一个分区中通常会发现足够多的消息，
并且这种情况占大多数。

现在导入消息到 Cassandra 中一切顺利，我们准备在线上环境尝试。

## Dark Launch

引入一个新的系统到线上环境总是让人不安，所以好的办法是在不影响用户的前提下去测试它，所以我们从 MongoDB
 到 Cassandra 设置了双读/写。

部署之后，我们的缺陷跟踪系统立刻发现了错误，它告诉我们 author_id 为 null，怎么会为 null 呢？author_id
是一个必填字段啊！

## 最终一致性

Cassandra 是一个 AP 数据库，这意味着它的可用性很强，这是我们想要的。在 Cassandra 当中，read-before-write
是一种反模式，Cassandra 做的每件事本质上都是一个upsert，即使你只提供某些列。您也可以将其写入任何节点，
并且它将在每列基础上使用 “last write wins” 语义自动解决冲突。它是怎么让我们出错的呢?

![Example of edit/delete race condition](https://cdn-images-1.medium.com/max/2000/1*t7VkLRKZVeHb_c6Tl1heew.gif)

在一个用户编辑消息而另外一个用户同时删除消息的场景中，我们最后得到的是一个丢失了所有数据的行，
除了主键和文本，因为所有 Cassandra 的写都是 upserts。有两种可能的方案来解决这个问题：

1. 编辑消息时，将整个消息写回。这就有可能恢复被删除的消息，并为并发写入其他列添加更多冲突的机会。
2. 计算出消息是损坏的，并从数据库中删除它。

我们选择了第二种方案，当 author_id 为 null 的时候就删除它。

当解决这个问题之后，我们发现写操作的效率非常差。因为 Cassandra 是最终一致性，所以它并不会马上
删除数据。它必须复制删除操作到其他节点，即使其他节点暂时不可用也要这样做。Cassandra 把删除作
为一种称为 “墓碑（tombstone）” 的写形式。读操作的时候，Cassandra 会跳过墓碑。墓碑的生命周期
是根据系统配置的时间（默认10天），时间过期后，会在数据压缩过程中被永久删除。

删除一列和把 null 写进这一列是完全相同的操作，它们都会生成一个墓碑。因为 Cassandra 上所有的写
都是 upserts，这意味着即使第一次写入 null，也会生成一个墓碑。在实践中，我们完整的消息表包含
16个列，但是平均每条消息只有4个值被设置，因此在大多数情况下，我们都是无缘无故的把12个墓碑
写入 Cassandra 。解决方案很简单：只写非 null 的值到 cassandra。

## 性能

众所周知，Cassandra 的写速度比读速度快，而且我们也观察到了这一点。写入是亚毫秒，读取时间低于5毫秒。
我们观察到，不管访问的是什么数据，在测试的一周内性能保持一致。没什么奇怪的，我们得到了我们所期望的。

![Read/Write Latency via Datadog](https://cdn-images-1.medium.com/max/1600/1*MrdDaSA6ghOQQ7WyzqztcQ.png)

以快速、一致的阅读性能为例，这是一年前在一个有数百万条消息的 channel 中跳转到某一条消息的示例:

![Jumping back one full year of chat](https://cdn-images-1.medium.com/max/2000/1*TGzjmCHRqrWSgarkjUrAwQ.gif)

## 巨大惊喜

一切都进展顺利，所以我们把 Cassandra 作为主要数据库，并在一周内逐步淘汰 MongoDB。它继续完美的运作...
大约6个月后，直到有一天，Cassandra 变得反应迟钝。

我们发现 Cassandra 运行了长达 10s 的 “stop-the-world” GC，但我们并不知道为什么会这样。我们开始
深入调查并发现了一个加载消息需要花费 20s 的 Discord channel。`Puzzles & Dragons Subreddit` 公共
服务器就是罪魁祸首。因为是公共频道，所以我们加入进去看了一下。让我们惊讶的是，这个 channel
里面只有1条消息。可想就在那一刻，他们使用了 API 删除了数百万条消息，在 channel 中只留下一条消息。

你应该还记得 Cassandra 是通过墓碑来执行删除操作。当用户加载一个 channel 的时候，虽然只有1条消息，
Cassandra 仍然会遍历数百万条消息墓碑（生成垃圾的速率超过了 JVM 的收集速率）。

我们通过以下方式来解决这个问题：

* 把墓碑的生命周期从10天降低到2天，每晚在消息集群服务器上运行 [Cassandra repairs](https://docs.datastax.com/en/cassandra/2.1/cassandra/tools/toolsRepair.html#toolsRepair__description)。

* 我们改进了查询代码去跟踪空的 buckets，避免它们之后出现在 channel 中，这意味着如果一个用户
再次查询，最坏的情况也只是遍历最近的几个 buckets。

## 未来

我们目前运行的是12个节点集群，其复制因子为3，并将在需要时继续添加新的 Cassandra 节点。我们
相信这将会持续很长一段时间，但是随着 Discord 的持续增长，在未来的某个时间我们每天要存储
数十亿条信息。Netflix 和苹果运行了数百个节点，因此我们知道我们可以对此对进行一些思考，我们也喜欢在口袋里有一些对未来的想法。

### 近期

* 把我们的消息集群从 Cassandra 2 升级到 Cassandra 3。Cassandra 3 有一种[新的存储格式](https://www.datastax.com/2015/12/storage-engine-30)可以把存储容量降低超过 50%。

* 新版本的 Cassandra 可以更好的在单节点上处理更多的数据。我们目前在每个节点上储存了接近1TB的
压缩数据。我们相信通过把这个数据提高到2TB，可以安全地减少集群的节点数。

### 长期

* 尝试使用 [Scylla](https://www.scylladb.com/)，一个用 c++ 编写的 Cassandra 兼容数据库。
在正常的操作过程中，我们的 Cassandra 节点实际上并没有使用太多的 CPU，但是在我们运行修复
(一个反熵过程)的非高峰时间时，它们会变得相当的高，并且持续时间会随着上次修复后的数据量的增加而增加。
Scylla 的时间明显较低。

* 构建一个系统，将未使用的 channel 归档到谷歌云存储上，并按需加载它们。我们要避免做这件事，也不要认为我们必须这样做。

## 结论

我们做了这个转换已经一年多了，尽管有“大惊喜”，但它一直一帆风顺。我们每天从超过1亿条信息到超过1.2亿条信息，
性能和稳定性保持一致。

由于这个项目的成功，我们已经将我们剩余的生产数据转移到 Cassandra，这也是一个成功。

在这篇文章的后续文章中，我们将探讨如何让数十亿的信息被搜索。
