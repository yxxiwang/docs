# hbase通过创建hive外部表问题

## hbase表信息

```bash
> hbase shell
> scan 'test'
ROW                           COLUMN+CELL                                                                         
 1                            column=f:age, timestamp=1468916224789, value=20                                     
 1                            column=f:name, timestamp=1468916224789, value=xiaoqiang                             
 1                            column=f:sex, timestamp=1468916224789, value=M                                      
 2                            column=f:age, timestamp=1468921354371, value=34                                     
 2                            column=f:name, timestamp=1468916257898, value=xiaomei                               
 2                            column=f:sex, timestamp=1468916257898, value=W                                      
 3                            column=f:age, timestamp=1468916287514, value=34                                     
 3                            column=f:name, timestamp=1468916287514, value=xiaoxiao                              
 3                            column=f:sex, timestamp=1468916287514, value=M                                      
3 row(s) in 0.2800 seconds
```

## spark-1.6.1

拷贝`hive-site.xml`和`hbase-site.xml`文件

```bash
> cp /opt/cloudera/parcels/CDH-5.6.0-1.cdh5.6.0.p0.45/lib/hive/conf/hive-site.xml /opt/spark-1.6.1/conf/
> git diff /opt/cloudera/parcels/CDH-5.6.0-1.cdh5.6.0.p0.45/lib/hive/conf/hive-site.xml /opt/spark-1.6.1/conf/hive-site.xml

> cp /opt/cloudera/parcels/CDH-5.6.0-1.cdh5.6.0.p0.45/lib/hbase/conf/hbase-site.xml /opt/spark-1.6.1/conf/
> git diff /opt/cloudera/parcels/CDH-5.6.0-1.cdh5.6.0.p0.45/lib/hbase/conf/hbase-site.xml /opt/spark-1.6.1/conf/hbase-site.xml
```

修改`spark-env.sh`文件

```base
> cat /opt/spark-1.6.1/conf/spark-env.sh
[...]
HIVE_HOME='/opt/cloudera/parcels/CDH-5.6.0-1.cdh5.6.0.p0.45/lib/hive'
HBASE_HOME='/opt/cloudera/parcels/CDH-5.6.0-1.cdh5.6.0.p0.45/lib/hbase'
export SPARK_CLASSPATH=$HIVE_HOME/lib/hive-hbase-handler.jar:$HBASE_HOME/hbase-client.jar:$HBASE_HOME/hbase-common.jar:$HBASE_HOME/hbase-protocol.jar:$HBASE_HOME/hbase-server.jar:$HBASE_HOME/lib/htrace-core.jar:$HBASE_HOME/lib/protobuf-java-2.5.0.jar:$HBASE_HOME/lib/guava-12.0.1.jar:$SPARK_CLASSPATH
```

启动`spark-1.6.1`

```bash
> cd /opt/spark-1.6.1
> sbin/start-all.sh
starting org.apache.spark.deploy.master.Master, logging to /opt/spark-1.6.1/logs/spark-root-org.apache.spark.deploy.master.Master-1-server03.out
localhost: starting org.apache.spark.deploy.worker.Worker, logging to /opt/spark-1.6.1/logs/spark-root-org.apache.spark.deploy.worker.Worker-1-server03.out
```

进入`pyspark`执行创建数据库表命令

```bash
> bin/pyspark --master spark://server03:7077
[... set up log ...]
>>> sqlContext.sql("CREATE EXTERNAL TABLE IF NOT EXISTS test_hbase_register_table_name_1 (`id` INT,`name` STRING,`age` INT,`sex` STRING) STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler' WITH SERDEPROPERTIES (\"hbase.columns.mapping\" = \":key,f:name,f:age,f:sex\") TBLPROPERTIES ('hbase.table.name' = 'test')").collect()
[... execute log ...]
[]
>>> sqlContext.sql("select * from test_hbase_register_table_name_1").collect()
[... execute log ...]
[Row(id=1, name=u'xiaoqiang', age=20, sex=u'M'), Row(id=2, name=u'xiaomei', age=34, sex=u'W'), Row(id=3, name=u'xiaoxiao', age=34, sex=u'M')]
>>>
```

停止`spark-1.6.1`服务

```bash
> cd /opt/spark-1.6.1
> sbin/stop-all.sh
localhost: stopping org.apache.spark.deploy.worker.Worker
stopping org.apache.spark.deploy.master.Master
```

## spark-2.0.0

拷贝`hive-site.xml`和`hbase-site.xml`文件

```bash
> cp /opt/spark-1.6.1/conf/hive-site.xml /opt/spark-2.0.0/conf/
> cp /opt/spark-1.6.1/conf/hbase-site.xml /opt/spark-2.0.0/conf/
> 
> git diff /opt/spark-1.6.1/conf/hive-site.xml /opt/spark-2.0.0/conf/hive-site.xml
> git diff /opt/spark-1.6.1/conf/hbase-site.xml /opt/spark-2.0.0/conf/hbase-site.xml
```

拷贝`spark-env.sh`配置文件

```bash
> cp /opt/spark-1.6.1/conf/spark-env.sh /opt/spark-2.0.0/conf/
cp: overwrite `/opt/spark-2.0.0/conf/spark-env.sh'? y
> git diff /opt/spark-1.6.1/conf/spark-env.sh /opt/spark-2.0.0/conf/spark-env.sh

> cat /opt/spark-2.0.0/conf/spark-env.sh
[...] 
HIVE_HOME='/opt/cloudera/parcels/CDH-5.6.0-1.cdh5.6.0.p0.45/lib/hive'
HBASE_HOME='/opt/cloudera/parcels/CDH-5.6.0-1.cdh5.6.0.p0.45/lib/hbase'
export SPARK_CLASSPATH=$HIVE_HOME/lib/hive-hbase-handler.jar:$HBASE_HOME/hbase-client.jar:$HBASE_HOME/hbase-common.jar:$HBASE_HOME/hbase-protocol.jar:$HBASE_HOME/hbase-server.jar:$HBASE_HOME/lib/htrace-core.jar:$HBASE_HOME/lib/protobuf-java-2.5.0.jar:$HBASE_HOME/lib/guava-12.0.1.jar:$SPARK_CLASSPATH
```

启动`spark-2.0.0`

```bash
> cd /opt/spark-2.0.0
> sbin/start-all.sh 
starting org.apache.spark.deploy.master.Master, logging to /opt/spark-2.0.0/logs/spark-root-org.apache.spark.deploy.master.Master-1-server03.out
server03: starting org.apache.spark.deploy.worker.Worker, logging to /opt/spark-2.0.0/logs/spark-root-org.apache.spark.deploy.worker.Worker-1-server03.out
```

进入`pyspark`执行创建数据库表命令

```bash
> bin/pyspark --master spark://server03:7077
[... set up log ...]
>>> spark.sql("CREATE EXTERNAL TABLE IF NOT EXISTS test_hbase_register_table_name_2 (`id` INT,`name` STRING,`age` INT,`sex` STRING) STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler' WITH SERDEPROPERTIES (\"hbase.columns.mapping\" = \":key,f:name,f:age,f:sex\") TBLPROPERTIES ('hbase.table.name' = 'test')").collect()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/opt/spark-2.0.0/python/pyspark/sql/session.py", line 541, in sql
    return DataFrame(self._jsparkSession.sql(sqlQuery), self._wrapped)
  File "/opt/spark-2.0.0/python/lib/py4j-0.10.1-src.zip/py4j/java_gateway.py", line 933, in __call__
  File "/opt/spark-2.0.0/python/pyspark/sql/utils.py", line 73, in deco
    raise ParseException(s.split(': ', 1)[1], stackTrace)
pyspark.sql.utils.ParseException: u'\nOperation not allowed: STORED BY(line 1, pos 117)\n\n== SQL ==\nCREATE EXTERNAL TABLE IF NOT EXISTS test_hbase_register_table_name_2 (`id` INT,`name` STRING,`age` INT,`sex` STRING) STORED BY \'org.apache.hadoop.hive.hbase.HBaseStorageHandler\' WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,f:name,f:age,f:sex") TBLPROPERTIES (\'hbase.table.name\' = \'test\')\n---------------------------------------------------------------------------------------------------------------------^^^\n'
>>>
>>>  
>>> sqlContext.sql("CREATE EXTERNAL TABLE IF NOT EXISTS test_hbase_register_table_name_2 (`id` INT,`name` STRING,`age` INT,`sex` STRING) STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler' WITH SERDEPROPERTIES (\"hbase.columns.mapping\" = \":key,f:name,f:age,f:sex\") TBLPROPERTIES ('hbase.table.name' = 'test')").collect()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/opt/spark-2.0.0/python/pyspark/sql/context.py", line 350, in sql
    return self.sparkSession.sql(sqlQuery)
  File "/opt/spark-2.0.0/python/pyspark/sql/session.py", line 541, in sql
    return DataFrame(self._jsparkSession.sql(sqlQuery), self._wrapped)
  File "/opt/spark-2.0.0/python/lib/py4j-0.10.1-src.zip/py4j/java_gateway.py", line 933, in __call__
  File "/opt/spark-2.0.0/python/pyspark/sql/utils.py", line 73, in deco
    raise ParseException(s.split(': ', 1)[1], stackTrace)
pyspark.sql.utils.ParseException: u'\nOperation not allowed: STORED BY(line 1, pos 117)\n\n== SQL ==\nCREATE EXTERNAL TABLE IF NOT EXISTS test_hbase_register_table_name_2 (`id` INT,`name` STRING,`age` INT,`sex` STRING) STORED BY \'org.apache.hadoop.hive.hbase.HBaseStorageHandler\' WITH SERDEPROPERTIES ("hbase.columns.mapping" = ":key,f:name,f:age,f:sex") TBLPROPERTIES (\'hbase.table.name\' = \'test\')\n---------------------------------------------------------------------------------------------------------------------^^^\n'
>>>
```



