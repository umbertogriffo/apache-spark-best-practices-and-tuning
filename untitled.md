# Use the right level of parallelism

Clusters will not be fully utilized unless the level of parallelism for each operation is high enough. Spark automatically sets the number of partitions of an input file according to its size and for distributed shuffles. When you use a **DataFrame** or a **Dataset,** it creates several partitions equal to **spark.sql.shuffle.partitions** parameter with a default value of 200. Most of the time works well, but when the dataset is starting to be massive, you can do better using the **repartition** function to balance the dataset across the workers.

The below picture I made inspired by \[25\] shows how to use the **Spark UI** to answer to the following questions:

· Any outlier in task execution?

· Skew in data size, compute time?

· Too many/few tasks \(partitions\)?

· Load balanced?

![](https://cdn-images-1.medium.com/max/2400/1*A3qKcL-ihUTMsH53HbJuGQ.png)

You can tune the number of partitions, asking those questions. The above example is a best-case scenario where all the tasks are balanced, and there isn’t skew in data size. In our specific use case, where we are dealing with **billions of rows**, we have found that partitions in the range of **10k** work most efficiently. Suppose we have to apply an aggregate function like a **groupBy** or a **Window function** on a dataset containing a **time-series** of billions of rows, and the column _id_ is a unique identifier, and the column _timestamp_ is the time. In order to well-balance the data, you can repartition properly the **DataFrame/Dataset** as follows:

```scala
val repartitionedDf = originalSizeDf
.repartition(10000, 
             Seq(col("id"), 
                 col("timestamp")): _*)
```

This repartition basically will balance the dataset and the load on the workers speeding up the pipeline. Another rule of thumb \[26\] is that tasks should take at most 100 ms to execute. You can ensure that this is the case by monitoring the task duration from the **Spark UI**. If your tasks take considerably longer than that keep increasing the level of parallelism, by say 1.5, until performance stops improving.

####  

