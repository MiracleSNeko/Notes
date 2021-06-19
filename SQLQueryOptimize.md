# MySQL查询优化


## 1. 概述

### 1.1 基本概述

- 查询执行过程

    - 语法分析

    - 生成逻辑查询执行计划

    - 生成物理查询执行计划

    - 执行查询执行计划

- 查询优化器通过 `JOIN` 对象的方法完成优化工作

    - `JOIN.prepare()` 子查询冗余子句消除，`IN` 类型子查询优化，将 `ALL/ANT` 转化为 `MIN/MAX` 等对简单子查询的优化

    - `JOIN.optimize()` 子查询上拉，外连接优化为内连接，消除嵌套链接，`WHERE/JOIN/ON/HAVING` 等子句的化简，优化没有 `GROUP BY` 子句情况下的聚合函数，剪裁分区，确定多表的连接路径，优化查询谓词，优化 `DISTINCT` ，创建临时表储存临时结果优化分组排序等操作

### 1.2 相关数据结构

#### 1.2.1 查询树

- 语法分析器的结果，用 `st_select_lex` 类表示

#### 1.2.2 基本对象

- 关系 `TABLE_LIST` 储存查询优化阶段需要的信息，也有其他阶段的信息，基本上，只要和表相关的信息都在该结构体中

- 索引 `Key` 定义了索引的类型和索引的关键信息。其中元数据 `KEY_CREATE_INFO` 定义了索引算法、名称、注释等

    ```c++
    enum Ha_key_alg {
        HA_KEY_ALG_UNDEF = 0,   // Not specified
        HA_KEY_ALG_BTREE = 1,   // BTree，默认使用
        HA_KEY_ALG_RTREE = 2,   // RTree，用于空间搜索
        HA_KEY_ALG_HASH = 3,    // Hash，用于堆表搜索
        HA_KEY_ALG_FULLTEXT = 4 // 全文索引，用于 MyISAM 储存索引的表
    }
    ```
- 连接表 `st_join_table` 是介于关系和连接类之间的过渡对象，存放了关系的一些相关信息，也存放了连接操作需要的一些信息，所以称为连接表

- 连接类 `JOIN` 对应的查询语句的连接关系内容，是优化和执行的基本单位，也是优化结果的储存对象

- 约束条件 `ITEM` 指 `WHERE/JOIN/ON/HAVING` 子句中的谓词表达式，其分为两种：第一是限制条件，用来过滤单表的元组；第二是连接条件，满足连接条件的元组才会连接，连接条件表达式一般包括两个或以上关系的变量

- 位置 `st_position` 被连接的表的位置，储存了被访问的表、使用的访问方法、半连接策略的选择、半连接优化状态等

- 代价估算类 `Cost_estimate` 估算 I/O 花费、CPU 花费、远程操作花费和内存操作花费


## 2. MySQL 查询优化（功能角度）

### 2.1 逻辑查询优化

#### 2.1.1 视图重写
