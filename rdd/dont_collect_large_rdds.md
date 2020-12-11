# Don’t collect large RDDs

When a collect operation is issued on a RDD, the dataset is copied to the driver, i.e. the master node. A memory exception will be thrown if the dataset is too large to fit in memory; `take`or `takeSample`can be used to retrieve only a capped number of elements instead.

Another way has been showed in [\[8\]](http://stackoverflow.com/questions/21698443/spark-best-practice-for-retrieving-big-data-from-rdd-to-local-machine) where you can get the array of partition indexes:

```scala
val parallel = sc.parallelize(1 to 9)
val parts = parallel.partitions
```

and then create a smaller rdd filtering out everything but a single partition. Collect the data from smaller rdd and iterate over values of a single partition:

```scala
for(p <- parts){
  val idx = p.index
  val partRDD = parallel.mapPartitionsWithIndex((index: Int, it: Iterator[Int]) => if(index == idx) it else Iterator(), true)
  val data = partRDD.collect
  // Data contains all values from a single partition in the form of array.
  // Now you can do with the data whatever you want: iterate, save to a file, etc.
}

// You can use also the foreachPartition operation
parallel.foreachPartition(partition => {
  partition.toArray
  // Your code
})
```

Of cause, it will work only if the partitions are small enough. If they aren't, you can always increase the number of partitions with `rdd.coalesce(numParts, true)`.

