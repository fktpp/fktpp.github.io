---
layout: post
title: Mysterious sqoop1 export options
date: 2019-03-23
categories: 
- tip
tags: 
- sqoop
- partition
- hcat
- hive
- rdbms
author: Kai Fan
excerpt: sqoop1 export options to limit dataset to specific partitions
---


# Ever want to export periodicly generated hive data to RDBMS?

There may be serval solution available depends on the data size and
update interval. But if you decide to use sqoop1 to export the
******Hive table******, don't follow those handcraft solution suggestions,
try the sqoop1 internal one.


## The handcraft solution: copy , export , delete

As title, this kind of solution mainly consist of 3 step.

1.  Copy data out from warehouse to general HDFS path
2.  Run sqoop command against the copied HDFS path
3.  Delete the copied HDFS path


## The sqoop1 internal solution: HCatalog arguments

Run <quote>sqoop-export &#x2013;help</quote> and pay attention to the
******HCatalog arguments****** section

    HCatalog arguments:
    --hcatalog-database <arg>                        HCatalog database name
    --hcatalog-home <hdir>                           Override $HCAT_HOME
    --hcatalog-partition-keys <partition-key>        Sets the partition
    						 keys to use when
    						 importing to hive
    --hcatalog-partition-values <partition-value>    Sets the partition
    						 values to use when
    						 importing to hive
    --hcatalog-table <arg>                           HCatalog table name
    --hive-home <dir>                                Override $HIVE_HOME
    --hive-partition-key <partition-key>             Sets the partition key
    						 to use when importing
    						 to hive
    --hive-partition-value <partition-value>         Sets the partition
    						 value to use when
    						 importing to hive
    --map-column-hive <arg>                          Override mapping for
    						 specific column to
    						 hive types.

Note that &#x2013;hcatalog-partition-keys and &#x2013;hcatalog-partition-values
and &#x2013;hive-partition-key and &#x2013;hive-partition-value. 

Try specify partitions and partiton values as comma sperated list
for pural ones.

Even these arguments' help string is wrongly stated they are
******import****** options, they did the partition filtering job very
good when exporting.

