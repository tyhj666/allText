# 写各自的实现

# 消费者的注册

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



# 消费者dubbo的配置

```xml
<beans>
    
    <context:component-scan base-package="..impl"></context:component-scan>
    
    <dubbo:application name="order-service-consumer"></dubbo:application>
    <dubbo:registry address="zookeeper://127.0.0.1:2181"></dubbo:registry>
    
    <!--
       声明需要调用的远程服务的接口，用来生成远程服务代理
       下面的这个配置是我们调用的远程服务提供这暴露的接口的配置
       这样配置结束之后我在我们的项目中就可以使用远程提供的接口了
     -->
    <dubbo:reference interface="com.atguigu.gmall.service.SerSerice" id="uSerService"></dubbo:reference>
    
    
</beans>
```

