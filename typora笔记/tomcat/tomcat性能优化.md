#### 查看Tomcat中的默认的垃圾收集器

1.在tomcat/bin/catalina.sh的配置中，加入如下配置

   ```html
JAVA_OPTS="-Djava.rim.serer.hcstname=192.168.192.138 -Dcom.sun.management.jmxremote.port=8999 -Dcom.sun.management.jmxremote.rm1.port=8999 -Dcom.sun.management.jmxremote.ss1=false -Dcom.sun.management.Jmxremote.authenticate=false"
   ```

2. 打开Jconsole,查看远程的tomcat的概要信息，连接远程tomcat
3. 双击打开本地JDK/bin目录下的jconsole.exe程序，输入1步骤配置的8999端口号

##### GC参数

| 参数                    | 描述                                                         |
| :---------------------- | ------------------------------------------------------------ |
| -XX：+UseSerialGC       | 启用串行收集器                                               |
| -XX：+UseParallelGC     | 启用并行垃圾收集器，配置了该选项，那么-XX:+UseParalleloldGC默认启用 |
| -XX:+UseparalleloldGC   | FullGC采并行收集，默认禁用，如果设置了 -XX:+userparallelGC则自动启用 |
| -XX:+UseParNewGC        | 年青代采用并行收集器，如果设置了-XX：+UseConcMarkSweepGC选项，自动启用 |
| -XX:+ParallelGCThreads  | 年青代及老年代垃圾回收使用的线程数，默认值依赖于JVM使用的CPU个数 |
| -XX:+UseConcMarkSweepGC | 对于老年代，启用CMS垃圾收集器。当并行收集器无法满足应用的延迟需求时，推荐使用CMS或G1收集器，启用该选项后，-XX:+UseParNewGC自动启用 |
| -XX:+UseG1GC            | 启用G1收集器，G1是服务器类型的收集器，用于多核，大内存的机器，它在保持高吞吐量的情况下，高概率满足GC暂停时间的目标 |

# Tomcat配置调优

调整tomcat/conf/server.xml中关于连接器的配置可以提升应用服务器的性能

| 参数           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| maxConnections | 最大连接数，当到达该值后，服务器接收但不会处理更多的请求，额外的请求将会阻塞直到连接数低于maxConnections,可以通过ulimit -a查看服务器限制，对于cpu要求更高时，建议不要配置过大，对于cup要求不是太高时，建议配置在2000左右。当然这个需要服务器硬件的支持 |
| maxThreads     | 最大线程数，需要根据服务器的硬件情况，进行一个合理的设置     |
| acceptCount    | 最大排队等待数，当服务器接收的请求数量达到maxConnections,此时Tomcat会将后面的请求，存放在任务队列中进行排序，acceptCount指的就是任务队列中排队等待的请求数，一条tomcat的最大请求处理数量，是maxConnections+acceptCount |

