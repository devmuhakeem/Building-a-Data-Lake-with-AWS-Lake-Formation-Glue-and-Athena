# Building a Data Lake with AWS Lake Formation, Glue, and Athena

A hands-on AWS lab where I built a data lake from a raw CSV dataset, crawled it to generate queryable metadata, transformed it into a more efficient columnar format, and measured the real performance difference between the two.

## Scenario
Acting as a data engineer, I was tasked with piloting a data lake for a movies dataset — registering S3 storage with Lake Formation, cataloging the raw data, querying it with Athena, then converting it to a more efficient format and proving the performance gain.

## What I did

### 1. Registered S3 storage with Lake Formation
Registered an S3 bucket location as a managed data lake storage location, using a dedicated IAM service role scoped to the exact S3 actions needed (`PutObject`, `GetObject`, `DeleteObject`, `ListBucket`).

### 2. Created a Glue Data Catalog database
Created a `movies-db` database to hold the metadata (schema, location, partitions) describing the underlying data — the catalog itself holds no data, just the structure describing it.

### 3. Crawled the raw data with AWS Glue
Built a Glue crawler pointed at the S3 source data, ran it, and let it automatically infer the schema from a raw CSV file (movie titles, ratings, genres, cast, directors from 1920–2018) and register a table in the catalog — reviewed the crawler's CloudWatch logs to confirm partition creation.

### 4. Queried the raw data with Athena
Granted myself SELECT permissions via Lake Formation, then ran SQL directly against the cataloged CSV data using Athena — including a query confirming exactly 1,002 movies had "Action" as their primary genre.

### 5. Transformed the data to Parquet with a Glue ETL job
Built a visual Glue ETL job reading from the Glue Data Catalog and writing the same dataset out in Parquet format — a columnar storage format — to a new S3 location, then re-ran the crawler to register the new Parquet table.

### 6. Measured the real performance difference
Ran the identical "count Action movies" query against both the CSV table and the Parquet table. Run time was comparable, but the **data scanned** for the Parquet query was dramatically smaller than for CSV — despite returning the exact same 1,002-row result.

## Key takeaways
- A data catalog separates metadata from data — tables in Glue's catalog don't store data themselves, they describe where it lives and what shape it's in, which is what lets services like Athena query it without moving it
- Parquet's columnar format lets a query scan only the specific columns it needs, instead of reading every row and column like CSV forces you to — at scale (a million+ rows), that difference compounds into real time and cost savings
- Lake Formation's permission model is granular down to the table level — registering storage and creating a database doesn't automatically grant query access; that's a separate, deliberate grant

## Tools
AWS Lake Formation, AWS Glue (Crawlers + ETL), Amazon Athena, Amazon S3, IAM

---
*Completed as an AWS hands-on lab.*
