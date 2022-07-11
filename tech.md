# 零碎笔记整理
## Spark
### Spark Web Ui
Exchange节点 ： Shuffle Map阶段。相当于实现数据并行化的算子
HashAggregate预聚合：数据算子下推的一种，直接在读取数据本地进行预聚合，功能上相当于上次讲的MapReduce中的combine操作，提前在map端进行局部聚合来减少reduce端数据。这个相当于是spark sql的一个优化。但并不是所有的聚合函数都可以进行预聚合的，比如说avg()，求平均数，不能说提前先求个平均数，那就有问题了。这里的count是可以的。

spark.driver.memory=12G
spark.driver.cores=1
spark.executor.memory=8G
spark.executor.cores=4

### 数据倾斜
3.  数据倾斜处理：分布式任务的优化就是合理的分配数据到各个节点上进行处理。但实际业务中数据很难均匀分布，比如热点用户、视频。那么数据倾斜就会产生在shuffle中：Aggregation和Join。其中Aggregation有Partial机制，问题不是很突出。在Join中，某些key数据量远大于其他时，处理这些key的任务就会非常慢，甚至挂掉。业务层面解决思路：
    1.  过滤无关数据：实际业务中大量数据倾斜都是由业务无关的数据导致。比如脏数据、null数据。排查后过滤即可。
    2.  小表广播：使用map join 也就是Broadcast Join的方式避免shuffle。
    3.  倾斜数据分离：将有数据倾斜表T先分为t1和t2，t1没有倾斜数据，t2只包含倾斜数据。分别join，在union。
   ```
        select * from (
            select * from t1,myT where t1.key = myT.key
            union all
            select /*+BROADCAST(t2)*/ * from t2,myT where t2.key = myT.key
        )
  ```
    4. 数据打散：思路为分散倾斜数据。例如：将A、B两表按id 进行Join，可以先将大表A中id加后缀（id0，id1，id2），起打散作用。小表B中每条数据都复制多份。这样就可以起到数据均匀处理的效果。

```
表A：
select id,value,concat(id,(rand() * 1000)%3) as id_new from A
表B：
select id,value,concat(id,suuffix) as id_new 
from (
    select id,value,suffix from B
    Lateral view explode(array(0,1,2)) n as suffix
)
```


数据倾斜会在哪一个环节出现？为什么？
数据倾斜会在shuffle阶段出现，因为涉及到了数据打散，如果某个值的数据异常多，则会导致这个值全部分布在一个partition中，造成数据倾斜。在sql中join和group by会产生shuffle操作，group by一般会有预处理步骤，不用太关注；join操作是发生数据倾斜的最主要场景。

shuffle过程会消耗哪些资源？哪些会成为瓶颈？
内存、cpu、网络io、磁盘io。shuffle中涉及到了数据的shuffle write、shuffle read，数据的序列化，网络传输。在我们可控制的内存、cpu资源来看，最主要的瓶颈是shuffle read阶段会因为数据量的过大而导致内存溢出，因此需要合理的调整每个partition的数据量并均匀的分散数据。


### Spark Join

https://zhuanlan.zhihu.com/p/88623042

总体上来说，Join的基本实现流程如下图所示，Spark将参与Join的两张表抽象为流式表(StreamTable)和查找表(BuildTable)，通常系统会默认设置为StreamTable大表，BuildTable为小表。流式表的迭代器为streamItr，查找表迭代器为BuidIter。Join操作就是遍历streamIter中每条记录，然后再buildIter中查找相匹配的记录。
spark join 图片

1.1 BroadcastJoin机制：
对小表进行广播，避免shuffle的产生。web ui的sql图可以看到driver collect的时间，build建表压缩时间，broadcast广播时间。所以频繁有广播时，会对driver端产生较大压力。需要注意的是：在Outer类型的Join中，基表不能被广播，比如A left outer join B时，只能广播右表B。
触发场景：
- 被广播表小于参数 spark.sql.autoBroadcastJoinThreshold=20971520，默认10MB。公司改默认20MB
- 在SQL中显示添加Hint（MAPJOIN、BROADCASTJOIN或BROADCAST）
[图片]

1.2 ShuffledHashJoin机制：
步骤：
1. 对两张表分别进行shuffle重分区，将相同key的记录分到对应分区中，这一步对应Exchange节点
2. 将查找表分区构造一个HashMap，然后在流式表中一行行对应查找。

要将来自BuildTable每个分区的记录放到hash表中，那么BuildTable就不能太大，否则就存不下，默认情况下hash join的实现是关闭状态，如果要使用hash join，原生spark必须满足以下四个条件：
1. 查找表总体估计大小超过spark.sql.autoBroadcastJoinThreshold设定的值，即不满足BroadcastJoin 条件
2. 关闭优先使用SortMergeJoin开关，spark.sql.join.preferSortMergeJoin=false
3. 每个分区的平均大小不超过spark.sql.autoBroadcastJoinThreshold设定的值，查找表数据量 < 广播数据阈值 * shuffle的partition数。
4. streamIter的大小是buildIter三倍以上

ShuffledHashJoin触发条件非常苛刻，我司有相关优化策略，下面会提到。

[图片]

1.3 SortMergeJoin机制：
此为spark sql的主要join方式，hash join是将一侧的数据完全加载到内存中，这对一定大小的表比较适用，但两个表都很大时，无论加载哪个表到内存都不理想。所以spark sql使用sort merge的方式。
步骤：
3. 对两张表分别进行shuffle重分区，将相同key的记录分到对应分区中并进行排序，这一步对应Exchange节点和sort节点。也就是spark 的sort merge shuffle过程。
4. 遍历流式表，对每条记录都采用顺序查找的方式从查找表中搜索，由于排序的特性，每次处理完一条记录，只需从上一次结束的位置开始继续查找。整体上来说，查找性能还是较优的。

[图片]


为什么要调节并行度？
  - 并行度如果太小，每个reducer处理的数据量多，task不停的溢写会造成大量的磁盘io，导致task执行时间过长，如果内存比较小+当时的物理机不是很健康(很常见)的情况，轻则gc时间过长，重则直接任务失败。
  - 并行度如果太大，每个task处理的数据量小，driver的压力会很大（调度task），并且task多，shuffle的时候map阶段(shuffle write)写的文件也会变多，reduce段去拉取数据的时候会读取大量的小文件。
 需要注意最大并行度的设置，不要设为2的幂次方（收起程序员的强迫症），这是由于Hash分区函数的特性导致
 
 并行度越大越好吗？为什么？
 并不是。在每个task处理的数据量过多时，可以适当增加并行度来减少每个task处理的数据量，以此增加速度。但当task处理的数据量非常小时，可以适当的减小并行度来减少task个数，因为driver在生成、分配、监视task时需要消耗时间，executor在切换处理task时也需要消耗时间。这些额外的消耗积少成多，增加了任务总时长。


### 输出小文件合并
Spark输出文件的数量由最后一个stage的并行度来决定。

合并小文件的目的：
1. 为了下游任务使用时task数量变少，虽然有相关参数调节，但从根本上解决最好。
2. hdfs是设计为存储大块儿数据使用的，如果分割的文件太小，则会导致文件数过多，那么管理文件元数据的namenode就会有很大压力。


### 一个sql语句是如何处理数据的
[图片1]
1. 逻辑计划：
  1. 未解析的逻辑算子树：仅有数据结构，不包含数据信息。
  2. 解析的逻辑算子树：节点中绑定各种信息。
  3. 优化后的逻辑算子树：应用各种优化规则对一些抵消的逻辑计划进行转换。
2. 物理计划：
  1. 根据逻辑算子树生成物理算子树的列表（可能1对多）
  2. 从列表中按一定的策略选取最优的物理算子树。
  3. 对选取的物理算子树进行提交前的准备工作：确保分区操作正确、物理算子树节点重用、执行代码生成等。

[图片2]
以上步骤都在driver端进行，不涉及分布式计算






