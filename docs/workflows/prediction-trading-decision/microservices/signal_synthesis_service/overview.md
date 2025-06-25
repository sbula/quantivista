# Signal Synthesis Service

## Responsibility
Transform instrument evaluations into trading signals through multi-timeframe signal aggregation, confidence scoring, and signal quality assessment. Provides high-performance signal processing using advanced data manipulation and ML optimization techniques.

## Technology Stack
- **Language**: Python + Polars + JAX for high-performance signal processing
- **Libraries**: Polars for DataFrame operations, JAX for ML models and mathematical optimization
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **ML Models**: JAX for neural networks and mathematical optimization
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **Scaling**: Horizontal by signal complexity, vertical for ML-intensive operations
- **NFRs**: P99 processing latency < 200ms, throughput > 10K signals/sec

## API Specification

### Internal APIs

#### Signal Synthesis API
```pseudo
// Enumerations
enum TimeframeType {
    MINUTE_1,
    MINUTE_5,
    MINUTE_15,
    HOUR_1,
    HOUR_4,
    DAY_1,
    WEEK_1,
    MONTH_1
}

enum SignalStrength {
    VERY_WEAK,
    WEAK,
    MODERATE,
    STRONG,
    VERY_STRONG
}

// Data Models
struct SynthesisRequest {
    instrument_evaluations: List<InstrumentEvaluation>
    timeframe_weights: Map<TimeframeType, Float>
    synthesis_method: String
    confidence_threshold: Float
}

struct InstrumentEvaluation {
    instrument_id: String
    timeframe: TimeframeType
    rating: String
    confidence: Float
    evaluation_timestamp: DateTime
    model_version: String
}

struct SynthesisResponse {
    trading_signal: TradingSignal
    synthesis_metadata: SynthesisMetadata
    processing_time_ms: Float
    quality_score: Float
}

struct TradingSignal {
    signal_id: String
    instrument_id: String
    direction: String
    strength: SignalStrength
    confidence: Float
    timeframe_consensus: TimeframeConsensus
    signal_timestamp: DateTime
}

struct SynthesisMetadata {
    timeframes_processed: List<TimeframeType>
    consensus_score: Float
    conflicting_signals: List<String>
    dominant_timeframe: TimeframeType
    synthesis_method_used: String
}
```

#### Health and Metrics API
```pseudo
// Health and Metrics Data Models
struct HealthResponse {
    status: ServiceStatus
    signal_queue_size: Integer
    memory_usage_mb: Float
    cpu_usage_percent: Float
    ml_model_status: String
}

struct MetricsResponse {
    signals_per_second: Float
    latency_percentiles: LatencyPercentiles
    synthesis_success_rate: Float
    confidence_distribution: ConfidenceDistribution
}

// REST API Endpoints
GET /api/v1/health
    Response: HealthResponse

GET /api/v1/metrics
    Response: MetricsResponse

POST /api/v1/synthesize
    Request: SynthesisRequest
    Response: SynthesisResponse
```

### Event Output

#### TradingSignalEvent
```pseudo
Event trading_signal_generated {
    event_id: String
    timestamp: DateTime
    trading_signal: TradingSignalPayload
    synthesis_metadata: SynthesisMetadataPayload
}

struct TradingSignalPayload {
    signal_id: String
    instrument_id: String
    direction: String
    strength: String
    confidence: Float
    timeframe_consensus: TimeframeConsensusData
    signal_timestamp: DateTime
    expiry_timestamp: DateTime
}

struct SynthesisMetadataPayload {
    timeframes_processed: List<String>
    consensus_score: Float
    dominant_timeframe: String
    synthesis_method: String
    quality_score: Float
    processing_time_ms: Float
}

struct TimeframeConsensusData {
    short_term_agreement: Float
    medium_term_agreement: Float
    long_term_agreement: Float
    overall_consensus: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-24T10:25:00.000Z",
    trading_signal: {
        signal_id: "sig_20250624_001",
        instrument_id: "AAPL",
        direction: "BUY",
        strength: "STRONG",
        confidence: 0.85,
        timeframe_consensus: {
            short_term_agreement: 0.90,
            medium_term_agreement: 0.80,
            long_term_agreement: 0.85,
            overall_consensus: 0.85
        },
        signal_timestamp: "2025-06-24T10:25:00.000Z",
        expiry_timestamp: "2025-06-24T14:25:00.000Z"
    },
    synthesis_metadata: {
        timeframes_processed: ["1h", "4h", "1d"],
        consensus_score: 0.85,
        dominant_timeframe: "4h",
        synthesis_method: "weighted_consensus",
        quality_score: 0.92,
        processing_time_ms: 125.5
    }
}
```

## Data Model

### Core Entities
```pseudo
// Enumerations
enum SynthesisMethod {
    WEIGHTED_CONSENSUS,
    MAJORITY_VOTE,
    DOMINANT_TIMEFRAME,
    ML_ENSEMBLE
}

// Data Models
struct TimeframeWeight {
    timeframe: TimeframeType
    weight: Float
    priority: Integer
    enabled: Boolean
}

struct SignalQualityMetrics {
    historical_accuracy: Float
    confidence_calibration: Float
    timeframe_consistency: Float
    signal_stability: Float
}

struct SynthesisRule {
    rule_id: String
    rule_name: String
    synthesis_method: SynthesisMethod
    timeframe_weights: List<TimeframeWeight>
    confidence_threshold: Float
    quality_threshold: Float
    enabled: Boolean
}
```

## Database Schema (CQRS Pattern)

### Command Side (PostgreSQL)
```pseudo
// Signal synthesis configuration
Table synthesis_rules {
    id: UUID (primary key, auto-generated)
    rule_name: String (required, unique, max_length: 100)
    synthesis_method: String (required, max_length: 50)
    timeframe_weights: JSON (required)
    confidence_threshold: Float (default: 0.7)
    quality_threshold: Float (default: 0.8)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
    updated_at: Timestamp (default: now)
}

// Timeframe configurations
Table timeframe_configurations {
    id: UUID (primary key, auto-generated)
    instrument_type: String (max_length: 50)
    timeframe: String (required, max_length: 10)
    weight: Float (required)
    priority: Integer (default: 1)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

// Signal quality tracking
Table signal_quality_metrics {
    id: UUID (primary key, auto-generated)
    signal_id: String (required, max_length: 100)
    quality_score: Float (required)
    accuracy_score: Float
    confidence_calibration: Float
    timeframe_consistency: Float
    created_at: Timestamp (default: now)
}

// Indexes
idx_synthesis_rules_method: (synthesis_method, enabled)
idx_timeframe_config_type: (instrument_type, enabled)
idx_signal_quality_id: (signal_id)
```

### Query Side (TimescaleDB)
```pseudo
// Trading signals storage
Table trading_signals_ts {
    timestamp: Timestamp (required, partition_key)
    signal_id: String (required, max_length: 100)
    instrument_id: String (required, max_length: 20)
    direction: String (required, max_length: 10)
    strength: String (required, max_length: 20)
    confidence: Float (required)
    
    // Consensus data
    timeframe_consensus: JSON
    synthesis_metadata: JSON
    
    // Quality metrics
    quality_score: Float
    processing_time_ms: Float
    
    // Expiry
    expiry_timestamp: Timestamp
    
    created_at: Timestamp (default: now)
    
    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
    partition_dimension: instrument_id (partitions: 16)
}

// Signal performance metrics
Table signal_performance_ts {
    timestamp: Timestamp (required, partition_key)
    signals_per_second: Float
    avg_processing_time_ms: Float
    p99_processing_time_ms: Float
    synthesis_success_rate: Float
    confidence_distribution: JSON
    memory_usage_mb: Float
    cpu_usage_percent: Float
    
    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 hour)
}

// Indexes for fast queries
idx_trading_signals_instrument_time: (instrument_id, timestamp DESC)
idx_trading_signals_direction_time: (direction, timestamp DESC)
idx_signal_performance_time: (timestamp DESC)
```

### Redis Caching Strategy
```pseudo
Cache signal_synthesis_cache {
    // Active signals
    "signal:{instrument_id}:latest": TradingSignal (TTL: 4h)
    
    // Synthesis rules
    "synthesis_rules:active": List<SynthesisRule> (TTL: 1h)
    
    // Timeframe weights
    "timeframe_weights:{instrument_type}": List<TimeframeWeight> (TTL: 1h)
    
    // Performance metrics
    "performance:current": SignalPerformanceMetrics (TTL: 5m)
    
    // Quality scores
    "quality:{signal_id}": Float (TTL: 24h)
}
```

## Implementation Estimation

### Priority: **HIGH** (Core signal processing component)
### Estimated Time: **4-5 weeks**

#### Week 1-2: Core Synthesis Engine
- Python service setup with Polars and JAX integration
- Multi-timeframe signal aggregation algorithms
- Basic confidence scoring and quality assessment
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3: Advanced Processing Features
- ML-based signal optimization using JAX
- Advanced consensus algorithms
- Signal quality filtering and validation
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 4: Performance Optimization
- High-throughput signal processing optimization
- Memory management and efficient data structures
- Batch processing for multiple instruments
- **Effort**: 1 senior developer × 1 week = 1 dev-week

#### Week 5: Integration & Testing
- Integration with Market Prediction workflow
- Performance testing (10K+ signals/sec target)
- Error handling and monitoring
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **9 dev-weeks**
### Team Size: **2 developers** (1 senior Python/ML developer, 1 mid-level developer)
### Dependencies:
- Market Prediction workflow operational
- TimescaleDB and PostgreSQL setup
- JAX and Polars libraries configured

### Risk Factors:
- **Medium**: High-throughput performance requirements
- **Low**: ML model complexity
- **Low**: Technology stack maturity

### Success Criteria:
- Process 10K+ signals per second
- P99 processing latency < 200ms
- 95% signal synthesis accuracy
- Support for 8+ timeframes simultaneously
- Zero signal loss during processing
