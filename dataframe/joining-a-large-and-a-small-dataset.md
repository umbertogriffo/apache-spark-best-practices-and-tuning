# Joining a large and a medium size Dataset

If the smaller **DataFrame** does not fit fully into memory, but its keyset does, it is possible to exploit this. As a join will discard all elements of the larger DataFrame that do not have a matching partner in the medium size DataFrame, we can use the medium key set to do this before the shuffle. If there is a significant amount of entries that get discarded this way, the resulting shuffle will need to transfer a lot fewer data.

```scala
import org.apache.spark.sql.functions._

val mediumDf = Seq((0, "zero"), (4, "one")).toDF("id", "value")
val largeDf = Seq((0, "zero"), (2, "two"), (3, "three"), (4, "four"), (5, "five")).toDF("id", "value")

mediumDf.show()
largeDf.show()

/*
+---+-----+
| id|value|
+---+-----+
|  0| zero|
|  4|  one|
+---+-----+
+---+-----+
| id|value|
+---+-----+
|  0| zero|
|  2|  two|
|  3|three|
|  4| four|
|  5| five|
+---+-----+
*/

val keys = mediumDf.select("id").as[Int].collect().toSeq
print(keys)
/*
keys: Seq[Int] = WrappedArray(0, 4)
*/

val reducedDataFrame = largeDf.filter(col("id").isin(keys:_*))
res.show()
/*
+---+-----+
| id|right|
+---+-----+
|  0| zero|
+---+-----+
*/

val result = reducedDataFrame.join(mediumDf, Seq("id"))
result.explain()
result.show()

/*
== Physical Plan ==
*(1) Project [id#246, value#247, value#238]
+- *(1) BroadcastHashJoin [id#246], [id#237], Inner, BuildRight
   :- LocalTableScan [id#246, value#247]
   +- BroadcastExchange HashedRelationBroadcastMode(List(cast(input[0, int, false] as bigint))), [id=#234]
      +- LocalTableScan [id#237, value#238]
+---+-----+-----+
| id|value|value|
+---+-----+-----+
|  0| zero| zero|
|  4| four|  one|
+---+-----+-----+
*/
```

It is important to note that the efficiency gain here depends on the filter operation, actually reducing the size of the larger DataFrame. If there are not a lot of entries lost here \(e.g., because the medium size DataFrame is some king of large dimension table\), there is nothing to be gained with this strategy.

