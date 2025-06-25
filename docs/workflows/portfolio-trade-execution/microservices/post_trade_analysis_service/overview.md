# Post-Trade Analysis Service

## Responsibility
Comprehensive transaction cost analysis (TCA) and execution quality measurement. Analyzes completed trades to measure execution performance, identify optimization opportunities, and provide detailed cost attribution analysis.

## Technology Stack
- **Language**: Python + NumPy + financial analytics libraries
- **Libraries**: numpy, scipy, financial analytics, visualization libraries
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for custom optimization algorithms and advanced models
- **Scaling**: Horizontal by analysis complexity
- **NFRs**: P99 analysis < 3s, comprehensive TCA metrics, accurate cost attribution

## API Specification

### Core APIs
```pseudo
// Enumerations
enum AnalysisType {
    TRANSACTION_COST_ANALYSIS,
    EXECUTION_QUALITY,
    MARKET_IMPACT_ANALYSIS,
    VENUE_PERFORMANCE,
    ALGORITHM_PERFORMANCE,
    COST_ATTRIBUTION
}

enum BenchmarkType {
    ARRIVAL_PRICE,
    VWAP,
    TWAP,
    CLOSE_PRICE,
    IMPLEMENTATION_SHORTFALL
}

enum CostComponent {
    MARKET_IMPACT,
    TIMING_COST,
    SPREAD_COST,
    COMMISSION,
    FEES,
    OPPORTUNITY_COST
}

// Data Models
struct PostTradeAnalysisRequest {
    trade_id: String
    analysis_types: List<AnalysisType>
    benchmark_type: BenchmarkType
    analysis_period: DateRange
    include_attribution: Boolean
}

struct PostTradeAnalysisResponse {
    trade_id: String
    analysis_results: Map<AnalysisType, AnalysisResult>
    overall_performance: OverallPerformance
    recommendations: List<String>
    analysis_time_ms: Float
}

struct TransactionCostAnalysis {
    total_cost_bps: Float
    cost_breakdown: Map<CostComponent, Float>
    benchmark_comparison: BenchmarkComparison
    market_impact: MarketImpactAnalysis
    execution_shortfall: Float
    cost_attribution: CostAttribution
}

struct ExecutionQualityMetrics {
    fill_rate: Float
    average_fill_time: Float
    price_improvement: Float
    venue_performance: Map<String, VenuePerformance>
    algorithm_efficiency: Float
    slippage: Float
}

struct MarketImpactAnalysis {
    temporary_impact: Float
    permanent_impact: Float
    impact_decay_profile: List<ImpactPoint>
    impact_attribution: Map<String, Float>
}

struct BenchmarkComparison {
    benchmark_type: BenchmarkType
    benchmark_price: Float
    execution_price: Float
    performance_bps: Float
    percentile_rank: Float
}

// REST API Endpoints
POST /api/v1/post-trade/analyze
    Request: PostTradeAnalysisRequest
    Response: PostTradeAnalysisResponse

GET /api/v1/post-trade/{trade_id}/tca
    Response: TransactionCostAnalysis

GET /api/v1/post-trade/performance/summary
    Parameters: start_date, end_date, portfolio_id
    Response: PerformanceSummary

POST /api/v1/post-trade/batch-analyze
    Request: List<PostTradeAnalysisRequest>
    Response: List<PostTradeAnalysisResponse>
```

### Event Output
```pseudo
Event post_trade_analysis_completed {
    event_id: String
    timestamp: DateTime
    post_trade_analysis: PostTradeAnalysisData
}

struct PostTradeAnalysisData {
    trade_id: String
    analysis_results: AnalysisResultsData
    overall_performance: OverallPerformanceData
    recommendations: List<String>
}

struct AnalysisResultsData {
    transaction_cost_analysis: TransactionCostAnalysisData
    execution_quality: ExecutionQualityData
}

struct TransactionCostAnalysisData {
    total_cost_bps: Float
    cost_breakdown: CostBreakdownData
    benchmark_comparison: BenchmarkComparisonData
}

struct CostBreakdownData {
    market_impact: Float
    spread_cost: Float
    commission: Float
}

struct BenchmarkComparisonData {
    benchmark_type: String
    performance_bps: Float
    percentile_rank: Float
}

struct ExecutionQualityData {
    fill_rate: Float
    average_fill_time: Float
    price_improvement: Float
    slippage: Float
}

struct OverallPerformanceData {
    execution_score: Float
    cost_efficiency: Float
    speed_score: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    post_trade_analysis: {
        trade_id: "trade_20250621_001",
        analysis_results: {
            transaction_cost_analysis: {
                total_cost_bps: 12.5,
                cost_breakdown: {
                    market_impact: 8.2,
                    spread_cost: 3.1,
                    commission: 1.2
                },
                benchmark_comparison: {
                    benchmark_type: "ARRIVAL_PRICE",
                    performance_bps: -2.3,
                    percentile_rank: 0.75
                }
            },
            execution_quality: {
                fill_rate: 0.98,
                average_fill_time: 245.6,
                price_improvement: 1.8,
                slippage: 0.15
            }
        },
        overall_performance: {
            execution_score: 0.82,
            cost_efficiency: 0.78,
            speed_score: 0.85
        },
        recommendations: [
            "Consider using more dark pool allocation for similar orders",
            "Execution timing could be optimized during high volatility periods"
        ]
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table tca_configurations {
    id: UUID (primary key, auto-generated)
    config_name: String (required, max_length: 100)
    analysis_types: JSON (required)
    benchmark_settings: JSON (required)
    cost_attribution_rules: JSON (required)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table benchmark_data {
    id: UUID (primary key, auto-generated)
    instrument_id: String (required, max_length: 20)
    benchmark_type: String (required, max_length: 50)
    benchmark_date: Date (required)
    benchmark_price: Decimal (required, precision: 15, scale: 6)
    volume_data: JSON
    created_at: Timestamp (default: now)

    // Constraints
    unique_instrument_benchmark_date: (instrument_id, benchmark_type, benchmark_date)
}
```

### TimescaleDB (Query Side)
```pseudo
Table trade_analysis_ts {
    timestamp: Timestamp (required, partition_key)
    trade_id: String (required, max_length: 100)
    analysis_type: String (required, max_length: 50)
    total_cost_bps: Float
    market_impact_bps: Float
    execution_score: Float
    benchmark_performance_bps: Float
    venue_performance: JSON
    cost_attribution: JSON

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
    partition_dimension: trade_id (partitions: 8)
}

Table execution_performance_ts {
    timestamp: Timestamp (required, partition_key)
    portfolio_id: String (max_length: 100)
    instrument_id: String (required, max_length: 20)
    avg_cost_bps: Float
    avg_execution_score: Float
    fill_rate: Float
    avg_slippage: Float
    trade_count: Integer

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
}
```

## Implementation Estimation

### Priority: **MEDIUM** (Important for optimization)
### Estimated Time: **5-6 weeks**

#### Week 1-2: Core TCA Engine
- Python service setup with financial analytics
- Basic transaction cost analysis
- Benchmark comparison framework
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3-4: Advanced Analysis Features
- Market impact analysis and attribution
- Execution quality metrics
- Performance benchmarking
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 5-6: Integration & Reporting
- Integration with trade data
- Performance reporting and visualization
- Analysis optimization and caching
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **12 dev-weeks**
### Team Size: **2 developers** (1 senior quantitative developer, 1 financial analyst)
### Dependencies: Trade data, market data, benchmark data

### Success Criteria:
- P99 analysis latency < 3 seconds
- Comprehensive TCA metrics coverage
- Accurate cost attribution > 95%
- Support for multiple benchmark types
- Actionable optimization recommendations
