# Portfolio State Service

## Responsibility
Real-time portfolio state management and position tracking. Maintains current portfolio positions, weights, cash balances, and provides real-time portfolio analytics and state queries.

## Technology Stack
- **Language**: Go + PostgreSQL + Redis for high-performance state management
- **Libraries**: Go standard library, database drivers, caching libraries
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for custom optimization algorithms and advanced models
- **Scaling**: Horizontal by portfolio sharding, vertical for state complexity
- **NFRs**: P99 state query < 10ms, 99.99% data consistency, real-time updates

## API Specification

### Core APIs
```pseudo
// Enumerations
enum PositionType {
    LONG,
    SHORT,
    CASH
}

enum PortfolioStatus {
    ACTIVE,
    SUSPENDED,
    LIQUIDATING,
    CLOSED
}

// Data Models
struct PortfolioState {
    portfolio_id: String
    total_value: Float
    cash_balance: Float
    positions: List<Position>
    weights: Map<String, Float>
    last_updated: DateTime
    status: PortfolioStatus
}

struct Position {
    instrument_id: String
    position_type: PositionType
    quantity: Float
    market_value: Float
    weight: Float
    unrealized_pnl: Float
    cost_basis: Float
    last_updated: DateTime
}

struct PortfolioUpdate {
    portfolio_id: String
    updates: List<PositionUpdate>
    timestamp: DateTime
}

struct PositionUpdate {
    instrument_id: String
    quantity_change: Float
    price: Float
    transaction_type: String
}

// REST API Endpoints
GET /api/v1/portfolio/{portfolio_id}/state
    Response: PortfolioState

POST /api/v1/portfolio/{portfolio_id}/update
    Request: PortfolioUpdate
    Response: PortfolioState

GET /api/v1/portfolio/{portfolio_id}/positions
    Response: List<Position>

GET /api/v1/portfolio/{portfolio_id}/analytics
    Response: PortfolioAnalytics
```

### Event Output
```pseudo
Event portfolio_state_updated {
    event_id: String
    timestamp: DateTime
    portfolio_state_updated: PortfolioStateData
}

struct PortfolioStateData {
    portfolio_id: String
    total_value: Float
    cash_balance: Float
    positions: List<PositionData>
    last_updated: DateTime
}

struct PositionData {
    instrument_id: String
    quantity: Float
    market_value: Float
    weight: Float
    unrealized_pnl: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    portfolio_state_updated: {
        portfolio_id: "portfolio_001",
        total_value: 1000000.00,
        cash_balance: 50000.00,
        positions: [
            {
                instrument_id: "AAPL",
                quantity: 500,
                market_value: 75125.00,
                weight: 0.075,
                unrealized_pnl: 1250.00
            }
        ],
        last_updated: "2025-06-21T10:00:00.000Z"
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table portfolios {
    id: UUID (primary key, auto-generated)
    portfolio_id: String (required, unique, max_length: 100)
    portfolio_name: String (required, max_length: 200)
    total_value: Decimal (default: 0, precision: 15, scale: 2)
    cash_balance: Decimal (default: 0, precision: 15, scale: 2)
    status: String (default: 'active', max_length: 20)
    created_at: Timestamp (default: now)
    updated_at: Timestamp (default: now)
}

Table positions {
    id: UUID (primary key, auto-generated)
    portfolio_id: String (required, max_length: 100, foreign_key: portfolios.portfolio_id)
    instrument_id: String (required, max_length: 20)
    position_type: String (required, max_length: 10)
    quantity: Decimal (required, precision: 15, scale: 6)
    cost_basis: Decimal (precision: 15, scale: 6)
    market_value: Decimal (precision: 15, scale: 2)
    weight: Decimal (precision: 8, scale: 6)
    unrealized_pnl: Decimal (precision: 15, scale: 2)
    last_updated: Timestamp (default: now)

    // Constraints
    unique_portfolio_instrument: (portfolio_id, instrument_id)
}

Table portfolio_transactions {
    id: UUID (primary key, auto-generated)
    portfolio_id: String (required, max_length: 100, foreign_key: portfolios.portfolio_id)
    instrument_id: String (required, max_length: 20)
    transaction_type: String (required, max_length: 20)
    quantity: Decimal (required, precision: 15, scale: 6)
    price: Decimal (required, precision: 15, scale: 6)
    transaction_value: Decimal (required, precision: 15, scale: 2)
    transaction_date: Timestamp (default: now)
}
```

### Redis Caching Strategy
```pseudo
Cache portfolio_cache {
    // Current state
    "portfolio:{portfolio_id}:state": PortfolioState (TTL: 5m)

    // Position cache
    "portfolio:{portfolio_id}:positions": List<Position> (TTL: 1m)

    // Analytics cache
    "portfolio:{portfolio_id}:analytics": PortfolioAnalytics (TTL: 15m)

    // Weight cache
    "portfolio:{portfolio_id}:weights": Map<String, Float> (TTL: 5m)
}
```

## Implementation Estimation

### Priority: **CRITICAL** (Foundation for portfolio management)
### Estimated Time: **4-5 weeks**

#### Week 1-2: Core State Management
- Go service setup with PostgreSQL integration
- Basic portfolio state CRUD operations
- Position tracking and updates
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3: Real-time Features
- Redis caching layer implementation
- Real-time state synchronization
- Event streaming integration
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 4-5: Advanced Features & Integration
- Portfolio analytics calculations
- Performance optimization
- Integration testing and validation
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **10 dev-weeks**
### Team Size: **2 developers** (1 senior Go developer, 1 mid-level developer)
### Dependencies: Market data for pricing, transaction feeds

### Success Criteria:
- P99 state query < 10ms
- 99.99% data consistency
- Real-time position updates
- Support for 1000+ portfolios simultaneously
- Zero data loss during updates
