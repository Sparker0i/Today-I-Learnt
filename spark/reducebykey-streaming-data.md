# Applying reduceByKey operation on a Streaming Data

So here's the task I had at hand. I would be receiving streams of data from a Kafka cluster. I will be using a `DStream` to capture the data coming from Kafka. 

Following is the weather data that will be sent by a producer application across Kafka into my consumer application:

`"725030:14732",2008,12,31,11,0.6,-6.7,1001.7,80,6.2,8,0.0,0.0)`

1. From this, I would mainly require `wsid` (1st parameter), `year`, `month`, `day` (2nd, 3rd, 4th parameters respectively) and `oneHourPrecip` (second last parameter).
2. Then the task was to aggregate all the values of `oneHourPrecip` values based on the key value of the other parameters as mentioned above.

Achieving the first part was easy. All I needed to do was perform a `map` operation on my stream. Assuming my stream was named `weatherStream`, which would contain the Stream mapped to objects of a `case class` (A data class otherwise), I would be performing:

```scala
weatherStream.map{weather =>
    (weather.wsid , weather.year , weather.month , weather.day, weather.oneHourPrecip)
}
```

In this way, I would be able to extract only the necessary informations.

Then I had to aggregate the `oneHourPrecip` values based upon the key `(wsid, year, month, day)`. I was sure we would require a `reduce` operation in this scenario. With further research I would find that `reduceByKey` would best fit my needs. 

But here's the catch. In order to perform `reduceByKey` operations, the stream should be represented as key-value pairs. With the way I had mapped the stream, it would not generate any key-value pairs, but rather objects.

I had to find a way to represent as key-value pairs. So:

Learning #1: **If you have to perform a `reduce`, you must structure your streams/dataframes as `K-V` pairs**.

For this, I had to re-structure my code as follows:

```scala
weatherStream.map{weather =>
    ((weather.wsid , weather.year , weather.month , weather.day), weather.oneHourPrecip)
}
```

Notice the new pair of parantheses across the key values. Now I have ensured that my stream will be represented as key-value pairs. Then it was only a matter of doing `reduceByKey` operation:

```scala
weatherStream.map{weather =>
    ((weather.wsid , weather.year , weather.month , weather.day), weather.oneHourPrecip)
}.reduceByKey((a , b) => a + b)
```