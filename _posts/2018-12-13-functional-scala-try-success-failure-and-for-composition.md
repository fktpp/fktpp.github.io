---
layout: post
title: Functional scala Try Success Failure and for composition
date: 2018-12-13
categories: 
- scala
- programming
tags: 
- scala
- functional
- exception
author: Kai Fan
excerpt: Functional programing in scala with exception handling
---


# Lesson learn scala programming

I got some scala to write in recent spark related
project. Specifily, the scala udf (aka. user defined function) to be
used in spark application.

I got a feel that [Try, Success, Failure](https://www.scala-lang.org/api/2.9.3/scala/util/Try.html) is not very useful until
being applied to the [For Comprehensions](https://docs.scala-lang.org/tour/for-comprehensions.html) for automatically
unpacking. Refer to the following code example (inspired by
[StackOverFlow Post](https://stackoverflow.com/questions/44886772/how-to-convert-a-string-column-with-milliseconds-to-a-timestamp-with-millisecond))

    import java.sql.Timestamp
    import org.apache.spark.sql.functions.udf
    import scala.util.{Try, Success, Failure}
    
    // combine time_date and time_time for a timestamp in UTC.
    val time_to_ts: ((String, String) => Option[Timestamp]) = (time_date, time_time) => {
    
      val time_time_a = time_time.split(':')
      val ts = for (
        y <- Try(time_date.take(4).toInt);              // toInt exception
        m <- Try(time_date.take(6).takeRight(2).toInt); // toInt exception
        d <- Try(time_date.take(8).takeRight(2).toInt); // toInt exception
        tsymd <- Try(Timestamp.valueOf(s"$y-$m-$d 0:0:0.0").getTime()); // parsing
    								    // exception
    
        h <- Try(time_time_a(0));  // index exception
        M <- Try(time_time_a(1));  // ditto
        s <- Try(time_time_a(2));  // ditto
        ms <- Try(time_time_a(3)); // ditto
        tshmsms <- Try(Timestamp.valueOf(s"1970-1-1 $h:$M:$s.$ms").getTime()); // parsing
    									   // exception
    
        ts <- Try(new Timestamp(tsymd - 3600 * 1000 * 8 + tshmsms)) // construct
    								// exception
      ) yield ts
    
      ts match {
        case Success(timestamp) => Some(timestamp)
        case Failure(_) => None
      }
    }
    
    val getTs = udf(time_to_ts)

