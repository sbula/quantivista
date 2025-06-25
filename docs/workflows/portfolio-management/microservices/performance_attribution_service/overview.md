# Performance Attribution Service

## Responsibility
Multi-level performance analysis and attribution across portfolio, strategy, sector, and security levels. Provides comprehensive factor attribution, risk-adjusted performance metrics, and benchmark comparison analysis.

## Technology Stack
- **Language**: Python + NumPy + QuantLib + performance analytics libraries
- **Libraries**: numpy, QuantLib, pyfolio, empyrical for performance analysis
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for custom optimization algorithms and advanced models
- **Scaling**: Horizontal by attribution complexity
- **NFRs**: P99 attribution calculation < 2s, accurate factor attribution, comprehensive analysis

## API Specification

### Core APIs
```pseudo
// Enumerations
enum AttributionLevel {
    PORTFOLIO,
    STRATEGY,
    SECTOR,
    SECURITY
}

enum AttributionMethod {
    BRINSON_FACHLER,
    BRINSON_HOOD_BEEBOWER,
    FACTOR_ATTRIBUTION,
    RISK_ATTRIBUTION
}

// Data Models
struct AttributionRequest {
    portfolio_id: String
    attribution_level: AttributionLevel
    method: AttributionMethod
    start_date: Date
    end_date: Date
    benchmark: Optional<String>
}

struct AttributionResponse {
    portfolio_id: String
    attribution_level: AttributionLevel
    total_return: Float
    benchmark_return: Float
    active_return: Float
    attribution_breakdown: AttributionBreakdown
    performance_metrics: Map<String, Float>
}

struct AttributionBreakdown {
    allocation_effect: Float
    selection_effect: Float
    interaction_effect: Float
    factor_contributions: Map<String, Float>
    sector_contributions: Map<String, Float>
}

// REST API Endpoints
POST /api/v1/attribution/calculate
    Request: AttributionRequest
    Response: AttributionResponse

GET /api/v1/attribution/{portfolio_id}/latest
    Parameters: attribution_level, period
    Response: AttributionResponse
```

### Event Output
```pseudo
Event performance_attribution_completed {
    event_id: String
    timestamp: DateTime
    performance_attribution: PerformanceAttributionData
}

struct PerformanceAttributionData {
    portfolio_id: String
    total_return: Float
    benchmark_return: Float
    active_return: Float
    attribution_breakdown: AttributionBreakdownData
}

struct AttributionBreakdownData {
    allocation_effect: Float
    selection_effect: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    performance_attribution: {
        portfolio_id: "portfolio_001",
        total_return: 0.085,
        benchmark_return: 0.072,
        active_return: 0.013,
        attribution_breakdown: {
            allocation_effect: 0.008,
            selection_effect: 0.005
        }
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table attribution_configurations {
    id: UUID (primary key, auto-generated)
    portfolio_id: String (required, max_length: 100)
    attribution_level: String (required, max_length: 50)
    method: String (required, max_length: 50)
    benchmark: String (max_length: 100)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}
```

### TimescaleDB (Query Side)
```pseudo
Table performance_attribution_ts {
    timestamp: Timestamp (required, partition_key)
    portfolio_id: String (required, max_length: 100)
    attribution_level: String (required, max_length: 50)
    total_return: Float
    benchmark_return: Float
    attribution_breakdown: JSON

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
}
```

## Implementation Estimation

### Priority: **HIGH** (Critical for performance tracking)
### Estimated Time: **4-5 weeks**

#### Week 1-2: Core Attribution Engine
- Python service with QuantLib integration
- Basic Brinson attribution methods
- Performance metric calculations
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3-5: Advanced Features & Integration
- Factor attribution implementation
- Multi-level attribution aggregation
- Integration with portfolio data
- **Effort**: 2 developers × 3 weeks = 6 dev-weeks

### Total Effort: **10 dev-weeks**
### Team Size: **2 developers** (1 senior quantitative developer, 1 mid-level developer)
### Dependencies: Portfolio data, benchmark data, factor models

### Success Criteria:
- P99 attribution calculation < 2 seconds
- Attribution accuracy > 98%
- Support for multiple attribution methods
- Real-time attribution capability
