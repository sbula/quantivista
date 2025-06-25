# Transaction Cost Analysis Service

## Responsibility
Comprehensive transaction cost analysis (TCA) and execution cost prediction. Analyzes completed trades to measure execution performance, provides detailed cost attribution, and generates predictive cost models for execution optimization.

## Technology Stack
- **Language**: Python + Polars + DuckDB for high-performance TCA processing
- **Libraries**: Polars for DataFrame operations, DuckDB for complex analytical queries
- **Data Processing**: Polars for high-performance cost calculations (10x faster than pandas)
- **Analytics**: DuckDB for complex cost attribution and benchmark analysis
- **ML Models**: JAX for predictive cost modeling and optimization
- **Scaling**: Horizontal by analysis complexity, vertical for ML-intensive computations
- **NFRs**: P99 TCA analysis < 2s, predictive cost accuracy >90%, comprehensive cost attribution

## API Specification

### Internal APIs

#### Transaction Cost Analysis API
```pseudo
// Enumerations
enum BenchmarkType {
    ARRIVAL_PRICE,
    VWAP,
    TWAP,
    CLOSE_PRICE,
    IMPLEMENTATION_SHORTFALL,
    CUSTOM_BENCHMARK
}

enum CostComponent {
    MARKET_IMPACT,
    TIMING_COST,
    SPREAD_COST,
    COMMISSION,
    FEES,
    OPPORTUNITY_COST,
    SLIPPAGE
}

enum AnalysisGranularity {
    TRADE_LEVEL,
    ORDER_LEVEL,
    FILL_LEVEL,
    PORTFOLIO_LEVEL,
    STRATEGY_LEVEL
}

// Data Models
struct TCARequest {
    trade_id: String
    benchmark_types: List<BenchmarkType>
    analysis_granularity: AnalysisGranularity
    include_predictions: Boolean
    attribution_depth: String
}

struct TCAResponse {
    trade_id: String
    total_cost_bps: Float
    cost_breakdown: Map<CostComponent, Float>
    benchmark_comparisons: List<BenchmarkComparison>
    market_impact_analysis: MarketImpactAnalysis
    execution_quality_score: Float
    cost_predictions: CostPredictions
    recommendations: List<String>
}

struct BenchmarkComparison {
    benchmark_type: BenchmarkType
    benchmark_price: Float
    execution_price: Float
    performance_bps: Float
    percentile_rank: Float
    statistical_significance: Float
}

struct MarketImpactAnalysis {
    temporary_impact_bps: Float
    permanent_impact_bps: Float
    impact_decay_profile: List<ImpactPoint>
    impact_attribution: Map<String, Float>
    liquidity_impact: Float
    volatility_impact: Float
}

struct CostPredictions {
    predicted_total_cost_bps: Float
    prediction_confidence: Float
    cost_range_bps: ConfidenceInterval
    optimal_execution_strategy: String
    expected_savings_bps: Float
}
```

#### Predictive Cost Modeling API
```pseudo
// Predictive modeling data models
struct CostPredictionRequest {
    instrument_id: String
    order_size: Float
    market_conditions: MarketConditions
    execution_strategy: String
    urgency_level: String
}

struct CostPredictionResponse {
    predicted_cost_bps: Float
    prediction_confidence: Float
    cost_components: Map<CostComponent, Float>
    optimal_strategy: OptimalStrategy
    sensitivity_analysis: SensitivityAnalysis
}

struct MarketConditions {
    volatility_level: Float
    liquidity_score: Float
    bid_ask_spread: Float
    order_book_imbalance: Float
    market_regime: String
    time_of_day: String
}

struct OptimalStrategy {
    recommended_algorithm: String
    recommended_parameters: Map<String, Float>
    expected_cost_reduction: Float
    execution_timeline: String
}

// REST API Endpoints
POST /api/v1/tca/analyze
    Request: TCARequest
    Response: TCAResponse

POST /api/v1/tca/predict-cost
    Request: CostPredictionRequest
    Response: CostPredictionResponse

GET /api/v1/tca/benchmarks/{instrument_id}
    Parameters: start_date, end_date
    Response: BenchmarkData

POST /api/v1/tca/batch-analyze
    Request: List<TCARequest>
    Response: List<TCAResponse>
```

### Event Output

#### TCACompletedEvent
```pseudo
Event tca_analysis_completed {
    event_id: String
    timestamp: DateTime
    tca_analysis: TCAAnalysisPayload
    cost_predictions: CostPredictionPayload
}

struct TCAAnalysisPayload {
    trade_id: String
    total_cost_bps: Float
    cost_breakdown: Map<String, Float>
    benchmark_performance: Map<String, Float>
    execution_quality_score: Float
    market_impact_bps: Float
    analysis_timestamp: DateTime
}

struct CostPredictionPayload {
    prediction_accuracy: Float
    model_version: String
    prediction_confidence: Float
    cost_savings_achieved: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-24T11:00:00.000Z",
    tca_analysis: {
        trade_id: "trade_20250624_001",
        total_cost_bps: 15.2,
        cost_breakdown: {
            "MARKET_IMPACT": 8.5,
            "SPREAD_COST": 4.2,
            "COMMISSION": 1.5,
            "TIMING_COST": 1.0
        },
        benchmark_performance: {
            "ARRIVAL_PRICE": -2.3,
            "VWAP": 1.8,
            "TWAP": 0.5
        },
        execution_quality_score: 0.82,
        market_impact_bps: 8.5,
        analysis_timestamp: "2025-06-24T11:00:00.000Z"
    },
    cost_predictions: {
        prediction_accuracy: 0.91,
        model_version: "v2.1",
        prediction_confidence: 0.88,
        cost_savings_achieved: 3.2
    }
}
```

## Data Model

### Core Entities
```pseudo
// Data Models
struct TCAConfiguration {
    config_id: String
    benchmark_settings: Map<BenchmarkType, BenchmarkConfig>
    cost_attribution_rules: List<AttributionRule>
    prediction_model_config: ModelConfig
    analysis_parameters: AnalysisParameters
}

struct BenchmarkConfig {
    calculation_method: String
    time_window: String
    volume_weighting: Boolean
    outlier_handling: String
    statistical_tests: List<String>
}

struct CostModel {
    model_id: String
    model_type: String
    feature_set: List<String>
    training_data_period: DateRange
    model_performance: ModelPerformance
    last_updated: DateTime
}
```

## Database Schema (CQRS Pattern)

### Command Side (PostgreSQL)
```pseudo
// TCA configurations
Table tca_configurations {
    id: UUID (primary key, auto-generated)
    config_name: String (required, unique, max_length: 100)
    benchmark_settings: JSON (required)
    cost_attribution_rules: JSON (required)
    prediction_model_config: JSON (required)
    analysis_parameters: JSON (required)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
    updated_at: Timestamp (default: now)
}

// Benchmark data
Table benchmark_data {
    id: UUID (primary key, auto-generated)
    instrument_id: String (required, max_length: 20)
    benchmark_type: String (required, max_length: 50)
    benchmark_date: Date (required)
    benchmark_price: Decimal (required, precision: 15, scale: 6)
    volume_data: JSON
    calculation_metadata: JSON
    created_at: Timestamp (default: now)
    
    // Constraints
    unique_instrument_benchmark_date: (instrument_id, benchmark_type, benchmark_date)
}

// Cost prediction models
Table cost_prediction_models {
    id: UUID (primary key, auto-generated)
    model_id: String (required, unique, max_length: 100)
    model_type: String (required, max_length: 50)
    feature_set: JSON (required)
    model_parameters: JSON (required)
    training_data_period: JSON (required)
    performance_metrics: JSON (required)
    model_artifact_location: String (required)
    active: Boolean (default: true)
    created_at: Timestamp (default: now)
}

// TCA analysis results
Table tca_analysis_results {
    id: UUID (primary key, auto-generated)
    analysis_id: String (required, unique, max_length: 100)
    trade_id: String (required, max_length: 100)
    total_cost_bps: Float (required)
    cost_breakdown: JSON (required)
    benchmark_comparisons: JSON (required)
    market_impact_analysis: JSON (required)
    execution_quality_score: Float
    analysis_timestamp: Timestamp (required)
    created_at: Timestamp (default: now)
}

// Indexes
idx_tca_configs_name: (config_name, enabled)
idx_benchmark_data_instrument_date: (instrument_id, benchmark_date DESC)
idx_cost_models_type_active: (model_type, active)
idx_tca_results_trade_time: (trade_id, analysis_timestamp DESC)
```

### Query Side (TimescaleDB)
```pseudo
// TCA metrics time series
Table tca_metrics_ts {
    timestamp: Timestamp (required, partition_key)
    trade_id: String (required, max_length: 100)
    instrument_id: String (required, max_length: 20)
    
    // Cost metrics
    total_cost_bps: Float (required)
    market_impact_bps: Float
    spread_cost_bps: Float
    commission_bps: Float
    timing_cost_bps: Float
    
    // Benchmark performance
    arrival_price_performance_bps: Float
    vwap_performance_bps: Float
    twap_performance_bps: Float
    
    // Quality metrics
    execution_quality_score: Float
    prediction_accuracy: Float
    cost_savings_bps: Float
    
    // Market context
    volatility_level: Float
    liquidity_score: Float
    market_regime: String (max_length: 20)
    
    created_at: Timestamp (default: now)
    
    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
    partition_dimension: instrument_id (partitions: 16)
}

// Cost prediction performance
Table cost_prediction_performance_ts {
    timestamp: Timestamp (required, partition_key)
    model_id: String (required, max_length: 100)
    prediction_accuracy: Float (required)
    mean_absolute_error: Float
    root_mean_square_error: Float
    prediction_bias: Float
    confidence_calibration: Float
    model_drift_score: Float
    
    created_at: Timestamp (default: now)
    
    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 week)
}

// Indexes for fast queries
idx_tca_metrics_instrument_time: (instrument_id, timestamp DESC)
idx_tca_metrics_cost_time: (total_cost_bps DESC, timestamp DESC)
idx_prediction_performance_model_time: (model_id, timestamp DESC)
```

### Redis Caching Strategy
```pseudo
Cache tca_cache {
    // Active configurations
    "tca_config:active": TCAConfiguration (TTL: 1h)
    
    // Recent benchmark data
    "benchmarks:{instrument_id}:latest": Map<BenchmarkType, Float> (TTL: 30m)
    
    // Cost prediction models
    "cost_models:active": List<CostModel> (TTL: 2h)
    
    // Recent TCA results
    "tca_results:{trade_id}": TCAResponse (TTL: 24h)
    
    // Performance metrics
    "performance:current": TCAPerformanceMetrics (TTL: 5m)
    
    // Market conditions cache
    "market_conditions:current": MarketConditions (TTL: 1m)
}
```

## Implementation Estimation

### Priority: **HIGH** (Core analytics capability)
### Estimated Time: **5-6 weeks**

#### Week 1-2: Core TCA Engine
- Python service setup with Polars and DuckDB integration
- Multi-benchmark TCA implementation
- Cost component breakdown and attribution
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3-4: Predictive Cost Modeling
- ML-based cost prediction using JAX
- Market impact modeling and forecasting
- Execution strategy optimization
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 5: Performance Optimization & Integration
- High-performance TCA processing optimization
- Integration with trade execution and market data
- Real-time cost prediction capabilities
- **Effort**: 1 senior developer × 1 week = 1 dev-week

#### Week 6: Advanced Analytics & Testing
- Advanced attribution analysis
- Statistical significance testing
- Comprehensive testing and validation
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **11 dev-weeks**
### Team Size: **2 developers** (1 senior quantitative developer, 1 ML engineer)
### Dependencies:
- Trade execution data and settlement confirmations
- Market data for benchmark calculations
- Historical execution data for model training

### Risk Factors:
- **Medium**: Predictive model accuracy requirements
- **Medium**: Real-time cost prediction performance
- **Low**: Technology stack maturity

### Success Criteria:
- Process TCA analysis within 2 seconds P99
- Achieve >90% cost prediction accuracy
- Support 10+ benchmark types simultaneously
- Provide actionable optimization recommendations
- Generate cost savings of 5+ bps on average
