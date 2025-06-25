# Data Ingestion Service

## Responsibility
Comprehensive data ingestion from all QuantiVista workflows for reporting and analytics. Aggregates data from portfolio management, trade execution, market intelligence, and other sources into a unified analytics data lake.

## Technology Stack
- **Language**: Python + Apache Airflow + Apache Kafka + Polars
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **Libraries**: Apache Airflow, Kafka, Polars, SQLAlchemy, data validation libraries
- **Scaling**: Horizontal by data source and volume
- **NFRs**: P99 ingestion latency < 5s, 99.9% data integrity, real-time streaming support

## API Specification

### Core APIs
```pseudo
// Enumerations
enum DataSourceType {
    PORTFOLIO_MANAGEMENT,
    TRADE_EXECUTION,
    MARKET_INTELLIGENCE,
    INSTRUMENT_ANALYSIS,
    MARKET_DATA,
    EXTERNAL_FEEDS
}

enum IngestionMode {
    REAL_TIME_STREAM,
    BATCH_SCHEDULED,
    ON_DEMAND,
    INCREMENTAL_SYNC
}

enum DataFormat {
    JSON,
    AVRO,
    PARQUET,
    CSV,
    PROTOBUF
}

// Data Models
struct DataIngestionRequest {
    source_id: String
    source_type: DataSourceType
    ingestion_mode: IngestionMode
    data_format: DataFormat
    schedule: Optional<String>
    transformation_rules: List<TransformationRule>
    quality_checks: List<QualityCheck>
}

struct IngestionJob {
    job_id: String
    source_id: String
    status: JobStatus
    records_processed: Integer
    records_failed: Integer
    start_time: DateTime
    end_time: Optional<DateTime>
    error_details: Optional<String>
}

struct DataQualityReport {
    job_id: String
    total_records: Integer
    valid_records: Integer
    invalid_records: Integer
    quality_score: Float
    validation_errors: List<ValidationError>
    data_profiling: DataProfile
}

struct TransformationRule {
    rule_id: String
    rule_type: String
    source_field: String
    target_field: String
    transformation_logic: String
    validation_criteria: Optional<String>
}

// REST API Endpoints
POST /api/v1/ingestion/jobs
    Request: DataIngestionRequest
    Response: IngestionJob

GET /api/v1/ingestion/jobs/{job_id}/status
    Response: IngestionJob

GET /api/v1/ingestion/jobs/{job_id}/quality
    Response: DataQualityReport

POST /api/v1/ingestion/sources/{source_id}/sync
    Response: IngestionJob

GET /api/v1/ingestion/sources
    Response: List<DataSource>
```

### Event Output
```pseudo
struct DataIngestedEvent {
    event_id: String  // "uuid"
    timestamp: DateTime  // "2025-06-21T10:00:00.000Z"
    data_ingested: {
        job_id: String  // "ingestion_20250621_001"
        source_id: String  // "portfolio_management_service"
        source_type: String  // "PORTFOLIO_MANAGEMENT"
        ingestion_mode: String  // "REAL_TIME_STREAM"
        records_processed: Integer  // 1250
        records_failed: Integer  // 3
        quality_score: Float  // 0.998
        processing_time_ms: Integer  // 2450
        data_categories: [
            String  // "portfolio_positions"
            String  // "performance_metrics"
            String  // "risk_metrics"
        ]
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table data_sources {
    id: UUID (primary key, default: gen_random_uuid())
    source_id: String (required, unique, max_length: 100)
    source_name: String (required, max_length: 200)
    source_type: String (required, max_length: 50)
    connection_config: JSON (required)
    ingestion_config: JSON (required)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: NOW())
}

Table ingestion_jobs {
    id: UUID (primary key, default: gen_random_uuid())
    job_id: String (required, unique, max_length: 100)
    source_id: String (required, max_length: 100)
    status: String (required, max_length: 20)
    records_processed: BigInt (default: 0)
    records_failed: BigInt (default: 0)
    start_time: Timestamp (required)
    end_time: Timestamp
    error_details: Text
    quality_report: JSON
    created_at: Timestamp (default: NOW())
}

Table transformation_rules {
    id: UUID (primary key, default: gen_random_uuid())
    rule_id: String (required, unique, max_length: 100)
    source_id: String (required, max_length: 100)
    rule_type: String (required, max_length: 50)
    transformation_config: JSON (required)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: NOW())
}
```

### Data Lake Storage (Parquet/Delta Lake)
```pseudo
// Portfolio data partitioned by date and portfolio_id
Table portfolio_data_lake {
    ingestion_date: Date
    portfolio_id: String (max_length: 100)
    data_type: String (max_length: 50)
    data_payload: JSON
    ingestion_job_id: String (max_length: 100)
    quality_score: Float

    // Partitioning
    partitioned_by: (ingestion_date, portfolio_id)
}

// Trade execution data partitioned by date
Table execution_data_lake {
    execution_date: Date
    trade_id: String (max_length: 100)
    order_data: JSON
    execution_data: JSON
    settlement_data: JSON
    ingestion_job_id: String (max_length: 100)

    // Partitioning
    partitioned_by: (execution_date)
}
```

## Implementation Estimation

### Priority: **HIGH** (Foundation for all reporting)
### Estimated Time: **4-5 weeks**

#### Week 1-2: Core Ingestion Engine
- Python service setup with Airflow
- Basic data source connectivity
- Real-time streaming with Kafka
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3: Data Quality & Transformation
- Data validation and quality checks
- Transformation rule engine
- Error handling and retry logic
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 4-5: Integration & Optimization
- Integration with all QuantiVista workflows
- Performance optimization and monitoring
- Data lineage and audit trails
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **10 dev-weeks**
### Team Size: **2 developers** (1 senior data engineer, 1 mid-level developer)
### Dependencies: All QuantiVista workflows, data lake infrastructure

### Success Criteria:
- P99 ingestion latency < 5 seconds
- 99.9% data integrity and quality
- Support for real-time and batch ingestion
- Comprehensive data validation
- Scalable to 10M+ records per day
