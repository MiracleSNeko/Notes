# Hadoop 笔记

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

  - 完成上述操作后，使用 VMWare 的克隆虚拟机功能，克隆 Master 为 Slave1 和 Slave2 两个从机，并且修改各自的 ip 和 hostname，互相 ping 通既为完成

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
    - 