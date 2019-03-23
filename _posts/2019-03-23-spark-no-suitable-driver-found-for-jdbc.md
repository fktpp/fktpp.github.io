---
layout: post
title: Spark No suitable driver found for jdbc
date: 2019-03-23
categories: 
- tip
tags: 
- spark
- jdbc
- driver
author: Kai Fan
excerpt: JDBC driver not found issue in Spark
---


# The famous driver not found issue

I was request to draw a branch of data out of big data platform to
MySQL recently. After some carefully consideration, I decide to code
the logic in spark with the DataFrameWriter.write.jdbc() interface.

Everthing goes well until the integration test in which I have to
install my spark app for a real run.

    java.sql.SQLException: No suitable driver found for jdbc:mysql://bla:bla/bla


## Background1: I do follow the spark jdbc guide

Actually I did the simulation by follow the [DATABRICKS: Connecting
to SQL Databases using JDBC](https://docs.databricks.com/spark/latest/data-sources/sql-databases.html) on my own zeppline notebook. and
everything went well.


## Background2: I do follow the MySQL jdbc guide

I did follow the official [MySQL Connector/j: 6.1 Connecting to
MySQL Using the JDBC DriverManager Interface](https://dev.mysql.com/doc/connector-j/5.1/en/connector-j-usagenotes-connect-drivermanager.html#connector-j-examples-connection-drivermanager) coding sample.

    // The newInstance() call is a work around for some
    // broken Java implementations
    
    Class.forName("com.mysql.jdbc.Driver").newInstance();


## Background3: I do attached the jar dependency

I did attached the connector/j jar in spark-submit &#x2013;jars argumnt


# Attampt 1: Use recent version of MySQL connector/j

As suggested by many stackoverflow user, it's a pre-jdbc 4.0 issue,
only jdbc driver complains with jdbc 4.0 can be discovered and
registerd automatically.

I upgraded from connector/j 5.1.18 to 5.1.38 and later. The
Exception only goes away on ******some****** of our cluster running
Cloudera 5.11 on JDK8 and JRE8.

On clusters with Cloudera 5.11 and offical cloudera JDK7 the issue
remains.


# Attempt 2: Explictly set the driver class in jdbc properties

After a lot of search I find the final workaround for the issue. AS TITLE.

with following code snipp, my app now works on any Cloudera
supported JDK with any version of MySQL connector/j.

    // explicitly set the driver, or, you will get the mysterious 'java.sql.SQLException: No suitable driver' on JDK 1.7.0_67
    connectionProperties.put("driver", "com.mysql.jdbc.Driver")


# Lesson learned

Explicit is not bad!!

