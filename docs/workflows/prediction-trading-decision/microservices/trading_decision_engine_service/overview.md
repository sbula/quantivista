# Trading Decision Engine Service

## Responsibility
Core trading decision generation and logic that transforms validated trading signals into actionable buy/sell/hold decisions. Provides decision confidence scoring, market condition adaptation, and optimal decision timing using high-performance analytics.

## Technology Stack
- **Language**: Go + Polars + DuckDB for high-performance decision processing
- **Libraries**: Polars for DataFrame operations, DuckDB for complex decision analytics
- **Data Processing**: Polars for high-performance signal analysis and decision logic
- **Analytics**: DuckDB for complex market condition analysis and decision optimization
- **ML Framework**: JAX for mathematical optimization and advanced models
- **Scaling**: Horizontal by decision complexity, vertical for analytics-intensive operations
- **NFRs**: P99 decision latency < 100ms, throughput > 25K decisions/sec

## API Specification

### Internal APIs

#### Decision Generation API
```pseudo
// Enumerations
enum DecisionType {
    BUY,
    SELL,
    HOLD,
    STRONG_BUY,
    STRONG_SELL
}

enum MarketCondition {
    BULLISH,
    BEARISH,
    SIDEWAYS,
    VOLATILE,
    LOW_VOLATILITY
}

// Data Models
struct DecisionRequest {
    trading_signal: TradingSignal
    risk_validation: RiskValidationResult
    market_conditions: MarketConditionData
    decision_parameters: DecisionParameters
}

struct DecisionResponse {
    trading_decision: TradingDecision
    decision_metadata: DecisionMetadata
    processing_time_ms: Float
    quality_score: Float
}

struct TradingDecision {
    decision_id: String
    signal_id: String
    instrument_id: String
    decision_type: DecisionType
    confidence_score: Float
    decision_timestamp: DateTime
    expiry_timestamp: DateTime
    reasoning: String
}

// REST API Endpoints
POST /api/v1/decide
    Request: DecisionRequest
    Response: DecisionResponse

GET /api/v1/health
    Response: HealthResponse

GET /api/v1/metrics
    Response: MetricsResponse
```

### Event Output

#### TradingDecisionEvent
```pseudo
Event trading_decision_generated {
    event_id: String
    timestamp: DateTime
    trading_decision: TradingDecisionPayload
    decision_metadata: DecisionMetadataPayload
}

struct TradingDecisionPayload {
    decision_id: String
    signal_id: String
    instrument_id: String
    decision_type: String
    confidence_score: Float
    decision_timestamp: DateTime
    expiry_timestamp: DateTime
    reasoning: String
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-24T10:25:00.000Z",
    trading_decision: {
        decision_id: "dec_20250624_001",
        signal_id: "sig_20250624_001",
        instrument_id: "AAPL",
        decision_type: "BUY",
        confidence_score: 0.88,
        decision_timestamp: "2025-06-24T10:25:00.000Z",
        expiry_timestamp: "2025-06-24T16:25:00.000Z",
        reasoning: "Strong signal consensus with favorable market conditions"
    }
}
```

## Database Schema (CQRS Pattern)

### Command Side (PostgreSQL)
```pseudo
// Decision rules and configuration
Table decision_rules {
    id: UUID (primary key, auto-generated)
    rule_name: String (required, unique, max_length: 100)
    algorithm: String (required, max_length: 50)
    confidence_threshold: Float (default: 0.7)
    market_condition_weights: JSON
    risk_adjustment_factor: Float (default: 1.0)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
    updated_at: Timestamp (default: now)
}

// Decision thresholds
Table decision_thresholds {
    id: UUID (primary key, auto-generated)
    threshold_name: String (required, max_length: 100)
    decision_type: String (required, max_length: 20)
    min_confidence: Float (required)
    max_risk_score: Float (required)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

// Indexes
idx_decision_rules_algorithm: (algorithm, enabled)
idx_decision_thresholds_type: (decision_type, enabled)
```

### Query Side (TimescaleDB)
```pseudo
// Trading decisions storage
Table trading_decisions_ts {
    timestamp: Timestamp (required, partition_key)
    decision_id: String (required, max_length: 100)
    signal_id: String (required, max_length: 100)
    instrument_id: String (required, max_length: 20)
    decision_type: String (required, max_length: 20)
    confidence_score: Float (required)
    reasoning: Text
    processing_time_ms: Float
    quality_score: Float
    expiry_timestamp: Timestamp
    created_at: Timestamp (default: now)
    
    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
    partition_dimension: instrument_id (partitions: 16)
}

// Indexes for fast queries
idx_trading_decisions_instrument_time: (instrument_id, timestamp DESC)
idx_trading_decisions_type_time: (decision_type, timestamp DESC)
```

### Redis Caching Strategy
```pseudo
Cache decision_engine_cache {
    // Active decision rules
    "decision_rules:active": List<DecisionRule> (TTL: 30m)
    
    // Recent decisions
    "decisions:{instrument_id}:recent": List<TradingDecision> (TTL: 1h)
    
    // Performance metrics
    "performance:current": DecisionPerformanceMetrics (TTL: 1m)
}
```

## Implementation Estimation

### Priority: **CRITICAL** (Core decision logic)
### Estimated Time: **4-5 weeks**

#### Week 1-2: Core Decision Engine
- Go service setup with Polars and DuckDB integration
- Basic decision generation algorithms
- Market condition analysis framework
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3-4: Advanced Features & Integration
- Decision timing optimization
- Integration with Signal Synthesis and Risk Policy services
- Performance testing and optimization
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 5: Testing & Monitoring
- Decision quality validation and monitoring
- Error handling and recovery
- **Effort**: 1 developer × 1 week = 1 dev-week

### Total Effort: **9 dev-weeks**
### Team Size: **2 developers** (1 senior Go developer, 1 mid-level developer)
### Dependencies:
- Signal Synthesis Service operational
- Risk Policy Engine Service operational
- Market condition data feeds

### Success Criteria:
- Process 25K+ decisions per second
- P99 decision latency < 100ms
- 85% decision accuracy over 30-day periods
- Support for 10+ decision algorithms simultaneously
