> Artifact: medallion_env

## 2026-01-15
- **Decision**: Created a dedicated Spark Environment (`medallion_env`) for the project.
- **Reason**: Ensure dependency isolation, reproducibility, and consistent execution across notebooks and Spark jobs.
- **Limitation**: Environment-level changes require restarting Spark sessions.

## 2026-01-15
- **Decision**: Added `faker==40.1.2` as a pinned pip dependency.
- **Reason**: Enable deterministic generation of synthetic data for Bronze ingestion simulations.
- **Limitation**: Synthetic data does not fully represent real-world edge cases.

## 2026-01-15
- **Decision**: Restricted this environment to Bronze ingestion and data generation workloads.
- **Reason**: Keep Silver and Gold layers lean, stable, and focused on transformations.
- **Limitation**: Additional environments may be required if other layers need external dependencies.

> Artifact: dataset_generator (Bronze)

## 2026-01-15
- **Decision**: Modeled purchase events as nested JSON, allowing one customer to have multiple products per transaction and embedding detailed customer information in each purchase.
- **Reason**: Simulate real-world transactional APIs and event-based ingestion patterns commonly found in e-commerce and streaming sources.
- **Limitation**: Data redundancy at the Bronze layer requires normalization and flattening in the Silver layer.

## 2026-01-15
- **Decision**: Persisted the generated synthetic datasets under the Lakehouse `/Files` path for downstream Bronze ingestion.
- **Reason**: Decouple data generation from ingestion logic, enabling reusability, repeatability, and independent execution of the generator and Bronze pipelines.
- **Limitation**: File-based persistence introduces an intermediate storage step that would be replaced by direct source integration in production scenarios.

> Artifact: bronze_ingestion (Bronze)

## 2026-01-15
- **Decision**: Added technical metadata columns (e.g., ingestion timestamp, source identifier) during ingestion.
- **Reason**: Enable observability, lineage tracking, and auditability without altering business data.
- **Limitation**: Metadata alone is insufficient for full data quality monitoring.

## 2026-01-15
- **Decision**: Performed a structural normalization by exploding the `data` array so that each purchase event is stored as a single row.
- **Reason**: Improve queryability and downstream processing while preserving the original business content of the source payload.
- **Limitation**: Partial structural transformation in the Bronze layer may blur the boundary between raw ingestion and Silver modeling.

> Artifact: silver_transformation (Silver)

## 2026-01-15
- **Decision**: Defined a clear scope of transformations for the Silver layer, including normalization, cleaning, standardization, derived columns, and enrichment.
- **Reason**: Establish a consistent, analytics-ready dataset by separating structural and semantic transformations from raw ingestion concerns.
- **Limitation**: Transformation scope assumes stable upstream schemas and may require adjustments if source structures evolve.

> Artifact: gold_modeling (Gold)

## 2026-01-15
- **Decision**: Implemented a dimensional model (star schema) with fact and dimension tables.
- **Reason**: Optimize analytical queries, simplify aggregations, and align with BI consumption patterns.
- **Limitation**: Dimensional models require additional maintenance when business entities evolve.

## 2026-01-15
- **Decision**: Modeled `dim_date` using an integer surrogate key in `YYYYMMDD` format with no updates.
- **Reason**: Ensure deterministic joins, improve query performance, and avoid dependency on mutable date attributes.
- **Limitation**: Date attributes corrections require full regeneration rather than incremental updates.

## 2026-01-15
- **Decision**: Implemented dimensions as Slowly Changing Dimensions (SCD Type 2).
- **Reason**: Preserve historical context and enable point-in-time analytical queries.
- **Limitation**: Increases storage usage and complexity of dimension management.

## 2026-01-15
- **Decision**: Designed fact tables as append-only, avoiding updates or deletions.
- **Reason**: Maintain immutable analytical facts and simplify data lineage and auditing.
- **Limitation**: Corrections require compensating records rather than in-place updates.

## 2026-01-15
- **Decision**: Used hash-based comparison to detect changes in dimension attributes.
- **Reason**: Simplify change detection logic and reduce column-by-column comparisons.
- **Limitation**: Hash collisions are theoretically possible, though unlikely.

## 2026-01-15
- **Decision**: Avoided identity/auto-increment columns for surrogate keys.
- **Reason**: Lakehouse and distributed processing limitations make identity columns unreliable at scale.
- **Limitation**: Surrogate key generation must be handled explicitly in the transformation logic.

> Artifact: Lakehouse (Architecture)

## 2026-01-15
- **Decision**: Used a single Lakehouse for the entire Medallion Architecture (Bronze, Silver, Gold).
- **Reason**: Simplifies governance, reduces operational overhead, and aligns with Microsoft Fabric recommended practices for portfolio and analytics-focused projects.
- **Implementation**: Logical separation of layers achieved through dedicated schemas (`bronze`, `silver`, `gold`) instead of multiple Lakehouses.
- **Benefit**: Easier data discovery, consistent security model, and simplified cross-layer transformations.
- **Limitation**: Requires strict naming conventions and access discipline to avoid unintended cross-layer dependencies.

> Artifact: Data Pipeline (Orchestration)

## 2026-01-15
- **Decision**: Implemented the pipeline without automated triggers or scheduled execution.
- **Reason**: This project is intended solely for portfolio demonstration and does not rely on continuous or production workloads.
- **Decision**: Excluded email notifications tied to triggers or schedules.
- **Reason**: Without scheduled or event-based execution, automated notifications would not represent a realistic operational scenario.
- **Scope**: Pipeline execution is manual and intended for controlled, end-to-end demonstration of the Medallion flow.
- **Limitation**: Does not cover production-grade monitoring, alerting, or SLA enforcement.
