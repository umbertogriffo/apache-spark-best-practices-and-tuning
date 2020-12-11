# Table of contents

* [Introduction](README.md)

## RDD

* [Don’t collect large RDDs](rdd/dont_collect_large_rdds.md)
* [Don't use count\(\) when you don't need to return the exact number of rows](rdd/dont_use_count_when_you_dont_need_to_return_the_ex.md)
* [Avoiding Shuffle "Less stage, run faster"](rdd/avoiding_shuffle_less_stage-_more_fast.md)
* [Picking the Right Operators](rdd/avoid_groupbykey_when_performing_an_associative_re/README.md)
  * [Avoid List of Iterators](rdd/avoid_groupbykey_when_performing_an_associative_re/avoid-list-of-iterators.md)
  * [Avoid groupByKey when performing a group of multiple items by key](rdd/avoid_groupbykey_when_performing_an_associative_re/avoid-groupbykey-when-performing-a-group-of-multiple-items-by-key.md)
  * [Avoid groupByKey when performing an associative reductive operation](rdd/avoid_groupbykey_when_performing_an_associative_re/avoid-groupbykey-when-performing-an-associative-reductiove-operation.md)
  * [Avoid reduceByKey when the input and output value types are different](rdd/avoid_groupbykey_when_performing_an_associative_re/avoid-reducebykey-when-the-input-and-output-value-types-are-different.md)
  * [Avoid the flatMap-join-groupBy pattern](rdd/avoid_groupbykey_when_performing_an_associative_re/avoid-the-flatmap-join-groupby-pattern.md)
  * [Use TreeReduce/TreeAggregate instead of Reduce/Aggregate](rdd/avoid_groupbykey_when_performing_an_associative_re/use-treereducetreeaggregate-instead-of-reduceaggregate.md)
  * [Hash-partition before transformation over pair RDD](rdd/avoid_groupbykey_when_performing_an_associative_re/hash-partition-before-transformation-over-pair-rdd.md)
  * [Use coalesce to repartition in decrease number of partition](rdd/avoid_groupbykey_when_performing_an_associative_re/use-coalesce-to-repartition-in-decrease-number-of-partition.md)
* [TreeReduce and TreeAggregate Demystified](rdd/treereduce_and_treeaggregate_demystified.md)
* [When to use Broadcast variable](rdd/when_to_use_broadcast_variable.md)
* [Joining a large and a small RDD](rdd/joining-a-large-and-a-small-rdd.md)
* [Joining a large and a medium size RDD](rdd/joining-a-large-and-a-medium-size-rdd.md)

## Dataframe

* [Joining a large and a small Dataset](dataframe/joining-a-large-and-a-medium-size-dataset.md)
* [Joining a large and a medium size Dataset](dataframe/joining-a-large-and-a-small-dataset.md)

## Storage

---

* [Use the Best Data Format](use-the-best-data-format.md)
* [Cache Judiciously and use Checkpointing](which_storage_level_to_choose.md)

## Parallelism

---

* [Use the right level of parallelism](untitled.md)
* [How to estimate the size of a Dataset](sparksqlshufflepartitions_draft.md)
* [How to estimate the number of partitions, executor's and driver's params \(YARN Cluster Mode\)](how-to-estimate-the-number-of-partitions-executors-and-drivers-params-yarn-cluster-mode.md)

## Serialization and GC

---

* [Serialization](serialization.md)
* [Tuning Java Garbage Collection](tuning-java-garbage-collection.md)

## References <a id="references-1"></a>

* [References](references-1/references.md)

