# Cache Judiciously and use Checkpointing

Just because you can cache an **RDD**, a **DataFrame**, or a **Dataset** in memory doesn’t mean you should blindly do so. Depending on how many times the dataset is accessed and the amount of work involved in doing so, recomputation can be faster than the price paid by the increased memory pressure. If the data pipeline is long, so we have to apply several transformations on a huge dataset without allocating an expensive cluster, a **checkpoint** could be your best friend. Here is an example of a query plan:

```scala
== Physical Plan ==
TakeOrderedAndProject(limit=1001, 
orderBy=[avg(arrdelay)#304 DESC NULLS LAST], 
output=[src#157,dst#149,avg(arrdelay)#314])
+- *(2) HashAggregate(keys=[destination#157, arrdelay#149],
       functions=[avg(arrdelay#152)],
       output=[destination#157, avg(arrdelay)#304])
+- Exchange hashpartitioning(destination#157, 200)
+- *(1) HashAggregate(keys=[destination#157],
              functions=[partial_avg(arrdelay#152)],  
              output=[destination#157, sum#321, count#322L])
+- *(1) Project[arrdelay#152, destination#157]
+- *(1) Filter (isnotnull(arrdelay#152) && (arrdelay#152 > 1.0))
+- *(1) FileScan parquet default.flights[arrdelay#152,destination#157] 
Batched: true, 
Format: Parquet, 
Location: PrunedInMemoryFileIndex[dbfs:/data/vehicles/destination=BOS], 
PartitionCount: 1, 
PartitionFilters: [isnotnull(destination#157), (destination#157 = BOS)], 
PushedFilters: [IsNotNull(arrdelay), GreaterThan(arrdelay,1.0)],
ReadSchema: struct<destination:string,arrdelay>
```

`Exchange` means a shuffle occurred between stages, while `HashAggregate` means an aggregation occurred. If the query contains a **join** operator, you could see `ShuffleHashJoin`, `BroadcastHashJoin` \(If one of the datasets is small enough to fit in memory\), or `SortMergeJoin` \(inner joins\). **Checkpoint** saves the data on disk truncating the query plan, and this is a nice feature because each time you apply a transformation or perform a query on a **Dataset**, the query plan grows. When the query plan starts to be huge, the performances decrease dramatically, generating bottlenecks. In order to avoid the **exponential growth** of query lineage, we can add **checkpoints** in some strategic points of the data pipeline. But how can we understand where to put them? A possible rule of dumb could be defining a **complexity score** as the deepness of the query plan. Each time an `Exchange`**,** `HashAggregate`**,** `ShuffleHashJoin`**,** `BroadcastHashJoin`**,** `SortMergeJoin` occurred, we add one point to the complexity score. Each time the sum of each of them is greater or equal to 9, do a checkpoint.

![](https://cdn-images-1.medium.com/max/1600/1*Ft5gWp7WLNYFlsjmw_Fqig.png)

In addition, using checkpointing will help you to debug the data pipeline because you will know precisely the status of the job.

## Which storage level to choose

By default Spark will **cache\(\)** data using **MEMORY\_ONLY** level,  
**MEMORY\_AND\_DISK\_SER** can help cut down on GC and avoid expensive recomputations.

* MEMORY\_ONLY
  * Store RDD as deserialized Java objects in the JVM. If the RDD does not fit in memory, some partitions will not be cached and will be recomputed on the fly each time they're needed. This is the default level.
* MEMORY\_AND\_DISK
  * Store RDD as deserialized Java objects in the JVM. If the RDD does not fit in memory, store the partitions that don't fit on disk, and read them from there when they're needed.
* MEMORY\_ONLY\_SER
  * Store RDD as serialized Java objects \(one byte array per partition\). This is generally more space-efficient than deserialized objects, especially when using a fast serializer, but more CPU-intensive to read.
* MEMORY\_AND\_DISK\_SER
  * Similar to MEMORY\_ONLY\_SER, but spill partitions that don't fit in memory to disk instead of recomputing them on the fly each time they're needed.
* DISK\_ONLY
  * Store the RDD partitions only on disk.
* MEMORY\_ONLY\_2, MEMORY\_AND\_DISK\_2, etc.
  * Same as the levels above, but replicate each partition on two cluster nodes.
* OFF\_HEAP \(experimental\)
  * Store RDD in serialized format in Tachyon. Compared to MEMORY\_ONLY\_SER, OFF\_HEAP reduces garbage collection overhead and allows executors to be smaller and to share a pool of memory, making it attractive in environments with large heaps or multiple concurrent applications. Furthermore, as the RDDs reside in Tachyon, the crash of an executor does not lead to losing the in-memory cache. In this mode, the memory in Tachyon is discardable. Thus, Tachyon does not attempt to reconstruct a block that it evicts from memory.

