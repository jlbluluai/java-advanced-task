## Week1-作业-4

### 题目

> 检查一下自己维护的业务系统的 JVM 参数配置，用 jstat 和 jstack、jmap 查看一下详情，并且自己独立分析一下大概情况，思考有没有不合理的地方，如何改进。

### 分析

简介：个人云服务器1核心1G

```
由于服务器太小，还需要供给部分其他服务，故指定了最大堆内存128M，未指定GC，
按JDK8的默认应该是Pararllel GC，但实际情况可能因为只检测到一核心，退化为Serial GC。

总体来说，个人使用暂时没啥问题，看GC情况，Eden区因为很小，很容易满就触发YGC，YGC平均10ms，也不算太慢，
实际观测服务器空闲内存还有200m，可以稍微再匀出80m空间给堆内存减缓YGC的频率。不过FGC就有点慢了，均150ms以上，
若是真正给用户用的系统，将会非常影响体验。

分析stack就发现问题了，平常自己写东西时，开启了很多实际没有使用的线程池，
而我们知道，一个线程栈默认分配1M空间，这就让本不富裕的内存空间雪上加霜了，决定清理遍代码，关闭不必要的线程池。
```

#### 启动参数

```
java -Xmx128m -Xms128m -jar -Dlogging.path=/usr/app/logs/linzone -Dspring.profiles.active=prod -DServer.port=8080 /usr/app/target/linzone-1.0.0.jar
```

#### 堆内存占用

```
Attaching to process ID 2053, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.211-b12

using thread-local object allocation.
Mark Sweep Compact GC

Heap Configuration:
   MinHeapFreeRatio         = 40
   MaxHeapFreeRatio         = 70
   MaxHeapSize              = 134217728 (128.0MB)
   NewSize                  = 44695552 (42.625MB)
   MaxNewSize               = 44695552 (42.625MB)
   OldSize                  = 89522176 (85.375MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
New Generation (Eden + 1 Survivor Space):
   capacity = 40239104 (38.375MB)
   used     = 28096640 (26.7950439453125MB)
   free     = 12142464 (11.5799560546875MB)
   69.82421875% used
Eden Space:
   capacity = 35782656 (34.125MB)
   used     = 25969680 (24.766616821289062MB)
   free     = 9812976 (9.358383178710938MB)
   72.57616650927197% used
From Space:
   capacity = 4456448 (4.25MB)
   used     = 2126960 (2.0284271240234375MB)
   free     = 2329488 (2.2215728759765625MB)
   47.72769703584559% used
To Space:
   capacity = 4456448 (4.25MB)
   used     = 0 (0.0MB)
   free     = 4456448 (4.25MB)
   0.0% used
tenured generation:
   capacity = 89522176 (85.375MB)
   used     = 60099896 (57.31572723388672MB)
   free     = 29422280 (28.05927276611328MB)
   67.1340875360313% used
```

#### gc情况

![](http://106.15.233.185:8983/aab7561d-6634-44ac-9936-3f5c60d0aca6.jpg)

#### jstack情况

./resource/2053_stack.txt
