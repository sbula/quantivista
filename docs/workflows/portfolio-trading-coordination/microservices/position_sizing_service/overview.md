# Position Sizing Service

## Responsibility
Kelly criterion and portfolio-aware position sizing. Calculates optimal position sizes based on signal confidence, portfolio constraints, correlation matrices, and risk management principles.

## Technology Stack
- **Language**: Python + NumPy + SciPy + optimization libraries
- **Libraries**: NumPy, SciPy, cvxpy, portfolio optimization libraries
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for mathematical optimization and advanced models
- **Scaling**: Horizontal by calculation complexity
- **NFRs**: P99 sizing calculation < 150ms, mathematically optimal sizing

## API Specification

### Core APIs
```pseudo
// Data Models
struct PositionSizingRequest {
    signal: TradingSignal
    portfolio_state: PortfolioState
    correlation_matrix: CorrelationMatrix
    risk_parameters: RiskParameters
}

struct PositionSizingResponse {
    instrument_id: String
    recommended_size: Float
    kelly_fraction: Float
    risk_adjusted_size: Float
    correlation_adjustment: Float
    sizing_rationale: SizingRationale
}

struct SizingRationale {
    base_kelly_size: Float
    correlation_penalty: Float
    portfolio_constraint_adjustment: Float
    risk_budget_adjustment: Float
    final_size: Float
    confidence_factor: Float
}

// REST API Endpoints
POST /api/v1/calculate
    Request: PositionSizingRequest
    Response: PositionSizingResponse

POST /api/v1/batch-calculate
    Request: List<PositionSizingRequest>
    Response: List<PositionSizingResponse>
```

### Event Output
```pseudo
Event position_sized {
    event_id: String
    timestamp: DateTime
    position_sized: PositionSizedData
}

struct PositionSizedData {
    instrument_id: String
    recommended_size: Float
    kelly_fraction: Float
    risk_adjusted_size: Float
    correlation_adjustment: Float
    sizing_rationale: SizingRationaleData
}

struct SizingRationaleData {
    base_kelly_size: Float
    correlation_penalty: Float
    final_size: Float
    confidence_factor: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    position_sized: {
        instrument_id: "AAPL",
        recommended_size: 0.025,
        kelly_fraction: 0.035,
        risk_adjusted_size: 0.025,
        correlation_adjustment: -0.010,
        sizing_rationale: {
            base_kelly_size: 0.035,
            correlation_penalty: -0.010,
            final_size: 0.025,
            confidence_factor: 0.85
        }
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table sizing_parameters {
    id: UUID (primary key, auto-generated)
    instrument_id: String (max_length: 20)
    parameter_name: String (required, max_length: 100)
    parameter_value: Float (required)
    parameter_type: String (required, max_length: 50)
    last_updated: Timestamp (default: now)
}

Table sizing_history {
    id: UUID (primary key, auto-generated)
    instrument_id: String (required, max_length: 20)
    signal_confidence: Float
    kelly_fraction: Float
    final_size: Float
    performance_outcome: Float
    created_at: Timestamp (default: now)
}
```

## Implementation Estimation

### Priority: **HIGH** (Critical for risk management)
### Estimated Time: **4-5 weeks**

#### Week 1-2: Core Sizing Engine
- Kelly criterion implementation
- Basic position sizing algorithms
- Portfolio constraint integration
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3-4: Advanced Features
- Correlation-aware sizing adjustments
- Multi-objective optimization
- Risk budget allocation
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 5: Integration & Testing
- Integration with coordination engine
- Performance testing and validation
- Sizing accuracy backtesting
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **10 dev-weeks**
### Team Size: **2 developers** (1 senior quantitative developer, 1 mid-level developer)
### Dependencies: Correlation matrices, portfolio state, risk parameters

### Success Criteria:
- P99 calculation latency < 150ms
- Mathematically optimal sizing accuracy > 95%
- Kelly criterion implementation accuracy > 99%
- Correlation adjustment effectiveness > 85%
