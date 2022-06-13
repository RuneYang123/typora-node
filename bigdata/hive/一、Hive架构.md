### 一、Hive架构

![image-20210723112738138](C:\Users\Rune\AppData\Roaming\Typora\typora-user-images\image-20210723112738138.png)

（1）Hive 处理的数据存储在 HDFS

（2）Hive 分析数据底层的实现是 MapReduce

（3）执行程序运行在 Yarn 上

（4）元数据(映射关系)包括：表名、表所属的数据库（默认是 default）、表的拥有者、列/分区字段、表的类型（是否是外部表）、表的数据所在目录等；默认存储在自带的 derby 数据库中，推荐使用 MySQL 存储 Metastore。

### 二、HQL DDL数据定义

#### 创建库

create datebase [if not exists](检查是否库以存在，存在也不报错) test_db [location '/user/test_db.db'](指定库再hdfs储存位置)

#### 查看库

show  databases;查看所有数据库

show database like 'hive*';条件查询

desc database test_db;数据库详情

#### 修改库

alter database db_hive 

set dbproperties('createtime'='20170830');

#### 删除库

drop database db_hive2;

#### 创建表

CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name    EXTERNAL 外部表

[(col_name data_type [COMMENT col_comment], ...)] 为表和列添加注释

[COMMENT table_comment]

[PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]分区表

[CLUSTERED BY (col_name, col_name, ...)分桶表

[SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS]按照字段分桶

[ROW FORMAT row_format]  定义行格式

[STORED AS file_format]  指定文件

[LOCATION hdfs_path]  表的存储位置

[TBLPROPERTIES (property_name=property_value, ...)] 属性（作用不大）

[AS select_statement]  

内部表：数据存储在hdfs/warehouse    create [inner] table       删除：元数据和表数据全部删除
外部表：数据存储在Hbase/mysql/..     create external table      删除：只删除元数据，表数据不删

local inpath      本地路径
inpath           hdfs 路径

like 只复制表结构，不复制表数据
as    复制表结构和表数据 