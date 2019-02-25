---
layout: post
title: Exchange data between zeppelin pyspark and spark session
date: 2019-02-25
categories: 
- tips
- zeppelin
tags: 
- spark
- pyspark
- zeppelin
- data
- exchange
author: Kai Fan
excerpt: exchange data between pyspark and spark session within one zeppelin notes 
---


# Problem: dataframe are not shared?

As of version Zepplin(0.7.0), Spark dataframe are not shared between
%pyspark (python) and %spark (scala) session.


# Solution: exchange by using temparary table

do the following

    #%pyspark
    
    somedf.registerTempTable("somedftable")

and then rebuild the DataFrame in scala session

    //%scala
    
    val somedf = sqlContext.table("somedftable")
    
    z.show(somedf.limit(20))

