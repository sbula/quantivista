# Data Integration Service

## Responsibility
High-throughput data integration service that consumes normalized market data from the Market Data Acquisition workflow and distributes it to analysis services. Handles corporate actions, data validation, and real-time event processing with 99.9% data accuracy.

## Technology Stack
- **Language**: Go + Apache Pulsar + PostgreSQL + TimescaleDB
- **Libraries**: pulsar-client-go, pgx, gorilla/mux, prometheus/client_golang
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for mathematical optimization and advanced models
- **Scaling**: Horizontal by instrument groups, vertical for data throughput
- **NFRs**: 99.9% data accuracy, 95% processing within 500ms, 99.99% uptime

## API Specification

### Core APIs
```pseudo
// Enumerations
enum DataType {
    MARKET_DATA,
    CORPORATE_ACTIONS,
    FUNDAMENTAL_DATA,
    ALTERNATIVE_DATA,
    REFERENCE_DATA
}

enum ProcessingStatus {
    RECEIVED,
    VALIDATED,
    PROCESSED,
    DISTRIBUTED,
    FAILED
}

enum QualityLevel {
    HIGH,
    MEDIUM,
    LOW,
    REJECTED
}

// Data Models
struct DataIntegrationRequest {
    source: String
    data_type: DataType
    instruments: List<String>
    timeframe: String
    quality_threshold: QualityLevel
}

struct DataIntegrationResponse {
    request_id: String
    status: ProcessingStatus
    processed_count: Integer
    failed_count: Integer
    quality_score: Float
    processing_time_ms: Float
}

struct DataQualityReport {
    instrument_id: String
    timestamp: DateTime
    completeness: Float
    accuracy: Float
    freshness: Float
    consistency: Float
    overall_score: Float
    issues: List<QualityIssue>
}

struct QualityIssue {
    type: String
    severity: String
    description: String
    field: String
    value: String
}

// REST API Endpoints
POST /api/v1/integration/process
    Request: DataIntegrationRequest
    Response: DataIntegrationResponse

GET /api/v1/integration/status/{request_id}
    Response: DataIntegrationResponse

GET /api/v1/integration/quality/{instrument_id}
    Parameters: date_range
    Response: List<DataQualityReport>

GET /api/v1/integration/health
    Response: ServiceHealth
```

### Event Input/Output
```pseudo
// Input Events (from Market Data Acquisition)
Event market_data_normalized {
    event_id: String
    timestamp: DateTime
    market_data: NormalizedMarketData
}

struct NormalizedMarketData {
    instrument_id: String
    timestamp: DateTime
    price_data: PriceData
    volume_data: VolumeData
    metadata: DataMetadata
}

// Output Events (to Analysis Services)
Event analysis_data_ready {
    event_id: String
    timestamp: DateTime
    analysis_data: AnalysisReadyData
}

struct AnalysisReadyData {
    instrument_id: String
    timeframe: String
    price_data: PriceData
    volume_data: VolumeData
    corporate_actions: List<CorporateAction>
    quality_score: Float
    processing_metadata: ProcessingMetadata
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    analysis_data: {
        instrument_id: "AAPL",
        timeframe: "5m",
        price_data: {
            open: 150.25,
            high: 150.75,
            low: 149.80,
            close: 150.50,
            adjusted_close: 150.50
        },
        volume_data: {
            volume: 1250000,
            dollar_volume: 188125000.0,
            trade_count: 2500
        },
        corporate_actions: [],
        quality_score: 0.98,
        processing_metadata: {
            source: "market_data_acquisition",
            processing_time_ms: 125.5,
            validation_passed: true
        }
    }
}
```

## Data Model & Database Schema

### PostgreSQL (Command Side)
```pseudo
Table data_processing_jobs {
    id: UUID (primary key, auto-generated)
    source: String (required, max_length: 50)
    data_type: String (required, max_length: 50)
    status: String (required, max_length: 20)
    instruments_count: Integer
    processed_count: Integer (default: 0)
    failed_count: Integer (default: 0)
    quality_score: Float
    started_at: Timestamp
    completed_at: Timestamp
    error_message: Text
    created_at: Timestamp (default: now)
}

Table corporate_actions {
    id: UUID (primary key, auto-generated)
    instrument_id: String (required, max_length: 20)
    action_type: String (required, max_length: 50) // SPLIT, DIVIDEND, MERGER
    action_date: Date (required)
    ex_date: Date
    record_date: Date
    adjustment_factor: Float
    cash_amount: Float
    description: Text
    processed: Boolean (default: false)
    created_at: Timestamp (default: now)
    
    // Constraints
    unique_instrument_action_date: (instrument_id, action_type, action_date)
}

Table data_quality_metrics {
    id: UUID (primary key, auto-generated)
    instrument_id: String (required, max_length: 20)
    timestamp: Timestamp (required)
    completeness: Float
    accuracy: Float
    freshness: Float
    consistency: Float
    overall_score: Float
    issues_count: Integer (default: 0)
    created_at: Timestamp (default: now)
}
```

### TimescaleDB (Query Side)
```pseudo
Table processed_market_data_ts {
    timestamp: Timestamp (required, partition_key)
    instrument_id: String (required, max_length: 20)
    timeframe: String (required, max_length: 10)
    open_price: Float
    high_price: Float
    low_price: Float
    close_price: Float
    adjusted_close: Float
    volume: BigInteger
    dollar_volume: Float
    trade_count: Integer
    quality_score: Float
    processing_time_ms: Float
    
    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 hour)
    partition_dimension: instrument_id (partitions: 16)
}

Table data_processing_metrics_ts {
    timestamp: Timestamp (required, partition_key)
    source: String (required, max_length: 50)
    data_type: String (required, max_length: 50)
    records_processed: Integer
    processing_time_ms: Float
    error_count: Integer
    quality_score: Float
    throughput_per_second: Float
}
```

### Redis Caching
```pseudo
Cache integration_cache {
    // Processing status
    "job:{job_id}:status": ProcessingStatus (TTL: 1h)
    
    // Quality scores
    "quality:{instrument_id}:latest": QualityScore (TTL: 5m)
    
    // Corporate actions
    "corp_actions:{instrument_id}": List<CorporateAction> (TTL: 1d)
    
    // Processing metrics
    "metrics:processing:latest": ProcessingMetrics (TTL: 1m)
}
```

## Implementation Estimation

### Priority: **CRITICAL** (Data foundation)
### Estimated Time: **3-4 weeks**

#### Week 1: Core Integration Infrastructure
- Go service setup with Pulsar and database clients
- Basic event consumption and processing pipeline
- Data validation and quality checking framework
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 2: Corporate Actions & Data Enhancement
- Corporate action processing and adjustment logic
- Data enrichment and normalization
- Multi-asset data support
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 3: Real-Time Processing & Distribution
- Real-time data processing optimization
- Event distribution to analysis services
- Performance monitoring and optimization
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 4: Quality Assurance & Integration
- Advanced data quality framework
- Integration testing with analysis services
- Error handling and recovery mechanisms
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **8 dev-weeks**
### Team Size: **2 developers (1 senior Go developer + 1 developer)**
### Dependencies: Market Data Acquisition workflow, Apache Pulsar, TimescaleDB

### Success Criteria:
- 99.9% data accuracy and completeness
- 95% of data processed within 500ms
- 99.99% uptime during market hours
- Support for multiple asset classes
- Real-time corporate action processing
