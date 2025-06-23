# Data Processing Service

## Responsibility
High-performance normalization, standardization, and enrichment of validated market data. Converts provider-specific formats into standardized schemas, performs currency conversions, timezone adjustments, and data enrichment for downstream consumption.

## Technology Stack
- **Language**: Rust + Polars + Apache Arrow for high-performance data processing
- **Libraries**: Polars for DataFrame operations, Arrow for zero-copy data transfer
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **Scaling**: Horizontal by data volume, vertical for CPU-intensive operations
- **NFRs**: P99 processing latency < 75ms, throughput > 1M events/sec

## API Specification

### Internal APIs

#### Data Processing API
```pseudo
// Enumerations
enum EnrichmentLevel {
    BASIC,      // Just normalization
    STANDARD,   // + derived fields
    FULL        // + all enrichments
}

// Data Models
struct ProcessingRequest {
    raw_data: RawMarketData
    processing_options: ProcessingOptions
    enrichment_level: EnrichmentLevel
}

struct ProcessingOptions {
    normalize_currency: Boolean
    adjust_timezone: Boolean
    apply_corporate_actions: Boolean
    calculate_derived_fields: Boolean
}

struct ProcessingResponse {
    normalized_data: NormalizedMarketData
    processing_metadata: ProcessingMetadata
    enrichments_applied: List<String>
    processing_time_ms: Float
}

struct ProcessingStats {
    events_processed: Integer
    processing_rate: Float
    avg_latency_ms: Float
    error_rate: Float
    enrichment_success_rate: Float
}
```

#### Health and Metrics API
```pseudo
// Health and Metrics Data Models
struct HealthResponse {
    status: ServiceStatus
    processing_queue_size: Integer
    memory_usage_mb: Float
    cpu_usage_percent: Float
}

struct MetricsResponse {
    throughput_per_second: Float
    latency_percentiles: LatencyPercentiles
    error_rates: ErrorRates
    enrichment_stats: EnrichmentStats
}

// REST API Endpoints
GET /api/v1/health
    Response: HealthResponse

GET /api/v1/metrics
    Response: MetricsResponse
```

### Event Output

#### NormalizedMarketDataEvent
```pseudo
Event normalized_market_data_processed {
    event_id: String
    timestamp: DateTime
    normalized_data: NormalizedMarketDataPayload
    processing_metadata: ProcessingMetadataData
    derived_fields: DerivedFieldsData
}

struct NormalizedMarketDataPayload {
    symbol: String
    exchange: String
    currency: String
    timestamp: DateTime
    price: Float
    volume: Integer
    bid: Float
    ask: Float
    spread: Float
    mid_price: Float
    vwap: Float
    market_cap: Integer
    data_type: String
}

struct ProcessingMetadataData {
    original_provider: String
    processing_time_ms: Integer
    enrichments_applied: List<String>
    quality_score: Float
    normalization_version: String
}

struct DerivedFieldsData {
    price_change: Float
    price_change_percent: Float
    volume_weighted_price: Float
    time_since_last_trade_ms: Integer
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T09:30:00.200Z",
    normalized_data: {
        symbol: "AAPL",
        exchange: "NASDAQ",
        currency: "USD",
        timestamp: "2025-06-21T09:30:00.120Z",
        price: 150.25,
        volume: 1000,
        bid: 150.24,
        ask: 150.26,
        spread: 0.02,
        mid_price: 150.25,
        vwap: 150.23,
        market_cap: 2450000000000,
        data_type: "trade"
    },
    processing_metadata: {
        original_provider: "bloomberg",
        processing_time_ms: 12,
        enrichments_applied: ["currency_conversion", "derived_fields", "market_cap"],
        quality_score: 0.95,
        normalization_version: "v2.1"
    },
    derived_fields: {
        price_change: 0.15,
        price_change_percent: 0.10,
        volume_weighted_price: 150.23,
        time_since_last_trade_ms: 50
    }
}
```

## Data Model

### Core Entities
```pseudo
// Enumerations
enum DataType {
    TRADE,
    QUOTE,
    LEVEL2,
    OHLC,
    VOLUME
}

enum EnrichmentType {
    CURRENCY_CONVERSION,
    DERIVED_FIELDS,
    MARKET_CAP_CALCULATION,
    VOLUME_WEIGHTED_PRICE,
    TECHNICAL_INDICATORS
}

// Data Models
struct NormalizedMarketData {
    symbol: String
    exchange: String
    currency: String
    timestamp: DateTime
    data_type: DataType

    // Price data
    price: Optional<Float>
    volume: Optional<Integer>
    bid: Optional<Float>
    ask: Optional<Float>

    // Derived fields
    spread: Optional<Float>
    mid_price: Optional<Float>
    vwap: Optional<Float>
    market_cap: Optional<Float>

    // Metadata
    processing_metadata: ProcessingMetadata
    derived_fields: Map<String, Float>
}

struct ProcessingMetadata {
    original_provider: String
    processing_time_ms: Float
    enrichments_applied: List<String>
    quality_score: Float
    normalization_version: String
}

struct EnrichmentRule {
    rule_id: String
    rule_type: EnrichmentType
    conditions: List<Condition>
    transformations: List<Transformation>
    priority: Integer
}
```

## Database Schema (CQRS Pattern)

### Command Side (PostgreSQL)
```pseudo
// Normalization rules and configuration
Table normalization_rules {
    id: UUID (primary key, auto-generated)
    rule_name: String (required, unique, max_length: 100)
    rule_type: String (required, max_length: 50) // 'currency', 'timezone', 'format', 'enrichment'
    source_provider: String (max_length: 50)
    target_format: String (max_length: 50)
    transformation_config: JSON (required)
    enabled: Boolean (default: true)
    priority: Integer (default: 1)
    created_at: Timestamp (default: now)
    updated_at: Timestamp (default: now)
}

// Currency conversion rates
Table currency_rates {
    id: UUID (primary key, auto-generated)
    from_currency: String (required, max_length: 3)
    to_currency: String (required, max_length: 3)
    rate: Decimal (required, precision: 15, scale: 8)
    effective_date: Date (required)
    source: String (required, max_length: 50)
    created_at: Timestamp (default: now)

    // Constraints
    unique_currency_pair_date: (from_currency, to_currency, effective_date)
}

// Enrichment rules
Table enrichment_rules {
    id: UUID (primary key, auto-generated)
    rule_id: String (required, unique, max_length: 100)
    rule_type: String (required, max_length: 50)
    conditions: JSON
    transformations: JSON (required)
    priority: Integer (default: 1)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

// Processing statistics (command side)
Table processing_stats {
    id: UUID (primary key, auto-generated)
    timestamp: Timestamp (required)
    events_processed: Integer (default: 0)
    processing_time_total_ms: Integer (default: 0)
    errors_count: Integer (default: 0)
    enrichments_applied: JSON
    created_at: Timestamp (default: now)
}

// Indexes
idx_normalization_rules_type: (rule_type, enabled)
idx_currency_rates_pair_date: (from_currency, to_currency, effective_date DESC)
idx_enrichment_rules_type_priority: (rule_type, priority)
idx_processing_stats_timestamp: (timestamp DESC)
```

### Query Side (TimescaleDB)
```pseudo
// Normalized market data storage
Table normalized_market_data {
    timestamp: Timestamp (required, partition_key)
    symbol: String (required, max_length: 20)
    exchange: String (required, max_length: 20)
    currency: String (required, max_length: 3)
    data_type: String (required, max_length: 20)

    // Price data
    price: Decimal (precision: 15, scale: 6)
    volume: Integer
    bid: Decimal (precision: 15, scale: 6)
    ask: Decimal (precision: 15, scale: 6)

    // Derived fields
    spread: Decimal (precision: 15, scale: 6)
    mid_price: Decimal (precision: 15, scale: 6)
    vwap: Decimal (precision: 15, scale: 6)
    market_cap: Integer

    // Metadata
    original_provider: String (max_length: 50)
    processing_time_ms: Float
    enrichments_applied: JSON
    quality_score: Float
    normalization_version: String (max_length: 10)

    // Additional derived fields
    derived_fields: JSON

    created_at: Timestamp (default: now)

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 hour)
    partition_dimension: symbol (partitions: 32)
}

// Processing performance metrics
Table processing_metrics_ts {
    timestamp: Timestamp (required, partition_key)
    throughput_per_second: Float
    avg_latency_ms: Float
    p99_latency_ms: Float
    error_rate: Float
    enrichment_success_rate: Float
    memory_usage_mb: Float
    cpu_usage_percent: Float

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 hour)
}

// Indexes for fast queries
idx_normalized_data_symbol_time: (symbol, timestamp DESC)
idx_normalized_data_exchange_time: (exchange, timestamp DESC)
idx_processing_metrics_time: (timestamp DESC)
```

### Redis Caching Strategy
```pseudo
Cache processing_cache {
    // Currency rates
    "currency_rate:{from}:{to}": Float (TTL: 1h)

    // Enrichment cache
    "enrichment:{symbol}:{rule_type}": EnrichedData (TTL: 30m)

    // Processing stats
    "processing_stats:current": ProcessingStats (TTL: 5m)

    // Normalization rules
    "norm_rules:{provider}": List<NormalizationRule> (TTL: 1h)
}
```

## Implementation Estimation

### Priority: **HIGH** (Core data pipeline component)
### Estimated Time: **4-5 weeks**

#### Week 1-2: Core Processing Engine
- Rust service setup with Polars and Arrow integration
- Basic normalization engine for common data formats
- Currency conversion and timezone adjustment
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3: Advanced Processing Features
- Derived field calculations (spread, mid-price, VWAP)
- Enrichment rule engine implementation
- Market cap and fundamental data enrichment
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 4: Performance Optimization
- High-throughput processing optimization
- Memory management and zero-copy operations
- Batch processing for efficiency
- **Effort**: 1 senior developer × 1 week = 1 dev-week

#### Week 5: Integration & Testing
- Integration with Data Quality Service
- Performance testing (1M+ events/sec target)
- Error handling and monitoring
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **9 dev-weeks**
### Team Size: **2 developers** (1 senior Rust developer, 1 mid-level developer)
### Dependencies:
- Data Quality Service operational
- TimescaleDB and PostgreSQL setup
- Currency rate data feeds

### Risk Factors:
- **Medium**: High-throughput performance requirements
- **Low**: Data format complexity
- **Low**: Technology stack maturity

### Success Criteria:
- Process 1M+ events per second
- P99 processing latency < 75ms
- 99.9% data normalization accuracy
- Support for 10+ different provider formats
- Zero data loss during processing
