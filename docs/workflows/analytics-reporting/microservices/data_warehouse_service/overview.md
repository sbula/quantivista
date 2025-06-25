# Data Warehouse Service

## Responsibility
Enterprise data warehouse management for historical data storage, data modeling, and analytical query processing. Provides optimized storage and retrieval for large-scale financial analytics with dimensional modeling and OLAP capabilities.

## Technology Stack
- **Language**: SQL + Python + Apache Spark + Snowflake/BigQuery
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **Libraries**: Apache Spark, dbt, SQLAlchemy, data modeling frameworks
- **Scaling**: Horizontal with cloud data warehouse scaling
- **NFRs**: P99 query response < 10s, petabyte-scale storage, 99.9% availability

## API Specification

### Core APIs
```pseudo
// Enumerations
enum DataModelType {
    FACT_TABLE,
    DIMENSION_TABLE,
    AGGREGATE_TABLE,
    MATERIALIZED_VIEW,
    DATA_MART
}

enum QueryType {
    ANALYTICAL_QUERY,
    REPORTING_QUERY,
    ETL_QUERY,
    DATA_EXPLORATION,
    PERFORMANCE_ANALYSIS
}

enum StorageFormat {
    PARQUET,
    DELTA_LAKE,
    ICEBERG,
    COLUMNAR,
    ROW_BASED
}

// Data Models
struct DataWarehouseQuery {
    query_id: String
    query_type: QueryType
    sql_query: String
    parameters: Map<String, Any>
    optimization_hints: List<String>
    cache_enabled: Boolean
}

struct QueryResult {
    query_id: String
    result_set: ResultSet
    execution_time_ms: Float
    rows_processed: Integer
    bytes_scanned: Integer
    cache_hit: Boolean
    query_plan: QueryPlan
}

struct DataModel {
    model_id: String
    model_name: String
    model_type: DataModelType
    schema_definition: SchemaDefinition
    partitioning_strategy: PartitioningStrategy
    indexing_strategy: IndexingStrategy
    retention_policy: RetentionPolicy
}

struct ETLJob {
    job_id: String
    job_name: String
    source_tables: List<String>
    target_table: String
    transformation_logic: String
    schedule: String
    status: JobStatus
}

struct DataLineage {
    entity_id: String
    entity_type: String
    upstream_dependencies: List<String>
    downstream_consumers: List<String>
    transformation_history: List<Transformation>
}

// REST API Endpoints
POST /api/v1/warehouse/query
    Request: DataWarehouseQuery
    Response: QueryResult

GET /api/v1/warehouse/models
    Response: List<DataModel>

POST /api/v1/warehouse/models
    Request: DataModel
    Response: DataModel

GET /api/v1/warehouse/lineage/{entity_id}
    Response: DataLineage

POST /api/v1/warehouse/etl/jobs
    Request: ETLJob
    Response: ETLJob

GET /api/v1/warehouse/performance/stats
    Parameters: start_date, end_date
    Response: PerformanceStats
```

### Event Output
```pseudo
Event warehouse_query_executed {
    event_id: String
    timestamp: DateTime
    warehouse_query_executed: WarehouseQueryData
}

struct WarehouseQueryData {
    query_id: String
    query_type: String
    execution_details: ExecutionDetailsData
    performance_metrics: PerformanceMetricsData
    optimization_applied: List<String>
}

struct ExecutionDetailsData {
    execution_time_ms: Integer
    rows_processed: Integer
    bytes_scanned: Integer
    cache_hit: Boolean
    partitions_scanned: Integer
}

struct PerformanceMetricsData {
    cpu_time_ms: Integer
    io_time_ms: Integer
    network_time_ms: Integer
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    warehouse_query_executed: {
        query_id: "query_20250621_001",
        query_type: "ANALYTICAL_QUERY",
        execution_details: {
            execution_time_ms: 5250,
            rows_processed: 2500000,
            bytes_scanned: 1073741824,
            cache_hit: false,
            partitions_scanned: 15
        },
        performance_metrics: {
            cpu_time_ms: 12500,
            io_time_ms: 2800,
            network_time_ms: 450
        },
        optimization_applied: [
            "partition_pruning",
            "predicate_pushdown",
            "column_pruning"
        ]
    }
}
```

## Database Schema

### Data Warehouse Schema (Dimensional Model)
```pseudo
// Fact Tables
Table fact_portfolio_performance {
    date_key: Integer
    portfolio_key: Integer
    instrument_key: Integer
    performance_metrics: PerformanceMetricsStruct
    risk_metrics: RiskMetricsStruct
    attribution_metrics: AttributionMetricsStruct

    // Partitioning
    partition_by: date_key
}

struct PerformanceMetricsStruct {
    daily_return: Float
    cumulative_return: Float
    volatility: Float
    sharpe_ratio: Float
    max_drawdown: Float
}

struct RiskMetricsStruct {
    var_95: Float
    var_99: Float
    expected_shortfall: Float
    beta: Float
}

struct AttributionMetricsStruct {
    allocation_effect: Float
    selection_effect: Float
    interaction_effect: Float
}

CREATE TABLE fact_trade_execution (
    execution_date_key INTEGER,
    trade_key INTEGER,
    order_key INTEGER,
    portfolio_key INTEGER,
    instrument_key INTEGER,
    venue_key INTEGER,
    execution_metrics STRUCT<
        quantity FLOAT,
        price FLOAT,
        commission FLOAT,
        market_impact FLOAT,
        slippage FLOAT
    >,
    timing_metrics STRUCT<
        order_time TIMESTAMP,
        execution_time TIMESTAMP,
        settlement_time TIMESTAMP,
        execution_duration_ms INTEGER
    >
) PARTITIONED BY (execution_date_key);

-- Dimension Tables
CREATE TABLE dim_portfolio (
    portfolio_key INTEGER PRIMARY KEY,
    portfolio_id VARCHAR(100),
    portfolio_name VARCHAR(200),
    portfolio_type VARCHAR(50),
    inception_date DATE,
    base_currency VARCHAR(3),
    benchmark VARCHAR(100),
    strategy_type VARCHAR(50)
);

CREATE TABLE dim_instrument (
    instrument_key INTEGER PRIMARY KEY,
    instrument_id VARCHAR(20),
    instrument_name VARCHAR(200),
    instrument_type VARCHAR(50),
    sector VARCHAR(100),
    industry VARCHAR(100),
    country VARCHAR(50),
    currency VARCHAR(3),
    market_cap_category VARCHAR(20)
);

CREATE TABLE dim_date (
    date_key INTEGER PRIMARY KEY,
    full_date DATE,
    year INTEGER,
    quarter INTEGER,
    month INTEGER,
    week INTEGER,
    day_of_week INTEGER,
    is_business_day BOOLEAN,
    is_month_end BOOLEAN,
    is_quarter_end BOOLEAN,
    is_year_end BOOLEAN
);
```

### Metadata Management
```pseudo
Table data_models {
    id: UUID (primary key, auto-generated)
    model_id: String (required, unique, max_length: 100)
    model_name: String (required, max_length: 200)
    model_type: String (required, max_length: 50)
    schema_definition: JSON (required)
    partitioning_strategy: JSON
    created_at: Timestamp (default: now)
}

Table etl_jobs {
    id: UUID (primary key, auto-generated)
    job_id: String (required, unique, max_length: 100)
    job_name: String (required, max_length: 200)
    source_tables: JSON (required)
    target_table: String (required, max_length: 200)
    transformation_logic: String (required)
    schedule: String (max_length: 100)
    status: String (required, max_length: 20)
    last_run_time: Timestamp
    created_at: Timestamp (default: now)
}
```

## Implementation Estimation

### Priority: **HIGH** (Foundation for analytics)
### Estimated Time: **6-8 weeks**

#### Week 1-2: Data Warehouse Setup
- Cloud data warehouse configuration
- Basic dimensional modeling
- ETL pipeline framework
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3-4: Data Modeling
- Fact and dimension table design
- Data lineage tracking
- Performance optimization
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 5-6: Advanced Features
- OLAP cube creation
- Advanced analytics support
- Query optimization and caching
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 7-8: Integration & Testing
- Integration with all data sources
- Performance testing with large datasets
- Data quality validation
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **16 dev-weeks**
### Team Size: **2 developers** (1 senior data engineer, 1 data warehouse specialist)
### Dependencies: Data ingestion service, cloud infrastructure

### Success Criteria:
- P99 query response < 10 seconds
- Petabyte-scale storage capability
- 99.9% availability and reliability
- Comprehensive data lineage tracking
- Optimized dimensional modeling
- Support for complex analytical queries
