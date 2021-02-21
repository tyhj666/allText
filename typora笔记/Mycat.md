一台机器远程访问另外一台机器的mysql数据库的命令:

​       mysql  -uroot -p123123 -h 192.168.140.127 -P 3306

# mycat的安装启动

1. ###### 安装

   - 1.解压后即可使用:解压文件拷贝到linux下 /usr/local/下面

   - 2.三个配置文件

     ​    schema.xml:定义逻辑库，表，分片节点等内容

     ```html
     <? xml version="1.0"?>
     <mycat:schema xmlns:mycat="http://io.mycat/">
         <!--schema标签代表定义逻辑库,dataNode="dn1"表示默认的数据库节点-->
         <schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1">     </schema>
         <dataNode name="dn1" dataHost="host1" database="testdb"/>
         <!--数据主机-->
         <dataHost name="host1" maxCon="1000" minCon="10" balance="0"
                   writeType="0" dbType="mysql" dbDriver="native" switchType="1"                     slaveThreshold="100">
               <!--heartbeat是心跳检测的标签-->
                <heartbeat>select user()</heartbeat> 
               <!--writeHost标签表示写主机-->
                <writeHost host="hostM1" url="192.168.1.128:3306" user="root" 
                           password="123456">
                    <readHOst host="hosts1" url="192.168.1.200:3306" user="root" 
                              password="xxx" />
             </writeHost>
         </dataHost>
     </mycat:schema>
     ```

     ​    rule.xml:定义分片规则

        server.xml：定义用户以及系统相关的变量，如果端口等

     ​      server.xml文件里面的user标签下的property标签

     
       ```html
     <user name="mycat">
          <property name="password">123456</property>
         <!--下面的schema表示的是mycat逻辑库，即mycat本身，应用程序连接的时候就连逻辑库，
             真正的数据库是配置在myCat里面的-->
          <property name="schema">TESTDB</property>
     </user>
            ```
     

2. ##### 启动

   - 控制台启动: 去mycat/bin 目录下执行 ./mycat console
   - 后台启动:去mycat/bin目录下 ./mycat start

      为了能第一时间看到启动日志，方便定位问题，我们选择控制台启动

3. #### 登录

   - 登录后台管理窗口，此登录方式用于管理维护Mycat

      mysql -umycat -p123456 -P 9006 -h 192.168.140.128

   - 登录数据窗口，此登录方式用于通过Mycat查询数据，我们选择这种方式询问Mycat

     mysql -umycat -p123456 -P 8066 -h 192.168.140.128

     show databases命令显示的是在配置文件中定义的逻辑库

     use TESTDB命令是切换到逻辑库，然后执行show tables就可以看到我们配置的真实的数据库和表

# 搭建Mycat读写分离

 在搭建Mycat的读写分离之前，要先搭建好mysql的主从复制，我们将搭建一主一从，双主双从两种读写分离模式

1. ### 一主一从

   - 搭建Mysql数据库主从复制

     redis主从复制原理，从机找主机，主机把内存中的数据写入到持久化文件（rdb）文件，并把持久化文件传给从机，然后从机用主机的rdb文件替换自己的rdb文件

     mysql主从复制的原理和redis主从复制的原理不一样，mysql将写的操作全部写入二进制文件Binarylog，从机读取主机的二进制文件，并写到Relay文件，存在延时性的问题，redis是从头开始复制，mysql是从接入点开始复制

   - 主机配置

     修改配置文件: vim /etc/my.cnf

     主服务器唯一ID:   server-id=1

     启用二进制日志:log-bin=mysql-bin

     设置不要复制的数据库（可以设置多个）:

     ​           binlog-ignore-db=mysql

     ​           binlog-ingore-db=information_schema     

     设置需要赋值的数据库: binlog-do-db=需要复制的主数据库名字，这个数据库的名字是之前没有的，等主从复制搭建好之后在在主机上面创建库，如果是主机之前有的库，从机读取到自己那会报错

     设置logbin格式: binlog_format=statement(大写)

     binlog日志格式有三种:statement,row,mixed

   - 从机配置

     修改配置文件: vim /etc/my.cnf

     从服务器唯一ID: server-id=2

     启用中继日志: relay-log=mysql-relay

   - 配置完之后，从启mysql服务: systemctl restart mysqld

      查看是否启动成功: ststemctl status mysqld

   - 关闭主从机防火墙

   - 在主机上面配置一个用户，赋予给从机复制的权限，即在主机mysql建立账户并授权slave

      grant replaceation slave on \*.\*  to  'salve' @'%' identified by '123123';

     查询master的状态:show master status;并记录下File和Position的值，执行完此步骤之后不要再操作主服务器mysql,防止主服务器状态值变化

   - 在从机上配置需要复制的主机

     复制主机的命令: create master to master_host='主机的IP地址',

     ​                             master_user='slave',

     ​                             master_password='123123',

     ​                             master_log_file='mysql-bin.具体数字',master_log_pos=具体值;

     启动从服务器的复制功能: start slave;

     查看从服务器的状态: show slave status\G;

   - 如何停止从服务复制功能:stop slave

   - 如何重新配置主从:  stop slave;   reset  master;

   - 启动MyCat

   - 登录Mycat：  mysql -umycat -p123456 -h 192.168.140.128 -P 8066

   - 切换逻辑库:use  TESTDB

   - 查看表信息:select  \* from mytbl

   - 验证在schema.xml配置的读写库是否正确

      1.在写主机插入: insert into mytbl values(1,@@host);主从主机数据不一致了

      2.在mycat里面查询:select \* from mytbl;

   - 验证发现并没有实现读写分离，是因为schema.xml配置文件配置的有问题，修改该配置文件如下

      ```html
      修改<dataHost>标签的balance属性，通过此属性配置读写分离的类型
          负载均衡类型，目录的取值有4种
          1.balance="0",不开启读写分离机制，所有读写操作都发送到当前可用的writeHost上
          2.balance="1",全部的readHost与stand by writeHost参与select语句的负载均衡，简单的说，当       双主双从模式(M1->s1,M2->s2,并且M1与M2互为主备)，正常情况下，M2，S1,S2都参与select语句
            的负载均衡
          3.balance="2",所有读操作都随机的在writeHost,readHOst上分发
          4.balance="3",所有读请求随机的分发到readhost执行，writeHost不负担读压力
      ```

2. ### 双主双从

   一个主机m1用于处理所有请求，它的从机s1和另一台主机m2还有它的从机s2负责所有读请求。当m1主机宕机后，m2主机负责写请求，m1,m2互为备机，架构图如下

   - ![image-20210221111300017](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210221111300017.png)

   - ![image-20210221112044546](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210221112044546.png)

   - 双主配置

     ```html
     <h2> Master1配置  </h2>
     
     修改配置文件:vim /etc/my.cnf
     #主服务器唯一ID
       server-id=1
     #启用二进制日志
       log-bin=mysql-bin
     #设置不要复制的数据库（可设置多个）
        binlog-do-db=需要复制的主数据库名字
     #设置logbin格式
        binlog_format=statement(大写)
     #在作为从数据库的时候，有写入操作也要更新二进制日志文件
        log-slave-updates
     #表示自增长字段每次递增的数量，指自增字段的起始值，其默认值是1，取值范围是1...65535
        auto-increment-increment=2
     #表示自增长字段从哪个数字开始，指字段一次递增多少，它的取值范围是1...65535
       auto-increment-offset=1
     
     ```

     ```htm
     Master2 配置
     
     修改配置文件:vim /etc/my.cnf
     #主服务器唯一ID
       server-id=3
     #启用二进制日志
       log-bin=mysql-bin
     #设置不要复制的数据库（可设置多个）
        binlog-ignore-db=mysql
        binlog-ignore-db=information_schema
     #设置需要复制的数据库
        binlog-do-db=需要复制的主数据库名字
     #设置logbin格式
        binlog_format=statement(大写)
     #在作为从数据库的时候，有写入操作也要更新二进制日志文件
        log-slave-updates
     #表示自增长字段每次递增的数量，指自增字段的起始值，其默认值是1，取值范围是1...65535
        auto-increment-increment=2
     #表示自增长字段从哪个数字开始，指字段一次递增多少，它的取值范围是1...65535
       auto-increment-offset=2
     ```

   - 双从配置

     ```html
     Slave1配置
     修改配置文件: vim /etc/my.cnf
     #从服务器唯一ID
     server-id=2
     #启用中继日志
     relay-log=mysql-relay
     ```

     ```html
     Slave2配置
     Slave1配置
     修改配置文件: vim /etc/my.cnf
     #从服务器唯一ID
     server-id=4
     #启用中继日志
     relay-log=mysql-relay
     
     ```

   - 4台机器配置好之后，分别重启mysql服务并查看服务状态

     ```html
     systemctl restart mysqld
     systemctl status mysqld
     ```

   - 主机从机都关闭防火墙

   - 在两台主机上建立账户并授权 slave

     ```html
     #在主机mysql里面执行授权命令
     grant replication slave on *.* to 'slave'@'%'identified by '123123';
     #查询Master1的状态
     show master status;
     ```

   - 从服务读取主服务器写的二进制文件

     ```html
     create master to master_host='主机的IP地址',
                               master_user='slave',
                               master_password='123123',
                               master_log_file='mysql-bin.具体数字',master_log_pos=具体值;
     ```

   - 两个主机相互复制

     ```html
     Master2复制Master1,Master1复制Master2
     Master2的复制命令
        change master to master_host='192.168.140.128',
                         master_user='slave',
                         master_password='123123'
                         master_log_file='mysql-bin.00008',master_log_pos=154;
     
     Master1的复制命令
        change master to master_host='192.168.140.126',
                         master_user='slave',
                         master_password='123123'
                         master_log_file='mysql-bin.00008',master_log_pos=154;
     ```

   - 用Mycata实现双主双从的读写分离，在schema.xml里面配置

     ```html
     <? xml version="1.0"?>
     <mycat:schema xmlns:mycat="http://io.mycat/">
         <!--schema标签代表定义逻辑库,dataNode="dn1"表示默认的数据库节点-->
         <schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1">     </schema>
         <dataNode name="dn1" dataHost="host1" database="testdb"/>
         <!--数据主机-->
         <dataHost name="host1" maxCon="1000" minCon="10" balance="1"
                   writeType="0" dbType="mysql" dbDriver="native" switchType="1"                     slaveThreshold="100">
               <!--heartbeat是心跳检测的标签-->
                <heartbeat>select user()</heartbeat> 
               <!--writeHost标签表示写主机-->
                <writeHost host="hostM1" url="192.168.1.128:3306" user="root" 
                           password="123456">
                    <readHOst host="hosts1" url="192.168.140.127:3306" user="root" 
                              password="123123" />
             </writeHost>
              <writeHost host="hostM2" url="192.168.140.126:3306" user="root" 
                           password="123456">
                    <readHOst host="hosts2" url="192.168.140.125:3306" user="root" 
                              password="123123" />
             </writeHost>
         </dataHost>
     </mycat:schema>
     
     #balance="1"：全部的readHost域stand by writeHost参与select 语句的负载均衡
     #writeType="0"：所有写操作发送到配置的第一个writeHost，第一个挂了切到还生存的第二个
     #writeType="1":所有写操作都随机发送到配置的writeHost，1.5以后废弃不推荐
     #writeHost,重新启动后，以切换后的为准，切换记录在配置文件中:dnindex.properties
     #switchType="1": 
           1,表示默认值，随机切换   
           -1表示不自动切换 
           2基于Mysql主从同步的状态决定是否切换
     ```

   - 启动Mycat

   - 验证读写分离

     ```html
     #在写主机Master1数据库表中插入带系统变量数据，造成主从数据不一致
     insert into mytabl values(2,@@hostname)
     ```



# Mycat实现数据库的垂直拆分------分库

一个数据库有很多表的构成，每个表对应着不同的业务，垂直切分是指按照业务将表进行分类，分布到不同的数据库上面，这样也就将数据或者压力分但到不同的库上面，如下图

分库的原则:有紧密关联关系的表应该在一个库里，相互没有关联关系的表可以分到不同的库里

1. 修改schema配置文件

   - ```html
     <? xml version="1.0"?>
     <mycat:schema xmlns:mycat="http://io.mycat/">
         <!--schema标签代表定义逻辑库,dataNode="dn1"表示默认的数据库节点-->
         <schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1"> 
             <table name="customer" dateNode="dn2"></table>
         </schema>
         <dataNode name="dn1" dataHost="host1" database="orders"/>
         <dataNode name="dn2" dataHost="host2" database="orders"
         <!--数据主机-->
         <!--垂直拆分不需要实现读写分离，所以balance属性的值要改成0-->
         <dataHost name="host1" maxCon="1000" minCon="10" balance="0"
                   writeType="0" dbType="mysql" dbDriver="native" switchType="1"                     slaveThreshold="100">
               <!--heartbeat是心跳检测的标签-->
                <heartbeat>select user()</heartbeat> 
               <!--writeHost标签表示写主机-->
               <!--垂直拆分不需要配置读的主机，所以要去掉<readHost>的标签-->
                <writeHost host="hostM1" url="192.168.1.128:3306" user="root" 
                           password="123456">
                </writeHost>
         </dataHost>
         <dataHost name="host2" maxCon="1000" minCon="10" balance="0"
                   writeType="0" dbType="mysql" dbDriver="native" switchType="1"                     slaveThreshold="100">
               <!--heartbeat是心跳检测的标签-->
                <heartbeat>select user()</heartbeat> 
               <!--writeHost标签表示写主机-->
               <!--垂直拆分不需要配置读的主机，所以要去掉<readHost>的标签-->
                <writeHost host="hostM2" url="192.168.140.127:3306" user="root" 
                           password="123456">
                </writeHost>
         </dataHost>
     </mycat:schema>
     ```

   - 新增两个空白库

     分库操作不是在原来的老数据库上面进行操作，需要准备两台机器分别安装新的数据库

     ```html
     #在数据节点dn1,dn2上分别创建数据库orders
     create database orders
     ```

   - 启动Mycat

   - 登录Mycat

     ```sql
     mysql -umycat -p123456 -h 192.168.140.128 -P 8066
     use TESTDB
     create table customer (
     id int auto_increment,
     name varchar2(200),
         primary key(id)
     );
     
     create table orders (
         id int auto_increment,
         order_type int,
         coustomer_id,int,
         amount declmal(10,2)
         primary key(id)
     );
     
     create table orders_detail (
        id int atuo_increment,
         detail varchar(2000),
         order_id int,
         primary key(id)
     );
     
     create table dict_order_type(
         id int auto_increment,
         order_type varchar(200),
         primary key(id)
     )
     
     
     ```

# 2.Mycat实现数据库的水平拆分------分表

​    		相对于垂直拆分，水平拆分不是将表做分类，而是安装某个字段的某种规则来分散到多个库中，每个表中，包含一部分数据，简单来说，我们可以将数据的水平切分理解为时按照数据行的切分，就是将表中的某些行切分到一个数据库，而另外的某些行又切分到其他数据库中，如图:

1. 配置分表

   - 选择要拆分的表

     MySql单表存储数据条数是有瓶颈的，单表达到1000万条数据就达到了瓶颈，会影响查询效率，需要进行水平拆分(分表)进行优化

2. 修改配置文件schema.xml，在垂直拆分的配置文件的基础上面，在<schema>标签中添加如下

   - ```html
     #为order表设置数据节点为dn1,dn2,并指定分片规则为mod_rule(自定义的名字)
     <table name="orders" dataNode="dn1,dn2" rule="mod_rule"> </table>
     配置如下:
       <schema name="TESTDB" checkSOLschema="false" sqlMaxLimit="100" dataNode="dn1">
             <!--数据库垂直拆分的配置-->
             <table name="customer" dataNode="dn2"></table>
           <!--表示将orders这张表分成dn1,dn2两片，rule是定义的分片规则-->
           <table name="orders" dataNode="dn1,dn2" rule="mod_rule"></table>
       </schema>
     ```

3. 配置rule.xml

   - ```html
     #在rule配置文件里新增分片规则，并指定规则使用字段为customer_id
     #还有选择分片算法mod-long(对字段求模运算),customer_id对两个节点求模，根据结果分片
     #配置算法mod-log参数count为2，两个节点
     <tableRule name="mod_rule">
         <rule>
             <colomns>customer_id</colomns>
             <algorithm>mod-long</algorithm>
         </rule>
         <function name="mod-long" class="io.myCat.route.function.PartitionByMod">
             <!--设置节点数-->
              <property name="count">2</property>
         </function>
     </tableRule>
     ```

   - 在dn2上面创建orders表

   - 重启Mycat,让配置生效

   - 访问Mycat，实现分片

     ```html
     #在mycat里向orders表插入数据，insert字段不能省略,不然会报错
     insert into orders(id,order,type,customer_id,amount) values(1,101,100,100100);
     ```

   - 通过查询语句查询插入的语句，发现顺序不是按id顺序排序的，是因为mycat将查询语句分发给两个数据库节点，然后mycat再进行分片分析，并将结果合并返回



### 2.1Mycat的分片join(分表后的订单表如何跟订单详情表关联查询)

我们要对orders_detail也要进行分片操作，join的原理如下图:

ER表:Mycat 借鉴了NewSQL领域的新秀Foundation DB的设计思路，Foundation DB创新性的提出了Table Gourp的概念，其将子表的存储位置依赖于主表，并且物理上紧邻存放，因此彻底解决了JION的效率和性能问题，根据这一思路，提出了基于E-R关系的数据分片策略，子表的记录与所关联的父表记录存放在同一个数据分片上。

1. 修改chema.xml配置文件

   

```html
#修改schema.xml配置文件
<table name="orders" dataNode="dn1,dn2" rule="mod_rule">
    <childTable name="orders_detail" primaryKey="id" joinKey="order_id" parentKey="id" />
</table>
```

```html
<? xml version="1.0"?>
<mycat:schema xmlns:mycat="http://io.mycat/">
    <!--schema标签代表定义逻辑库,dataNode="dn1"表示默认的数据库节点-->
    <schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1"> 
        <table name="customer" dateNode="dn2"></table>
        <table name="orders" dataNode="dn1,dn2" rule="mod_rule">
    	      <childTable name="orders_detail" primaryKey="id" joinKey="order_id"    
                          parentKey="id" />
		</table>
    </schema>
    <dataNode name="dn1" dataHost="host1" database="orders"/>
    <dataNode name="dn2" dataHost="host2" database="orders"
    <!--数据主机-->
    <!--垂直拆分不需要实现读写分离，所以balance属性的值要改成0-->
    <dataHost name="host1" maxCon="1000" minCon="10" balance="0"
              writeType="0" dbType="mysql" dbDriver="native" switchType="1"                     slaveThreshold="100">
          <!--heartbeat是心跳检测的标签-->
           <heartbeat>select user()</heartbeat> 
          <!--writeHost标签表示写主机-->
          <!--垂直拆分不需要配置读的主机，所以要去掉<readHost>的标签-->
           <writeHost host="hostM1" url="192.168.1.128:3306" user="root" 
                      password="123456">
           </writeHost>
    </dataHost>
    <dataHost name="host2" maxCon="1000" minCon="10" balance="0"
              writeType="0" dbType="mysql" dbDriver="native" switchType="1"                     slaveThreshold="100">
          <!--heartbeat是心跳检测的标签-->
           <heartbeat>select user()</heartbeat> 
          <!--writeHost标签表示写主机-->
          <!--垂直拆分不需要配置读的主机，所以要去掉<readHost>的标签-->
           <writeHost host="hostM2" url="192.168.140.127:3306" user="root" 
                      password="123456">
           </writeHost>
    </dataHost>
</mycat:schema>
```

2. 在dn2创建orders_detail表

3. 重启Mycat

4. 访问Mycat向orders_detail表插入数据

   - ```mysql
     insert into orders_detail (id,detail,order_id)values(1,'detail',1);
     ```

5. 在mycat,dn1,dn2中运行两个表join语句

   - ```mysql
     select o.*,od.detail from orders o inner join orders_detail od on o.id=od.order_id;
     ```



## 2.2全局表

​	在分片的情况下，当业务表因为规模而进行分片以后，，业务表与这些附属的字典表之间的关联，就成了比较棘手的问题，考虑到字典表具有如下几个特性

    1. 变动不频繁   2.数据量总体变化不大    3.数据规模不大，很少有超过数十万条记录

鉴于此，Mycat定义了一个特殊的表，称之为全局表，全局表具有如下特性

​    1.全局表的插入更新操作会实时在所有节点上执行，保持各个分片的数据一致性

​    2.全局表的查询操作，只从一个节点获取

​    3.全局表可以跟任何一个表进行join操作

修改schema.xml配置文件

```html
<table name="orders" dataNode="dn1,dn2" rule="mod_rule">
    <childTable name="orders_detail" primaryKey="id" joinKey="order_id" parentKey="id"/>
</table>
<table name="drct+order_type" dataNode="dn1,dn2" type="global"></table>
```

启动mycat,在mycat中插入数据

## 2.3常用的分片规则

1. 取模:此规则为对分片字段求模运算，也就是水平规则最常用规则。

2. 分片枚举

   通过在配置文件中配置可能的枚举id，自己配置分片，本规则适用于特定的场景，比如有些业务需要按照省份或区县来做保存，而全国省份区县固定的，这类业务使用本条规则

   ```html
   #修改schemal.xml配置文件
   <table name="orders_ware_info" dataNode="dn1,dn2",rule="sharding_by_intfile"></table>
   ```

   ```html
   #修改rule.xml配置文件
   <tableRule name="sharding_by_intfile">
      <rule>
         <columns>areacode</columns>
         <algorithm>hash-int</algorithm>
      </rule>
   </tableRule>
   <function name="hash-int"
             class="io.mycat.route.function.PartionByFileMap">
       <property name="mapFile">partion-hash-int.txt</property>
       <property name="type">1</property>
       <property name="defaultNode">0</property>
   </function>
   
   #columns:分片字段，algorithm：分片函数
   #mapFile:标识配置文件名称，
   #type:0为int型，非0为String
   #defaultNode:默认节点:小于0表示不设置默认节点   大于0表示设置默认节点
   #设置默认节点，如果碰到不识别的枚举值，就让它路由到默认节点，如不设置不识别就报错
   ```

   ```html
   #修改partion-hash-int.txt配置文件
   110=0
   120=1
   ```

   2.1 重启Mycat

   2.2 访问Mycat创建表

   ```sql
   #订单归属区域信息表
   create table orders_ware_info(
       'id' intauto_incremnt comment '编号',
       'order_id' int comment '订单编号',
       'address' varchar(200) comment '地址',
       'areacode' varchar(20) comment '区域编号',
       primary key(id)
   );
   
   #插入数据
   insert into orders_ware_info(id,order_id,address,areacode) values(1,1,"北京",'110');
   ```

3. 范围约定:此分片适用于，提取规划好分片字段某个范围数据哪个分片

   - ```html
     #修改schemal.xml配置文件
     <table name="payment_info" dataNode="dn1,dn2",rule="aout_sharding_long"></table>
     ```

   - ```html
     #修改rule.xml配置文件
     <tableRule name="aout_sharding_long">
        <rule>
           <columns>order_id</columns>
           <algorithm>rang-long</algorithm>
        </rule>
     </tableRule>
     <function name="rang-long"
               class="io.mycat.route.function.AutoPartionByLong">
         <property name="mapFile">autopartion-long.txt></property>
         <property name="defaultNode">0</property>
     </function>
     
     #columns:分片字段，algorithm：分片函数
     #mapFile:标识配置文件名称
     #defaultNode:默认节点:小于0表示不设置默认节点   大于0表示设置默认节点
     #设置默认节点，如果碰到不识别的枚举值，就让它路由到默认节点，如不设置不识别就报错
     ```

   - 修改autopartion-long.txt配置文件

     ```html
     0-102=0
     103-200=1
     ```

   - 重启Mycat

   - 访问Mycat，创建表

     ```sql
     use TESTDB
     create table payment_info(
        'id' int_autoincpemnt comment '编号',
         'order_id' int comment '订单编号',
         'payment_status' int comment '支付状态',
         primary key(id)
      );
      
      #插入数据
      insert into payment_info(id,order_id,payment_status) values(1,101,0);
      insert into payment_info(id,order_id,payment_status) values(2,102,1);
      insert into payment_info(id,order_id,payment_status) values(3,103,0);
      insert into payment_info(id,order_id,payment_status) values(4,104,01);
     ```

     







​                



