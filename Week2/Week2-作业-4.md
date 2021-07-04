## Week2-作业-4

### 题目

> 对于不同 GC 和堆内存的总结。

### 环境

涉及数据分析相关，均基于2.6 GHz 六核Intel Core i7以及JDK8

### 总结

#### Serial GC（串行GC）

单核、内存小优先考虑该种策略（个人买的学习、玩玩的小服务器），无论young区还是old区的GC均只启动一个线程，并且全程STW。

YGC采用mark-copy算法，FGC采用mark-sweep-compact算法。

##### 通过GCLogAnalysis分析

**堆内存512m**

小内存下，Serial GC还是相当优异的，均生成对象在10000左右，每次YGC的时间二三十毫秒。

**堆内存4g**

大内存下，Serial GC的劣势立马就出来了，均生成对象直接掉到8500左右，观察GC日志，没有出现FC并且YGC次数大幅度降低，但是每次YGC的时间直接到150多毫秒，这也是严重降低1s内创建对象数量的根本原因。

---


#### Parallel GC（并行GC）

JDk8的默认GC策略，无论young区还是old区的GC都会开启多个线程（默认CPU核心数，-XX:ParallelGCThreads=N可以设置），并且全程STW。

YGC采用mark-copy算法，FGC采用mark-sweep-compact算法，总体来看就可以看作是Serial GC的多线程版，适应CPU越来越强的新时代。

##### 通过GCLogAnalysis分析

**堆内存512m**

小内存下，Parallel GC均生成对象9500左右（这就奇怪了，还不如Serial GC？），实际上观察堆内存使用情况会发现，Parallel下survivor区分配的大小比Serial大（分配算法问题），
因此eden区大小相对就小，这样发生YGC的上限就低了，所以1s内会出现更多次的YGC造成总GC时长相较Serial多一点


**堆内存4g**

大内存下，Parallel的多线程优势立马显现出来，均生成对象达到11000，观察GC日志，发现只有少量YGC，并且时间还能控制在100ms左右。


---


#### CMS GC（Concurrent Mark Sweep GC）

young区采用并行STW的mark-copy算法。

old区采用并发mark-sweep算法，分为7个阶段：

1. Initial Mark（初始标记） STW
2. Concurrent Mark（并发标记）
3. Concurrent Preclean（并发预清理）
4. Concurrent Abortable Preclean（可取消的并发预清理）
5. Final Remark（最终标记） STW
6. Concurrent Sweep（并发清除）
7. Concurrent Reset（并发重置）

需注意，由于没有整理，CMS GC old区会有很多内存碎片，这样一不注意就会引发Full GC（Serial模式），这将是灾难的。因此用CMS，首先内存不能太小，
其次还要配合参数做好控制：

- -XX:CMSInitiatingOccupancyFraction=N（设定内存占用率达到多少开始CGC，调小这样CGC开始的早，old不容易满，减低FGC的可能性） -XX:+CMSScavengeBeforeRemark（设定触发CGC的阈值固定，也就是前面那个值，不设定，默认会自动调整，也就是只配前面那个等于没用）
- -XX:+UseCMSCompactAtFullCollection（设定Full GC时作整理，默认就是开启的） -XX:CMSFullGCBeforeCompaction=N（设定每N次Full GC后才进行整理，默认是0，也就是每次Full GC都会整理。在内存碎片化和Full GC时长间作取舍，可以适当调整）


##### 通过GCLogAnalysis分析

**堆内存512m**

小内存下，CMS GC均生成对象10000，每次YGC的时间二三十毫秒，观测堆内存分配算法和Serial一致，所以eden区没有Parallel GC在本身内存就小还要低点的尴尬。
由于内存小，确实出现了几次concurrent mode failure，这时候退化为Serial的Full GC，还是很影响性能的。

**堆内存4g**

大内存下，CMS GC均生成对象12000，也没有了Full GC的表现，并发的优势立马显现出来，


---


#### G1 GC（Garbage First GC）

堆不再划分成young区和old区，而是划分成2048个（通常）region（小块堆区域），每个region一会是eden，一会是survivor，一会是old。

young区采用并行STW的mark-copy算法。

old区采用增量方式处理，总体来说分来两大阶段并发标记阶段和mixed GC，

并发标记阶段分为5个小步骤：

1. 初始标记（Initial Mark）
2. Root区扫描（Root Region Scan）
3. 并发标记（Concurrent Mark）
4. 再次标记（Remark） STW
5. 清理（Cleanup） 短暂STW

mixed GC阶段一旦开启，就取代了young GC，相较于young GC只清理young区，mixed GC还会附带清理一部分old区。直到满足可回收占比小于5%（-XX:G1HeapWastePercent控制这个比例），mixed GC才会退回为young gc。

G1 GC和CMS一样作为并发的GC，也会面临一个问题，内存真正满额不够时会引发串行化的Full GC，当然由于G1是会整理内存碎片的，真正达到满额的概率是远低于CMS GC的，具体出现问题还得结合其他指标一起分析。


##### 通过GCLogAnalysis分析

**堆内存512m**

小内存下，G1 GC均生成对象10000，粗略的观察了下，无论young gc还是mixed gc都在几毫秒，真不愧是后面版本jdk的默认GC算法。
全程就出现了一次Full GC，比CMS的数次在小内存下也优异的多。

**堆内存4g**

大内存下，G1 GC均生成对象12500，young gc均20毫秒，没有出现转移成mixed GC，看着倒是和其他算法没太大差距，主要是没测出G1对old区的处理。


#### 汇总

- Serial GC: 单核、内存小的首选，硬件配置已经这么低了，就别想着什么高端的玩法了。
- Parallel GC：jdk8默认GC算法，追求吞吐量的，可以考虑。
- CMS GC：并发模式的GC算法，追求响应速度可以考虑，但内存碎片是个问题，内存太小频繁Serial的Full GC就得不偿失了。
- G1 GC：可以看作是CMS GC的升级版算法，硬件越优秀它表现的越优异，合理运用基本满足大多数的生产业务系统的需求了。