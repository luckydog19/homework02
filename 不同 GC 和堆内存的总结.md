 #对于不同 GC 和堆内存的总结
 ##一、系统配置说明
 ###本次分析系统如下
处理器：Intel(R) Core(TM) i7-7700HQ CPU @ 2.80GHz   2.80 GHz
机带 RAM：16.0 GB
系统类型：64 位操作系统, 基于 x64 的处理器
##二、各类型GC分析
本次GC分析为学习提供的GCLogAnalysis类运行对各个类型的GC进行分析
JDK版本为：
java version "1.8.0_201"
Java(TM) SE Runtime Environment (build 1.8.0_201-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.201-b09, mixed mode)
###1.默认情况
####1）不指定xmx和xms大小
命令：java -Xloggc:gc.demo.log -XX:+PrintGCDetails GCLogAnalysis
初始化堆内存：-XX:InitialHeapSize=267682048（-Xms256m）
最大堆内存：-XX:MaxHeapSize=4282912768(Xmx:4g)即约默认内存的1/4
GC类型：-XX:+UseParallelGC
共生成对象次数：19249个
发生GC回收次数：16次 用时最短：0.132s 用时最长：1.034s
发生Full GC次数： 4次 用时最短：0.203s 用时最长: 0.915s
####2）指定xmx和xms大小
命令：java -Xms1g -Xmx1g -Xloggc:gc.demo1.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps GCLogAnalysis
GC类型：-XX:+UseParallelGC
共生成对象次数：19192个
发生GC回收次数：36次 用时最短：0.187s 用时最长：1.059s
发生Full GC次数： 3次 用时最短：0.539s 用时最长: 0.980s

较第一次不指定内存大小而言系统默认的而言生成对象个数更少，但是GC发生次数多了20次 但是发生Full GC的时间少了一次

###2.串行GC
命令：java -Xms512m -Xmx512m -Xloggc:gc.demo2.log -XX:+UseSerialGC -XX:+PrintGCDetails GCLogAnalysis
生成对象次数 | GC总次数 | Full GC次数 | 垃圾收集前 | 垃圾收集后 | 年轻代总空间大小 | GC事件持续的时间
-----------|---------|------------|-----------|----------|---------------|---------------
12427|23|1|139776~157247|17472~13977|157248|0.0000102~0.0186404

命令：java -Xms1g -Xmx1g -Xloggc:gc.demoChuanXing1g.log -XX:+UseSerialGC -XX:+PrintGCDetails GCLogAnalysis
生成对象次数 | GC总次数 | Full GC次数 | 垃圾收集前 | 垃圾收集后 | 年轻代总空间大小 | GC事件持续的时间
-----------|---------|------------|-----------|----------|---------------|---------------
19135|17|0|279616~314560|34940~34944|341560|0.0000104~0.0379814
总结：
串行GC中分配的内存越小发生GC的次数较多，Full GC的次数及频率越高相对内存分配的较大给GC发生频率降低，Full GC的发生次数也少，更利于程序的稳定运行


###3.并行GC
命令：java -Xms512m -Xmx512m -Xloggc:gc.demo4.log -XX:+UseParallelGC -XX:+UseParallelOldGC -XX:+PrintGCDetails GCLogAnalysis
生成对象次数 | GC总次数 | Full GC次数 | 垃圾收集前 | 垃圾收集后 | 年轻代总空间大小 | GC事件持续的时间
-----------|---------|------------|-----------|----------|---------------|---------------
8515|37|12|58843~153086|20089~55660|80384~153088|0.0020718~0.0088320

Heap：
 PSYoungGen      total 116736K, used 48984K
  eden space 58880K, 83%
  from space 57856K, 0%
  to   space 57856K, 0%
 ParOldGen       total 349696K, used 338030K
  object space 349696K, 96%
 Metaspace       used 2634K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 291K, capacity 386K, committed 512K, reserved 1048576K

命令：java -Xms1g -Xmx1g -Xloggc:gc.demo4.log -XX:+UseParallelGC -XX:+UseParallelOldGC -XX:+PrintGCDetails GCLogAnalysis
生成对象次数 | GC总次数 | Full GC次数 | 垃圾收集前 | 垃圾收集后 | 年轻代总空间大小 | GC事件持续的时间
-----------|---------|------------|-----------|----------|---------------|---------------
18642|36|3|11673~305663|33888~95834|160256~305664|0.0035406~0.0142670
Heap:
PSYoungGen      total 259584K, used 92327K
  eden space 172032K, 21%
  from space 87552K, 64%
  to   space 86016K, 0%
 ParOldGen       total 699392K, used 379517K
  object space 699392K, 54%
 Metaspace       used 2634K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 291K, capacity 386K, committed 512K, reserved 1048576K

总结：
并发GC分配的内存越小能存储的对象越少，发生GC次数越多越频繁，Full GC的频率越高越连续

###4.CMS GC
命令：java -Xms512m -Xmx512m -Xloggc:gc.demo5.log -XX:+UseConcMarkSweepGC -XX:+PrintGCDetails GCLogAnalysis
生成对象次数 | GC总次数 | Full GC次数 | 垃圾收集前 | 垃圾收集后 | 年轻代总空间大小 | GC事件持续的时间
-----------|---------|------------|-----------|----------|---------------|---------------
12072|27|12|13977~157247|17469~157226|157248|0.0000126~0.0195843

Heap:
 par new generation   total 157248K, used 33070K
  eden space 139776K,  23%
  from space 17472K,   0%
  to   space 17472K,   0%
 concurrent mark-sweep generation total 349568K, used 349480K
 Metaspace       used 2634K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 291K, capacity 386K, committed 512K, reserved 1048576K

命令：java -Xms1g -Xmx1g -Xloggc:gc.demo6.log -XX:+UseConcMarkSweepGC -XX:+PrintGCDetails GCLogAnalysis
生成对象次数 | GC总次数 | Full GC次数 | 垃圾收集前 | 垃圾收集后 | 年轻代总空间大小 | GC事件持续的时间
-----------|---------|------------|-----------|----------|---------------|---------------
16228|18|3|279616~314560|34942~314560|314560|0.0000135~0.0361579

Heap:
 par new generation   total 314560K, used 163414K
  eden space 279616K,  45%
  from space 34944K,  99%
  to   space 34944K,   0%
 concurrent mark-sweep generation total 699072K, used 431810K
 Metaspace       used 2634K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 291K, capacity 386K, committed 512K, reserved 1048576K
总结：
cms创建对象数量不是很多，吞吐量不如并发甚至串行收集器，垃圾回收时间与并行回收相差无异。
###5.G1 GC
命令：java -Xms512m -Xmx512m -Xloggc:gc.demoG1512m.log -XX:+UseG1GC -XX:+PrintGCDetails GCLogAnalysis
G1纯年轻代转移暂停工发生91次，
最早一次发生在JVM启动后0.109秒开始，持续时常为0.0034212秒；后面活动由8个Worker线程并行执行，耗时1.5毫秒；
最后一次发生在JVM启动后1.084秒开始，持续时长为0.0027422秒；后面活动由8个Worker线程并行执行，耗时1.8毫秒；
Heap：
 garbage-first heap   total 524288K, used 376854K
  region size 1024K, 11 young (11264K), 1 survivors (1024K)
 Metaspace       used 2634K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 291K, capacity 386K, committed 512K, reserved 1048576K

命令：java -Xms1g -Xmx1g -Xloggc:gc.demoG11g.log -XX:+UseG1GC -XX:+PrintGCDetails GCLogAnalysis
G1纯年轻代转移暂停共发生33次，
最早一次发生在JVM启动后0.139秒开始，持续时常为0.0042765秒；后面活动由8个Worker线程并行执行，耗时2.4毫秒；
最后一次发生在JVM启动后1.104秒开始，持续时长为0.0070605秒；后面活动由8个Worker线程并行执行，耗时6.3毫秒；

Heap
 garbage-first heap   total 1048576K, used 623129K
  region size 1024K, 7 young (7168K), 6 survivors (6144K)
 Metaspace       used 2634K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 291K, capacity 386K, committed 512K, reserved 1048576K
总结：
在扩大内存后G1 GC发生频率大大降低，但是相对每次执行耗时时间较长，这样每次花费在移动暂停发生的时间就会更长


