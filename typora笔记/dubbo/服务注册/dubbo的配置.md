# 1. 将服务提供者注册到注册中心

​        1.导入dubbo依赖（2.6.2版本）

            ```xml
#在maven reporisty里面去找dubbo的版本
<dependency>
    <groupId>com.alibaba</groupId>
    <artfactId>dubbo</artfactId>
    <version>2.6.2</version>
</dependency>
<!--注册中心使用的是zookeeper,引入操作zookeeper的客户端-->
<!--2.6以前用的版本-->
<dependency>
    <groupId>com.101tec</groupId>
    <artfactId>zkclient</artfactId>
    <version>0.10</version>
</dependency>
<!--2.6以后用的版本-->
<dependency>
    <groupId>org.apache.curator</groupId>
    <artfactId>curator-framework</artfactId>
    <version>2.12.0</version>
</dependency>
            ```

2.配置服务提供者

```xml
<beans>
  <!--加入dubbo的名称空间，这样配置的时候就会有提示-->
    <!--1.指定当前服务/应用的名称(同样的服务名字系统，不要和别的服务同名)-->
   <dubbo:application name="user-service-provider"></dubbo:application>
    
    <!--2.指定注册中心的位置-->
    <dubbo:registry address="zookeeper://127.0.0.1:2181"></dubbo:registry>
    或者是
    <dubbo:registry protocol="zookeeper" address="27.0.0.1:2181"></dubbo:registry>
    
    <!--
        说明:完成1，2两个步骤后，当我们把服务器启动之后，就会将服务注册到注册中心
    -->
    
    <!--3.指定通信规则以及通信使用的协议和端口,下面的配置意识是使用dubbo协议用20080端口协议通信-->
    <dubbo:protocol name="dubbo" port="20080"></dubbo:protocol>
    
    <!--4.暴露服务-->
    <dubbo:service interface="接口的全类名" ref="接口真正的实现类"></dubbo:service>
    
    <bean id="接口真正的实现" class="实现类的全类名"></bean>
</beans>
```



​            

# 2.让服务消费者去注册中心订阅服务器提供者的服务地址



