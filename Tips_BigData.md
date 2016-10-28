# Tips about Hadoop 
This notes try to show the most useful command lines that I use in my daily work with Hadoop environment.

## HDFS and YARN
Copy a file to HDFS:  
`hdfs dfs -put <localDocumentName> <HDFSDocumentName>`  
Copy a file to Local from HDFS:  
`hdfs dfs -get <HDFS directory>/<HDFS filename>  <Localfilename>`  
Get directory HDFS:  
`hdfs dfs -ls`  
Yarn apps list:  
`yarn application -list`   
Yarn kill process:  
`yarn application -kill <ApplicationID>`  
## Sqoop
Import a database with Sqoop and put in a directory:  
`sqoop import-all-tables --connect jdbc:mysql://dbhost/<nameOfDirectory>
--username <name> --password <password>
--warehouse-dir /<nameOfDirectory>`  
## Hive and Impala
Run files with Queries in Impala:  
`impala-shell -f <fileName>.sql`
