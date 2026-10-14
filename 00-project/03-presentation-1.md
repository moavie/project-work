---
project_outline: "[[00-project/01-project-outline]]"
---


<!-- 5xl -->
**# League of Legends Match Data Warehouse**

Author: Le Dinh Trung Tin 
Neptun: L94273  
Supervisor: Anett Lenart  
Project ID: 2026-LA-02

---

## Problem

League of Legends generates large amounts of match data, including player performance, champion selection, roles, match results, and game patches. However, analyzing historical data across these different dimensions requires collecting, organizing, and processing the data consistently.

This project aims to build a data warehouse that transforms raw match data from the Riot Games API into structured data suitable for historical and comparative analysis.

## Target users

| **Role** | **Needs** | **Interaction** |
|---|---|---|
| **Data analyst / researcher** | Analyze champion performance, player statistics, and historical gameplay trends | Run analytical SQL queries and compare results across patches, roles, and time |
| **Data engineer / database developer** | Ensure reliable data collection, transformation, storage, and query performance | Manage data pipelines, validate records, and inspect database performance |

## Planned scope

**In scope**

- Collect match data from the Riot Games API and store raw responses in a staging layer.
- Build a data processing pipeline for data validation, transformation, and incremental loading.
- Design a dimensional data warehouse with a clearly defined fact table granularity.
- Support historical analysis by player, champion, role, patch, match type, and time where data is available.
- Develop at least 10 complex analytical SQL queries and evaluate optimizations for at least 5 queries.
- Compare star and snowflake schemas and investigate at least two historization approaches.
- Test the system with an increasingly large dataset and document data quality and query performance.

**Out of scope**

- Developing a dedicated website, interactive dashboard, or mobile application.
- Collecting and processing all available League of Legends matches without a defined collection scope.

## Initial user journeys

### 1. Collecting match data

1. The data engineer starts the data collection process.
2. The system retrieves match data from the Riot Games API and stores the raw responses in the staging layer.
3. The system records collection status and identifies failed requests or duplicate records.

**Result:** match data is available for processing and historical analysis.

### 2. Analyzing match data

1. An analyst wants to compare champion performance or investigate changes across game patches.
2. The analyst runs an SQL query against the data warehouse.
3. The results summarize statistics such as match counts and win rates, grouped by champion, role, or patch.

**Result:** the analyst can compare gameplay statistics and explore historical trends using structured match data.

### 3. Checking data quality

1. The data engineer wants to verify that the warehouse contains consistent data.
2. The engineer runs validation checks for missing values, duplicate records, and invalid relationships between fact and dimension tables.
3. The system records detected issues for further investigation.

**Result:** potential data quality problems can be identified before they affect analytical results.


## Next steps

- Review the Riot Games API documentation, available endpoints, and rate limits.
- Test data collection with a small sample of match data.
- Define the fact table granularity and design an initial dimensional data model.
- Compare star and snowflake schemas and investigate suitable historization approaches.
- Evaluate database technologies and implementation options.
- Prepare initial analytical SQL queries and a plan for measuring query performance.





