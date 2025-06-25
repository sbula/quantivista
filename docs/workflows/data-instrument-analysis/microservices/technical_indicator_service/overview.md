# Technical Indicator Service

## Responsibility
High-performance real-time technical indicator computation with SIMD optimizations. Computes 50+ technical indicators across multiple timeframes with sub-50ms latency for trading-critical applications.

## Technology Stack
- **Language**: Rust + RustQuant + TA-Lib + SIMD optimizations
- **Libraries**: rayon (parallelism), nalgebra (linear algebra), serde (serialization)
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for custom optimization algorithms and advanced models
- **Scaling**: Horizontal by instrument groups, vertical for computation intensity
- **NFRs**: P99 computation latency < 50ms, throughput > 100K indicators/sec, 99.99% accuracy

## API Specification

### Core APIs
```pseudo
// Enumerations
enum IndicatorType {
    SMA,                // Simple Moving Average
    EMA,                // Exponential Moving Average
    RSI,                // Relative Strength Index
    MACD,               // MACD
    BOLLINGER_BANDS,    // Bollinger Bands
    STOCHASTIC,         // Stochastic Oscillator
    ATR,                // Average True Range
    ADX,                // Average Directional Index
    CCI,                // Commodity Channel Index
    WILLIAMS_R          // Williams %R
}

enum SignalType {
    BUY,
    SELL,
    NEUTRAL
}

// Data Models
struct IndicatorRequest {
    instrument_id: String
    timeframe: String  // "1m", "5m", "15m", "1h", "4h", "1d"
    indicators: List<IndicatorType>
    period: Optional<Integer>
    real_time: Boolean
}

struct IndicatorResponse {
    instrument_id: String
    timeframe: String
    timestamp: DateTime
    indicators: Map<String, IndicatorValue>
    computation_time_ms: Float
    data_points_used: Integer
}

struct IndicatorValue {
    value: Float
    confidence: Float
    signal: Optional<SignalType>
    metadata: Map<String, Float>
}

// REST API Endpoints
POST /api/v1/indicators/compute
    Request: IndicatorRequest
    Response: IndicatorResponse

GET /api/v1/indicators/{instrument_id}/latest
    Parameters: timeframe
    Response: IndicatorResponse

POST /api/v1/indicators/batch
    Request: List<IndicatorRequest>
    Response: List<IndicatorResponse>
```

### Event Output
```pseudo
Event technical_indicator_updated {
    event_id: String
    timestamp: DateTime
    indicator_update: IndicatorUpdateData
}

struct IndicatorUpdateData {
    instrument_id: String
    timeframe: String
    indicators: IndicatorsData
    computation_time_ms: Float
    data_points_used: Integer
}

struct IndicatorsData {
    sma_20: IndicatorValueData
    rsi_14: IndicatorValueData
    macd: IndicatorValueData
}

struct IndicatorValueData {
    value: Float
    confidence: Float
    signal: String
    metadata: JSON
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    indicator_update: {
        instrument_id: "AAPL",
        timeframe: "5m",
        indicators: {
            sma_20: {
                value: 150.25,
                confidence: 0.98,
                signal: "NEUTRAL",
                metadata: {trend: "sideways"}
            },
            rsi_14: {
                value: 65.4,
                confidence: 0.95,
                signal: "NEUTRAL",
                metadata: {overbought_threshold: 70}
            },
            macd: {
                value: 0.45,
                confidence: 0.92,
                signal: "BUY",
                metadata: {histogram: 0.12, signal_line: 0.33}
            }
        },
        computation_time_ms: 12.5,
        data_points_used: 200
    }
}
```

## Data Model & Database Schema

### PostgreSQL (Command Side)
```pseudo
Table indicator_configurations {
    id: UUID (primary key, auto-generated)
    instrument_id: String (required, max_length: 20)
    timeframe: String (required, max_length: 10)
    indicator_type: String (required, max_length: 50)
    parameters: JSON (required)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)

    // Constraints
    unique_instrument_timeframe_indicator: (instrument_id, timeframe, indicator_type)
}

Table computation_metrics {
    id: UUID (primary key, auto-generated)
    timestamp: Timestamp (required)
    instrument_group: String (max_length: 50)
    indicators_computed: Integer
    avg_computation_time_ms: Float
    throughput_per_second: Float
    error_count: Integer (default: 0)
    created_at: Timestamp (default: now)
}
```

### TimescaleDB (Query Side)
```pseudo
Table technical_indicators_ts {
    timestamp: Timestamp (required, partition_key)
    instrument_id: String (required, max_length: 20)
    timeframe: String (required, max_length: 10)
    indicator_type: String (required, max_length: 50)
    value: Float (required)
    confidence: Float
    signal: String (max_length: 10)
    metadata: JSON
    computation_time_ms: Float

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 hour)
    partition_dimension: instrument_id (partitions: 16)
}
```

### Redis Caching
```pseudo
Cache indicator_cache {
    // Latest indicators
    "indicators:{instrument_id}:{timeframe}": IndicatorResponse (TTL: 1m)

    // Sliding windows
    "window:{instrument_id}:{timeframe}": PriceWindow (TTL: 5m)

    // Computation cache
    "computed:{instrument_id}:{indicator_hash}": IndicatorValue (TTL: 30m)
}
```

## Implementation Estimation

### Priority: **CRITICAL** (Foundation for analysis)
### Estimated Time: **6-7 weeks**

#### Week 1-2: Core Indicator Engine
- Rust service setup with TA-Lib integration
- Basic indicator implementations (SMA, EMA, RSI, MACD)
- SIMD optimizations for parallel computation
- **Effort**: 2 senior Rust developers × 2 weeks = 4 dev-weeks

#### Week 3-4: Advanced Indicators
- Complex indicators (Bollinger Bands, Stochastic, ADX)
- Multi-timeframe support and synchronization
- Signal generation and confidence scoring
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 5: Performance Optimization
- Memory-efficient sliding windows
- Batch processing and parallel computation
- Cache optimization and invalidation
- **Effort**: 1 senior developer × 1 week = 1 dev-week

#### Week 6-7: Integration & Testing
- Integration with market data services
- Accuracy validation against reference implementations
- Performance testing (100K+ indicators/sec)
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **13 dev-weeks**
### Team Size: **2 senior Rust developers**
### Dependencies: Market data services, TimescaleDB, Redis

### Success Criteria:
- Compute 100K+ indicators per second
- P99 computation latency < 50ms
- 99.99% calculation accuracy
- Support for 50+ technical indicators
- Real-time streaming capability
