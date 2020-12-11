# Serialization

Serialization plays an important role in the performance of any distributed application. Formats that are slow to serialize objects into, or consume a large number of bytes, will greatly slow down the computation. Often, this will be the first thing you should tune to optimize a Spark application. **The Java default serializer has very mediocre performance** regarding runtime as well as the size of its results. Therefore the Spark team recommends to use the [Kryo serializer](https://github.com/EsotericSoftware/kryo) instead.

The following code shows an example of how you can turn on Kryo and how you can register the classes that you will be serializing:

```scala
 val conf = new SparkConf().setAppName(...).setMaster(...)
      .set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
      .set("spark.kryoserializer.buffer.max", "128m")
      .set("spark.kryoserializer.buffer", "64m")
      .registerKryoClasses(
        Array(classOf[ArrayBuffer[String]], classOf[ListBuffer[String]])
      )
```

You can find another example here [BasicAvgWithKryo ](https://github.com/holdenk/learning-spark-examples/blob/master/src/main/java/com/oreilly/learningsparkexamples/java/BasicAvgWithKryo.java)

