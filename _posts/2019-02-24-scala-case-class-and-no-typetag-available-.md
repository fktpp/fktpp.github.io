---
layout: post
title: scala case class and No TypeTag available 
date: 2019-02-24
categories: 
- code
- issue
tags: 
- no
- typetag
- case class
- scala
- spark
author: Kai Fan
excerpt: No TypeTag available error raised from a method of which its code runs well in zeppelin
---


# No TypeTag available compiling&#x2026;

following code works whell in zeppelin section

    val textRdd = sc.textFile("hdfs://nameservice1/user/myname/bank/bank.csv")
    
    case class TextLine(lineText: String)
    
    val modelDates = textRdd.map( s => TextLine(s.trim)).toDF()
    
    modelDates.sort(col("lineText").desc).as[String].first()

But if I make a function from it, err!!

    def maxValueIn(hdfsPath: String) = {
      val textRdd = sc.textFile(hdfsPath)
    
      case class TextLine(lineText: String)
    
      val modelDates = textRdd.map( s => TextLine(s.trim)).toDF()
    
      modelDates.sort(col("lineText").desc).as[String].first()
    }


# solution: Move the case class out of def!!!

As described in the [stack overflow question](https://stackoverflow.com/questions/29143756/scala-spark-app-with-no-typetag-available-error-in-def-main-style-app), move the case class out
of the method, the code finally compiles.

    import org.apache.spark.SparkContext
    import org.apache.spark.sql.hive.HiveContext
    import org.apache.spark.sql.functions.col
    
    trait HdfsTextFile {
      val spark: SparkContext
      val sqlContext: HiveContext
    
      import sqlContext.implicits._
    
      case class Line(lineText: String)
    
      def maxValueIn(hdfsPath: String) = {
        val textRdd = spark.textFile(hdfsPath)
    
        val modelDates =
          textRdd.map(s => Line(s.trim)).toDF()
    
        modelDates.sort(col("lineText").desc).as[String].first
      }
    }

