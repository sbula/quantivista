# Impact Assessment Service

## Responsibility
Market impact prediction and correlation analysis for news events, social media trends, and sentiment changes. Provides impact scoring and market movement predictions.

## Technology Stack
- **Language**: Python + scikit-learn + XGBoost + time series analysis
- **Models**: Impact prediction, correlation analysis, event classification
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for custom optimization algorithms and advanced models
- **Scaling**: Horizontal by analysis complexity
- **NFRs**: P99 assessment latency < 1s, 85% impact prediction accuracy

## API Specification

### Core APIs
```pseudo
// Enumerations
enum EventType {
    NEWS,
    SOCIAL_TREND,
    EARNINGS,
    ECONOMIC_DATA,
    REGULATORY_ANNOUNCEMENT,
    CORPORATE_ACTION
}

enum ImpactDirection {
    POSITIVE,
    NEGATIVE,
    NEUTRAL
}

enum TimeHorizon {
    IMMEDIATE,
    SHORT_TERM,
    MEDIUM_TERM,
    LONG_TERM
}

// Data Models
struct ImpactAssessmentRequest {
    event_id: String
    event_type: EventType
    content: String
    sentiment_score: Float
    entities_mentioned: List<String>
    source_credibility: Float
    timestamp: DateTime
}

struct ImpactAssessmentResponse {
    event_id: String
    impact_score: Float  // 0.0 to 1.0
    impact_direction: ImpactDirection
    affected_instruments: List<String>
    time_horizon: TimeHorizon
    confidence: Float
    correlation_factors: Map<String, Float>
}

// REST API Endpoints
POST /api/v1/impact/assess
    Request: ImpactAssessmentRequest
    Response: ImpactAssessmentResponse
```

### Event Output
```pseudo
Event impact_assessment_completed {
    event_id: String
    timestamp: DateTime
    impact_assessment: ImpactAssessmentData
}

struct ImpactAssessmentData {
    event_id: String
    impact_score: Float
    impact_direction: String
    affected_instruments: List<String>
    time_horizon: String
    confidence: Float
    correlation_factors: CorrelationFactorsData
}

struct CorrelationFactorsData {
    earnings_beat: Float
    revenue_growth: Float
    guidance_raise: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    impact_assessment: {
        event_id: "earnings_aapl_q4_2025",
        impact_score: 0.85,
        impact_direction: "positive",
        affected_instruments: ["AAPL", "QQQ", "TECH_SECTOR"],
        time_horizon: "immediate",
        confidence: 0.78,
        correlation_factors: {
            earnings_beat: 0.65,
            revenue_growth: 0.45,
            guidance_raise: 0.30
        }
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table impact_models {
    id: UUID (primary key, auto-generated)
    model_name: String (required, max_length: 100)
    event_type: String (required, max_length: 50)
    model_config: JSON (required)
    accuracy_metrics: JSON
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table historical_impacts {
    id: UUID (primary key, auto-generated)
    event_id: String (required, max_length: 100)
    predicted_impact: Float
    actual_impact: Float
    accuracy_score: Float
    event_timestamp: Timestamp
    created_at: Timestamp (default: now)
}
```

## Implementation Estimation

### Priority: **MEDIUM** (Advanced intelligence feature)
### Estimated Time: **5-6 weeks**

#### Week 1-3: Core Impact Models
- Impact prediction model development
- Historical correlation analysis
- Event classification and scoring
- **Effort**: 2 ML engineers × 3 weeks = 6 dev-weeks

#### Week 4-5: Advanced Features
- Multi-timeframe impact analysis
- Cross-asset correlation modeling
- Model validation and backtesting
- **Effort**: 1 ML engineer × 2 weeks = 2 dev-weeks

#### Week 6: Integration & Testing
- Integration with sentiment and quality services
- Performance optimization
- Monitoring and alerting
- **Effort**: 1 developer × 1 week = 1 dev-week

### Total Effort: **9 dev-weeks**
### Team Size: **2 ML engineers + 1 developer**
### Dependencies: Historical market data, sentiment analysis service

### Success Criteria:
- 85% impact prediction accuracy
- P99 assessment latency < 1 second
- Support for 1K+ assessments per minute
- Cross-asset correlation analysis
