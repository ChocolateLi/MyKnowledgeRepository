# mapreduce调优

没做任务参数优化的时候。运行时间花了30分钟。map任务数达到了804个，reduce任务数达到了904个。

![](D:\Github\MyKnowledgeRepository\img\bigdata\mapreduce\过程1.png)

做了以下参数优化。比较关键的一步是将renduce的最大任务限制在300。这些参数的设置运行效率提高了7分钟。但是map数依旧很高。

```sql
set mapreduce.job.queuename=queue1;
set mapreduce.map.memory.mb=4096;
set mapreduce.map.java.opts=-Xmx3072m;
set hive.exec.reducers.bytes.per.reducer=512000000;
set hive.exec.reducers.max=300;
set mapreduce.reduce.memory.mb=8192;
set mapreduce.reduce.java.opts=-Xmx6144m;
set mapreduce.map.output.compress=true;
set mapreduce.map.output.compress.codec=org.apache.hadoop.io.compress.SnappyCodec;
set hive.exec.compress.output=true;
set mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.SnappyCodec;
set mapreduce.output.fileoutputformat.compress.type=BLOCK;
set hive.auto.convert.join=true;
set hive.optimize.sort.dynamic.partition=true;
```

![](D:\Github\MyKnowledgeRepository\img\bigdata\mapreduce\过程2.png)

map数跟切片有关，目前的配置是最大切片为256M，最小切片为1M。导致划分的切片太多，从而导致map任务数太多。所以需要调整切片的大小。切片大小不能直接设置，需要修改配置文件，hive-site.xml。我将切片大小提高到了1GB，最小切片大小为512M。这极大减少了map任务数，将运行效率提高到16分钟完成。

```sql
<property>
  <name>mapreduce.input.fileinputformat.split.maxsize</name>
  <value>1024000000</value> <!-- 设置为1GB -->
</property>
<property>
  <name>mapreduce.input.fileinputformat.split.minsize</name>
  <value>512000000</value> <!-- 设置为512MB -->
</property>
```

![](D:\Github\MyKnowledgeRepository\img\bigdata\mapreduce\过程3.png)

到了这一步能够可优化的点就很少了，但是reduce任务还可以优化，map的任务数是200个左右，reduce只需要它的一半即可，即设置为100。以下是完整参数优化。

```sql
set mapreduce.job.queuename=queue1;
set mapreduce.map.memory.mb=4096;--map任务内存
set mapreduce.map.java.opts=-Xmx3072m;--map jvm的内存堆大小
set hive.exec.reducers.bytes.per.reducer=512000000;--设置Reduce处理的数据量，单位为字节。此处设置为512MB（默认值通常为64MB）。我感觉设置小了，应该可以设置为4GB，毕竟我内存都给了8GB给他。
set hive.exec.reducers.max=100;--设置reduce最大任务数
set mapreduce.reduce.memory.mb=8192;--reduce任务内存
set mapreduce.reduce.java.opts=-Xmx6144m;--reduce jvm的内存堆大小
set mapreduce.map.output.compress=true;----map jvm的内存堆大小
set mapreduce.map.output.compress.codec=org.apache.hadoop.io.compress.SnappyCodec;--设置Map阶段中间数据的压缩算法为Snappy。Snappy是一种高效、快速的压缩算法，压缩速度快，解压开销低，适用于大数据场景。
set hive.exec.compress.output=true;--启用Reduce阶段的最终输出压缩。输出压缩可以减少存储占用和后续处理数据的传输开销。
set mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.SnappyCodec;
set mapreduce.output.fileoutputformat.compress.type=BLOCK;--设置压缩类型（BLOCK或RECORD）
set hive.auto.convert.join=true;--启用自动MapJoin优化
set hive.optimize.sort.dynamic.partition=true;--启用动态分区排序优化
```

这样设置参数任务，可以将30分钟的任务，提高到15分钟完成，运行效率整整提高了一倍。（这是一个很恐怖的优化，为什么这么说呢，如果这个任务是8小时的任务，通过这样优化，只需要4小时跑完，这节约多少时间成本和资源。）

![](D:\Github\MyKnowledgeRepository\img\bigdata\mapreduce\过程4.png)

进一步优化reduce处理的数据量，将其设置为4GB。

```sql
set hive.exec.reducers.bytes.per.reducer=4098000000;--设置Reduce处理的数据量，单位为字节。此处设置为512MB（默认值通常为64MB）。我感觉设置小了，应该可以设置为4GB，毕竟我内存都给了8GB给他。
```

所以最终的任务参数如下：

```sql
set mapreduce.job.queuename=queue1;
set mapreduce.map.memory.mb=4096;
set mapreduce.map.java.opts=-Xmx3072m;
set hive.exec.reducers.bytes.per.reducer=4096000000;
set hive.exec.reducers.max=100;
set mapreduce.reduce.memory.mb=8192;
set mapreduce.reduce.java.opts=-Xmx6144m;
set mapreduce.map.output.compress=true;
set mapreduce.map.output.compress.codec=org.apache.hadoop.io.compress.SnappyCodec;
set hive.exec.compress.output=true;
set mapreduce.output.fileoutputformat.compress.codec=org.apache.hadoop.io.compress.SnappyCodec;
set mapreduce.output.fileoutputformat.compress.type=BLOCK;
set hive.auto.convert.join=true;
set hive.optimize.sort.dynamic.partition=true;
```

可以通过以下方式查看是否设置成功

```sql
set mapreduce.input.fileinputformat.split.maxsize;
set mapreduce.input.fileinputformat.split.minsize;
set hive.input.format;
set mapreduce.job.queuename;
set mapreduce.map.memory.mb;
set mapreduce.map.java.opts;
set hive.exec.reducers.bytes.per.reducer;
set hive.exec.reducers.max;
set mapreduce.reduce.memory.mb;
set mapreduce.reduce.java.opts;
set mapreduce.map.output.compress;
set mapreduce.map.output.compress.codec;
set hive.exec.compress.output;
;set mapreduce.output.fileoutputformat.compress.codec;
set mapreduce.output.fileoutputformat.compress.type;
set hive.auto.convert.join;
set hive.optimize.sort.dynamic.partition;
```

reduece任务数减小到了57，从904个任务优化到57个任务数。但是整体的运行时间差不了多少，大概快20秒左右。

![](D:\Github\MyKnowledgeRepository\img\bigdata\mapreduce\过程5.png)

**总结**

运行时间：从30分钟优化到了15分钟，运行效率整整提高了1倍。

map数量：从804个优化到了223个，map数量减少了4倍左右。

reduce数量：从904个优化到了57个，reduce数量减少了8倍左右。



## **有时候觉得自己就是个小丑**

当你资源给的足够多的时候，一切的优化手段就显得不堪一击。之前的优化是基于只给集群40%的资源的情况下。当我把百分百资源给他的时候，他就能迅速完成计算。

![](D:\Github\MyKnowledgeRepository\img\bigdata\mapreduce\过程6.jpg)

# Spark并不一定比mapreduce快

使用mr引擎和spark引擎分别执行以下sql，mr的速度比spark快，mr花费 2min21s，spark花费 6min32s。

```sql
select  
trunc(a.ADVICE_GIVEN_TIME,'MM') as date_start
,count(distinct a.VISITING_SERIAL_NUMBER) statist_result	--统计结果
from ods.ods_cdr_cis_inhos_dradvice_detail_mi a
inner join ods.ods_cdr_cis_inhos_dradvice_detail_mi b on a.VISITING_SERIAL_NUMBER=b.VISITING_SERIAL_NUMBER 
                                          and a.ADVICE_GIVEN_TIME<b.ADVICE_GIVEN_TIME 
                                          and b.REVOCAT_SIGN='1' 
                                          and b.ADVICE_GIVEN_DEP_CODE IN ('541','1258','1250','582','163','161','1254','775','169','773','165')  
                                          and b.hospital_item_code IN('311202004','N330100013','N330100014','N330100014-1','N330100013E','N330100014E','N330100013T','N330100014T')
where a.year='2024' and b.year='2024'
and a.ADVICE_GIVEN_TIME >= cast('2024-01-01' as timestamp) and a.ADVICE_GIVEN_TIME<cast('2025-01-01' as timestamp)
and a.hospital_item_code='ZT0863' 
and a.ADVICE_GIVEN_DEP_CODE IN ('541','1258','1250','582','163','161','1254','775','169','773','165') 
and a.REVOCAT_SIGN='1'
and (UNIX_TIMESTAMP(b.ADVICE_GIVEN_TIME) - UNIX_TIMESTAMP(a.ADVICE_GIVEN_TIME)) / 3600 <=48 
group by trunc(a.ADVICE_GIVEN_TIME,'MM');
```

在这个例子中，MR（MapReduce）引擎比Spark引擎的性能更快，可能是由以下几个原因导致的：

1. **数据处理模式的不同**：
   - **MapReduce（MR）**：MapReduce是传统的分布式计算框架，它适用于大规模的数据集。MR的计算模型比较简单，适用于对数据进行顺序处理（比如逐行扫描数据，进行简单的分组、筛选等）。虽然MR的处理速度在某些复杂任务上较慢，但在简单的JOIN和过滤操作上，它有时表现得比Spark更高效。
   - **Spark**：Spark的执行引擎比MR更复杂，它使用内存中的数据处理和计算优化。然而，Spark会有一些开销，比如任务调度、内存管理等。如果这些优化不被有效利用，或者任务本身的数据量非常庞大且复杂时，Spark的性能可能会较差。
2. **数据倾斜问题**： 这条SQL查询有一个双表的自连接操作（inner join），这对于MR和Spark来说都可能引起**数据倾斜**问题。具体而言，如果表`ods_cdr_cis_inhos_dradvice_detail_mi`非常大，且某些`VISITING_SERIAL_NUMBER`值的记录数量远大于其他值，Spark可能在分配任务时出现不均衡，导致某些节点处理的数据量过大，从而影响性能。
   - **MR的处理方式**可能在处理这种数据倾斜时，利用了更为简单的规则，使得它的执行更加稳定。
   - **Spark**在这种情况下，可能由于任务分配不均衡，导致部分节点负载过重，从而使得整体执行时间增大。
3. **资源配置和并行度**： Spark通常依赖于集群的资源配置（如内存和CPU的分配）。如果Spark集群的配置不够优化，可能导致计算瓶颈。Spark的性能还受到许多因素的影响，如任务调度器、内存分配策略、并行度设置等。如果这些没有得到优化，可能导致执行时间远超MR。
4. **内存管理**： Spark的内存管理机制比MR复杂。如果Spark在处理大数据时无法有效管理内存，可能会频繁进行垃圾回收（GC）或者溢写到磁盘，导致性能下降。而MR的计算方式较为简单，通常只依赖于磁盘IO进行处理，因此在某些场景下，MR的执行可能反而更稳定。
5. **查询优化和物理执行计划**： Spark执行SQL时会生成物理执行计划。如果在某些情况下，Spark的查询优化器（Catalyst）没有产生最优的执行计划，可能会导致不必要的Shuffle操作，从而拖慢执行速度。

## 是否合理？

从传统的意义上讲，Spark比MapReduce应该要更快，尤其是在内存计算和数据处理的灵活性方面。然而，在某些特殊情况下，比如数据量过大、存在数据倾斜、配置不优化等，MapReduce的稳定性和简单性可能使它在某些场景下反而执行得更快。

## 可能的改进措施：

1. **检查Spark的资源配置**，如内存、CPU等分配情况，确保其资源能够满足处理需求。
2. **优化数据分区和并行度**，避免数据倾斜，确保每个节点的负载均衡。
3. **优化查询执行计划**，使用`EXPLAIN`分析Spark的执行计划，看看是否有不必要的Shuffle操作，可以通过调整SQL或配置参数来改进。
4. **调整缓存和内存管理策略**，确保Spark能够高效地处理数据，减少磁盘IO。

因此，MR比Spark更快并不一定是不合理的，可能是因为Spark在特定的环境或配置下没有发挥出最佳性能。

## 总结

在没有任何优化配置下，有些场景spark是比mr快，但有些场景是mr比spark快的。简单查询场景下，spark性能是远远大于mr性能的，在大规模大数据集下，如果不做任何优化措施下，spark的速度未必优于mr，反而因为MapReduce的稳定性和简单性可能使它在某些场景下反而执行得更快。