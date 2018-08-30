---
layout: post
title: Enabling Native Acceleration for MLlib
date: 2018-08-30
categories: 
- machine learning
tags: 
- machinelearning
- cloudera
- spark
- native
- netlib-java
- MKL
- openblas
- libblas
author: Kai Fan
excerpt: How to enable Enabling Native Acceleration for MLlib on RHEL 7 based cloudera cluster
---


# The undefined symbol issue

Got onto the ship of machine learning. And soon I hit the wall of
\`undefined symbol issue' on lab cluster.

Hi, it just the simple \`MinMaxScaler' example code published on spark [MLlib(Machine Learning)](https://spark.apache.org/docs/1.6.2/ml-features.html#minmaxscaler) guide!! 

everything goes well until the last line \`scaledData.show()\`,
boom. Spark-shell died with following message on the console:

    /usr/java/jdk1.7.0_67-cloudera/bin/java: symbol lookup error: /tmp/jniloader82069440205403545netlib-native_system-linux-x86_64.so: undefined symbol: cblas_daxpy

NO log. NO hisotry server record. Nothing could be used for debug as first glance.


# Solution (Wrapup)

My solution is based on CDH 5.11.0 (parcel) plus cloudera GPLExtra
(parcel) plus Intel MKL library (parcel). The steps to enable MKL
native acceleration for cloudera spark should be as simple as bellow:

1.  Install \`netlib-java\` by integrate GPLExtra parcel as described in
    the [Enable Native Accerleration For MLlib](https://www.cloudera.com/documentation/enterprise/5-11-x/topics/spark_mllib.html#concept_xzv_bj2_lt)
2.  Install MKL library parcel by follow the [Download Intel® Math
    Kernel Library (Intel® MKL) for Cloudera](https://www.cloudera.com/downloads/partner/intel.html)
3.  Enable MKL library for \`netlib-java\` by perform some dirty hack on
    **every** cluster node:

    alternatives --install /usr/lib64/libblas.so.3 libblas.so.3 /opt/cloudera/parcels/mkl/linux/mkl/lib/intel64/libmkl_rt.so 2000
    alternatives --install /usr/lib64/liblapack.so.3 liblapack.so.3 /opt/cloudera/parcels/mkl/linux/mkl/lib/intel64/libmkl_rt.so 2000


## Verification NOTES

Do **not** run **local** mode, as cloudera parcels (GPLExtra and Intel
MKL) is enabled by evaluate parcel environment shell scripts. This
scripts seems do not be run in local mode.


## Final setup summary

-   OS: RHEL 7
-   Cluster: CDH 5.11.0
    -   GPLExtra Parcel
    -   MKL Parcel


# Touble Shooting Log and Background information

The <span class="underline"><span class="underline">Solution</span></span> was developed by many times of cluster
reconfiguration and verification.

The information to setup the Native Acceleration for Cloudera spark is
from various sources includes:

-   Some are learned from cloudera CDH user guide
-   Some are learned from netlib-java github readme.md
-   Some are learned from cloudera community forumn
-   Some are learned from stackoverflow comunity


## The undefined symbol error

As described in the beginning, the clue of the issue was the error
massage. Google for the error lead to a [github issue](https://github.com/fommil/netlib-java/issues/66) of netlib-java

The solution is to make use of **LD_PRELOAD** to override os default
\`libblas.so' with openblas library.

To my understanding, this will need script hacking on all cluster
node. With our cloudera based cluster, the GPLExtras are setup by
parcel. I can't draw a clear concolution how much effort will need
to be done by next deployment.


## netlib-java

To ease the adoption, I decide to find a openblas parcel for
cloudera. During the search I discover the press release [Cloudera
And Intel Speed Up Machine Learning Workloads With Apache Spark,
Intel® Math Kernel Library Integration](https://www.cloudera.com/more/news-and-blogs/press-releases/2017-02-08-cloudera-and-intel-speed-up-machine-learning-workloads-with-apache-spark-intel-math-kernel-library-integration.html) showing that Intel MKL is a
great option.

Contine with Intel MKL, I find the [great thread](https://community.cloudera.com/t5/Advanced-Analytics-Apache-Spark/Intel-MKL-Spark-on-CDH-5-9/m-p/48962) in cloudera
community forumn. In which **openblas** and **MKL** were mentioned,
together with the **netlib-java** (the jni adopter for Native Liner
Alegbra Algorithm libraries). 

The thread also mentions that accroding to [netlib-java readme.md](https://github.com/fommil/netlib-java#linux)
the library is able to work with any **BLAS** implementation.

One can easily change a BLAS implementation with another by replace
the share library while keep the original library
filename. netlib-java jni loader will seemlessly find the new
library with same filename.

The recommended way to swith library on linux was to make use of
debian based **update-alternatives** scripts.

At the end of the netlib-java readme, the author also provide a
great list of performance benchmark of various BLAS librarys. Which
shows that Intel MKL gives **best performance** on intel based CPUs.


## The Intel MKL parcel

As described in previous section, there's a press release showing
Cloudera and Intel already work out a solution to enable MKL based
acceleration on Cloudera spark.

At the end of the press release, there's a great news
that. Cloudera and Intel will provide a [MKL **parcel**](https://www.cloudera.com/downloads/partner/intel.html). 

A parcel to cloudera usually means that the packaged feature will
be ready to use once the parcel was activated in the cluster. But,
this is not true for the MKL parcel. The parcel does not work as
expected out of the box, the undefined symbol issue remains.

It turns out that Intel did not provide any alternatives.json in
their MKL parcel (el7 flavor). So even after parcel activated, the
os bundled libblas.so is not changed. As a result the native BLAS
library netlib-java found was the old system bundled one, still.


## RHEL/Fedora BLAS library packaging mass

But, why Intel did not provide the libblas alternatives for el7
parcel?

Read the following issue records:

-   [#352 BLAS and LAPACK packaging](https://pagure.io/packaging-committee/issue/352)
-   [#588 BLAS and LAPACK packaging](https://pagure.io/packaging-committee/issue/588)

Per my understanding, this kind of thing is a opinion issue. It
will remain no-fix for a long time.

So as a RHEl/Fedora user, I am on my own. I will go create the
alternatives for myself.


## The openblas journal

Accroding to netlib-java, the openblas library is the second best
option. I'd also gave it a try.

Within the EPEL repo, openblas comes with all kinds of different
flavors.

There's no much description for these different packagings, one
have to choose the library **by name**.

After created alternatives for the openmp x64 flavor. The cluster
goes very well, until I got following coredump while running
[Multilayer perceptron classifier](https://spark.apache.org/docs/1.6.2/ml-classification-regression.html#multilayer-perceptron-classifier) example code:

    org.apache.spark.SparkException: Job aborted due to stage failure: Task 1 in stage 28.0 failed 4 times, most recent failure: Lost task 1.3 in stage 28.0 (TID 113, datanode2, executor 4): ExecutorLostFailure (executor 4 exited caused by one of the running tasks) Reason: Container marked as failed: container_1535469577308_0009_01_000005 on host: datanode2. Exit status: 134. Diagnostics: Exception from container-launch.
    Container id: container_1535469577308_0009_01_000005
    Exit code: 134
    Exception message: /bin/bash: line 1: 27674 Aborted                 (core dumped) LD_LIBRARY_PATH=/opt/cloudera/parcels/CDH-5.11.0-1.cdh5.11.0.p0.34/lib/hadoop/../../../CDH-5.11.0-1.cdh5.11.0.p0.34/lib/hadoop/lib/native:/opt/cloudera/parcels/CDH-5.11.0-1.cdh5.11.0.p0.34/lib/hadoop/../../../GPLEXTRAS-5.11.0-1.cdh5.11.0.p0.30/lib/hadoop/lib/native:/opt/cloudera/parcels/mkl-2018.3.222/linux/tbb/lib/intel64_lin/gcc4.7:/opt/cloudera/parcels/mkl-2018.3.222/linux/compiler/lib/intel64_lin:/opt/cloudera/parcels/mkl-2018.3.222/linux/mkl/lib/intel64_lin:/opt/cloudera/parcels/GPLEXTRAS-5.11.0-1.cdh5.11.0.p0.30/lib/impala/lib:/opt/cloudera/parcels/GPLEXTRAS-5.11.0-1.cdh5.11.0.p0.30/lib/hadoop/lib/native:/opt/cloudera/parcels/CDH-5.11.0-1.cdh5.11.0.p0.34/lib/hadoop/lib/native /usr/java/jdk1.7.0_67-cloudera/bin/java -server -XX:OnOutOfMemoryError='kill %p' -Xms16384m -Xmx16384m -Djava.io.tmpdir=/data/4/yarn/nm/usercache/ericsson/appcache/application_1535469577308_0009/container_1535469577308_0009_01_000005/tmp '-Dspark.authenticate.enableSaslEncryption=false' '-Dspark.authenticate=false' '-Dspark.driver.port=42674' '-Dspark.shuffle.service.port=7337' -Dspark.yarn.app.container.log.dir=/data/5/yarn/container-logs/application_1535469577308_0009/container_1535469577308_0009_01_000005 -XX:MaxPermSize=256m org.apache.spark.executor.CoarseGrainedExecutorBackend --driver-url spark://CoarseGrainedScheduler@10.163.170.11:42674 --executor-id 4 --hostname datanode2 --cores 5 --app-id application_1535469577308_0009 --user-class-path file:/data/4/yarn/nm/usercache/ericsson/appcache/application_1535469577308_0009/container_1535469577308_0009_01_000005/__app__.jar > /data/5/yarn/container-logs/application_1535469577308_0009/container_1535469577308_0009_01_000005/stdout 2> /data/5/yarn/container-logs/application_1535469577308_0009/container_1535469577308_0009_01_000005/stderr
    Stack trace: ExitCodeException exitCode=134: /bin/bash: line 1: 27674 Aborted                 (core dumped) LD_LIBRARY_PATH=/opt/cloudera/parcels/CDH-5.11.0-1.cdh5.11.0.p0.34/lib/hadoop/../../../CDH-5.11.0-1.cdh5.11.0.p0.34/lib/hadoop/lib/native:/opt/cloudera/parcels/CDH-5.11.0-1.cdh5.11.0.p0.34/lib/hadoop/../../../GPLEXTRAS-5.11.0-1.cdh5.11.0.p0.30/lib/hadoop/lib/native:/opt/cloudera/parcels/mkl-2018.3.222/linux/tbb/lib/intel64_lin/gcc4.7:/opt/cloudera/parcels/mkl-2018.3.222/linux/compiler/lib/intel64_lin:/opt/cloudera/parcels/mkl-2018.3.222/linux/mkl/lib/intel64_lin:/opt/cloudera/parcels/GPLEXTRAS-5.11.0-1.cdh5.11.0.p0.30/lib/impala/lib:/opt/cloudera/parcels/GPLEXTRAS-5.11.0-1.cdh5.11.0.p0.30/lib/hadoop/lib/native:/opt/cloudera/parcels/CDH-5.11.0-1.cdh5.11.0.p0.34/lib/hadoop/lib/native /usr/java/jdk1.7.0_67-cloudera/bin/java -server -XX:OnOutOfMemoryError='kill %p' -Xms16384m -Xmx16384m -Djava.io.tmpdir=/data/4/yarn/nm/usercache/ericsson/appcache/application_1535469577308_0009/container_1535469577308_0009_01_000005/tmp '-Dspark.authenticate.enableSaslEncryption=false' '-Dspark.authenticate=false' '-Dspark.driver.port=42674' '-Dspark.shuffle.service.port=7337' -Dspark.yarn.app.container.log.dir=/data/5/yarn/container-logs/application_1535469577308_0009/container_1535469577308_0009_01_000005 -XX:MaxPermSize=256m org.apache.spark.executor.CoarseGrainedExecutorBackend --driver-url spark://CoarseGrainedScheduler@10.163.170.11:42674 --executor-id 4 --hostname datanode2 --cores 5 --app-id application_1535469577308_0009 --user-class-path file:/data/4/yarn/nm/usercache/ericsson/appcache/application_1535469577308_0009/container_1535469577308_0009_01_000005/__app__.jar > /data/5/yarn/container-logs/application_1535469577308_0009/container_1535469577308_0009_01_000005/stdout 2> /data/5/yarn/container-logs/application_1535469577308_0009/container_1535469577308_0009_01_000005/stderr
    	at org.apache.hadoop.util.Shell.runCommand(Shell.java:601)
    	at org.apache.hadoop.util.Shell.run(Shell.java:504)
    	at org.apache.hadoop.util.Shell$ShellCommandExecutor.execute(Shell.java:786)
    	at org.apache.hadoop.yarn.server.nodemanager.DefaultContainerExecutor.launchContainer(DefaultContainerExecutor.java:213)
    	at org.apache.hadoop.yarn.server.nodemanager.containermanager.launcher.ContainerLaunch.call(ContainerLaunch.java:302)
    	at org.apache.hadoop.yarn.server.nodemanager.containermanager.launcher.ContainerLaunch.call(ContainerLaunch.java:82)
    	at java.util.concurrent.FutureTask.run(FutureTask.java:262)
    	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
    	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
    	at java.lang.Thread.run(Thread.java:745)
    Container exited with a non-zero exit code 134
    Driver stacktrace:
    	at org.apache.spark.scheduler.DAGScheduler.org$apache$spark$scheduler$DAGScheduler$$failJobAndIndependentStages(DAGScheduler.scala:1433)
    	at org.apache.spark.scheduler.DAGScheduler$$anonfun$abortStage$1.apply(DAGScheduler.scala:1421)
    	at org.apache.spark.scheduler.DAGScheduler$$anonfun$abortStage$1.apply(DAGScheduler.scala:1420)
    	at scala.collection.mutable.ResizableArray$class.foreach(ResizableArray.scala:59)
    	at scala.collection.mutable.ArrayBuffer.foreach(ArrayBuffer.scala:47)
    	at org.apache.spark.scheduler.DAGScheduler.abortStage(DAGScheduler.scala:1420)
    	at org.apache.spark.scheduler.DAGScheduler$$anonfun$handleTaskSetFailed$1.apply(DAGScheduler.scala:799)
    	at org.apache.spark.scheduler.DAGScheduler$$anonfun$handleTaskSetFailed$1.apply(DAGScheduler.scala:799)
    	at scala.Option.foreach(Option.scala:236)
    	at org.apache.spark.scheduler.DAGScheduler.handleTaskSetFailed(DAGScheduler.scala:799)
    	at org.apache.spark.scheduler.DAGSchedulerEventProcessLoop.doOnReceive(DAGScheduler.scala:1644)
    	at org.apache.spark.scheduler.DAGSchedulerEventProcessLoop.onReceive(DAGScheduler.scala:1603)
    	at org.apache.spark.scheduler.DAGSchedulerEventProcessLoop.onReceive(DAGScheduler.scala:1592)
    	at org.apache.spark.util.EventLoop$$anon$1.run(EventLoop.scala:48)
    	at org.apache.spark.scheduler.DAGScheduler.runJob(DAGScheduler.scala:620)
    	at org.apache.spark.SparkContext.runJob(SparkContext.scala:1862)
    	at org.apache.spark.SparkContext.runJob(SparkContext.scala:1982)
    	at org.apache.spark.rdd.RDD$$anonfun$reduce$1.apply(RDD.scala:1025)
    	at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:150)
    	at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:111)
    	at org.apache.spark.rdd.RDD.withScope(RDD.scala:316)
    	at org.apache.spark.rdd.RDD.reduce(RDD.scala:1007)
    	at org.apache.spark.rdd.RDD$$anonfun$treeAggregate$1.apply(RDD.scala:1150)
    	at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:150)
    	at org.apache.spark.rdd.RDDOperationScope$.withScope(RDDOperationScope.scala:111)
    	at org.apache.spark.rdd.RDD.withScope(RDD.scala:316)
    	at org.apache.spark.rdd.RDD.treeAggregate(RDD.scala:1127)
    	at org.apache.spark.mllib.optimization.LBFGS$CostFun.calculate(LBFGS.scala:218)
    	at org.apache.spark.mllib.optimization.LBFGS$CostFun.calculate(LBFGS.scala:204)
    	at breeze.optimize.CachedDiffFunction.calculate(CachedDiffFunction.scala:23)
    	at breeze.optimize.FirstOrderMinimizer.calculateObjective(FirstOrderMinimizer.scala:108)
    	at breeze.optimize.FirstOrderMinimizer.initialState(FirstOrderMinimizer.scala:101)
    	at breeze.optimize.FirstOrderMinimizer.iterations(FirstOrderMinimizer.scala:146)
    	at org.apache.spark.mllib.optimization.LBFGS$.runLBFGS(LBFGS.scala:178)
    	at org.apache.spark.mllib.optimization.LBFGS.optimize(LBFGS.scala:117)
    	at org.apache.spark.ml.ann.FeedForwardTrainer.train(Layer.scala:878)
    	at org.apache.spark.ml.classification.MultilayerPerceptronClassifier.train(MultilayerPerceptronClassifier.scala:170)
    	at org.apache.spark.ml.classification.MultilayerPerceptronClassifier.train(MultilayerPerceptronClassifier.scala:110)
    	at org.apache.spark.ml.Predictor.fit(Predictor.scala:90)
    	at $iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC.<init>(<console>:43)
    	at $iwC$$iwC$$iwC$$iwC$$iwC$$iwC$$iwC.<init>(<console>:48)
    	at $iwC$$iwC$$iwC$$iwC$$iwC$$iwC.<init>(<console>:50)
    	at $iwC$$iwC$$iwC$$iwC$$iwC.<init>(<console>:52)
    	at $iwC$$iwC$$iwC$$iwC.<init>(<console>:54)
    	at $iwC$$iwC$$iwC.<init>(<console>:56)
    	at $iwC$$iwC.<init>(<console>:58)
    	at $iwC.<init>(<console>:60)
    	at <init>(<console>:62)
    	at .<init>(<console>:66)
    	at .<clinit>(<console>)
    	at .<init>(<console>:7)
    	at .<clinit>(<console>)
    	at $print(<console>)
    	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
    	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    	at java.lang.reflect.Method.invoke(Method.java:606)
    	at org.apache.spark.repl.SparkIMain$ReadEvalPrint.call(SparkIMain.scala:1045)
    	at org.apache.spark.repl.SparkIMain$Request.loadAndRun(SparkIMain.scala:1326)
    	at org.apache.spark.repl.SparkIMain.loadAndRunReq$1(SparkIMain.scala:821)
    	at org.apache.spark.repl.SparkIMain.interpret(SparkIMain.scala:852)
    	at org.apache.spark.repl.SparkIMain.interpret(SparkIMain.scala:800)
    	at sun.reflect.GeneratedMethodAccessor13.invoke(Unknown Source)
    	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    	at java.lang.reflect.Method.invoke(Method.java:606)
    	at org.apache.zeppelin.spark.Utils.invokeMethod(Utils.java:38)
    	at org.apache.zeppelin.spark.SparkInterpreter.interpret(SparkInterpreter.java:1000)
    	at org.apache.zeppelin.spark.SparkInterpreter.interpretInput(SparkInterpreter.java:1205)
    	at org.apache.zeppelin.spark.SparkInterpreter.interpret(SparkInterpreter.java:1172)
    	at org.apache.zeppelin.spark.SparkInterpreter.interpret(SparkInterpreter.java:1165)
    	at org.apache.zeppelin.interpreter.LazyOpenInterpreter.interpret(LazyOpenInterpreter.java:97)
    	at org.apache.zeppelin.interpreter.remote.RemoteInterpreterServer$InterpretJob.jobRun(RemoteInterpreterServer.java:498)
    	at org.apache.zeppelin.scheduler.Job.run(Job.java:175)
    	at org.apache.zeppelin.scheduler.FIFOScheduler$1.run(FIFOScheduler.java:139)
    	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:471)
    	at java.util.concurrent.FutureTask.run(FutureTask.java:262)
    	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$201(ScheduledThreadPoolExecutor.java:178)
    	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:292)
    	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
    	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
    	at java.lang.Thread.run(Thread.java:745)


# References

-   [Intel MKL + Spark on CDH 5.9](https://community.cloudera.com/t5/Advanced-Analytics-Apache-Spark/Intel-MKL-Spark-on-CDH-5-9/m-p/48962)
-   [Enabling Native Acceleration for MLlib](https://community.cloudera.com/t5/Advanced-Analytics-Apache-Spark/Enabling-Native-Acceleration-for-MLlib/m-p/67656#M3341)
-   [Installing the GPL Extras Parcel](https://www.cloudera.com/documentation/enterprise/5-11-x/topics/cm_ig_install_gpl_extras.html#xd_583c10bfdbd326ba-3ca24a24-13d80143249--7ec6__section_v21_pl4_t4)

