# Portfolio State Service

## Responsibility
Real-time portfolio state tracking and management including position tracking, exposure calculation, cash management, and portfolio performance monitoring. Provides high-performance portfolio analytics using advanced data processing frameworks.

## Technology Stack
- **Language**: Go + Polars + DuckDB for high-performance portfolio state management
- **Libraries**: Polars for DataFrame operations, DuckDB for complex portfolio analytics
- **Data Processing**: Polars for high-performance portfolio data manipulation and aggregations
- **Analytics**: DuckDB for complex exposure calculations and performance analytics
- **ML Framework**: JAX for mathematical optimization and advanced models
- **Scaling**: Horizontal by portfolio count, vertical for analytics-intensive operations
- **NFRs**: P99 state update latency < 50ms, throughput > 100K state updates/sec

## API Specification

### Internal APIs

#### Portfolio State API
```pseudo
// Enumerations
enum PositionStatus {
    ACTIVE,
    PENDING,
    CLOSED,
    SUSPENDED
}

enum ExposureType {
    INSTRUMENT,
    SECTOR,
    GEOGRAPHIC,
    CURRENCY,
    ASSET_CLASS
}

// Data Models
struct PortfolioStateRequest {
    portfolio_id: String
    include_positions: Boolean
    include_exposures: Boolean
    include_performance: Boolean
    as_of_timestamp: Optional<DateTime>
}

struct PortfolioStateResponse {
    portfolio_state: PortfolioState
    positions: List<Position>
    exposures: Map<ExposureType, List<Exposure>>
    performance_metrics: PerformanceMetrics
    last_updated: DateTime
}

struct PortfolioState {
    portfolio_id: String
    total_value: Float
    cash_balance: Float
    invested_value: Float
    available_cash: Float
    margin_used: Float
    margin_available: Float
    leverage_ratio: Float
    position_count: Integer
}

struct Position {
    position_id: String
    instrument_id: String
    quantity: Float
    market_value: Float
    cost_basis: Float
    unrealized_pnl: Float
    weight: Float
    status: PositionStatus
    last_updated: DateTime
}

struct Exposure {
    exposure_id: String
    exposure_type: ExposureType
    category: String
    exposure_value: Float
    exposure_percentage: Float
    limit_percentage: Float
    utilization: Float
}

struct PerformanceMetrics {
    total_return: Float
    daily_return: Float
    volatility: Float
    sharpe_ratio: Float
    max_drawdown: Float
    beta: Float
    alpha: Float
}
```

#### Position Management API
```pseudo
// Position update models
struct PositionUpdateRequest {
    position_id: String
    quantity_change: Float
    price: Float
    transaction_type: String
    timestamp: DateTime
}

struct PositionUpdateResponse {
    updated_position: Position
    portfolio_impact: PortfolioImpact
    exposure_changes: List<ExposureChange>
    processing_time_ms: Float
}

struct PortfolioImpact {
    value_change: Float
    weight_change: Float
    risk_change: Float
    cash_impact: Float
}

struct ExposureChange {
    exposure_type: ExposureType
    category: String
    old_percentage: Float
    new_percentage: Float
    change: Float
}

// REST API Endpoints
GET /api/v1/portfolio/{portfolio_id}/state
    Response: PortfolioStateResponse

POST /api/v1/portfolio/{portfolio_id}/positions/update
    Request: PositionUpdateRequest
    Response: PositionUpdateResponse

GET /api/v1/portfolio/{portfolio_id}/exposures
    Response: Map<ExposureType, List<Exposure>>

GET /api/v1/health
    Response: HealthResponse

GET /api/v1/metrics
    Response: MetricsResponse
```

### Event Output

#### PortfolioStateUpdatedEvent
```pseudo
Event portfolio_state_updated {
    event_id: String
    timestamp: DateTime
    portfolio_state: PortfolioStatePayload
    position_changes: List<PositionChangePayload>
    exposure_changes: List<ExposureChangePayload>
}

struct PortfolioStatePayload {
    portfolio_id: String
    total_value: Float
    cash_balance: Float
    invested_value: Float
    leverage_ratio: Float
    position_count: Integer
    last_updated: DateTime
}

struct PositionChangePayload {
    position_id: String
    instrument_id: String
    old_quantity: Float
    new_quantity: Float
    quantity_change: Float
    market_value: Float
    weight: Float
    change_type: String
}

struct ExposureChangePayload {
    exposure_type: String
    category: String
    old_percentage: Float
    new_percentage: Float
    change: Float
    limit_percentage: Float
    utilization: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-24T10:25:00.000Z",
    portfolio_state: {
        portfolio_id: "portfolio_001",
        total_value: 1000000.0,
        cash_balance: 50000.0,
        invested_value: 950000.0,
        leverage_ratio: 1.05,
        position_count: 25,
        last_updated: "2025-06-24T10:25:00.000Z"
    },
    position_changes: [
        {
            position_id: "pos_20250624_001",
            instrument_id: "AAPL",
            old_quantity: 100.0,
            new_quantity: 110.0,
            quantity_change: 10.0,
            market_value: 16527.5,
            weight: 0.0165,
            change_type: "INCREASE"
        }
    ],
    exposure_changes: [
        {
            exposure_type: "SECTOR",
            category: "Technology",
            old_percentage: 0.25,
            new_percentage: 0.26,
            change: 0.01,
            limit_percentage: 0.30,
            utilization: 0.87
        }
    ]
}
```

## Data Model

### Core Entities
```pseudo
// Data Models
struct Portfolio {
    portfolio_id: String
    portfolio_name: String
    base_currency: String
    portfolio_type: String
    inception_date: DateTime
    status: String
    benchmark: String
}

struct CashFlow {
    cash_flow_id: String
    portfolio_id: String
    amount: Float
    currency: String
    flow_type: String
    description: String
    timestamp: DateTime
}

struct MarginRequirement {
    requirement_id: String
    portfolio_id: String
    initial_margin: Float
    maintenance_margin: Float
    excess_liquidity: Float
    buying_power: Float
    last_updated: DateTime
}
```

## Database Schema (CQRS Pattern)

### Command Side (PostgreSQL)
```pseudo
// Portfolio configuration
Table portfolios {
    id: UUID (primary key, auto-generated)
    portfolio_id: String (required, unique, max_length: 50)
    portfolio_name: String (required, max_length: 100)
    base_currency: String (required, max_length: 3)
    portfolio_type: String (required, max_length: 50)
    inception_date: Date (required)
    status: String (required, max_length: 20)
    benchmark: String (max_length: 50)
    created_at: Timestamp (default: now)
    updated_at: Timestamp (default: now)
}

// Current positions
Table positions {
    id: UUID (primary key, auto-generated)
    position_id: String (required, unique, max_length: 100)
    portfolio_id: String (required, max_length: 50)
    instrument_id: String (required, max_length: 20)
    quantity: Decimal (precision: 15, scale: 6)
    cost_basis: Decimal (precision: 15, scale: 2)
    market_value: Decimal (precision: 15, scale: 2)
    unrealized_pnl: Decimal (precision: 15, scale: 2)
    weight: Float
    status: String (required, max_length: 20)
    last_updated: Timestamp (default: now)
    created_at: Timestamp (default: now)
}

// Cash flows
Table cash_flows {
    id: UUID (primary key, auto-generated)
    cash_flow_id: String (required, unique, max_length: 100)
    portfolio_id: String (required, max_length: 50)
    amount: Decimal (precision: 15, scale: 2)
    currency: String (required, max_length: 3)
    flow_type: String (required, max_length: 20)
    description: String (max_length: 255)
    timestamp: Timestamp (required)
    created_at: Timestamp (default: now)
}

// Margin requirements
Table margin_requirements {
    id: UUID (primary key, auto-generated)
    portfolio_id: String (required, max_length: 50)
    initial_margin: Decimal (precision: 15, scale: 2)
    maintenance_margin: Decimal (precision: 15, scale: 2)
    excess_liquidity: Decimal (precision: 15, scale: 2)
    buying_power: Decimal (precision: 15, scale: 2)
    last_updated: Timestamp (default: now)
    created_at: Timestamp (default: now)
}

// Indexes
idx_portfolios_id: (portfolio_id)
idx_positions_portfolio: (portfolio_id, status)
idx_positions_instrument: (instrument_id, status)
idx_cash_flows_portfolio_time: (portfolio_id, timestamp DESC)
idx_margin_requirements_portfolio: (portfolio_id, last_updated DESC)
```

### Query Side (TimescaleDB)
```pseudo
// Portfolio state time series
Table portfolio_state_ts {
    timestamp: Timestamp (required, partition_key)
    portfolio_id: String (required, max_length: 50)
    
    // Portfolio metrics
    total_value: Float
    cash_balance: Float
    invested_value: Float
    available_cash: Float
    margin_used: Float
    margin_available: Float
    leverage_ratio: Float
    position_count: Integer
    
    // Performance metrics
    daily_return: Float
    total_return: Float
    volatility: Float
    sharpe_ratio: Float
    max_drawdown: Float
    beta: Float
    alpha: Float
    
    created_at: Timestamp (default: now)
    
    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
    partition_dimension: portfolio_id (partitions: 8)
}

// Position changes time series
Table position_changes_ts {
    timestamp: Timestamp (required, partition_key)
    position_id: String (required, max_length: 100)
    portfolio_id: String (required, max_length: 50)
    instrument_id: String (required, max_length: 20)
    
    // Position data
    quantity: Float
    market_value: Float
    weight: Float
    unrealized_pnl: Float
    
    // Change data
    quantity_change: Float
    value_change: Float
    weight_change: Float
    change_type: String (max_length: 20)
    
    created_at: Timestamp (default: now)
    
    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
}

// Exposure tracking time series
Table exposures_ts {
    timestamp: Timestamp (required, partition_key)
    portfolio_id: String (required, max_length: 50)
    exposure_type: String (required, max_length: 20)
    category: String (required, max_length: 100)
    exposure_value: Float
    exposure_percentage: Float
    limit_percentage: Float
    utilization: Float
    
    created_at: Timestamp (default: now)
    
    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 hour)
}

// Indexes for fast queries
idx_portfolio_state_portfolio_time: (portfolio_id, timestamp DESC)
idx_position_changes_portfolio_time: (portfolio_id, timestamp DESC)
idx_position_changes_instrument_time: (instrument_id, timestamp DESC)
idx_exposures_portfolio_type_time: (portfolio_id, exposure_type, timestamp DESC)
```

### Redis Caching Strategy
```pseudo
Cache portfolio_state_cache {
    // Current portfolio state
    "portfolio_state:{portfolio_id}": PortfolioState (TTL: 5m)
    
    // Current positions
    "positions:{portfolio_id}": List<Position> (TTL: 5m)
    
    // Current exposures
    "exposures:{portfolio_id}": Map<ExposureType, List<Exposure>> (TTL: 10m)
    
    // Performance metrics
    "performance:{portfolio_id}": PerformanceMetrics (TTL: 15m)
    
    // Margin requirements
    "margin:{portfolio_id}": MarginRequirement (TTL: 10m)
    
    // Service performance
    "performance:current": StateServiceMetrics (TTL: 1m)
}
```

## Implementation Estimation

### Priority: **CRITICAL** (Essential for portfolio tracking)
### Estimated Time: **4-5 weeks**

#### Week 1-2: Core State Management
- Go service setup with Polars and DuckDB integration
- Portfolio state tracking and position management
- Real-time state updates and caching
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3: Advanced Analytics Features
- Exposure calculations using DuckDB
- Performance metrics computation with Polars
- Margin requirement tracking
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 4: Performance Optimization
- High-throughput state update optimization
- Memory management and efficient data structures
- Batch processing for multiple portfolios
- **Effort**: 1 senior developer × 1 week = 1 dev-week

#### Week 5: Integration & Testing
- Integration with other trading decision services
- Performance testing (100K+ updates/sec target)
- Error handling and state consistency validation
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **9 dev-weeks**
### Team Size: **2 developers** (1 senior Go developer, 1 mid-level developer)
### Dependencies:
- Market data feeds for position valuation
- Trade execution events for position updates
- TimescaleDB and PostgreSQL setup
- Redis for caching

### Risk Factors:
- **High**: Real-time state consistency requirements
- **Medium**: High-throughput performance requirements
- **Low**: Technology stack maturity

### Success Criteria:
- Process 100K+ state updates per second
- P99 state update latency < 50ms
- 100% state consistency across all operations
- Support for 1000+ concurrent portfolios
- Zero data loss during state updates
