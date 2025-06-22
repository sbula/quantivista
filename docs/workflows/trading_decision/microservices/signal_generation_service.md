# Signal Generation Service

## Responsibility
Convert instrument evaluations to pure trading signals without portfolio considerations. Transforms ML predictions and technical analysis into standardized trading signals with confidence scoring and reasoning.

## Technology Stack
- **Language**: Python + FastAPI + NumPy + asyncio
- **Libraries**: NumPy, pandas, scipy, asyncio for concurrent processing
- **Scaling**: Horizontal by instrument groups
- **NFRs**: P99 signal generation < 100ms, 99.9% signal consistency, high throughput

## API Specification

### Core APIs
```pseudo
// Enumerations
enum SignalDirection {
    BUY,
    SELL,
    HOLD
}

enum SignalStrength {
    WEAK,
    MODERATE,
    STRONG
}

// Data Models
struct SignalGenerationRequest {
    instrument_id: String
    evaluation: InstrumentEvaluation  // From Market Prediction workflow
    market_conditions: MarketConditions
    signal_parameters: Optional<Map<String, Any>>
}

struct TradingSignal {
    signal_id: String
    instrument_id: String
    direction: SignalDirection
    strength: SignalStrength
    confidence: Float  // 0.0 to 1.0
    timeframe: String
    entry_price: Optional<Float>
    target_price: Optional<Float>
    stop_loss: Optional<Float>
    reasoning: List<String>
    technical_factors: Map<String, Float>
    sentiment_factors: Map<String, Float>
    risk_factors: Map<String, Float>
    timestamp: DateTime
}

struct SignalGenerationResponse {
    signals: List<TradingSignal>
    generation_metadata: GenerationMetadata
    processing_time_ms: Float
}

struct GenerationMetadata {
    evaluation_quality: Float
    signal_consistency: Float
    market_regime: String
    volatility_adjustment: Float
}

// REST API Endpoints
POST /api/v1/generate
    Request: SignalGenerationRequest
    Response: SignalGenerationResponse

POST /api/v1/batch-generate
    Request: List<SignalGenerationRequest>
    Response: List<SignalGenerationResponse>

GET /api/v1/signals/{instrument_id}/latest
    Response: List<TradingSignal>
```

### Event Output
```pseudo
Event signal_generated {
    event_id: String
    timestamp: DateTime
    signal_generated: TradingSignalData
}

struct TradingSignalData {
    signal_id: String
    instrument_id: String
    direction: String
    strength: String
    confidence: Float
    timeframe: String
    entry_price: Float
    target_price: Float
    stop_loss: Float
    reasoning: List<String>
    technical_factors: Map<String, Float>
    sentiment_factors: Map<String, Float>
    risk_factors: Map<String, Float>
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    signal_generated: {
        signal_id: "signal_20250621_001",
        instrument_id: "AAPL",
        direction: "BUY",
        strength: "STRONG",
        confidence: 0.85,
        timeframe: "1d",
        entry_price: 150.25,
        target_price: 155.00,
        stop_loss: 147.50,
        reasoning: [
            "Strong technical momentum with RSI recovery",
            "Positive sentiment from earnings beat",
            "Bullish pattern completion"
        ],
        technical_factors: {
            "rsi_signal": 0.75,
            "macd_signal": 0.68,
            "pattern_signal": 0.82
        },
        sentiment_factors: {
            "news_sentiment": 0.78,
            "social_sentiment": 0.65
        },
        risk_factors: {
            "volatility_risk": 0.25,
            "sector_risk": 0.30
        }
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table signal_generation_rules {
    id: UUID (primary key, auto-generated)
    rule_name: String (required, max_length: 100)
    rule_type: String (required, max_length: 50) // 'entry', 'exit', 'strength', 'confidence'
    conditions: JSON (required)
    actions: JSON (required)
    priority: Integer (default: 1)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table signal_parameters {
    id: UUID (primary key, auto-generated)
    instrument_id: String (max_length: 20)
    timeframe: String (max_length: 10)
    parameter_name: String (required, max_length: 100)
    parameter_value: Float (required)
    parameter_type: String (required, max_length: 50)
    last_updated: Timestamp (default: now)
}

Table signal_generation_stats {
    id: UUID (primary key, auto-generated)
    timestamp: Timestamp (required)
    signals_generated: Integer (default: 0)
    avg_generation_time_ms: Float
    avg_confidence: Float
    signal_distribution: JSON  // buy/sell/hold counts
    created_at: Timestamp (default: now)
}
```

### TimescaleDB (Query Side)
```pseudo
Table trading_signals_ts {
    timestamp: Timestamp (required, partition_key)
    signal_id: String (required, max_length: 100)
    instrument_id: String (required, max_length: 20)
    direction: String (required, max_length: 10)
    strength: String (required, max_length: 20)
    confidence: Float (required)
    timeframe: String (required, max_length: 10)
    entry_price: Float
    target_price: Float
    stop_loss: Float
    reasoning: JSON
    technical_factors: JSON
    sentiment_factors: JSON
    risk_factors: JSON
    generation_metadata: JSON

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
    partition_dimension: instrument_id (partitions: 8)
}
```

### Redis Caching
```pseudo
Cache signal_cache {
    // Latest signals
    "signals:{instrument_id}:latest": List<TradingSignal> (TTL: 5m)

    // Signal parameters
    "params:{instrument_id}:{timeframe}": SignalParameters (TTL: 1h)

    // Generation rules
    "rules:{rule_type}": List<GenerationRule> (TTL: 30m)

    // Market conditions
    "market_conditions:current": MarketConditions (TTL: 2m)
}
```

## Implementation Estimation

### Priority: **CRITICAL** (Core trading functionality)
### Estimated Time: **3-4 weeks**

#### Week 1-2: Core Signal Engine
- Python service setup with FastAPI
- Basic signal generation logic and rules engine
- Integration with Market Prediction workflow
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3: Advanced Signal Features
- Multi-timeframe signal synthesis
- Confidence calibration and scoring
- Market regime-aware signal adjustment
- **Effort**: 1 senior developer × 1 week = 1 dev-week

#### Week 4: Integration & Testing
- Integration testing with upstream workflows
- Performance testing and optimization
- Signal quality validation and backtesting
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **7 dev-weeks**
### Team Size: **2 developers** (1 senior Python developer, 1 mid-level developer)
### Dependencies: Market Prediction workflow, market data, TimescaleDB

### Success Criteria:
- P99 signal generation < 100ms
- 99.9% signal consistency across timeframes
- Support for 1K+ instruments simultaneously
- Signal confidence calibration accuracy > 90%
- Real-time signal generation capability
