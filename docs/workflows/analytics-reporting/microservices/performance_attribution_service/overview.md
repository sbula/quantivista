# Performance Attribution Service

## Responsibility
Specialized performance attribution analysis for reporting workflows. Provides detailed factor attribution, sector attribution, and security selection analysis with comprehensive benchmarking and peer comparison capabilities.

## Technology Stack
- **Language**: Python + QuantLib + Polars + NumPy
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for custom optimization algorithms and advanced models
- **Libraries**: QuantLib, Polars, numpy, empyrical, pyfolio for performance analytics
- **Scaling**: Horizontal by attribution complexity
- **NFRs**: P99 attribution calculation < 5s, comprehensive factor coverage

## API Specification

### Core APIs
```pseudo
// Enumerations
enum AttributionMethod {
    BRINSON_FACHLER,
    BRINSON_HOOD_BEEBOWER,
    FACTOR_ATTRIBUTION,
    SECTOR_ATTRIBUTION,
    SECURITY_SELECTION
}

enum AttributionPeriod {
    DAILY,
    WEEKLY,
    MONTHLY,
    QUARTERLY,
    YEARLY,
    CUSTOM
}

// Data Models
struct AttributionRequest {
    portfolio_id: String
    benchmark_id: String
    attribution_method: AttributionMethod
    period: AttributionPeriod
    start_date: Date
    end_date: Date
    factor_model: Optional<String>
}

struct AttributionResult {
    portfolio_id: String
    attribution_method: AttributionMethod
    total_return: Float
    benchmark_return: Float
    active_return: Float
    attribution_breakdown: AttributionBreakdown
    factor_exposures: Map<String, Float>
}

struct AttributionBreakdown {
    allocation_effect: Float
    selection_effect: Float
    interaction_effect: Float
    currency_effect: Optional<Float>
    sector_attribution: Map<String, Float>
    factor_attribution: Map<String, Float>
}

// REST API Endpoints
POST /api/v1/attribution/calculate
    Request: AttributionRequest
    Response: AttributionResult

GET /api/v1/attribution/{portfolio_id}/history
    Parameters: start_date, end_date
    Response: List<AttributionResult>

GET /api/v1/attribution/benchmarks
    Response: List<BenchmarkInfo>
```

### Event Output
```pseudo
struct AttributionCalculatedEvent {
    event_id: String  // "uuid"
    timestamp: DateTime  // "2025-06-21T10:00:00.000Z"
    attribution_calculated: {
        portfolio_id: String  // "portfolio_001"
        attribution_method: AttributionMethod  // BRINSON_FACHLER
        total_return: Float  // 0.085
        benchmark_return: Float  // 0.072
        active_return: Float  // 0.013
        attribution_breakdown: {
            allocation_effect: Float  // 0.008
            selection_effect: Float  // 0.005
            sector_attribution: {
                technology: Float  // 0.004
                healthcare: Float  // 0.002
            }
        }
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table attribution_calculations {
    id: UUID (primary key, default: gen_random_uuid())
    portfolio_id: String (required, max_length: 100)
    benchmark_id: String (required, max_length: 100)
    attribution_method: String (required, max_length: 50)
    calculation_date: Date (required)
    attribution_results: JSON (required)
    created_at: Timestamp (default: NOW())
}

Table benchmark_definitions {
    id: UUID (primary key, default: gen_random_uuid())
    benchmark_id: String (required, unique, max_length: 100)
    benchmark_name: String (required, max_length: 200)
    benchmark_type: String (required, max_length: 50)
    composition: JSON
    data_source: String (max_length: 100)
    created_at: Timestamp (default: NOW())
}
```

## Implementation Estimation

### Priority: **HIGH** (Critical for performance reporting)
### Estimated Time: **4-5 weeks**

#### Week 1-2: Core Attribution Engine
- Attribution calculation algorithms
- Benchmark integration
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3-5: Advanced Features & Integration
- Factor attribution models
- Integration with analytics engine
- Performance optimization
- **Effort**: 2 developers × 3 weeks = 6 dev-weeks

### Total Effort: **10 dev-weeks**
### Team Size: **2 developers** (1 senior quantitative analyst, 1 developer)
### Dependencies: Analytics engine, benchmark data

### Success Criteria:
- P99 attribution calculation < 5 seconds
- Comprehensive factor coverage
- Multiple attribution methodologies
- Accurate benchmark comparisons
