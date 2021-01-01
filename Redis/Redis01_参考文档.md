# 参考资料
> [Redis简介](https://redis.io/topics/introduction)
> 
> [Redis下载](https://redis.io/download)
> 
> [Redis命令](https://redis.io/commands)
> 
> [Redis命令](http://www.redis.cn/commands.html)
> 
> [Redis数据类型](http://www.redis.cn/topics/data-types.html)

# Redis简介
Redis是开源的（BSD许可）内存数据结构存储，用作数据库，缓存和消息中间件。

它支持多种类型的数据结构，例如 字符串（strings），哈希（hashes），列表（lists），集合（sets），
带范围查询的排序集合（sorted sets with range queries），位图（bitmaps），超日志（hyperloglogs），
带有半径查询的流空间索引（geospatial indexes with radius queries）和流（streams）。

Redis具有内置的复制（replication），Lua脚本（Lua scripting），LRU逐出（LRU eviction），
事务（transactions）和不同级别的磁盘持久性（different levels of on-disk persistence），
通过Redis哨兵（Redis Sentinel）提供高可用性，通过Redis集群（Redis Cluster）提供自动分区。

Redis可以对这些数据类型运行原子操作，例如追加到字符串； 在哈希中增加值 ; 将元素推送到列表 ; 计算集的交集， 并集和差 ; 或获得排序集中排名最高的成员。

为了获得出色的性能，Redis使用 内存中的数据集。根据您的用例，您可以通过将数据集 偶尔转储到磁盘上，或者通过将每个命令附加到log来持久化它。
如果只需要功能丰富的网络内存缓存，则可以选择禁用持久性。

Redis还支持琐碎的设置主从异步复制，具有非常快速的非阻塞式第一次同步，自动重新连接以及网络拆分中的部分重新同步。

其他功能包括：
+ 交易次数
+ 发布/订阅
+ Lua脚本
+ 生存时间有限的键
+ LRU收回钥匙
+ 自动故障转移

Redis是用ANSI C编写的，并且可以在大多数POSIX系统中使用，例如Linux，* BSD，OS X，而无需外部依赖。
Linux和OS X是Redis开发和测试最多的两个操作系统，我们建议使用Linux进行部署。
Redis可以在基于Solaris的系统中使用，例如SmartOS，但是尽力提供了支持。Windows版本没有官方支持。

# Redis下载
Redis 使用标准版本标记进行版本控制：major.minor.patchlevel。
偶数的版本号表示稳定的版本， 例如 1.2，2.0，2.2，2.4，2.6，2.8。
奇数的版本号用来表示非标准版本,例如2.9.x是非稳定版本，它的稳定版本是3.0。

[下载页面](https://redis.io/download)

# Redis安装
下载、解压、编译Redis：
```
$ wget http://download.redis.io/releases/redis-5.0.5.tar.gz
$ tar xzf redis-5.0.5.tar.gz
$ cd redis-5.0.5
$ make
```

进入到解压后的 src 目录，通过如下命令启动Redis：
```
$ src/redis-server
```

您可以使用内置的客户端与Redis进行交互：
```
$ src/redis-cli
redis> set foo bar
OK
redis> get foo
"bar"
```

# Redis 数据类型
## 字符串（Strings）
字符串是一种最基本的Redis值类型。一个字符串类型的值最多能存储512M字节的内容。

Redis字符串是二进制安全的，这意味着一个Redis字符串能包含任意类型的数据，例如： 一张JPEG格式的图片或者一个序列化的Ruby对象。

使用场景：
+ 利用INCR命令簇（INCR, DECR, INCRBY）来把字符串当作原子计数器使用。
+ 使用APPEND命令在字符串后添加内容。
+ 将字符串作为GETRANGE 和 SETRANGE的随机访问向量。
+ 在小空间里编码大量数据，或者使用 GETBIT 和 SETBIT创建一个Redis支持的Bloom过滤器。

## 列表（Lists）
Redis列表是简单的字符串列表，按照插入顺序排序。一个列表最多可以包含 (2^32)-1 个元素（4294967295，每个表超过40亿个元素）。

可以添加一个元素到列表的头部（左边）或者尾部（右边）。LPUSH 命令插入一个新元素到列表头部，而RPUSH命令 插入一个新元素到列表的尾部。

当 对一个空key执行其中某个命令时，将会创建一个新表。 类似的，如果一个操作要清空列表，那么key会从对应的key空间删除。
这是个非常便利的语义， 因为如果使用一个不存在的key作为参数，所有的列表命令都会像在对一个空表操作一样。

一些列表操作及其结果：
```
LPUSH mylist a   # now the list is "a"
LPUSH mylist b   # now the list is "b","a"
RPUSH mylist c   # now the list is "b","a","c" (RPUSH was used this time)
```

从时间复杂度的角度来看，Redis列表主要的特性就是访问列表两端的元素是非常快的，支持时间常数的插入和靠近头尾部元素的删除，即使是需要插入上百万的条目。
但如果你试着访问一个非常大 的列表的中间元素仍然是十分慢的，因为那是一个时间复杂度为 O(N) 的操作。

使用场景：
+ 在社交网络中建立一个时间线模型，使用LPUSH去添加新的元素到用户时间线中，使用LRANGE去检索一些最近插入的条目。
+ 你可以同时使用LPUSH和LTRIM去创建一个永远不会超过指定元素数目的列表并同时记住最后的N个元素。
+ 列表可以用来当作消息传递的基元（primitive），例如，众所周知的用来创建后台任务的Resque Ruby库。
+ 你可以使用列表做更多事，这个数据类型支持许多命令，包括像BLPOP这样的阻塞命令。请查看所有可用的列表操作命令获取更多的信息。

## 集合（Sets）
Redis集合是一个无序的字符串合集。一个集合最多可以包含 (2^32)-1 个元素（4294967295，每个集合超过40亿个元素）。

可以以 O(1) 的时间复杂度（无论集合中有多少元素时间复杂度都为常量）完成添加，删除以及测试元素是否存在的操作。

Redis集合有着不允许相同成员存在的优秀特性。向集合中多次添加同一元素，在集合中最终只会存在一个此元素。
这就意味着，在添加元素前，并不需要事先进行检验此元素是否已经存在的操作。

Redis支持一些服务端的命令从现有的集合出发去进行集合运算，可以在很短的时间内完成合并（union），求交(intersection)，找出不同元素的操作。

使用场景：
+ 用集合跟踪一个独特的事。想要知道所有访问某个博客文章的独立IP？只要每次都用SADD来处理一个页面访问。那么你可以肯定重复的IP是不会插入的。
+ Redis集合能很好的表示关系。你可以创建一个tagging系统，然后用集合来代表单个tag。接下来你可以用SADD命令把所有拥有tag的对象的所有ID添加进集合，这样来表示这个特定的tag。如果你想要同时有3个不同tag的所有对象的所有ID，那么你需要使用SINTER.
+ 使用SPOP或者SRANDMEMBER命令随机地获取元素。

## 哈希（Hashes）
Redis Hashes是字符串字段和字符串值之间的映射，所以它们是完美的表示对象（eg:一个有名，姓，年龄等属性的用户）的数据类型。
```
@cli
HMSET user:1000 username antirez password P1pp0 age 34
HGETALL user:1000
HSET user:1000 password 12345
HGETALL user:1000
```

一个hash最多可以包含232-1 个key-value键值对（超过40亿）。

一个拥有少量（100个左右）字段的hash需要 很少的空间来存储，所有你可以在一个小型的 Redis实例中存储上百万的对象。

尽管Hashes主要用来表示对象，但它们也能够存储许多元素，所以你也可以用Hashes来完成许多其他的任务。

## 有序集合（Sorted sets）
Redis有序集合和Redis集合类似，是不包含相同字符串的合集。
它们的差别是，每个有序集合的成员都关联着一个评分，这个评分用于把有序集 合中的成员按最低分到最高分排列。

使用有序集合，你可以非常快地（O(log(N))）完成添加，删除和更新元素的操作。
因为元素是在插入时就排好序的，所以很快地通过评分(score)或者 位次(position)获得一个范围的元素。

访问有序集合的中间元素同样也是非常快的，因此你可以使用有序集合作为一个没用重复成员的智能列表。
在这个列表中， 你可以轻易地访问任何你需要的东西: 有序的元素，快速的存在性测试，快速访问集合中间元素！

简而言之，使用有序集合你可以很好地完成 很多在其他数据库中难以实现的任务。

使用有序集合你可以：

在一个巨型在线游戏中建立一个排行榜，每当有新的记录产生时，使用ZADD 来更新它。你可以用ZRANGE轻松地获取排名靠前的用户， 你也可以提供一个用户名，然后用ZRANK获取他在排行榜中的名次。 同时使用ZRANK和ZRANGE你可以获得与指定用户有相同分数的用户名单。 所有这些操作都非常迅速。
有序集合通常用来索引存储在Redis中的数据。 例如：如果你有很多的hash来表示用户，那么你可以使用一个有序集合，这个集合的年龄字段用来当作评分，用户ID当作值。用ZRANGEBYSCORE可以简单快速地检索到给定年龄段的所有用户。
有序集合或许是最高级的Redis数据类型，所以花些时间查看完整的有序集合（Sorted sets）命令列表去探索你能用Redis干些什么吧！

# Redis 命令
## Redis 命令总览
 Redis命令十分丰富， 一共14个redis命令组两百多个redis命令，包括的命令组有：
 + Cluster
 + Connection
 + Geo
 + Hashes
 + HyperLogLog
 + Keys
 + Lists
 + Pub/Sub
 + Scripting
 + Server
 + Sets
 + Sorted Sets
 + Strings 字符串
 + Transactions
 
## Cluster
## Connection
## Geo
## Hashes
## HyperLogLog
## Keys

## Lists 列表（链表）
### BLPOP
**删除，并获得该列表中的第一元素，或阻塞，直到有一个可用。**

BLPOP 是**阻塞式**列表的弹出原语。 
它是命令 LPOP 的阻塞版本，这是因为当给定列表内没有任何元素可供弹出的时候， 连接将被 BLPOP 命令阻塞。 
当给定多个 key 参数时，按参数 key 的先后顺序依次检查各个列表，弹出第一个非空列表的头元素。

#### 语法
```
BLPOP key [key ...] timeout
```

#### 返回值
多批量回复(multi-bulk-reply): 具体来说:
+ 当没有元素的时候会弹出一个 nil 的多批量值，并且 timeout 过期。
+ 当有元素弹出时会返回一个双元素的多批量值，其中第一个元素是弹出元素的 key，第二个元素是 value。

#### 例子
```
redis> DEL list1 list2
(integer) 0
redis> RPUSH list1 a b c
(integer) 3
redis> BLPOP list1 list2 0
1) "list1"
2) "a"
```

#### 设计场景：事件提醒
用来阻塞 list 的操作有可能是不同的阻塞原语。 
比如在某些应用里，你也许会为了等待新元素进入 Redis Set 而阻塞队列，直到有个新元素加入到 Set 中，这样就可以在不轮询的情况下获得元素。
这就要求要有一个 SPOP 的阻塞版本，而这事实上并不可用。
但是我们可以通过阻塞 list 操作轻易完成这个任务。

消费者会做的：
```
LOOP forever
    WHILE SPOP(key) returns elements
        ... process elements ...
    END
    BRPOP helper_key
END
```

而在生产者这角度我们可以这样简单地使用：
```
MULTI
SADD key element
LPUSH helper_key x
EXEC
```

#### 可靠的队列
当 BLPOP 返回一个元素给客户端的时候，它也从 list 中把该元素移除。
这意味着该元素就只存在于客户端的上下文中：如果客户端在处理这个返回元素的过程崩溃了，那么这个元素就永远丢失了。

在一些我们希望是更可靠的消息传递系统中的应用上，这可能会导致一些问题。
在这种时候，请查看 BRPOPLPUSH 命令，这是 BLPOP 的一个变形，它会在把返回元素传给客户端之前先把该元素加入到一个目标 list 中。

#### 非阻塞行为
当 BLPOP 被调用时，如果给定 key 内至少有一个非空列表，那么弹出遇到的第一个非空列表的头元素，并和被弹出元素所属的列表的名字 key 一起，组成结果返回给调用者。

当存在多个给定 key 时， BLPOP 按给定 key 参数排列的先后顺序，依次检查各个列表。
我们假设 key list1 不存在，而 list2 和 list3 都是非空列表。考虑以下的命令：
```
BLPOP list1 list2 list3 0
```
BLPOP 保证返回一个存在于 list2 里的元素（因为它是从 list1 –> list2 –> list3 这个顺序查起的第一个非空列表）。

#### 阻塞行为
如果所有给定 key 都不存在或包含空列表，那么 BLPOP 命令将阻塞连接， 直到有另一个客户端对给定的这些 key 的任意一个执行 LPUSH 或 RPUSH 命令为止。

一旦有新的数据出现在其中一个列表里，那么这个命令会解除阻塞状态，并且返回 key 和弹出的元素值。

当 BLPOP 命令引起客户端阻塞并且设置了一个非零的超时参数 timeout 的时候， 若经过了指定的 timeout 仍没有出现一个针对某一特定 key 的 push 操作，则客户端会解除阻塞状态并且返回一个 nil 的多组合值(multi-bulk value)。

timeout 参数表示的是一个指定阻塞的最大秒数的整型值。当 timeout 为 0 是表示阻塞时间无限制。

### BRPOP
**删除，并获得该列表中的最后一个元素；或阻塞，直到有一个可用。**

BRPOP 是一个阻塞的列表弹出原语。
它是 RPOP 的阻塞版本，因为这个命令会在给定list无法弹出任何元素的时候阻塞连接。 
该命令会按照给出的 key 顺序查看 list，并在找到的第一个非空 list 的尾部弹出一个元素。

起始版本：2.0.0

时间复杂度：O(1)

#### 语法
```
BRPOP key [key ...] timeout
```

#### 返回值
多批量回复(multi-bulk-reply): 具体来说:
+ 当没有元素可以被弹出时返回一个 nil 的多批量值，并且 timeout 过期。
+ 当有元素弹出时会返回一个双元素的多批量值，其中第一个元素是弹出元素的 key，第二个元素是 value。

#### 例子：
```
redis> DEL list1 list2
(integer) 0
redis> RPUSH list1 a b c
(integer) 3
redis> BRPOP list1 list2 0
1) "list1"
2) "c"
```

### BRPOPLPUSH
**弹出一个列表的值，将它推到另一个列表，并返回它，或阻塞，直到有一个可用。**

BRPOPLPUSH 是 RPOPLPUSH 的阻塞版本。 
当 source 包含元素的时候，这个命令表现得跟 RPOPLPUSH 一模一样。 
当 source 是空的时候，Redis将会阻塞这个连接，直到另一个客户端 push 元素进入或者达到 timeout 时限。 
timeout 为 0 能用于无限期阻塞客户端。

起始版本：2.2.0

时间复杂度：O(1)

#### 语法
```
BRPOPLPUSH source destination timeout
```

#### 返回值
批量回复(bulk-reply): 元素从 source 中弹出来，并压入 destination 中。 如果达到 timeout 时限，会返回一个空的多批量回复(nil-reply)。

### LINDEX
**通过列表索引，获取一个元素。**

返回列表里的元素的索引 index 存储在 key 里面。
下标是从0开始索引的，所以 0 是表示第一个元素， 1 表示第二个元素，并以此类推。
负数索引用于指定从列表尾部开始索引的元素。在这种方法下，-1 表示最后一个元素，-2 表示倒数第二个元素，并以此往前推。
当 key 位置的值不是一个列表的时候，会返回一个error。

起始版本：1.0.0

时间复杂度：O(N)

#### 语法
```
LINDEX key index
```

#### 返回值
```
bulk-reply：请求的对应元素，或者当 index 超过范围的时候返回 nil。
```

#### 例子
```
redis> LPUSH mylist "World"
(integer) 1
redis> LPUSH mylist "Hello"
(integer) 2
redis> LINDEX mylist 0
"Hello"
redis> LINDEX mylist -1
"World"
redis> LINDEX mylist 3
(nil)
redis> 
```


### LINSERT
**在列表中的一个元素之前或之后插入另一个元素。**

把 value 插入存于 key 的列表中在基准值 pivot 的前面或后面。
当 key 不存在时，这个list会被看作是空list，任何操作都不会发生。当 key 存在，但保存的不是一个list的时候，会返回error。

起始版本：2.2.0

时间复杂度：O(N)

#### 语法
```
LINSERT key BEFORE|AFTER pivot value
```

#### 返回值
integer-reply: 经过插入操作后的list长度，或者当 pivot 值找不到的时候返回 -1。

#### 例子
```
redis> RPUSH mylist "Hello"
(integer) 1
redis> RPUSH mylist "World"
(integer) 2
redis> LINSERT mylist BEFORE "World" "There"
(integer) 3
redis> LRANGE mylist 0 -1
1) "Hello"
2) "There"
3) "World"
redis> 
```


### LLEN
**获得队列(List)的长度。**

返回存储在 key 里的list的长度。 如果 key 不存在，那么就被看作是空list，并且返回长度为 0。 
当存储在 key 里的值不是一个list的话，会返回error。

起始版本：1.0.0

时间复杂度：O(1)

#### 语法
```
LLEN key
```

#### 返回值
integer-reply: key对应的list的长度。

#### 例子
```
redis> LPUSH mylist "World"
(integer) 1
redis> LPUSH mylist "Hello"
(integer) 2
redis> LLEN mylist
(integer) 2
redis> 
```

### LPOP
**从队列的左边出队一个元素。**

移除并且返回 key 对应的 list 的第一个元素。

起始版本：1.0.0

时间复杂度：O(1)

#### 语法
```
LPOP key
```

#### 返回值
bulk-string-reply: 返回第一个元素的值，或者当 key 不存在时返回 nil。

#### 例子
```
redis> RPUSH mylist "one"
(integer) 1
redis> RPUSH mylist "two"
(integer) 2
redis> RPUSH mylist "three"
(integer) 3
redis> LPOP mylist
"one"
redis> LRANGE mylist 0 -1
1) "two"
2) "three"
redis> 
```

### LPUSH
**从队列的左边入队一个或多个元素。**

将所有指定的值插入到存于 key 的列表的头部。
**如果 key 不存在，那么在进行 push 操作前会创建一个空列表。** 如果 key 对应的值不是一个 list 的话，那么会返回一个错误。

可以使用一个命令把多个元素 push 进入列表，只需在命令末尾加上多个指定的参数。
元素是从最左端的到最右端的、一个接一个被插入到 list 的头部。 所以对于这个命令例子 LPUSH mylist a b c，返回的列表是 c 为第一个元素， b 为第二个元素， a 为第三个元素。

起始版本：1.0.0

时间复杂度：O(1)

#### 语法
```
LPUSH key value [value ...]
```

#### 返回值
integer-reply: 在 push 操作后的 list 长度。

#### 例子
```
redis> LPUSH mylist "world"
(integer) 1
redis> LPUSH mylist "hello"
(integer) 2
redis> LRANGE mylist 0 -1
1) "hello"
2) "world"
redis> 
```


### LPUSHX
**当队列存在时，从队到左边入队一个元素。**

只有当 key 已经存在并且存着一个 list 的时候，在这个 key 下面的 list 的头部插入 value。 
与 LPUSH 相反，当 key 不存在的时候不会进行任何操作。

起始版本：2.2.0

时间复杂度：O(1)

#### 语法
```
LPUSHX key value
```

#### 返回值
integer-reply: 在 push 操作后的 list 长度。

#### 例子
```
redis> LPUSH mylist "World"
(integer) 1
redis> LPUSHX mylist "Hello"
(integer) 2
redis> LPUSHX myotherlist "Hello"
(integer) 0
redis> LRANGE mylist 0 -1
1) "Hello"
2) "World"
redis> LRANGE myotherlist 0 -1
(empty list or set)
redis> 
```

### LRANGE
**从列表中获取指定范围的元素。**

返回存储在 key 的列表里指定范围内的元素。 start 和 end 偏移量都是基于0的下标，即list的第一个元素下标是0（list的表头），第二个元素下标是1，以此类推。

偏移量也可以是负数，表示偏移量是从list尾部开始计数。 例如， -1 表示列表的最后一个元素，-2 是倒数第二个，以此类推。

#### 语法
```
LRANGE key start stop
```

#### 返回值
array-reply: 指定范围里的列表元素。

#### 例子
```
redis> RPUSH mylist "one"
(integer) 1
redis> RPUSH mylist "two"
(integer) 2
redis> RPUSH mylist "three"
(integer) 3
redis> LRANGE mylist 0 0
1) "one"
redis> LRANGE mylist -3 2
1) "one"
2) "two"
3) "three"
redis> LRANGE mylist -100 100
1) "one"
2) "two"
3) "three"
redis> LRANGE mylist 5 10
(empty list or set)
redis>
```

### LREM

### LSET
设置 index 位置的list元素的值为 value。当index超出范围时会返回一个error。

起始版本：1.0.0

时间复杂度：O(N)

#### 语法
```
LSET key index value
```

#### 返回值
simple-string-reply

#### 例子
```
redis> RPUSH mylist "one"
(integer) 1
redis> RPUSH mylist "two"
(integer) 2
redis> RPUSH mylist "three"
(integer) 3
redis> LSET mylist 0 "four"
OK
redis> LSET mylist -2 "five"
OK
redis> LRANGE mylist 0 -1
1) "four"
2) "five"
3) "three"
redis> 
```

### LTRIM
修剪(trim)一个已存在的 list，这样 list 就会只包含指定范围的指定元素。
start 和 stop 都是由0开始计数的， 这里的 0 是列表里的第一个元素（表头），1 是第二个元素，以此类推。

start 和 end 也可以用负数来表示与表尾的偏移量，比如 -1 表示列表里的最后一个元素， -2 表示倒数第二个，等等。
超过范围的下标并不会产生错误：如果 start 超过列表尾部，或者 start > end，结果会是列表变成空表（即该 key 会被移除）。
如果 end 超过列表尾部，Redis 会将其当作列表的最后一个元素。

起始版本：1.0.0

时间复杂度：O(N)

#### 语法
```
LTRIM key start stop
```

#### 返回值
simple-string-reply

#### 例子
```
redis> RPUSH mylist "one"
(integer) 1
redis> RPUSH mylist "two"
(integer) 2
redis> RPUSH mylist "three"
(integer) 3
redis> LTRIM mylist 1 -1
OK
redis> LRANGE mylist 0 -1
1) "two"
2) "three"
redis> 
```


### RPOP
### RPOPLPUSH
### RPUSH
### RPUSHX

## Pub/Sub
## Scripting
## Server
## Sets
## Sorted Sets

## Strings 字符串
### APPEND
如果 key 已经存在，并且值为字符串，那么这个命令会把 value 追加到原来值（value）的结尾。 
如果 key 不存在，那么它将首先创建一个空字符串的key，再执行追加操作，这种情况 APPEND 将类似于 SET 操作。

起始版本：2.0.0

时间复杂度：O(1)。均摊时间复杂度是O(1)， 因为redis用的动态字符串的库在每次分配空间的时候会增加一倍的可用空闲空间，所以在添加的value较小而且已经存在的 value是任意大小的情况下，均摊时间复杂度是O(1) 。

#### 语法
```
APPEND key value 
```

#### 返回值
Integer reply：返回append后字符串值（value）的长度。

#### 例子
```
redis> EXISTS mykey
(integer) 0
redis> APPEND mykey "Hello"
(integer) 5
redis> APPEND mykey " World"
(integer) 11
redis> GET mykey
"Hello World"
redis>
```

### BITCOUNT
统计字符串被设置为1的bit数。
一般情况下，给定的整个字符串都会被进行计数，通过指定额外的 start 或 end 参数，可以让计数只在特定的位上进行。
start 和 end 参数的设置和 GETRANGE 命令类似，都可以使用负数值：比如 -1 表示最后一个位，而 -2 表示倒数第二个位，以此类推。
不存在的 key 被当成是空字符串来处理，因此对一个不存在的 key 进行 BITCOUNT 操作，结果为 0 。

起始版本：2.6.0

时间复杂度：O(N)

#### 语法
```
BITCOUNT key [start end]
```

#### 返回值
Integer reply：被设置为 1 的位的数量。

#### 例子
```
redis> SET mykey "foobar"
OK
redis> BITCOUNT mykey
(integer) 26
redis> BITCOUNT mykey 0 0
(integer) 4
redis> BITCOUNT mykey 1 1
(integer) 6
redis>
```

### BITFIELD
本命令会把Redis字符串当作位数组，并能对变长位宽和任意未字节对齐的指定整型位域进行寻址。在实践中，可以使用该命令对一个有符号的5位整型数的1234位设置指定值，也可以对一个31位无符号整型数的4567位进行取值。类似地，在对指定的整数进行自增和自减操作，本命令可以提供有保证的、可配置的上溢和下溢处理操作。

BITFIELD命令能操作多字节位域，它会执行一系列操作，并返回一个响应数组，在参数列表中每个响应数组匹配相应的操作。

起始版本：3.2.0

时间复杂度：O(1)

#### 语法
```
BITFIELD key [GET type offset] [SET type offset value] [INCRBY type offset increment] [OVERFLOW WRAP|SAT|FAIL]
```

#### 例子
下面的命令是对一个8位有符号整数偏移100位自增1，并获取4位无符号整数的值：
```
> BITFIELD mykey INCRBY i5 100 1 GET u4 0
1) (integer) 1
2) (integer) 0
```

### BITOP

对一个或多个保存二进制位的字符串 key 进行位元操作，并将结果保存到 destkey 上。

BITOP 命令支持 AND 、 OR 、 NOT 、 XOR 这四种操作中的任意一种参数：
+ BITOP AND destkey srckey1 srckey2 srckey3 ... srckeyN ，对一个或多个 key 求逻辑并，并将结果保存到 destkey 。
+ BITOP OR destkey srckey1 srckey2 srckey3 ... srckeyN，对一个或多个 key 求逻辑或，并将结果保存到 destkey 。
+ BITOP XOR destkey srckey1 srckey2 srckey3 ... srckeyN，对一个或多个 key 求逻辑异或，并将结果保存到 destkey 。
+ BITOP NOT destkey srckey，对给定 key 求逻辑非，并将结果保存到 destkey 。

除了 NOT 操作之外，其他操作都可以接受一个或多个 key 作为输入。

执行结果将始终保持到destkey里面。

起始版本：2.6.0

时间复杂度：O(N)

#### 语法
```
BITOP operation destkey key [key ...]
```

#### 返回值
Integer reply：保存到 destkey 的字符串的长度，和输入 key 中最长的字符串长度相等。

#### 例子
```
redis> SET key1 "foobar"
OK
redis> SET key2 "abcdef"
OK
redis> BITOP AND dest key1 key2
(integer) 6
redis> GET dest
"`bc`ab"
redis>
```

### BITPOS
返回字符串里面第一个被设置为1或者0的bit位。
返回一个位置，把字符串当做一个从左到右的字节数组，第一个符合条件的在位置0，其次在位置8，等等。

默认情况下整个字符串都会被检索一次，只有在指定start和end参数(指定start和end位是可行的)，
该范围被解释为一个字节的范围，而不是一系列的位。所以start=0 并且 end=2是指前三个字节范围内查找。
注意，返回的位的位置始终是从0开始的，即使使用了start来指定了一个开始字节也是这样。

和GETRANGE命令一样，start和end也可以包含负值，负值将从字符串的末尾开始计算，-1是字符串的最后一个字节，-2是倒数第二个，等等。

不存在的key将会被当做空字符串来处理。

起始版本：2.8.7

时间复杂度：O(N)

#### 语法
```
BITPOS key bit [start] [end]
```

#### 返回值
Integer reply：
命令返回字符串里面第一个被设置为1或者0的bit位。

如果我们在空字符串或者0字节的字符串里面查找bit为1的内容，那么结果将返回-1。

如果我们在字符串里面查找bit为0而且字符串只包含1的值时，将返回字符串最右边的第一个空位。如果有一个字符串是三个字节的值为0xff的字符串，那么命令BITPOS key 0将会返回24，因为0-23位都是1。
基本上，我们可以把字符串看成右边有无数个0。然而，如果你用指定start和end范围进行查找指定值时，如果该范围内没有对应值，结果将返回-1。

#### 例子
```
redis> SET mykey "\xff\xf0\x00"
OK
redis> BITPOS mykey 0 ## 查找字符串里面bit值为0的位置
(integer) 12
redis> SET mykey "\x00\xff\xf0"
OK
redis> BITPOS mykey 1 0 ## 查找字符串里面bit值为1从第0个字节开始的位置
(integer) 8
redis> BITPOS mykey 1 2 ## 查找字符串里面bit值为1从第2个字节(12)开始的位置
(integer) 16
redis> set mykey "\x00\x00\x00"
OK
redis> BITPOS mykey 1 ## 查找字符串里面bit值为1的位置
(integer) -1
redis>
```

### DECR
对key对应的数字做减1操作。如果key不存在，那么在操作之前，这个key对应的值会被置为0。
如果key有一个错误类型的value或者是一个不能表示成数字的字符串，就返回错误。
这个操作最大支持在64位有符号的整型数字。

起始版本：1.0.0

时间复杂度：O(1)

#### 语法
```
DECR key
```

#### 返回值
数字：减小之后的value

#### 例子
```
redis> SET mykey "10"
OK
redis> DECR mykey
(integer) 9
redis> SET mykey "234293482390480948029348230948"
OK
redis> DECR mykey
ERR value is not an integer or out of range
redis> 
```

### DECRBY
将key对应的数字减decrement。如果key不存在，操作之前，key就会被置为0。
如果key的value类型错误或者是个不能表示成数字的字符串，就返回错误。
这个操作最多支持64位有符号的正型数字。

起始版本：1.0.0

时间复杂度：O(1)

#### 语法
```
DECRBY key decrement
```

#### 返回值
返回一个数字：减少之后的value值。

#### 例子
```
redis> SET mykey "10"
OK
redis> DECRBY mykey 5
(integer) 5
redis> 
```

### GET
返回key的value。如果key不存在，返回特殊值nil。如果key的value不是string，就返回错误，因为GET只处理string类型的values。

起始版本：1.0.0

时间复杂度：O(1)

#### 语法
```
GET key
```

#### 返回
```
simple-string-reply:key对应的value，或者nil（key不存在时）
```

#### 例子
```
redis> GET nonexisting
(nil)
redis> SET mykey "Hello"
OK
redis> GET mykey
"Hello"
redis> 
```

### GETBIT
返回key对应的string在offset处的bit值 当offset超出了字符串长度的时候，这个字符串就被假定为由0比特填充的连续空间。
当key不存在的时候，它就认为是一个空字符串，所以offset总是超出范围，然后value也被认为是由0比特填充的连续空间。到内存分配。

起始版本：2.2.0

时间复杂度：O(1)

#### 语法
```
GETBIT key offset
```

#### 返回值
integer-reply：在offset处的bit值

#### 例子
```
redis> SETBIT mykey 7 1
(integer) 0
redis> GETBIT mykey 0
(integer) 0
redis> GETBIT mykey 7
(integer) 1
redis> GETBIT mykey 100
(integer) 0
redis> 
```

### GETRANGE
返回key对应的字符串value的子串，这个子串是由start和end位移决定的（两者都在string内）。
可以用负的位移来表示从string尾部开始数的下标。所以-1就是最后一个字符，-2就是倒数第二个，以此类推。
这个函数处理超出范围的请求时，都把结果限制在string内。

起始版本：2.4.0

时间复杂度：O(N) N是字符串长度，复杂度由最终返回长度决定，但由于通过一个字符串创建子字符串是很容易的，它可以被认为是O(1)。

#### 语法
```
GETRANGE key start end
```

#### 返回值
bulk-reply

#### 例子
```
redis> SET mykey "This is a string"
OK
redis> GETRANGE mykey 0 3
"This"
redis> GETRANGE mykey -3 -1
"ing"
redis> GETRANGE mykey 0 -1
"This is a string"
redis> GETRANGE mykey 10 100
"string"
redis> 
```

### GETSET
以原子方式设置key为value并返回存储在中的旧值key。如果key存在但不包含字符串值，则返回错误。

自1.0.0起可用。

时间复杂度： O（1）


#### 语法
```
GETSET key value
```

#### 返回值
bulk-string-reply: 返回之前的旧值，如果之前Key不存在将返回nil。

#### 例子
```
redis> INCR mycounter
(integer) 1
redis> GETSET mycounter "0"
"1"
redis> GET mycounter
"0"
redis> 
```

### INCR
对存储在指定key的数值执行原子的加1操作。
**如果指定的key不存在，那么在执行incr操作之前，会先将它的值设定为0。**
如果指定的key中存储的值不是字符串类型（fix：）或者存储的字符串类型不能表示为一个整数，那么执行这个命令时服务器会返回一个错误(eq:(error) ERR value is not an integer or out of range)。
这个操作仅限于64位的有符号整型数据。

起始版本：1.0.0

时间复杂度：O(1)

#### 语法
```
INCR key
```

#### 返回值
integer-reply:执行递增操作后key对应的值。

#### 例子
```
redis> SET mykey "10"
OK
redis> INCR mykey
(integer) 11
redis> GET mykey
"11"
redis> 
```

#### 设计场景
##### 计数器
Redis的原子递增操作最常用的使用场景是计数器。
使用思路是：每次有相关操作的时候，就向Redis服务器发送一个incr命令。
例如这样一个场景：我们有一个web应用，我们想记录每个用户每天访问这个网站的次数。
web应用只需要通过拼接用户id和代表当前时间的字符串作为key，每次用户访问这个页面的时候对这个key执行一下incr命令。

这个场景可以有很多种扩展方法:
+ 通过结合使用INCR和EXPIRE命令，可以实现一个只记录用户在指定间隔时间内的访问次数的计数器
+ 客户端可以通过GETSET命令获取当前计数器的值并且重置为0
+ 通过类似于DECR或者INCRBY等原子递增/递减的命令，可以根据用户的操作来增加或者减少某些值 比如在线游戏，需要对用户的游戏分数进行实时控制，分数可能增加也可能减少。

##### 限速器
限速器是一种可以限制某些操作执行速率的特殊场景。
传统的例子就是限制某个公共api的请求数目。
假设我们要解决如下问题：限制某个api每秒每个ip的请求次数不超过10次。

我们可以通过incr命令来实现**两种方法**解决这个问题。

限速器1，更加简单和直接的实现如下：
```
FUNCTION LIMIT_API_CALL(ip)
ts = CURRENT_UNIX_TIME()
keyname = ip+":"+ts
current = GET(keyname)
IF current != NULL AND current > 10 THEN
    ERROR "too many requests per second"
ELSE
    MULTI
        INCR(keyname,1)
        EXPIRE(keyname,10)
    EXEC
    PERFORM_API_CALL()
END
```
这种方法的基本点是每个ip每秒生成一个可以记录请求数的计数器。
但是这些计数器每次递增的时候都设置了10秒的过期时间，这样在进入下一秒之后，redis会自动删除前一秒的计数器。
注意上面伪代码中我们用到了MULTI和EXEC命令，将递增操作和设置过期时间的操作放在了一个事务中， 从而保证了两个操作的原子性。

限速器2，另外一个实现是对每个ip只用一个单独的计数器（不是每秒生成一个），但是需要注意避免竟态条件。 我们会对多种不同的变量进行测试。
```
FUNCTION LIMIT_API_CALL(ip):
current = GET(ip)
IF current != NULL AND current > 10 THEN
    ERROR "too many requests per second"
ELSE
    value = INCR(ip)
    IF value == 1 THEN
        EXPIRE(value,1)
    END
    PERFORM_API_CALL()
END
```
上述方法的思路是，从第一个请求开始设置过期时间为1秒。如果1秒内请求数超过了10个，那么会抛异常。
否则，计数器会清零。

### INCRBY
将key对应的数字加increment。如果key不存在，操作之前，key就会被置为0。
如果key的value类型错误或者是个不能表示成数字的字符串，就返回错误。这个操作最多支持64位有符号的正型数字。

起始版本：1.0.0

时间复杂度：O(1)

#### 语法
```
INCRBY key increment
```

#### 返回值
integer-reply： 增加之后的value值。

#### 例子
```
redis> SET mykey "10"
OK
redis> INCRBY mykey 5
(integer) 15
redis> 
```

### INCRBYFLOAT
通过指定浮点数key来增长浮点数(存放于string中)的值. 当键不存在时,先将其值设为0再操作.下面任一情况都会返回错误:
+ key 包含非法值(不是一个string)。
+ 当前的key或者相加后的值不能解析为一个双精度的浮点值.(超出精度范围了)。
如果操作命令成功, 相加后的值将替换原值存储在对应的键值上, 并以string的类型返回。
string中已存的值或者相加参数可以任意选用指数符号，但相加计算的结果会以科学计数法的格式存储，无论各计算的内部精度如何，输出精度都固定为小数点后17位。

起始版本：2.6.0

时间复杂度：O(1)

#### 语法
```
INCRBYFLOAT key increment
```

#### 返回值
Bulk-string-reply: 当前key增加increment后的值。

#### 例子
```
redis> SET mykey 10.50
OK
redis> INCRBYFLOAT mykey 0.1
"10.6"
redis> SET mykey 5.0e3
OK
redis> INCRBYFLOAT mykey 2.0e2
"5200"
redis> 
```

### MGET
返回所有指定的key的value。对于每个不对应string或者不存在的key，都返回特殊值nil。正因为此，这个操作从来不会失败。

起始版本：1.0.0

时间复杂度：O(N) where N is the number of keys to retrieve.

#### 语法
```
MGET key [key ...]
```

#### 返回值
array-reply: 指定的key对应的values的list

#### 例子
```
redis> SET key1 "Hello"
OK
redis> SET key2 "World"
OK
redis> MGET key1 key2 nonexisting
1) "Hello"
2) "World"
3) (nil)
redis>
```

### MSET
对应给定的keys到他们相应的values上。
MSET会用新的value替换已经存在的value，就像普通的SET命令一样。
MSET是原子的，所以所有给定的keys是一次性set的。客户端不可能看到这种一部分keys被更新而另外的没有改变的情况。
如果你不想覆盖已经存在的values，请参看命令MSETNX。

起始版本：1.0.1

时间复杂度：O(N) 

#### 语法
```
MSET key value [key value ...]
```

#### 返回值
simple-string-reply：总是OK，因为MSET不会失败。

#### 例子
```
redis> MSET key1 "Hello" key2 "World"
OK
redis> GET key1
"Hello"
redis> GET key2
"World"
redis> 
```

### MSETNX
对应给定的keys到他们相应的values上。
**只要有一个key已经存在，MSETNX一个操作都不会执行。**
由于这种特性，MSETNX可以实现要么所有的操作都成功，要么一个都不执行，这样可以用来设置不同的key，来表示一个唯一的对象的不同字段。
MSETNX是原子的，所以所有给定的keys是一次性set的。客户端不可能看到这种一部分keys被更新而另外的没有改变的情况。

起始版本：1.0.1

时间复杂度：O(N) where N is the number of keys to set.

#### 语法
```
MSETNX key value [key value ...]
```

#### 返回值
nteger-reply，只有以下两种值：
+ 1 如果所有的key被set
+ 0 如果没有key被set(至少其中有一个key是存在的)

#### 例子
```
redis> MSETNX key1 "Hello" key2 "there"
(integer) 1
redis> MSETNX key2 "there" key3 "world"
(integer) 0
redis> MGET key1 key2 key3
1) "Hello"
2) "there"
3) (nil)
redis> 
```

### PSETEX
PSETEX和SETEX一样，唯一的区别是到期时间以毫秒为单位，而不是秒。

起始版本：2.6.0

时间复杂度：O(1)

#### 语法
```
PSETEX key milliseconds value
```

#### 例子
```
redis> PSETEX mykey 1000 "Hello"
OK
redis> PTTL mykey
(integer) 999
redis> GET mykey
"Hello"
redis>
```

### SET
将键key设定为指定的“字符串”值。
如果 key 已经保存了一个值，那么这个操作会直接覆盖原来的值，并且忽略原始类型。
当 set 命令执行成功之后，之前设置的过期时间都将失效。

起始版本：1.0.0

时间复杂度：O(1)

#### 语法
```
SET key value [EX seconds] [PX milliseconds] [NX|XX]
```

#### 可选项
+ EX seconds - 设置键key的过期时间，单位时秒。
+ PX milliseconds - 设置键key的过期时间，单位时毫秒。
+ NX - 只有键key不存在的时候才会设置key的值。
+ XX - 只有键key存在的时候才会设置key的值。
+ KEEPTTL - 保留与key关联的生存时间。

#### 返回值
Simple string reply: 如果SET命令正常执行那么回返回 OK。
Null reply: 如果由于用户指定了NX或XX选项但未满足条件而未执行SET操作，则返回 (nil)。

#### 历史
```
>= 2.6.12：增加了EX，PX，NX和XX选项。
>= 6.0：添加了该KEEPTTL选项。
```

#### 例子
```
redis> SET mykey "Hello"
"OK"
redis> GET mykey
"Hello"
redis> SET anotherkey "will expire in a minute" EX 60
"OK"
redis> 
```

#### 模式
命令 SET resource-name anystring NX EX max-lock-time 是一种用 Redis 来实现锁机制的简单方法。
如果上述命令返回OK，那么客户端就可以获得锁（如果上述命令返回Nil，那么客户端可以在一段时间之后重新尝试），并且可以通过DEL命令来释放锁。
客户端加锁之后，如果没有主动释放，会在过期时间之后自动释放。

### SETBIT
那个位置的bit要么被设置，要么被清空，这个由value（只能是0或者1）来决定。
当key不存在的时候，就创建一个新的字符串value。要确保这个字符串大到在offset处有bit值。
参数offset需要大于等于0，并且小于232(限制bitmap大小为512)。当key对应的字符串增大的时候，新增的部分bit值都是设置为0。

起始版本：2.2.0

时间复杂度：O(1)

#### 语法
```
SETBIT key offset value
```

#### 返回值
integer-reply：在offset处原来的bit值

#### 例子
```
redis> SETBIT mykey 7 1
(integer) 0
redis> SETBIT mykey 7 0
(integer) 1
redis> GET mykey
"\x00"
redis> 
```

### SETEX
设置key对应字符串value，并且设置key在给定的seconds时间之后超时过期。

这个命令等效于执行下面的命令：
```
SET mykey value
EXPIRE mykey seconds
```

SETEX是原子的，也可以通过把上面两个命令放到MULTI/EXEC块中执行的方式重现。相比连续执行上面两个命令，它更快，因为当Redis当做缓存使用时，这个操作更加常用。

起始版本：2.0.0

时间复杂度：O(1)

#### 语法
```
SETEX key seconds value
```

#### 返回值
simple-string-reply

#### 例子
```
redis> SETEX mykey 10 "Hello"
OK
redis> TTL mykey
(integer) 10
redis> GET mykey
"Hello"
redis> 
```

### SETNX
将key设置值为value，如果key不存在，这种情况下等同SET命令。 当key存在时，什么也不做。SETNX是”SET if Not eXists”的简写。

起始版本：1.0.0

时间复杂度：O(1)

#### 语法
```
SETNX key value
```

#### 返回值
Integer reply：特定值: 1 如果key被设置了，0 如果key没有被设置。

#### 例子
```
redis> SETNX mykey "Hello"
(integer) 1
redis> SETNX mykey "World"
(integer) 0
redis> GET mykey
"Hello"
redis> 
```

### SETRANGE
这个命令的作用是覆盖key对应的string的一部分，从指定的offset处开始，覆盖value的长度。
如果offset比当前key对应string还要长，那这个string后面就补0以达到offset。
不存在的keys被认为是空字符串，所以这个命令可以确保key有一个足够大的字符串，能在offset处设置value。

起始版本：2.2.0

时间复杂度：O(1)

#### 语法
```
SETRANGE key offset value
```

#### 返回值
integer-reply：该命令修改后的字符串长度

#### 例子
基本使用方法:
```
redis> SET key1 "Hello World"
OK
redis> SETRANGE key1 6 "Redis"
(integer) 11
redis> GET key1
"Hello Redis"
redis> 
```

补0的例子:
```
redis> SETRANGE key2 6 "Redis"
(integer) 11
redis> GET key2
"\x00\x00\x00\x00\x00\x00Redis"
redis>
```
 
### STRLEN
返回key的string类型value的长度。如果key对应的非string类型，就返回错误。

起始版本：2.2.0

时间复杂度：O(1)

#### 语法
```
SETRANGE key offset value
```

#### 返回值
integer-reply：key对应的字符串value的长度，或者0（key不存在）

#### 例子
```
redis> SET mykey "Hello world"
OK
redis> STRLEN mykey
(integer) 11
redis> STRLEN nonexisting
(integer) 0
redis> 
```

## Transactions