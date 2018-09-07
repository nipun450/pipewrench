# Sqoop to IMPALA Pipeline
This pipeline consists of the following steps
1. Sqoop data from a relational database into an Avro backed table. 
2. The data is inserted into a partitioned Parquet table. The different partitions consist of different historical ingests(partion 1 consists of the first ingest, partition 2 the second ingest, and so on). There are a total of 4 partitions which round robin (after data is inserted into the fourth partition, it is ingested into the first partition again). An HDFS file called latest_partition tracks which partition was last ingested into. 
3. The final report table (the one queried by end users) is altered to point to the latest partition 
This pipeline will sqoop data into a Avro (staging) table insert data to Parquet table with partitions and creates destination 
table pointing the location to fresh copy of data inserted

## Artifacts Created
- Impala Avro table
- Impala Parquet table with partitions
- Impala Parquet (Destination) table 
- Sqoop import job

## Reasons the pipeline is designed this way.
1. Avro is chosen as the file format because it uses row based compression, and is less likely to run into memory errors in comparsion to Parquet.
2. A partitoned table is used in the second step to maintain history for ingests.
3. The final table is always available for querying because it is never directly inserted into, and is just pointed to the correct partition whenever it is ready

## Running the pipeline

`make first-run` :
- create and execute Sqoop import job
- create Impala avro tables
- create Impala parquet table
- create Destination table
- Insert data into Parquet from Avro table by casting column datatypes

`make update` :
- execute the Sqoop job
- updates data in Avro table
- Insert data into new partition of Parquet from Avro table by casting column datatypes
- Alter destination table location to the new partition

`make clean` :
- Clean up _all_ data on HDFS and Impala DDL. This is non reversible.
