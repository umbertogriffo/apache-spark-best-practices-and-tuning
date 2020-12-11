# Tuning Java Garbage Collection

## **Understanding Memory Management in Spark**

A Resilient Distributed Dataset \(RDD\) is the core abstraction in Spark. Creation and caching of RDD’s closely related to memory consumption. Spark allows users to persistently cache data for reuse in applications, thereby avoid the overhead caused by repeated computing. One form of persisting RDD is to cache all or part of the data in JVM heap. Spark’s executors divide JVM heap space into two fractions: one fraction is used to store data persistently cached into memory by Spark application; the remaining fraction is used as JVM heap space, responsible for memory consumption during RDD transformation. We can adjust the ratio of these two fractions using the **spark.storage.memoryFraction** parameter to let Spark control the total size of the cached RDD by making sure it doesn’t exceed RDD heap space volume multiplied by this parameter’s value.

The unused portion of the RDD cache fraction can also be used by JVM.

Therefore, GC analysis for Spark applications should cover memory usage of both memory fractions.

When an efficiency decline caused by GC latency is observed, we should first check and make sure the Spark application uses the limited memory space in an effective way. The less memory space RDD takes up, the more heap space is left for program execution, which increases GC efficiency; on the contrary, excessive memory consumption by RDDs leads to significant performance loss due to a large number of buffered objects in the old generation.

So when GC is observed as too frequent or long lasting, it may indicate that memory space is not used efficiently by Spark process or application.

**You can improve performance by explicitly cleaning up cached RDD’s after they are no longer needed.**

## **Choosing a Garbage Collector**

The Hotspot JVM version 1.6 introduced the **Garbage-First GC \(G1 GC\)**. The **G1** collector is planned by Oracle as the long term replacement for the **CMS GC**. Most importantly, respect to the **CMS** the **G1** collector aims to achieve both **high throughput** and **low latency**.

Is recommend trying the **G1 GC** because Finer-grained optimizations can be obtained through GC log analysis [\[17\]](https://databricks.com/blog/2015/05/28/tuning-java-garbage-collection-for-spark-applications.html).

To avoid full GC in **G1 GC**, there are two commonly-used approaches:

1. Decrease the **InitiatingHeapOccupancyPercent** option’s value \(the default value is 45\), to let G1 GC starts initial concurrent marking at an earlier time, so that we are more likely to avoid full GC.
2. Increase the **ConcGCThreads** option’s value, to have more threads for concurrent marking, thus we can speed up the concurrent marking phase. Take caution that this option could also take up some effective worker thread resources, depending on your workload CPU utilization.

```javascript
spark.executor.extraJavaOptions = -XX:+UseConcMarkSweepGC -XX:+CMSClassUnloadingEnabled -XX:UseG1GC XX:InitiatingHeapOccupancyPercent=35 -XX:ConcGCThread=20
```

