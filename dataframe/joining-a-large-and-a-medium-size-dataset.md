# Joining a large and a small Dataset

A technique to improve the performance is analyzing the **DataFrame** size to get the best join strategy. 

If the smaller DataFrame is small enough to fit into the memory of each worker, we can turn **ShuffleHashJoin** or **SortMergeJoin** into a **BroadcastHashJoin**. In broadcast join, the smaller DataFrame will be broadcasted to all worker nodes. Using the BROADCAST hint guides Spark to broadcast the smaller DataFrame when joining them with the bigger one:

```scala
largeDf.join(smallDf.hint("broadcast"), Seq("id"))
```

This way, the larger DataFrame does not need to be shuffled at all.

> _Recently Spark has increased the maximum size for the broadcast table from 2GB to 8GB. Thus, it is not possible to broadcast tables which are greater than 8GB._

