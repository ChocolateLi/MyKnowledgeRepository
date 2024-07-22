# Hadoop集群运维

# 脚本配置

## jps脚本

```shell
touch jps.sh
vim jps.sh

#!/bin/bash
echo "==============查询当前所有服务器的jps情况=============="
        for i in cesdb cesdb1 cesdb2
        do
                echo "**************$i当前jps情况***************"
                ssh $i '/usr/jdk/bin/jps'
        done

echo "=======================查询完毕========================"

chmod -R 777 jps.sh

vim /etc/profile

# jps.sh的脚本路径(添加到最后一行)
export PATH=$PATH:/data/chenli/sh

source /etc/profile

# 然后就可以全局使用了
[hadoop@cesdb sh]$ jps.sh
==============查询当前所有服务器的jps情况==============
**************cesdb当前jps情况***************
10113 DataNode
10787 ResourceManager
9893 NameNode
11014 NodeManager
149652 Jps
13933 RunJar
13934 RunJar
**************cesdb1当前jps情况***************
155888 DataNode
33810 Jps
156072 SecondaryNameNode
156361 NodeManager
**************cesdb2当前jps情况***************
136547 NodeManager
136208 DataNode
13707 Jps
=======================查询完毕========================

```

