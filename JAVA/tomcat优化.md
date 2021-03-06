# 执行器（线程池）
##### 默认的tomcat没有启用线程池,在tomcat中每一个用户请求都是一个线程，所以可以使用线程池提高性能。这里前台其实有一个调度线程，然后调度线程会放入线程池内，然后到到一定的时候线程池的任务变成工作线程。
### 找到以下位置代码
![](/images/tomcat%E5%BC%80%E5%90%AF%E7%BA%BF%E7%A8%8B%E6%B1%A0.png)
### 更改为以下代码
```xml
<Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
        maxThreads="800" minSpareThreads="100"  maxQueueSize="100" prestartminSpareThreads="true" />
```
#### 参数说明
| 参数 | 说明  |
| ------------ | ------------ |
|threadPriority|优先级|
|daemon|守护进程|
|namePrefix|名称前缀|
|maxThreads|最大线程数|
|minSpareThreads|最小活跃线程数|
|maxIdleTime|空闲线程等待时间|
|maxQueueSize|最大的等待队里数，超过则请求拒绝|
|prestartminSpareThreads|是否在启动时就生成minSpareThreads个线程|
|threadRenewalDelay|重建线程的时间间隔|

### 指定线程池
![](/images/Tomcat%E5%90%AF%E7%94%A8%E7%BA%BF%E7%A8%8B%E6%B1%A0.png)

# 连接器（Connector）优化
#####  Connector是连接器，负责接收客户的请求，以及向客户端回送响应的消息。所以 Connector的优化是重要部分。默认情况下 Tomcat只支持200线程访问，超过这个数量的连接将被等待甚至超时放弃，所以我们需要提高这方面的处理能力。
#####  其中port代表服务接口；protocol代表协议类型；connectionTimeout代表连接超时时间，单位为毫秒；redirectPort代表安全通信（https）转发端口，一般配置成443。
![](/images/Tomcat%E8%BF%9E%E6%8E%A5%E5%99%A8%E4%BC%98%E5%8C%96.png)

### 常用的参数如下
| 参数 | 说明  |
| ------------ | ------------ |
|maxPostSize|参数形式处理的最大长度，默认值为2097152（2兆字节）。上传提交的时候可以用的|
|acceptCount|请求的最大队列长度，当队列满时收到的任何请求将被拒绝|
|acceptorThreadCount|用于接受连接的线程的数量|
|disableUploadTimeout|此标志允许servlet容器在数据上传时使用不同的连接超时，通常较长。如果没有指定，该属性被设置为true，禁用上传超时。|
|connectionUploadTimeout|上传数据过程中，指定的以毫秒为单位超时时间。只有在设置disableUploadTimeout为false有效。|
|connectionTimeout|在将提交的请求URI行呈现之后，连接器将等待接受连接的毫秒数。使用值-1表示没有超时（即无限）。默认值是60000（60秒），但请注意，Tomcat的标准server.xml中，设置为20000（即20秒）。|
|maxConnections|服务器接受并处理的最大连接数|
|SSLEnabled|在连接器上使用此属性来启用SSL加密传输|
|port|TCP端口号，连接器利用该端口号将创建一个服务器套接字，并等待传入的连接。|
|protocol|设置协议来处理传入流量。默认值是 HTTP/1.1，将使用自动切换机制来选择阻塞的基于Java的连接器或APR /native 为基础的连接器。可用以下值：org.apache.coyote.http11.Http11Protocol -阻塞式的Java连接器;org.apache.coyote.http11.Http11NioProtocol -不阻塞Java连接器;org.apache.coyote.http11.Http11AprProtocol的 -的APR / native 连接器|
|maxThreads|最多同时处理的连接数，Tomcat使用线程来处理接收的每个请求。这个值表示Tomcat可创建的最大的线程数。如果没有指定，该属性被设置为200。如果使用了execute将忽略此连接器的该属性，连接器将使用execute，而不是一个内部线程池来处理请求。|
|minSpareThreads|始终保持运行最小线程数。如果没有指定，则默认为10。|
|maxHttpHeaderSize|请求和响应的HTTP头的（以字节为单位的）最大尺寸。如果没有指定，该属性被设置为8192（8 KB）。|
|tcpNoDelay|如果设置为true，TCP_NO_DELAY选项将被设置在服务器上的套接字上，在大多数情况下，这样可以提高性能。默认设置为true。|
|compression| 通常会在ngnix里面配置压缩开启压缩GZIP，当值是“off ”（禁用压缩），“on ”（允许压缩，这会导致文本数据被压缩），“force ”（强制在所有的情况下压缩），或者一个整数值（这是相当于为“on”，但指定了输出之前被压缩的数据最小量）。如果不知道内容长度但被设置为“on”或更积极的压缩，输出的数据也将被压缩。如果没有指定，该属性被设置为“off”。|
|compressionMinSize|如果压缩被设置为“on”，那么该属性可以用于指定在输出之前被压缩的数据的最小量。如果未指定，此属性默认为“2048”。|
|redirectPort|如果该连接器支持非SSL请求，并且接收到的请求为满足安全约束需要SSL传输， Catalina 将自动将请求重定向到指定的端口号。|
|enableLookups|设置为false时跳过DNS查找，并返回字符串情势的IP地址（从而提高性能）。默认景象下，禁用DNS查找。|
|URIEncoding|指定使用的字符编码，来解码URI字符。如果没有指定，ISO-8859-1将被使用。|
|executor|指向Executor元素的引用。|
### 最好实例
```xml
        <!-- maxPostSize 参数形式处理的最大长度，默认为2097152（2兆字节）,上传提交的时候可以用的,这里设置1GB
             acceptCount 请求的最大队列长度，当队列满时收到的任何请求将被拒绝
             acceptorThreadCount 用于接受连接的线程的数量
             disableUploadTimeout 禁用上传超时
             maxConnections 服务器接受并处理的最大连接数
             SSLEnabled 在连接器上使用此属性来启用SSL加密传输 -->
             
        <Connector executor="tomcatThreadPool"
                connectionTimeout="20000"
                port="8080"
                protocol="org.apache.coyote.http11.Http11NioProtocol"
                redirectPort="8443"
                maxHttpHeaderSize="8192"
                enableLookups="false"
                maxPostSize="1048576000"
                URIEncoding="UTF-8"
                acceptCount="1000"
                acceptorTreadCount="100"
                disableUploadTimeout="true"
                maxConnections="10000"
                SSLEnabled="false"/>
```
# 禁用AJP连接器
#### 如果是使用Nginx+tomcat的架构，所以用不着AJP协议，所以把AJP连接器禁用。
##### AJPv13协议是面向包的。WEB服务器和Servlet容器通过TCP连接来交互；为了节省SOCKET创建的昂贵代价，WEB服务器会尝试维护一个永久TCP连接到servlet容器，并且在多个请求和响应周期过程会重用连接。
![](/images/Tomcat%E7%A6%81%E7%94%A8AJP.png)

# JVM参数的优化
#### 因为Tomcat运行在JAVA虚拟机之上,适当调整Tomcat的运行JVM参数可以提升整体性能。
### 常用参数

| 参数 | 说明  |
| ------------ | ------------ |
|  file.encoding |  默认文件编码 |
|  -Xmx1024m | 设置JVM最大可用内存为1024MB  |
|  -Xms1024m | 设置JVM最小内存为1024m。此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存。  |
|  -XX:NewSize | 设置年轻代大小  |
|  -XX:MaxNewSize | 设置最大的年轻代大小 |
|  -XX:PermSize | 设置永久代大小  |
|  -XX:MaxPermSize| 设置最大永久代大小 |
|-XX:NewRatio=4|设置年轻代（包括Eden和两个Survivor区）与终身代的比值（除去永久代）。设置为4，则年轻代与终身代所占比值为1：4，年轻代占整个堆栈的1/5|
|-XX:MaxTenuringThreshold|设置垃圾最大年龄，默认为：15。如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代。对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象再年轻代的存活时间，增加在年轻代即被回收的概论。 |
|-XX:+DisableExplicitGC|这个将会忽略手动调用GC的代码使得System.gc()的调用就会变成一个空调用，完全不会触发任何GC|

### windows
#### 修改bin/catalina.bat文件,在setlocal下面一行添加
```shell
set JAVA_OPTS=-Dfile.encoding=UTF-8 -server-Xms1024m -Xmx2048m -XX:NewSize=512m -XX:MaxNewSize=1024m -XX:PermSize=256m-XX:MaxPermSize=256m -XX:MaxTenuringThreshold=10 -XX:NewRatio=2-XX:+DisableExplicitGC
```
![](/images/Tomcat%E4%BF%AE%E6%94%B9JVM%E5%8F%82%E6%95%B0Windows.png)

### linux
#### 修改bin/catalina.sh文件,在os400=false之前添加
```shell
JAVA_OPTS="-Dfile.encoding=UTF-8-server -Xms1024m -Xmx2048m -XX:NewSize=512m -XX:MaxNewSize=1024m-XX:PermSize=256m -XX:MaxPermSize=256m -XX:MaxTenuringThreshold=10-XX:NewRatio=2 -XX:+DisableExplicitGC"
```
![](/images/Tomcat%E4%BF%AE%E6%94%B9JVM%E5%8F%82%E6%95%B0Linux.png)


---
## Tomcat在 7.0.73, 8.0.39, 8.5.7 版本后，添加了对于http头的验证。
### 配置tomcat的 conf/catalina.properties 在末尾添加或者修改最后一行：
```properties
tomcat.util.http.parser.HttpParser.requestTargetAllow=|{}
```

## Tomcat热部署
```diff
+替换WEB-INF/lib目录中的jar文件或WEB-INF/classes目录中的class文件时，reloadable="true"会让修改生效（但代价不小），该选项适合调试。
<Context docBase="xxx" path="/xxx" reloadable="true"/> 
+在webapps目录中增加新的目录、war文件、修改WEB-INF/web.xml，autoDeploy="true"会新建或重新部署应用，该选项方便部署。
<Context docBase="xxx" path="/xxx" autoDeploy="true"/> 
```
