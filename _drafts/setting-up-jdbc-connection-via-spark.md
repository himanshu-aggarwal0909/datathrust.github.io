---
layout: post
title: Setting up JDBC Connection Via Spark.
# author: "Himanshu Aggarwal"
# description: "Setting up JDBC Cnnection Via Spark"
date: 2022-03-01
categories: data engineer code wall
---


## Checking connectivity
Check the connectivity of your spark running server to the db hosted server.

For example, SPARK_SERVER_IP = w.x.y.z
DB_HOSTED_SERVER_IP = a.b.c.d

From, your spark server ip, ping the DB_HOSTED_SERVER_IP
```bash
$ ping a.b.c.d
PING a.b.c.d 56(84) bytes of data.
```
If you are not getting the reply back then, please check the connection setup before continuing further.

## Downloading Jar Path
Donwload Link : https://dev.mysql.com/downloads/connector/j/
Go to the above link, select the operating system and download the zip archive, extract and find the jar file

## Spark Code
```bash
# Starting Spark Shell [ replace the jar location ]
pyspark-shell --jars mysql-connector-java-8.0.26/mysql-connector-java-8.0.26.jar
```

```python
# Importing packages
import pyspark
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("Mysql Connection Via PySpark").getOrCreate()

# Note : Always store your credentials separate from the code base (like, AWS secrets manager if using, AWS)
# Only for better explaination purpose , we have stored in the code as a dictionary object.
CREDENTIALS_MASTER = {
    "db_hostname" : "a.b.c.d",
    "db_name" : "dbname_value",
    "db_username" : "username_value",
    "db_password" : "password_value",
    "db_port" : 3306
}

JDBC_URL = "jdbc:mysql://{0}:{1}".format(CREDENTIALS_MASTER["db_hostname"], CREDENTIALS_MASTER["db_port"])

dbTableDf = spark.read \
    .format("jdbc") \
    .option("driver", 'com.mysql.jdbc.Driver') \
    .option("url", JDBC_URL) \
    .option("dbtable", "schema.table_name") \
    .option("user", CREDENTIALS_MASTER["db_username"]) \
    .option("password", CREDENTIALS_MASTER["db_password"]) \
    .load()

dbTableDf
```