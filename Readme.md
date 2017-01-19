# Notes
Notes with basic command lines and some concepts.

**Table of Content**

1. [HDFS and YARN](#hdfs-and-yarn)  
2. [Sqoop](#sqoop)  
3. [Hive and Impala](#hive-and-impala)
  - [Impala in Shell](#impala-in-shell)
  - [Hive in Shell](#hive-in-shell)
  - [DLL](#ddl)
  - [Partitioning](#partitioning)
4. [Data formats](#data-formats)
  - [Types of Data](#types-of-data)
  - [More deeply in Avro](#more-deeply-in-avro)
  - [Using Avro in Sqoop, Hive and Impala](#using-avro-in-sqoop-hive-and-impala)
  - [Using Parquet in Sqoop, Hive and Impala](#using-parquet-in-sqoop-hive-and-impala)
5. [Flume](#flume)
  - [Configuration](#configuration)
6. [Spark](#spark)
  - [Starting](#starting)


## HDFS and YARN
Show the content of HDFS directory:
```
hdfs dfs -ls
```
Upload a file to HDFS:
```
hdfs dfs -put <localDocumentName> <HDFSDocumentName>
```
Download a file to Local from HDFS:
```
hdfs dfs -get <HDFS directory>/<HDFS filename> <Localfilename>
```
Remove a fields from HDFS:
```
hdfs dfs -rm -R [-skipTrash]
```
> :exclamation: Be careful with `-skipTrash` option because it will bypass trash, if enabled, and delete the specified file(s) immediately. This can be useful when it is necessary to delete files from an over-quota directory.

:bulb: More information about HDFS click in this [**link**](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html#), or put `hdfs dfs` in command line.

Yarn app list:
```
yarn application -list
```
Yarn kill process:
```
yarn application -kill <ApplicationID>
```

## Sqoop
Show all the sqoop options:
```
sqoop help
```
Show all the tables in a database:
```
sqoop list-tables --connect jdbc:mysql://<dataBase> --username <name> --password <password>
```
> You can see that we are using a jdbc connection to MySQL dataBase

Import a database with Sqoop and put in a base directory:
```
sqoop import-all-tables \
--connect jdbc:mysql://dbhost/<dataBase> --username  <name> --password <password> \
--warehouse-dir /<nameOfDirectory> \
```
- You can use `--target-dir` to specify a directory.
- `--target-dir` is incompatible with `--warehouse-dir`

Incremental import can be used by `lastmodified`:
```
sqoop import --table <nameTable> \
--connect jdbc:mysql://dbhost/<database> --username <name> --password <password> \
--incremental lastmodified \
--check-column <columnName> \
--last-value '<timeStampValue>' \
```

or we can use `append` mode:
```
sqoop import --table <nameTable> \
--connect jdbc:mysql://dbhost/<database> --username <name> --password <password> \
--incremental append \
--check-column <columnId> \
--last-value <valueId>
```

## Hive and Impala

### Impala in Shell
See all info in Impala Shell with next command:
```
impala-shell --help
```
Some interesting commands to begin:
```bash
# Start impala-shell
impala-shell
# Start impala-shell in a different server
impala-shell -i myserver.example.com:2100
```
Some interesting options:
* If you have a document with some queries, the `-f` can be used:
```
impala-shell -f myFileWithQueries.sql
```
* If you want to run queries directly from terminal use `-q`:
```
impala-shell -q 'SELECT * FROM users'
```
### Hive in Shell
Some interesting commands to begin:
```bash
# Start Hive
beeline
# Start beeline in a different server with corresponding name and password
beeline -u jdbc:hive2://localhost:10000/default -n scott -w password_file
```
Some interesting options:
* If you have a document with some queries, the `-f` can be used to run it:
```
beeline -f myFileWithQueries.hql
```
* If you want to run queries directly from terminal use `-e`:
```
impala-shell -e 'SELECT * FROM users'
```

:bulb: More information about the command options click in the [link](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients#HiveServer2Clients-BeelineCommandOptions)

### DDL

Create a new database:
```sql
CREATE DATABASE IF NOT EXISTS exampleDatabase
```
 > A database is simply an HDFS directory containing one or more files. its path is: `/user/hive/warehouse/exampleDatabase.db`

Drop a database:
```sql
DROP DATABASE IF EXISTS exampleDatabase
```
Create a table:
```sql
CREATE TABLE people (colname DATATYPE, ...)
ROW FORMAT DELIMITED FIELDS TERMINATED BY char
```
 > A table is simply an HDFS directory containing one or more files. its path is: `/user/hive/warehouse/people`


Create new table using `LIKE` statement:
```sql
CREATE TABLE people_otherCountry LIKE people
```
Create a new table using `SELECT` statement as well:
```sql
CREATE TABLE people_otherCountry AS SELECT * FROM people
```
If remove problems will be avoid, the `EXTERNAL` statement can be used:
```sql
CREATE EXTERNAL TABLE people (id INT,name STRING,telephone INT) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';
```
> Remember to use `Location` when a tables should be created in a specific directory

If the user wants to explore the tables in the current database: `SHOW TABLES`. Sometimes users wants to see
information about tables, users can use `DESCRIBE tableName` or `DESCRIBE FORMATTED tableName`.

Add data in tables uploading the info in the HDFS directory:
```bash
hdfs dfs -mv /tmp/people.txt /user/hive/warehouse/people/
```
or
```sql
LOAD DATA INPATH 'tmp/people.txt' INTO TABLE people
```
> This command moves the data just the command above.
:bulb: More information about [DML](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML#LanguageManualDML-HiveDataManipulationLanguage)

Remove all the data using the `OVERWRITE` as:
```sql
LOAD DATA INPATH 'tmp/people.txt' OVERWRITE INTO TABLE people
```

Another way is to populate a table is through a query using `INSERT INTO` statement:
```sql
INSERT INTO TABLE people_copy SELECT * FROM people
```

In case of using Impala,  is important to keep in mind *metastore* due to be outsite of impala.
The metastore could be changed by *Hive*, *HDFS*, *HCatalog* or *Metastore Manager*. Because of that, find below some important commands about that:

| External Metadata Change                                       | Required Actions |
|:---------------------------------------------------------------|:-----------------|
| New table added                                                | `INVALIDATE METADATA`   |
| Table schema modified or New data added to a table             | `REFRESH <table>`     |
| Data in a table extensively altered, such as by HDFS balancing | `INVALIDATE METADATA <table>`  |

:bulb: More information about the *DDL* options click in the [link](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL)


### Partitioning

We said before *"By default, a table is simply an HDFS directory containing one or more files"*, so all the data is located in the same place, in other words, a `full scan` is done in this folder because all files are read.
```sql
create table example_cities (num_pers int,city string) LOCATION /example_folder/cities
```
with partitioning, you divide the data. In time analysis you only read the relevant subsets
```sql
create table example_cities (num_pers int) PARTITIONED BY(city String) LOCATION /example_folder/example_cities
```
and the folder structure is:
```
|_ example_cities
      |
      |_ Barcelona
      |_ Paris
      |_ London
```
> :exclamation:  partition column is a virtual column, data is not stored in the field

The data can be loaded in partitioned table through 2 ways:  
1. Dynamic partitioning
2. Static partitioning

#### Dynamic
- Partitions are created based in the last column
- If the partition has not already exist, it will be created. If exists, it will be overwritten
```sql
INSERT OVERWRITE TABLE example_cities PARTITION(city) SELECT num_pers,city FROM example;
```
:bulb: More about [Dynamic partition configuration](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML#LanguageManualDML-DynamicPartitionInserts)
#### Static
```sql
ALTER TABLE example_cities ADD PARTITION (city = 'Madrid')
```
**Actions:**  
1. Adds the partition to the table metadata
2. Create a subdirectory
```
|_ example_cities
      |
      |_ Barcelona
      |_ Paris
      |_ London
      |_ Madrid
```
Then, the data has to be loaded
```sql
LOAD DATA INPATH '/user/data.log' INTO TABLE example_cities PARTITION (city = 'Madrid')
```

## Data formats

### Types of Data

**Text File**
- The most basic type in Hadoop
- Useful when debugging
- Representing numerics like string wastes storage space

**Sequence File**
- Less verbose than *text file*
- Capable of stoting binary data

**Avro File**
- Efficient storage due to optimized binary encoding
- Widely supported
- Ideal for long-term
  - Read/Write from a lot of languages
  -  Embed schema -> Readable data
  - Schema evolution can accommodate changes

**Columnar**
- Organize the information in columns
- Very efficient with small subsets of a table's column

**Parquet File**
- Schema metadata embedded in the file
- Advanced optimizations from [Dremel paper](https://static.googleusercontent.com/media/research.google.com/es//pubs/archive/36632.pdf)
- Most efficient when adding many records at once

### More deeply in Avro
To understand Avro, we must understand [*serialization*](https://en.wikipedia.org/wiki/Serialization), which is the process of translating data structures or object state into a format that can be stored (for example, in a file or memory buffer, or transmitted across a network connection link) and reconstructed later in the same or another computer environment.

Avro hold the next kind of types:
- **Simple**: *null, Boolean, int, long, float, double, bytes, String*
- **Complex** *record, enum, array, map, union, fixed*

A schema is represented in *JSON* by one of:

1. A *JSON* string, naming a defined type
2. A *JSON* object, of the form:
`{"type": "typeName ...attributes... "}`
where typeName is either a primitive or derived type name.
3. A *JSON* array, representing a union of embedded types.

An example of *Avro* schema:
```json
{"namespace": "example.avro",
 "type": "record",
 "name": "User",
 "fields": [
     {"name": "name", "type": "string"},
     {"name": "favorite_number",  "type": ["int", "null"]},
     {"name": "favorite_color", "type": ["string", "null"]}
 ]
}
```
> A schema file can only contain a single schema definition

At minimum, for this example a record definition must include:
- `type`
- `name`
- `fields`
- `nameSpace`
- `docs` (Optional)
- `aliases` (Optional)

> :bulb: More information about [field types](http://avro.apache.org/docs/current/spec.html#schema_declaration)

Following the schema of the example:
```
{"type": "record",
"name": "nameOfRecord",
"namespace": "qualifies the name",
"aliases": "alias" //Optional
"docs": "comments to clarify" //Optional
"fields": [{"name":...,"type":...,
          "default":...}] //Optional
}
```
The *full name Schema* is defined as = `nameSpace` + `name`. In this case, `example.avro.User`  

If you want to inspect Avro files, you can do it with *Avro Tools*.
```
This product includes software developed at
The Apache Software Foundation (http://www.apache.org/).
----------------
Available tools:
          cat  extracts samples from files
      compile  Generates Java code for the given schema.
       concat  Concatenates avro files without re-compressing.
   fragtojson  Renders a binary-encoded Avro datum as JSON.
     fromjson  Reads JSON records and writes an Avro data file.
     fromtext  Imports a text file into an avro data file.
      getmeta  Prints out the metadata of an Avro data file.
    getschema  Prints out schema of an Avro data file.
          idl  Generates a JSON schema from an Avro IDL file
 idl2schemata  Extract JSON schemata of the types from an Avro IDL file
       induce  Induce schema/protocol from Java class/interface via reflection.
   jsontofrag  Renders a JSON-encoded Avro datum as binary.
       random  Creates a file with randomly generated instances of a schema.
      recodec  Alters the codec of a data file.
       repair  Recovers data from a corrupt Avro Data file
  rpcprotocol  Output the protocol of a RPC service
   rpcreceive  Opens an RPC Server and listens for one message.
      rpcsend  Sends a single RPC message.
       tether  Run a tethered mapreduce job.
       tojson  Dumps an Avro data file as JSON, record per line or pretty.
       totext  Converts an Avro data file to a text file.
     totrevni  Converts an Avro data file to a Trevni file.
  trevni_meta  Dumps a Trevni file's metadata as JSON.
trevni_random  Create a Trevni file filled with random instances of a schema.
trevni_tojson  Dumps a Trevni file as JSON.
```

### Using Avro in Sqoop, Hive and Impala
If we want to export some table in sqoop:
```
sqoop import \
--connect jdbc:mysql://localhost/loudacre \
--username training --password training \
--table accounts \
--target-dir /loudacre/accounts_avro \
--as-avrodatafile
```
> The Sqoop import will save the json schema in local directory

If we want to use *Avro* in *Hive* or *Impala* keep in mind that:
- *Hive* supports all types
- *Impala* doesn't support complex types, [see more information in impala doc](https://www.cloudera.com/documentation/enterprise/5-4-x/topics/impala_avro.html#avro_data_types)

Add the schema to a table:
```sql
CREATE TABLE example_avro
STORED AS AVRO
TBLPROPERTIES ('avro.schema.url'=
'hdfs://localhost/example/schema.json');
```
or embed the schema:
```
CREATE TABLE example_avro
STORED AS AVRO
TBLPROPERTIES ('avro.schema.literal'=
'{"name": "order",
"type": "record",
"fields": [
  {"name":"id", "type":"int"},
  {"name":"cust_id", "type":"int"},
  {"name":"date", "type":"string"}]
}');
```
> :exclamation: For http schemas, this works for testing and small-scale clusters, but as the schema will be accessed at least once from each task in the job, this can quickly turn the job into a DDOS attack.

> :bulb: More information about [Avro in Hive](https://cwiki.apache.org/confluence/display/Hive/AvroSerDe#AvroSerDe-SpecifyingtheAvroschemaforatable)

### Using Parquet in Sqoop, Hive and Impala
If we want to export some table in sqoop:
```
sqoop import \
--connect jdbc:mysql://localhost/loudacre \
--username training --password training \
--table accounts \
--target-dir /loudacre/accounts_avro \
--as-parquetfile
```
Create table stored as *Parquet*:
```sql
CREATE TABLE example_parquet (
id INT,
prod_id INT)
STORED AS PARQUET
LOCATION '/folderExample/example_parquet';
```
## Flume

![Flume-dataflow](/images/flume.png)

An ***event*** is the *Flume* works unit and it is composed by:
- Body (payload)
- Headers (metadata)

The components of Flume architecture are:
- Source: event reciever
- Channel: Buffer
- Sink: event deliver

### configuration

Flume agent is configured through a java properties, we show an example:
```properties
# define components
agents1.sources = s1
agents1.channels = c1
agents1.sinks = s1

# define source from Syslog
agents1.sources.s1.type = syslogtcp
agents1.sources.s1.port = 5140
agents1.sources.s1.host = localhost
agents1.sources.s1.channels = c1

#define channel
agents1.channels.c1.type = memory

#define sink
agents1.sinks.s1.channel = c1
agents1.sinks.s1.hdfs.path = /flume/events/%y-%m-%d/%H%M/%S
agents1.sinks.s1.hdfs.filePrefix = events-
agents1.sinks.s1.hdfs.round = true
agents1.sinks.s1.hdfs.roundValue = 10
agents1.sinks.s1.hdfs.roundUnit = minute
```
:bulb: More information about [Flume](https://flume.apache.org/FlumeUserGuide.html)

## Spark

### Starting

If you want to begin with *Apache Spark*:
- Download the [package](http://d3kbcqa49mib13.cloudfront.net/spark-1.6.3-bin-hadoop2.6.tgz)
- Move to the parent folder and run `./bin/spark-shell`

Every *Spark application* requires a `sparkContext`, it is the main entry point to the *Spark* API.
You can see it in the image. If you want more information about `sparkContext` you can visit his [library](http://spark.apache.org/docs/1.6.3/api/scala/index.html#org.apache.spark.SparkContext)

![spark-shell](/images/spark-shell.png)

:mag_right: Use `:help` to see the help commands

The basic *Spark* unit is the ***RDD***:
- **R** esilient : If data is losed, it can be created again
- **D** istributed : Compute across the cluster
- **D** ataset : Data used to work with

There are two ways to create RDDs: parallelizing an existing collection in your driver program,
or referencing a dataset in an external storage system, such as a shared filesystem, HDFS, HBase,
or any data source offering a Hadoop InputFormat.

Remenber that:
- **RDD** are immutable
- Data is partioned accross the cluster and it is done automatically by *Spark*, but you
can control it.
- Transform to `Sequence` to modify the data as needed

And the RDD operations are divided in two blocks:
- `Transformations` arround the *pipelines*
- `Actions` to return values

To see the *transformations* and *action* concepts we would like to present
an example of *Create* a **RDD**. In our case, we use the *Quixote* intro to practise:
```bash
# Load the file in Spark memory
scala> sc.textFile("/home/exampleSpark/quixote.txt")
```

<!--TO DO: ADD example how to load a field
textFile
WholeTexFile
json
-->

Important thing is that *Spark* is **lazy evaluation** that means that
*Transformations* are not calculated until an action.
<!--TO DO: ADD example of map -->

Once you have seen some *maps* transformations, you could find interesting to  
evaluate the *lineage* excuation , we can use `.toDebugString` for that:
<!--TO DO: ADD example -->

Depending of what kind of transformations are you using, you have *skew* data, so be careful.


<!--TO DO:
ADD Spark terminology
ADD cache/persist
 -->

 Once you have seen something about Spark, we show you 3 important concepts:

 | Concept         | Description     |
 | :-------------  | :-------------  |
 | Job         | A set of tasks executed as a result of an action       |
 | Stage       | A set of task in a job    |
 | Task        | Unit of work  |


If you want to now more about **RDD**, you can visit the [API documentation](http://spark.apache.org/docs/1.6.3/api/scala/index.html#org.apache.spark.rdd.RDD) about this.

Some interesting thing to metion are *Pair RDD*. It will have `(key,value)` (*tuples*) structure, and it
has some addicional functions in his [PairRDDFunctions](http://spark.apache.org/docs/1.6.3/api/scala/index.html#org.apache.spark.rdd.PairRDDFunctions) in the **API**
<!--TO DO:
ADD example create a pair of fields
 apply a reduceByKey -->

### Runnings apps
 Spark applications run as independent sets of processes on a cluster, coordinated by the `SparkContext` object in your main program.
 I recommend you to read some interesting concepts about Spark in cluster in the [Spark glossary](http://spark.apache.org/docs/1.6.3/cluster-overview.html#glossary)

To launch a Spark Application, the `spark-submit` script is used for that.
If your code depends on other projects, you will need to package them alongside your application in order to distribute the code to a Spark cluster.
Both sbt and Maven have assembly plugins. When creating assembly jars, list Spark and Hadoop as provided dependencies;
these need not be bundled since they are provided by the cluster manager at runtime.

First of all, You can remember that if you need help, the `help` command will be there:
```Shell
spark-submit --help
```
Spark has many parameters to tune depeding of our needs, we are going to take a look:
<!--
http://spark.apache.org/docs/1.6.3/submitting-applications.html#launching-applications-with-spark-submit
http://spark.apache.org/docs/1.6.3/submitting-applications.html#master-urls
http://spark.apache.org/docs/1.6.3/configuration.html#spark-configuration
-->
