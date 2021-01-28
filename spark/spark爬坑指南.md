# spark集成到应用环境的爬坑

## 集群运行问题总结

##### 集群运行问题

   部分单机能运行的sql到集群里有不同程度的报错，对应修改

例如：concat_ws(';',collect_set(column)) 单机可行，集群报错

更改其他符号通过。。

#### 集群第三方jar包问题

1.  项目上使用scala版本为2.11.12,spark2.4.6版本jar包地址位于 /home/spark/spark-2.4.6-bin-hadoop2.7/jars

    替换scala相关jar包为2.11.12，避免 class not found 问题

2. 集群运行报kudu相关问题，将kudu相关包放入jars目录下

#### 应用运行问题

1.  wroker执行时报错，sparkconf.setjar(xx.jar)将jar包分发到worker节点运行。
2.   java.lang.ClassCastException: cannot assign instance of scala.collection.immutable.List$SerializationProxy to field org.apache.spark.rdd.RDD.org$apache$spark$rd
   d$RDD$$dependencies_ of type scala.collection.Seq in instance of org.apache.spark.rdd.MapPartitionsRDD 

因springboot-plugin打包执行的jar文件执行的类加载机制问题，导致不同加载器加载了spark相关jar。

解决方法：采用maven的maven-shade-plugin插件或者gradle的 shadowJar进行打包

注：不兼容gradle6.0 ，gradle6.0打出的jar需要屏蔽掉多余的log4J，否则运行报错，而spark使用的log正是log4j

而gradle5兼容这种情况。。

​	3. lassCastException: cannot assign instance of java.lang.invoke.SerializedLambda to field org.apache.spark.sql.Dataset$$anonfun$foreach$2.func$5 of type org.apache.spark.api.java.function.ForeachFunction in instance of org.apache.spark.sql.Dataset$$anonfun$foreach$2

集群模式下操作rw的数据不能使用Lambda表达式！

## 配置及数据库转换

1. sparkconf参数详解可在此类中查找

   SparkSubmitArguments.scala 

   官方配置指南：https://spark.apache.org/docs/2.4.6/configuration.html

2. spark api文档：

   http://spark.apache.org/docs/latest/api/java/index.html

3. spark支持的函数：

   http://spark.apache.org/docs/latest/api/sql/index.html#first_value

   1. 类型转换

      不支持column：：text的方式，使用 cast（column as type）

   2. 时间 计算

      不支持，先转换到时间戳格式进行计算

      ```
      to_unix_timestamp({0}, 'yyyy-MM-dd HH:mm:ss)
      ```

   3. 不支持 interval 求前几分钟、前几天操作

      ```
      from_unixtime(unix_timestamp(time) - 1*60*60*24,''yyyy-MM-dd HH:mm:ss'')
      ```

   4. 不支持 string_agg聚合列函数

   ```
   concat_ws(',',collect_set(column));
   ```

   5. 中位数计算

      ```
      percentile_approx(coloumn,0.5)
      ```

4. spark 指南

   https://blog.csdn.net/u013560925/article/details/80398081

5. 配置参数

          .set("spark.driver.cores","4")  //设置driver的CPU核数      .set("spark.driver.maxResultSize","2g") //设置driver端结果存放的最大容量，这里设置成为2G，超过2G的数据,job就直接放弃，不运行了      .set("spark.driver.memory","4g")  //driver给的内存大小      .set("spark.executor.memory","8g")// 每个executor的内存      .set("spark.submit.deployMode","cluster")  //spark 任务提交模式，线上使用cluster模式，开发使用client模式      .set("spark.worker.timeout" ,"500") //基于standAlone模式下提交任务，worker的连接超时时间      .set("spark.cores.max" , "10")  //基于standAlone和mesos模式下部署，最大的CPU和数量      .set("spark.rpc.askTimeout" , "600s")  //spark任务通过rpc拉取数据的超时时间      .set("spark.locality.wait" , "5s") //每个task获取本地数据的等待时间，默认3s钟，如果没获取到，依次获取本进程，本机，本机架数据      .set("spark.task.maxFailures" , "5")  //允许最大失败任务数，根据自身容错情况来定      .set("spark.serializer" ,"org.apache.spark.serializer.KryoSerializer")  //配置序列化方式      .set("spark.streaming.kafka.maxRatePerPartition" , "5000")  //使用directStream方式消费kafka当中的数据，获取每个分区数据最大速率      .set("spark.streaming.backpressure.enabled" , "true")  //开启sparkStreaming背压机制，接收数据的速度与消费数据的速度实现平衡    //  .set("spark.streaming.backpressure.pid.minRate","10")      .set("spark.driver.host", "localhost")  //配置driver地址      //shuffle相关参数调优开始      .set("spark.reducer.maxSizeInFlight","96m")  //reduceTask拉取map端输出的最大数据量，调整太大有OOM的风险      .set("spark.shuffle.compress","true")  //开启shuffle数据压缩      .set("spark.default.parallelism","10")  //设置任务的并行度      .set("spark.files.fetchTimeout","120s")  //设置文件获取的超时时间      //网络相关参数      .set("spark.rpc.message.maxSize","256")  //RPC拉取数据的最大数据量，单位M      .set("spark.network.timeout","120s")  //网络超时时间设置      .set("spark.scheduler.mode","FAIR")  //spark 任务调度模式  使用 fair公平调度      //spark任务资源动态划分  https://spark.apache.org/docs/2.3.0/job-scheduling.html#configuration-and-setup      .set("spark.dynamicAllocation.enabled","true")      .set("spark.shuffle.service.enabled","true")      .set("spark.dynamicAllocation.executorIdleTimeout","120s")  //executor空闲时间超过这个值，该executor就会被回收      .set("spark.dynamicAllocation.minExecutors","0")  //最少的executor个数      .set("spark.dynamicAllocation.maxExecutors","32")  //最大的executor个数  根据自己实际情况调整      .set("spark.dynamicAllocation.initialExecutors","4")//初始executor个数      .set("spark.dynamicAllocation.schedulerBacklogTimeout","5s")  //pending 状态的task时间，过了这个时间继续pending ，申请新的executor      .setMaster("local[1]")      .setAppName("Stream")    sparkConf.set("spark.speculation", "true")   //开启推测执行    sparkConf.set("spark.speculation.interval", "100s")  // 每隔多久检测一次是否需要进行推测执行任务    sparkConf.set("spark.speculation.quantile","0.9")  //完成任务的百分比，然后才能启动推测执行    sparkConf.set("spark.streaming.backpressure.initialRate" , "500")  // //开启sparkStreaming的背压机制，然后第一批次获取数据的最大速率 


调整后的配置项为：

spark: 
  master: spark://NameNode:7077
  driver:
    core: 8
    maxResultSize: 1g
    memory: 1g
  executor:
    memory: 1g
  defalut:
    parallelism: 64
  num:
    executors: 8