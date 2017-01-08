# Notes
These notes try to show the most useful command lines that I use in my daily work with Hadoop environment.

**Table of Content**

1. [HDFS and YARN](#hdfs-and-yarn)  
2. [Sqoop](#sqoop)  
3. [Hive and Impala](#hive-and-impala)
4. [Data formats](#data-formats)


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

More information about HDFS click in this [**link**](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html#), or put `hdfs dfs` in command line.

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

More information about the command options click in the [link](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients#HiveServer2Clients-BeelineCommandOptions)

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
> This command moves the data just the command above

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
