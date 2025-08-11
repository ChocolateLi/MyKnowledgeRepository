# KingBase

# 安装

参考链接：[Linux平台安装](https://docs.kingbase.com.cn/cn/KES-V9R1C10/install/iso/iso-linux-install#%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%A2%9E%E5%88%A0%E7%BB%84%E4%BB%B6)

## 安装前准备工作

安装用户

```bash
[root@doris soft]# useradd -u2000 kingbase
[root@doris soft]# passwd kingbase
```

安装目录

```bash
# 自定义的安装目录
[root@doris app]# mkdir -p /data/u01/app/kingbase/ES/V9
[root@doris app]# chmod o+rwx /data/u01/app/kingbase/ES/V9/
# 更改用户组和用户
[root@doris app]# chown -R kingbase:kingbase /data/u01/app/kingbase
```

数据目录

```bash
# 切换kingbase用户操作，无特殊说明，接下来都是用kingbase用户操作
su kingbase
[kingbase@doris kingbase]$ mkdir -p /data/u01/app/kingbase/ES/V9/data
```

安装包准备

```bash
# 切换root用户
# 将安装文件上传至/data/u01/soft
KingbaseES_V009R001C010B0004_Lin64_install.iso
# 创建目录目录
[root@doris soft]# mkdir KingbaseESV9
[root@doris soft]# ll
总用量 6790868
drwxr-xr-x. 3 root root         24 7月   3 16:45 apache-doris-2.1.10-bin-x64
-rw-r--r--. 1 root root 2933616665 6月  26 14:34 apache-doris-2.1.10-bin-x64.tar.gz
drwxr-xr-x. 2 root root       4096 6月  24 17:34 gcc
-rw-r--r--. 1 root root 2974226432 8月   5 16:19 KingbaseES_V009R001C010B0004_Lin64_install.iso
drwxr-xr-x. 2 root root          6 8月   6 10:35 KingbaseESV9
-rw-r--r--. 1 root root 1045995520 7月   4 10:14 mysql-8.0.38-1.el7.x86_64.rpm-bundle.tar
drwxrwxr-x. 2 root root        130 7月   4 15:07 telnet
```

安装包的挂载与取消

```bash
# iso格式的安装程序包需要先挂载才能使用,需要使用root用户挂载iso文件
[root@doris soft]# cd /data/u01/soft/
[root@doris soft]# mount KingbaseES_V009R001C010B0004_Lin64_install.iso ./KingbaseESV9
mount: /dev/loop0 写保护，将以只读方式挂载
[root@doris soft]# cd ./KingbaseESV9/
# 可以看到setup目录和setup.sh脚本
[root@doris KingbaseESV9]# ls
setup  setup.sh
# 安装完成后可以运行如下命令取消挂载.iso文件
umount ./KingbaseESV9
# 此时KingbaseES已经和iso文件解除挂载关系，在KingbaseES目录下不会再看到安装相关文件。
```

## 安装KingbaseES

1.启动安装程序

```bash
# 先查看系统语言
[root@doris backup]# echo $LANG
zh_CN.UTF-8
# 进入setup.sh所在目录，以kingbase用户执行
[root@doris backup]# cd /data/u01/soft/KingbaseESV9/
[root@doris KingbaseESV9]# su kingbase
[kingbase@doris KingbaseESV9]$ sh setup.sh -i console

# 输入一个绝对路径，或按ENTER键以接受缺省路径 [默认： /opt/Kingbase/ES/V9]
/data/u01/app/kingbase/ES/V9

# 最后成功的界面
  恭喜您！安装完成

  安装目录：
    /data/u01/app/kingbase/ES/V9

  
    引导组件:/data/u01/app/kingbase/ES/V9/KESRealPro/V009R001C010/install
    产品手册:/data/u01/app/kingbase/ES/V9/KESRealPro/V009R001C010/doc
    数据库运维工具:/data/u01/app/kingbase/ES/V9/KESRealPro/V009R001C010/SupTools
    数据库服务器:/data/u01/app/kingbase/ES/V9/KESRealPro/V009R001C010/Server
    高可用组件:/data/u01/app/kingbase/ES/V9/KESRealPro/V009R001C010/KingbaseHA
    接口:/data/u01/app/kingbase/ES/V9/KESRealPro/V009R001C010/Interface
    数据库集群部署工具:/data/u01/app/kingbase/ES/V9/KESRealPro/V009R001C010/ClientTools/guitools/DeployTools
    数据库迁移工具:/data/u01/app/kingbase/ES/V9/KESRealPro/V009R001C010/ClientTools/guitools/KDts
    数据库开发工具(CS):/data/u01/app/kingbase/ES/V9/KESRealPro/V009R001C010/ClientTools/guitools/KStudio


如需初始化数据库，请启动Kconsole:
/data/u01/app/kingbase/ES/V9/Server/bin/kconsole.sh
手动初始化数据库:
/data/u01/app/kingbase/ES/V9/Server/bin/initdb -U "system" -W -D "/data/u01/app/kingbase/ES/V9/data"
[ Writing the uninstaller data ... ]
[ 命令行安装完成 ]
```

## 初始化数据库

```bash
[kingbase@doris bin]$ /data/u01/app/kingbase/ES/V9/Server/bin/initdb -U "system" -W -D "/data/u01/app/kingbase/ES/V9/data"
创建目录 /data/u01/app/kingbase/ES/V9/data ... 成功
正在创建子目录 ... 成功
选择动态共享内存实现 ......posix
选择默认最大联接数 (max_connections) ... 100
选择默认共享缓冲区大小 (shared_buffers) ... 128MB
选择默认时区...Asia/Shanghai
创建配置文件 ... 成功
开始设置加密设备
正在初始化加密设备...成功
正在运行自举脚本 ...成功
正在执行自举后初始化 ...成功
创建安全数据库...成功
加载安全数据库...成功
同步数据到磁盘...成功

initdb: 警告: enabling "trust" authentication for local connections
你可以通过编辑 sys_hba.conf 更改或你下次
执行 initdb 时使用 -A或者--auth-local和--auth-host选项.

成功。您现在可以用下面的命令开启数据库服务器：

    /data/u01/app/kingbase/ES/V9/Server/bin/sys_ctl -D /data/u01/app/kingbase/ES/V9/data -l 日志文件 start

```

启动数据库

```bash
[kingbase@doris data]$ /data/u01/app/kingbase/ES/V9/Server/bin/sys_ctl -D /data/u01/app/kingbase/ES/V9/data -l 日志文件 start
等待服务器进程启动 .... 完成
服务器进程已经启动
[kingbase@doris data]$ netstat -tulnp | grep 54321
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 0.0.0.0:54321           0.0.0.0:*               LISTEN      24934/kingbase      
tcp6       0      0 :::54321                :::*                    LISTEN      24934/kingbase      
[kingbase@doris data]$ netstat -tulnp | grep 54321
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 0.0.0.0:54321           0.0.0.0:*               LISTEN      24934/kingbase      
tcp6       0      0 :::54321                :::*                    LISTEN      24934/kingbase      
[kingbase@doris data]$ 
```

登录数据

```bash
./ksql -U system test;
```

