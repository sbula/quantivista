# Position Sizing Service

## Responsibility
Optimal position sizing using quantitative methods including Kelly Criterion implementation, risk-adjusted position sizing, and portfolio impact assessment. Provides mathematical optimization for capital allocation using advanced ML and analytics frameworks.

## Technology Stack
- **Language**: Python + Polars + JAX + DuckDB for quantitative position optimization
- **Libraries**: Polars for DataFrame operations, JAX for mathematical optimization, DuckDB for analytical queries
- **Data Processing**: Polars for high-performance portfolio data manipulation
- **ML Models**: JAX for Kelly Criterion and mathematical optimization models
- **Analytics**: DuckDB for complex portfolio impact analysis and risk calculations
- **Scaling**: Horizontal by portfolio complexity, vertical for optimization-intensive calculations
- **NFRs**: P99 sizing latency < 300ms, throughput > 5K sizing calculations/sec

## API Specification

### Internal APIs

#### Position Sizing API
```pseudo
// Enumerations
enum SizingMethod {
    KELLY_CRITERION,
    FIXED_PERCENTAGE,
    VOLATILITY_ADJUSTED,
    RISK_PARITY,
    EQUAL_WEIGHT
}

enum RiskModel {
    HISTORICAL_VAR,
    MONTE_CARLO,
    PARAMETRIC,
    GARCH
}

// Data Models
struct PositionSizingRequest {
    trading_decision: TradingDecision
    portfolio_state: PortfolioState
    sizing_parameters: SizingParameters
    risk_constraints: RiskConstraints
}

struct SizingParameters {
    sizing_method: SizingMethod
    risk_model: RiskModel
    confidence_level: Float
    lookback_period: Integer
    max_position_size: Float
    min_position_size: Float
    volatility_target: Float
}

struct RiskConstraints {
    max_portfolio_risk: Float
    max_instrument_weight: Float
    max_sector_weight: Float
    correlation_limit: Float
    leverage_limit: Float
}

struct PositionSizingResponse {
    optimal_position: OptimalPosition
    sizing_analysis: SizingAnalysis
    risk_metrics: PositionRiskMetrics
    processing_time_ms: Float
}

struct OptimalPosition {
    position_id: String
    instrument_id: String
    recommended_size: Float
    recommended_weight: Float
    currency: String
    sizing_method_used: SizingMethod
    confidence_score: Float
}

struct SizingAnalysis {
    kelly_fraction: Float
    expected_return: Float
    expected_volatility: Float
    sharpe_ratio: Float
    portfolio_impact: PortfolioImpact
    optimization_details: OptimizationDetails
}

struct PortfolioImpact {
    portfolio_risk_change: Float
    correlation_impact: Float
    diversification_benefit: Float
    concentration_risk: Float
}
```

#### Kelly Criterion API
```pseudo
// Kelly Criterion specific models
struct KellyCalculationRequest {
    expected_return: Float
    win_probability: Float
    average_win: Float
    average_loss: Float
    portfolio_volatility: Float
    correlation_matrix: Matrix
}

struct KellyCalculationResponse {
    kelly_fraction: Float
    adjusted_kelly: Float
    confidence_interval: ConfidenceInterval
    sensitivity_analysis: SensitivityAnalysis
}

struct ConfidenceInterval {
    lower_bound: Float
    upper_bound: Float
    confidence_level: Float
}

// REST API Endpoints
POST /api/v1/calculate-position-size
    Request: PositionSizingRequest
    Response: PositionSizingResponse

POST /api/v1/kelly-criterion
    Request: KellyCalculationRequest
    Response: KellyCalculationResponse

GET /api/v1/health
    Response: HealthResponse

GET /api/v1/metrics
    Response: MetricsResponse
```

### Event Output

#### PositionSizedEvent
```pseudo
Event position_sized {
    event_id: String
    timestamp: DateTime
    optimal_position: OptimalPositionPayload
    sizing_analysis: SizingAnalysisPayload
}

struct OptimalPositionPayload {
    position_id: String
    decision_id: String
    instrument_id: String
    recommended_size: Float
    recommended_weight: Float
    currency: String
    sizing_method: String
    confidence_score: Float
}

struct SizingAnalysisPayload {
    kelly_fraction: Float
    expected_return: Float
    expected_volatility: Float
    sharpe_ratio: Float
    portfolio_risk_change: Float
    processing_time_ms: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-24T10:25:00.000Z",
    optimal_position: {
        position_id: "pos_20250624_001",
        decision_id: "dec_20250624_001",
        instrument_id: "AAPL",
        recommended_size: 10000.0,
        recommended_weight: 0.025,
        currency: "USD",
        sizing_method: "KELLY_CRITERION",
        confidence_score: 0.89
    },
    sizing_analysis: {
        kelly_fraction: 0.032,
        expected_return: 0.12,
        expected_volatility: 0.18,
        sharpe_ratio: 0.67,
        portfolio_risk_change: 0.002,
        processing_time_ms: 185.3
    }
}
```

## Data Model

### Core Entities
```pseudo
// Data Models
struct PositionSizingRule {
    rule_id: String
    rule_name: String
    sizing_method: SizingMethod
    risk_model: RiskModel
    parameters: Map<String, Float>
    constraints: RiskConstraints
    enabled: Boolean
}

struct HistoricalPerformance {
    instrument_id: String
    period_start: DateTime
    period_end: DateTime
    returns: List<Float>
    volatility: Float
    sharpe_ratio: Float
    max_drawdown: Float
}

struct CorrelationMatrix {
    matrix_id: String
    instruments: List<String>
    correlation_data: Matrix
    calculation_date: DateTime
    lookback_period: Integer
}
```

## Database Schema (CQRS Pattern)

### Command Side (PostgreSQL)
```pseudo
// Position sizing rules and configuration
Table position_sizing_rules {
    id: UUID (primary key, auto-generated)
    rule_name: String (required, unique, max_length: 100)
    sizing_method: String (required, max_length: 50)
    risk_model: String (required, max_length: 50)
    parameters: JSON (required)
    constraints: JSON (required)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
    updated_at: Timestamp (default: now)
}

// Historical performance data
Table historical_performance {
    id: UUID (primary key, auto-generated)
    instrument_id: String (required, max_length: 20)
    period_start: Date (required)
    period_end: Date (required)
    returns_data: JSON (required)
    volatility: Float (required)
    sharpe_ratio: Float
    max_drawdown: Float
    created_at: Timestamp (default: now)
}

// Correlation matrices
Table correlation_matrices {
    id: UUID (primary key, auto-generated)
    matrix_id: String (required, unique, max_length: 100)
    instruments: JSON (required)
    correlation_data: JSON (required)
    calculation_date: Date (required)
    lookback_period: Integer (required)
    created_at: Timestamp (default: now)
}

// Position sizing history
Table position_sizing_history {
    id: UUID (primary key, auto-generated)
    position_id: String (required, unique, max_length: 100)
    decision_id: String (required, max_length: 100)
    instrument_id: String (required, max_length: 20)
    recommended_size: Decimal (precision: 15, scale: 2)
    recommended_weight: Float
    sizing_method: String (required, max_length: 50)
    kelly_fraction: Float
    confidence_score: Float
    created_at: Timestamp (default: now)
}

// Indexes
idx_sizing_rules_method: (sizing_method, enabled)
idx_historical_performance_instrument: (instrument_id, period_end DESC)
idx_correlation_matrices_date: (calculation_date DESC)
idx_sizing_history_decision: (decision_id)
idx_sizing_history_instrument_time: (instrument_id, created_at DESC)
```

### Query Side (TimescaleDB)
```pseudo
// Position sizing metrics time series
Table position_sizing_metrics_ts {
    timestamp: Timestamp (required, partition_key)
    instrument_id: String (max_length: 20)
    portfolio_id: String (max_length: 50)
    
    // Sizing metrics
    recommended_size: Float
    recommended_weight: Float
    kelly_fraction: Float
    expected_return: Float
    expected_volatility: Float
    sharpe_ratio: Float
    
    // Risk metrics
    portfolio_risk_change: Float
    correlation_impact: Float
    concentration_risk: Float
    
    // Performance metrics
    calculations_per_second: Float
    avg_processing_time_ms: Float
    optimization_success_rate: Float
    
    created_at: Timestamp (default: now)
    
    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
    partition_dimension: portfolio_id (partitions: 8)
}

// Kelly criterion calculations time series
Table kelly_calculations_ts {
    timestamp: Timestamp (required, partition_key)
    calculation_id: String (required, max_length: 100)
    instrument_id: String (required, max_length: 20)
    kelly_fraction: Float (required)
    adjusted_kelly: Float
    expected_return: Float
    win_probability: Float
    confidence_interval_lower: Float
    confidence_interval_upper: Float
    processing_time_ms: Float
    
    created_at: Timestamp (default: now)
    
    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
}

// Indexes for fast queries
idx_sizing_metrics_portfolio_time: (portfolio_id, timestamp DESC)
idx_sizing_metrics_instrument_time: (instrument_id, timestamp DESC)
idx_kelly_calculations_instrument_time: (instrument_id, timestamp DESC)
```

### Redis Caching Strategy
```pseudo
Cache position_sizing_cache {
    // Active sizing rules
    "sizing_rules:active": List<PositionSizingRule> (TTL: 1h)
    
    // Correlation matrices
    "correlation_matrix:latest": CorrelationMatrix (TTL: 30m)
    
    // Historical performance
    "performance:{instrument_id}:latest": HistoricalPerformance (TTL: 1h)
    
    // Recent calculations
    "sizing:{instrument_id}:recent": List<OptimalPosition> (TTL: 30m)
    
    // Performance metrics
    "performance:current": SizingPerformanceMetrics (TTL: 5m)
    
    // Kelly calculations cache
    "kelly:{instrument_id}:latest": KellyCalculationResponse (TTL: 15m)
}
```

## Implementation Estimation

### Priority: **HIGH** (Critical for position optimization)
### Estimated Time: **5-6 weeks**

#### Week 1-2: Core Sizing Engine
- Python service setup with Polars, JAX, and DuckDB integration
- Kelly Criterion implementation using JAX
- Basic position sizing algorithms
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3-4: Advanced Optimization Features
- Risk-adjusted position sizing using DuckDB analytics
- Portfolio impact assessment with Polars
- Correlation-based sizing adjustments
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 5: Performance Optimization & Integration
- Mathematical optimization using JAX
- High-throughput calculation optimization
- Integration with Trading Decision Engine
- **Effort**: 1 senior developer × 1 week = 1 dev-week

#### Week 6: Testing & Validation
- Quantitative model validation
- Performance testing and monitoring
- Error handling and recovery
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **11 dev-weeks**
### Team Size: **2 developers** (1 senior Python/Quant developer, 1 mid-level developer)
### Dependencies:
- Trading Decision Engine Service operational
- Portfolio state data available
- Historical market data for backtesting
- Correlation matrices from Instrument Analysis workflow

### Risk Factors:
- **Medium**: Mathematical optimization complexity
- **Medium**: Performance requirements for real-time sizing
- **Low**: Technology stack maturity

### Success Criteria:
- Process 5K+ sizing calculations per second
- P99 sizing latency < 300ms
- 95% Kelly Criterion calculation accuracy
- Support for 5+ sizing methods simultaneously
- Optimal position sizes within 2% of theoretical optimum
