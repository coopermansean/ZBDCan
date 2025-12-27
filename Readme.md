\# ZBD Data Engineering Assessment Submission

\# ZBD Data Engineering Assessment Submission



\*\*Sean R.C. Cooperman\*\*  

\*\*December 27, 2025\*\*



\## Data Modeling \& Normalization



The original CSV files were messy, with duplicates (for example, Valve appeared as “Valve,” “VALVE CORP,” etc.), inconsistent references (sometimes by ID, sometimes by name), and mixed date formats. This made reliable analysis—like calculating total revenue per publisher—very difficult.



To address this, I normalized the data into a clean relational model following 3rd Normal Form. This eliminates redundancy, ensures referential integrity, and makes queries reliable and consistent. All publisher and game names were cleaned and standardized to maintain consistent references across tables.



\### Data Model Overview



Publishers

Column          | Type     | Notes

-----------------+----------+------------------------

publisher\_id     | SERIAL   | Primary Key

name             | TEXT     | UNIQUE NOT NULL

country          | CHAR(2)  | Country code



Games

Column           | Type     | Notes

-----------------+----------+------------------------

game\_id          | SERIAL   | Primary Key

title            | TEXT     | UNIQUE NOT NULL

publisher\_id     | INTEGER  | FK ? Publishers

release\_date     | DATE     | Game release date

genre            | TEXT     | Game genre



Transactions

Column           | Type         | Notes

-----------------+--------------+------------------------

transaction\_id   | CHAR(5)      | Primary Key

txn\_type         | TEXT         | sale/refund/payout

txn\_ts           | TIMESTAMPTZ  | Transaction timestamp

currency         | CHAR(3)      | Currency code

total\_amount     | NUMERIC(10,2)| Total transaction amount

customer\_ref     | TEXT         | Optional customer reference



Transaction\_Lines

Column           | Type         | Notes

-----------------+--------------+------------------------

transaction\_id   | CHAR(5)      | FK Transactions

line\_no          | INTEGER      | Composite PK with transaction\_id

game\_id          | INTEGER      | FK Games

region           | TEXT         | Region of sale

platform         | TEXT         | Platform (PC, Console, etc.)

gross\_amount     | NUMERIC(10,2)| Gross amount of line

platform\_fee     | NUMERIC(10,2)| Platform fee

net\_amount       | NUMERIC(10,2)| Net amount



Primary Key: (transaction\_id, line\_no)

\### Walkthrough of the Model



\- \*\*Publishers\*\*: Single source of truth for publisher information. Each publisher has a unique `publisher\_id`, `name`, and `country`. Deduplication and standardization were applied to avoid inconsistent entries.



\- \*\*Games\*\*: Each publisher can have many games. The Games table stores deduplicated titles linked to their publisher via `publisher\_id`, along with `release\_date` and `genre`. This ensures every game has a consistent reference to its publisher.



\- \*\*Transactions\*\*: Captures header-level data for each sale, refund, or payout. Each transaction includes `transaction\_id`, type (sale, refund, or payout), timestamp, currency, total amount, and an optional customer reference.



\- \*\*Transaction\_Lines\*\*: Each line links a transaction to a specific game. This table supports multi-platform purchases and bundles, and includes `region`, `platform`, `gross\_amount`, `platform\_fee`, and `net\_amount`. The primary key is a composite of `transaction\_id` and `line\_no` to ensure uniqueness within each transaction.



\### Relationships



\- One Publisher → many Games  

\- One Game → many Transaction\_Lines  

\- One Transaction → many Transaction\_Lines  



This normalized structure ensures all entities map cleanly to real-world concepts. It eliminates redundancy, enforces referential integrity, and maintains consistent IDs and names. It also makes querying straightforward, allowing accurate revenue calculations by publisher, platform, or region, while the composite key and constraints maintain data integrity and optimize performance.



\## Redshift Provisioning



I provisioned an \*\*Amazon Redshift Serverless\*\* workgroup in the same region as the RDS instance (us-east-2).  



Key configuration choices:  

\- Started with a low base capacity (32 RPU) to control costs during testing  

\- Enabled public access after creation (required Elastic IP allocation and appropriate security group rules on port 5439)  

\- Used the default VPC and security settings, with inbound rules allowing access from my IP range  



This setup provides an auto-scaling analytics environment ideal for running complex queries on the normalized dataset.



\## Schema Creation in Redshift Serverless



I connected to the development database in the Redshift Serverless workgroup using Query Editor v2 and recreated the normalized schema with Redshift-compatible data types and constraints.  



All four tables (`publishers`, `games`, `transactions`, and `transaction\_lines`) were successfully created. Adjustments for Serverless compatibility included using `IDENTITY` for primary keys, `DECIMAL` for monetary fields, and omitting foreign keys and explicit `DISTKEY`/`SORTKEY`, since Redshift Serverless handles distribution and sorting automatically.  



The schema is now fully staged and ready for analytics workloads.



\## ETL / Data Migration Plan from PostgreSQL RDS to Redshift Serverless



To move the data from RDS PostgreSQL to Redshift, my preferred method is a simple, native AWS pipeline using S3 as an intermediary:



1\. Create an S3 bucket in the same region.  

2\. Export the normalized tables from RDS to CSVs in S3 using the built-in `aws\_s3` extension (`aws\_s3.table\_export\_to\_s3`).  

3\. Attach an IAM role with S3 read access to the Redshift Serverless workgroup.  

4\. Load the data using Redshift `COPY` commands from S3 (CSV format with automatic timestamp handling).  



This approach is fast, secure, serverless-friendly, and cost-efficient. Due to time constraints in the assessment, I focused on provisioning Redshift Serverless and validating the schema, but the full data load can be executed quickly in a follow-up.



\## Automation of Mock Data Generation and Continuous Population (Optional)



Given time constraints, I’m outlining my preferred approach rather than implementing it fully. This method leverages \*\*Fivetran\*\* (or \*\*AWS Glue\*\*) for automated, scheduled ingestion, combined with \*\*DBT\*\*-based data generation, orchestrated via \*\*Airflow (MWAA)\*\* or other applications.



\### High-level design



1\. Create a lightweight “mock source” schema in PostgreSQL mimicking the original denormalized structure.  

2\. Use DBT seeds and models with PostgreSQL random functions to generate realistic new transactions daily or hourly, referencing existing games, realistic bundles, and refunds.  

3\. Schedule DBT jobs via Airflow DAGs or other applications.  

4\. Use Fivetran (PostgreSQL connector with CDC) or Glue to sync changes into the normalized schema, or have DBT transform directly into the clean tables.  

5\. Extend the existing S3 export + Redshift COPY process into a scheduled Airflow or Glue job to keep Redshift current.



\### Benefits



\- Fully managed, observable, and scalable  

\- Aligns with modern data stack practices I’ve successfully implemented in prior roles  

\- Ensures consistent data integrity while scaling from test to production volumes



\## Summary



I analyzed the messy source data, designed and implemented a normalized 3NF schema in PostgreSQL RDS, and populated it with cleaned records. I provisioned Redshift Serverless, recreated the schema with adjustments for Serverless, and validated the environment. It’s now ready for analytics workloads and scalable ETL processes.



For reference, I’ve included details on the PostgreSQL RDS instance (configuration, schema, etc.) and the Redshift Serverless instance (workgroup settings, schema adjustments, etc.) in the repository.



Thank you again for the opportunity.

\*\*Sean R.C. Cooperman\*\*  

\*\*December 27, 2025\*\*



\## Data Modeling \& Normalization



The original CSV files were messy, with duplicates (for example, Valve appeared as “Valve,” “VALVE CORP,” etc.), inconsistent references (sometimes by ID, sometimes by name), and mixed date formats. This made reliable analysis—like calculating total revenue per publisher—very difficult.



To address this, I normalized the data into a clean relational model following 3rd Normal Form. This eliminates redundancy, ensures referential integrity, and makes queries reliable and consistent. All publisher and game names were cleaned and standardized to maintain consistent references across tables.



\### Data Model Overview



I created the data model in Notepad++ for readability, then took a screenshot to include for easier reference.



\### Walkthrough of the Model



\- \*\*Publishers\*\*: Single source of truth for publisher information. Each publisher has a unique `publisher\_id`, `name`, and `country`. Deduplication and standardization were applied to avoid inconsistent entries.



\- \*\*Games\*\*: Each publisher can have many games. The Games table stores deduplicated titles linked to their publisher via `publisher\_id`, along with `release\_date` and `genre`. This ensures every game has a consistent reference to its publisher.



\- \*\*Transactions\*\*: Captures header-level data for each sale, refund, or payout. Each transaction includes `transaction\_id`, type (sale, refund, or payout), timestamp, currency, total amount, and an optional customer reference.



\- \*\*Transaction\_Lines\*\*: Each line links a transaction to a specific game. This table supports multi-platform purchases and bundles, and includes `region`, `platform`, `gross\_amount`, `platform\_fee`, and `net\_amount`. The primary key is a composite of `transaction\_id` and `line\_no` to ensure uniqueness within each transaction.



\### Relationships



\- One Publisher → many Games  

\- One Game → many Transaction\_Lines  

\- One Transaction → many Transaction\_Lines  



This normalized structure ensures all entities map cleanly to real-world concepts. It eliminates redundancy, enforces referential integrity, and maintains consistent IDs and names. It also makes querying straightforward, allowing accurate revenue calculations by publisher, platform, or region, while the composite key and constraints maintain data integrity and optimize performance.



\## Redshift Provisioning



I provisioned an \*\*Amazon Redshift Serverless\*\* workgroup in the same region as the RDS instance (us-east-2).  



Key configuration choices:  

\- Started with a low base capacity (32 RPU) to control costs during testing  

\- Enabled public access after creation (required Elastic IP allocation and appropriate security group rules on port 5439)  

\- Used the default VPC and security settings, with inbound rules allowing access from my IP range  



This setup provides an auto-scaling analytics environment ideal for running complex queries on the normalized dataset.



\## Schema Creation in Redshift Serverless



I connected to the development database in the Redshift Serverless workgroup using Query Editor v2 and recreated the normalized schema with Redshift-compatible data types and constraints.  



All four tables (`publishers`, `games`, `transactions`, and `transaction\_lines`) were successfully created. Adjustments for Serverless compatibility included using `IDENTITY` for primary keys, `DECIMAL` for monetary fields, and omitting foreign keys and explicit `DISTKEY`/`SORTKEY`, since Redshift Serverless handles distribution and sorting automatically.  



The schema is now fully staged and ready for analytics workloads.



\## ETL / Data Migration Plan from PostgreSQL RDS to Redshift Serverless



To move the data from RDS PostgreSQL to Redshift, my preferred method is a simple, native AWS pipeline using S3 as an intermediary:



1\. Create an S3 bucket in the same region.  

2\. Export the normalized tables from RDS to CSVs in S3 using the built-in `aws\_s3` extension (`aws\_s3.table\_export\_to\_s3`).  

3\. Attach an IAM role with S3 read access to the Redshift Serverless workgroup.  

4\. Load the data using Redshift `COPY` commands from S3 (CSV format with automatic timestamp handling).  



This approach is fast, secure, serverless-friendly, and cost-efficient. Due to time constraints in the assessment, I focused on provisioning Redshift Serverless and validating the schema, but the full data load can be executed quickly in a follow-up.



\## Automation of Mock Data Generation and Continuous Population (Optional)



Given time constraints, I’m outlining my preferred approach rather than implementing it fully. This method leverages \*\*Fivetran\*\* (or \*\*AWS Glue\*\*) for automated, scheduled ingestion, combined with \*\*DBT\*\*-based data generation, orchestrated via \*\*Airflow (MWAA)\*\* or other applications.



\### High-level design



1\. Create a lightweight “mock source” schema in PostgreSQL mimicking the original denormalized structure.  

2\. Use DBT seeds and models with PostgreSQL random functions to generate realistic new transactions daily or hourly, referencing existing games, realistic bundles, and refunds.  

3\. Schedule DBT jobs via Airflow DAGs or other applications.  

4\. Use Fivetran (PostgreSQL connector with CDC) or Glue to sync changes into the normalized schema, or have DBT transform directly into the clean tables.  

5\. Extend the existing S3 export + Redshift COPY process into a scheduled Airflow or Glue job to keep Redshift current.



\### Benefits



\- Fully managed, observable, and scalable  

\- Aligns with modern data stack practices I’ve successfully implemented in prior roles  

\- Ensures consistent data integrity while scaling from test to production volumes



\## Summary



I analyzed the messy source data, designed and implemented a normalized 3NF schema in PostgreSQL RDS, and populated it with cleaned records. I provisioned Redshift Serverless, recreated the schema with adjustments for Serverless, and validated the environment. It’s now ready for analytics workloads and scalable ETL processes.



For reference, I’ve included details on the PostgreSQL RDS instance (configuration, schema, etc.) and the Redshift Serverless instance (workgroup settings, schema adjustments, etc.) in the repository.



Thank you again for the opportunity.

