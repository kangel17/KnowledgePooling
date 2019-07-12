### jvm
#### JVM堆内存分为2块：Permanent Space 和 Heap Space  
1. **Permanent** 即 持久代（Permanent Generation），主要存放的是Java类定义信息，与垃圾收集器要收集的Java对象关系不大。  
2. **Heap** = { Old + NEW = {Eden, from, to} }，Old 即 年老代（Old Generation），New 即 年轻代（Young Generation）。年老代和年轻代的划分对垃圾收集影响比较大。  

##### 年轻代
所有新生成的对象首先都是放在年轻代。年轻代的目标就是尽可能快速的收集掉那些生命周期短的对象。年轻代一般分3个区，**1个Eden区**，**2个Survivor区**（from 和 to）。针对年轻代的垃圾回收即 **Young GC**。  

1. 大部分对象在**Eden区**中生成。 当Eden区满时，还存活的对象将被复制到Survivor区（两个中的一个），当一个Survivor区满时，此区的存活对象将被复制到另外一个Survivor区，当另一个Survivor区也满了的时候，从前一个Survivor区复制过来的并且此时还存活的对象，将可能被复制到年老代。  
2. **2个Survivor区是对称的，没有先后关系**，所以同一个Survivor区中可能同时存在从Eden区复制过来对象，和从另一个Survivor区复制过来的对象；而复制到年老区的只有从另一个Survivor区过来的对象。而且，因为需要交换的原因，**Survivor区至少有一个是空的**。特殊的情况下，根据程序需要，**Survivor区是可以配置为多个的**（多于2个），这样可以增加对象在年轻代中的存在时间，减少被放到年老代的可能。  

##### 年老代
在年轻代中经历了N次（可配置）垃圾回收后仍然存活的对象，就会被复制到年老代中。因此，可以认为年老代中存放的都是一些生命周期较长的对象。针对年老代的垃圾回收即 **Full GC**。

##### 持久代
用于存放静态类型数据，如 Java Class, Method 等。持久代对垃圾回收没有显著影响。但是有些应用可能动态生成或调用一些Class，例如 Hibernate CGLib 等，在这种时候往往需要设置一个比较大的持久代空间来存放这些运行过程中动态增加的类型。

##### 内存申请过程  
1. JVM会试图为相关Java对象在年轻代的Eden区中初始化一块内存区域。
2. 当Eden区空间足够时，内存申请结束。否则执行下一步。
3. JVM试图释放在Eden区中所有不活跃的对象（Young GC）。释放后若Eden空间仍然不足以放入新对象，JVM则试图将部分Eden区中活跃对象放入Survivor区。
4. Survivor区被用来作为Eden区及年老代的中间交换区域。当年老代空间足够时，Survivor区中存活了一定次数的对象会被移到年老代。
5. 当年老代空间不够时，JVM会在年老代进行完全的垃圾回收（Full GC）。
6. Full GC后，若Survivor区及年老代仍然无法存放从Eden区复制过来的对象，则会导致JVM无法在Eden区为新生成的对象申请内存，即出现“Out of Memory”。

##### OOM（“Out of Memory”）异常
一般主要有如下2种原因：  
1. 年老代溢出，表现为：java.lang.OutOfMemoryError:Javaheapspace
	- 这是最常见的情况，产生的原因可能是：设置的内存参数Xmx过小或程序的内存泄露及使用不当问题。例如循环上万次的字符串处理、创建上千万个对象、在一段代码内申请上百M甚至上G的内存。还有的时候虽然不会报内存溢出，却会使系统不间断的垃圾回收，也无法处理其它请求。这种情况下除了检查程序、打印堆内存等方法排查，还可以借助一些内存分析工具，比如MAT就很不错。  
2. 持久代溢出，表现为：java.lang.OutOfMemoryError:PermGenspace
	- 通常由于持久代设置过小，动态加载了大量Java类而导致溢出，解决办法唯有将参数 -XX:MaxPermSize 调大（一般256m能满足绝大多数应用程序需求）。将部分Java类放到容器共享区（例如Tomcat share lib）去加载的办法也是一个思路，但前提是容器里部署了多个应用，且这些应用有大量的共享类库。

---
### jvm命令
1. **-Xms**：jvm启动时分配的内存，比如：-Xms200m，表示分配200m内存  
2. **-Xmx**：jvm运行过程中分配的最大内存，比如：-Xmx2048m，表示jvm进程最多只能够占用2048m，程序运行需要占用的内存超出这个设置值，就会抛出OutOfMemory异常 
3. **-Xss**jvm启动的每个线程分配的内存大小，默认jdk1.5+中是1m

三个参数的设置都默认以**Byte**为单位的，也可以在数字后面添加**[k/K]**或者**[m/M]**来表示**KB**或者**MB**

---

| Total | Memory | -Xms | -Xmx | -XssSpare | Memory JDKThread Count |
| --- | --- | --- | --- | --- |  --- |
| 1024M | 256M | 256M | 256K | 768M | 1.43072 |
| 1024M | 256M | 256M | 256K | 768M | 1.5768 |

---
### java.lang.Runtime    
1. **totalMemory()**：java虚拟机现在已经从操作系统那里获取的内存大小，也就是java虚拟机这个进程此时所占用的所有内存。如果在运行java的时候没有添加-Xms参数，那么，在java程序运行的过程的，内存总是慢慢的从操作系统那里挖的，基本上是用多少挖多少，直到挖到maxMemory()为止，所以totalMemory()是慢慢增大的。如果用了-Xms参数，程序在启动的时候就会无条件的从操作系统中挖 -Xms后面定义的内存数，然后在这些内存用的差不多的时候，再去挖。
  
2. **maxMemory()**：jvm进程能够从操作系统那里获取到的最大内存，以Byte为单位，如果在运行java程序的时候，没有设置-Xmx参数，那么jvm默认就是64m，也就是大约64 * 1024 * 1024字节，这就是jvm默认情况下可以从操作系统获取到的最大的内存。如果添加了-Xmx参数，将以该参数后面的值为准。    
 
3. **freeMemory()**：在java程序刚刚启动起来的时候freeMemory()这个方法返回的只有一两兆字节，而随着java程序往前运行，创建了不少的对象，freeMemory()这个方法的返回有时候不但没有减少，反而会增加。如果在运行java的时候没有添加-Xms参数，那么，在java程序运行的过程的，内存总是慢慢的从操作系统那里挖的，基本上是用多少挖多少，但是java虚拟机100％的情况下是会稍微多挖一点的，这些挖过来而又没有用上的内存，实际上就是 freeMemory()，所以freeMemory()的值一般情况下都是很小的，但是如果你在运行java程序的时候使用了-Xms，这个时候因为程序在启动的时候就会无条件的从操作系统中挖-Xms后面定义的内存数，这个时候，挖过来的内存可能大部分没用上，所以这个时候freeMemory()可能会有些大。  

--- 
### 堆大小设置   
jvm中最大堆大小有三方面的限制：  
1. 操作系统的数据模型（32bit还是64bit）限制；  
    - 32位系统下，一般限制在1.5G~2G；  
    - 64位系统对内存无限制；  
2. 系统的可用虚拟内存限制：  
3. 系统的可用物理内存限制： 

---  
### 典型设置  
##### java -Xmx2048m -Xms2048m -Xmn2g -Xss128k  
1. **-Xmx2048m**：设置jvm最大可用内存为2048m。  
2. **-Xms2048m**：设置jvm初始内存为2048m。此值可以设置与-Xmx相同，以避免每次垃圾回收完成后jvm重新分配内存。  
3. **-Xmn2g**：设置年轻代大小为2g。整个堆大小=年轻代大小+年老代大小+持久代大小。持久代一般固定为64m，所以增大年轻代后，将会减小年老代大小。此值对系统性能影响较大，Sun官方推荐配置为整个堆的3/8。  
4. **-Xss128k**：设置每个线程的堆栈大小。JDK5.0以后每个线程堆栈大小为1M，以前每个线程堆栈大小为256K。更具应用的线程所需内存大小进行调整。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右。当这个值被设置的较大（例如>2MB）时将会在很大程度上降低系统的性能。  
##### java -Xmx3550m -Xms3550m -Xss128k -XX:NewRatio=4 -XX:SurvivorRatio=4 -XX:MaxPermSize=16m -XX:MaxTenuringThreshold=0  
1. **-XX:NewRatio=4**：设置年轻代（包括Eden和两个Survivor区）与年老代的比值（除去持久代）。设置为4，则年轻代与年老代所占的比值为1:4，年轻代占整个堆栈的1/5。  
2. **-XX:SurvivorRatio=4**：设置年轻代中Eden区与Survivor区的大小比值。设置为4，则两个Survivor区与一个Eden区的比值为2:4，一个Survivor区占整个年轻代的1/6。  
3. **-XX:MaxPermSize=16m**：设置持久代大小为16m。
4. **-XX:MaxTenuringThreshold=0**：设置垃圾最大年龄。如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代。对于年老代比较多的应用，可以提高效率，如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象在年轻代的存活时间，增加在年轻代即将被回收的概率。  

---
### 回收器选择  
1. **串行收集器**：只适用于**小数据量**的情况，jdk5.0以前默认使用该回收器，如果想使用其他收集器需要在启动时加入相应的参数。jdk5.0以后，jvm会根据当前系统配置进行判断。  
2. **并行收集器**：以**吞吐量优先**的回收器。主要以达到一定的吞吐量为目标，适用于科学技术和后台处理等。  
3. **并发收集器**：以**响应时间优先**的回收器。主要是保证系统的响应时间，减少垃圾收集时的停顿时间。适用于应用服务器、电信领域等。 

--- 
#### 并行收集器
##### 典型配置  
###### java -Xmx3800m -Xms3800m -Xmn2g -Xss128k-XX:+UseParallelGC -XX:ParallelGCThreads=20 -XX:+UseParallelOldGC -XX:MaxGCPauseMillis=100 -XX:+UseAdaptiveSizePolicy
1. **-XX:+UseParallelGC**：选择垃圾收集器为并行收集器。此配置仅对年轻代有效。即上述配置下，年轻代使用并行收集，而年老代仍旧使用串行收集。
2. **XX:ParallelGCThreads=20**：配置并行收集器的线程数，即：同时多少个线程一起进行垃圾回收。此值最好配置与处理器数目相等。
3. **-XX:+UseParallelOldGC**：配置年老代垃圾收集方式为并行收集。jdk6.0支持年老代并行收集。
4. **-XX:MaxGCPauseMillis=100**：设置每次年轻代垃圾回收的最长时间，如果无法满足此时间，jvm会自动调整年轻代大小，以满足此值。
5. **-XX:+UseAdaptiveSizePolicy**：设置此选项后，并行收集器会自动选择年轻代区大小和相应的Survivor区比例，以达到目标系统规定的最低响应时间或者收集频率等，此值建议使用并行收集器时，一直打开。

#### 并发收集器
##### 典型配置
###### java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:ParallelGCThreads=20-XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:CMSFullGCsBeforeCompaction=5 -XX:+UseCMSCompactAtFullCollection
1. **-XX:+UseConcMarkSweepGC**：设置年老代为并发收集。测试中配置这个以后，-XX:NewRatio=4的配置失效了，原因不明。所以，此时年轻代大小最好用-Xmn设置。
2. **-XX:+UseParNewGC**：设置年轻代为并发收集。可与CMS收集同时使用。JDK5.0以上，JVM会根据系统配置自行设置，所以无需再设置此值。
3. **-XX:CMSFullGCsBeforeCompaction=5**：由于并发收集器不对内存空间进行压缩、整理，所以运行一段时间以后会产生“碎片”，使得运行效率降低。此值设置运行多少次GC以后对内存空间进行压缩、整理。
4. **-XX:+UseCMSCompactAtFullCollection**：打开对年老代的压缩。可能会影响性能，但是可以消除碎片。

### 辅助信息
jvm提供了大量命令行参数，打印信息，供调试使用。主要有以下一些：  

- **-XX:+PrintGC**
> 输出形式：  
> [GC 118250K->113543K(130112K), 0.0094143 secs]    
> [Full GC 121376K->10414K(130112K), 0.0650971 secs]    

- **-XX:+PrintGCDetails**
> 输出形式：  
> [GC [DefNew: 8614K->781K(9088K), 0.0123035 secs] 118250K->113543K(130112K), 0.0124633 secs]    
> [GC [DefNew: 8614K->8614K(9088K), 0.0000665 secs][Tenured: 112761K->10414K(121024K), 0.0433488 secs] 121376K->10414K(130112K), 0.0436268 secs]  

- **-XX:+PrintGCTimeStamps**：可与上面两个混合使用
> 输出形式：11.851: [GC 98328K->93620K(130112K), 0.0082960 secs]

- **-XX: +PrintGCApplicationConcurrentTime**:打印每次垃圾回收前，程序未中断的执行时间。可与上面混合使用。
> 输出形式：Application time: 0.5291524 seconds

- **-XX:+PrintGCApplicationStoppedTime**：打印垃圾回收期间程序暂停的时间。可与上面混合使用。
> 输出形式：Total time for which application threads were stopped: 0.0468229 seconds

- **-XX:PrintHeapAtGC**:打印GC前后的详细堆栈信息。
> 输出形式：  
> 34.702: [GC {Heap before gc invocations=7:  
> def new generation   total 55296K, used 52568K [0x1ebd0000, 0x227d0000, 0x227d0000)  
> eden space 49152K, 99% used[0x1ebd0000, 0x21bce430, 0x21bd0000)  
> from space 6144K, 55% used[0x221d0000, 0x22527e10, 0x227d0000)  
> to space 6144K,   0% used [0x21bd0000, 0x21bd0000, 0x221d0000)  
> tenured generation   total 69632K, used 2696K [0x227d0000, 0x26bd0000, 0x26bd0000)  
> the space 69632K,   3% used[0x227d0000, 0x22a720f8, 0x22a72200, 0x26bd0000)  
> compacting perm gen total 8192K, used 2898K [0x26bd0000, 0x273d0000, 0x2abd0000)  
> the space 8192K, 35% used [0x26bd0000, 0x26ea4ba8, 0x26ea4c00, 0x273d0000)  
> ro space 8192K, 66% used [0x2abd0000, 0x2b12bcc0, 0x2b12be00, 0x2b3d0000)  
> rw space 12288K, 46% used [0x2b3d0000, 0x2b972060, 0x2b972200, 0x2bfd0000)  
> 34.735: [DefNew: 52568K->3433K(55296K), 0.0072126 secs] 55264K->6615K(124928K)Heap after gc invocations=8:  
> 
> def new generation   total 55296K, used 3433K [0x1ebd0000, 0x227d0000, 0x227d0000)  
> eden space 49152K,   0% used[0x1ebd0000, 0x1ebd0000, 0x21bd0000)  
> from space 6144K, 55% used [0x21bd0000, 0x21f2a5e8, 0x221d0000)  
> to   space 6144K,   0% used [0x221d0000, 0x221d0000, 0x227d0000)  
> tenured generation   total 69632K, used 3182K [0x227d0000, 0x26bd0000, 0x26bd0000)  
> the space 69632K,   4% used[0x227d0000, 0x22aeb958, 0x22aeba00, 0x26bd0000)  
> compacting perm gen total 8192K, used 2898K [0x26bd0000, 0x273d0000, 0x2abd0000)  
> the space 8192K, 35% used [0x26bd0000, 0x26ea4ba8, 0x26ea4c00, 0x273d0000)  
> ro space 8192K, 66% used [0x2abd0000, 0x2b12bcc0, 0x2b12be00, 0x2b3d0000)  
> rw space 12288K, 46% used [0x2b3d0000, 0x2b972060, 0x2b972200, 0x2bfd0000)}, 0.0757599 secs]

- **-Xloggc:filename**:与上面几个配合使用，把相关日志信息记录到文件以便分析。

---
### 常见配置汇总
#### 堆设置
1. -Xms:初始堆大小  
2. -Xmx:最大堆大小
3. -XX:NewSize=n:设置年轻代大小
4. -XX:NewRatio=n:设置年轻代和年老代的比值。如:为3，表示年轻代与年老代比值为1：3，年轻代占整个年轻代年老代和的1/4
5. -XX:SurvivorRatio=n:年轻代中Eden区与两个Survivor区的比值。注意Survivor区有两个。如：3，表示Eden：Survivor=3：2，一个Survivor区占整个年轻代的1/5
6. -XX:MaxPermSize=n:设置持久代大小
#### 收集器设置  
1. -XX:+UseSerialGC:设置串行收集器
2. -XX:+UseParallelGC:设置并行收集器
3. -XX:+UseParalledlOldGC:设置并行年老代收集器
4. -XX:+UseConcMarkSweepGC:设置并发收集器
#### 垃圾回收统计信息
1. -XX:+PrintGC
2. -XX:+PrintGCDetails
3. -XX:+PrintGCTimeStamps
4. -Xloggc:filename
#### 并行收集器设置
1. -XX:ParallelGCThreads=n:设置并行收集器收集时使用的CPU数。并行收集线程数。
2. -XX:MaxGCPauseMillis=n:设置并行收集最大暂停时间
3. -XX:GCTimeRatio=n:设置垃圾回收时间占程序运行时间的百分比。公式为1/(1+n)
#### 并发收集器设置
1. -XX:+CMSIncrementalMode:设置为增量模式。适用于单CPU情况。
2. -XX:ParallelGCThreads=n:设置并发收集器年轻代收集方式为并行收集时，使用的CPU数。并行收集线程数

---
### 调优总结
#### 年轻代大小选择
1. **响应时间优先的应用**：尽可能设大，直到接近系统的最低响应时间限制（根据实际情况选择）。在此种情况下，年轻代收集发生的频率也是最小的。同时，减少到达年老代的对象。
2. **吞吐量优先的应用**：尽可能的设置大，可能到达Gbit的程度。因为对响应时间没有要求，垃圾收集可以并行进行，一般适合8CPU以上的应用。
#### 年老代大小选择
1. **响应时间优先的应用**：年老代使用并发收集器，所以其大小需要小心设置，一般要考虑并发会话率和会话持续时间等一些参数。如果堆设置小了，可以会造成内存碎片、高回收频率以及应用暂停而使用传统的标记清除方式；如果堆大了，则需要较长的收集时间。最优化的方案，一般需要参考以下数据获得：
	- 并发垃圾收集信息
	- 持久代并发收集次数
	- 传统GC信息
	- 花在年轻代和年老代回收上的时间比例
	- 减少年轻代和年老代花费的时间，一般会提高应用的效率
2. **吞吐量优先的应用**：一般吞吐量优先的应用都有一个很大的年轻代和一个较小的年老代。原因是，这样可以尽可能回收掉大部分短期对象，减少中期的对象，而年老代仅存放长期存活对象。
3. **较小堆引起的碎片问题**：因为年老代的并发收集器使用标记、清除算法，所以不会对堆进行压缩。当收集器回收时，他会把相邻的空间进行合并，这样可以分配给较大的对象。但是，当堆空间较小时，运行一段时间以后，就会出现“碎片”，如果并发收集器找不到足够的空间，那么并发收集器将会停止，然后使用传统的标记、清除方式进行回收。如果出现“碎片”，可能需要进行如下配置：
	- -XX:+UseCMSCompactAtFullCollection：使用并发收集器时，开启对年老代的压缩。
	- -XX:CMSFullGCsBeforeCompaction=0：上面配置开启的情况下，这里设置多少次Full GC后，对年老代进行压缩

