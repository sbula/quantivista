# Risk Budget Service

## Responsibility
Portfolio risk budget allocation and monitoring across strategies, sectors, and instruments. Manages risk limits, tracks risk utilization, and provides real-time risk budget optimization.

## Technology Stack
- **Language**: Python + NumPy + SciPy + risk modeling libraries
- **Libraries**: NumPy, SciPy, risk management libraries
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for custom optimization algorithms and advanced models
- **Scaling**: Horizontal by risk calculation complexity
- **NFRs**: P99 risk calculation < 500ms, accurate risk budgeting

## API Specification

### Core APIs
```pseudo
// Enumerations
enum RiskBudgetType {
    VOLATILITY_BUDGET,
    VAR_BUDGET,
    TRACKING_ERROR_BUDGET,
    CONCENTRATION_BUDGET
}

enum AllocationMethod {
    EQUAL_WEIGHT,
    RISK_PARITY,
    INVERSE_VOLATILITY,
    CUSTOM_WEIGHTS
}

// Data Models
struct RiskBudgetRequest {
    portfolio_id: String
    budget_type: RiskBudgetType
    total_budget: Float
    allocation_method: AllocationMethod
    constraints: RiskConstraints
}

struct RiskBudgetResponse {
    portfolio_id: String
    budget_allocations: Map<String, RiskAllocation>
    total_budget_used: Float
    budget_utilization: Float
    risk_concentration: Map<String, Float>
}

struct RiskAllocation {
    entity_id: String  // strategy, sector, or instrument
    allocated_budget: Float
    current_usage: Float
    utilization_rate: Float
    remaining_budget: Float
}

struct RiskConstraints {
    max_concentration: Float
    min_allocation: Float
    max_allocation: Float
    excluded_entities: List<String>
}

// REST API Endpoints
POST /api/v1/risk-budget/allocate
    Request: RiskBudgetRequest
    Response: RiskBudgetResponse

GET /api/v1/risk-budget/{portfolio_id}/status
    Response: RiskBudgetStatus

PUT /api/v1/risk-budget/{portfolio_id}/update
    Request: RiskBudgetUpdate
    Response: RiskBudgetResponse
```

### Event Output
```pseudo
Event risk_budget_updated {
    event_id: String
    timestamp: DateTime
    risk_budget_updated: RiskBudgetData
}

struct RiskBudgetData {
    portfolio_id: String
    budget_type: String
    budget_allocations: Map<String, StrategyAllocationData>
    total_budget_used: Float
    budget_utilization: Float
}

struct StrategyAllocationData {
    allocated_budget: Float
    current_usage: Float
    utilization_rate: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    risk_budget_updated: {
        portfolio_id: "portfolio_001",
        budget_type: "VAR_BUDGET",
        budget_allocations: {
            "strategy_momentum": {
                allocated_budget: 0.008,
                current_usage: 0.006,
                utilization_rate: 0.75
            },
            "strategy_value": {
                allocated_budget: 0.006,
                current_usage: 0.005,
                utilization_rate: 0.83
            }
        },
        total_budget_used: 0.011,
        budget_utilization: 0.79
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table risk_budgets {
    id: UUID (primary key, auto-generated)
    portfolio_id: String (required, max_length: 100)
    budget_type: String (required, max_length: 50)
    total_budget: Float (required)
    allocation_method: String (required, max_length: 50)
    constraints: JSON
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table risk_allocations {
    id: UUID (primary key, auto-generated)
    portfolio_id: String (required, max_length: 100)
    entity_id: String (required, max_length: 100)
    entity_type: String (required, max_length: 50)
    allocated_budget: Float (required)
    current_usage: Float (default: 0)
    last_updated: Timestamp (default: now)
}
```

## Implementation Estimation

### Priority: **HIGH** (Critical for risk management)
### Estimated Time: **3-4 weeks**

#### Week 1-2: Core Risk Budget Engine
- Risk budget allocation algorithms
- Risk utilization tracking
- Basic budget monitoring
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3-4: Advanced Features & Integration
- Advanced allocation methods
- Real-time risk monitoring
- Integration with portfolio optimization
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **8 dev-weeks**
### Team Size: **2 developers** (1 senior quantitative developer, 1 mid-level developer)
### Dependencies: Portfolio state, risk models, correlation data

### Success Criteria:
- P99 risk calculation < 500ms
- Risk budget accuracy > 98%
- Real-time risk monitoring
- Optimal budget allocation efficiency > 90%
