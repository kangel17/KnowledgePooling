```
java.util.Random
java.util.concurrent.ThreadLocalRandom
java.security.SecureRandom
java.util.SplittableRandom
```

#### Random  
> 最常用的就是Random，适用于绝大部分场景。  
用来生成**伪随机数**，默认使用**48位种子**、**线性同余公式**进行修改。我们可以通过构造器传入初始seed，或者通过setSeed重置（同步）。默认seed为系统时间的纳秒数，真大！  
如果两个（多个）不同的Random实例，使用相同的seed，按照相同的顺序调用相同方法，那么它们得到的数字序列也是相同的。这看起来不太随机。  这种设计策略，既有优点也有缺点，优点是“相同seed”生成的序列是一致的，使过程具有可回溯和校验性（平台无关、运行时机无关）；缺点就是，这种一致性，潜在引入其“可被预测”的风险。  
Random的实例是**线程安全**的。  但是，跨线程并发使用相同的java.util.Random实例可能会遇到争用，从而导致性能稍欠佳（nextX方法中，在对seed赋值时使用了CAS，测试结果显示，其实性能损耗很小）。 请考虑在多线程设计中使用ThreadLocalRandom。同时，我们在并发环境下，也没有必要刻意使用多个Random实例。  
Random实例不具有加密安全性。  相反，请考虑使用SecureRandom来获取加密安全的伪随机数生成器，以供安全敏感应用程序使用。  

#### 性能检测
简析，基准：100000随机数，单线程

1. Random ：2毫秒
2. ThreadLocalRandom ：1毫秒
3. SecureRandom
	1）默认算法，即SHAR1PRNG：80毫秒左右。
	2）NativePRNG：90毫秒左右。
4. SplittableRandom ：1毫秒


平常使用Random，或者大多数时候使用，都是没有问题的，它也是线程安全的。SplittableRandom是内部使用的类，应用较少，即使它也是public的也掩饰不了偏门。ThreadLocalRandom是为了在高并发环境下节省一点细微的时间，追求性能的应用推荐使用。而对于有安全需求的，又希望更随机一些的，用SecureRandom再好不过了
