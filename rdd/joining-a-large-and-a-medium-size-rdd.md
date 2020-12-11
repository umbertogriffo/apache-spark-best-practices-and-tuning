# Joining a large and a medium size RDD

If the medium size RDD does not fit fully into memory but its key set does, it is possible to exploit this [\[23\]](https://www.gitbook.com/book/umbertogriffo/apache-spark-best-practices-and-tuning/edit#). As a join will discard all elements of the larger RDD that do not have a matching partner in the medium size RDD, we can use the medium key set to do this before the shuffle. If there is a significant amount of entries that gets discarded this way, the resulting shuffle will need to transfer a lot less data.

```scala
val keys = sc.broadcast(mediumRDD.map(_._1).collect.toSet)
val reducedRDD = largeRDD.filter{ case(key, value) => keys.value.contains(key) }
reducedRDD.join(mediumRDD)
```

It is important to note that the efficiency gain here depends on the filter operation actually reducing the size of the larger RDD. If there are not a lot of entries lost here \(e.g., because the medium size RDD is some king of large dimension table\), there is nothing to be gained with this strategy.

You can find more details for efficient shuffles in [this Databricks presentation](http://www.slideshare.net/databricks/strata-sj-everyday-im-shuffling-tips-for-writing-better-spark-programs).

