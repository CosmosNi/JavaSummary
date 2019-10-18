# JVM常用调优参数

###### 1.1 堆参数

| 参数              | 描述                                              |
| ----------------- | ------------------------------------------------- |
| -Xms              | 设置JVM启动时堆内存的初始化大小                   |
| -Xmx              | 设置堆内存最大值                                  |
| -Xmn              | 设置年轻代的空间大小，剩下的为老年代的空间大小    |
| -XX:PermGen       | 设置永久代内存的初始化大小（1.8中废弃永久代）     |
| -XX:MaxPermGen    | 设置永久代的最大值                                |
| -XX:SurvivorRatio | 设置Eden区和Survivor区的空间比例：Eden/S0=Eden/S1 |
| -XX:NewRatio      | 设置年老代和年轻代的比例大小，默认值为2           |



###### 1.2 回收器参数

| 参数                   | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| -XX:+UseSerialGC       | 串行，Young区和Old区都使用串行，使用复制算法回收，逻辑简单高效，无线程切换开销。 |
| -XX:+UseParallelGC     | 并行，Young区:使用Parallel scavenge回收算法，会产生多个线程并行回收。通过-XX:ParallelGCThreads=n 参数指定有线程数，默认是CPU核数；Old区：单线程。 |
| -XX:+UseParallelOldGC  | 并行，和UseParallelGC一样，Young区和Old区的垃圾回收时都用多线程收集。 |
| -XX:UseConcMarkSweepGC | 并发，短暂停顿的并发的收集。Young区：可以使用普通的或者parallel垃圾回收算法，由参数-XX:+UseParNewGC来控制；Old区：只能使用Concurrent Mark Sweep |
| -XX:UseG1GC            | 并行的，并发的和增量式压缩短暂停顿的垃圾收集器。不区分Young区和Old区空间。它把堆空间划分为多个大小相等的区域。优先收集存活对象较少的区域。 |



###### 1.3 项目常用配置

| 参数设置                              | 描述                              |
| ------------------------------------- | --------------------------------- |
| -Xms4800m                             | 初始化堆空间大小                  |
| -Xmx4800m                             | 最大堆空间大小                    |
| -Xmn1800m                             | 年轻代的空间大小                  |
| -Xss512k                              | 设置线程栈的空间大小              |
| -XX:PermSize=256m                     | 永久区空间大小（1.8废弃）         |
| -XX:MaxPermSize=256m                  | 最大永久区空间大小                |
| -XX:+UseStringCache                   | 默认开启，启用缓存常用的字符串    |
| -XX:+UseConcMarkSweepGC               | 老年代使用CMS收集器               |
| -XX:+UseParNewGC                      | 新生代使用并行收集器              |
| -XX:ParallelGCThreads=4               | 并行线程数量4                     |
| -XX:+CMSClassUnloadingEnabled         | 允许对类的元数据进行清理          |
| -XX:+DisableExplicitGC                | 禁止显示的GC                      |
| -XX:+UseCMSInitiatingOccupancyOnly    | 表示只有大的阈值之后才进行CMS回收 |
| -XX:CMSInitiatingOccupancyFraction=68 | 设置CMS在老年代回收的阈值为68%    |
| -verbose:gc                           | 输出虚拟机GC详情                  |
| -XX:+PrintGCDetails                   | 打印GC详情日志                    |
| -XX:+PrintGCDateStamps                | 答应GC的耗时                      |
| -XX:+PrintTenuringDistribution        | 答应Tenuring年龄信息              |
| -XX:+HeadDumpOnOutOfMemoryError       | 当抛出OOM时进行HeadDump           |
| -XX:HeadDumpPath=/home/admin/logs     | 指定HeadDump的文件路径或目录      |



###### 1.4 常用参数组合

| young                    | old                 | JVM参数                                          |
| ------------------------ | ------------------- | ------------------------------------------------ |
| Serial                   | Serial              | -XX:+UseSerialGC                                 |
| Parallel scavenge        | Parallel Old/Serial | -XX:+UseParallelGC   <br />-XX:+UseParallelOldGC |
| Serial/Parallel scavenge | CMS                 | -XX:+UseParNewGC<br />-XX:+UseConcMarkSweepGC    |
| G1                       |                     | -XX:+UseG1Gc                                     |



###### 1.5 GC调优策略

1. 将新对象预留在新生代。Full GC的成本远高于Minor GC

2. 大对象进入老年代。XX:PretenureSizeThreshold 可以设置直接进入老年代的对象大小。

3. 合理设置进入老年代对象的年龄，-XX:MaxTenuringThreshold 设置对象进入老年代的年龄大小，减少老年代的内存占用，降低 full gc 发生的频率。

4. 设置稳定的堆大小，堆大小设置有两个参数：-Xms 初始化堆大小，-Xmx 最大堆大小。

5. 注意：如果满足下面的指标，**则一般不需要进行 GC 优化：**

   > MinorGC 执行时间不到50ms；
   >
   > Minor GC 执行不频繁，约10秒一次；
   >
   > Full GC 执行时间不到1s；
   >
   > Full GC 执行频率不算频繁，不低于10分钟1次。