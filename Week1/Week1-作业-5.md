## Week1-作业-5

### 题目

> 本机使用 G1 GC 启动一个程序，仿照课上案例分析一下 JVM 情况。

### 分析

#### 启动参数

为了方便测出Mixed GC，堆内存故意调的比较小
```
java -Xmx90m -Xms90m -XX:-UseAdaptiveSizePolicy -XX:+UseG1GC -XX:MaxGCPauseMillis=50 -jar gateway-server-0.0.1-SNAPSHOT.jar
```

#### 堆内存情况

**分析**
```
可以看到开启的是G1 GC并且分配了10个线程，
虽然默认是分配1024个region，但因为单个分配1m，总的只有90m，所以这里只有90个region，
目前已分配53个eden区，4个s区，16个old区，还有剩下的就是待分配区，
关注一个参数，o区目前占用15m左右，90m的45%大概40.5m，待会压测时关注是不是这时候会触发Mixed GC
```

```
Attaching to process ID 31177, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 11.0.11+9-LTS-194

using thread-local object allocation.
Garbage-First (G1) GC with 10 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 40
   MaxHeapFreeRatio         = 70
   MaxHeapSize              = 94371840 (90.0MB)
   NewSize                  = 1363144 (1.2999954223632812MB)
   MaxNewSize               = 56623104 (54.0MB)
   OldSize                  = 5452592 (5.1999969482421875MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 1048576 (1.0MB)

Heap Usage:
G1 Heap:
   regions  = 90
   capacity = 94371840 (90.0MB)
   used     = 54910048 (52.366302490234375MB)
   free     = 39461792 (37.633697509765625MB)
   58.184780544704864% used
G1 Young Generation:
Eden Space:
   regions  = 33
   capacity = 55574528 (53.0MB)
   used     = 34603008 (33.0MB)
   free     = 20971520 (20.0MB)
   62.264150943396224% used
Survivor Space:
   regions  = 4
   capacity = 4194304 (4.0MB)
   used     = 4194304 (4.0MB)
   free     = 0 (0.0MB)
   100.0% used
G1 Old Generation:
   regions  = 16
   capacity = 34603008 (33.0MB)
   used     = 16112736 (15.366302490234375MB)
   free     = 18490272 (17.633697509765625MB)
   46.56455300071023% used
```


#### GC情况

**压测参数**
```
wrk -t12 -c1000 -d60s http://localhost:8088/api/hello
```

![](http://106.15.233.185:8983/fe077dba-2331-45ae-b516-7dafa772774a.jpg)

**分析**

```
压测开始时可以看到YGC显著增多，大概在o区到37964字节（约37m），由于日志1s打印一次，错过了45%的峰值，
但是差不多可以估摸出这过程中达到45%的限制，开始触发Mixed GC，
还有再看GC的时间，无论YGC的均值，还是CGC的均值，均都控制在个位数毫秒内，是相当优秀了。
```