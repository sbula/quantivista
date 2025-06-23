# Pre-Trade Risk Service

## Responsibility
Real-time risk validation and compliance checking before order execution. Validates orders against risk limits, regulatory requirements, and portfolio constraints with ultra-low latency.

## Technology Stack
- **Language**: Rust + Tokio + high-performance risk calculations
- **Libraries**: Tokio, serde, nalgebra for mathematical operations
- **Scaling**: Horizontal by risk calculation complexity
- **NFRs**: P99 risk validation < 25ms, 100% compliance accuracy, real-time monitoring

## API Specification

### Core APIs
```pseudo
// Enumerations
enum RiskCheckType {
    POSITION_LIMIT,
    EXPOSURE_LIMIT,
    CONCENTRATION_LIMIT,
    REGULATORY_LIMIT,
    LIQUIDITY_CHECK,
    MARGIN_CHECK
}

enum RiskStatus {
    APPROVED,
    REJECTED,
    WARNING,
    CONDITIONAL_APPROVAL
}

// Data Models
struct RiskValidationRequest {
    order_id: String
    instrument_id: String
    side: OrderSide
    quantity: Float
    price: Optional<Float>
    portfolio_id: String
    account_id: String
    risk_checks: List<RiskCheckType>
}

struct RiskValidationResponse {
    order_id: String
    overall_status: RiskStatus
    risk_results: List<RiskCheckResult>
    risk_score: Float
    validation_time_ms: Float
}

struct RiskCheckResult {
    check_type: RiskCheckType
    status: RiskStatus
    current_value: Float
    limit_value: Float
    utilization_percentage: Float
    details: String
}

// REST API Endpoints
POST /api/v1/risk/validate
    Request: RiskValidationRequest
    Response: RiskValidationResponse

GET /api/v1/risk/limits/{account_id}
    Response: RiskLimits

GET /api/v1/risk/status/{portfolio_id}
    Response: RiskStatus
```

### Event Output
```pseudo
Event risk_validation_completed {
    event_id: String
    timestamp: DateTime
    risk_validation: RiskValidationData
}

struct RiskValidationData {
    order_id: String
    overall_status: String
    risk_score: Float
    validation_time_ms: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    risk_validation: {
        order_id: "order_20250621_001",
        overall_status: "APPROVED",
        risk_score: 0.25,
        validation_time_ms: 15.2
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table risk_limits {
    id: UUID (primary key, auto-generated)
    account_id: String (required, max_length: 100)
    limit_type: String (required, max_length: 50)
    instrument_id: String (max_length: 20)
    limit_value: Decimal (required, precision: 15, scale: 2)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table risk_violations {
    id: UUID (primary key, auto-generated)
    order_id: String (required, max_length: 100)
    violation_type: String (required, max_length: 50)
    severity: String (required, max_length: 20)
    resolved: Boolean (default: false)
    created_at: Timestamp (default: now)
}
```

## Implementation Estimation

### Priority: **CRITICAL** (Compliance requirement)
### Estimated Time: **4-5 weeks**

#### Week 1-2: Core Risk Engine
- Rust service setup with Tokio
- Basic risk validation algorithms
- Risk limit management
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3-5: Advanced Features & Integration
- Complex risk calculations
- Integration with order management
- Performance testing and optimization
- **Effort**: 2 developers × 3 weeks = 6 dev-weeks

### Total Effort: **10 dev-weeks**
### Team Size: **2 developers** (1 senior Rust developer, 1 mid-level developer)
### Dependencies: Portfolio state, market data, regulatory requirements

### Success Criteria:
- P99 risk validation < 25ms
- 100% compliance accuracy
- Real-time risk monitoring
- Zero false positives in risk validation
