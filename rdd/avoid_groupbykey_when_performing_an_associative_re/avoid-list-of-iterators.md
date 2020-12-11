# Avoid List of Iterators

Often when reading in a file [\[22\]](http://codingjunkie.net/spark-corner-cases/), we want to work with the individual values contained in each line separated by some delimiter. Splitting a delimited line is a trivial operation:

```scala
newRDD = textRDD.map(line => line.split(","))
```

But the issue here is the returned RDD will be an iterator composed of iterators. What we want is the individual values obtained after calling the split function. In other words, we need an `Array[String]`not an `Array[Array[String]]`. For this we would use the `flatMap` function. For those with a functional programming background, using a `flatMap` operation is nothing new. But if you are new to functional programming it’s a great operation to become familiar with.

```scala
val inputData = sc.parallelize (Array ("foo,bar,baz", "larry,moe,curly", "one,two,three") ).cache ()

val mapped = inputData.map (line => line.split (",") )
val flatMapped = inputData.flatMap (line => line.split (",") )

val mappedResults = mapped.collect ()
val flatMappedResults = flatMapped.collect ();

println ("Mapped results of split")
println (mappedResults.mkString (" : ") )

println ("FlatMapped results of split")
println (flatMappedResults.mkString (" : ") )
```

When we run the program we see these results:

```scala
Mapped results of split
[Ljava.lang.String;@45e22def : [Ljava.lang.String;@6ae3fb94 : [Ljava.lang.String;@4417af13
FlatMapped results of split
foo : bar : baz : larry : moe : curly : one : two : three
```

As we can see the map example returned an Array containing 3 `Array[String]` instances, while the `flatMap` call returned individual values contained in one Array.

