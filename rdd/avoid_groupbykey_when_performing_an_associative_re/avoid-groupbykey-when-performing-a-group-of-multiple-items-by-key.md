# Avoid groupByKey when performing a group of multiple items by key

As already showed in[ \[21\]](http://stackoverflow.com/questions/36447057/spark-group-multiple-rdd-items-by-key) let's suppose we've got a RDD items like:

```text
(3922774869,10,1)
(3922774869,11,1)
(3922774869,12,2)
(3922774869,13,2)
(1779744180,10,1)
(1779744180,11,1)
(3922774869,14,3)
(3922774869,15,2)
(1779744180,16,1)
(3922774869,12,1)
(3922774869,13,1)
(1779744180,14,1)
(1779744180,15,1)
(1779744180,16,1)
(3922774869,14,2)
(3922774869,15,1)
(1779744180,16,1)
(1779744180,17,1)
(3922774869,16,4)
...
```

which represent **\(id, age, count\)** and we want to group those lines to generate a dataset for which each line represent the distribution of age of each id like this \(\(id, age\) is unique\):

```text
(1779744180, (10,1), (11,1), (12,2), (13,2) ...)
(3922774869, (10,1), (11,1), (12,3), (13,4) ...)
```

which is **\(id, \(age, count\), \(age, count\) ...\)**

The easiest way is first reduce by both fields and then use groupBy:

```scala
rdd
.map { case (id, age, count) => ((id, age), count) }.reduceByKey(_ + _)
.map { case ((id, age), count) => (id, (age, count)) }.groupByKey()
```

Which returns an `RDD[(Long, Iterable[(Int, Int)])]`, for the input above it would contain these two records:

```text
(1779744180,CompactBuffer((16,3), (15,1), (14,1), (11,1), (10,1), (17,1)))
(3922774869,CompactBuffer((11,1), (12,3), (16,4), (13,3), (15,3), (10,1), (14,5)))
```

But if you have a **very large dataset**, in order to reduce shuffling, you should not to use `groupByKey`.

Instead you can use `aggregateByKey`:

```scala
import scala.collection.mutable

val rddById = rdd.map { case (id, age, count) => ((id, age), count) }.reduceByKey(_ + _)
val initialSet = mutable.HashSet.empty[(Int, Int)]
val addToSet = (s: mutable.HashSet[(Int, Int)], v: (Int, Int)) => s += v
val mergePartitionSets = (p1: mutable.HashSet[(Int, Int)], p2: mutable.HashSet[(Int, Int)]) => p1 ++= p2
val uniqueByKey = rddById.aggregateByKey(initialSet)(addToSet, mergePartitionSets)
```

This will result in:

```java
uniqueByKey: org.apache.spark.rdd.RDD[(AnyVal, scala.collection.mutable.HashSet[(Int, Int)])]
```

And you will be able to print the values as:

```java
scala> uniqueByKey.foreach(println)
(1779744180,Set((15,1), (16,3)))
(1779744180,Set((14,1), (11,1), (10,1), (17,1)))
(3922774869,Set((12,3), (11,1), (10,1), (14,5), (16,4), (15,3), (13,3)))
```

Shuffling can be a great bottleneck. Having many big HashSet's \(according to your dataset\) could also be a problem. However, it's more likely that you'll have a large amount of ram than network latency which results in faster reads/writes across distributed machines.

Here are more functions to prefer over`groupByKey`:

* `combineByKey` can be used when you are combining elements but your return type differs from your input value type. You can see an example [here](http://codingjunkie.net/spark-combine-by-key/)
* `foldByKey` merges the values for each key using an associative function and a neutral "zero value".

