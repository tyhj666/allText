# mysql

1. 在安装路径的my.ini文件里面可以修改端口，存储引擎，字符集等，修改完之后需要重新启动mysql服务才可以生效

##### 如何启动mysql的服务

1. 以管理员省份运行cmd 

   ```shell
   net start mysql(服务名)
   
   #登录客户端
   mysql -h localhost -P 3306 -p +回车再输入密码
   或
   mysql -h localhost -P 3306 -u root -p密码(-p和密码不能有空格)
   
   #退出客户端
    exit
    
   #显示都有哪些数据库
   show databases
   test库是测试库，可以随便删除
   information_schema,mysql,performance_schema这三个库不能随便动
   
   #切换库
   use test;
   #在test库里面查看mysql库里面有多少张表
   shows tables from mysql
   #查看我当前在哪个库里面操作
   select database();
   #查看表结构
   desc 表名
   #查看当前mysql的版本
      1. select version();
      2. 在cmd命令行里面看
         mysql -version 或 mysql -v
   ```

##### mysql的语法规范

1. 不区分大小写，但是建议关键字大写，表名，列名小写

2. 每条命令最好用分号结尾

3. 每条命令根据需要，可以进行缩进或换行

4. 注释

   单行注释:#注释文字

   单行注释:-- 注释文字

   多行注释:/\*注释文字*/

5. 在命令行里面导入sql脚本

   source  sql脚本的路径

##### 基础查询

```mysql
/*
   语法:select 查询列表 from 表名;
   特点:
     1.查询列表可以是:表中的字段，常量值，表达式，函数
     2.查询的结果是一个虚拟的表格
*/
#查询常量值
select 100;
select 'john';

#查询表达式
select 100*99;

#查询函数
select version();

#起别名
方式一:select 1000*99 as 结果;
方式二: select last_name 名 from employees;
#下面的别名out put会报错，因为有特殊字符在中间，需要用双引号或单引号引起来
select salary as out put from employees


#+号的作用
仅仅只有一个功能:运算符
select  '123' + 90;其中一方为字符型，视图将字符型数值转换成数值型，如果转换成功，则继续做加法运算
select 'john' +90;如果转换失败了，则字符型数值转换为0
select null + 0;只要其中一方为null，则结果肯定为null

#多个字段连接
select concat('a','b','c') as 结果 
null和任何的字段拼接结果都是null

select ifnull(commission_pct,0) as 奖金率

```

##### 条件查询

```shell
语法:
   select 查询列表
    from  表名
    where 
       筛选条件;
    分类:
       1.条件表达式筛选
           > < = !=(不等于) <>(不等于) >= <=
       2.逻辑表达式
           &&  ||  !
       3.模糊查询
          like：
          %任意多个字符，包含0个
          
          查询员工名中第三个字符为e,第5个字符为a的员工名和工资
          select lasst_name,salary from employees where last_name like '__e_a%'
          查询员工名中第二个字符为_的员工名,这个地方\相当于转义符
          select last_name from employees where last_name like '_\_%';
          或者,'_$_%' escape(大写) '$'告诉系统$是转义字符
          select last_name from employees where last_name like '_$_%' escape(大写) '$';
          _:
          
          
          between and:
          select * from employees where employee_id>= 100 and employee_id <= 120;
          等价于
          select * from employees where employee_id between 100 and 120;
          注意:between和and连接的两个数字不能换位置
          
          
          in
         select last_name,job_id from employees where job_id='' or job_id='' or job_id=''
          等价于
         select last_name,job_id from employees where job_id in ('','','');
        
            
          
          is null
          where commision_pct = null，这种写法不能判断为null的值
          where commision_pct is null，这种可以判断为null的值
          
          安全等于:<=>
          select * from employees where commisssion_pct <=> null;
          select * from employees where salary <=>12000
          比较:is null只可以判断null值，<=>即可以判断null值，也可以判断普通数值
          
          is not null
          
          select 
            last_name,depatement_id,salary*10*(1+ifnull(commission_pct,0)) as 年薪 
            from employee;
          
```

