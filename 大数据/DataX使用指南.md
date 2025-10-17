# DataX使用指南

## 1.模板配置

```bash
datax.py -r oraclereader -w kingbaseeswriter > oracle_to_kingbase.json
```

### oracle2kingbase

```json
{
    "job": {
        "content": [
            {
                "reader": {
                    "name": "oraclereader", 
                    "parameter": {
                        "column": ["ID", "NAME", "AGE", "SCHOOL", "CLASS", "CREATETIME", "UPDATETIME"], 
                        "connection": [
                            {
                                "jdbcUrl": ["jdbc:oracle:thin:@10.201.100.75:1521:cesdb"], 
                                "table": ["test_student"]
                            }
                        ], 
                        "password": "********", 
                        "username": "ods_user"
                    }
                }, 
                "writer": {
                    "name": "kingbaseeswriter", 
                    "parameter": {
                        "column": ["ID", "NAME", "AGE", "SCHOOL", "CLASS", "CREATETIME", "UPDATETIME"], 
                        "connection": [
                            {
                                "jdbcUrl": "jdbc:kingbase8://192.168.200.180:54321/test", 
                                "table": ["ods.test_student"]
                            }
                        ], 
                        "password": "*****", 
                        "postSql": [], 
                        "preSql": ["TRUNCATE TABLE ods.test_student"], 
                        "username": "data"
                    }
                }
            }
        ], 
        "setting": {
            "speed": {
                "channel": "1"
            }
        }
    }
}
```

sql模式

` datax.py -p "-Dstart_time=2025-08-04 -Dend_time=2025-08-05" oracle2kingbase.json `

```json
{
    "job": {
        "content": [
            {
                "reader": {
                    "name": "oraclereader", 
                    "parameter": {
                        "connection": [
                            {
                                "jdbcUrl": ["jdbc:oracle:thin:@10.201.100.75:1521:cesdb"], 
                                "querySql": ["SELECT ID, NAME, AGE, SCHOOL, CLASS, CREATETIME, UPDATETIME FROM test_student WHERE UPDATETIME >= TO_DATE('$start_time', 'YYYY-MM-DD') AND UPDATETIME < TO_DATE('$end_time', 'YYYY-MM-DD')"]
                            }
                        ], 
                        "password": "****", 
                        "username": "ods_user"
                    }
                }, 
                "writer": {
                    "name": "kingbaseeswriter", 
                    "parameter": {
                        "column": ["ID", "NAME", "AGE", "SCHOOL", "CLASS", "CREATETIME", "UPDATETIME"], 
                        "connection": [
                            {
                                "jdbcUrl": "jdbc:kingbase8://192.168.200.180:54321/test", 
                                "table": ["ods.test_student"]
                            }
                        ], 
                        "password": "**********", 
                        "postSql": [], 
                        "preSql": ["TRUNCATE TABLE ods.test_student"], 
                        "username": "data"
                    }
                }
            }
        ], 
        "setting": {
            "speed": {
                "channel": "1"
            }
        }
    }
}

```

设置时间参数。start_time和end_time不用添加{}

```bash
datax.py -p "-Dstart_time=2025-08-01 -Dend_time=2025-08-02" oracle2kingbase.json
```

### sqlserver2kingbase

```json
{
    "job": {
        "content": [
            {
                "reader": {
                    "name": "sqlserverreader", 
                    "parameter": {
                        "column": ["FID","FPRN","FTIMES","FZDLX","FICDVERSION","FICDM","FJBNAME","FZLJGBH","FZLJG","FPX","FRYBQBH","FRYBQ"],
                        "connection": [
                            {
                                "jdbcUrl": ["jdbc:sqlserver://;serverName=199.168.200.244;databaseName=bagl_java"], 
                                "table": ["TDIAGNOSE"]
                            }
                        ], 
                        "password": "***" 
                        "username": "sa"
                    }
                }, 
                "writer": {
                    "name": "kingbaseeswriter", 
                    "parameter": {
                        "column": ["FID","FPRN","FTIMES","FZDLX","FICDVERSION","FICDM","FJBNAME","FZLJGBH","FZLJG","FPX","FRYBQBH","FRYBQ"], 
                        "connection": [
                            {
                                "jdbcUrl": "jdbc:kingbase8://192.168.200.180:54321/test", 
                                "table": ["ods.bagl_diagnosis_df"]
                            }
                        ], 
                        "password": "****", 
                        "postSql": [], 
                        "preSql": [], 
                        "username": "data"
                    }
                }
            }
        ], 
        "setting": {
            "speed": {
                "channel": "1"
            }
        }
    }
}
```

1个channer，一千多万条的数据，同步速度为270s，相当于每秒38万条左右。

```
任务启动时刻                    : 2025-09-22 16:53:17
任务结束时刻                    : 2025-09-22 16:57:48
任务总计耗时                    :                270s
任务平均流量                    :            1.37MB/s
记录写入速度                    :          38234rec/s
读出记录总数                    :            10323224
读写失败总数                    :                   0
```

10个channer，但是速率并没有提上来，说明配置并没有生效，或者机器资源不支持？但是不应该呀，我64核128GB的机器应该是支持的

```
任务启动时刻                    : 2025-09-22 17:04:23
任务结束时刻                    : 2025-09-22 17:08:54
任务总计耗时                    :                271s
任务平均流量                    :            1.37MB/s
记录写入速度                    :          38234rec/s
读出记录总数                    :            10323371
读写失败总数                    :                   0
```

优化版本

```json
{
    "core":{
        "transport":{
            "channel":{
                "speed":{   
                    "byte": 10485760        //单个channel限速10MB/S，计算出有10个channel。默认的限速是1MB/S            
                }
            }
        }
    },
    "job": {
        "content": [
            {
                "reader": {
                    "name": "sqlserverreader", 
                    "parameter": {
                        "column": ["FID","FPRN","FTIMES","FZDLX","FICDVERSION","FICDM","FJBNAME","FZLJGBH","FZLJG","FPX","FRYBQBH","FRYBQ"],
                        "connection": [
                            {
                                "jdbcUrl": ["jdbc:sqlserver://;serverName=199.168.200.244;databaseName=bagl_java"], 
                                "table": ["TDIAGNOSE"]
                            }
                        ], 
                        "password": "****", 
                        "username": "sa"
                    }
                }, 
                "writer": {
                    "name": "kingbaseeswriter", 
                    "parameter": {
                        "column": ["FID","FPRN","FTIMES","FZDLX","FICDVERSION","FICDM","FJBNAME","FZLJGBH","FZLJG","FPX","FRYBQBH","FRYBQ"], 
                        "connection": [
                            {
                                "jdbcUrl": "jdbc:kingbase8://192.168.200.180:54321/test", 
                                "table": ["ods.bagl_diagnosis_df"]
                            }
                        ], 
                        "password": "***", 
                        "postSql": [], 
                        "preSql": ["TRUNCATE TABLE ods.bagl_diagnosis_df"], 
                        "username": "data"
                    }
                }
            }
        ], 
        "setting": {
            "speed": {
                "byte": 104857600 //整体限速 100MB/S
            }
        }
    }
}
```

反而更慢了，从日志分析来看，问题主要在于WaitWriterTime过高

- **传输速度**: 1.32MB/s ❌（太慢）
- **WaitWriterTime**: 190-226秒 ⚠️（非常高）
- **WaitReaderTime**: 14-17秒 ✅（正常）
- **结论**: 写入端（KingbaseES）是性能瓶颈

```bash
任务启动时刻                    : 2025-09-23 15:18:32
任务结束时刻                    : 2025-09-23 15:23:13
任务总计耗时                    :                280s
任务平均流量                    :            1.32MB/s
记录写入速度                    :          36878rec/s
读出记录总数                    :            10325932
读写失败总数                    :                   0
```

### oracle2hdfs

全量同步

```json
{
    "job": {
        "content": [
            {
                "reader": {
                    "name": "oraclereader", 
                    "parameter": {
                        "column": ["ID", "NAME", "AGE", "SCHOOL", "CLASS", "CREATETIME", "UPDATETIME"], 
                        "connection": [
                            {
                                "jdbcUrl": ["jdbc:oracle:thin:@10.201.100.75:1521:cesdb"], 
                                "table": ["test_student"]
                            }
                        ], 
                        "password": "********", 
                        "username": "ods_user"
                    }
                }, 
                "writer": {
                    "name": "hdfswriter", 
                    "parameter": {
                        "column": [
                            {"name":"id","type":"string"},
                            {"name":"name","type":"string"},
                            {"name":"age","type":"string"},
                            {"name":"school","type":"string"},
                            {"name":"class","type":"string"},
                            {"name":"createtime","type":"string"},
                            {"name":"updatetime","type":"string"}
                        ], 
                        "defaultFS": "hdfs://cesdb:8020", 
                        "fieldDelimiter": "\t", 
                        "fileName": "test_student", 
                        "fileType": "orc", 
                        "path": "/user/hive/warehouse/test.db/test_student", 
                        "writeMode": "truncate"
                    }
                }
            }
        ], 
        "setting": {
            "speed": {
                "channel": "1"
            }
        }
    }
}

```



# DS调度datax

因为我的dolphinscheduler是分布式，所以要想保证每个节点都能够正常调度datax，需要每个节点都安装datax

分发datax节点

```bash
xsync.sh /data/u01/app/datax

# 每个节点自检
python /data/u01/app/datax/datax/bin/datax.py /data/u01/app/datax/datax/job/job.json 
```

修改dolphinscheduler_env.sh

```bash
cd /data/u01/app/dolphinscheduler/apache-dolphinscheduler-3.2.2-bin/bin/env
vim dolphinscheduler_env.sh

export DATAX_HOME=${DATAX_HOME:-/data/u01/app/datax/datax}
# 添加DataX相关配置
export PYTHON_LAUNCHER=/usr/bin/python
export DATAX_LAUNCHER=/data/u01/app/datax/datax/bin/datax.py
export PATH=$HADOOP_HOME/bin:$JAVA_HOME/bin:$HIVE_HOME/bin:$DATAX_HOME/bin:$PATH

# 分发
xsync.sh dolphinscheduler_env.sh
```

dolphinscheduler配置datax环境变量

![](D:\Github\MyKnowledgeRepository\img\bigdata\dolphinscheduler\datax环境变量.png)

配置流程，添加环境

![](D:\Github\MyKnowledgeRepository\img\bigdata\dolphinscheduler\配置datax流程.png)

设置时间参数。start_time和end_time要添加{}

参考文档：[dolphinscheduler内置参数](https://dolphinscheduler.apache.org/zh-cn/docs/3.3.1/guide/parameter/built-in)

```json
{
    "job": {
        "content": [
            {
                "reader": {
                    "name": "oraclereader", 
                    "parameter": {
                        "connection": [
                            {
                                "jdbcUrl": ["jdbc:oracle:thin:@10.201.100.75:1521:cesdb"], 
                                "querySql": ["SELECT ID, NAME, AGE, SCHOOL, CLASS, CREATETIME, UPDATETIME FROM test_student WHERE UPDATETIME >= TO_DATE('${start_time}', 'YYYY-MM-DD') AND UPDATETIME < TO_DATE('${end_time}', 'YYYY-MM-DD')"]
                            }
                        ], 
                        "password": "*******", 
                        "username": "ods_user"
                    }
                }, 
                "writer": {
                    "name": "kingbaseeswriter", 
                    "parameter": {
                        "column": ["ID", "NAME", "AGE", "SCHOOL", "CLASS", "CREATETIME", "UPDATETIME"], 
                        "connection": [
                            {
                                "jdbcUrl": "jdbc:kingbase8://192.168.200.180:54321/test", 
                                "table": ["ods.test_student"]
                            }
                        ], 
                        "password": "**********", 
                        "postSql": [], 
                        "preSql": ["TRUNCATE TABLE ods.test_student"], 
                        "username": "data"
                    }
                }
            }
        ], 
        "setting": {
            "speed": {
                "channel": "1"
            }
        }
    }
}
```

![](D:\Github\MyKnowledgeRepository\img\bigdata\dolphinscheduler\dolphinscheduler参数设置.png)

# 报错及解决

## 1.The authentication type 10 is not supported

datax同步oracle数据到kingbase报这个错误，这个错误是kingbase驱动版本不匹配导致的。

解决：检查kingbase版本和datax(/data/u01/app/datax/datax/plugin/writer/kingbaseeswriter/libs)目录下kingbase驱动jar包版本是否太低，替换对应的高版本jar包即可解决问题。
