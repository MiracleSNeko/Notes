# Hadoop 框架原理

> 尚硅谷Hadoop教程：BV1cW411r7c5

## 1. 数据输入

### 1.1 切片与 MapTask 并行度决定机制

- 数据块：Bolck 是 HDFS 物理上将数据分块
- 数据切片：指逻辑上对输入进行分片，并不会在磁盘上将其切分成片进行储存
- MapTask 并行度决定机制
    - 一个 Job 的Map 阶段并行度由客户端在提交 Job 时的切片数决定
    - 每一个切片分配一个 MapTask 并行处理
    - 默认情况下，切片大小等于 Block 大小
    - 切片时不考虑数据集整体，而是逐一针对每一个文件单独切片

### 1.2 Job 提交流程

- 建立连接，创建提交 Job 的代理
    - 判断是本地 YARN 还是远程
- 提交 Job
    - 创建给集群提交数据的 Stag 路径
    - 获取 jobid ，并创建 Job 路径
    - 拷贝 .jar 包到集群
    - 计算切片，生成切片规划文件
    - 向 Stag 路径写入 .xml 配置文件
    - 提交 Job ，返回提交状态

### 1.3 FileInputFormat 切片

- 切片流程

    - 程序首先找到数据储存目录

    - 遍历处理规划切片目录下的每一个文件

        - 获取文件大小
        - 计算切片大小（默认切片大小为 Block 大小）

        ```java
        computeSliteSize(Math.max(minSize, Math.min(maxSize, blocksize)))
        ```

        > `mapreduce.input.fileinputformat.split.minsize` 默认为 1，`mapreduce.input.fileinputformat.split.maxsize` 默认为 `Long.MAXValue`

        - 开始形成切片。每次划分切片时都要判断剩下的部分是否大于块的 1.1 倍，不大于就划分一块切片
        - 将切片信息写入切片规划文件
        - 整个切片的核心过程在 `getSplit()` 方法中完成
        - InputSplit 只记录了切片的元数据信息，如起始位置、长度及所在节点列表等

    - 提交切片规划文件到 YARN ，YARN 上的 MrAppMaster 根据切片规划文件计算开启 MapTask 个数

- 切片机制

    - 简单地按照文件的内容长度进行切片
    - 切片大小默认为 Block 大小
    - 切片时不考虑数据集整体，而是逐个对每一个文件进行切片

### 1.4 CombineTextInputFormat 切片

- 适用于小文件过多的场景， CombineTextInputFormat 将多个小文件从逻辑上规划到一个切片中
- 切片机制
    - 虚拟储存过程
        - 小于 `MaxInputSplitSize` 划分一块
        - 介于一倍到二倍之间，均分为两块
    - 切片过程
        - 判断虚拟储存文件是否大于 `MaxInputSplitSize` ，如果大于等于则单独形成一个切片
        - 如果不大于，则和下一个虚拟储存文件合并，共同形成一个切片

### 1.5 KeyValueTextInputFormat

- 每一行均为一条记录，被分割为 Key 和 Value

- 设定分隔符。默认分隔符为 `\t`

    ```java
    conf.set(KeyValueLineRecordReader.KEY_VALUE_SEPERATOR, "\t");
    ```

### 1.6 NLineInputFormat

- 如果使用 NLineInputFormat ，代表每个 MapTask 处理的 InputSplit 不再按 Block 块区划分，而是按照 NLineInputFormat 指定的行数 N 划分，即 `输入文件的总行数/N = 切片数` 
- 如果不整除，向上取整

### 1.7 自定义 InputFormat

- 自定义 InputFormat 步骤
    - 自定义一个类继承 FileInputFormat
        - 重写 `isSplitable()` 方法，返回 `false`  （即不可分割）
        - 重写 `createRecordReader()` 方法，创建自定义的 RecordReader 对象并初始化
    - 改写 RecordReader ，实现一次读取一个完整文件封装为键值对
        - 采用 IO 流一次读取一个文件输出到 value 中，因为设置了不可切片，最终将所有文件都封装到了 value 中
        - 获取文件路径信息和名称，并设置 key
    - 设置 Driver
        - 设置输入的 inputFormat
        - 设置输出的 outputFormat
- InputFormat 常见接口类
    - TextInputFormat
        - 默认的 InputFormat ，每条记录是一行输入
        - 键是 LongWritable 类型，储存该行在整个文件中的起始字节偏移量
        - 值是该行的内容， Text 类型，不包括任何终止符
    - KeyValueTextInputFormat
        - 按行切片，每条记录是一行输入
        - 键值按照给定的分隔符分割
    - NLineInputFormat
        - 按 N 行切片
        - 键值同 TextInputFormat
    - CombineTextInputFormat
        - 按  `MaxInputSplitSize` 切片
        - 键值同 TextInputFormat



## 2. MapReduce 工作流程

### 2.1 Map 阶段

- 输入待处理文本
- 客户端调用 `submit()` 前，获取待处理数据的信息，然后根据参数配置形成一个任务分配的规划
- 提交切片信息
- 计算 MapTask 的数量并启动对应数量的 MapTask
- InputFormat 处理输入
- 进行 `map()`  运算
- 像环形缓冲区写入键值对数据
    - 环形缓冲区从中间开始，一侧写入键值对元数据和索引，一侧写入键值对数据
    - 环形缓冲区填满 80% 后，缓冲区内容写入磁盘，指针反向
- 分区，并在每个分区内部排序（按 Key 字典序），实现手段是快排
- 排序后内容溢出到文件
- 对分区且内部有序的文件进行归并排序

### 2.2 Reduce 阶段

- 所有 MapTask 任务完成后，MrappMaster 启动与分区数量对应的 ReduceTask ，并告知 ReduceTask 处理的数据范围（数据分区）
- 下载分区到 ReduceTask 本地磁盘，并对所有分区进行归并排序，合并分区
- 按 Key 分组，一次读取一组进入 Reducer
- OutputFormat 处理输出



## 3. Shuffle 机制

> Map 方法之后，Reduce 方法之前的数据处理过程成为 Shuffle

### 3.1 分区

- 系统默认分区方式 HashPartitioner

```java
public class HashPartitioner<K, V> extends Partitioner<K, V>
{
    public int getPartition(K key, V value, int numReduceTasks)
    {
        return(key.hashCode() & Integer.MAX_VALUE) % numReduceTasks;
    }
}
```

- 默认分区时，用户无法控制哪个 Key 储存到哪个分区

### 3.2 自定义 Partitioner

- 自定义类继承 Partitioner ，重写 `getPartitioner()` 方法
    - 返回一个整数值 partition
    - partition 从0开始
- 在 Job 设定自定义 Partitioner

```java
job.setPartitionerClass(CustomPartitioner.class)
```

- 自定义 Partitioner 后，要根据自定义 Partitioner 的逻辑设置相应数量的 ReduceTask

```java
job.setNumReduceTasks(5);
```

- 分区结果
    - 如果 ReduceTask 数量多于 getPartition 的结果，会产生空的输出文件
    - 如果 ReduceTask 数量大于 1 且少于 getPartition 的结果，有一部分的分区数据会无处安放，抛出Exception
    - 如果只有一个 ReduceTask ，则不管 MapTask 输出多少分区文件，最终结果都交给一个结果文件 `part-r-00000`

### 3.3 WritableComparable 排序

- MapTask 和 ReduceTask 均会对数据按照 key 进行排序
- 排序操作是 Hadoop 的默认行为，任何应用程序中的数据均会被排序，而不管逻辑上是否需要
- 默认按照字典序进行快速排序
- 排序分类
    - 部分排序：MapReduce 根据输入记录的键对数据集排序，保证输出的每个文件内部有序
    - 全排序：最终输出的结果只有一个文件，并且文件内部有序，实现方式是只设置一个 ReduceTask
    - 辅助排序：在 Reduce 阶段对键进行分组
    - 二次排序：自定义排序中的判断条件为两个
- 自定义排序 WritableComparable
    - 实现  WritableComparable 接口，重写 `compareTo()` 
    - 返回值是 `-1 (LT) | 0 (EQ) | 1 (GT)`





