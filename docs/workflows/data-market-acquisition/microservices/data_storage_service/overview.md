# Data Storage Service

## Responsibility
High-performance time-series data storage service using TimescaleDB for efficient storage, indexing, and retrieval of normalized market data. Provides optimized query performance with 95% of queries under 1 second and 80% compression ratio for cost-effective storage.

## Technology Stack
- **Language**: Go + TimescaleDB + Redis + PostgreSQL
- **Libraries**: pgx, gorilla/mux, prometheus/client_golang, compress/gzip
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Scaling**: Horizontal with TimescaleDB clustering, vertical for query performance
- **NFRs**: 95% queries <1s, 80% compression ratio, 99.99% uptime, 100% data integrity

## API Specification

### Core APIs
```pseudo
// Enumerations
enum TimeFrame {
    ONE_MINUTE,
    FIVE_MINUTES,
    FIFTEEN_MINUTES,
    ONE_HOUR,
    ONE_DAY
}

enum DataType {
    OHLCV,
    TICK,
    QUOTE,
    TRADE
}

enum CompressionLevel {
    NONE,
    LOW,
    MEDIUM,
    HIGH
}

// Data Models
struct StorageRequest {
    instrument_id: String
    timeframe: TimeFrame
    data_type: DataType
    data_points: List<MarketDataPoint>
    compression: CompressionLevel
}

struct StorageResponse {
    request_id: String
    stored_count: Integer
    failed_count: Integer
    storage_time_ms: Float
    compression_ratio: Float
}

struct QueryRequest {
    instrument_id: String
    timeframe: TimeFrame
    start_time: DateTime
    end_time: DateTime
    fields: List<String>
    limit: Optional<Integer>
    offset: Optional<Integer>
}

struct QueryResponse {
    instrument_id: String
    timeframe: TimeFrame
    data_points: List<MarketDataPoint>
    total_count: Integer
    query_time_ms: Float
    cache_hit: Boolean
}

struct MarketDataPoint {
    timestamp: DateTime
    open: Float
    high: Float
    low: Float
    close: Float
    volume: BigInteger
    dollar_volume: Float
    trade_count: Integer
}

// REST API Endpoints
POST /api/v1/storage/store
    Request: StorageRequest
    Response: StorageResponse

GET /api/v1/storage/query
    Request: QueryRequest (as query parameters)
    Response: QueryResponse

GET /api/v1/storage/instruments/{instrument_id}/latest
    Parameters: timeframe, fields
    Response: MarketDataPoint

DELETE /api/v1/storage/instruments/{instrument_id}
    Parameters: start_time, end_time
    Response: DeletionResponse

GET /api/v1/storage/health
    Response: StorageHealth
```

### Event Input/Output
```pseudo
// Input Events (from Data Processing)
Event normalized_market_data {
    event_id: String
    timestamp: DateTime
    market_data: NormalizedMarketData
}

struct NormalizedMarketData {
    instrument_id: String
    timeframe: TimeFrame
    timestamp: DateTime
    ohlcv_data: OHLCVData
    quality_score: Float
    source_attribution: String
}

// Output Events (Storage Confirmation)
Event market_data_stored {
    event_id: String
    timestamp: DateTime
    storage_data: StorageEventData
}

struct StorageEventData {
    instrument_id: String
    timeframe: TimeFrame
    stored_count: Integer
    storage_time_ms: Float
    compression_ratio: Float
    storage_location: String
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    storage_data: {
        instrument_id: "AAPL",
        timeframe: "FIVE_MINUTES",
        stored_count: 1,
        storage_time_ms: 25.5,
        compression_ratio: 0.82,
        storage_location: "hypertable_market_data_2025_06"
    }
}
```

## Data Model & Database Schema

### TimescaleDB (Primary Storage)
```pseudo
Table market_data_ts {
    timestamp: Timestamp (required, partition_key)
    instrument_id: String (required, max_length: 20)
    timeframe: String (required, max_length: 10)
    open_price: Float (required)
    high_price: Float (required)
    low_price: Float (required)
    close_price: Float (required)
    volume: BigInteger (required)
    dollar_volume: Float
    trade_count: Integer
    quality_score: Float (default: 1.0)
    source: String (max_length: 50)
    created_at: Timestamp (default: now)
    
    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 hour)
    partition_dimension: instrument_id (partitions: 16)
    
    // Compression Policy
    compression_policy: {
        compress_after: 24 hours,
        compression_level: medium
    }
    
    // Retention Policy
    retention_policy: {
        raw_data: 2 years,
        compressed_data: 10 years
    }
}

Table storage_metadata_ts {
    timestamp: Timestamp (required, partition_key)
    instrument_id: String (required, max_length: 20)
    timeframe: String (required, max_length: 10)
    record_count: Integer
    storage_size_mb: Float
    compression_ratio: Float
    last_updated: Timestamp
    data_quality: Float
}
```

### PostgreSQL (Metadata)
```pseudo
Table storage_configurations {
    id: UUID (primary key, auto-generated)
    instrument_id: String (required, max_length: 20)
    timeframe: String (required, max_length: 10)
    compression_level: String (default: 'medium')
    retention_days: Integer (default: 730)
    indexing_strategy: String (default: 'time_instrument')
    created_at: Timestamp (default: now)
    updated_at: Timestamp (default: now)
    
    // Constraints
    unique_instrument_timeframe: (instrument_id, timeframe)
}

Table storage_statistics {
    id: UUID (primary key, auto-generated)
    date: Date (required)
    total_records: BigInteger
    total_size_gb: Float
    compression_ratio: Float
    query_count: Integer
    avg_query_time_ms: Float
    cache_hit_ratio: Float
    created_at: Timestamp (default: now)
}
```

### Redis (Query Cache)
```pseudo
Cache query_cache {
    // Query result caching
    "query:{hash}": QueryResult (TTL: 5m)
    
    // Latest data caching
    "latest:{instrument_id}:{timeframe}": MarketDataPoint (TTL: 1m)
    
    // Aggregated data caching
    "agg:{instrument_id}:{timeframe}:{period}": AggregatedData (TTL: 15m)
    
    // Storage statistics
    "stats:storage:latest": StorageStats (TTL: 1m)
}
```

## Implementation Estimation

### Priority: **CRITICAL** (Data foundation)
### Estimated Time: **3-4 weeks**

#### Week 1: Core Storage Infrastructure
- Go service setup with TimescaleDB and Redis clients
- Hypertable design and creation
- Basic storage operations (insert, query)
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 2: Query Optimization & Indexing
- Advanced indexing strategies
- Query optimization and performance tuning
- Compression implementation
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 3: Caching & Performance
- Redis query caching implementation
- Performance monitoring and metrics
- Backup and recovery mechanisms
- **Effort**: 1 senior developer × 1 week = 1 dev-week

#### Week 4: Integration & Monitoring
- Integration with data processing service
- Prometheus metrics and alerting
- Load testing and optimization
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **7 dev-weeks**
### Team Size: **2 developers (1 senior Go developer + 1 database specialist)**
### Dependencies: TimescaleDB cluster, Redis, Data Processing Service

### Success Criteria:
- 95% of queries completed within 1 second
- 80% compression ratio achieved
- 99.99% uptime during market hours
- 100% data integrity validation
- 1M+ records per second ingestion capacity
