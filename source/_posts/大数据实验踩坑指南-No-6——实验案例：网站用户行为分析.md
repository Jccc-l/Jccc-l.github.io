---
title: 大数据实验踩坑指南_No.6——实验案例：网站用户行为分析
mathjax: false
katex: false
date: 2024-06-16 16:31:29
description: 林子雨老师的网站用户行为分析案例的复现记录
categories:
- Big Data
tags:
- Big Data
- Linux
- Hadoop
- Hive
- HBase
- R
- Java
- ZooKeeper
---

# 大数据实验踩坑指南_No.6——实验案例：网站用户行为分析

## 实验环境搭建

在这个实验中，需要用到下列软件包，以下是我的软件版本，点击即可跳转到官网或国内镜像站的下载链接：
- [Linux](https://mirrors.tuna.tsinghua.edu.cn/ubuntu-releases/22.04/ubuntu-22.04.4-desktop-amd64.iso): Ubuntu 22.04.4 LTS
- [Hadoop](https://dlcdn.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz): 3.3.6
- [Hive](https://dlcdn.apache.org/hive/hive-3.1.3/apache-hive-3.1.3-bin.tar.gz): 3.1.3
- [ZooKeeper](https://dlcdn.apache.org/hive/hive-3.1.3/apache-hive-3.1.3-bin.tar.gz): 3.9.2
- [R](https://cran.r-project.org/index.html)有两种安装方式，效果是一样的，二选一即可
	- [Ubuntu Packages For R - Brief Instructions](https://cran.r-project.org/bin/linux/ubuntu/)
	- [Ubuntu Packages For R - Full Instructions](https://cran.r-project.org/bin/linux/ubuntu/fullREADME.html)
- [HBase](https://www.apache.org/dyn/closer.lua/hbase/2.5.8/hbase-2.5.8-hadoop3-bin.tar.gz): 2.5.8-hadoop3-bin
- [JetBrains IntelliJ IDEA](https://www.jetbrains.com/help/idea/installation-guide.html)
	- 使用Linux版本的Standalone installation单独下载IDEA

请根据前面的教程安装配置好这些软件
- {% post_link 大数据实验踩坑指南-No-1——Hadoop的安装配置 %}
- {% post_link 大数据实验踩坑指南-No-2——Hive的安装配置 %}
- {% post_link 大数据实验踩坑指南-No-3——ZooKeeper的安装配置 %}
- {% post_link 大数据实验踩坑指南-No-4——HBase的安装配置 %}
- {% post_link 大数据实验踩坑指南-No-5——R语言和软件包的安装 %}

[林子雨老师的教程](https://dblab.xmu.edu.cn/blog/4189/)

IDE可以自选IntelliJ IDEA[^1]或Eclipse[^2]

[^1]: [Install IntelliJ IDEA](https://www.jetbrains.com/help/idea/installation-guide.html#-aa9dmj_178)

[^2]: [Eclipse 阿里镜像站](https://mirrors.aliyun.com/eclipse/eclipse/downloads/drops4/R-4.23-202203080310/eclipse-SDK-4.23-linux-gtk-x86_64.tar.gz)


## 本地数据集上传到数据仓库Hive

### 实验数据集的下载

- 本案例采用的数据集为`user.zip`，包含了一个大规模数据集`raw_user.csv`（包含2000万条记录），和一个小数据集`small_user.csv`（只包含30万条记录）。小数据集`small_user.csv`是从大规模数据集`raw_user.csv`中抽取的一小部分数据。之所以抽取出一少部分记录单独构成一个小数据集，是因为在第一遍跑通整个实验流程时，会遇到各种错误、各种问题，先用小数据集测试，可以大量节约程序运行时间。等到第一次完整实验流程都顺利跑通以后，可以最后用大规模数据集进行最后的测试。
- 把数据集`user.zip`文件下载到Linux系统的`/home/hadoop/Downloads/`目录下面

下载案例的数据集`user.zip`到目录`/home/hadoop/Downloads/`内（系统语言如果是中文，则是`/home/hadoop/下载）

创建一个存储数据集的目录，将`user.zip`进行解压缩

```sh
sudo mkdir -p /usr/local/bigdatacase/dataset
sudo chown -R hadoop:hadoop /usr/local/bigdatacase # 修改bigdatacase目录权限
unzip /home/hadoop/Downloads/user.zip -d /usr/local/bigdatacase/dataset # 解压缩数据文件user.zip
cd /usr/local/bigdatacase/dataset
```

现在可以看到`dataset`目录下有两个文件：`raw_user.csv`和`small_user.csv`。执行以下命令查看前5条数据

```sh
head -5 raw_user.csv
```

### 数据集的预处理

删除文件的第一行记录（字段名称）

```sh
sed -i '1d' raw_user.csv
sed -i '1d' small_user.csv
head -5 raw_user.csv
head -5 small_user.csv
```

查看到已经看不到字段名成这一行

对字段进行预处理

下面要建一个脚本文件`pre_deal.sh`，请把这个脚本文件放在`dataset`目录下，和数据集`raw_user.csv`放在同一个目录下：

```sh /usr/local/bigdatacase/dataset/pre_deal.sh
#!/bin/bash
#下面设置输入文件，把用户执行pre_deal.sh命令时提供的第一个参数作为输入文件名称
infile=$1
#下面设置输出文件，把用户执行pre_deal.sh命令时提供的第二个参数作为输出文件名称
outfile=$2
#注意，最后的$infile> $outfile必须跟在}’这两个字符的后面
awk -F "," 'BEGIN{
srand();
        id=0;
        Province[0]="山东";Province[1]="山西";Province[2]="河南";Province[3]="河北";Province[4]="陕西";Province[5]="内蒙古";Province[6]="上海市";
        Province[7]="北京市";Province[8]="重庆市";Province[9]="天津市";Province[10]="福建";Province[11]="广东";Province[12]="广西";Province[13]="云南"; 
        Province[14]="浙江";Province[15]="贵州";Province[16]="新疆";Province[17]="西藏";Province[18]="江西";Province[19]="湖南";Province[20]="湖北";
        Province[21]="黑龙江";Province[22]="吉林";Province[23]="辽宁"; Province[24]="江苏";Province[25]="甘肃";Province[26]="青海";Province[27]="四川";
        Province[28]="安徽"; Province[29]="宁夏";Province[30]="海南";Province[31]="香港";Province[32]="澳门";Province[33]="台湾";
    }
    {
        id=id+1;
        value=int(rand()*34);       
        print id"\t"$1"\t"$2"\t"$3"\t"$5"\t"substr($6,1,10)"\t"Province[value]
    }' $infile> $outfile
```

下面就可以执行pre_deal.sh脚本文件，来对raw_user.csv进行数据预处理，命令如下：

```sh
cd /usr/local/bigdatacase/dataset
bash ./pre_deal.sh raw_user.csv user_table.txt
```

可以使用head命令查看前10行数据

```sh
head -10 user_table.txt
```

### 导入数据库

**启动HDFS**

执行下面的命令启动Hadoop：

```sh
start-dfs.sh
```

然后输入jps命令查看当前的进程：

```
43529 NameNode
43692 DataNode
63644 Jps
43901 SecondaryNameNode
```

**把user_table.txt上传到HDFS中**

在HDFS的根目录下创建一个新的目录bigdatacase并在这个目录下创建一个子目录`dataset`

```sh
hdfs dfs -mkdir -p /bigdatacase/dataset
```

然后把本地的user_table.txt上传到分布式文件系统HDFS的`/bigdatacase/dataset`目录下

```sh
hdfs dfs -put /usr/local/bigdatacase/dataset/user_table.txt /bigdatacase/dataset
```

可以查看以下HDFS中`user_table.txt`的前10条记录

```sh
hdfs dfs -cat /bigdatacase/dataset/user_table.txt | head -10
```

**在Hive上创建数据库**

首先启动MySQL数据库

```sh
service mysql start
```

启动hive的元数据

```sh
hive --service metastore
```

在新的终端下进入Hive

```sh
hive
```

在Hive中创建一个数据库dblab

```hive
hive> create database dblab;
hive> use dblab;
```

**创建外部表**

在hive命令提示符下输入

```hive
hive> CREATE EXTERNAL TABLE dblab.bigdata_user(id INT,uid STRING,item_id STRING,behavior_type INT,item_category STRING,visit_date DATE,province STRING) COMMENT 'Welcome to xmudblab!' ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' STORED AS TEXTFILE LOCATION '/bigdatacase/dataset';
```

**查询数据**

在“hive>”命令提示符状态下执行下面命令查看表的信息：

```hive
hive> use dblab;//使用dblab数据库
OK
Time taken: 0.613 seconds
hive> show tables;//显示数据库中所有表
OK
bigdata_user
Time taken: 0.161 seconds, Fetched: 1 row(s)
hive> show create table bigdata_user;//查看bigdata_user表的各种属性；
OK
CREATE EXTERNAL TABLE `bigdata_user`(
  `id` int, 
  `uid` string, 
  `item_id` string, 
  `behavior_type` int, 
  `item_category` string, 
  `visit_date` date, 
  `province` string)
COMMENT 'Welcome to xmudblab!'
ROW FORMAT SERDE 
  'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe' 
WITH SERDEPROPERTIES ( 
  'field.delim'='\t', 
  'serialization.format'='\t') 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  'hdfs://localhost:9000/bigdatacase/dataset'
TBLPROPERTIES (
  'bucketing_version'='2', 
  'transient_lastDdlTime'='1718472356')
Time taken: 0.198 seconds, Fetched: 23 row(s)
```

还可以执行下面命令查看表的简单结构：

```hive
hive> desc bigdata_user;
OK
id                      int                                         
uid                     string                                      
item_id                 string                                      
behavior_type           int                                         
item_category           string                                      
visit_date              date                                        
province                string                                      
Time taken: 0.055 seconds, Fetched: 7 row(s)
```

现在可以使用下面命令查询相关数据：

```hive
hive> select * from bigdata_user limit 10;
OK
1       10001082        285259775       1       4076    2014-12-08      湖南
2       10001082        4368907 1       5503    2014-12-12      安徽
3       10001082        4368907 1       5503    2014-12-12      新疆
4       10001082        53616768        1       9762    2014-12-02      上海市
5       10001082        151466952       1       5232    2014-12-12      黑龙江
6       10001082        53616768        4       9762    2014-12-02      广东
7       10001082        290088061       1       5503    2014-12-12      青海
8       10001082        298397524       1       10894   2014-12-12      香港
9       10001082        32104252        1       6513    2014-12-12      山西
10      10001082        323339743       1       10894   2014-12-12      宁夏
Time taken: 1.483 seconds, Fetched: 10 row(s)
hive> select behavior_type from bigdata_user limit 10;
OK
1
1
1
1
1
4
1
1
1
1
Time taken: 0.106 seconds, Fetched: 10 row(s)
hive> 
```

## Hive数据分析

### 简单查询分析

首先执行一条简单的指令：

```hive
hive> select behavior_type from bigdata_user limit 10;#查看前10位用户对商品的行为
OK
1
1
1
1
1
4
1
1
1
1
Time taken: 0.116 seconds, Fetched: 10 row(s)
```

如果要查出每位用户购买商品时的多种信息，输出语句格式如下：
 select 列1，列2，….，列n from 表名；
比如查询前20位用户购买商品时的时间和商品的种类，语句如下：

```hive
hive> select visit_date, item_category from bigdata_user limit 20;
OK
2014-12-08      4076
2014-12-12      5503
2014-12-12      5503
2014-12-02      9762
2014-12-12      5232
2014-12-02      9762
2014-12-12      5503
2014-12-12      10894
2014-12-12      6513
2014-12-12      10894
2014-12-12      2825
2014-11-28      2825
2014-12-15      3200
2014-12-03      10576
2014-11-20      10576
2014-12-13      10576
2014-12-08      10576
2014-12-14      7079
2014-12-02      6669
2014-12-12      5232
Time taken: 0.089 seconds, Fetched: 20 row(s)
```

有时在表中查询可以利用嵌套语句，如果列名太复杂可以设置该列的别名，以简化操作的难度，举例如下：

```hive
hive> select e.bh, e.it from (select behavior_type as bh, item_category as it from bigdata_user) as e  limit 20;
OK
1       4076
1       5503
1       5503
1       9762
1       5232
4       9762
1       5503
1       10894
1       6513
1       10894
1       2825
1       2825
1       3200
1       10576
1       10576
1       10576
1       10576
1       7079
1       6669
1       5232
Time taken: 0.102 seconds, Fetched: 20 row(s)
```

### 查询条数统计

用聚合函数count()计算出表内有多少行数据

```hive
hive> select count(*) from bigdata_user;
Query ID = hadoop_20240616014249_4ec3aa87-0a06-409c-9c43-40b0dd14ced7
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2024-06-16 01:42:51,137 Stage-1 map = 0%,  reduce = 0%
2024-06-16 01:42:52,142 Stage-1 map = 100%,  reduce = 0%
2024-06-16 01:42:57,166 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local460403743_0001
MapReduce Jobs Launched: 
Stage-Stage-1:  HDFS Read: 5169193912 HDFS Write: 0 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
23291027
Time taken: 8.173 seconds, Fetched: 1 row(s)
```

在函数内部加上distinct，查出uid不重复的数据有多少条

```hive
hive> select count(distinct uid) from bigdata_user;
Query ID = hadoop_20240616014323_2c95c94e-e87b-4d88-9ae6-8a9ca4e8fc61
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2024-06-16 01:43:24,499 Stage-1 map = 0%,  reduce = 0%
2024-06-16 01:43:27,501 Stage-1 map = 100%,  reduce = 0%
2024-06-16 01:43:40,630 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local544803189_0002
MapReduce Jobs Launched: 
Stage-Stage-1:  HDFS Read: 12623097568 HDFS Write: 0 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
20000
Time taken: 17.605 seconds, Fetched: 1 row(s)
```

查询不重复的数据有多少条(为了排除客户刷单情况)

```hive
hive> select count(*) from (select uid,item_id,behavior_type,item_category,visit_date,province from bigdata_user group by uid,item_id,behavior_type,item_category,visit_date,province       having count(*)=1) a;
Query ID = hadoop_20240616014446_fc30d1c3-d4c1-486e-85ce-0a79b93223f2
Total jobs = 2
Launching Job 1 out of 2
Number of reduce tasks not specified. Estimated from input data size: 5
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2024-06-16 01:44:48,421 Stage-1 map = 0%,  reduce = 0%
2024-06-16 01:45:05,693 Stage-1 map = 7%,  reduce = 0%
2024-06-16 01:45:30,144 Stage-1 map = 18%,  reduce = 0%
2024-06-16 01:45:32,205 Stage-1 map = 100%,  reduce = 0%
2024-06-16 01:45:44,425 Stage-1 map = 20%,  reduce = 0%
2024-06-16 01:45:50,471 Stage-1 map = 27%,  reduce = 0%
2024-06-16 01:46:08,032 Stage-1 map = 33%,  reduce = 0%
2024-06-16 01:46:13,059 Stage-1 map = 100%,  reduce = 0%
2024-06-16 01:46:24,425 Stage-1 map = 40%,  reduce = 0%
2024-06-16 01:46:30,623 Stage-1 map = 47%,  reduce = 0%
2024-06-16 01:46:48,985 Stage-1 map = 54%,  reduce = 0%
2024-06-16 01:46:54,057 Stage-1 map = 100%,  reduce = 0%
2024-06-16 01:47:06,424 Stage-1 map = 60%,  reduce = 0%
2024-06-16 01:47:12,464 Stage-1 map = 67%,  reduce = 0%
2024-06-16 01:47:30,843 Stage-1 map = 74%,  reduce = 0%
2024-06-16 01:47:33,963 Stage-1 map = 100%,  reduce = 0%
2024-06-16 01:47:46,284 Stage-1 map = 80%,  reduce = 0%
2024-06-16 01:47:51,291 Stage-1 map = 91%,  reduce = 0%
2024-06-16 01:47:57,476 Stage-1 map = 95%,  reduce = 0%
2024-06-16 01:47:59,479 Stage-1 map = 100%,  reduce = 0%
2024-06-16 01:48:11,495 Stage-1 map = 100%,  reduce = 20%
2024-06-16 01:48:24,516 Stage-1 map = 100%,  reduce = 40%
2024-06-16 01:48:36,535 Stage-1 map = 100%,  reduce = 60%
2024-06-16 01:48:48,554 Stage-1 map = 100%,  reduce = 100%
2024-06-16 01:48:49,556 Stage-1 map = 100%,  reduce = 80%
2024-06-16 01:49:00,573 Stage-1 map = 100%,  reduce = 99%
2024-06-16 01:49:01,578 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local924134795_0003
Launching Job 2 out of 2
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2024-06-16 01:49:02,850 Stage-2 map = 100%,  reduce = 100%
Ended Job = job_local1145871970_0004
MapReduce Jobs Launched: 
Stage-Stage-1:  HDFS Read: 34984890456 HDFS Write: 0 SUCCESS
Stage-Stage-2:  HDFS Read: 7453944616 HDFS Write: 0 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
22080451
Time taken: 255.925 seconds, Fetched: 1 row(s)
```

### 关键字条件查询分析

**以关键字的存在区间为条件的查询**

查询2014年12月10日到2014年12月13日有多少人浏览了商品。

```hive
hive> select count(*) from bigdata_user where behavior_type='1' and visit_date<'2014-12-13' and visit_date>'2014-12-10';
Query ID = hadoop_20240616015218_900aa649-9b2c-4a24-8ebf-6b6a50cb5a06
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2024-06-16 01:52:19,677 Stage-1 map = 0%,  reduce = 0%
2024-06-16 01:52:27,914 Stage-1 map = 100%,  reduce = 0%
2024-06-16 01:52:59,420 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local2129098898_0005
MapReduce Jobs Launched: 
Stage-Stage-1:  HDFS Read: 27530904880 HDFS Write: 0 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
2137738
Time taken: 41.237 seconds, Fetched: 1 row(s)
```

以月的第n天为统计单位，依次显示第n天网站卖出去的商品的个数

```hive
hive> select count(distinct uid), day(visit_date) from bigdata_user where behavior_type='4' group by day(visit_date);
Query ID = hadoop_20240616015346_c33511b1-be1b-4631-9e87-74152ae033e7
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks not specified. Estimated from input data size: 5
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2024-06-16 01:53:48,126 Stage-1 map = 0%,  reduce = 0%
2024-06-16 01:53:51,128 Stage-1 map = 100%,  reduce = 0%
2024-06-16 01:54:04,164 Stage-1 map = 100%,  reduce = 60%
2024-06-16 01:54:05,167 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local1718417321_0006
MapReduce Jobs Launched: 
Stage-Stage-1:  HDFS Read: 59831235976 HDFS Write: 0 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
3098    1
2969    8
3110    13
3225    16
5267    18
2780    21
3104    24
3121    4
2804    5
2910    11
2922    19
2763    22
2990    25
2825    26
2828    28
3137    3
2957    14
3284    15
3049    17
2893    23
2867    7
2794    9
7604    12
2862    20
2913    27
3161    2
2850    6
2869    10
2710    29
3018    30
Time taken: 18.451 seconds, Fetched: 30 row(s)
```

**关键字赋予给定值为条件，对其他数据进行分析**

取给定时间和给定地点，求当天发出到该地点的货物的数量。

```hive
hive> select count(*) from bigdata_user where province='江西' and visit_date='2014-12-12' and behavior_type='4';
Query ID = hadoop_20240616015514_7f149dba-f8a0-4191-a9d9-399ca649580b
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2024-06-16 01:55:15,459 Stage-1 map = 0%,  reduce = 0%
2024-06-16 01:55:24,635 Stage-1 map = 100%,  reduce = 0%
2024-06-16 01:55:58,440 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local2110211223_0007
MapReduce Jobs Launched: 
Stage-Stage-1:  HDFS Read: 42438712192 HDFS Write: 0 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
885
Time taken: 44.359 seconds, Fetched: 1 row(s)
```

### 根据用户行为分析

**查询一件商品在某天的购买比例或浏览比例**

查询有多少用户在2014-12-11购买了商品

```hive
hive> select count(*) from bigdata_user where visit_date='2014-12-11'and behavior_type='4';
Query ID = hadoop_20240616015657_97eeb057-f1c4-4615-8b9c-b52eecb32691
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2024-06-16 01:56:58,654 Stage-1 map = 0%,  reduce = 0%
2024-06-16 01:57:07,712 Stage-1 map = 100%,  reduce = 0%
2024-06-16 01:57:39,940 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local208501678_0008
MapReduce Jobs Launched: 
Stage-Stage-1:  HDFS Read: 49892615848 HDFS Write: 0 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
6771
Time taken: 42.782 seconds, Fetched: 1 row(s)
```

查询有多少用户在2014-12-11点击了该店

```hive
hive> select count(*) from bigdata_user where visit_date ='2014-12-11';
Query ID = hadoop_20240616015820_54043f5d-8fb0-4dd9-97c8-8f2f29d8cc1f
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2024-06-16 01:58:22,332 Stage-1 map = 0%,  reduce = 0%
2024-06-16 01:58:30,479 Stage-1 map = 100%,  reduce = 0%
2024-06-16 01:59:01,726 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local981603111_0009
MapReduce Jobs Launched: 
Stage-Stage-1:  HDFS Read: 57346519504 HDFS Write: 0 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
944979
Time taken: 40.809 seconds, Fetched: 1 row(s)
```

**查询某个用户在某一天点击网站占该天所有点击行为的比例（点击行为包括浏览、加入购物车、收藏、购买）**

查询用户10001082在2014-12-12点击网站的次数

```hive
hive> select count(*) from bigdata_user where uid=10001082 and visit_date='2014-12-12';
Query ID = hadoop_20240616020035_9ad77af3-2cba-4456-924d-9646ed7be7d0
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2024-06-16 02:00:36,927 Stage-1 map = 0%,  reduce = 0%
2024-06-16 02:00:46,135 Stage-1 map = 100%,  reduce = 0%
2024-06-16 02:01:20,744 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local219554971_0010
MapReduce Jobs Launched: 
Stage-Stage-1:  HDFS Read: 64800423160 HDFS Write: 0 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
69
Time taken: 45.22 seconds, Fetched: 1 row(s)
```

查询所有用户在这一天点击该网站的次数

```hive
hive> select count(*) from bigdata_user where visit_date='2014-12-12';
Query ID = hadoop_20240616020246_fcb99cc6-d131-4e1c-9cc0-ce38b7de499d
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2024-06-16 02:02:48,411 Stage-1 map = 0%,  reduce = 0%
2024-06-16 02:02:56,581 Stage-1 map = 100%,  reduce = 0%
2024-06-16 02:03:28,882 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local1580275434_0011
MapReduce Jobs Launched: 
Stage-Stage-1:  HDFS Read: 72254326816 HDFS Write: 0 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
1344980
Time taken: 41.911 seconds, Fetched: 1 row(s)
```

**给定购买商品的数量范围，查询某一天在该网站的购买该数量商品的用户id**

查询某一天在该网站购买商品超过5次的用户id

```hive
hive> select uid from bigdata_user where behavior_type='4' and visit_date='2014-12-12' group by uid having count(behavior_type='4')>5;
Query ID = hadoop_20240616020903_0ad7cef8-7e49-4107-8683-fd98599a18e1
Total jobs = 1
Launching Job 1 out of 1
Number of reduce tasks not specified. Estimated from input data size: 5
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2024-06-16 02:09:07,206 Stage-1 map = 0%,  reduce = 0%
2024-06-16 02:09:18,314 Stage-1 map = 100%,  reduce = 0%
2024-06-16 02:09:56,917 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local1592485813_0001
MapReduce Jobs Launched: 
Stage-Stage-1:  HDFS Read: 10138340136 HDFS Write: 0 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
100605
100728355
100796503
10095384
...
64315314
6500849
65095287
6534626
Time taken: 53.599 seconds, Fetched: 1564 row(s)
```

### 用户实时查询分析

**查询某个地区的用户当天浏览网站的次数**

创建新的数据表进行存储

```hive
hive> create table scan(province STRING,scan INT) COMMENT 'This is the search of bigdataday' ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' STORED AS TEXTFILE;
OK
Time taken: 1.17 seconds
```

导入数据

```hive
hive> insert overwrite table scan select province,count(behavior_type) from bigdata_user where behavior_type='1' group by province;
Query ID = hadoop_20240616021413_4ab776e4-cc98-475c-95bc-6973ee82e279
Total jobs = 2
Launching Job 1 out of 2
Number of reduce tasks not specified. Estimated from input data size: 5
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2024-06-16 02:14:15,184 Stage-1 map = 0%,  reduce = 0%
2024-06-16 02:14:17,188 Stage-1 map = 100%,  reduce = 0%
2024-06-16 02:14:27,290 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local52434044_0002
Loading data to table dblab.scan
Launching Job 2 out of 2
Number of reduce tasks determined at compile time: 1
In order to change the average load for a reducer (in bytes):
  set hive.exec.reducers.bytes.per.reducer=<number>
In order to limit the maximum number of reducers:
  set hive.exec.reducers.max=<number>
In order to set a constant number of reducers:
  set mapreduce.job.reduces=<number>
Job running in-process (local Hadoop)
2024-06-16 02:14:28,626 Stage-3 map = 100%,  reduce = 100%
Ended Job = job_local2032099341_0003
MapReduce Jobs Launched: 
Stage-Stage-1:  HDFS Read: 22561512896 HDFS Write: 2644 SUCCESS
Stage-Stage-3:  HDFS Read: 4969269104 HDFS Write: 1654 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
Time taken: 15.49 seconds
```

显示结果

```hive
hive> select * from scan;
OK
台湾    645421
四川    646636
宁夏    643964
安徽    645594
广东    643067
广西    645380
河南    645191
贵州    645203
辽宁    644007
陕西    645452
吉林    645223
江西    644830
甘肃    645495
西藏    644691
重庆市  645230
上海市  644384
云南    645295
北京市  645735
江苏    646701
浙江    645915
海南    646311
湖北    645080
青海    645554
内蒙古  645438
天津市  645704
山东    646162
山西    644977
新疆    644826
湖南    645725
澳门    645513
香港    644758
黑龙江  645571
河北    645313
福建    646174
Time taken: 0.093 seconds, Fetched: 34 row(s)
```

## Hive、MySQL、HBase数据互导

### Hive预操作

创建临时表`user_action`

```hive
hive> create table dblab.user_action(id STRING,uid STRING, item_id STRING, behavior_type STRING, item_category STRING, visit_date DATE, province STRING) COMMENT 'Welcome to XMU dblab! ' ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t' STORED AS TEXTFILE;
OK
Time taken: 0.277 seconds
```

现在可以新建一个终端，执行命令查看一下，确认这个数据文件在HDFS中确实已经被创建，在新建的终端中执行下面命令

```sh
hdfs dfs -ls /user/hive/warehouse/dblab.db/
```

可以看到目录内容：

```
Found 2 items
drwxr-xr-x   - hadoop supergroup          0 2024-06-16 02:14 /user/hive/warehouse/dblab.db/scan
drwxr-xr-x   - hadoop supergroup          0 2024-06-16 02:18 /user/hive/warehouse/dblab.db/user_action
```

**将`bigdata_user`表中的数据插入到`user_action`**

下面把`dblab.bigdata_user`数据插入到`dblab.user_action`表中，命令如下：

```hive
hive> INSERT OVERWRITE TABLE dblab.user_action select * from dblab.bigdata_user;
Query ID = hadoop_20240616022349_4259971c-f495-4711-be42-69c01bc36c59
Total jobs = 3
Launching Job 1 out of 3
Number of reduce tasks is set to 0 since there's no reduce operator
Job running in-process (local Hadoop)
2024-06-16 02:23:51,241 Stage-1 map = 0%,  reduce = 0%
2024-06-16 02:24:03,244 Stage-1 map = 10%,  reduce = 0%
2024-06-16 02:24:06,246 Stage-1 map = 100%,  reduce = 0%
2024-06-16 02:24:18,254 Stage-1 map = 30%,  reduce = 0%
2024-06-16 02:24:24,260 Stage-1 map = 100%,  reduce = 0%
2024-06-16 02:24:36,268 Stage-1 map = 50%,  reduce = 0%
2024-06-16 02:24:41,272 Stage-1 map = 100%,  reduce = 0%
2024-06-16 02:24:53,281 Stage-1 map = 70%,  reduce = 0%
2024-06-16 02:24:58,285 Stage-1 map = 100%,  reduce = 0%
Ended Job = job_local1890104177_0013
Stage-4 is selected by condition resolver.
Stage-3 is filtered out by condition resolver.
Stage-5 is filtered out by condition resolver.
Moving data to directory hdfs://localhost:9000/user/hive/warehouse/dblab.db/user_action/.hive-staging_hive_2024-06-16_02-23-49_848_6247200484593972572-1/-ext-10000
Loading data to table dblab.user_action
MapReduce Jobs Launched: 
Stage-Stage-1:  HDFS Read: 72254308806 HDFS Write: 3926636398 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
OK
Time taken: 79.633 seconds
```

然后执行下面命令查询上面的插入命令是否成功执行

```hive
hive> select * from user_action limit 10;
OK
1       10001082        285259775       1       4076    2014-12-08      湖南
2       10001082        4368907 1       5503    2014-12-12      安徽
3       10001082        4368907 1       5503    2014-12-12      新疆
4       10001082        53616768        1       9762    2014-12-02      上海市
5       10001082        151466952       1       5232    2014-12-12      黑龙江
6       10001082        53616768        4       9762    2014-12-02      广东
7       10001082        290088061       1       5503    2014-12-12      青海
8       10001082        298397524       1       10894   2014-12-12      香港
9       10001082        32104252        1       6513    2014-12-12      山西
10      10001082        323339743       1       10894   2014-12-12      宁夏
Time taken: 0.081 seconds, Fetched: 10 row(s)
```

### 使用Java API将数据从Hive导入MySQL

**将前面生成的临时表数据从Hive导入到 MySQL中**

登录MySQL

```sh
sudo mysql -u root
```

创建数据库并授权给`hadoop`用户

```sql
mysql> show databases; # 显示所有数据库
+--------------------+
| Database           |
+--------------------+
| hive               |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+

mysql> create database dblab; 创建dblab数据库
Query OK, 1 row affected (0.02 sec)

mysql> grant all privileges on dblab.* to hadoop; # 将dblab数据库授权给hadoop用户
Query OK, 0 rows affected (0.01 sec)
```

退出`root`用户
```sql
mysql> exit;
Bye
```

登陆`hadoop`用户

```sql
mysql> show databases; # 显示所有数据库
+--------------------+
| Database           |
+--------------------+
| dblab              |
| hive               |
| information_schema |
| performance_schema |
+--------------------+
4 rows in set (0.00 sec)

mysql> use dblab;
Database changed
```

注意：用下面命令查看数据库的编码

```sql
mysql> show variables like "char%";
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8mb4                    |
| character_set_connection | utf8mb4                    |
| character_set_database   | utf8mb4                    |
| character_set_filesystem | binary                     |
| character_set_results    | utf8mb4                    |
| character_set_server     | utf8mb4                    |
| character_set_system     | utf8mb3                    |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.01 sec)
```

> 如果character_set_server不是utf8之类的编码，则需要修改配置文件
> 
> 在`[mysqld]`下添加一行`character_set_server=utf8`
> 
> ```cnf /etc/mysql/mysql.conf.d/mysqld.cnf
> [mysqld]
> character_set_server=utf8
> ```
> 
> 然后重启MySQL服务
> 
> ```sh
> sudo service mysql restart
> ```

创建表

下面在MySQL的数据库`dblab`中创建一个新表`user_action`，并设置其编码为`utf-8`：

```sql
mysql> CREATE TABLE `dblab`.`user_action` (`id` varchar(50),`uid` varchar(50),`item_id` varchar(50),`behavior_type` varchar(10),`item_category` varchar(50), `visit_date` DATE,`province` varchar(20)) ENGINE=InnoDB DEFAULT CHARSET=utf8;
Query OK, 0 rows affected, 1 warning (0.03 sec)
```

创建成功后`exit`退出MySQL

导入数据

通过JDBC连接Hive和MySQL，将数据从Hive导入MySQL。通过JDBC连接Hive，需要通过Hive的thrift服务实现跨语言访问Hive，实现thrift服务需要开启hiveserver2。

启动HDFS以后，在终端中启动`metastore`

```sh
hive --service metastore
```

在另一个终端中执行以下命令开启`HiveServer2`，并设置默认端口为10000（如果按照前面教程配置了hiveserver2的端口，这里的`-hiveconf`选项不需要添加）

```sh
hive --service hiveserver2 -hiveconf hive.server2.thrift.port=10000
```

启动时，当屏幕上出现“Hive Session ID = 6bd1726e-37c5-41fc-93ea-ef7e176b24f2”信息时，会停留较长的时间，需要出现几个“Hive Session ID=...”以后，Hive才会真正启动。启动成功以后，会出现如下信息

```
Hive Session ID = bb1c1106-83ce-4653-9541-c79a86945ea0
Hive Session ID = 0e577e34-83c1-40de-9fb6-917be3d5e022
```

不要关闭这个终端，保持服务的运行

在新的终端下运行以下命令查看10000号和10002号端口是否已经被占用，如果被占用，说明启动成功，可以在浏览器访问`127.0.0.1:10002`打开hiveserver2的网页接口

```sh
sudo netstat -anp |grep "10000\|10002"
```

端口信息如下

```
tcp6       0      0 :::10000                :::*                    LISTEN      76077/java          
tcp6       0      0 :::10002                :::*                    LISTEN      76077/java
```

启动IntelliJ IDEA，创建一个项目名为HivetoMySQL

将`$HADOOP_HOME/share/hadoop/common`和`$HIVE_HOME/lib`目录下JAR包导入到项目中，[^3] <a id="jar"></a>
1. 点击工具栏的 File 选项
2. 点击 Project Structure（CTRL + SHIFT + ALT + S on Windows/Linux，⌘ + ; on Mac OS X）
3. 选择左侧的 Modules 选项
4. 选择 Dependencies tab
5. 点击 + → JARs or directories
6. 选择需要导入JAR包的所在目录
7. 点击 Apply 应用选项

[^3]: [Add a new dependency](https://www.jetbrains.com/help/idea/working-with-module-dependencies.html#add-a-new-dependency)

导入后可以在左边的External Libraries中看到导入的JAR包

删除原有的`Main`类，创建一个公共类`HivetoMySQL`

编写`HivetoMySQL.java`代码，注意用户名和密码，以及端口是否正确，运行代码等待导入

然后就是漫漫漫漫漫漫漫漫漫漫漫漫长长长长长长长长长长长长长长的等待时间.....................（人生苦短，不要用raw_user............）

```java HivetoMySQL
import java.sql.*;
import java.sql.SQLException;

public class HivetoMySQL {
    private static String driverName = "org.apache.hive.jdbc.HiveDriver";
    private static String driverName_mysql = "com.mysql.jdbc.Driver";

    public static void main(String[] args) throws SQLException {
        try {
            Class.forName(driverName);
        } catch (ClassNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
            System.exit(1);
        }
        Connection con1 = DriverManager.getConnection("jdbc:hive2://localhost:10000/default", "hive", "hive");//后两个参数是用户名密码

        if (con1 == null) System.out.println("连接失败");
        else {
            Statement stmt = con1.createStatement();
            String sql = "select * from dblab.user_action";
            System.out.println("Running: " + sql);
            ResultSet res = stmt.executeQuery(sql);

            //InsertToMysql
            try {
                Class.forName(driverName_mysql);
                Connection con2 = DriverManager.getConnection("jdbc:mysql://localhost:3306/dblab", "hadoop", "hadoop");
                String sql2 = "insert into user_action(id,uid,item_id,behavior_type,item_category,visit_date,province) values (?,?,?,?,?,?,?)";
                PreparedStatement ps = con2.prepareStatement(sql2);
                while (res.next()) {
                    ps.setString(1, res.getString(1));
                    ps.setString(2, res.getString(2));
                    ps.setString(3, res.getString(3));
                    ps.setString(4, res.getString(4));
                    ps.setString(5, res.getString(5));
                    ps.setDate(6, res.getDate(6));
                    ps.setString(7, res.getString(7));
                    ps.executeUpdate();
                }
                ps.close();
                con2.close();
                res.close();
                stmt.close();
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            }
        }
        con1.close();
    }
}
```

### 使用HBase Java API把数据从本地导入到HBase中

**数据准备**

先将之前的`user_action`数据从HDFS复制到Linux系统的本地文件系统中

```sh
hdfs dfs -get /user/hive/warehouse/dblab.db/user_action /usr/local/bigdatacase/dataset/
cat /usr/local/bigdatacase/dataset/user_action/* | head -10     # 查看前十行数据
cat /usr/local/bigdatacase/dataset/user_action/00000* > /usr/local/bigdatacase/dataset/user_action.output    #将00000*数据复制一份命名为`user_action.output`
head /usr/local/bigdatacase/dataset/user_action.output      # 查看user_action.output的前十行
```

创建一个项目，起名叫`ImportHBase`，像[上面](#jar)一样将目录`$HADOOP_HOME/share/hadoop/common`和`$HBASE_HOME/lib`下的JAR包导入到项目中，删除原有的`Main`类，新建一个`ImportHBase`类

编写代码`ImportHBase.java`

```java
import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.List;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.*;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

public class ImportHBase extends Thread {
    public Configuration config;
    public Connection conn;
    public Table table;
    public Admin admin;

    public ImportHBase() {
        config = HBaseConfiguration.create();
//      config.set("hbase.master", "master:60000");
//      config.set("hbase.zookeeper.quorum", "master");
        try {
            conn = ConnectionFactory.createConnection(config);
            admin = conn.getAdmin();
            table = conn.getTable(TableName.valueOf("user_action"));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws Exception {
        if (args.length == 0) {       //第一个参数是该jar所使用的类，第二个参数是数据集所存放的路径
            throw new Exception("You must set input path!");
        }
        String fileName = args[args.length - 1];  //输入的文件路径是最后一个参数
        ImportHBase test = new ImportHBase();
        test.importLocalFileToHBase(fileName);
    }

    public void importLocalFileToHBase(String fileName) {
        long st = System.currentTimeMillis();
        BufferedReader br = null;
        try {
            br = new BufferedReader(new InputStreamReader(new FileInputStream(
                    fileName)));
            String line = null;
            int count = 0;
            while ((line = br.readLine()) != null) {
                count++;
                put(line);
                if (count % 10000 == 0)
                    System.out.println(count);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (br != null) {
                try {
                    br.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            try {
                table.close(); // must close the client
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        long en2 = System.currentTimeMillis();
        System.out.println("Total Time: " + (en2 - st) + " ms");
    }

    @SuppressWarnings("deprecation")
    public void put(String line) throws IOException {
        String[] arr = line.split("\t", -1);
        String[] column = {"id", "uid", "item_id", "behavior_type", "item_category", "date", "province"};
        if (arr.length == 7) {
            Put put = new Put(Bytes.toBytes(arr[0]));// rowkey
            for (int i = 1; i < arr.length; i++) {
                put.addColumn(Bytes.toBytes("f1"), Bytes.toBytes(column[i]), Bytes.toBytes(arr[i]));
            }
            table.put(put); // put to server
        }
    }

    public void get(String rowkey, String columnFamily, String column,
                    int versions) throws IOException {
        long st = System.currentTimeMillis();
        Get get = new Get(Bytes.toBytes(rowkey));
        get.addColumn(Bytes.toBytes(columnFamily), Bytes.toBytes(column));
        Scan scanner = new Scan(get);
        scanner.readVersions(versions);
        ResultScanner rsScanner = table.getScanner(scanner);
        for (Result result : rsScanner) {
            final List<Cell> list = result.listCells();
            for (final Cell kv : list) {
                System.out.println(Bytes.toStringBinary(kv.getValueArray()) + "\t"
                        + kv.getTimestamp()); // mid + time
            }
        }
        rsScanner.close();
        long en2 = System.currentTimeMillis();
        System.out.println("Total Time: " + (en2 - st) + " ms");
    }
}
```

打包成可执行jar包[^4]
1. 点击工具栏 Files 选项
2. 点击 Project Structure（CTRL + SHIFT + ALT + S on Windows/Linux，⌘ + ; on Mac OS X）
3. 点击 Artifacts
4. 点击 + 号，选择JAR，From Modules with dependencies
5. Main Class 选择ImportHBase
6. JAR files from libraries 选择 extract to the target JAR
7. 选择META-INF/MANIFEST.MF的路径，路径选择src即可
8. 点击OK完成配置
9. 点击工具栏 Build 选项
10. 点击Build Artifact
11. 点击 Build 等待编译

[^4]: [Deployment/Artifacts](https://www.jetbrains.com/help/idea/working-with-artifacts.html#configure_artifact)

将生成的jar包复制到`/usr/local/bigdatacase/hbase`目录

```sh
mkdir /usr/local/bigdatacase/hbase
cp ImportHBase.jar /usr/local/bigdatacase/hbase
```

启动HDFS、ZooKeeper和HBase

```sh
start-dfs.sh
zkServer.sh start
start-hbase.sh
```

启动HBase Shell

```sh
hbase shell
```

启动成功后，进入"hbase>"命令提示符状态

创建`user_action`表

```hbase
hbase:001:0> create 'user_action', { NAME => 'f1', VERSIONS => 5}
Created table user_action
Took 1.5141 seconds                                                                                                                     
=> Hbase::Table - user_action
```

确保`user_action`表为空

```hbase
hbase:001:0> truncate 'user_action'
Truncating 'user_action' table (it may take a while):
Disabling table...
Truncating table...
Took 2.9350 seconds
```

通过hadoop jar命令运行上面的Java程序

```sh
hadoop jar /usr/local/bigdatacase/hbase/ImportHBase ImportHBase /usr/local/bigdatacase/dataset/user_action.output
```

然后就是漫长的等待时间（相较于上面的Hive导入MySQL，已经是非常非常快了）

导入完成以后在HBase Shell窗口，执行下面命令查询数据

```hbase
hbase:001:0> scan 'user_action',{LIMIT=>10} 
ROW                                                                   COLUMN+CELL                                                                                                                                                                                                  
 1                                                                    column=f1:behavior_type, timestamp=2024-06-16T04:51:32.554, value=1                                                                                                                                          
 1                                                                    column=f1:date, timestamp=2024-06-16T04:51:32.554, value=2014-12-08                                                                                                                                          
 1                                                                    column=f1:item_category, timestamp=2024-06-16T04:51:32.554, value=4076                                                                                                                                       
 1                                                                    column=f1:item_id, timestamp=2024-06-16T04:51:32.554, value=285259775                                                                                                                                        
 1                                                                    column=f1:province, timestamp=2024-06-16T04:51:32.554, value=\xE6\xB9\x96\xE5\x8D\x97                                                                                                                        
 1                                                                    column=f1:uid, timestamp=2024-06-16T04:51:32.554, value=10001082                                                                                                                                             
 10                                                                   column=f1:behavior_type, timestamp=2024-06-16T04:51:32.573, value=1                                                                                                                                          
 10                                                                   column=f1:date, timestamp=2024-06-16T04:51:32.573, value=2014-12-12                                                                                                                                          
 10                                                                   column=f1:item_category, timestamp=2024-06-16T04:51:32.573, value=10894                                                                                                                                      
 10                                                                   column=f1:item_id, timestamp=2024-06-16T04:51:32.573, value=323339743                                                                                                                                        
 10                                                                   column=f1:province, timestamp=2024-06-16T04:51:32.573, value=\xE5\xAE\x81\xE5\xA4\x8F                                                                                                                        
 10                                                                   column=f1:uid, timestamp=2024-06-16T04:51:32.573, value=10001082                                                                                                                                             
 100                                                                  column=f1:behavior_type, timestamp=2024-06-16T04:51:32.709, value=1                                                                                                                                          
 100                                                                  column=f1:date, timestamp=2024-06-16T04:51:32.709, value=2014-12-02                                                                                                                                          
 100                                                                  column=f1:item_category, timestamp=2024-06-16T04:51:32.709, value=10576                                                                                                                                      
 100                                                                  column=f1:item_id, timestamp=2024-06-16T04:51:32.709, value=275221686                                                                                                                                        
 100                                                                  column=f1:province, timestamp=2024-06-16T04:51:32.709, value=\xE6\xB1\x9F\xE8\x8B\x8F                                                                                                                        
 100                                                                  column=f1:uid, timestamp=2024-06-16T04:51:32.709, value=10001082                                                                                                                                             
 1000                                                                 column=f1:behavior_type, timestamp=2024-06-16T04:51:33.352, value=1                                                                                                                                          
 1000                                                                 column=f1:date, timestamp=2024-06-16T04:51:33.352, value=2014-12-02                                                                                                                                          
 1000                                                                 column=f1:item_category, timestamp=2024-06-16T04:51:33.352, value=3381                                                                                                                                       
 1000                                                                 column=f1:item_id, timestamp=2024-06-16T04:51:33.352, value=168463559                                                                                                                                        
 1000                                                                 column=f1:province, timestamp=2024-06-16T04:51:33.352, value=\xE5\x86\x85\xE8\x92\x99\xE5\x8F\xA4                                                                                                            
 1000                                                                 column=f1:uid, timestamp=2024-06-16T04:51:33.352, value=100068031                                                                                                                                            
 10000                                                                column=f1:behavior_type, timestamp=2024-06-16T04:51:35.031, value=1                                                                                                                                          
 10000                                                                column=f1:date, timestamp=2024-06-16T04:51:35.031, value=2014-12-05                                                                                                                                          
 10000                                                                column=f1:item_category, timestamp=2024-06-16T04:51:35.031, value=12488                                                                                                                                      
 10000                                                                column=f1:item_id, timestamp=2024-06-16T04:51:35.031, value=45571867                                                                                                                                         
 10000                                                                column=f1:province, timestamp=2024-06-16T04:51:35.031, value=\xE8\xB4\xB5\xE5\xB7\x9E                                                                                                                        
 10000                                                                column=f1:uid, timestamp=2024-06-16T04:51:35.031, value=100198255                                                                                                                                            
 100000                                                               column=f1:behavior_type, timestamp=2024-06-16T04:51:46.061, value=1                                                                                                                                          
 100000                                                               column=f1:date, timestamp=2024-06-16T04:51:46.061, value=2014-11-29                                                                                                                                          
 100000                                                               column=f1:item_category, timestamp=2024-06-16T04:51:46.061, value=6580                                                                                                                                       
 100000                                                               column=f1:item_id, timestamp=2024-06-16T04:51:46.061, value=78973192                                                                                                                                         
 100000                                                               column=f1:province, timestamp=2024-06-16T04:51:46.061, value=\xE5\xB1\xB1\xE4\xB8\x9C                                                                                                                        
 100000                                                               column=f1:uid, timestamp=2024-06-16T04:51:46.061, value=101480065                                                                                                                                            
 1000000                                                              column=f1:behavior_type, timestamp=2024-06-16T04:53:21.091, value=1                                                                                                                                          
 1000000                                                              column=f1:date, timestamp=2024-06-16T04:53:21.091, value=2014-11-29                                                                                                                                          
 1000000                                                              column=f1:item_category, timestamp=2024-06-16T04:53:21.091, value=10961                                                                                                                                      
 1000000                                                              column=f1:item_id, timestamp=2024-06-16T04:53:21.091, value=88763784                                                                                                                                         
 1000000                                                              column=f1:province, timestamp=2024-06-16T04:53:21.091, value=\xE5\xB1\xB1\xE8\xA5\xBF                                                                                                                        
 1000000                                                              column=f1:uid, timestamp=2024-06-16T04:53:21.091, value=111256954                                                                                                                                            
 10000000                                                             column=f1:behavior_type, timestamp=2024-06-16T05:09:10.864, value=1                                                                                                                                          
 10000000                                                             column=f1:date, timestamp=2024-06-16T05:09:10.864, value=2014-11-27                                                                                                                                          
 10000000                                                             column=f1:item_category, timestamp=2024-06-16T05:09:10.864, value=1854                                                                                                                                       
 10000000                                                             column=f1:item_id, timestamp=2024-06-16T05:09:10.864, value=87168683                                                                                                                                         
 10000000                                                             column=f1:province, timestamp=2024-06-16T05:09:10.864, value=\xE6\xB9\x96\xE5\x8D\x97                                                                                                                        
 10000000                                                             column=f1:uid, timestamp=2024-06-16T05:09:10.864, value=123265349                                                                                                                                            
 10000001                                                             column=f1:behavior_type, timestamp=2024-06-16T05:09:10.864, value=1                                                                                                                                          
 10000001                                                             column=f1:date, timestamp=2024-06-16T05:09:10.864, value=2014-11-18                                                                                                                                          
 10000001                                                             column=f1:item_category, timestamp=2024-06-16T05:09:10.864, value=13712                                                                                                                                      
 10000001                                                             column=f1:item_id, timestamp=2024-06-16T05:09:10.864, value=125559152                                                                                                                                        
 10000001                                                             column=f1:province, timestamp=2024-06-16T05:09:10.864, value=\xE6\xB9\x96\xE5\x8C\x97                                                                                                                        
 10000001                                                             column=f1:uid, timestamp=2024-06-16T05:09:10.864, value=123265349                                                                                                                                            
 10000002                                                             column=f1:behavior_type, timestamp=2024-06-16T05:09:10.864, value=1                                                                                                                                          
 10000002                                                             column=f1:date, timestamp=2024-06-16T05:09:10.864, value=2014-12-18                                                                                                                                          
 10000002                                                             column=f1:item_category, timestamp=2024-06-16T05:09:10.864, value=14079                                                                                                                                      
 10000002                                                             column=f1:item_id, timestamp=2024-06-16T05:09:10.864, value=222706509                                                                                                                                        
 10000002                                                             column=f1:province, timestamp=2024-06-16T05:09:10.864, value=\xE5\x86\x85\xE8\x92\x99\xE5\x8F\xA4                                                                                                            
 10000002                                                             column=f1:uid, timestamp=2024-06-16T05:09:10.864, value=123265349
```

> 没啦，别问为什么没写完，因为数据没导完啊啊啊啊啊啊啊啊啊啊啊啊啊啊啊啊
> 
> 不愧是大数据
> 

在这里放一只小猫，学累了就摸摸它

```
　　　　　　 ＿＿
　　　　　／＞　　フ
　　　　　| 　_　 _ l
　 　　　／` ミ＿xノ
　　 　 /　　　 　 |
　　　 /　 ヽ　　 ﾉ
　 　 │　　|　|　|
　／￣|　　 |　|　|
　| (￣ヽ＿_ヽ_)__)
　＼二つ
```
