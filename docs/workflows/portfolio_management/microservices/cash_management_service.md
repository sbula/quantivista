# Cash Management Service

## Responsibility
Portfolio cash flow management, liquidity optimization, and cash allocation across strategies. Manages cash positions, handles deposits/withdrawals, and optimizes cash utilization for trading operations.

## Technology Stack
- **Language**: Python + FastAPI + PostgreSQL + Redis
- **Libraries**: FastAPI, SQLAlchemy, pandas, cash flow optimization libraries
- **Scaling**: Horizontal by portfolio groups
- **NFRs**: P99 cash operation < 100ms, 100% cash reconciliation accuracy

## API Specification

### Core APIs
```pseudo
// Enumerations
enum CashTransactionType {
    DEPOSIT,
    WITHDRAWAL,
    TRADE_SETTLEMENT,
    DIVIDEND,
    INTEREST,
    FEE
}

enum CashAllocationStrategy {
    EQUAL_WEIGHT,
    STRATEGY_BASED,
    OPPORTUNITY_BASED,
    MANUAL
}

// Data Models
struct CashManagementRequest {
    portfolio_id: String
    transaction_type: CashTransactionType
    amount: Float
    allocation_strategy: Optional<CashAllocationStrategy>
}

struct CashPosition {
    portfolio_id: String
    total_cash: Float
    available_cash: Float
    reserved_cash: Float
    pending_settlements: Float
    cash_allocations: Map<String, Float>
    last_updated: DateTime
}

// REST API Endpoints
POST /api/v1/cash/transaction
    Request: CashManagementRequest
    Response: CashPosition

GET /api/v1/cash/{portfolio_id}/position
    Response: CashPosition

POST /api/v1/cash/{portfolio_id}/allocate
    Request: CashAllocationRequest
    Response: CashAllocationResponse
```

### Event Output
```pseudo
Event cash_transaction_processed {
    event_id: String
    timestamp: DateTime
    cash_transaction: CashTransactionData
}

struct CashTransactionData {
    portfolio_id: String
    transaction_type: String
    amount: Float
    new_cash_position: CashPositionData
}

struct CashPositionData {
    total_cash: Float
    available_cash: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    cash_transaction: {
        portfolio_id: "portfolio_001",
        transaction_type: "DEPOSIT",
        amount: 50000.00,
        new_cash_position: {
            total_cash: 125000.00,
            available_cash: 120000.00
        }
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table cash_positions {
    id: UUID (primary key, auto-generated)
    portfolio_id: String (required, unique, max_length: 100)
    total_cash: Decimal (required, default: 0, precision: 15, scale: 2)
    available_cash: Decimal (required, default: 0, precision: 15, scale: 2)
    reserved_cash: Decimal (required, default: 0, precision: 15, scale: 2)
    last_updated: Timestamp (default: now)
}

Table cash_transactions {
    id: UUID (primary key, auto-generated)
    portfolio_id: String (required, max_length: 100)
    transaction_type: String (required, max_length: 50)
    amount: Decimal (required, precision: 15, scale: 2)
    transaction_date: Timestamp (default: now)
    status: String (default: 'pending', max_length: 20)
}
```

## Implementation Estimation

### Priority: **HIGH** (Essential for portfolio operations)
### Estimated Time: **3-4 weeks**

#### Week 1-2: Core Cash Management
- Cash transaction processing
- Cash position tracking and reconciliation
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3-4: Advanced Features & Integration
- Cash allocation optimization
- Integration with portfolio state service
- Performance testing and optimization
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **8 dev-weeks**
### Team Size: **2 developers**
### Dependencies: Portfolio state service, trading systems

### Success Criteria:
- P99 cash operation < 100ms
- 100% cash reconciliation accuracy
- Real-time cash position updates
- Optimal cash allocation efficiency > 95%
