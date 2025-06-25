# Investment Rating Service

## Responsibility
Generate comprehensive instrument evaluations and investment ratings by synthesizing multi-timeframe predictions into actionable investment recommendations with confidence scoring and technical confirmation integration.

## Technology Stack
- **Language**: Python + FastAPI + NumPy + Redis
- **Libraries**: NumPy, Scikit-learn, Redis client, Apache Pulsar client
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for custom optimization algorithms and advanced models
- **Scaling**: Horizontal by instrument volume, rating type partitioning
- **NFRs**: P99 evaluation latency < 1s, 2K+ evaluations/sec throughput, 80% rating confidence

## API Specification

### Core APIs
```pseudo
// Enumerations
enum InvestmentRating {
    STRONG_BUY,
    BUY,
    NEUTRAL,
    SELL,
    STRONG_SELL
}

enum EvaluationTimeframe {
    SHORT_TERM,    // 1h-4h
    MEDIUM_TERM,   // 1d-1w
    LONG_TERM      // 1w-1mo
}

enum ConfidenceLevel {
    VERY_HIGH,     // 90%+
    HIGH,          // 80-90%
    MEDIUM,        // 70-80%
    LOW,           // 60-70%
    VERY_LOW       // <60%
}

// Data Models
struct EvaluationRequest {
    instrument_id: String
    evaluation_timeframes: List<EvaluationTimeframe>
    include_technical_confirmation: Boolean
    minimum_confidence: Float
}

struct InstrumentEvaluation {
    evaluation_id: String
    instrument_id: String
    overall_rating: InvestmentRating
    timeframe_ratings: Map<EvaluationTimeframe, InvestmentRating>
    confidence_score: Float
    confidence_level: ConfidenceLevel
    technical_confirmation: TechnicalConfirmation
    price_targets: Map<EvaluationTimeframe, Float>
    risk_assessment: RiskAssessment
    timestamp: DateTime
}

struct TechnicalConfirmation {
    momentum_confirmation: Boolean
    trend_confirmation: Boolean
    volume_confirmation: Boolean
    pattern_confirmation: Boolean
    overall_confirmation: Boolean
}

struct RiskAssessment {
    volatility_risk: String
    liquidity_risk: String
    correlation_risk: String
    overall_risk: String
}

// REST API Endpoints
POST /api/v1/evaluations/generate
    Request: EvaluationRequest
    Response: InstrumentEvaluation

POST /api/v1/evaluations/batch
    Request: List<EvaluationRequest>
    Response: List<InstrumentEvaluation>

GET /api/v1/evaluations/{instrument_id}
    Response: InstrumentEvaluation

GET /api/v1/evaluations/ratings/distribution
    Response: RatingDistribution

POST /api/v1/evaluations/confidence/update
    Request: ConfidenceFeedback
    Response: UpdateResult
```

### Event Output
```pseudo
Event instrument_evaluated {
    event_id: String
    timestamp: DateTime
    instrument_evaluated: InstrumentEvaluatedData
}

struct InstrumentEvaluatedData {
    evaluation_batch_id: String
    instrument_id: String
    overall_rating: String
    confidence_score: Float
    evaluation_duration_ms: Integer
    timeframe_count: Integer
    technical_confirmation: Boolean
    rating_distribution: RatingDistribution
}

struct RatingDistribution {
    strong_buy_count: Integer
    buy_count: Integer
    neutral_count: Integer
    sell_count: Integer
    strong_sell_count: Integer
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:25:00.000Z",
    instrument_evaluated: {
        evaluation_batch_id: "evaluation_20250621_001",
        instrument_id: "AAPL",
        overall_rating: "BUY",
        confidence_score: 0.82,
        evaluation_duration_ms: 800,
        timeframe_count: 3,
        technical_confirmation: true,
        rating_distribution: {
            strong_buy_count: 1,
            buy_count: 2,
            neutral_count: 0,
            sell_count: 0,
            strong_sell_count: 0
        }
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table evaluation_jobs {
    id: UUID (primary key, auto-generated)
    job_id: String (required, unique, max_length: 100)
    instrument_id: String (required, max_length: 50)
    timeframes: JSON (required)
    status: String (required, max_length: 20)
    overall_rating: String (max_length: 20)
    confidence_score: Float
    evaluation_duration_ms: Float
    created_at: Timestamp (default: now)
}

Table rating_history {
    id: UUID (primary key, auto-generated)
    instrument_id: String (required, max_length: 50)
    rating: String (required, max_length: 20)
    confidence_score: Float (required)
    timeframe: String (required, max_length: 20)
    technical_confirmation: Boolean
    created_at: Timestamp (default: now)
}

Table confidence_feedback {
    id: UUID (primary key, auto-generated)
    evaluation_id: String (required, max_length: 100)
    predicted_rating: String (required, max_length: 20)
    actual_outcome: String (max_length: 20)
    confidence_score: Float (required)
    accuracy: Boolean
    feedback_date: Timestamp (default: now)
}
```

### Redis Caching
```pseudo
Cache evaluation_cache {
    // Generated evaluations
    "evaluations:{instrument_id}": InstrumentEvaluation (TTL: 15m)
    
    // Rating distributions
    "rating_distribution": RatingDistribution (TTL: 30m)
    
    // Confidence metrics
    "confidence_metrics": ConfidenceMetrics (TTL: 1h)
    
    // Evaluation status
    "evaluation_status:{job_id}": EvaluationStatus (TTL: 1h)
}
```

## Implementation Estimation

### Priority: **HIGH** (Core evaluation and rating engine)
### Estimated Time: **3-4 weeks**

#### Week 1-2: Core Evaluation Engine
- Python service setup with evaluation algorithms
- Multi-timeframe rating synthesis
- Confidence scoring implementation
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3: Advanced Features
- Technical confirmation integration
- Risk assessment capabilities
- Performance optimization
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 4: Integration & Testing
- Integration with prediction engine
- Quality assurance and testing
- Performance tuning
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **8 dev-weeks**
### Team Size: **2 developers** (1 senior Python developer, 1 quantitative analyst)
### Dependencies: Market Prediction Engine Service, Instrument Analysis Service

### Success Criteria:
- P99 evaluation latency < 1 second
- 2K+ evaluations per second throughput
- 80% minimum rating confidence
- Multi-timeframe evaluation capability
- Technical confirmation integration
- Risk assessment functionality
