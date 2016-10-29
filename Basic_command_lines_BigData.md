# Notes
These notes try to show the most useful command lines that I use in my daily work with Hadoop environment.

## HDFS and YARN
Show the content of HDFS directory:  
`hdfs dfs -ls`  
Upload a file to HDFS:  
`hdfs dfs -put <localDocumentName> <HDFSDocumentName>`  
Download a file to Local from HDFS:  
`hdfs dfs -get <HDFS directory>/<HDFS filename>  <Localfilename>`  
Remove a fields from HDFS:  
`hdfs dfs -rm -R [-skipTrash]`  
> :exclamation: Be careful with `-skipTrash` option because it will bypass trash, if enabled, and delete the specified file(s) immediately. This can be useful when it is necessary to delete files from an over-quota directory.

More information about HDFS click in this [**link**](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html#),  or put `hdfs dfs` in command line.

Yarn app list:  
`yarn application -list`   
Yarn kill process:  
`yarn application -kill <ApplicationID>`

## Sqoop
Show all the sqoop options:  
`sqoop help`  
Show all the tables in a database:  
`sqoop list-tables --connect jdbc:mysql://<dataBase> --username <name> --password <password>`
> You can see that we are using a jdbc connection to MySQL dataBase

Import a database with Sqoop and put in a directory:  
`sqoop import-all-tables --connect jdbc:mysql://dbhost/<nameOfDirectory>
--username <name> --password <password>
--warehouse-dir /<nameOfDirectory>`

## Hive and Impala
Run files with Queries in Impala:  
`impala-shell -f <fileName>.sql`
