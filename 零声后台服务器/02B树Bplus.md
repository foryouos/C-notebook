# B树
> `多路平衡`搜索树
> 索引在`内存`，`数据`映射`磁盘(`磁盘页`4K`的整数倍)，
> 多路，降低红黑树和二叉树的`层`高，`降低IO`访问次数

## B树和B+树

> * `B树`节点中即`存储数据`信息，也会`存储索引`信息
> * `B+树`节点中即存储数据信息，也会存储索引信息，`非叶子`节点`只有索引`信息
> * `B+ 树`期待更`少`的磁盘`IO`  - 将索引信息和数据信息进行`分层`管理
> * `B+树`加载到内存的`无效`数据`更少`

###  `etcd`

> `强`一致性、`高`可用性的数据访问服务，用来存储`少量重要`的数据。刷盘的时候更快，B树和B+树都是`映射着磁盘页`
>
> * 内存中使用B树
> * 磁盘中使用B+树

### `MySQL`

> 索引 : 索引对应一个B+树，支持的行锁，MySQL拥有缓存
>
> * 聚集索引B+树 ： 主键索引
> * 辅助索引:  先通过辅助索引`获取聚集索引`，然后根据聚集索引获得叶子结点记录的信息
>
> 索引覆盖: select查询具体数据时，这个数据恰好在辅助索引时，就可以获得所有数据，不用再到聚集索引
>
> 最左匹配规则: 组合索引时，根据从左到右顺序依次匹配。
>
> 事务：服务器和数据库可能需要`多条语句`实现某个逻辑，多个数据库请求。数据库提供一个`机制`，让多个语句一同执行，当查询的数据表`不存在`时，对数据表的查询和插入就`不再执行`

### 时间轮
> 处理`海量定时任务`，多线程下定时器设计，按照执行顺序进行组织0(1),`分层`的目的`减少内存开销`。只关注`最近60秒`要执行的任务
> * `Linux内核`
> * `kafka`
> * `skynet`
> * `netty`


### 跳表
> 多层级结构，多层级的`有序链表`，从上往下跳，随机性的数据结构` 0(log2n)` (以2为低)

* `redis`  :  kv   v-> zset 有序集合，score : 有序， member确保唯一
* `levelrocksdb`,`rocksdb`(存储引擎)


![理想跳表](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFrMbKfR4VGcUYWfjlpibDUn6pKOD8fxpoPbg1nibZjMp6OaZ3x2yfF9cw/640?wx_fmt=png "理想跳表")

![跳表插入](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFPEEL5WIVZVfVozDHcaSTXl1jXYgR7G7h6X75WE2ZrG4Cn7HWP4oTmw/640?wx_fmt=png "跳表插入")

![跳表索引失衡](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFq59zC6oeIZvbcI77tZichaRhRKibsgmnkEnDqusOzuRicAP3c6iawKicePg/640?wx_fmt=png "跳表索引失衡")

![B+树结构](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbsHH5T1E68OFdicfagNibRxrz1sIfBsnGxsrOVDeYb1mXDmrklPnVlYO33wtND8sSg2Q6OJlYpQ6WyQ/640?wx_fmt=jpeg)


## 引入

> 二叉树层高，对比次数多，找到下一个节点多，
> 数据存储到磁盘，其读取到内存，`读取`效率`下降`。若每个节点都存储在磁盘，每次对比后找下一个节点都是一次磁盘寻址。二叉树就极其耗时。
> 急需降低层高的数据存储，使用`多叉`
> B树，所有的`叶子节点在同一层`

![磁盘寻址](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFhhZfdeKNQzgIOAEPOInK36jXxqhwWbn3A5MvDq1lZibHp8qNuwmomSw/640?wx_fmt=png "磁盘寻址")

## 性质

* 每个节点最多拥有`M颗`子树
* `根`节点`至少`拥有`两`颗子树
* 除根节点以外，其余`每个分支`节点`至少`拥有`M/2颗`子树
* 所有的`叶节点`都在同`一层`上
* 有`K颗子`树的分支结点则存在`k-1`个关键字，关键字按照`递增`顺序进行排序
* 关键字数量满足`ceil(M/2)-1 <=n <=M-1`   `ceil向上取整`（删除时注意)

## B树B+树区别

* B树:`所有节点存储数据`(部分数据在内存，部分节点在磁盘)
* B+树: `叶子节点存储数据`，`内节点索引引用存储内存`(`B+树`索引在`内存数量` > `B树`)提高索引效率，常用与索引磁盘数据大量数据(`MySql `，`MongoDB`等)


## 添加
* 设置`B树`的M为`6`，最多为6个子树 
* 添加数据为`A-Z`依次加入
* 根节点分裂:当添加的节点F时，节点数`超过6`，将进行节点分裂，以原有节点`中心C`为中心节点分为左右结点进行分裂（`先分裂再添加`)

![节点添加](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFOPePW4z7aV7dxQQTaMWlJTJ6BSicL5eqaWibVMvnNvLHpLmWF9ecI6IQ/640?wx_fmt=pngg "节点添加")

* `叶节点分裂`:继续添加数据：

![节点添加](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFzwTm0vYsJyHEuiczkWfcspkzc6HcnTjr4PjNyK2FfZkTEL8EHPPQZQg/640?wx_fmt=png "节点添加")

* 当加到L时原有达到`分裂条件`，`先分裂`，让中心节点`I上移`

![根节点拆分](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFF0icrh4ibtU14zeCiaIbEVAmOae6o7jqAfRUpNk1Me7pl2gIAzcXaq48A/640?wx_fmt=png "根节点拆分")

* 在往后的`添加过程`中，若`节点也满`了，将如第一部分对`节点进行分裂` ,`先分裂后添加`

![image-20230629091213538](E:\学习笔记\C-notebook\零声Linux后台服务器\assets\image-20230629091213538.png)





## 删除

> 删除条件：
>
> * `idx`子树，`ceil(m/2) -1`
>   * `借位`
>     * 从`idx -1` 大于`ceil(m/2) -1`
>     * 从`idx +1`大于`ceil(m/2) -1`
>   * 合并
>     * `{childs[idx].keys} {keys[idx]} {childs[idx+1].keys}`



* `节点归并`
![节点归并](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFnhKl0GXYlUmkZqxAplRbSNGU1Wgepfbw8NISWwRcjC0sxibSED0Ze1g/640?wx_fmt=png "节点归并")
* 若关键字在`叶子节点`中，`直接删除`即可
![删除节点](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFiao2rHHZhNnZOVGrr63aAoHAV7Zdscib84JxOeAZ5ZoM1OoE3dKlic0nA/640?wx_fmt=png "删除节点")
* 若删除节点后不满足性质 `关键字`数量满足`ceil(M/2)-1 <=n <=M-1` `先合并或借位再删除`
	![不满足关键词](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFohvIAC35iaXNtLfWhpcDSQQmdhJblnlutLo4jje6E89c4VLFo2V08icw/640?wx_fmt=png "不满足关键词")
	*  要删除`节点A`，其`父节点 `，若其`树=M/2-1` 需要`借位`.避免`后面节点资源不足` (因为删除A之后需要合并，会导致资源不足)。将`I节点`借过去之后，其`I节点`的有节点`最小`将随着I节点的变化变为`左边的最大`。并将`右边的L`替代为`根节点`，（也是为了下面合并之后其父节点仅剩F自己的提前安排）如下:
	![借根节点I](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFRCzfOwITyxEtiaSbYkd4iaY5KLJZwGkcWDykysW8AWZ2njBLSfmdC8lQ/640?wx_fmt=png "借根节点I")
	* 删除A之后，不满足性质，则`进行合并`，将其`父与相邻相邻节点`进行合并。 `先合并再删除`
	![合并](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFWP3cJZhY9nOTg8mvmhNJnEP2LBs9kU0xZY4WjWkkYca6ZZNNo5PsKQ/640?wx_fmt=png "合并")
	* `删除节点A`
	![删除节点A](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFKsX5kHLY67Ma0N0PpHRGdv1ScaWicdE64wmIxjrw4vJjibQ1hIjVLuWA/640?wx_fmt=png "删除节点A")

* 删除节点B，
	* 其`父节点和删除的节点`都满足条件
	![删除节点B](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFKsX5kHLY67Ma0N0PpHRGdv1ScaWicdE64wmIxjrw4vJjibQ1hIjVLuWA/640?wx_fmt=png "删除节点B")
	* `直接删除`
	![B节点已删除](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFjvJUu2aKnPKicFZwBGDrJk39rOENp7OPtNyedAyUS3FC0oMLHrJErJQ/640?wx_fmt=png "B节点已删除")



* `删除节点D`
  
	* 删除D之后，其节点不满足最后一条性质`ceil(M/2)-1 <=n <=M-1`,其父节点符合条件，不用借位，`直接合并`
	![直接合并](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFg74uxm5L4yg4CV7uPWToR8KiahBjm0BP0bLYtCibb46lwtGeGrOdoEFg/640?wx_fmt=png "直接合并")
	* 删除D节点
	
  ![删除D节点](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFegWSOIwYkVa3yTibtYK6KEMicoXgeXyrvpkBicleRrianuCyCElIxjAR4A/640?wx_fmt=png "删除D节点")
  
* `删除E节点`，其`父节点`满足`树=M/2-1`，需要先进行`借位`，在进行`合并删除`，由于`右边`也等于`M/2-1`，`无法`进行`借位`，则进行对`父节点进行合并`
	![需要合并](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFjvJUu2aKnPKicFZwBGDrJk39rOENp7OPtNyedAyUS3FC0oMLHrJErJQ/640?wx_fmt=png "需要合并")
	
	* 合并
	
	![合并](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFXjLM8fEzrXxibeWkHU0donOxh0pnOxficvf8ExiafibcvgrGZkbIr0tTYA/640?wx_fmt=png "合并后")
	
	* 直接删除E节点
	
	![删除节点E](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFXjLM8fEzrXxibeWkHU0donOxh0pnOxficvf8ExiafibcvgrGZkbIr0tTYA/640?wx_fmt=png "删除节点E")
	
* 删除F节点，删除后满足所有添加，直接删除即可

​	![删除节点F](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFOPePW4z7aV7dxQQTaMWlJTJ6BSicL5eqaWibVMvnNvLHpLmWF9ecI6IQ/640?wx_fmt=png)

* 删除G之后，其所在节点不满足条件，则合并节点，在删除
	* 合并节点

	![删除节点G](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFticibmBlYSmugrh3khXBJ15P6B71iaC5TSCNTBYHIicxPLzMFsVNt1WTSQ/640?wx_fmt=png)
	
	* 删除
	
	![删除节点G之后](https://mmbiz.qpic.cn/mmbiz_png/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFamFSpMiarzud3p5iaaDRicpottcTFcWTuQM8Whpl7eoVWRqHIWwOnBy3g/640?wx_fmt=png "删除节点G")



## 插入

* 找到`对应的节点`
* 对节点的`key对比`，找到`合适的位置`
* 插入的数是`插在叶子节点上面`



## 代码实现

> // 根节点分裂
>
> //    1 - 3 特殊情况
>
> //    其它一分为2
>
> //    根节点不同
>
> // 分裂：添加
>
> // 合并与错位: 删除
>
> //  合并



### B树定义

```cpp
//1, 定义B/B-树
//#define SUM_M 3
typedef int KEY_VALUE;
//2, B树结构体
typedef struct _btree_node
{
    // int keys[2*SUM_M -1];   // 5
    // struct _btree_node *childrens[2*SUM_M]; //6
    KEY_VALUE *keys;   // 5
    struct _btree_node **childrens; //6
    int num;  //存储多少数据
    int leaf; //是否为叶子节点 0为内节点
}btree_node;

// 定义根节点
typedef struct _btree
{
    struct _btree_node *root;
    int t;  //定义支持节点个数
} btree;


```

### 创建节点

```cpp
// 创建节点,叶节点
btree_node *btree_create_node(int t,int leaf)
{
    btree_node* node = (btree_node*)calloc(1,sizeof(btree_node));  //自带初始化为0  calloc
    if(node == NULL)
    {
        assert(0);
        return NULL; //出错是内存不够用，
    }
    node->leaf = leaf;
    node->keys = (KEY_VALUE*)calloc(1,(2*t -1)*sizeof(KEY_VALUE));
    node->childrens = (btree_node**)calloc(1,(2*t)*sizeof(btree_node*));
    node->num = 0;
    return node;
}
void btree_create(btree *T, int t) {
	T->t = t;
	
	btree_node *x = btree_create_node(t,1);
	T->root = x;
}
```

### 销毁节点

```cpp

// 销毁节点
void btree_destory_node(btree_node* node)
{
    assert(node);
    free(node->childrens);
    free(node->keys);
    free(node);
}
```



### 节点分裂

![结点分裂](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFgWJoVMtkPA65L2wMGmogrHf9m1GYcxM5A8wA702CXTZQaujusibfKZw/640?wx_fmt=jpeg "节点分裂")

```cpp
// 分裂
// 参数 T代表这棵树 x为需要分裂的父节点 i为x的节点的第几个子树(从0开始)
void btree_split_child(btree *T,btree_node *x,int idx)
{
    int t = T->t;
    // 把需要斐裂的节点全部赋值给了Z
    btree_node *y = x->childrens[idx];  //根据条件找到需要分裂的节点
    btree_node *z = btree_create_node(t,y->leaf);
    // Z放前面
    z->num = t -1;  // 复制两个
    //1 把y的节点右边复制给z节点（把y和z copy到z）
    int j;
    for(j = 0;j< t-1;j++)
    {
        z->keys[j] = y->keys[t+j];
    }
    // 如果是内节点，把y的的y和z子节点叶复制过去
    if(y->leaf == 0)
    {
        for(j = 0;j<t;j++)
        {
            z->childrens[j] = y->childrens[t +j];
        }
    }
    y->num = t -1; // 修改为2

    //2，将X的节点向后移动，
    // 当x放上去,若x后面还有数据
    for(j = x->num;j >= idx+1;j--)
    {
        x->childrens[j+1] = x->childrens[j]; //向后移动

    }
    //3 x在定义的节点子孩子
    x->childrens[idx+1] = z;
    //4 将x节点里面的key值进行交换进行往后移
    for(j = x->num-1;j>=idx;j--)
    {
        x->keys[j+1] = x->keys[j];
    }

    // 5，x 中间节点向上
    x->keys[idx] = y->keys[t-1];
    // x的节点+1
    x->num += 1;
}
```

### 插入未满节点

```cpp
//插入进去一个未满的节点  x为要插入节点
void btree_insert_notfull(btree* T,btree_node *x,KEY_VALUE key)
{
    int i = x->num -1;
    //如果不是叶子节点就往下递归，如果是叶子节点，再插入
    if(x->leaf == 1)
    {
        // 插入
        // 当i大于等于0并且x对应的key值比k大时，向前移动i--
        // x的值后移，为存储key留下空间
        while (i>=0 && x->keys[i] > key)
        {
            /* code */
            x->keys[i+1] = x->keys[i];
            i--;
        }
        //找到插入的位置(while循环的再次i--，需要加1)
        x->keys[i+1] = key;
        x->num +=1;
    }
    else
    {
        // 继续递归
        // 找到对应key值应该再那个子树上面
        while(i>=0 && x->keys[i] > key)
        {
            i--;
        }
        // 如果子树是满的
        if(x->childrens[i+1]->num == 2*(T->t)-1) 
        {
            //进行分裂
            btree_split_child(T,x,i+1);
            //如果key大于keys后的节点，++
            if(key>x->keys[i+1])
            {
                i++;
            }
            
        }
        // 进行向下递归
        btree_insert_notfull(T,x->childrens[i+1],key);

    }

}
```

### 插入节点

```cpp
// 根节点如何分裂，一分为三

void btree_insert(btree* T,int key)
{
    btree_node *r = T->root;
    if(r->num == 2*SUM_M-1) // 
    {
        // 创建新节点
        btree_node* node = btree_create_node(0); //0为非叶子节点
        T->root = node;  // 将根节点指向此
        node->childrens[0] = r;  // 根节点的子节点为r
        btree_split_child(T,node,0); //分裂完成之后
        // 再进行插入
        
        int i = 0;
        // 判断node的值找到key对应的node子节点位置
		if (node->keys[0] < key) 
        {
            i++;
        }
        // 插入非满节点
		btree_insert_nonfull(T, node->childrens[i], key);

    }
    else
    {
        btree_insert_nonfull(T, r, key);
    }
}
```

### 节点合并

```cpp
// 合并
// x为当前的节点（即需要删除节点的父节点)，idx要删除的节点位置
void btree_merge(btree *T,btree_node *x,int idx)
{
    btree_node *left = x->childrens[idx]; // 左
    btree_node *right = x->childrens[idx+1]; // 右

    
    left->keys[T->t-1] = x->keys[idx]; //将x的C拷贝到子节点
     //开始循环
    int i = 0;
     // 将x的右边的值复制到左边
    for(i=0;i<T->t-1;i++)
    {
        left->keys[T->t+i] = right->keys[i];
    }
    // 判断是不是叶子节点
    // 将右边的子树也全部拷贝到左边
    if(!left->leaf)
    {
        for(i=0;i<T->t;i++)
        {
            left->childrens[T->t+i]= right->childrens[i];
        }
    }
    left->num +=T->t;
    btree_destory_node(right);// 右边节点已经全部赋值到左边，右边的清楚释放
    // 删除的父节点少了一个节点需要，后面的节点需要前移
    // 从1开始
    for(i=idx+1;i<x->num;i++)
    {
        x->keys[i-1]=x->keys[i];
        x->childrens[i] = x->childrens[i+1];
    }
    x->childrens[i+1] = NULL;
	x->num -= 1;
    // 若为根
	if (x->num == 0) {
		T->root = left;
		btree_destory_node(x);
	}
}
```

### 节点输出

```cpp
// B树输出
// T B树，node为输出的节点,根节点T.root, layer层
void btree_print(btree *T, btree_node *node, int layer)
{

	btree_node* p = node;
	int i;
    // 如果p不为空
	if(p)
    {
		printf("\nlayer = %d keynum = %d is_leaf = %d\n", layer, p->num, p->leaf);
		// 对node的节点全部输出
        for(i = 0; i < node->num; i++)
			printf("%c ", p->keys[i]);
		printf("\n");
#if 0
		printf("%p\n", p);
		for(i = 0; i <= 2 * T->t; i++)
			printf("%p ", p->childrens[i]);
		printf("\n");
#endif
        // 层数 ++
		layer++;
        // 子节点进行遍历递归循环
		for(i = 0; i <= p->num; i++)
			if(p->childrens[i])
				btree_print(T, p->childrens[i], layer);
	}
	else printf("the tree is empty\n");
}
```

### 节点删除

![结点删除](https://mmbiz.qpic.cn/mmbiz_jpg/ORog4TEnkbsZ0Q0UO5h5wcuDyvjQkaPFfoC9kiariaqRKssU4nFLrWVokMZEdicXSK21CMLtkicIUwBvqj0sRTZWRQ/640?wx_fmt=jpeg "节点删除")

```cpp
// B树删除

// 1，递归找到对应的子树
// 2,  * `idx`子树，`ceil(m/2) -1`
/*   * `借位`
        * 从`idx -1` 大于`ceil(m/2) -1`
        * 从`idx +1`大于`ceil(m/2) -1`
    * 合并
        * `{childs[idx].keys} {keys[idx]} {childs[idx+1].keys}`
*/
// node要删除的节点，key要删除的关键字
void btree_delete_key(btree* T,btree_node *node,KEY_VALUE key)
{
    // 如果输出的节点为空
    if(node == NULL)
    {
        return ;
    }
    // 找节点
    int idx = 0 ,i;
    while(idx < node->num && key > node->keys[idx])
    {
        idx ++;
    }
    // 找到节点
    if(idx <node->num && key == node->keys[idx])
    {
        //1， 如果节点是叶节点
        if(node->leaf)
        {
            //直接删除
            // 将删除之后的节点前移
            for (i = idx;i < node->num-1;i ++) {
				node->keys[i] = node->keys[i+1];
			}
            // 初始化最后一个删除节点的空间
            node->keys[node->num - 1] = 0;
			node->num--;
            // 如果节点为root释放空间
            if (node->num == 0) 
            { //root
				free(node);
				T->root = NULL;
			}
            return ;
        }
        else if(node->childrens[idx]->num >= T->t)
        {
            // 2(不完善图例)，如果不是叶节点 像前面idx借
            btree_node *left = node->childrens[idx];
			node->keys[idx] = left->keys[left->num - 1];

			btree_delete_key(T, left, left->keys[left->num - 1]);
        }
        else if(node->childrens[idx+1]->num >= T->t)
        {
            //3， 向后面借
            btree_node *right = node->childrens[idx+1];
			node->keys[idx] = right->keys[0];

			btree_delete_key(T, right, right->keys[0]);
        }
        else
        {
            //2,情况 数量小于3 SUM_M合并 再调用删除
            btree_merge(T, node, idx);
			btree_delete_key(T, node->childrens[idx], key);

        }
    }
    // 左右借孩子
    else //idx < node-> num 但是 key != node->keys[idx],需要在其孩子中寻找
    {
        // 4,锁定到孩子的节点
        btree_node *child = node->childrens[idx];
        // 如果孩子的结点为空
		if (child == NULL) 
        {
			printf("Cannot del key = %d\n", key);
			return ;
		}
        // 如果孩子的数量为2
        if (child->num == T->t - 1) {
            // 定义此节点的左右孩子
			btree_node *left = NULL;
			btree_node *right = NULL;
            // 左孩子
			if (idx - 1 >= 0)
				left = node->childrens[idx-1];
            // 右孩子
			if (idx + 1 <= node->num) 
				right = node->childrens[idx+1];
            // 如果左/ 右孩子节点数量 > 3
			if ((left && left->num >= T->t) ||
				(right && right->num >= T->t)) {
                // 记录左右结点的谁的数量大，
				int richR = 0;  
				if (right) richR = 1;
                // 1，是右边大，0是右边小
				if (left && right) richR = (right->num > left->num) ? 1 : 0;
                // 如果右节点的数量大于3并且右边数量大 ，从右边借
				if (right && right->num >= T->t && richR) 
                {    //borrow from next从右边借
                    //先将node的值复制到child的后面
					child->keys[child->num] = node->keys[idx];
                    // 孩子节点的孩子最后叶等于右边的第一个
					child->childrens[child->num+1] = right->childrens[0];
                    // 孩子的数量++
					child->num ++;
                    // node节点的最后一个为rigth节点的第一个
					node->keys[idx] = right->keys[0];
                    // 右孩子节点(由于第一个keys移动，全部向前移动)
					for (i = 0;i < right->num - 1;i ++) 
                    {
						right->keys[i] = right->keys[i+1];
						right->childrens[i] = right->childrens[i+1];
					}
                    // 全部前移之后，右节点的最后一个对其进行销毁
					right->keys[right->num-1] = 0;
					right->childrens[right->num-1] = right->childrens[right->num];
					right->childrens[right->num] = NULL;
                    // 右节点数量-1 
					right->num --;
					
				} 
                else  // 如果没有右节点或者数量不足3或者右边数量小，考虑左边，从左边借
                { //borrow from prev
                    // 孩子节点全部后移
					for (i = child->num;i > 0;i --) {
						child->keys[i] = child->keys[i-1];
						child->childrens[i+1] = child->childrens[i];
					}
                    //将孩子的第一个指向左边的最后一个
					child->childrens[1] = child->childrens[0];
					child->childrens[0] = left->childrens[left->num];
					child->keys[0] = node->keys[idx-1];
					// 孩子的数量++
					child->num ++;
                    // keys赋值
					node->keys[idx-1] = left->keys[left->num-1];
                    //最左节点最后一个进行初始化为NULL
					left->keys[left->num-1] = 0;
					left->childrens[left->num] = NULL;
					left->num --;
				}

			} 
            // 如果其左节点/右节点为2时 进行合并
            else if ((!left || (left->num == T->t - 1))
				&& (!right || (right->num == T->t - 1))) 
            {

				if (left && left->num == T->t - 1) 
                {
					btree_merge(T, node, idx-1);					
					child = left;
				} else if (right && right->num == T->t - 1) 
                {
					btree_merge(T, node, idx);
				}
			}
		}
        // 将其孩子节点调用删除函数
        btree_delete_key(T, child, key);
    }
   
}
int btree_delete(btree *T, KEY_VALUE key) {
	if (!T->root) return -1;
	btree_delete_key(T, T->root, key);
	return 0;
}
```

### 测试函数

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <assert.h>
int main() {
	btree T = {0};
    // 创建节点
	btree_create(&T, 3);
	srand(48); //随机数

	int i = 0;
	char key[26] = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
    // 将26英文字母插入
	for (i = 0;i < 26;i ++) {
		//key[i] = rand() % 1000;
		printf("%c ", key[i]);
		btree_insert(&T, key[i]);
	}
    //输出
	btree_print(&T, T.root, 0);

	for (i = 0;i < 26;i ++) {
		printf("\n---------------------------------\n");
		btree_delete(&T, key[25-i]);
		//btree_traverse(T.root);
		btree_print(&T, T.root, 0);
	}
	
}
```

