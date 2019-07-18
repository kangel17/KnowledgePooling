### Java监控工具
**jcmd**：打印java进程涉及的基本类、线程和vm信息  
**jconsole**：提供JVM活动的图形化视图，包括线程的使用，类的使用和GC活动  
**jhat**：读取内存堆转储，并有助于分析  
**jmap**：提供堆转储和其他JVM内存使用的信息  
**jinfo**：查看JVM的系统属性，可以动态设置的一些系统属性  
**jstack**：转储java进程的栈信息  
**jstat**：提供GC和类装载活动的信息  
**jvisualvm**：监视JVM的GUI工具，可以用来剖析运行的应用，分析JVM堆转储  

### dump生成
dump可以是内存溢出时让其自动生成，或者手工直接导。  
1. **自动生成**
	配置jvm参数：-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/home/biapp/m.hprof  
2. **手工直接导**，PID为进程号    
	jmap -dump:live,format=b,file=m.hprof PID

### Shallow Size、Retained Size、Heap Size 和 Allocated
#### Shallow Size
Shallow Size是对象本身占据的内存的大小，不包含其引用的对象。对于常规对象（非数组）的Shallow Size由其成员变量的数量和类型来定，而数组的ShallowSize由数组类型和数组长度来决定，它为数组元素大小的总和。

#### Retained Size
Retained Size=当前对象大小+当前对象可直接或间接引用到的对象的大小总和。(间接引用的含义：A->B->C,C就是间接引用) ，并且**排除被GC Roots直接或者间接引用的对象**

换句话说，Retained Size就是当前对象被GC后，从Heap上总共能释放掉的内存。 
不过，释放的时候还要排除被GC Roots直接或间接引用的对象。他们暂时不会被被当做Garbage。  

```flow  
st=>start: Start|past:>http://www.google.com[blank]
e=>end: End:>http://www.google.com
op1=>operation: My Operation|past
op2=>operation: Stuff|current
sub1=>subroutine: My Subroutine|invalid
cond=>condition: Yes
or No?|approved:>http://www.google.com
c2=>condition: Good idea|rejected
io=>inputoutput: catch something…|request

st->op1(right)->cond
cond(yes, right)->c2
cond(no)->sub1(left)->op1
c2(yes)->io->e
c2(no)->op2->e
```
