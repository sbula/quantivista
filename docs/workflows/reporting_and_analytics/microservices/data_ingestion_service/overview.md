# Data Ingestion Service

## Responsibility
Comprehensive data ingestion from all QuantiVista workflows for reporting and analytics. Aggregates data from portfolio management, trade execution, market intelligence, and other sources into a unified analytics data lake.

## Technology Stack
- **Language**: Python + Apache Airflow + Apache Kafka + Pandas
- **Libraries**: Apache Airflow, Kafka, pandas, SQLAlchemy, data validation libraries
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
```json
{
  "eventId": "uuid",
  "timestamp": "2025-06-21T10:00:00.000Z",
  "data_ingested": {
    "job_id": "ingestion_20250621_001",
    "source_id": "portfolio_management_service",
    "source_type": "PORTFOLIO_MANAGEMENT",
    "ingestion_mode": "REAL_TIME_STREAM",
    "records_processed": 1250,
    "records_failed": 3,
    "quality_score": 0.998,
    "processing_time_ms": 2450,
    "data_categories": [
      "portfolio_positions",
      "performance_metrics",
      "risk_metrics"
    ]
  }
}
```

## Database Schema

### PostgreSQL (Command Side)
```sql
CREATE TABLE data_sources (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    source_id VARCHAR(100) NOT NULL UNIQUE,
    source_name VARCHAR(200) NOT NULL,
    source_type VARCHAR(50) NOT NULL,
    connection_config JSONB NOT NULL,
    ingestion_config JSONB NOT NULL,
    enabled BOOLEAN DEFAULT true,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE ingestion_jobs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    job_id VARCHAR(100) NOT NULL UNIQUE,
    source_id VARCHAR(100) NOT NULL,
    status VARCHAR(20) NOT NULL,
    records_processed BIGINT DEFAULT 0,
    records_failed BIGINT DEFAULT 0,
    start_time TIMESTAMPTZ NOT NULL,
    end_time TIMESTAMPTZ,
    error_details TEXT,
    quality_report JSONB,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE transformation_rules (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    rule_id VARCHAR(100) NOT NULL UNIQUE,
    source_id VARCHAR(100) NOT NULL,
    rule_type VARCHAR(50) NOT NULL,
    transformation_config JSONB NOT NULL,
    enabled BOOLEAN DEFAULT true,
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### Data Lake Storage (Parquet/Delta Lake)
```sql
-- Portfolio data partitioned by date and portfolio_id
CREATE TABLE portfolio_data_lake (
    ingestion_date DATE,
    portfolio_id VARCHAR(100),
    data_type VARCHAR(50),
    data_payload JSONB,
    ingestion_job_id VARCHAR(100),
    quality_score FLOAT
) PARTITIONED BY (ingestion_date, portfolio_id);

-- Trade execution data partitioned by date
CREATE TABLE execution_data_lake (
    execution_date DATE,
    trade_id VARCHAR(100),
    order_data JSONB,
    execution_data JSONB,
    settlement_data JSONB,
    ingestion_job_id VARCHAR(100)
) PARTITIONED BY (execution_date);
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
