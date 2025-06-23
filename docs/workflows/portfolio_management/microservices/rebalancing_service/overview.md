# Rebalancing Service

## Responsibility
Portfolio rebalancing trigger generation and execution coordination. Monitors portfolio drift, generates rebalancing recommendations, and coordinates with trading systems for optimal portfolio maintenance.

## Technology Stack
- **Language**: Python + NumPy + SciPy + optimization libraries
- **Libraries**: NumPy, SciPy, pandas, optimization libraries
- **Scaling**: Horizontal by portfolio complexity
- **NFRs**: P99 rebalancing analysis < 1s, optimal rebalancing decisions

## API Specification

### Core APIs
```pseudo
// Enumerations
enum RebalancingTrigger {
    DRIFT_THRESHOLD,
    TIME_BASED,
    VOLATILITY_CHANGE,
    CORRELATION_CHANGE,
    MANUAL
}

enum RebalancingMethod {
    FULL_OPTIMIZATION,
    THRESHOLD_REBALANCING,
    PROPORTIONAL_REBALANCING,
    TACTICAL_REBALANCING
}

// Data Models
struct RebalancingRequest {
    portfolio_id: String
    trigger_type: RebalancingTrigger
    method: RebalancingMethod
    constraints: RebalancingConstraints
    force_rebalance: Boolean
}

struct RebalancingConstraints {
    max_turnover: Float
    min_trade_size: Float
    transaction_cost_limit: Float
    excluded_instruments: List<String>
}

struct RebalancingResponse {
    portfolio_id: String
    rebalancing_required: Boolean
    recommended_trades: List<RebalancingTrade>
    expected_turnover: Float
    estimated_costs: Float
    rebalancing_rationale: String
}

struct RebalancingTrade {
    instrument_id: String
    current_weight: Float
    target_weight: Float
    trade_amount: Float
    trade_direction: String
    priority: Integer
}

// REST API Endpoints
POST /api/v1/rebalancing/analyze
    Request: RebalancingRequest
    Response: RebalancingResponse

GET /api/v1/rebalancing/{portfolio_id}/status
    Response: RebalancingStatus

POST /api/v1/rebalancing/{portfolio_id}/trigger
    Request: RebalancingTrigger
    Response: RebalancingResponse
```

### Event Output
```pseudo
Event rebalancing_triggered {
    event_id: String
    timestamp: DateTime
    rebalancing_triggered: RebalancingData
}

struct RebalancingData {
    portfolio_id: String
    trigger_type: String
    rebalancing_required: Boolean
    recommended_trades: List<RecommendedTradeData>
    expected_turnover: Float
    estimated_costs: Float
}

struct RecommendedTradeData {
    instrument_id: String
    current_weight: Float
    target_weight: Float
    trade_amount: Float
    trade_direction: String
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    rebalancing_triggered: {
        portfolio_id: "portfolio_001",
        trigger_type: "DRIFT_THRESHOLD",
        rebalancing_required: true,
        recommended_trades: [
            {
                instrument_id: "AAPL",
                current_weight: 0.12,
                target_weight: 0.10,
                trade_amount: -5000,
                trade_direction: "sell"
            }
        ],
        expected_turnover: 0.05,
        estimated_costs: 125.50
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table rebalancing_rules {
    id: UUID (primary key, auto-generated)
    portfolio_id: String (required, max_length: 100)
    trigger_type: String (required, max_length: 50)
    trigger_parameters: JSON (required)
    method: String (required, max_length: 50)
    constraints: JSON
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table rebalancing_history {
    id: UUID (primary key, auto-generated)
    portfolio_id: String (required, max_length: 100)
    trigger_type: String (required, max_length: 50)
    trades_executed: Integer
    turnover_achieved: Float
    actual_costs: Float
    execution_date: Timestamp
    created_at: Timestamp (default: now)
}
```

## Implementation Estimation

### Priority: **HIGH** (Critical for portfolio maintenance)
### Estimated Time: **3-4 weeks**

#### Week 1-2: Core Rebalancing Engine
- Drift detection and threshold monitoring
- Basic rebalancing algorithms
- Trade recommendation generation
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3-4: Advanced Features & Integration
- Advanced rebalancing methods
- Cost optimization and transaction analysis
- Integration with trading coordination
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **8 dev-weeks**
### Team Size: **2 developers**
### Dependencies: Portfolio state, market data, trading coordination

### Success Criteria:
- P99 analysis latency < 1 second
- Optimal rebalancing decisions > 95% accuracy
- Cost-effective trade recommendations
- Real-time drift monitoring
