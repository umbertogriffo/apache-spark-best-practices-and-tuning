# Use the Best Data Format

**Apache Spark** supports several data formats, including **CSV**, **JSON**, **ORC**, and **Parquet**, but just because Spark supports a given data storage or format doesn’t mean you’ll get the same performance with all of them. **Parquet** is a columnar storage format designed to only select data from columns that we actually are using, skipping over those that are not requested. This format reduces the size of the files dramatically and makes the Spark SQL query more efficient. The following picture from **\[24\]** about parquet file, explains what we could achieve with **Column Pruning** \(projection push down\) and **Predicate Push Down**:

![](https://cdn-images-1.medium.com/max/1600/1*uV2UpLrovyXCAcjZ3McWfQ.png)

As an example, we can imagine having a vehicle dataset containing information like:

```javascript
{    
"id": "AA-BB-00-77",
"type": "truck",
"origin": "ATL",
"destination": "LGA",
"depdelay": 0.0,
"arrdelay": 0.0,
"distance": 762.0
}
```

Here is the code to persist a vehicles **DataFrame** as a table consisting of Parquet files partitioned by the _destination_ column:

```scala
df.write.format("parquet")
.partitionBy("destination")
.option("path", "/data/vehicles")
.saveAsTable("vehicles")
```

Below is the resulting directory structure as shown by a Hadoop list files command:

```text
hadoop fs -ls /data/vehicles

  /data/vehicles/destination=ATL
  /data/vehicles/destination=BOS
  /data/vehicles/destination=CLT
  /data/vehicles/destination=DEN
  /data/vehicles/destination=DFW
  /data/vehicles/destination=EWR
  /data/vehicles/destination=IAH
  /data/vehicles/destination=LAX
  /data/vehicles/destination=LGA
  /data/vehicles/destination=MIA
  /data/vehicles/destination=ORD
  /data/vehicles/destination=SEA
  /data/vehicles/destination=SFO
```

Given a table of vehicles containing the information as above, using the **Column pruning** technique if the table has 7 columns, but in the query, we list only 2, the other 5 will not be read from disk. **Predicate pushdown** is a performance optimization that limits with what values will be scanned and not what columns. So, if you apply a filter on column _destination_ to only return records with value BOS, the **predicate push down** will make parquet read-only blocks that may contain values BOS. So, improve performance by allowing Spark to only read a subset of the directories and files. For example, the following query reads only the files in the `destination=BOS` partition directory in order to query the average arrival delay for vehicles destination to Boston:

```scala
df.filter("destination = 'BOS' and arrdelay > 1") 
.groupBy("destination").avg("arrdelay") .sort(desc("avg(arrdelay)")).show()

+-----------+------------------+
|destination|     avg(depdelay)|
+-----------+------------------+
|    EWR    |54.352020860495436|
|    MIA    | 48.95263157894737|
|    SFO    |47.189473684210526|
|    ORD    | 46.47721518987342|
|    DFW    |44.473118279569896|
|    CLT    |37.097744360902254|
|    LAX    |36.398936170212764|
|    LGA    | 34.59444444444444|
|    BOS    |33.633187772925766|
|    IAH    | 32.10775862068966|
|    SEA    |30.532345013477087|
|    ATL    | 29.29113924050633|
+-----------+------------------+
```

You can see the **physical plan** for a **DataFrame** calling the `explain` method as follow:

```scala
df.filter("destination = 'BOS' and arrdelay > 1") 
.groupBy("destination")
.avg("arrdelay")
.sort(desc("avg(arrdelay)"))
.explain

== Physical Plan ==
TakeOrderedAndProject(limit=1001, orderBy=[avg(arrdelay)#304 DESC NULLS LAST], 
output=[src#157,dst#149,avg(arrdelay)#314])
+- *(2) HashAggregate(keys=[destination#157, arrdelay#149],
       functions=[avg(arrdelay#152)],
       output=[destination#157, avg(arrdelay)#304])
+- Exchange hashpartitioning(destination#157, 200)
+- *(1) HashAggregate(keys=[destination#157],
              functions=[partial_avg(arrdelay#152)],  
              output=[destination#157, sum#321, count#322L])
+- *(1) Project[arrdelay#152, destination#157]
+- *(1)Filter (isnotnull(arrdelay#152) && (arrdelay#152 > 1.0))
+- *(1) FileScan parquet default.flights[arrdelay#152,destination#157] 
Batched: true, Format: Parquet, 
Location: PrunedInMemoryFileIndex[dbfs:/data/vehicles/destination=BOS], 
PartitionCount: 1, 
PartitionFilters: [isnotnull(destination#157), (destination#157 = BOS)], 
PushedFilters: [IsNotNull(arrdelay), GreaterThan(arrdelay,1.0)],
ReadSchema: struct<destination:string,arrdelay>
```

Here in `PartitionFilters`, we can see partition filter push down, which means that the `destination=BOS` filter is pushed down into the Parquet file scan. This minimizes the files and data scanned and reduces the amount of data passed back to the Spark engine for the aggregation average on the arrival delay.

