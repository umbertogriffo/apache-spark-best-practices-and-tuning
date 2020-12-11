# Avoid reduceByKey when the input and output value types are different

For example, consider writing a transformation that finds all the unique strings corresponding to each key. One way would be to use map to transform each element into a Set and then combine the Sets with `reduceByKey`:

```scala
rdd.map(kv => (kv._1, new Set[String]() + kv._2)) .reduceByKey(_ ++ _)
```

This code results in tons of unnecessary object creation because a new Set must be allocated for each record. It’s better to use `aggregateByKey`, which performs the map-side aggregation more efficiently:

```scala
val zero = new collection.mutable.Set[String]() 
rdd.aggregateByKey(zero)( (set, v) => set += v, (set1, set2) => set1 ++= set2)
```

