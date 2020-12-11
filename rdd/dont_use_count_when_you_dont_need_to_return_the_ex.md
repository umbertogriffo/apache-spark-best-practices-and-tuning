# Don't use count\(\) when you don't need to return the exact number of rows

When you don't need to return the exact number of rows use:

```scala
DataFrame inputJson = sqlContext.read().json(...);
if (inputJson.takeAsList(1).size() == 0) {...}

or

if (inputJson.queryExecution.toRdd.isEmpty()) {...}
```

instead of:

```scala
if (inputJson.count() == 0) {...}
```

In RDD you can use **isEmpty\(\)** because if you see the code: [https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/rdd/RDD.scala](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/rdd/RDD.scala)

```scala
def isEmpty(): Boolean = withScope { 
    partitions.length == 0 || take(1).length == 0 
}
```



