# Joining a large and a small RDD

If the small RDD is small enough to fit into the memory of each worker we can turn it into a broadcast variable and turn the entire operation into a so called map side join for the larger RDD [\[23\]](http://fdahms.com/2015/10/04/writing-efficient-spark-jobs/). In this way the larger RDD does not need to be shuffled at all. This can easily happen if the smaller RDD is a dimension table.

```scala
val smallLookup = sc.broadcast(smallRDD.collect.toMap)
largeRDD.flatMap { case(key, value) =>
  smallLookup.value.get(key).map { otherValue =>
    (key, (value, otherValue))
  }
}
```

