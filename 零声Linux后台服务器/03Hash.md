# Hash

# 平衡二叉树

> 通过`比较`保证`有序`，`每次搜索`都能够`排除一半`，`时间复杂`度`O(log2为低n)`
>
> `100万`节点  --最`多`比较次数 ` 20次`
>
> `10亿`节点   -- 最`多`比较次数 ` 30次`

# 散列表

> 根据key计算key在表中的位置的数据结构，是key和其所在存储地址的映射关系

```cpp
struct node
{
    void *key;
    void *val;
    struct node *next;
};
```

![散列表](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtl7K6Fvc4NQBMOZBTMjagzmxMvWaTbHzRGlXz9MibcqY5BX1nbdA63lNlBzcOcqLU5vC4Y6oCyODA/640?wx_fmt=png "拉链法")

## 散列表组成

## hash函数

> 通过映射函数`Hash(key) = addr`;` hash函数`可能会把`两个或两个以上`的`不同key`映射到`同一地址`，这种情况称之为`冲突`(或者`Hash碰撞`)。

# 选择hash

* 计算`速度快`
* `强随机分布`(等概率，`均匀地`分布在整个地址空间)
* 常见hash算法: `murmurhash2` -使用最频繁的,`cityhash`强随机分布性,`siphash ` -redis的主要解决字符串接近的强随机分布性 [测试地址](https://github.com/aappleby/smhasher)

# hash 冲突

## 负载因子

> 数组存储的元素个数/数组长度：用来形容散列表的存储密度；负载因子越小，冲突概率越小，负载因子越大，冲突概率越大

## 解决冲突

### 链表法

> 将冲突元素用链表链接起来。(极端情况，冲突元素越多，冲突链表过长，可将此链表转换为红黑树，最小堆 --可以采用`超过256个`节点(经验值)将链表结构转换为`红黑树`或堆结构)
>
> redis，stl-unorder

![散列表](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtl7K6Fvc4NQBMOZBTMjagz2fIyVicPtMTl3Wm4mgNTRJaWbSxGCiaMNrwRbmQcblTXSZfcXc96ibaicQ/640?wx_fmt=png "拉链法")

### 开放寻址法

> 将所有的元素都存放在哈希表的数组中，不使用额外的数据结构
>
> * 当插入`新元素时，`使用哈希函数在哈希表中定位元素位置
> * 检查数组中该槽位索引是否存在元素，若槽位为`空`，则`插入`，否则3
> * 在2检测的槽位索引`加上一定步长`接着检查2
>
> 也可使用`双重hash`解决`hash聚集`现象
> 在 `. net HashTable` 类的 hash 函数 Hk 定义如下： 
> `Hk(key) = [GetHash(key) + k * (1 + (((GetHash(key) >> 5) + 1) % (hashsize – 1)))] % hashsize `
> 在此 `(1 + (((GetHash(key) >> 5) + 1) % (hashsize – 1))) `与 `hashsize `互为`素数`（两数互为素数表示两者没有`共同的质因 ⼦` ） ； 
> 执 ⾏ 了` hashsize 次探查后`， 哈希表中的`每⼀个位置`都有 且`只有⼀次`被`访问到`， 也就是说， 对于给定的` key` ，对哈希表中的同 ⼀ 位置不会同时使 ⽤Hi和Hj ；

### 负载因子不再合理范围内

> `used > size`  --`扩容`| `used < 0.1size` --`缩容`
>
> 扩容/缩容之后  -- `rehash`



# `STL`散列表实现

> `unordered * `
>
> 为了实现`迭代器`，将后面具体节点`串成一个单链表`,
>
> 当插入一个新的节点是，`hash`之后将该节点指向`上一层的最后一个节点`。以实现`迭代器`

![STL散列表实现](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtl7K6Fvc4NQBMOZBTMjagz7TowynNLNfvicg8ymicorwE9yT1YFDWVCDJYSYIqJ98apfOsODb85HPg/640?wx_fmt=png "STL散列表实现")

# 布隆过滤器

> 布隆过滤器是一种`概率性数据结构`，`高效`地`插入`和`查询`,不存储具体数据，占用空间小，查询结果存在误差`，可以确定一定不存在`，但不能确定一定存在，`不支持删除操作`

## 背景

> 内存`有限`，只想确定某个`key`存不存在，不想知道具体内容
>
> 当数据key,value存入某个文件时，将对应的`key`，`映射到文`件的`布隆过滤器`中，当查询时，不需要读取文件到内存，`只需`要`查询`布隆`过滤器(其放在内存当中)`，对应的key是否存在即可。  -- 数据库rocksdb
>
> 数据库`MySql` -- 查看`key`是否在`MySQL`当中，在服务器端`部署布隆过滤器`，查询时，`查布隆过滤器`



## 构成

> 使用位图`BIT数组` + `n个hash函数`
>

```cpp
vector<char> bitmap;
uint64_t bitmap ;  //数组
```

![image-20230705084805312](E:\学习笔记\C-notebook\零声Linux后台服务器\assets\image-20230705084805312.png)

## 原理

> * 当一个元素加入位图时，通过`k个hash函数`将这个元素`映射到`位图的`k个点`，并把他们`置为1`。
> * 当检索时，再通过`k个hash函数`运算检测`位图的k个点`是否`都是1,`如果有`不为1`的点，那么认为该`key不存在`，
> * 如果`全部为1`，则`可能`存在。
> * 不支持删除:位图中每个槽位`只有`两种状态`1或0`，`不确定`槽位`被设`置多少`次`，也不知道被`多少个key hash`映射而来以及是被具体`哪个hash函数`映射而来。

![布隆过滤器](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtl7K6Fvc4NQBMOZBTMjagzNXfXQmI8sxib5jbRdTBRzGO4QsQbDBVKUocQxzUjj9dqweCqP37spHA/640?wx_fmt=png "布隆过滤器")

## 应用分析

> * `n -`- 预期布隆过滤器中元素的`个数`
> * `p ` -- `假阳率` 在`0-1`之间
> * `m` -- `位图`所占空间
> * `k` --` hash`函数的`个数`
>
> [`n,p确认m,k`](https://hur.st/bloomfilter/)

```cpp
n = ceil(m / (-k / log(1 - exp(log(p) / k))))
p = pow(1 - exp(-k / (m / n)), k)
m = ceil((n * log(p)) / log(1 / pow(2, log(2)))); 
k = round((m / n) * log(2));
```



* 随着`n`越来越`多`，假阳率也越来`越高`

![pVSn](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtl7K6Fvc4NQBMOZBTMjagzrcwoA7V4Hf6umkNQyMeBDkehrNsLbAa4gDSYtwZUBpz2jiatS42yoEw/640?wx_fmt=png "PVSn")

* `位图`所占`空间`越来越大，`假阳率`也就越来越`低`

![pVSm](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtl7K6Fvc4NQBMOZBTMjagzVAQH8PReLUpsfzJicpmIVBXOO68f2ulicVeEM3bRpRrqjHyickKZxCtrQ/640?wx_fmt=png "pVSM")

* `hash函数`的个数`越多`，假阳率降低到一个水平，开始缓慢上升。 大约`31最低`

![pVSk](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtl7K6Fvc4NQBMOZBTMjagzMgAVgz46ZfNsGembvYH7QQrlLjT2Bdw1hXic3nNKOUTnmZMcFSl3FwQ/640?wx_fmt=png "pVSk")

## 应用场景

> 布隆过滤器通常用于判断某个`key`一定不存在的场景，同时允许判断存在时有误差的情况
>
> * `缓存穿透`解决
> * `热key限流`

![数据库redis](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtl7K6Fvc4NQBMOZBTMjagzL2mGexduqP3Ae7lH9PnSgG0RYtY6rYtBiaFBCZelLIeX0Randsib6Bhg/640?wx_fmt=png "数据库redis")

* 缓冲穿透

> `redis`，`MySQL`都没有数据，黑客可以利用此漏洞导致`MySQL压力过大`，如果以来真个系统将`陷入瘫痪`

* 读取步骤

> * 先访问`redis`，如存在，直接返回，如不存在走2
> * `访问MySQ`L，如果不存在，直接返回，如存在走3
> * 将`MySQL存在的key`写回`redis`

* 解决步骤

> * 在`redis`端设置`<key,null>` 键值对，以此避免访问MySQL；缺点是`<key,null>过多`的话，占用`过多内存`
> * 可以给key设置过期` expire key 600ms`，停止攻击后最后`由redis自动清除`这些无用的key
> * 在`server端`存储一个布隆过滤器，将MySQL包含的`key`放入布隆过滤器中，布隆过滤器一定不存在的数据

>* 为了减轻数据`MySQL`的访问压力，在s`erver端`与数据库MySQL之间加入缓存用来存储热点数据
>* 描述缓存穿透，server端请求数据时，缓存和数据库都不包含该数据，最终请求压力全部涌向数据库

# 只用2G内存在20亿个整数中找到出现次数最多的数

> `大文件` hash`拆成小文件`
>
> 单台机器  hash 分流到多台机器
>
> 主要解决 : `分布式缓存`扩容问题
>
> k 整数
>
> v 出现次数   -- -需要内存`uint32` ` 4个`字节  （`21.49亿`
>
> 一个key value对`8`个字节   `2亿`个 -- 需要`1.6G内存`
>
> 20亿  --- 需要`16G内存`
>
> 使用散列表
>
> * 拆分成若`干等份`(把10亿个整数的大文件拆分成多个文件中)
>* 目的: 把相同的整数放到同一个文件
> * 通过Hash的强随机性将相同整数放到统一文件中
> * 分别在每个文件中找出最大值。

# 分布式一致性hash

> 分布式一致性hash算法将哈希空间组织称一个虚拟的圆环，圆环的大小是`2^32`；
>
> 算法为：`hash(ip)%2^32` ,最终会得到一个`[0~2^32-1] `之间的无符号整型，这个整数代表服务器的`编号`；多个服务器都通过这种方式在hash换上映射一个点来标识该服务器的位置，当用户操作某个`key`，通过同样的算法生成一个值，沿环`顺时针定位`某个服务器，那么该key就在该服务器中

![分布式hash](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtl7K6Fvc4NQBMOZBTMjagzMzticgSQ9hI4CNJDv9xI2ogMS6jeMkhDIqPtv97ICZTKl7Ev9eN3eLg/640?wx_fmt=png "分布式一致性hash")

## 应用场景

> 将数据均衡地分散在不同的服务器当中，用来`分摊缓存服务器的压力`
>
> 解决缓存服务器数量变化尽量不影响缓存失效

## hash偏移

> 服务器承受的压力`不均匀`

![hash偏移](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtl7K6Fvc4NQBMOZBTMjagz7pCcq1HPttnae4icc1SWgETf2QbqLO7TZuPL4mLy2ib0F3ut0uPuicybw/640?wx_fmt=png "hash偏移")

## 虚拟节点

> 添加虚拟节点的概念；理论上哈希环上节点数越多，数据分布越均衡
>
> 为每个服务器节点计算多个哈希节点(虚拟节点); 通常做法是，`hash("IP:PORT:seqno")%2^32;`
>
> hash(key) % 分布式个数   确认`存储位置`
>
> * 当分布式个数增加一个之后，算法发生改变，原有映射
>
> 原有三个分布式节点
>
> `1,2,3,4     % 3 `      存储位置：` 1,2,0,1`
>
> 添加一个节点后:
>
> `1,2,3,4     % 4   `    存储位置：` 1,2,3,0`
>
> 算法发生改变，3,4，会出现`大面积缓存失效，`
>
> 解决方法:
>
> * 固定算法解决缓存失效
>
> `hash(key) % 2^32 = index`
>
> * 改变查找节点的映射关系，把具体的地址hash到`圆环(逻辑)上`，(顺时针查找)  -- `局部缓存失效`
>
> `hash(node-ip:port) % 2^32 = index`
>
> * `hash迁移`，  -- 解决局部缓存失效
>* `hash强随机性`，样本越大，

![虚拟节点](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbtl7K6Fvc4NQBMOZBTMjagzBRQVfeVzSaTHia5QhyVFIxF6v6SlCWC4zs57wj5KyejQEYu1Z1m2YicQ/640?wx_fmt=png "虚拟节点")
