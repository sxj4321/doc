#### tomcat是什么？
tomcat实际是运行在jvm中的一个进程。我们把它定义为【中间件】，顾名思义，他是一个在java项目与jvm之间的
中间容器。

#### 为什么需要tomcat？
web项目的本质，是一大堆的资源文件和方法。我们的web项目一般是没有main方法的，意味着web项目中的方法不会自动运行起来。
我们把web项目部署进tomcat的webapp中的目的是很明确的，那就是希望tomcat去调用我们写好的方法去为客户端返回需要的资源和数据。

#### http请求
http://127.0.0.1:8080/test?a=1&b=2

- 协议名称：http
- host：127.0.0.1
- 端口：8080
- 请求路径：test
- 参数：a=1&b=2

#### tomcat配置

##### 公共属性
| 屬性 | 描述 |
| ------ | ------ |
| asyncTimeout | 设置异步请求超时时间，单位毫秒。默认为30000 |
| enableLookups | 设置为true时request.getRemoteHost()返回实际的主机名，设置为false时request.getRemoteHost()返回IP地址。默认为false。 |
| maxHeaderCount | 限制请求Header的长度，如果超过请求将被拒绝。0表示没有限制，默认值为100 |
| maxParameterCount | 设置参数的最大长度，参数由容器自动解析，超出长度的参数将被忽略，0表示没有限制，默认值为10000。请注意， FailedRequestFilter 过滤器可以用来拒绝达到了极限值的请求。 |
| maxPostSize | 设置由容器解析的URL参数的最大长度，-1(小于0)为禁用这个属性，默认为2097152(2M) 请注意， FailedRequestFilter 过滤器可以用来拒绝达到了极限值的请求。 |
| port | 设置连接器监听的端口(0-65535)。如果设置成0，将随机生成(通常只用于嵌入式和测试应用程序) |
| protocol | 设置连接器 处理类。现在tomcat提供4种连接器:org.apache.coyote.http11.Http11Protocol - 阻塞的的Java连接器org.apache.coyote.http11.Http11NioProtocol - 非阻塞的的Java NIO连接器；org.apache.coyote.http11.Http11Nio2Protocol - 非阻塞的的Java NIO2连接器；org.apache.coyote.http11.Http11AprProtocol - 本地连接器也可以使用自定义实现的连接器。注意: 如果配置的是默认的HTTP/1.1，将自动配置一个 非阻塞的java NIO连接器 或 APR/native(本地连接器)。 如果环境变量(window path 和 LD_LIBRARY_PATH unix\linux)含有tomcat本地库，将使用APR/native连接器。 如果环境变量不存在将使用 非阻塞的java NIO连接器。 APR/native连接器 和 非阻塞的java NIO连接器 配置参数是不一样的。建议: 在生产环境中配置一个固定的连接器，不使用自动配置。看看我们的连接器比较图表。 Java连接器的配置是相同的,http和https。有关APR 连接器的更多信息和APR 具体的SSL设置APR 请访问文档 |
| redirectPort | 配置指定端口来 ssl连接，一般默认配置是8443，但是浏览器默认的是443端口请求ssl服务器，所以在https 下将8443改为443.|
| URIEncoding | 配置URI使用的字符编码，来解码?之前的字符串。 一般情况下默认使用utf-8，在org.apache.catalina.STRICT_SERVLET_COMPLIANCE(系统属性)为true的情况下使用 ISO-8859-1。|
| URIEncoding | 配置URI使用的字符编码，来解码?之前的字符串。 一般情况下默认使用utf-8，在org.apache.catalina.STRICT_SERVLET_COMPLIANCE(系统属性)为true的情况下使用 ISO-8859-1。|

##### 标准属性
标准的HTTP连接器(BIO、NIO NIO2和APR/native)都支持以下属性除了常见的连接器上面列出的属性。
| 屬性 | 描述 |
| ------ | ------ |
| acceptCount | 当tomcat起动的线程数达到最大时，接受排队的请求个数，默认值为100。 |
| compression | 连接器可以使用HTTP/1.1 GZIP压缩为了节省服务器的带宽。 参数的可接受的值是“关闭”(禁用压缩),“on”(允许压缩,导致文本数据压缩),“力”(力量压缩在所有情况下),或一个数值整数值(相当于“上”,但指定的最小输出压缩之前)的数据量。 如果内容长度是未知的和压缩设置为“on”或更激进,输出也将被压缩。 如果不指定,这个属性被设置为“关闭”。 |
| compressionMinSize | 如果压缩设置为“on”,那么这个属性可用于指定输出前的最低数量的数据压缩。如果不指定,该属性默认为“2048”。 |
| executor | 在一个线程池的引用名称。 如果设置了这个属性,连接器将使用执行程序,和所有其他线程属性将被忽略。 注意,如果没有指定一个线程池,连接器将使用一个私有的,内部执行人提供线程池。 |
| connectionTimeout | 这个连接器将等待的毫秒数,接受一个连接后,请求URI提交。 使用一个值为1表示没有(无限)超时。默认值为60000(即60秒),但请注意,标准的server.xml附带Tomcat这个设置为20000(即20秒)。 |
| connectionUploadTimeout | 指定超时时间,以毫秒为单位,使用数据上传是在进步。 这只生效disableUploadTimeout是否设置为false。 |
| disableUploadTimeout | 这个标志允许servlet容器使用一个不同的,通常长在数据上传连接超时。 如果不指定,这个属性被设置为true,表示禁用该时间超时。 |
| maxConnections | 最大连接数,服务器将接受和处理在任何给定的时间。 这个数字已经达到时,服务器将接受,但不是过程,另外一个连接。 这些额外的连接被阻塞,直到正在处理的连接数量低于maxConnections此时服务器将重新开始接受和处理新连接。 注意,一旦达到极限,操作系统可能仍然基于acceptCount接受连接设置。默认值不同的连接器类型。 对于生物默认的值是maxThreads除非使用一个执行人在这种情况下,默认的值将maxThreads执行人。 NIO和NIO2默认是10000。APR/native,默认是8192。 |
| maxThreads | 请求处理线程的最大数量是由这个连接器,因此决定了同时发生的请求的最大数量,可以处理。 如果不指定,这个属性被设置为200。 如果一个执行人与这个连接器,该属性将被忽略的连接器使用执行程序将执行任务而不是一个内部线程池。 |
| minSpareThreads | 最低数量的线程总是运行。如果没有指定,默认为10。 |

#### tomcat优化
tomcat优化需要根据部署系统环境、cpu核数、内存大小、jdk版本来调整
##### 内存优化
| 参数 | 说明 |
| ------ | ------ |
| -server | -Server模式启动时，速度较慢，但是一旦运行起来后，性能将会有很大的提升。当虚拟机运行在-client模式的时候,使用的是一个代号为C1的轻量级编译器, 而-server模式启动的虚拟机采用相对重量级,代号为C2的编译器。 C2比C1编译器编译的相对彻底,服务起来之后,性能更高。 |
| -Xms -XmX | 堆的初始大小与最大大小，一般设置为一样，避免频繁分配内存引起性能开销。一般业内规则为配置物理内存的80%。 |
| -Xmn | 新生代大小，通过次参数控制普通GC与full GC的频率和时间。|
##### io模式优化
| io模式 | 说明 |
| ------ | ------ |
| bio | 适用于简单项目，采用java老bio模型。一个线程处理一个请求。缺点：并发量高时，线程数较多，浪费资源。|
| nio | 适用于后台耗时较多的请求的操作，采用java nio模型。 |
| nio2 | 适用于后台耗时较多的请求的操作，采用java aio模型。nio2是tomcat8新增的一种io模式，在速度上比nio快50%左右，无限接近apr。 |
| apr | 即Apache Portable Runtime，从操作系统层面解决io阻塞问题。 |
##### 配置优化
```xml
<!--连接器设置-->
<Connector
port="8080"
protocol="org.apache.coyote.http11.Http11AprProtocol" --协议类型
disableUploadTimeout="true"
keepAliveTimeout="20000"
connectionTimeout="20000" --已接受，但未被处理的请求的等待超时时间 ms
redirectPort="8443" --安全通信的转发端口
URIEncoding="UTF-8"--URL编码字符集
minSpareThreads="100" --默认初始化和保持空闲的线程数
enableLookups="false"--关闭DNS反向查询
useURIValidationHack="false" --关闭不必要的检查
maxThreads="1000" --处理请求线程的最大数目 未配置为200 此属性会被忽略
acceptCount="1000" --所用可能的线程都在使用时传入连接请求的最大长度
disableUploadTimeout="true" --设置允许更长的超时连接
maxConnections="1000"--接受和处理的最大连接数(nio/nio2 1000，apr 8192)
maxHttpHeaderSize="8192"--请求和响应http头的最大大小 8k
tcpNoDelay="true" --tcp不延迟
compression="on"--是否启用压缩 on off force
compressionMinSize="2048" --压缩前数据最小值 2k byte
noCompressionUserAgents="gozilla,traviata" --设置哪些浏览器不压缩
compressableMimeType="text/html,text/xml,text/css,application/javascript,text/plain" --设置压缩的文件类型
/>
```

```xml
<!--连接池设置-->
<Executor
name="tomcatThreadPool" --线程池名
namePrefix="catalina-exec-" --线程名称前缀 namePrefix+threaNumber
maxThreads="1000" --池中最大线程数
minSpareThreads="100" --活跃线程数 会一直存在
maxIdleTime="60000" --线程空闲时间，超过该时间，线程会被销毁 ms
maxQueueSize="Integer.MAX_VALUE" --被执行前线程的排队数目
prestartminSpareThreads="false" --启动线程池时，是否启用minSpareThreads部分线程
threadPriority="5" --线程池中线程优先级 1~10
className="org.apache.catalina.core.StandardThreadExecutor" --线程实现类 自定义线程需时间 org.apache.catalina.Executor类
/>
<!--当配置了连接池时，需要配置该连接器-->
<Connector
executor="tomcatThreadPool" --线程池名
port="8080"
protocol="org.apache.coyote.http11.Http11AprProtocol"
connectionTimeout="20000"
redirectPort="8443" />
```
请尽量使用Executor，他因为Executor可在各个Connector之间共享，减少创建销毁线程的消耗，提高线程的使用效率！
