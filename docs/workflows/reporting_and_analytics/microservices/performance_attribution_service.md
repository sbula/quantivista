# Performance Attribution Service

## Responsibility
Specialized performance attribution analysis for reporting workflows. Provides detailed factor attribution, sector attribution, and security selection analysis with comprehensive benchmarking and peer comparison capabilities.

## Technology Stack
- **Language**: Python + QuantLib + pandas + NumPy
- **Libraries**: QuantLib, pandas, numpy, empyrical, pyfolio for performance analytics
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
```json
{
  "eventId": "uuid",
  "timestamp": "2025-06-21T10:00:00.000Z",
  "attribution_calculated": {
    "portfolio_id": "portfolio_001",
    "attribution_method": "BRINSON_FACHLER",
    "total_return": 0.085,
    "benchmark_return": 0.072,
    "active_return": 0.013,
    "attribution_breakdown": {
      "allocation_effect": 0.008,
      "selection_effect": 0.005,
      "sector_attribution": {
        "technology": 0.004,
        "healthcare": 0.002
      }
    }
  }
}
```

## Database Schema

### PostgreSQL (Command Side)
```sql
CREATE TABLE attribution_calculations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    portfolio_id VARCHAR(100) NOT NULL,
    benchmark_id VARCHAR(100) NOT NULL,
    attribution_method VARCHAR(50) NOT NULL,
    calculation_date DATE NOT NULL,
    attribution_results JSONB NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE benchmark_definitions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    benchmark_id VARCHAR(100) NOT NULL UNIQUE,
    benchmark_name VARCHAR(200) NOT NULL,
    benchmark_type VARCHAR(50) NOT NULL,
    composition JSONB,
    data_source VARCHAR(100),
    created_at TIMESTAMPTZ DEFAULT NOW()
);
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
