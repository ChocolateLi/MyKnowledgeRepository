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