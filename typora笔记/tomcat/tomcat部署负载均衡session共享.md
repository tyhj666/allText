# tomcat用Nginx部署负载均衡时各服务器session共享的配置方法

#### 一. ip_hash策略

#### 二. session赋值的方法，这种方法限制于tomcat服务器少的情况

session同步的配置如下:

  1.在tomcat的conf/server.xml配置如下:

       ```html
<Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"/>
       ```

 2.在tomcat部署的应用程序webapps/项目名称/WEB-INF/web.xml中加入如下配置

```html
<distributable/>
```

3.重启tomcat服务器

#### 三.sso-单点登录

sso的定义是在多个应用系统中，用户只需要登录一次就可以访问所有相互信任的应用系统，也就是用来解决集群环境Session共享的方案之一