# Hadoop 概述

> 尚硅谷Hadoop教程：BV1cW411r7c5

## 1. 大数据概论

- 大数据主要解决海量数据的存储和分析计算问题
- 海量指 `TB` `PB` `EB` 级别数据
- 大数据特点： Volume（大量）、Velocity（高速）、Variety（多样）、Value（低价值密度）
- 数据分为结构化数据（数据库/文本为主）和非结构化数据（网络日志、音视频、图片、地理位置信息等）

## 2. Hadoop 基础

### 2.1 Hadoop 介绍

- Hadoop 是由 Apache 基金会开发的分布式系统基础架构
- Hadoop 主要解决海量数据的存储和分析计算问题
- Hadoop 三个主要发行版本： Apache、Cloudera、Hortonworks
- Hadoop 具有高可靠性、高扩展性、高效性和高容错性
  - 高可靠性：底层维护多个（至少 3 个）数据副本，即使某个计算元素或储存出现故障也不会导致数据丢失
  - 高扩展性：在集群间分配任务数据，可方便地扩展数以千计的节点
  - 高效性：在 MapReduce 思想下，Hadoop 是并行工作的，以加快任务处理速度
  - 高容错性：能够自动将失败的任务重新分配

### 2.2 Hadoop 组成

- Hadoop 1.x
  - MapReduce （计算+资源调度）
  - HDFS （数据储存）
  - Common （辅助工具）
- Hadoop 2.x
  - MapReduce （计算）
  - Yarn （资源调度）
  - HDFS （数据储存）
  - Common （辅助工具）
- HDFS 架构
  - NameNode (nn)：存储文件的元数据，如文件名、文件目录结构、文件属性（生成时间、副本数、文件权限）以及每个文件的块列表和块所在的 DataNode 等
  - DataNode (dn)：在本地文件系统储存文件块数据，以及块数据的校验和
  - Secondary NameNode (2nn)：用来监视 HDFS 状态的辅助后台程序，每隔一段时间获取 HDFS 元数据的快照
- YARN 架构
  - ResourceManager (RM)：
    - 处理客户端请求
    - 监控 NodeManager
    - 启动或监控 ApplicationMaster (MapReduce Status)
  - NodeManager (NM)：
    - 管理单个节点上的资源
    - 处理来自 ResourceManager 的命令
    - 处理来自 ApplicationMaster 的命令 
  - ApplicationMaster (AM)：
    - 负责数据的切分
    - 为应用程序申请资源并分配给内部的任务
    - 任务的监控与容错
  - Container
    - YARN 中资源的抽象，封装了某个节点上的多维度资源，如内存、CPU、磁盘、网络等
- MapReduce 架构
  - Map 阶段并行处理输入数据
  - Reduce 阶段对 Map 结果进行汇总

### 2.3 Hadoop 环境搭建

- VMWare 15 + CentOS 7

  > 下列 `bash` 操作如无特殊说明，均是在 `root` 账户 `~` 目录下完成
  >
  > 参考了[这篇](https://blog.csdn.net/ErnestW/article/details/88929968?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.edu_weight&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.edu_weight)和[这篇](https://blog.csdn.net/aic1999/article/details/104660016#9-hadoop%E4%BC%AA%E5%88%86%E5%B8%83%E5%BC%8F%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AEhdfs)文章

  - 安装过程略，建议使用[这个镜像](https://mirrors.bfsu.edu.cn/centos/7.8.2003/isos/x86_64/CentOS-7-x86_64-Minimal-2003.iso)

  - 在 VMWare 中加载镜像新建虚拟机，建议配置：内存 8G，虚拟硬盘 20G 以上

  - 安装后，设置网络连接

    > 视实际情况，内容可能有所不同。IP 可在主机上查看 VMnet8 的信息

    ```bash
    $ vi /etc/sysconfig/network-scripts/ifcfg-ens33
    # 在文件中添加下列行
    > IPADDR=192.168.38.129
    > GATEWAY=192.168.38.2
    > NETMASK=255.255.255.0
    > DNS1=114.114.114.114
    # 在文件中修改下列行
    > BOOTPROTO=none
    > ONBOOT=yes
    $ service network restart
    Restarting network (via systemctl):				[ OK ]
    ```
  
- 网络连接设置完成后，换源
  
    ```bash
    $ mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
    $ wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
    # 如果没有 wget 则运行下面这条指令
    # $ curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
    $ yum clean all
    $ yum makecache
    $ yum list | grep epel-release
    $ yum install epel-release
    # 如果没有 wget 则先运行下面这条指令
    # $ yum install wget
    $ wget -O /etc/yum.repos.d/epel-7.repo http://mirrors.aliyun.com/repo/epel-7.repo
    $ yum clean all
    $ yum makecache
  ```
  
- 更改 Hosts
  
    ```bash
    $ vi /etc/hosts
    # 在文件中添加如下行
    > 192.168.38.129 Master
    > 192.168.38.130 Slave1
    > 192.168.38.131 Slave2
    $ vi /etc/hostname
    # 在文件中修改，对两台 Slave 从机做同样操作
    > Master
  ```
  
- 安装 Jdk-1.8 和 Hadoop 
  
    > Oracle 账号密码来自[这篇文章](https://blog.csdn.net/u010590120/article/details/94736800?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.edu_weight&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.edu_weight)，由于直接下载会存在无法解压的问题，所以在 Windows 本地下载再通过 scp 上传到虚拟机 
    >
  > <del>(傻逼 Oracle) </del>
  
    ```powershell
    PS E:\MSN\Downloads> scp .\jdk-8u251-linux-x64.tar.gz root@192.168.38.129:/root/jdk-8u251-linux-x64.tar.gz
    root@192.168.38.129's password:
    jdk-8u251-linux-x64.tar.gz                                                            100%  186MB 114.9MB/s   00:01
    ```
  
    ```bash
    $ tar -zxvf jdk-8u251-linux-x64.tar.gz -C /usr/local/java
    $ vim /etc/profile
    # 在文件中添加如下行
    > export JAVA_HOME="/usr/local/java/jdk1.8.0_251"
    > export JRE_HOME="$JAVA_HOME/jre"
    > export PATH="$PATH:$JAVA_HOME/bin:$JRE_HOME/bin"
    > export CLASSPATH=".:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib"
    $ source /etc/profile
    $ wget https://mirrors.aliyun.com/apache/hadoop/core/hadoop-3.2.1/hadoop-3.2.1.tar.gz
    $ tar -zxvf hadoop-3.2.1.tar.gz -C /usr/local/hadoop
    ```
  
- 配置密钥
  
    ```bash
  $ ssh-keygen -t rsa -P ''
    $ cd /root/.ssh
    [.ssh] $ touch /root/.ssh/authorized_keys
    [.ssh] $ vim authorized_keys
    # 将三台虚拟机的 id_rsa.pub 全部保存至 authorized_keys 中，之后用 ssh 测试链接
    [.ssh] $ ssh Master # 或 Slave1 Slave2
  ```
  
- 配置 Hadoop

  ```bash
  $ mkdir /root/hadoop
    $ mkdir /root/hadoop/tmp
    $ mkdir /root/hadoop/var
    $ mkdir /root/hadoop/dfs
    $ mkdir /root/hadoop/dfs/name
    $ mkdir /root/hadoop/dfs/data
    # 修改 etc/hadoop 下的配置文件
    $ cd /usr/local/hadoop/hadoop-3.2.1/etc/hadoop
    [hadoop] $ vim hadoop-env.sh
    # 添加 JAVA_HOME
    > export JAVA_HOME="/usr/local/java/jdk1.8.0_251"
    [hadoop] $ vim core-site.xml
    # 添加如下配置
    >   <property>
    >        <name>hadoop.tmp.dir</name>
    >        <value>/root/hadoop/tmp</value>
    >        <description>Abase for other temporary directories.</description>
    >   </property>
    >   <property>
    >        <name>fs.default.name</name>
    >        <value>hdfs://Master:9000</value>
    >   </property>
    [hadoop] $ vim hdfs-site.xml
    # 添加如下配置
    >   <property>
    >       <name>dfs.name.dir</name>
    >       <value>/root/hadoop/dfs/name</value>
    >   <description>Path on the local filesystem where theNameNode stores the namespace and transactions logs persistently.
    </description>
    >   </property>
    >   <property>
    >       <name>dfs.data.dir</name>
    >       <value>/root/hadoop/dfs/data</value>
    >   <description>Comma separated list of paths on the localfilesystem of a DataNode where it should store its blocks.</description>
    >   </property>
    >   <property>
    >       <name>dfs.replication</name>
    >       <value>2</value>
    >   </property>
    >   <property>
    >       <name>dfs.permissions</name>
    >       <value>true</value>
    >       <description>need not permissions</description>
    >   </property>
    [hadoop] $ vim mapred-site.xml
    # 添加如下配置
    >   <property>
    >   <name>mapred.job.tracker</name>
    >       <value>Master:49001</value>
    >   </property>
    >   <property>
    >       <name>mapred.local.dir</name>
    >       <value>/root/hadoop/var</value>
    >   </property>
    >   <property>
    >       <name>mapreduce.framework.name</name>
    >       <value>yarn</value>
    >   </property>
    [hadoop] $ vim yarn-site.xml
    # 添加如下配置
    >   <property>
    >        <name>yarn.resourcemanager.hostname</name>
    >        <value>Master</value>
    >   </property>
    >   <property>
    >        <description>The address of the applications manager interface in the RM.</description>
    >        <name>yarn.resourcemanager.address</name>
    >        <value>${yarn.resourcemanager.hostname}:8032</value>
    >   </property>
    >   <property>
    >        <description>The address of the scheduler interface.</description>
    >        <name>yarn.resourcemanager.scheduler.address</name>
    >        <value>${yarn.resourcemanager.hostname}:8030</value>
    >   </property>
    >   <property>
    >        <description>The http address of the RM web application.</description>
    >        <name>yarn.resourcemanager.webapp.address</name>
    >        <value>${yarn.resourcemanager.hostname}:8088</value>
    >   </property>
    >   <property>
    >        <description>The https adddress of the RM web application.</description>
    >        <name>yarn.resourcemanager.webapp.https.address</name>
    >        <value>${yarn.resourcemanager.hostname}:8090</value>
    >   </property>
    >   <property>
    >        <name>yarn.resourcemanager.resource-tracker.address</name>
    >        <value>${yarn.resourcemanager.hostname}:8031</value>
    >   </property>
    >   <property>
    >        <description>The address of the RM admin interface.</description>
    >        <name>yarn.resourcemanager.admin.address</name>
    >        <value>${yarn.resourcemanager.hostname}:8033</value>
    >   </property>
    >   <property>
    >        <name>yarn.nodemanager.aux-services</name>
    >        <value>mapreduce_shuffle</value>
    >   </property>
    >   <property>
    >        <name>yarn.scheduler.maximum-allocation-mb</name>
    >        <value>8192</value>
    >        <discription>每个节点可用内存,单位MB,默认8192MB</discription>
    >   </property>
    >   <property>
    >        <name>yarn.nodemanager.vmem-pmem-ratio</name>
    >        <value>2.1</value>
    >   </property>
    >   <property>
    >        <name>yarn.nodemanager.resource.memory-mb</name>
    >        <value>4096</value>
    >   </property>
    >   <property>
    >        <name>yarn.nodemanager.vmem-check-enabled</name>
    >        <value>false</value>
    >   </property>
    [hadoop] $ cd /usr/local/hadoop/hadoop-3.2.1/sbin
    [sbin] $ vim start-dfs.sh
    # 添加以下行
    > HDFS_DATANODE_USER=root
    > HADOOP_SECURE_DN_USER=hdfs
    > HDFS_NAMENODE_USER=root
    > HDFS_SECONDARYNAMENODE_USER=root
    [sbin] $ vim stop-dfs.sh
    # 添加以下行
    > HDFS_DATANODE_USER=root
    > HADOOP_SECURE_DN_USER=hdfs
    > HDFS_NAMENODE_USER=root
    > HDFS_SECONDARYNAMENODE_USER=root
    [sbin] $ vim start-yarn.sh
    # 添加以下行
    > YARN_RESOURCEMANAGER_USER=root
    > HADOOP_SECURE_DN_USER=yarn
    > YARN_NODEMANAGER_USER=root
    [sbin] $ vim stop-yarn.sh
    # 添加以下行
    > YARN_RESOURCEMANAGER_USER=root
    > HADOOP_SECURE_DN_USER=yarn
    > YARN_NODEMANAGER_USER=root
    [sbin] $ vim /etc/selinux/config
    # 修改以下行
    > SELINUX=disabled
    [sbin] $ systemctl disable firewalld.service
    [sbin] $ cd /usr/local/hadoop/hadoop-3.2.1/etc/hadoop
    [hadoop] $ touch Master # 从机的话文件名是从机名字
    [hadoop] $ vim Master
    # 主机写入主机名，从机写入主机名和自己的名字
    > Master
    [hadoop] $ vim workers
    > Master
    > Slave1
    > Slave2
  ```

  >   如果运行示例时报错
  >
  >   ```xml
  >   Please check whether your etc/hadoop/mapred-site.xml contains the below configuration:
  >   <property>
  >     <name>yarn.app.mapreduce.am.env</name>
  >     <value>HADOOP_MAPRED_HOME=${full path of your hadoop distribution directory}</value>
  >   </property>
  >   <property>
  >     <name>mapreduce.map.env</name>
  >     <value>HADOOP_MAPRED_HOME=${full path of your hadoop distribution directory}</value>
  >   </property>
  >   <property>
  >     <name>mapreduce.reduce.env</name>
  >     <value>HADOOP_MAPRED_HOME=${full path of your hadoop distribution directory}</value>
  >   </property>
  >   ```
  >
  >   则把 `${full path of your hadoop distribution directory}` 改成安装 hadoop 的位置
  >
  >   ```xml
  >   <value>HADOOP_MAPRED_HOME=/usr/local/hadoop/hadoop-3.2.1</value>
  >   ```

  - 启动 Hadoop

    ```bash
  $ cd /usr/local/hadoop/hadoop-3.2.1/bin
    [bin] $ ./hadoop namenode -format
    [bin] $ cd /usr/local/hadoop/hadoop-3.2.1/sbin
    [sbin] $ ./start-all.sh
    # $ ./stop-all.sh
    ```
    

- Hadoop 运行模式

  - 本地运行模式
    
    - 参考官方文档，注意每次需要删除 output 文件夹
    
  - 伪分布式运行模式
    - 配置同完全分布式，但是只有 Master 机
    - 上传文件 `bin/hdfs dfs -put`
    - 格式化 NameNode 注意事项
      1. 在 `dfs/name/current/VERSION` 文件里有 `clusterID`
      2. 在 `dfs/data/current/VERSION` 里一样有 `clusterID` 并且跟上面的一样
      3. 重新格式化时，两边的 `clusterID` 不一样，无法通讯， DataNode 和 NameNode 无法通信
      4. 重新格式化之前，需要删除 DataNode 里面的信息
    - 启动 YARN 并运行 MapReduce 程序
      1. 在 `yarn-env.sh` 里配置 `JAVA_HOME`
      2. 在 `mapred-env.sh` 里配置 `JAVA_HOME`
    
  - 完全分布式运行模式

    - rsync 更新差异文件，主要用于备份和镜像
    - xsync 集群分发脚本，循环复制文件到所有节点的相同目录下

    ```bash
    #!/bin/bash
    
    pcount=$#
    if((pcount==0)); then
    echo no args;
    exit;
    fi
    
    p1=$1
    fname=`basename $p1`
    echo fname=$fname
    
    pdir=`cd -P $(dirname $p1); pwd`
    echo pdir=$pdir
    
    user=`whoami`
    
    for((snum=1; snum<3; snum++)); do
    	echo -----Slave$snum-----
    	srync -rvl $pdir/$fname $user@Slave$snum:$pdir
    done
    ```

    - 可以把 NameNode，ResourceManager 和 SecondaryNameNode 放在不同节点，配置 .xml 文件即可
    - 上传数据

    ```bash
    $ bin/hdfs $target_dir dsf -put $source_dir
    ```

    - 上传的数据储存在 `data/tmp/dfs/data/current/$ClusterID/current/finalized/subdir`
    - YARN 应该在 ResourceManager 所在的机器上启动




## 3. HDFS

### 3.1 HDFS 概述

- HDFS 是分布式文件管理系统，通过**目录树**来定位文件
- HDFS 适合一次写入多次读出的场景，且不支持文件的修改，适合用来数据分析，并不适合网盘应用
- 优点
  - 高容错性：数据自动保存多个副本，某一个副本丢失后可以自动恢复
  - 适合处理大数据：处理规模最高可达 `PB` 级别和百万以上规模的文件数量
  - 可构建在廉价机器上
- 缺点
  - 不适合低延时的数据访问
  - 无法高效的对大量小文件进行存储：储存大量小文件会占用 NameNode 大量的内存储存文件目录和块信息
  - 不支持并发写入和文件随机修改：一个文件只能由一个写，且只支持数据的追加
- HDFS 组成架构
  - NameNode：储存了 Metadata。管理 HDFS 的名称空间和 Block 映射信息，配置副本策略和处理客户端读写请求
  - DataNode：储存 Blocks，执行 Blocks 的读写操作
  - Client：客户端。进行文件切分上传，与 NameNode 交互获取文件位置信息，与 DataNode 交互读写数据，管理和访问 HDFS
  - SecondaryNameNode：辅助 NameNode工作，如定期合并 Fsimage 和 Edits 并推送给 NameNode；紧急情况下辅助回复 NameNode。不是 NameNode 的热备份
- HDFS 的 Blocks 大小
  - 通过参数 `dfs.blocksize` 确定， 2.x 版本默认为 128 MB，1.x 版本默认是 64 MB
  - 寻址时间为传输时间的 1% 时为最佳状态
  - Blocks 太小，会增加寻址时间；Blocks 太大，从磁盘传输数据的时间会明显大于定位这个块开始位置所需的时间（即寻址时间），导致程序处理数据时过慢

### 3.2 HDFS Shell 命令

> 基本语法：`bin/hadoop fs command` 或者 `bin/hdfs dfs command`。`dfs` 是 `fs` 的实现类。

- 输入 `dfs` 或 `fs` 会显示命令参数列表
- 常用命令
  - `-help` 输出命令参数
  - `-ls` 查看目录信息
  - `-mkdir` 创建目录
  - `-moveFromLocal` 从本地上传到 HDFS（剪切）
  -  `-appendToFile` 追加内容到文件末尾
  - `-cat` 查看文件内容
  - `-chgrp` `-chown` `-chmod` 修改文件所属权限
  - `-copyFromLocal` 从本地复制到 HDFS
  - `-copyToLocal` 从 HDFS 复制到本地
  - `-cp` 从 HDFS 路径复制到另一个 HDFS 路径
  -  `-mv` 从 HDFS 路径移动到另一个 HDFS 路径
  - `-get ` 等于 `-copyToLocal`
  - `-put` 等于 `-copyFromLocal`
  - `-getmerge` 合并下载多个文件
  - `-tail` 显示文件末尾
  - `-rm` 删除文件或文件夹
  - `-rmdir` 删除空目录
  - `-du` 统计文件夹大小信息
  - `-setrep` 设置 HDFS 中文件的副本数量

### 3.3 HDFS API 操作

- 在 Java 代码中，通过 FileSystem 提供的与 Shell 命令同名的方法进行操作

- 参数优先级顺序：客户端代码中的设置 > ClassPath 下用户自定义配置文件 > 服务器默认配置
- 如果 API 没有提供需要的操作，需要通过 IO 流进行操作

### 3.4 HDFS 数据流

- 数据写入流程
  1. 客户端向 NameNode 请求上传文件
  2. NameNode 响应可以上传文件
  3. 请求上传第一个 Block，返回 DataNode
  4. NameNode 返回 DataNode 节点
  5. 客户端请求与 DataNode 建立通道
  6. DataNode 应答成功
  7. 传输数据 Packet
  8. 通知 NameNode 传输完成
- 网络拓扑-节点距离计算
  - 节点距离：两个节点到达最近的共同祖先的距离总和
- 机架感知（副本储存节点选择）
  - 以三个为例。第一个副本位于 Client 所处节点上。如果客户端在集群外，随机选一个
  - 第二个副本和第一个副本位于相同机架上，随机节点
  - 第三个副本位于不同机架随机节点
- 数据读取流程
  - 客户端向 NameNode 请求下载文件
  - NameNode 返回文件的元数据
  - 客户端向 DataNode 请求读取数据
  - DataNode 传输数据
  - 对所有储存了待下载文件块对 DataNode 重复上述两个操作

### 3.5 HDFS 2.X 新特性

- 集群间数据拷贝 `dictcp`
- 小文件存档 HAR 将文件存入 HDFS块，是独立小文件，但是对 NN 而言是一个整体
- 回收站 `fs.trash.interval = 0` 默认关闭，其他值表示设置文件的存活时间，默认检查回收站的间隔时间是 `fs.trash.checkpoint.interval`
- 快照管理，对目录做备份
    - `hdfs dfsadmin -allowSnapshot $dir` 开启指定目录的快照功能
    - `hdfs dfsadmin -disallowSnapshot $dir` 关闭指定目录的快照功能
    - `hdfs dfs -createSnapshot $dir [$name]` 创建指定名称的目录快照
    - `hdfs dfs -renameSnapshot $dir $old_name $new_name `  重命名快照
    - `hdfs lsSnapshottableDir` 列出当前目录所有可快照目录
    - `hdfs snapshotDiff $dir1 $dir2` 比较两个快照目录
    - `hdfs dfs -deleteSnapshot $dir $name` 删除快照



## 4. NameNode 和 SecondaryNameNode

### 4.1 NN 和 2NN 的工作机制

- HDFS 1.0
  - 元数据储存在内存中
  - 磁盘中备份了元数据 FsImage
  - 引入 Edits 文件，只进行追加操作。当元数据更新或添加时记录
  - 2NN 用于定期完成 FsImage 和 Edits 的合并
- NN 的工作机制
  - 加载编辑日志和镜像文件到内存
  - 客户端进行元数据到增删改
  - NN 记录操作日志，再更新滚动日志
  - 进行内存数据的增删改
- 2NN 的工作机制
  - 请求 NN 是否需要 CheckPoint
  - 请求执行 CheckPoint
  - NN 滚动正在写的 Edits
  - 拷贝 FsImage 和 Edits 到 2NN
  - 合并，生成新的 FsImage 并拷贝到 NN

### 4.2 FsImage 和 Edits

- FsImage 和 Edits 在 `/data/tmp/dfs/name/current` 里

- `hdfs oiv` 和 `hdfs oev` 用于查看文件

  ```bash
  $ hdfs oiv/oev -p XML -i $input_file -o $output_file.xml
  ```

### 4.3 CheckPoint 设置

- 在 .xml 中有设置

- 默认 3600s
- 一分钟检查一次操作次数，达到 1000000 次立刻合并

### 4.4 NameNode 故障处理

- 将 2NN 数据拷贝到 NN 储存数据到目录
  - 删除 `name` 文件夹
  - 拷贝 `namesecondary` 文件夹到 `name` 文件夹

- 使用 `-importCheckpoint` 选项启动 NN 守护进程，从而拷贝 2NN 到 NN
  - 第一步与上个方法一样
  - 如果 2NN 不和 NN 在一个主机节点上，拷贝 2NN 储存数据的目录到 NN 储存数据的评级目录并删除 in_use.lock
  - `hdfs namenode -importCheckpoint`

### 4.5 集群安全模式

- NN 启动时首先将 FsImage 载入内存，并执行 Edits 中的各项操作
- 在内存中建立文件系统元数据的映像之后，会创建一个新的 FsImage 文件和一个空的 Edits， 此时 NN 开始监听 DataNode 的请求
- 以上两个过程中， NN 一直运行在安全模式中，即 NN 的文件系统对客户端是只读的
- 系统中数据块的位置以块列表形式储存在 DataNode 中，在系统正常操作期间 NN 会在内存中保存所有块位置的映射信息
- 在安全模式下， 各个 DataNode 会向 NN 发送最新的块列表信息
- 如果满足最小副本条件：在整个文件系统中 99.9% 的块满足最小副本级别 （默认值是 `dfs.relication.min=1`），则 NN 在 30s 后退出安全模式
- 集群处于安全模式下，不能执行重要操作（写操作）
- 安全模式基本命令
    - `hdfs dfsadmin -safemode get` 查看安全模式状态
    - `hdfs dfsadmin -safemode enter` 进入安全模式
    - `hdfs dfsadmin -safemode leave` 离开安全模式
    - `hdfs dfsadmin -safemode wait` 等待安全模式状态



## 5. DataNode

### 5.1 DataNode 工作机制

- DataNode 的 Block 中存放数据、数据长度、校验和、时间戳
- DataNode 工作机制
    - DN 启动后向 NN 注册
    - NN 将 DN 注册成功写入元数据
    - 每个一个周期（一个小时） DN 上报所有块信息
    - 心跳（每三秒一次）返回，结果带有 NN 给该 DN 的命令
    - 超过十分钟没有收到 DN 的心跳包，则认为该节点不可用

### 5.2 数据完整性

- 保证数据完整性的方法
    - 当 DN 读取 Block 时，计算校验和
    - 如果计算后的校验和与 Block 创建时不一样，说明 Block 已经损坏
    - Client 读取其他 DN 上的 Block
    - DN 在其文件创建后周期验证校验和

### 5.3 掉线时限参数设置

- 默认超时时长是 10min + 30s
- 计算公式为 $TimeOut = 2*dfs.namenode.heartbeat.recheckinterval + 10*dfs.heartbeat.interval$
- 配置在 hdfs-site.xml 文件中

### 5.4 添加新DN

- 修改 IP 地址和 Host 文件
- 删除原来 HDFS 文件系统留存的文件 `hadoop-x.x.x/data` 和 `log`
- 执行 `source /etc/profile`

### 5.5 DN 退役旧数据节点

- 不在白名单内的主机节点都会被退出
    - 白名单在 NN 的 `hadoop-x.x.x/etc/hadoop/dfs.hosts`
    - 在 hdfs-site.xml 配置文件中增加 dfs.hosts 属性
    - 配置文件分发，刷新 NN
    - 更新 ResourcesManager
- 黑名单中的节点都会被踢出
    - 黑名单在 `hadoop-x-x-x/etc/hadoop/dfs.hosts.exclude`
    - 在黑名单中添加要退役节点的名称
    - 在 hdfs-site.xml 配置文件中添加 dfs.hosts.exclude 属性
    - 刷新 NN 和 RM
- 黑名单和白名单中不能出现同一个主机名称



## 6. MapReduce

### 6.1 MapReduce 概述

- MapReduce 是分布式运算程序的编程框架
- MapReduce 优点
    - 易于编程，实现一些接口即可完成一个分布式程序
    - 良好的扩展性
    - 高容错性，节点挂掉的时候计算任务可以自动转移
    - 适合 `PB` 级别以上海量数据的离线处理
- MapReduce 缺点
    - 不擅长实时计算
    - 不擅长流式计算
    - 不擅长有向图计算

### 6.2 MapReduce 核心思想

> 以单词统计为例

- MapReduce 程序分为 Map 阶段和 Reduce 阶段
- Map 阶段分发数据，启动 MapTask （并发）
    - 读取数据，并按行处理
    - 按空格切分行内单词
    - KV 键值对按单词首字母，分为两个分区写入磁盘
- Reduce 阶段输入数据来源上个阶段的 MapTask输出
    - 启动 ReduceTask 从 MapTask 拉取相应数据
    - ReduceTask 输出各自数据
- MapReduce 模型只能包含一个 Map 阶段和一个 Reduce 阶段

### 6.3 MapReduce 进程

- 一个完整的 MapReduce 程序在分布式运行时有三类实例进程
    - MrAppMaster 负责整个程序的过程调度及状态协调
    - MapTask 负责 Map 阶段的整个数据处理流程
    - ReduceTask 负责 Reduce 阶段的整个数据处理流程

### 6.4 MapReduce 编程

- 常见类型的封装

    | Java 类型 | Hadoop Writable 类型 |
    | --------- | -------------------- |
    | boolean   | BooleanWritable      |
    | byte      | ByteWritable         |
    | int       | IntWritable          |
    | float     | FloatWritable        |
    | double    | DoubleWritable       |
    | String    | Text                 |
    | map       | MapWritable          |
    | array     | ArrayWritable        |

- Mapper 阶段
    - 用户自定义的 Mapper 要继承父类
    - Mapper 的输入数据必须是键值对，但是键值的类型可以自己定义
    - Mapper 中的业务逻辑写在 `map()` 方法中
    - Mapper 的输出数据必须是键值对，但是键值的类型可以自己定义
    - `map()` 方法（ MapTask 进程）对每个键值对调用一次
- Reduce 阶段
    - 用户自定义的 Reducer 要继承父类
    - Reducer 的输入数据对应 Mapper 的输出数据类型
    - Reducer 的业务逻辑写在 `reduce()` 方法中
    - ReduceTask 进程对每一组**键相同**的键值对调用一次 `reduce()` 方法
- Driver 阶段
  
    - 提交程序到 YARN 集群，提交的是封装了 MapReduce 程序相关运行参数的 `job` 对象
- Mapper `run()` 方法

```Java
public void run(Context context) throws IOExceptation, InterruptedExceptation
{
    setup(context);
    try
    {
        while(context.nextKeyValue())
        {
            map(context.getCurrentKey(), context.getCurrentValue(), context);
        }
    }
    finally
    {
        cleanup(context);
    }
}
```

- Reducer `run()` 方法

```Java
public void run(Context context) throws IOExceptation, InterruptedExceptation
{
    setup(context);
    try
    {
        while(context.nextKey())
        {
            reduce(context.getCurrentKey, context.getValues(), context);
            Iterator<VALUEIN> iter = context.getValues().iterator();
            if(iter instanceof ReduceContext.ValueIterator)
            {
                ((ReduceContext.ValueIterator<VALUEIN>)iter).resetBackupStore();
            }
        }
    }
    finally
    {
        cleanup(context);
    }
}
```

- Driver 类内容
    - 获取 Job 对象
    - 设置 .jar 储存位置
    - 关联 Map 类和 Reduce 类
    - 设置 Mapper 阶段输出数据的 Key 和 Value 类型
    - 设置最终数据输出的 Key 和 Value 类型
    - 设置输入路径和输出路径
    - 提交 Job



## 7. Hadoop 序列化

### 7.1 序列化概述

- 序列化就是把内存中的对象转换成字节序列或其他数据传输协议，以便于储存到磁盘和网络传输
- 反序列化就是将收到的字节序列或其他数据传输协议转换成内存中的对象
- Java 的序列化自带很多额外信息，不便在网络中传输，因此 Hadoop开发了自己的序列化机制 Writable
- Hadoop 序列化特点
    - 紧凑：高效使用储存空间
    - 快速：数据读写额外开销小
    - 可扩展：随着通信协议的升级而升级
    - 互操作：支持多语言的交互

### 7.2 Hadoop 序列化实现

- 序列化对象必须实现 Writable 接口
- 反序列化时，需要反射调用空参构造函数，所以必须有空参构造
- 重写序列化方法 `write()` 和反序列化方法 `readFields()`

> 以下是一个示例

```java
@Override
public void write(DataOutput out) throws IOExceptation
{
    out.writeLong(upFlow);
    out.writeLong(downFlow);
    out.writeLong(sumFlow);
}

@Override
public void readFields(DataInput in) throws IOExceptation
{
    upFlow = in.readLong();
    downFlow = in.readLong();
    sumFlow = in.readLong();
}
```

- 反序列化的顺序必须和序列化的顺序**完全一致**
- 如果要将结果显示在文件中，需要重写 `toString()` 方法
- 如果自定义反序列化的对象要在 Key 中进行传输，则还需要实现 Comparable 接口





- 下一节：[MapReduce 框架原理](./HadoopNote_02.md)





























