# Restful相关知识

## java中常见的restful开发框架

1. jersey
2. play
3. SpringMvc

```java
//1.设计restful接口步骤
/**
   1.确定资源
   2.确定请求方式
   3.确定返回结果（类型，请求头，状态码）
*/
@Controller
public class EmployeeController{
    /**获取所有员工的信息
   1.确定资源
   2.确定请求方式
   3.确定返回结果（类型，请求头，状态码）
*/
    @RequestMapping(value="employees",method=RequestMethod.GET)
    @GetMapping("employees")//这句更上面的是同样的功能，两个注解不能同时存在
    @ResponseBody
    public List<Employee>list(){
        ArrayList<Employee> list = new ArrayList<>();
        list.add(new Employee(1,admin));
        list.add(new Employee(2,wang));
        return list;
    }
    
    
    /** 获取某个员工的信息
   1.确定资源   /employees/{name}使用路径占位符
   2.确定请求方式  get
   3.确定返回结果（类型，请求头，状态码） 员工对象，get请求返回的状态码是201
   */
    @GetMapping("employee/{id}")
    //下面的参数必须和上面路径的参数名称一致，如果参数的名称和路径的名称不一致，需要在@PathVar({"id"})，这个地方的id是路径里面对应的id名称
    public Employee getById(@PathVarlable Long id){//这个地方要用注解才可以获取到路径的可变参数，如果不用注解，将获取不到可变参数，不写注解能获取到?id=2这样的参数
        return new Employee(id,"admin");
    }
    
    
    /**删除一个员工
   1.确定资源 /employees/{id}
   2.确定请求方式 delete  返回的状态码是204
   3.确定返回结果（类型，请求头，状态码）
*/
     @DeleteMapping("employees/{id}")
     @ResponseBody
    public void deleteById(@PathVariable Long id){
        System.out.println("删除员工"+id)
    }
    
    
    /**获取某个员工某个月的薪资记录
   1.确定资源 /employees/{employeeId}/salaryies/{month}
   2.确定请求方式 get
   3.确定返回结果（类型，请求头，状态码）
*/
     @Getmapping{"employees/{employeeId}/salaries/{month}"}
    @ResponseBody
    public Salary getSalaryByEmployee(@PathVariable Long employeeId,@PathVariable 
         //下面@DateTimeFormat注解的作用是前台传日期参数到后台接收时使用的注解
         //后台返回的日期格式方式是:在字段对应的实体类上面添加@JsonFormat(pattern="yyyy-MM",timezone="GMT+8")
                                      @DateFormat(pattern = "yyyy-MM") Date month){
        return new Salary(id,employeeId,BigDecimal.TEN,month);
    }
    
    /***消费的数据类型，客户端浏览器访问服务器端
    下面的consumes="text/xml"相当于headers="content-type="text/xml"
    */
    @RequestMapping(value="consumes",consumes="text/xml")
    public void consumes(){
        System.out.println("consumes方法...xml")
    }
    
    /**
      生产的数据类型，服务器返回给客户端
    */
     @RequestMapping(value="produces",produces="text/xml")
    public void produces(){
        System.out.println("produces方法...xml")
    }
    
    /**
    @RequestBody表示前台请求的数据格式是JSON格式，后台接收的数据类型可以是字符类型也可以是对象类型
    **/
    @RequestMapping("employee")
    public void test1(@RequestBody Employee employee){
        System.out.print(".........");
    }
    
    
    
   /**
     当用ajax发送put请求的时候，请求的参数在后台不会处理，需要配置过滤器才会处理并接收到
   **/
    
    <servlet>
        <servlet-name>SpringMVC</servlet-name>
        <servlet-class>org.springframework.wseb.servlet.DispatcherServlet</servlet-class>
        <init-param>
              <parame-name>contextConfigLocation</param-name>
              <param-values>classpth:mvc.xml</param-values>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <param-name>springmvc</param-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    <filter>
        <filter-name>httpPutFormContentFilter</filter-name>
        <filter-class>org.springframework.web.filter.HttpPutFormContentFilter</filter-classs>
    </filter>
    
    <filter-mapping>
         <filter-name>httpPutFormContentFilter</filter-name>    
         <servlet-name>springMVC</servlet-name>
    </filter-mapping>
    
}

```





# webservices相关学习

webservice即web服务，java中共有三种webservice规范

- JAX_WS(JAX-RPC)

   JAX-WS(Java API For xML -WebService),JDK1.6自带的版本，底层支持为JAXB,JAX-WS(JSR224)规范的API位于java.xml.ws.*，包中，其中大多数都是注解，提供API操作web服务（通常在客户端用的比较多，由于客户端可以借助SDK生成，因此这个包中的API我们较少会直接使用）

- JAXM&SAAJ

     JAXM(JAVA API For XML Message)主要定义了包含了发送和接收消息所需要的API，当对于web服务器的服务端，其中API 位于javaxmmessaging.*包，它是JAVA EE的可选包，因此你要单独下载

- JAX-RS

   JAX-RS是JAVA针对REST(RepresentationState Trancation)分格制定的一套web服务器规范

webservices三种要素:soap,wsdl,uddi

1. ### SOAP协议

   ​       (1). SOAP即简单对象访问协议（Simple Object Access Protocol）,它是用于XML编码信息的轻量级协议，它有三个主要方面，XML-envelope为描述信息内容和如何处理内容定义了框架，将程序对象编码成为XML对象的规则执行远程过程调用（RPC）的约定，SOAP可以运行在任何其他传输协议上。

   ​      (2). SOAP作为一个基于XML语言的协议用于网上传输数据

   ​      (3). SOAP=在HTTP的基础上+XML数据

   ​      (4). SOAP是基于HTTP的

   ​      (5). HTTP的组成如下

   ​                a:envelope-必须的部分，以XML的根元素出现

   ​                b:Headers-可选的

   ​                c:Body-必须的，在body部分，包含要执行的服务器的方法。和发送到服务器的数据

   

2. ### WSDL说明书

   ​     WebService描述语言WSDL（Sebservice Definition Language）就是用机器能阅读的方式提供的一个正式描述文档而基于XML的语言，用于描述WebService及其函数，参数和返回值，因为是基于xml的，所以WSDL既是机器可以阅读的，又是人可阅读的

    在WSDL说明中，描述了对外发布的服务名称（类），接口方法名称（方法），接口参数（方法参数），服务返回的数据类型（方法返回值）

3. UDDI

      web服务提供商又如何将自己开发的web服务公布到因特网上，这就需要使用UUDI了，UDDL是一个跨产业跨平台的开放性架构，可以帮助web服务提供商在互联网上发布wev服务的信息

   UDDI是一种目录服务，企业可以通过UDDI来注册和搜索Web服务

   简单的来说，UDDI就是一个目录，只不过这个目录中存放的是一些关于web服务的信息而已，并且，UDDI通过SOAP进行通信，构建于.Net之上

   

   ### webservice的优缺点

   1. 优点

      - 异构平台的互通性

           理论上，Web Service最大的优势是提供了异构平台的无缝衔接技术手段，由于不同的用户，使用不同的硬件平台，不同的操作平台，不同的操作系统，不同的软件，不同的协议通信，这就产生了互通性的需求，Web Service使任何两个应用程序，只要能读写XML，那么就能互相通信

      - 更广泛的软件复用（例如，手机淘宝可以复用已有淘宝的业务逻辑）

        软件的复用技术通过组合已有模块来搭建应用程序，能大幅度提高软件的生产效率和质量。用户只要获得了描述Web Service的WSDL文件，就可以方便的生成客户端代理，并通过代理访问Web Service

   2. 缺点

        由于soap是基于xml传输，本身是有XML传输会传输一些无关内容从而影响效率，随着soap协议的完善，soap协议增加了许多内容，这样就导致了使用soap去完成简单的数据传输而携带的数据信息更多效率再受影响

         Web Service作为web跨平台访问的标准技术，很多公司都限定要求使用Web Service，但如果是简单的接口可以直接使用HTTP传输自定义数据格式，开发更快捷。

      webservice采用http作为传输协议，soap作为传输消息的格式

   

   ### ApacheCXF框架的介绍

   1. 关于**Apache CXF**

         Apache CXF = Celtix+XFire,ApacheCXF的前身叫Apache CeltiXfire,现在已经正式更名为Apache CXF了，以下简称为CXF。CXF继承了Celtix和XFire两大开源项目的精华，提供了对JAX-WS全面的支持，并且提供了多种Binding,DataBinding,Transport以及各种Format的支持，并且可以根据实际项目的需要，采用代码优先(Code First)或者WSDL优先（WSDL First）来轻松地实现Web Service的发布和使用，目前它任只是一个Apache的一个孵化项目

   2. **功能特性**

      

   ### ApacheCXF实现WebService(Jax-ws)

   **服务端的代码**

   ```java
   <dependencies>
       <!--要进行jaxws开发-->
       <dependency>
          <groupid>org.apache.cxf</groupId>
          <artifactId>cxf-rt-frontend-jaxws></artifactId>
          <version>3.0.1</version>
       </denendency>
   
         <!--内置jetty web服务器-->
       <dependency>
          <groupid>org.apache.cxf</groupId>
          <artifactId>cxf-rt-transports-http-jetty></artifactId>
          <version>3.0.1</version>
       </denendency>  
   </dependencies>
      
   /**
     对外发布服务的接口
   **/
    @WebService
   public interface HelloService{
       /**
         对外发布服务的接口方法
       **/
       public String sayHello(String name);
   }
   
   public class HelloServiceImpl implements HelloService{
       @override
       public String sayHello(String name){
           return name +“，Welcome to ”
       }
   }
   
   
   //发布服务
   public class Server{
       public static void main(String[] args){
           //发布服务的工厂
           JaxWsServerFactoryBean factory = new JaxWsServerFactoryBean();
           //设置webservice服务器地址
           factory.setAddress("http://localhost:80000/ws/hello");
           //设置服务类
           factory.setServiceBean(new HelloServiceImpl());
           
           //添加日志输入，输出拦截器，观察soap请求，soap响应内容
           factory.getlnlnterceptors().add(new Logginglnlnterceptor());
           factory.getOutlnterceptors().add(new LoggingOutlnterceptor());
           
           //发布服务
           factory.create();
           System.out.println("发布服务成功，端口8000..........")
       }
   }
   
   
   //将http://localhost:80000/ws/hello？wsdl这个地址复制到浏览器里面就可以访问wsdll说明书
   
   
   ```
   
   **写客户端的代码，来调用上面的服务端**
   
   ```java
   //添加依赖
   <dependencies>
       <!--要进行jaxws开发-->
       <dependency>
          <groupid>org.apache.cxf</groupId>
          <artifactId>cxf-rt-frontend-jaxws></artifactId>
          <version>3.0.1</version>
       </denendency>
   
         <!--内置jetty web服务器-->
       <dependency>
          <groupid>org.apache.cxf</groupId>
          <artifactId>cxf-rt-transports-http-jetty></artifactId>
          <version>3.0.1</version>
       </denendency>  
   </dependencies>
       
   public class Client{
       public static void main(Stirng[] args){
           //服务接口访问地址http://localhost:8000/ws/hello
           //创建CXF代理工厂
           JaxWsProxYFactoryBean factory = new JaxWsProxyFactoryBean();
           //设置远程访问服务端地址
           factory.setAddress("http://localhost:8000/ws/hello");
           //设置接口类型
           factory.setServiceClass(HelloService.class);
           //对接口生成代理对象
          HelloService helloService= factory.create(HelloService.class);
           //远程访问服务端方法
           String content=helloService.sayHello("lfk");
               System.out.printlin(content);
       }
   }
   ```
   
   
   
   
   
   
   
   ## Spring整合ApacheCXF实现WebService（Jax-WS）
   
   
   
   #### spring整合ApacheCXF服务端的编写
   
   ``` java
   //添加依赖
   <dependenceies>
      <dependency>
          <groupId>org.apache.cxf</groupId>
          <artifactId>cxf-rt-frontend-jaxws</artifactId>
          <version>3.0.1</version>
      </dependency>   
       
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-context</artifactId>
          <version>4.2.4RELEASE</version>
      </dependency>  
       
        <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-web</artifactId>
          <version>4.2.4RELEASE</version>
      </dependency> 
   </dependenceies>
       
       
    
   //配置web.xml
       <!--cxfservlet配置-->
       <servlet>
          <servlet-name>cxfservlet</servlet-name>
          <servlet-class>org.apache.cxf.transprot.servlet.CXFServlet</servlet-class>
       </servlet>
       <servlet-mapping>
           <servlet-name>cxfservlet</servlet-name>
           <url-pattern>/ws/*</url-pattern>
       </servlet-mapping>
       
       <!--spring容器配置-->
       <context-param>
          <param-name>contextConfigLocation</param-name>
          <param-value>applicationContext.xml</param-value>
       </context-param>
       
       <listener>
           <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
       </listener>
      
      
      
       /**
     对外发布服务的接口
   **/
    @WebService
   public interface HelloService{
       /**
         对外发布服务的接口方法
       **/
       public String sayHello(String name);
   }
   
   public class HelloServiceImpl implements HelloService{
       @override
       public String sayHello(String name){
           return name +“，Welcome to ”
       }
   }
       
   
   
   //在applicationContexttext.xml中发布服务
   <!--
       spirng整合cxf发布服务，关键点
       1.服务地址
       2.服务类
       服务完整访问地址:http://localhost:8080/ws/hello
       3.启动项目发布服务
       4.在浏览器中输入http://localhost:8080/ws/hello？wsdl来查看服务的说明书
       -->
       <jaxws:server address="/hello">
           <jaxws:serviceBean>
              <bean class="服务接口的的实现类:HelloServiceImpl"></bean> 
           </jaxws:serviceBean>    
       </jaxes:server>
       
   ```
   
   
   
   #### Spring整合ApacheCXF客户端的编写
   
   ``` java
   //1.创建spring项目
   //2.添加依赖
   <dependenceies>
      <dependency>
          <groupId>org.apache.cxf</groupId>
          <artifactId>cxf-rt-frontend-jaxws</artifactId>
          <version>3.0.1</version>
      </dependency>   
       
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-context</artifactId>
          <version>4.2.4RELEASE</version>
      </dependency>  
       
        <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-web</artifactId>
          <version>4.2.4RELEASE</version>
      </dependency> 
   </dependenceies>
   
   //spring整合ApacheCXF客户端的配置applicationContext.xml
   <!--
        Spring整合cxf客户端配置
        1.服务地址: http://localhost:8080/ws/hello
        2.服务接口类型
            
       -->
          <jaxws:client id="helloService" serviceClass="com.itheima.service.HelloService"
               address="http://localhost:8080/ws/hello"
          </jaxws:client>
              
              
              
     //JUnit4测试
             public class Client{
                @Resource
                 private HelloService helloService;
                 @Test
                 public void remote(){
                     //查看接口的代理对象
                     System.out.println(helloService.getClass());
                     //远程访问服务端方法
                     System.out.println(helloService.sayHello("Jerry"));
                 }
             }
   ```





# 基于Restful风格的webservice(Jax-rs)实现

#### 基于Restful风格的服务端代码

1. 创建项目

2. 添加依赖

   - ```java
     <depnedencies>
         <dependency>
            <gorupId>org.apache.cxf</gorupId>
            <artifactid>cxf-rt-frontend-jaxrs</artifactid>
            <version>3.0.1</version>
         </dependency>  
         <dependency>
            <gorupId>org.apache.cxf</gorupId>
            <artifactid>cxf-rt-transports-http-jetty</artifactid>
            <version>3.0.1</version>
         </dependency>
          <dependency>
            <gorupId>org.apache.cxf</gorupId>
            <artifactid>cxf-rt-rs-client</artifactid>
            <version>3.0.1</version>
         </dependency>
         <!--JSON支持的jar包-->
         <dependency>
            <gorupId>org.apache.cxf</gorupId>
            <artifactid>cxf-rt-rs-extension-providers</artifactid>
            <version>3.0.1</version>
         </dependency>
          <dependency>
            <gorupId>org.codehaus.jettison</gorupId>
            <artifactid>jettison</artifactid>
            <version>1.3.7</version>
         </dependency>
         
     </dependencies>
     ```

     

3. 开发访问的实现接口，实现类，实体类

   - ```java
     @XmlRootElement(name="User")
     //基于restful风格的webservice,客户端与服务器端之间通信可以传递xml数据，也可以传递json数据
     //@XmlRootElement指定对象序列化为xml或json数据时根节点的名称
     /**
        xml:
          <User>
              <id></id>
              <username></username>
              <city></city>
          </User>
         json:
             {"User""{"id":100,"username":"lfk","city":"北京"}}
     **/
     public class User{
         private Integer id;
         private Stirng username;
         private String city;
         private List<Car> cars = new ArrayList<Car>();
         public Integer getId(){return id;}
         public void setId(Integer id){this.id=id;}
         public String getUsername()){return username};
         public void setUsername(String username){this.username=username;}
         public String getCity(){return city}
         public void setCity(String city){this.city=city;}
     }
     ```

   - ```java
     @Path("/userService")
     @Produces("*/*")
     public interface IUserService{
         @POST
         @Path("/user")
         @Consumes("application/xml","application/json")
         public void saveUser(User user);
         
         @PUT
         @Path("/user")
         //@Consumes表示服务器支持的请求的数据类型
         @Consumes("application/xml","application/json")
         public void updateUser(User user);
         
         @GET
         @Path("/user")
         @Produces("application/xml","application/json")
         public List<User> findAllUsers(User user);
         
         @GET
         @Path("/user/{id}")
         //@Produces表示服务器支持的返回的数据类型
         @Produces("application/xml","application/json")
         public User findUserById(@PathParam("id") Integer id);
         
          @DELETE
         @Path("/user/{id}")
         @Consumes("application/xml","application/json")
         public User deleteUser(@PathParam("id") Integer id);
     }
     ```

   - ```java
     public class UserServiceImpl implements IUserService{
         public void saveUser(User user){System.out.println("save user"+user);}
         public void updateUser(User user){System.out.println("update user"+user);}
         public List<User> findAllUsers(){
             List<User> users = new ArrayList<User>();
             User user1 = new User();
             user.setId(1);
             user1.setUsername("小明");
             user1.setCity("北京");
         }
     }
     ```

4. 发布服务

   - ```java
     public class Server{
         public static void main(String[] args){
             //创建发布服务的工厂
             JAXRSServerFactoryBran factory = new JAXRSServerFactoryBran();
             //设置服务地址
             factory.setAddress("http://localhost:8001/ws/");
             //设置服务类
             factory.setServiceBean(new UserServiceImpl());
             //添加日志输入输出拦截器,可以看到客户端请求的报文格式以及服务器端返回的报文的格式
             factory.getInInterceptors().add(new LoggingInInterceptor());
              factory.getOutInterceptors().add(new LoggingOutInterceptor());
             //发布服务
             factory.create();
             System.out.println("发布服务成功，端口8001");
         }
     }
     ```

     

#### 基于Restful风格的客户端代码

1. 创建项目

2. 添加依赖

   - ```java
     <depnedencies>
         <dependency>
            <gorupId>org.apache.cxf</gorupId>
            <artifactid>cxf-rt-frontend-jaxrs</artifactid>
            <version>3.0.1</version>
         </dependency>  
         <dependency>
            <gorupId>org.apache.cxf</gorupId>
            <artifactid>cxf-rt-transports-http-jetty</artifactid>
            <version>3.0.1</version>
         </dependency>
          <dependency>
            <gorupId>org.apache.cxf</gorupId>
            <artifactid>cxf-rt-rs-client</artifactid>
            <version>3.0.1</version>
         </dependency>
         <!--JSON支持的jar包-->
         <dependency>
            <gorupId>org.apache.cxf</gorupId>
            <artifactid>cxf-rt-rs-extension-providers</artifactid>
            <version>3.0.1</version>
         </dependency>
          <dependency>
            <gorupId>org.codehaus.jettison</gorupId>
            <artifactid>jettison</artifactid>
            <version>1.3.7</version>
         </dependency>
         
     </dependencies>
     ```

     

3. 写junt，远程访问服务端

   - ```java
     public class Client{
         User user = new User();
         user.setId(100);
         user.setUsername("jerry");
         user.setCity("北京");
         //通过webclient对象远程调用服务端
         /**
            .create()指定服务端地址
            .type()指定请求数据格式(xml,json)
            .accept()指定响应数据格式
            
         **/
         WebClient.create("http://localhst:8080/ws/userServices/user")
                  .post(user)
                  .type(MediaType.APPLICATION_JSON);//指定请求的数据格式为json，默认是xml
         
         
         
        @Test
         public void testGet(){
             WebClient.create("http://localhost:8080/ws/userService/user/1")
                 .type(MediaType.APPLICATION_JSON)
                 .get(User.class)//返回的是什么类型的数据，这个地方就写哪个类型
         }
     }
     ```

     

# Spring整合CXF实现基于Restful风格的webservice(jax-rs)

#### 服务端

1. 创建web项目

2. 添加依赖

   - ```java
     <!--cxf进行rs开发必须导入-->
         <dependencies>
            <dependency>
               <groupId>orgcapache.cxf</groupId>
               <artifactId>cxf-rt-frontend-jaxrs</artifactId>
               <version>3.0.1</version>
            </dependency>
         
         <!--客户端-->
         <dependency>
               <groupId>orgcapache.cxf</groupId>
               <artifactId>cxf-rt-rs-client</artifactId>
               <version>3.0.1</version>
            </dependency>
         </dependencies>
         
         <!--扩展JSON提供者-->
           <dependency>
               <groupId>orgcapache.cxf</groupId>
               <artifactId>cxf-rt-rs-extendsion-providers</artifactId>
               <version>3.0.1</version>
            </dependency>
         
            <!--转换JSON工具包，被extension providers依赖-->
             <dependency>
               <groupId>ord.codehaus.jettison</groupId>
               <artifactId>jettison</artifactId>
               <version>1.3.7</version>
            </dependency>
         
          <!--Spring核心-->
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-context</artifactId>
               <version>4.2.4.RRLEASE</version>
            </dependency>
         
           <!--Spring web集成-->
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-web</artifactId>
               <version>4.2.4.RRLEASE</version>
            </dependency>
         
         <!--Spring整合Junt-->
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-test</artifactId>
               <version>4.2.4.RRLEASE</version>
            </dependency>
         
         <!--junit开发包-->
           <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>4.12</version>
            </dependency>
        
         </dependencies>
     ```

3. web.xml

   - ```java
     <!--cxfservlet配置-->
      <servlet>
         <servlet-name>cxfservlet</servlet-name>
         <servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>
      </servlet>
      <servlet-mapping>
         <servlet-name>cxfservlet</servlet-name>
         <url-pattern>/ws/</url-pattern>
      </servlet-mapping>
             
      <!--spirng容器配置-->
         <context-param>
             <param-name>contextConfigLocation</param-name>
             <param-value>applicationContext.xml</param-value>
         </context-param>
             
         <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
        </listener>
             
     ```

4. 服务接口，实现，实体类

   - ```java
     @XmlRootElement(name="User")
     //基于restful风格的webservice,客户端与服务器端之间通信可以传递xml数据，也可以传递json数据
     //@XmlRootElement指定对象序列化为xml或json数据时根节点的名称
     /**
        xml:
          <User>
              <id></id>
              <username></username>
              <city></city>
          </User>
         json:
             {"User""{"id":100,"username":"lfk","city":"北京"}}
     **/
     public class User{
         private Integer id;
         private Stirng username;
         private String city;
         private List<Car> cars = new ArrayList<Car>();
         public Integer getId(){return id;}
         public void setId(Integer id){this.id=id;}
         public String getUsername()){return username};
         public void setUsername(String username){this.username=username;}
         public String getCity(){return city}
         public void setCity(String city){this.city=city;}
     }
     ```

   - ```java
     @Path("/userService")
     @Produces("*/*")
     public interface IUserService{
         @POST
         @Path("/user")
         @Consumes("application/xml","application/json")
         public void saveUser(User user);
         
         @PUT
         @Path("/user")
         //@Consumes表示服务器支持的请求的数据类型
         @Consumes("application/xml","application/json")
         public void updateUser(User user);
         
         @GET
         @Path("/user")
         @Produces("application/xml","application/json")
         public List<User> findAllUsers(User user);
         
         @GET
         @Path("/user/{id}")
         //@Produces表示服务器支持的返回的数据类型
         @Produces("application/xml","application/json")
         public User findUserById(@PathParam("id") Integer id);
         
          @DELETE
         @Path("/user/{id}")
         @Consumes("application/xml","application/json")
         public User deleteUser(@PathParam("id") Integer id);
     }
     ```

   - ```java
     public class UserServiceImpl implements IUserService{
         public void saveUser(User user){System.out.println("save user"+user);}
         public void updateUser(User user){System.out.println("update user"+user);}
         public List<User> findAllUsers(){
             List<User> users = new ArrayList<User>();
             User user1 = new User();
             user.setId(1);
             user1.setUsername("小明");
             user1.setCity("北京");
         }
     }
     ```

5. Spring整合CXF,applicationContext.xml

   - ```java
     <!--
         Spring整合cxf发布基于restful风格的服务
         1.服务地址
         2.服务类
         3.服务完整访问地址
     -->
         <jaxrs:server address="/userService">
             <jaxrs:serviceBeans>
                <bean class="com.itheima.service.UserServiceImpl"></bean>
             <jarxrs:serviceBeans>
         </jaxrs:servier>
     ```

6. 发布服务：运行服务，发布服务



# Spring整合CXF实现基于Restful风格的webservice(jax-rs)-客户端

1. 创建项目

2. 添加依赖
3. 调用服务端，跟单独的restful风格的客户端代码的编写一样











# schema规范

##### 这是一个自己写的约束文档

```html

<?xml version="1.0" encoding="UTF-8"?>
<schema xmlns="http://www.w3.org/2001/XMLSchema"
        targetNamespace="http://wwww.atguigu.cn"
        elementFormDefault="qualified">
    <element name="书架">
       <complexType>
           <sequence maxOccurs="unbounded">
              <element name="书">
                 <compleType>
                     <sequence>
                         <element name="书名" type="String"/>
                         <element name="作者" type="String"/>
                         <element name="售价" type="String"/>
                     </sequence>
                 </compleType>
           </element>
           </sequence>
        </complexType>
    </element>
</schema>
```

##### 下面这个文档是根据上面的约束文档生成的

```html
<?xml version="1.0" encoding="UTF-8"?>
<书架 xmlns="http://www.atguigu.cn"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://www.atguigu.cn book.xsd">
   <书>
       <书名>JavaScripth开发</书名>
       <作者>老</作者>
       <售价>28.00元</售价>
   </书>    
</书架>
```




















​     