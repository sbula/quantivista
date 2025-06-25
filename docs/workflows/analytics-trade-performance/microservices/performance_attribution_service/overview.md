# Performance Attribution Service

## Responsibility
Strategy-level and execution-focused performance attribution analysis providing detailed breakdown of trading performance sources. Focuses on trading-specific attribution rather than portfolio-level business attribution.

## Technology Stack
- **Language**: Python + QuantLib + Polars + DuckDB for quantitative attribution analysis
- **Libraries**: QuantLib for financial calculations, Polars for DataFrame operations, DuckDB for complex queries
- **Data Processing**: Polars for high-performance attribution calculations
- **Analytics**: DuckDB for complex multi-factor attribution analysis
- **Scaling**: Horizontal by attribution complexity, vertical for computation-intensive analysis
- **NFRs**: P99 attribution analysis < 3s, comprehensive factor breakdown, accurate attribution

## API Specification

### Internal APIs

#### Performance Attribution API
```pseudo
// Enumerations
enum AttributionLevel {
    TRADE_LEVEL,
    STRATEGY_LEVEL,
    EXECUTION_LEVEL,
    TIMEFRAME_LEVEL,
    FACTOR_LEVEL
}

enum AttributionMethod {
    BRINSON_FACHLER,
    FACTOR_BASED,
    RISK_ADJUSTED,
    EXECUTION_BASED,
    TIMING_BASED
}

enum PerformanceFactor {
    SIGNAL_STRENGTH,
    EXECUTION_QUALITY,
    TIMING_ACCURACY,
    MARKET_CONDITIONS,
    STRATEGY_SELECTION,
    RISK_MANAGEMENT
}

// Data Models
struct AttributionRequest {
    analysis_scope: AttributionScope
    attribution_method: AttributionMethod
    attribution_level: AttributionLevel
    time_period: TimePeriod
    benchmark_comparison: Boolean
    factor_decomposition: Boolean
}

struct AttributionScope {
    strategy_ids: List<String>
    instrument_ids: List<String>
    trade_ids: List<String>
    portfolio_filter: String
}

struct AttributionResponse {
    attribution_id: String
    total_return: Float
    benchmark_return: Float
    excess_return: Float
    attribution_breakdown: AttributionBreakdown
    factor_attribution: FactorAttribution
    execution_attribution: ExecutionAttribution
    timing_attribution: TimingAttribution
}

struct AttributionBreakdown {
    strategy_attribution: Map<String, Float>
    execution_attribution: Map<String, Float>
    timing_attribution: Map<String, Float>
    factor_attribution: Map<PerformanceFactor, Float>
    residual_attribution: Float
}

struct FactorAttribution {
    signal_contribution: Float
    execution_contribution: Float
    timing_contribution: Float
    market_contribution: Float
    interaction_effects: Map<String, Float>
}

struct ExecutionAttribution {
    venue_selection_impact: Float
    algorithm_selection_impact: Float
    timing_impact: Float
    cost_impact: Float
    quality_impact: Float
}
```

#### Risk-Adjusted Attribution API
```pseudo
// Risk-adjusted attribution models
struct RiskAdjustedAttributionRequest {
    attribution_scope: AttributionScope
    risk_model: String
    risk_factors: List<String>
    confidence_level: Float
}

struct RiskAdjustedAttributionResponse {
    risk_adjusted_return: Float
    sharpe_attribution: SharpeAttribution
    information_ratio_attribution: InformationRatioAttribution
    risk_factor_attribution: Map<String, Float>
    risk_adjusted_breakdown: RiskAdjustedBreakdown
}

struct SharpeAttribution {
    return_component: Float
    volatility_component: Float
    correlation_component: Float
    sharpe_contribution: Float
}

// REST API Endpoints
POST /api/v1/attribution/analyze
    Request: AttributionRequest
    Response: AttributionResponse

POST /api/v1/attribution/risk-adjusted
    Request: RiskAdjustedAttributionRequest
    Response: RiskAdjustedAttributionResponse

GET /api/v1/attribution/strategy-comparison
    Parameters: strategy_ids, time_period
    Response: StrategyComparisonReport

GET /api/v1/attribution/factor-analysis
    Parameters: factors, time_period
    Response: FactorAnalysisReport
```

### Event Output

#### PerformanceAttributionEvent
```pseudo
Event performance_attribution_completed {
    event_id: String
    timestamp: DateTime
    attribution_analysis: AttributionAnalysisPayload
    factor_breakdown: FactorBreakdownPayload
}

struct AttributionAnalysisPayload {
    attribution_id: String
    analysis_scope: String
    total_return: Float
    benchmark_return: Float
    excess_return: Float
    attribution_method: String
    analysis_timestamp: DateTime
}

struct FactorBreakdownPayload {
    signal_contribution: Float
    execution_contribution: Float
    timing_contribution: Float
    market_contribution: Float
    strategy_contributions: Map<String, Float>
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-24T11:00:00.000Z",
    attribution_analysis: {
        attribution_id: "attr_20250624_001",
        analysis_scope: "STRATEGY_LEVEL",
        total_return: 0.0245,
        benchmark_return: 0.0198,
        excess_return: 0.0047,
        attribution_method: "FACTOR_BASED",
        analysis_timestamp: "2025-06-24T11:00:00.000Z"
    },
    factor_breakdown: {
        signal_contribution: 0.0032,
        execution_contribution: 0.0018,
        timing_contribution: 0.0012,
        market_contribution: -0.0008,
        strategy_contributions: {
            "MOMENTUM_STRATEGY": 0.0025,
            "MEAN_REVERSION_STRATEGY": 0.0015,
            "ARBITRAGE_STRATEGY": 0.0007
        }
    }
}
```

## Data Model

### Core Entities
```pseudo
// Data Models
struct AttributionModel {
    model_id: String
    model_name: String
    attribution_method: AttributionMethod
    factor_definitions: List<FactorDefinition>
    calculation_parameters: Map<String, Float>
    enabled: Boolean
}

struct FactorDefinition {
    factor_id: String
    factor_name: String
    factor_type: PerformanceFactor
    calculation_method: String
    data_sources: List<String>
    weight: Float
}

struct BenchmarkDefinition {
    benchmark_id: String
    benchmark_name: String
    benchmark_type: String
    calculation_method: String
    rebalancing_frequency: String
}
```

## Database Schema (CQRS Pattern)

### Command Side (PostgreSQL)
```pseudo
// Attribution models and configurations
Table attribution_models {
    id: UUID (primary key, auto-generated)
    model_name: String (required, unique, max_length: 100)
    attribution_method: String (required, max_length: 50)
    factor_definitions: JSON (required)
    calculation_parameters: JSON (required)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
    updated_at: Timestamp (default: now)
}

// Factor definitions
Table factor_definitions {
    id: UUID (primary key, auto-generated)
    factor_name: String (required, unique, max_length: 100)
    factor_type: String (required, max_length: 50)
    calculation_method: Text (required)
    data_sources: JSON (required)
    weight: Float (default: 1.0)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

// Benchmark definitions
Table benchmark_definitions {
    id: UUID (primary key, auto-generated)
    benchmark_name: String (required, unique, max_length: 100)
    benchmark_type: String (required, max_length: 50)
    calculation_method: Text (required)
    rebalancing_frequency: String (max_length: 20)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

// Attribution results
Table attribution_results {
    id: UUID (primary key, auto-generated)
    attribution_id: String (required, unique, max_length: 100)
    analysis_scope: JSON (required)
    total_return: Float (required)
    benchmark_return: Float
    excess_return: Float
    attribution_breakdown: JSON (required)
    factor_attribution: JSON
    execution_attribution: JSON
    analysis_timestamp: Timestamp (required)
    created_at: Timestamp (default: now)
}

// Indexes
idx_attribution_models_method: (attribution_method, enabled)
idx_factor_definitions_type: (factor_type, enabled)
idx_benchmark_definitions_type: (benchmark_type, enabled)
idx_attribution_results_timestamp: (analysis_timestamp DESC)
```

### Query Side (TimescaleDB)
```pseudo
// Performance attribution time series
Table performance_attribution_ts {
    timestamp: Timestamp (required, partition_key)
    attribution_id: String (required, max_length: 100)
    strategy_id: String (max_length: 50)
    
    // Return metrics
    total_return: Float (required)
    benchmark_return: Float
    excess_return: Float
    risk_adjusted_return: Float
    
    // Attribution breakdown
    signal_contribution: Float
    execution_contribution: Float
    timing_contribution: Float
    market_contribution: Float
    
    // Risk metrics
    volatility: Float
    sharpe_ratio: Float
    information_ratio: Float
    max_drawdown: Float
    
    // Analysis metadata
    attribution_method: String (max_length: 50)
    confidence_level: Float
    
    created_at: Timestamp (default: now)
    
    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
    partition_dimension: strategy_id (partitions: 8)
}

// Factor attribution time series
Table factor_attribution_ts {
    timestamp: Timestamp (required, partition_key)
    attribution_id: String (required, max_length: 100)
    factor_name: String (required, max_length: 100)
    factor_contribution: Float (required)
    factor_weight: Float
    factor_return: Float
    factor_volatility: Float
    
    created_at: Timestamp (default: now)
    
    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
}

// Indexes for fast queries
idx_attribution_strategy_time: (strategy_id, timestamp DESC)
idx_attribution_return_time: (excess_return DESC, timestamp DESC)
idx_factor_attribution_factor_time: (factor_name, timestamp DESC)
```

### Redis Caching Strategy
```pseudo
Cache attribution_cache {
    // Attribution models
    "attribution_models:active": List<AttributionModel> (TTL: 1h)
    
    // Factor definitions
    "factor_definitions:all": List<FactorDefinition> (TTL: 1h)
    
    // Benchmark definitions
    "benchmark_definitions:all": List<BenchmarkDefinition> (TTL: 2h)
    
    // Recent attribution results
    "attribution_results:{attribution_id}": AttributionResponse (TTL: 24h)
    
    // Strategy performance cache
    "strategy_performance:{strategy_id}:latest": StrategyPerformance (TTL: 30m)
    
    // Performance metrics
    "performance:current": AttributionServiceMetrics (TTL: 5m)
}
```

## Implementation Estimation

### Priority: **HIGH** (Critical for strategy optimization)
### Estimated Time: **5-6 weeks**

#### Week 1-2: Core Attribution Engine
- Python service setup with QuantLib, Polars, and DuckDB
- Multi-method attribution calculation framework
- Factor-based attribution implementation
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3-4: Advanced Attribution Features
- Risk-adjusted attribution using QuantLib
- Execution-focused attribution analysis
- Multi-level attribution decomposition
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 5: Performance Optimization & Integration
- High-performance attribution calculations
- Integration with strategy and execution data
- Attribution validation and accuracy testing
- **Effort**: 1 senior developer × 1 week = 1 dev-week

#### Week 6: Testing & Validation
- Attribution accuracy validation with known benchmarks
- Performance testing and optimization
- Integration testing with dependent services
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **11 dev-weeks**
### Team Size: **2 developers** (1 senior quantitative analyst, 1 performance specialist)
### Dependencies:
- Strategy performance data and trade execution data
- Market data for benchmark calculations
- Risk model data for risk-adjusted attribution

### Risk Factors:
- **Medium**: Attribution accuracy and validation complexity
- **Medium**: Multi-factor attribution calculation performance
- **Low**: Technology stack maturity

### Success Criteria:
- Process attribution analysis within 3 seconds P99
- Achieve >98% attribution accuracy vs known benchmarks
- Support 10+ attribution methods simultaneously
- Provide detailed factor breakdown with statistical significance
- Real-time attribution updates for active strategies
