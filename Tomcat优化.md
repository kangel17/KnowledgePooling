### tomcat优化
1. tomcat7安装目录\bin\catalina.bat(linux修改的是catalina.sh文件)，添加如下语句：  
	- JAVA_OPTS=-Djava.awt.headless=true -Dfile.encoding=UTF-8 -server -Xms1024m -Xmx1024m -Xss1m -XX:NewSize=256m -XX:MaxNewSize=512m -XX:PermSize=256M  -XX:MaxPermSize=512m -XX:+DisableExplicitGC
2. 使用Tomcat的连接池  
	- executor="tomcatThreadPool"
	- protocol="org.apache.coyote.http11.Http11NioProtocol"
	- redirectPort="8443"  
	- connectionTimeout="30000"   
	- enableLookups="false"   
	- keepAliveTimeout="15000"   
	- URIEncoding="UTF-8"  
	- maxHttpHeaderSize="32768"  
	- acceptCount="200"/>  
	说明：  
	- maxThreads：最大线程数300，一旦线程超过这个值，Tomcat就会关闭不再需要的线程  
	- minSpareThreads：初始化建立的线程数 50  
	- maxIdleTime：为最大空闲时间、单位为毫秒。  
	- executor为线程池的名字，对应Executor 中的name属性；Connector 标签中不再有maxThreads的设置。   
3. tomcat不使用线程池则基本配置如下：  
	- protocol="HTTP/1.1"     
	- redirectPort="8443"     
	- connectionTimeout="30000"     
	- keepAliveTimeout="15000"    
	- enableLookups="false"    
	- URIEncoding="UTF-8"    
	- maxHttpHeaderSize="32768"    
	- maxThreads="300"    
	- acceptCount="200" />  
	修改Tomcat的/conf目录下面的server.xml文件，针对端口为8080的连接器添加如下参数：  
    - connectionTimeout：连接失效时间，单位为毫秒、默认为60s、这里设置为30s，如果用户请求在30s内未能进入请求队列，视为本次连接失败。   
	- keepAliveTimeout：连接的存活时间，默认和connectionTimeout一致，这里可以设为15s、这意味着15s之后本次连接关闭. 如果页面需要加载大量图片、js等静态资源，需要将参数适当调大一点、以免多次创建TCP连接。          
	- enableLookups：是否对连接到服务器的远程机器查询其DNS主机名，一般情况下这并不必要，因此设为false即可。          
	- URIEncoding：设置URL参数的编码格式为UTF-8编码，默认为ISO-8859-1编码。          
	- maxHttpHeaderSize：设置HTTP请求、响应的头部内容大小，默认为8192字节(8k)，此处设置为32768字节(32k)、和Nginx的设置保持一致。          
	- maxThreads：最大线程数、用于处理用户请求的线程数目，默认为200、此处设置为300          
	- acceptCount：用户请求等候队列的大小，默认为100、此处设置为200  
4. Linux系统默认一个进程能够创建的最大线程数为1024、因此对高并发应用需要进行Linux内核调优，

