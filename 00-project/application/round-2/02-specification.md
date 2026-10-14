---
project_outline: "[[00-project/01-project-outline]]"
---
# League of Legends Match Data Warehouse

## 1. Actors and permissions

| Role | Needs | Responsibility / access |
|---|---|---|
| Data Engineer | Collect and process match data | Runs the extraction and ETL pipeline and manages the data warehouse. |
| Data Analyst / SQL User | Analyze match data | Runs SQL queries and uses the results to analyze champion and player performance. |  

## 2. Use Cases / User Stories

### UC-01: Retrieve Match Data

- **Actor:** Data Engineer
- **External system:** Riot Games API
- **Precondition:** A valid API key is configured, the required API endpoints are available, and the retrieval parameters are defined.

**Main flow:**

1. The Data Engineer starts the data extraction process.
2. The system retrieves eligible match IDs and requests the corresponding match details from the Riot Games API.
3. The API returns match data in JSON format.
4. The system checks the responses and passes successfully retrieved data to the staging process.
5. The system records the retrieval results.

**Error flow:**

1. If authentication fails or match data are unavailable, the system records the error.
2. If a rate limit or temporary network error occurs, the system retries according to the configured policy.
3. If a request still fails, the system logs the failure and continues with other eligible matches when possible.

**Postcondition:**

- Successfully retrieved data are available for staging.
- Failed requests are recorded and are not reported as successful.

### UC-02: Store Raw Match Data

- **Actor:** Data Engineer
- **Precondition:** Match data have been retrieved successfully, and the staging database is available.

**Main flow:**

1. The system receives the raw match JSON data and identifies each match using its source match ID.
2. The system checks for existing records and applies the defined duplicate-handling rule.
3. The system stores the original JSON payload together with relevant retrieval metadata.
4. The system records the storage status.

**Error flow:**

1. If the database connection or storage transaction fails, the system records the error.
2. If the payload is empty or invalid, the system rejects it and records the reason.
3. If a duplicate is detected, the system handles it according to the defined policy.

**Postcondition:**

- Successfully stored raw data remain available for later processing and traceability.
- Failed storage operations are recorded.

### UC-03: Transform and Load Data

- **Actor:** Data Engineer
- **Precondition:** Eligible raw data are available in staging, warehouse tables have been created, and transformation rules are defined.

**Main flow:**

1. The Data Engineer starts the ETL/ELT process.
2. The system extracts match and participant information from the staged JSON data and standardizes the values.
3. The system prepares and loads the required dimension records.
4. The system maps participant data to dimension keys and loads fact records according to the defined fact table granularity.
5. The system records the numbers of processed, loaded, and rejected records.

**Error flow:**

1. If required fields are missing or the JSON structure is unexpected, the system records the transformation error.
2. If dimension keys, database constraints, or loading operations fail, the system records the affected records.
3. Failed records are handled according to the defined error-handling policy and may be reprocessed when appropriate.

**Postcondition:**

- Successfully processed data are stored in the appropriate warehouse tables.
- Failed or incomplete records are identified and recorded.

### UC-04: Validate Data Quality

- **Actor:** Data Engineer
- **Precondition:** Raw or transformed records are available, and data quality rules are defined.

**Main flow:**

1. The Data Engineer starts the validation process, or the system performs the checks during ETL/ELT.
2. The system checks required values, data types, duplicates, and fact-to-dimension references.
3. The system compares selected warehouse values with the corresponding source data.
4. The system records validation results and identifies invalid or inconsistent records.

**Error flow:**

1. If data quality problems are found, the system flags the affected records for correction, rejection, or investigation.
2. If a technical error prevents validation, the system records the failure and marks the validation as incomplete.

**Postcondition:**

- Validation results identify records that pass the defined checks and those requiring further investigation.
- Original source data remain available for traceability.

### UC-05: Perform Incremental Data Loading

- **Actor:** Data Engineer
- **Precondition:** The initial warehouse loading process is implemented, and a valid loading checkpoint or equivalent processing status is maintained.

**Main flow:**

1. The Data Engineer starts the incremental loading process.
2. The system reads the last successfully committed checkpoint and identifies new or unprocessed match records.
3. The system checks for existing records, then validates and transforms eligible data.
4. The system loads new dimension and fact records and updates existing records when required.
5. After successful loading, the system commits the changes, updates the checkpoint, and records the processing results.

**Error flow:**

1. If the checkpoint is missing or invalid, the system records the problem and follows the documented recovery procedure.
2. If the API, database, or loading process fails, the system preserves the last committed checkpoint.
3. If duplicate or invalid records are encountered, the system applies the defined handling policy and records the affected records.

**Postcondition:**

- New eligible records are loaded without unintended duplicates.
- The checkpoint reflects successfully committed progress.
- Failed records can be investigated and reprocessed.

### UC-06: Analyze Match Data

- **Actor:** Data Analyst / SQL User
- **Precondition:** The warehouse contains accessible data, and the user has permission to execute analytical queries.

**Main flow:**

1. The Data Analyst selects an analytical question and prepares the corresponding SQL query.
2. The database retrieves and joins the required fact and dimension data.
3. The database aggregates the data and calculates relevant metrics, such as win rate, pick rate, and player performance.
4. The database returns the results in tabular form.
5. The Data Analyst interprets the results and documents findings when required.

**Error flow:**

1. If the SQL query is invalid or the user lacks permission, the database reports an error.
2. If no records are returned or execution limits are exceeded, the user investigates the filters or database error.
3. If incomplete source data affect the results, the limitation is documented.

**Postcondition:**

- The user receives analytical results based on the available warehouse data.
- The results can support comparisons across champions, players, roles, patches, and match types.

### UC-07: Measure and Optimize Query Performance

- **Actor:** Data Engineer
- **Precondition:** The warehouse contains a sufficiently large test dataset, at least five analytical queries have been selected, and execution statistics can be collected.

**Main flow:**

1. The Data Engineer selects at least five representative analytical queries and measures their baseline performance.
2. The Data Engineer examines execution plans to identify bottlenecks.
3. The Data Engineer applies suitable optimizations, such as indexing, query rewriting, or partitioning where justified.
4. The system executes the same queries under comparable conditions, and the Data Engineer records the new measurements.
5. The Data Engineer compares performance, verifies result correctness, and documents the findings.

**Error flow:**

1. If the dataset or available execution statistics are insufficient, the limitation is documented.
2. If an optimization changes query results, fails to improve performance, or introduces unacceptable overhead, it is corrected, revised, or reverted.
3. If reliable measurements cannot be obtained, the test limitations are documented.

**Postcondition:**

- Before-and-after performance measurements for at least five queries are documented.
- Optimization decisions are justified, and query correctness is verified.
- The evaluation conditions and limitations are recorded.

## 3. Functional requirements

| Requirement | Acceptance Criteria | Priority |
|---|---|---|
| The system retrieves match data via the Riot Games API. | Given a valid match ID, when extraction runs, then the system retrieves the match JSON payload. | Must |
| The system stores raw data in a staging layer. | Given retrieved match data, when ingestion runs, then the raw payload and relevant metadata are stored in the staging database. | Must |
| The ETL pipeline transforms raw data into dimensional tables. | Given valid staging data, when transformation runs, then the required fact and dimension tables are populated according to the defined data model. | Must |
| The system validates data quality. | Given source or transformed data, when validation runs, then missing values, invalid records, duplicates, and inconsistent references are identified. | Must |
| The system supports incremental data loading. | Given previously loaded data and new match records, when incremental loading runs, then new records are loaded without unintended duplicates. | Must |
| The warehouse supports analytical SQL queries. | Given stored match data, when analytical queries run, then metrics such as win rates, pick rates, and performance comparisons can be calculated by champion, player, role, patch, and time where data are available. | Must |
| The project evaluates query performance. | Given at least five analytical queries, when optimization is applied, then performance before and after optimization is measured and documented. | Must |
| The project compares dimensional modelling and historization approaches. | Given the project requirements, when the alternatives are evaluated, then star and snowflake schemas and at least two historization approaches are compared, with the selected approaches justified. | Must |
| The system handles API rate limits and processing failures. | Given a rate limit or temporary failure, when the error occurs, then the system follows the defined retry or recovery policy and records the failure. | Should |

Priority: **Must** = required for the final product; **Should** = important; **Could** = optional if time permits.

## 4. Business Rules and Constraints

| Rule or constraint | Rationale |
|---|---|
| API keys must never be hardcoded or committed to version control. | Protects credentials and prevents unauthorized API access. |
| Data extraction must respect Riot Games API rate limits. | Prevents excessive requests and ensures compliant API usage. |
| Each fact record must follow the defined fact table granularity. | Ensures consistent aggregation and prevents double-counting. |
| Fact and dimension tables must maintain referential integrity. | Prevents orphaned records and ensures reliable analytical results. |
| Duplicate match records must be identified and handled consistently. | Prevents duplicate data from affecting analytical results. |
| Missing or invalid source data must be identified and handled according to defined rules. | Maintains data quality and makes data limitations traceable. |
| Incremental loading must not create unintended duplicate records. | Preserves warehouse consistency during repeated loading operations. |
| Historical match data must be retained to support analysis across time and patches. | Enables historical comparisons and trend analysis. |
| Data extraction and processing are limited by API availability, rate limits, and the data accessible through the selected endpoints. | Defines the practical limits of data collection and dataset completeness. |
| Analytical results must be based on the data available in the warehouse. | Prevents unsupported conclusions when data are incomplete or unavailable. |

## 5. User Interface and Workflow

The system primarily uses command-line scripts, database management tools, and SQL query interfaces rather than a dedicated graphical user interface.

The workflow consists of retrieving match data from the Riot Games API, storing raw data in the staging layer, transforming and loading the data into the data warehouse, and executing analytical SQL queries. Users can inspect staging and warehouse tables, validate data, and evaluate query performance using database management tools.

Data extraction and loading are initiated through scripts, while analytical results are presented in tabular form through SQL query interfaces. A dedicated website or dashboard is outside the scope of this project.

## 6. Non-functional requirements

| Requirement | How it will be verified |
|---|---|
| The ETL pipeline shall process match data reliably under normal operating conditions. | Test data ingestion and loading with a representative batch and record execution time and failures. |
| Analytical queries shall provide acceptable performance on the populated dataset. | Measure execution times for representative analytical queries and document the results. |
| The data warehouse shall support increasing data volumes. | Test selected queries and the ETL pipeline with larger datasets and compare performance. |
| Database schemas and transformation logic shall be documented and maintainable. | Review the database documentation, SQL scripts, and transformation logic. |

## 7. Open questions and risks

| Question / risk | Impact | Owner | Resolution / decision |
|---|---|---|---|
| How should API schema changes across game patches be handled? | May break data extraction and transformation scripts. | Student | Keep parsing logic modular and validate incoming JSON structures. |
| Will Riot Games API rate limits restrict large-scale data collection? | May increase data collection time and limit dataset size. | Student | Implement rate-limit handling and collect match data progressively. |
| How many matches can realistically be collected within the project timeframe? | Affects dataset size and analytical coverage. | Student | Estimate collection capacity through initial tests and adjust the collection scope if necessary. |
| How should missing or invalid source data be handled? | May reduce data quality and affect analytical results. | Student | Define validation rules and document how invalid records are handled. |
| Should a star schema or a snowflake schema be selected for the warehouse? | Affects database complexity and query performance. | Student | Compare both approaches and justify the final choice. |
| Which historization approach is most suitable for the project? | Affects how historical changes are stored and analyzed. | Student | Investigate at least two approaches and select one based on project requirements. |
| Will the selected database handle several million fact records efficiently? | May affect loading and analytical query performance. | Student | Test with an increasingly large dataset and optimize queries and indexes where necessary. |
| How should interrupted or repeated ETL runs be handled? | May cause incomplete loading or duplicate records. | Student | Implement processing status tracking, safe checkpoints, and duplicate-handling rules where needed. |
