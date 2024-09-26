---
title: 大数据实验踩坑指南_No.2——Hive的安装配置
mathjax: false
katex: false
date: 2024-06-16 16:27:21
description: 分布式数据仓库系统Hive的安装、配置过程记录
categories:
- Big Data
tags:
- Big Data
- MySQL
- Database
---

# 大数据实验踩坑指南_No.2——Hive的安装配置

> 事先说明，Beeline还有很多坑没有处理好，暂时先跳过，只使用传统Hive Cli

## Hive基础安装

通过[官网](https://hive.apache.org)下载[apache-hive-3.1.3-bin.tar.gz](https://dlcdn.apache.org/hive/hive-3.1.3/apache-hive-3.1.3-bin.tar.gz)

根据另一篇文章[^1]安装好MySQL

[^1]: {% post_link Ubuntu下的MySQL安装配置 %}

将文件解压到`/usr/local`目录内

```sh
sudo tar -zxvf /home/hadoop/Downloads/apache-hive-3.1.3-bin.tar.gz -C /usr/local/
```

进入该目录并将Hive的主目录改名

```sh
cd /usr/local
sudo mv apache-hive-3.1.3-bin hive
```

修改目录拥有者

```sh
sudo chown -R hadoop:hadoop hive
```

配置Hive相关的环境变量

```sh /home/hadoop/.bashrc
export HIVE_HOME=/usr/local/hive
export PATH=$PATH:$HIVE_HOME/bin
```

加载配置文件

```sh
source /home/hadoop/.bashrc
```

## MySQL配置

启动MySQL服务

```sh
sudo service mysql start
```

登录MySQl Shell

```sh
sudo mysql -u root
```

创建`hadoop`用户，密码为`hadoop`

```sql
mysql> CREATE USER 'hadoop'@'%' IDENTIFIED BY 'hadoop';
```

创建`hive`数据库并把数据库的权限赋予`hadoop`用户

```sql
mysql> create database hive;
mysql> grant all privileges on hive.* to hadoop;
```

最后记得刷新权限

```sql
mysql> flush privileges;
```

输入`exit`退出MySQL

登录`hadoop`目录，输入密码`hadoop`

```sh
mysql -u hadoop -p
```

下载JDBC连接驱动的[deb安装包](https://downloads.mysql.com/archives/c-j/)，我的MySQL版本为8.0.37，网站上写的Product Version只有8.0.33，不过也是兼容的，选择好MySQL版本和Ubuntu版本，即可下载[deb安装包](https://downloads.mysql.com/archives/get/p/3/file/mysql-connector-j_8.0.33-1ubuntu22.04_all.deb)
> 现在MySQL版本更新到了9，可以直接使用MySQL 9和对应的JDBC驱动

通过`apt`工具安装驱动

```sh
sudo apt-get install /home/hadoop/Downloads/mysql-connector-j_8.0.33-1ubuntu22.04_all.deb
```

然后将安装好的驱动复制到Hive的lib目录下

```sh
cp /usr/share/java/mysql-connector-j-8.0.33.jar $HIVE_HOME/lib/
```

## MySQL注意事项

#### 通信问题

为了让Hive与MySQL能通信，需要修改以下配置

```cnf /etc/mysql/mysql.conf.d/mysqld.cnf
[mysqld]
bind-address    = 0.0.0.0
```

#### 命令差异

MySQL8和MySQL5.7命令有差异

##### 创建用户与赋权

在MySQL5.7版本上，使用以下命令可以同时创建账户并赋权

```sql
mysql> GRANT ALL PRIVILEGES ON *.* TO 'hadoop'@'%' IDENTIFIED BY 'hadoop' WITH GRANT OPTION;
```

但是在新版本的`MySQL 8.x`版本上，已经将两个操作分离，执行以上命令会导致语法错误

```sql
mysql> 'hadoop'@'%' IDENTIFIED BY 'hadoop' WITH GRANT OPTION;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'IDENTIFIED BY '123456' WITH GRANT OPTION' at line 1
```

应将创建用户与授权命令分离：

```sql
mysql> CREATE USER 'hadoop'@'%' IDENTIFIED BY '123456';
mysql> GRANT ALL ON hive.* TO 'hadoop'@'%';
```

##### 修改密码

MySQL修改密码有三种方式[^2]

[^2]: [MySQL修改密码（三种方法示例）](https://blog.csdn.net/happyxin_/article/details/125614272)

**方法一**：使用UPDATE语句更改MySQL用户密码

```sql
mysql> USE mysql;
mysql> UPDATE user 
            SET password = PASSWORD('newpasswd')
            WHERE user = 'hadoop' AND host = '%';
mysql> FLUSH PRIVILEGES;
```

> 注意：MySQL 5.7.6版本以下，才能使用此方法来修改密码。从MySQL 5.7.6版本起，user表仅使用authentication_string列代替之前版本中的password列来存储密码。此外，它删除了password列。

版本在5.7.6+，必须在UPDATE语句中使用authentication_string列代替password列

```sql
mysql> USE mysql;
mysql> UPDATE user 
mysql> SET authentication_string = PASSWORD('newpasswd')
            WHERE user = 'jccc' AND 
                  host = '%';
mysql> FLUSH PRIVILEGES;
```

**方法二**：使用SET PASSWORD语句更改MySQL用户密码

```sql
mysql> SET PASSWORD FOR 'jccc'@'%' = PASSWORD('newpasswd2');
```

从MySQL 5.7.6版本开始，MySQL不推荐使用此语法，可能会在将来的版本中将其删除。作为一个代替的解决方案，它使用明文密码如下：

```sql
mysql> SET PASSWORD FOR 'jccc'@'%' = 'newpasswd2';
```

**方法三**：使用`ALTER USER`语句改MySQL用户密码

```sql
mysql> ALTER USER jccc@% IDENTIFIED BY 'newpasswd3';
```

## Hive配置

### Hive自身配置

进入Hive的目录，复制几个配置文件的模板

```sh
cd $HIVE_HOME/conf
cp hive-env.sh.template hive-env.sh
cp hive-default.xml.template hive-default.xml
```

配置`hive-site.xml`文件，跟使用TiDB[^3]的配置类似
[^3]: [Using TiDB as the Hive Metastore database](https://cwiki.apache.org/confluence/display/Hive/Using+TiDB+as+the+Hive+Metastore+database)

各个选项的说明可以在[这里](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties?spm=5176.28103460.0.0.7d815d27rIvLt1#ConfigurationProperties-AuthenticationandAuthorization)看到

> **注意**：
> 
> [创建用户和授权命令在MySQL8.x版本中有所不同](#命令差异)，配置文件中根据你创建的MySQL用户和密码进行修改。
> 
> `hive-site.xml`配置文件中，需要将`javax.jdo.option.ConnectionDriverName`的值修改为`com.mysql.cj.jdbc.Driver`，而原本的`com.mysql.jdbc.Driver`在较新的MySQL Connector/J版本中已被标记为过时，新的驱动类提供了更好的功能和性能。如果使用原本的值，初始化Hive会出现以下警告。
> 
> ```
> Loading class 'com.mysql.jdbc.Driver'. This is deprecated. The new driver class is 'com.mysql.cj.jdbc.Driver'. The driver is automatically registered via the SPI and manual loading of the driver class is generally unnecessary.
> ```
> 
> 还有用户名密码要设为MySQL中对应的。

```xml $HIVE_HOME/conf/hive-site.xml
<configuration>
	<!-- MySQL连接配置 -->
	<property>
		<name>javax.jdo.option.ConnectionURL</name>
		<value>jdbc:mysql://localhost:3306/hive</value>
		<description>MySQL address</description>
	</property>

	<property>
		<name>javax.jdo.option.ConnectionUserName</name>
		<value>hadoop</value>
		<description>MySQL username</description>
	</property>

	<property>
		<name>javax.jdo.option.ConnectionPassword</name>
		<value>hadoop</value>
		<description>MySQL password</description>
	</property>

	<property>
		<name>javax.jdo.option.ConnectionDriverName</name>
		<value>com.mysql.cj.jdbc.Driver</value>
	</property>

	<!-- Metastore配置 -->
	<property>
		<name>hive.metastore.uris</name>
		<value>thrift://localhost:9083</value>
	</property>

	<property>
		<name>hive.metastore.schema.verification</name>
		<value>false</value>
	</property>

	<!-- 
		新版本（4.0）使用Beeline代替了传统的hive cli
		Beeline需要通过HiveServer2网络服务对Hive进行访问
		如果使用3.x的版本，可以直接在hive cli中输入SQL语句，hive cli并不依赖于HivesServer2
		（但是即使你使用的是3.x的版本，在实验后面把数据从Hive导入MySQL时
		仍然需要使用到HiveServer2）
		下面是hiveserver2的配置
	-->
	<property>
		<name>hive.server2.thrift.port</name>
		<value>10000</value>
	</property>

	<property>
		<name>hive.server2.thrift.bind.host</name>
		<value>127.0.0.1</value>
	</property>

	<!-- 这里为了方便，我们将HiveServer2设置为无需认证（也可以省略这个配置，默认值即为NONE）
	任何客户端可直接连接到HiveServer2
	安全的认证方式有简单用户名/密码认证、Kerberos 认证、LDAP 认证、PAM认证等
	如有需要，自行进行配置
	-->
	<property>
		<name>hive.server2.authentication</name>
		<value>NONE</value>
	</property>
</configuration>
```


配置Hive的环境变量`hive-env.sh`

```sh $HIVE_HOME/conf/hive-env.sh
export HADOOP_HOME=/usr/local/hadoop
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HIVE_CONF_DIR=/usr/local/hive/conf
export HIVE_AUX_JARS_PATH=/usr/local/hive/lib
```

运行下面的命令初始化MySQL元数据

```sh
schematool -dbType mysql -initSchema --verbose
```

### 配置Hadoop的代理用户

HiveServer2的使用需要用到Hadoop的代理用户，使用以下配置允许hadoop用户代理所有主机中和所有用户

```xml $HADOOP_HOME/etc/hadoop/core-site.xml
<property>
    <name>hadoop.proxyuser.hadoop.hosts</name>
    <value>*</value>
</property>
<property>
    <name>hadoop.proxyuser.hadoop.groups</name>
    <value>*</value>
</property>
```

> **注意**：这里的hadoop.proxyuser.xxx.hosts和hadoop.proxy.xxx.groups要把中间的xxx换成你的用户名[^4]，也就是允许xxx代理所有主机所有用户（折腾几天才发现hiveserver2启动不起来是这里的问题(≖_≖ )）

[^4]: [Proxy user - Superusers Acting On Behalf Of Other Users](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/Superusers.html)



### 启动并试用Hive

首先启动Hadoop集群

```sh
start-all.sh
```

启动Metastore（传统Hive Cli不需要HiveServer2）

```sh
hive --service metastore
```

启动Hive客户端并进行测试

```sh
hive
```

在里面可以输入SQL语句，如果要退出Hive交互式执行环境，可以输入如下命令：

```hive
hive> exit;
```

> （Beeline部分待完善）
> 如果使用的是4.0版本，Beeline代替了传统的hive cli，需要先启动HiveServer2再进入Beeline，大部分Hive的SQL语句也是可以用于Beeline的
> 
> 启动HiveServer2：
> 
> ```sh
> hive --service hiveserver2
> ```
> 
> Beeline连接HiveServer2。为了拥有访问hadoop目录的权限，把`username`改为你的系统用户名
> 
> > 如果不指定用户，使用的是匿名用户，没有权限访问/user/hive/warehouse，无法创建数据库
> 
> ```sh
> beeline -u jdbc:hive2://localhost:10000 -n username
> ```
> 
> 使用`!quit`或`!exit`退出Beeline

## Hive排错

### 无法执行Hive的SQL语句

运行hive命令如果报错：

```
FAILED: HiveException java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient
```

说明`Hive`在尝试连接到Hive MetaStore时失败了

可能是因为`Hive MetaStore`服务没有运行，在后台运行以下命令：

```sh
hive --service metastore &
```

然后再运行`hive`即可正常使用

如果还是不行，就重新初始化`hive`

尝试进入MySQL将hive数据库删除。由于已经授权给用户`hadoop`，直接登录`hadoop`对hive数据库重置即可，无需登录`root`用户进行删除并重新创建授权

```sql
mysql> drop database hive;
mysql> create database hive;
```

然后删除HDFS中的`/tmp`目录

```sh
hdfs dfs -rm -r /tmp
```

再重新升级元数据

```sh
schematool -initSchema -dbType mysql --verbose
```

### 格式化失败

如果在运行`schematool`命令进行初始化时报以下错误

```
Error: Table 'CTLGS' already exists (state=42S01,code=1050)
org.apache.hadoop.hive.metastore.HiveMetaException: Schema initialization FAILED! Metastore state would be inconsistent !!
Underlying cause: java.io.IOException : Schema script failed, errorcode 2
Use --verbose for detailed stacktrace.
*** schemaTool failed ***
```

可以尝试先进入`MySQL`，删除`hive`数据库再重建

```sql
DROP DATABASE hive;
CREATE DATABASE hive;
```
