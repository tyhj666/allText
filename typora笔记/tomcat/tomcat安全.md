# Tomcat安全

### 一. 配置安全

   1. 删除webapps目录下的所有文件,禁用tomcat管理界面

   2. 注释或删除tomcat-users.xml文件内的所有用户权限

   3. 更改关闭tomcat指令或禁用:

      tomcat的server.xml中定义了可以直接关闭Tomcat实例的管理端口(默认8005)，可以通过talent连接上该端口之后，输入SHUTDOWN(此为默认关闭指令)即可关闭tomcat实例(注意，此时虽然实例关闭了，但是进程还是存在的)，由于默认关闭tomcat的端口和指令都很简单，默认端口为8005，指令为SHUTDOWN

      方案一:

      ```html
      更改端口号和指令:
      <server port="8456" shutdown="itcast_shut"></server>
      telnet 127.0.0.1 8005 表示远程连接上8005这个端口，并执行shutdown命令
      ```

      方案二:

      ```html
      禁用8005端口
      <Server port="-1" shutdown="SHUTDOWN"></Server>
      ```

      ### 二.配置错误页面，不要把错误信息抛给页面

      ### 三.应用安全

      ​       在大部分的web应用中，特别是一些后台应用系统，都会实现自己的安全管理模块(权限模块)，用于控制应用系统的安全访问，基本包含两个部分认证(登录/单点登录)和授权(功能权限，数据加权限)两个部分，对于当前的业务系统，可以自己做一套适用于自己业务系统的权限模块，也有很多的应用系统直接使用一些功能完善的安全框架，将其集成到我们的web应用中，如:SpringSecurity,Apache shiro等

      ### 四.传输安全

      ​        HTTPS的全称是超文本传输安全协议，是一种网络安全传输协议，在HTTP的基础上加入SSL/TLS来进行数据加密，保护交换数据不被泄漏，窃取。

      ​     SSL和TLS是用于网络通信安全的加密协议，它允许客户端和服务器之间通过安全连接通信。SSL协议的三个特性:

      1. 保密:通过SSL链接传输的数据是加密的

                2. 鉴别:通信双方的省份鉴别，通常是可选的，至少有一方需要验证
                   3. 完整性:传输数据的完整性检查

​        

HTTPS和HTTP的区别主要为以下四点:

       1. HTTPS协议需要到证书颁发机构CA申请SSL证书，然后与域名进行绑定，HTTP不用申请证书
          2. HTTP是指超文本传输协议，属于应用层信息传输,HTTPS则是具有SSL加密安全性的传输协议，对数据的传输进行加密，相当于HTTP的升级版
          3. HTTP和HTTPS使用的是完全不同的连接方式，用的端口也不一样，前者是8080，后者是8443
          4. HTTP的连接很简单，是无状态的，HTTS协议是由SSL+HTTP协议构建的可进行加密传输，省份认证的网络协议，比HTTP协议安全

### Tomcat支持HTTPS

   1. 生成密钥库文件,在tomcat的安装目录下执行

      ```html
      keytool -genkey -alias tomcat -keyalg RSA -keystore tomcatkey.keystore
      执行完这个命令之后，输入的第一个密码是密钥库的密码，第二个的密室是密钥的密码
      输入对应的密钥库密码，密钥密码等信息之后，会在当前文件夹中出现一个密钥库文件:tomcatkey.keystore
      ```

        2. 将密钥库文件tomcatkey.keystore复制到tomcat/conf目录下

        3. 配置tocmcat/conf/server.xml

           ```html
           <Connector port="8443" protocol="org.apache.coyote.http11.Http11NioPortocol"
                      maxThreads="150" schema="https" secure="true" SSLEnabled="true">
                    <SSLHostConfig certificateVerification="false">
                          <Certificate certificateKeystoreFile="..../tomcatkey.keystore" certificateKeystorePassword="itcast" type="RSA"/>
                    </SSLHostConfig>
           </Connector>
           ```





# ApatchBence进行apache性能测试

apacheBence(ab)是一款apacheServer基准的测试工具，用户测试apache server的服务能力(每秒处理请求数)，它不仅可以用户apache的测试，还可以用于测试tomcat,lighthttp,IIs等服务器

  1.安装

     ```html
yum  install httpd-tools
     ```

2. 查看版本

   ```html
   ab -v
   ```

3. 在linux服务器上面用tomcat部署应用

4. 测试性能

   ```html
   ab -n 1000 -c 100 -p data.json -T application/json http://localhost:9000/course/search.do?page=1&pageSize=10
   
   -n  在测试会话中所执行的请求格式，默认只执行一次请求
   -c  一次产生的请求个数，默认一次一个
   -p  包含了需要Post的数据文件，data.json是json格式的请求参数
   -t 测试所进行的最大秒数，默认没有时间限制
   -T  POST数据所需要使用的Context-Type头信息
   ```

   