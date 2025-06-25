# Corporate Actions Service

## Responsibility
Corporate actions data collection, processing, and distribution. Handles dividends, stock splits, mergers, spin-offs, and other corporate events that affect instrument pricing and portfolio positions.

## Technology Stack
- **Language**: Python + FastAPI + financial data libraries
- **Libraries**: FastAPI, numpy, financial data processing libraries
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for mathematical optimization and advanced models
- **Scaling**: Horizontal by corporate action complexity
- **NFRs**: P99 processing < 2s, 99.9% data accuracy, comprehensive event coverage

## API Specification

### Core APIs
```pseudo
// Enumerations
enum CorporateActionType {
    DIVIDEND,
    STOCK_SPLIT,
    STOCK_DIVIDEND,
    MERGER,
    ACQUISITION,
    SPIN_OFF,
    RIGHTS_OFFERING,
    SPECIAL_DIVIDEND,
    REVERSE_SPLIT
}

enum ActionStatus {
    ANNOUNCED,
    CONFIRMED,
    EX_DATE,
    RECORD_DATE,
    PAYMENT_DATE,
    COMPLETED,
    CANCELLED
}

// Data Models
struct CorporateAction {
    action_id: String
    instrument_id: String
    action_type: CorporateActionType
    status: ActionStatus
    announcement_date: Date
    ex_date: Date
    record_date: Date
    payment_date: Date
    action_details: ActionDetails
    adjustment_factor: Optional<Float>
    cash_amount: Optional<Float>
}

struct ActionDetails {
    description: String
    ratio: Optional<String>  // e.g., "2:1" for stock split
    new_instrument_id: Optional<String>  // for spin-offs
    cash_in_lieu: Optional<Float>
    tax_implications: Optional<String>
    source: String
}

struct CorporateActionRequest {
    instrument_ids: List<String>
    action_types: List<CorporateActionType>
    date_range: DateRange
    include_historical: Boolean
}

struct CorporateActionResponse {
    actions: List<CorporateAction>
    total_count: Integer
    processing_time_ms: Float
}

// REST API Endpoints
GET /api/v1/corporate-actions
    Parameters: instrument_id, action_type, start_date, end_date
    Response: CorporateActionResponse

GET /api/v1/corporate-actions/{action_id}
    Response: CorporateAction

POST /api/v1/corporate-actions/upcoming
    Request: CorporateActionRequest
    Response: CorporateActionResponse

GET /api/v1/corporate-actions/adjustments/{instrument_id}
    Parameters: date
    Response: List<PriceAdjustment>
```

### Event Output
```pseudo
Event corporate_action_processed {
    event_id: String
    timestamp: DateTime
    corporate_action: CorporateActionData
}

struct CorporateActionData {
    action_id: String
    instrument_id: String
    action_type: String
    status: String
    announcement_date: String
    ex_date: String
    record_date: String
    payment_date: String
    action_details: ActionDetailsData
    cash_amount: Float
}

struct ActionDetailsData {
    description: String
    cash_amount: Float
    source: String
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    corporate_action: {
        action_id: "ca_20250621_001",
        instrument_id: "AAPL",
        action_type: "DIVIDEND",
        status: "ANNOUNCED",
        announcement_date: "2025-06-21",
        ex_date: "2025-07-15",
        record_date: "2025-07-16",
        payment_date: "2025-08-01",
        action_details: {
            description: "Quarterly dividend payment",
            cash_amount: 0.25,
            source: "company_filing"
        },
        cash_amount: 0.25
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table corporate_actions {
    id: UUID (primary key, auto-generated)
    action_id: String (required, unique, max_length: 100)
    instrument_id: String (required, max_length: 20)
    action_type: String (required, max_length: 50)
    status: String (required, max_length: 20)
    announcement_date: Date
    ex_date: Date
    record_date: Date
    payment_date: Date
    action_details: JSON (required)
    adjustment_factor: Decimal (precision: 10, scale: 6)
    cash_amount: Decimal (precision: 15, scale: 4)
    created_at: Timestamp (default: now)
    updated_at: Timestamp (default: now)
}

Table price_adjustments {
    id: UUID (primary key, auto-generated)
    action_id: String (required, max_length: 100, foreign_key: corporate_actions.action_id)
    instrument_id: String (required, max_length: 20)
    adjustment_date: Date (required)
    adjustment_factor: Decimal (required, precision: 10, scale: 6)
    adjustment_type: String (required, max_length: 50)
    applied: Boolean (default: false)
    created_at: Timestamp (default: now)
}
```

### TimescaleDB (Query Side)
```pseudo
Table corporate_actions_ts {
    timestamp: Timestamp (required, partition_key)
    action_id: String (required, max_length: 100)
    instrument_id: String (required, max_length: 20)
    action_type: String (required, max_length: 50)
    status: String (required, max_length: 20)
    ex_date: Date
    payment_date: Date
    cash_amount: Decimal (precision: 15, scale: 4)
    adjustment_factor: Decimal (precision: 10, scale: 6)
    action_details: JSON

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 month)
    partition_dimension: instrument_id (partitions: 8)
}
```

### Redis Caching
```pseudo
Cache corporate_action_cache {
    // Upcoming actions
    "upcoming_actions:{instrument_id}": List<CorporateAction> (TTL: 1h)

    // Price adjustments
    "adjustments:{instrument_id}:{date}": List<PriceAdjustment> (TTL: 24h)

    // Action lookup
    "action:{action_id}": CorporateAction (TTL: 6h)

    // Ex-date calendar
    "ex_dates:{date}": List<String> (TTL: 24h)
}
```

## Implementation Estimation

### Priority: **MEDIUM** (Important for accurate pricing)
### Estimated Time: **3-4 weeks**

#### Week 1-2: Core Corporate Actions Engine
- Python service setup with FastAPI
- Corporate action data ingestion and processing
- Basic action type handling (dividends, splits)
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3: Advanced Features
- Complex corporate actions (mergers, spin-offs)
- Price adjustment calculations
- Historical data processing
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 4: Integration & Testing
- Integration with market data services
- Data validation and quality checks
- Performance testing and optimization
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **8 dev-weeks**
### Team Size: **2 developers** (1 senior Python developer, 1 financial data specialist)
### Dependencies: Market data feeds, corporate action data sources

### Success Criteria:
- P99 processing latency < 2 seconds
- 99.9% data accuracy
- Comprehensive corporate action coverage
- Accurate price adjustment calculations
- Real-time corporate action alerts
