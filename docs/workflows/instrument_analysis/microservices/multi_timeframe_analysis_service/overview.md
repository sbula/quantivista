# Multi-Timeframe Analysis Service

## Responsibility
Synchronized multi-timeframe technical analysis with trend alignment detection. Analyzes instruments across 1m, 5m, 15m, 1h, 4h, and daily timeframes to identify trend convergence and divergence patterns.

## Technology Stack
- **Language**: Python + NumPy + Pandas + asyncio
- **Libraries**: pandas, numpy, scipy, asyncio for concurrent processing
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for custom optimization algorithms and advanced models
- **Scaling**: Horizontal by instrument groups, parallel timeframe processing
- **NFRs**: P99 analysis latency < 2s for 6 timeframes, 95% trend alignment accuracy

## API Specification

### Core APIs
```pseudo
// Enumerations
enum Timeframe {
    M1,   // 1 minute
    M5,   // 5 minutes
    M15,  // 15 minutes
    H1,   // 1 hour
    H4,   // 4 hours
    D1    // 1 day
}

enum TrendDirection {
    BULLISH,
    BEARISH,
    SIDEWAYS,
    UNCERTAIN
}

enum SignalType {
    ENTRY,
    EXIT,
    REVERSAL
}

enum SignalDirection {
    BUY,
    SELL
}

// Data Models
struct MultiTimeframeRequest {
    instrument_id: String
    timeframes: List<Timeframe>
    analysis_depth: Integer  // Number of periods to analyze
    include_signals: Boolean
}

struct TimeframeTrend {
    timeframe: Timeframe
    trend_direction: TrendDirection
    trend_strength: Float  // 0.0 to 1.0
    key_levels: Map<String, Float>  // support, resistance
    indicators: Map<String, Float>
    confidence: Float
}

struct MultiTimeframeResponse {
    instrument_id: String
    timestamp: DateTime
    timeframe_analysis: Map<Timeframe, TimeframeTrend>
    trend_alignment: TrendAlignment
    trading_signals: List<TradingSignal>
    analysis_time_ms: Float
}

struct TrendAlignment {
    overall_direction: TrendDirection
    alignment_score: Float  // 0.0 to 1.0
    conflicting_timeframes: List<Timeframe>
    dominant_timeframes: List<Timeframe>
    confidence: Float
}

struct TradingSignal {
    signal_type: SignalType
    direction: SignalDirection
    strength: Float
    timeframes_supporting: List<Timeframe>
    price_target: Optional<Float>
    stop_loss: Optional<Float>
}

// REST API Endpoints
POST /api/v1/multi-timeframe/analyze
    Request: MultiTimeframeRequest
    Response: MultiTimeframeResponse

GET /api/v1/multi-timeframe/{instrument_id}/latest
    Response: MultiTimeframeResponse

POST /api/v1/multi-timeframe/batch-analyze
    Request: List<MultiTimeframeRequest>
    Response: List<MultiTimeframeResponse>
```

### Event Output
```pseudo
Event multi_timeframe_analysis_completed {
    event_id: String
    timestamp: DateTime
    multi_timeframe_analysis: MultiTimeframeAnalysisData
}

struct MultiTimeframeAnalysisData {
    instrument_id: String
    timeframe_analysis: Map<String, TimeframeTrendData>
    trend_alignment: TrendAlignmentData
    trading_signals: List<TradingSignalData>
}

struct TimeframeTrendData {
    trend_direction: String
    trend_strength: Float
    key_levels: Map<String, Float>
    confidence: Float
}

struct TrendAlignmentData {
    overall_direction: String
    alignment_score: Float
    conflicting_timeframes: List<String>
    dominant_timeframes: List<String>
    confidence: Float
}

struct TradingSignalData {
    signal_type: String
    direction: String
    strength: Float
    timeframes_supporting: List<String>
    price_target: Float
    stop_loss: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    multi_timeframe_analysis: {
        instrument_id: "AAPL",
        timeframe_analysis: {
            "1m": {
                trend_direction: "BULLISH",
                trend_strength: 0.65,
                key_levels: {"support": 149.80, "resistance": 150.50},
                confidence: 0.78
            },
            "1h": {
                trend_direction: "BULLISH",
                trend_strength: 0.82,
                key_levels: {"support": 149.00, "resistance": 151.00},
                confidence: 0.89
            },
            "1d": {
                trend_direction: "BULLISH",
                trend_strength: 0.91,
                key_levels: {"support": 145.00, "resistance": 155.00},
                confidence: 0.94
            }
        },
        trend_alignment: {
            overall_direction: "BULLISH",
            alignment_score: 0.85,
            conflicting_timeframes: [],
            dominant_timeframes: ["1h", "1d"],
            confidence: 0.87
        },
        trading_signals: [
            {
                signal_type: "ENTRY",
                direction: "BUY",
                strength: 0.78,
                timeframes_supporting: ["1m", "1h", "1d"],
                price_target: 152.00,
                stop_loss: 149.50
            }
        ]
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table timeframe_configurations {
    id: UUID (primary key, auto-generated)
    instrument_id: String (required, max_length: 20)
    timeframes: JSON (required)
    analysis_parameters: JSON (required)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table trend_alignment_rules {
    id: UUID (primary key, auto-generated)
    rule_name: String (required, max_length: 100)
    timeframe_weights: JSON (required)
    alignment_thresholds: JSON (required)
    signal_generation_rules: JSON (required)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}
```

### TimescaleDB (Query Side)
```pseudo
Table multi_timeframe_analysis_ts {
    timestamp: Timestamp (required, partition_key)
    instrument_id: String (required, max_length: 20)
    timeframe: String (required, max_length: 10)
    trend_direction: String (required, max_length: 20)
    trend_strength: Float (required)
    key_levels: JSON
    indicators: JSON
    confidence: Float
    analysis_metadata: JSON

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 hour)
    partition_dimension: instrument_id (partitions: 8)
}

Table trend_alignment_ts {
    timestamp: Timestamp (required, partition_key)
    instrument_id: String (required, max_length: 20)
    overall_direction: String (required, max_length: 20)
    alignment_score: Float (required)
    conflicting_timeframes: JSON
    dominant_timeframes: JSON
    confidence: Float
    trading_signals: JSON

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 hour)
}
```

### Redis Caching
```pseudo
Cache multi_timeframe_cache {
    // Latest analysis
    "mtf_analysis:{instrument_id}": MultiTimeframeResponse (TTL: 5m)

    // Timeframe trends
    "trend:{instrument_id}:{timeframe}": TimeframeTrend (TTL: 10m)

    // Alignment cache
    "alignment:{instrument_id}": TrendAlignment (TTL: 5m)

    // Signal cache
    "signals:{instrument_id}": List<TradingSignal> (TTL: 15m)
}
```

## Implementation Estimation

### Priority: **HIGH** (Critical for trend analysis)
### Estimated Time: **4-5 weeks**

#### Week 1-2: Core Multi-Timeframe Engine
- Python service setup with pandas/numpy
- Timeframe synchronization and data alignment
- Basic trend detection across timeframes
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3: Advanced Trend Analysis
- Trend alignment scoring and conflict detection
- Key level identification across timeframes
- Signal generation based on alignment
- **Effort**: 1 senior developer × 1 week = 1 dev-week

#### Week 4-5: Integration & Optimization
- Integration with technical indicator service
- Performance optimization for concurrent processing
- Accuracy validation and backtesting
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **9 dev-weeks**
### Team Size: **2 developers** (1 senior Python developer, 1 mid-level developer)
### Dependencies: Technical indicator service, market data, TimescaleDB

### Success Criteria:
- Analyze 6 timeframes in < 2 seconds
- 95% trend alignment accuracy
- Support for 1K+ instruments simultaneously
- Real-time trend convergence detection
- Accurate signal generation based on alignment
