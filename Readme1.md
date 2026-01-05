# ZBD Data Engineering Assessment Submission

**Sean R. C. Cooperman**  
**December 27, 2025**

---

## Overview

This repository contains my submission for the ZBD Data Engineering assessment. The goal was to take a set of messy, denormalized CSV files and design a clean, analytics-ready data model, then provision an AWS analytics environment capable of supporting scalable reporting and future automation.

The focus of this submission is correctness, clarity of modeling decisions, and production-oriented thinking rather than raw volume or overengineering.

---

## What’s Included in This Repository

- A normalized relational data model (3rd Normal Form)
- PostgreSQL schema design used for initial data cleanup and normalization
- Amazon Redshift Serverless provisioning and schema recreation
- A clear ETL / migration plan from PostgreSQL to Redshift
- An optional design for automated mock data generation and continuous ingestion

---

## How to Review This Submission

1. Start with **Data Modeling & Normalization** to understand how messy source data was cleaned and structured.
2. Review the **Data Model Overview** and relationships to see how publishers, games, and transactions are represented.
3. Look at **Redshift Provisioning** and **Schema Creation** to understand how the model was adapted for analytics.
4. Review the **ETL / Migration Plan** for how data would be moved into Redshift in a production-ready way.

No application code was required for this assessment. The emphasis is on data modeling, cloud architecture, and scalability.

---

## Data Modeling & Normalization

The original CSV files contained several issues:

- Duplicate publishers and games (for example, “Valve”, “VALVE CORP”, etc.)
- Inconsistent references (sometimes IDs, sometimes names)
- Mixed and inconsistent date formats

These issues made reliable analysis difficult, especially for revenue calculations by publisher, platform, or region.

To address this, I normalized the data into a clean relational model following **3rd Normal Form (3NF)**. This removes redundancy, enforces referential integrity, and ensures consistent, queryable relationships across entities.

All publisher and game names were cleaned and standardized before loading.

---

## Data Model Overview

### Publishers

| Column       | Type   | Notes            |
|-------------|--------|------------------|
| publisher_id | SERIAL | Primary Key      |
| name         | TEXT   | UNIQUE, NOT NULL |
| country      | CHAR(2)| ISO country code |

### Games

| Column       | Type    | Notes                     |
|-------------|---------|---------------------------|
| game_id      | SERIAL  | Primary Key               |
| title        | TEXT    | UNIQUE, NOT NULL          |
| publisher_id | INTEGER | FK to publishers          |
| release_date | DATE    | Game release date         |
| genre        | TEXT    | Game genre                |

### Transactions

| Column         | Type         | Notes                    |
|---------------|--------------|--------------------------|
| transaction_id| CHAR(5)      | Primary Key              |
| txn_type      | TEXT         | sale, refund, payout     |
| txn_ts        | TIMESTAMPTZ  | Transaction timestamp    |
| currency      | CHAR(3)      | ISO currency code        |
| total_amount | NUMERIC(10,2)| Total transaction amount |
| customer_ref | TEXT         | Optional reference       |

### Transaction_Lines

| Column         | Type         | Notes                                |
|---------------|--------------|--------------------------------------|
| transaction_id| CHAR(5)      | FK to transactions                   |
| line_no       | INTEGER      | Composite PK with transaction_id     |
| game_id       | INTEGER      | FK to games                          |
| region        | TEXT         | Region of sale                       |
| platform      | TEXT         | Platform (PC, Console, etc.)         |
| gross_amount | NUMERIC(10,2)| Gross line amount                    |
| platform_fee | NUMERIC(10,2)| Platform fee                         |
| net_amount   | NUMERIC(10,2)| Net amount                           |

**Primary Key:** (transaction_id, line_no)

---

## Model Walkthrough

- **Publishers**  
  Single source of truth for publisher data with standardized naming.

- **Games**  
  Deduplicated titles linked to publishers.

- **Transactions**  
  Header-level transaction data.

- **Transaction_Lines**  
  Line-level detail supporting bundles, platforms, and regions.

### Relationships

- One Publisher → many Games  
- One Game → many Transaction_Lines  
- One Transaction → many Transaction_Lines

---

## Redshift Provisioning

An **Amazon Redshift Serverless** workgroup was provisioned in the same region as PostgreSQL RDS (us-east-2).

Key choices:
- Low base capacity (32 RPU) for cost control
- Public access with restricted security group rules
- Default VPC configuration

---

## Schema Creation in Redshift Serverless

The schema was recreated using Query Editor v2 with adjustments for Serverless:

- `IDENTITY` columns instead of `SERIAL`
- `DECIMAL` for monetary values
- Foreign keys and explicit DISTKEY/SORTKEY omitted initially, relying on Serverless optimization

All tables were validated successfully.

---

## ETL / Data Migration Plan

1. Export PostgreSQL tables to S3 using `aws_s3.table_export_to_s3`
2. Attach an IAM role to Redshift Serverless
3. Load data via Redshift `COPY` commands

This approach is fast, secure, and cost-efficient.

---

## Automation & Continuous Data Generation (Optional)

Proposed approach:
- DBT for mock data generation
- Airflow (MWAA) for orchestration
- Fivetran or AWS Glue for ingestion

Benefits:
- Fully managed
- Scalable
- Production-aligned

---

## Summary

This submission demonstrates a clean, normalized data model and a production-oriented analytics architecture using AWS managed services. The environment is provisioned, validated, and ready for analytics workloads.

Thank you for the opportunity.

**Sean R. C. Cooperman**  
**December 27, 2025**
