# Apache Sqoop



## What is Apache Sqoop?

Apache Sqoop (SQL-to-Hadoop) is a tool designed for efficiently transferring bulk data between Apache Hadoop and structured datastores such as relational databases, enterprise data warehouses, and NoSQL systems.

### ETL Tools

ETL is short for **extract**, **transform**, **load**, three database functions that are combined into one tool to pull data out of one database and place it into another database.

* **Extract** is the process of reading data from a database. In this stage, the data is collected, often from multiple and different types of sources.

* **Transform** is the process of converting the extracted data from its previous form into the form it needs to be in so that it can be placed into another database. Transformation occurs by using rules or lookup tables or by combining the data with other data.

* **Load** is the process of writing the data into the target database.


![ETL Process](https://cdn.buttercms.com/compress/LjY9fP8fQWyBQ4aVlLy7)

## Process
With Sqoop, you can import data from a relational database system or a mainframe into HDFS. The input to the import process is either database table or mainframe datasets. For databases, Sqoop will read the table row-by-row into HDFS. For mainframe datasets, Sqoop will read records from each mainframe dataset into HDFS. The output of this import process is a set of files containing a copy of the imported table or datasets. The import process is performed in parallel. For this reason, the output will be in multiple files. These files may be delimited text files (for example, with commas or tabs separating each field), or binary Avro or SequenceFiles containing serialized record data.

## Compatibility
This document assumes you are using a **Linux** or Linux-like environment. If you are using **Windows**, you may be able to use cygwin to accomplish most of the following tasks. If you are using **Mac OS X**, you should see few (if any) compatibility errors. Sqoop is predominantly operated and tested on Linux.

## Sqoop Architecture
Data transfer between Sqoop and external storage system is made possible with the help of Sqoop's connectors.

Sqoop has connectors for working with a range of popular relational databases, including MySQL, PostgreSQL, Oracle, SQL Server, and DB2. Each of these connectors knows how to interact with its associated DBMS. There is also a generic JDBC connector for connecting to any database that supports Java's JDBC protocol. In addition, **Sqoop provides optimized MySQL and PostgreSQL connectors** that use database-specific APIs to perform bulk transfers efficiently.

![sqoop architecture](https://www.guru99.com/images/Big_Data/061114_1038_Introductio1.png)

## Why should I use Sqoop?

Analytical processing using Hadoop requires loading of huge amounts of data from diverse sources into Hadoop clusters.  
 This process of bulk data load into Hadoop, from heterogeneous sources and then processing it, comes with a certain set of challenges. Maintaining and ensuring data consistency and ensuring efficient utilization of resources, are some factors to consider before selecting the right approach for data load.
  
    

**Data load using Scripts**  
The traditional approach of using scripts to load data is not suitable for bulk data load into Hadoop; this approach is inefficient and very time-consuming.


  
  
**Direct access to external data via Map-Reduce application**  
Providing direct access to the data residing at external systems(without loading into Hadoop) for map-reduce applications complicates these applications. So, this approach is not feasible.

## Sqoop vs Flume

|   Sqoop      |   Flume        |
|:-------------|:---------------|
| Sqoop is used for importing data from structured data sources such as RDBMS.| Flume is used for moving bulk streaming data into HDFS
|Sqoop has a connector based architecture. Connectors know how to connect to the respective data source and fetch the data.|Flume has an agent-based architecture. Here, a code is written (which is called as 'agent') which takes care of fetching data. 
|Data flows to HDFS through zero or more channels.|HDFS is an ultimate destination for data storage.|
|Sqoop data load is not event-driven.|	Flume data load can be driven by an event.||In order to load streaming data such as tweets generated on Twitter or log files of a web server, Flume should be used. Flume agents are built for fetching streaming data.

![Sqoop vs Flume](https://s3.amazonaws.com/files.dezyre.com/images/blog/Sqoop+vs.+Flume+%E2%80%93+Battle+of+the+Hadoop+ETL+tools/Difference+between+Sqoop+and+Flume.png)



______________________________________________________________________

## How to start?

**Note:**  

* For this tutorial I'm using **macOS Mojave (version 10.14.6)**.

Fisrt you need to install **Hadoop**, it must be installed and configured. Sqoop is currently supporting 4 major Hadoop releases - 0.20, 0.23, 1.0 and 2.0.

#### How to install hadoop?
Hadoop 3.2.1 

You have to have istalled **HomeBrew**.

~~~bash
waltervives$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

~~~
Also, you have to check that you have the correct version of **Java** (1.8).

~~~bash
waltervives$ java -version
~~~
**Hadoop**  
Then type in the command line:

```
waltervives$ brew install hadoop
```

**Configurate Hadoop**

To configurate Hadoop we need to edit some files.

* Updating the environment variable settings
* Make changes to core-, hdfs-, mapred- and yarn-site.xml files
* Remove password requirement (if necessary)
* Format NameNode

Open the document containing the environment variable settings:

```
$ cd /usr/local/cellar/hadoop/3.2.1_1/libexec/etc/hadoop
$ open hadoop-env.sh

```

Add the location for export JAVA_HOME

```
export JAVA_HOME="/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home"
```

Replace information for export HADOOP_OPTS

change

```
export HADOOP_OPTS="-Djava.net.preferIPv4Stack=true"

```
to 

```
export HADOOP_OPTS="-Djava.net.preferIPv4Stack=true -Djava.security.krb5.realm= -Djava.security.krb5.kdc="

```
**Make changes to core files**

```
waltervives$ open core-site.xml

<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
  </property>
</configuration>
```
**Make changes to hdfs files**

```
waltervives$ open hdfs-site.xml

<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
</configuration>
```

**Make changes to mapred files**

```
waltervives$ open mapred-site.xml

<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
  <property>
    <name>mapreduce.application.classpath</name>   <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
  </property>
</configuration>
```


**Make changes to yarn files**


```
waltervives$ open yarn-site.xml

<configuration>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.env-whitelist</name>
<value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
  </property>
</configuration>

```


**Remove password requirement**  
Check if you’re able to ssh without a password before moving to the next step to prevent unexpected results when formatting the NameNode.

```
waltervives$ ssh localhost
```

If this does not return a last login time, use the following commands to remove the need to insert a password.

```
waltervives$ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
waltervives$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
waltervives$ chmod 0600 ~/.ssh/authorized_keys

```
Also if you still have the next error: ```connect to host localhost port 22: Connection refused``` you can go to **System Preferences** then, **File Sharing**. This worked on my machine.

![System Preference](https://i.stack.imgur.com/4ZP4J.png)

**Format NameNode**

```
waltervives$ cd /usr/local/cellar/hadoop/3.2.1/libexec/bin
waltervives$ hdfs namenode -format

```
A warning will tell you that a directory for logs is being created. You will be prompted to re-format filesystem in Storage Directory root. Say **Y** and press **RETURN**.



**Run Hadoop**

```
waltervives$ cd /usr/local/cellar/hadoop/3.2.1_1/libexec/sbin
waltervives$ ./start-all.sh
waltervives$ jps
```
**After running jps, you should have confirmation that all the parts of Hadoop have been installed and running. You should see something like this:**

```
66896 ResourceManager
66692 SecondaryNameNode
66535 DataNode
67350 Jps
66422 NameNode67005 NodeManager
```

**Open a web browser to see your configurations for the current session**

[http://localhost:9870](http://localhost:9870)

![Hadoop Sesion](https://miro.medium.com/max/1400/1*RhtBQmHug9dNKDgH-niLAA.png)    




**Close Hadoop**

```
waltervives$ ./stop-all.sh
```

### Installing Sqoop 

Using **HomeBrew**:

```
waltervives$ brew instal sqoop

```

Then, we have to install the dependencies and configure it. 

---

## Hadoop Ecosystem

![hadoop ecosystem](http://blogs.saphana.com/wp-content/uploads/2015/11/hadoop3.jpg)


## References

* https://towardsdatascience.com/installing-hadoop-on-a-mac-ec01c67b003c

* https://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html#_introduction

* https://www.guru99.com/introduction-to-flume-and-sqoop.html#:~:text=Apache%20Sqoop%20(SQL-to-,connectivity%20to%20new%20external%20systems.

* https://www.dezyre.com/article/sqoop-vs-flume-battle-of-the-hadoop-etl-tools-/176#:~:text=The%20major%20difference%20between%20Sqoop,a%20stream%20of%20moving%20data.

* https://stackoverflow.com/questions/17335728/connect-to-host-localhost-port-22-connection-refused









