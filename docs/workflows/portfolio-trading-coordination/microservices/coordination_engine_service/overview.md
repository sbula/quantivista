# Coordination Engine Service

## Responsibility
Core coordination logic matching trading signals with portfolio needs and constraints. Orchestrates the entire coordination process, manages signal-portfolio alignment, and ensures optimal decision-making across multiple signals and portfolio requirements.

## Technology Stack
- **Primary Language**: Go
- **Data Processing**: Polars (high-performance DataFrames)
- **Analytics Database**: DuckDB (analytical queries)
- **Machine Learning**: JAX (optimization and ML computations)
- **API Framework**: Gin (Go web framework)
- **Message Queue**: Apache Pulsar
- **Caching**: Redis
- **Monitoring**: Prometheus + Grafana
- **Containerization**: Docker

## Core Responsibilities

### Signal Synthesis and Decision Generation
- Aggregate and synthesize signals from Signal Aggregation Service
- Weight signals based on source reliability and market conditions
- Generate composite signal scores and decision recommendations
- Apply portfolio-level constraints to signal processing
- Create coordinated trading decisions from multiple signal sources

### Portfolio Impact Assessment
- Evaluate potential impact of trading decisions on portfolio using DuckDB
- Assess risk implications of proposed trades
- Calculate expected portfolio changes and metrics
- Evaluate correlation effects and portfolio balance
- Estimate transaction costs and market impact

### Rebalancing Integration
- Process and prioritize rebalancing requests from Portfolio Management
- Integrate rebalancing requirements with trading signals
- Optimize rebalancing timing and coordination
- Perform cost-benefit analysis for rebalancing decisions
- Coordinate tax-efficient rebalancing strategies

### Coordination Analytics
- Measure coordination effectiveness and decision quality
- Perform signal-to-outcome attribution analysis
- Track decision accuracy and portfolio impact
- Generate coordination performance metrics
- Optimize coordination algorithms based on outcomes

### Execution Coordination
- Coordinate timing of trade execution across instruments
- Manage execution priorities and sequencing
- Handle partial fills and execution adjustments
- Coordinate with market conditions and liquidity
- Optimize execution timing for portfolio efficiency

## API Specification

### Core APIs
```pseudo
// Enumerations
enum CoordinationMode {
    AGGRESSIVE,
    BALANCED,
    CONSERVATIVE
}

enum TradingAction {
    BUY,
    SELL,
    HOLD
}

// Data Models
struct CoordinationRequest {
    signals: List<TradingSignal>
    portfolio_state: PortfolioState
    rebalance_requests: Optional<List<RebalanceRequest>>
    coordination_parameters: CoordinationParameters
}

struct CoordinationParameters {
    max_position_size: Float  // 5% max position
    correlation_threshold: Float
    risk_budget_limit: Float  // 2% portfolio risk
    priority_weights: Map<String, Float>
    coordination_mode: CoordinationMode
}

struct CoordinatedTradingDecision {
    decision_id: String
    instrument_id: String
    action: TradingAction
    quantity: Float
    priority: Integer
    confidence: Float
    coordination_reasoning: List<String>
    portfolio_impact: PortfolioImpact
    risk_assessment: RiskAssessment
    timing_recommendation: TimingRecommendation
}

class PortfolioImpact(BaseModel):
    position_change: float
    weight_change: float
    risk_contribution_change: float
    correlation_impact: float
    diversification_effect: float

class CoordinationResponse(BaseModel):
    coordination_id: str
    decisions: List[CoordinatedTradingDecision]
    coordination_summary: CoordinationSummary
    rejected_signals: List[RejectedSignal]
    processing_time_ms: float

class CoordinationSummary(BaseModel):
    total_signals_processed: int
    decisions_generated: int
    signals_rejected: int
    portfolio_utilization: float
    risk_budget_used: float
    coordination_quality: float

@app.post("/coordinate")
async def coordinate_trading_decisions(request: CoordinationRequest) -> CoordinationResponse:
    pass

@app.post("/batch-coordinate")
async def batch_coordinate_decisions(requests: List[CoordinationRequest]) -> List[CoordinationResponse]:
    pass

@app.get("/coordination/{coordination_id}")
async def get_coordination_details(coordination_id: str) -> CoordinationResponse:
    pass
```

### Event Output
```pseudo
Event coordination_completed {
    event_id: String
    timestamp: DateTime
    coordination_completed: CoordinationCompletedData
}

struct CoordinationCompletedData {
    coordination_id: String
    decisions: List<DecisionData>
    coordination_summary: CoordinationSummaryData
}

struct DecisionData {
    decision_id: String
    instrument_id: String
    action: String
    quantity: Integer
    priority: Integer
    confidence: Float
    coordination_reasoning: List<String>
    portfolio_impact: PortfolioImpactData
}

struct PortfolioImpactData {
    position_change: Float
    weight_change: Float
    risk_contribution_change: Float
    correlation_impact: Float
}

struct CoordinationSummaryData {
    total_signals_processed: Integer
    decisions_generated: Integer
    signals_rejected: Integer
    portfolio_utilization: Float
    risk_budget_used: Float
    coordination_quality: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    coordination_completed: {
        coordination_id: "coord_20250621_001",
        decisions: [
            {
                decision_id: "decision_001",
                instrument_id: "AAPL",
                action: "buy",
                quantity: 100,
                priority: 1,
                confidence: 0.85,
                coordination_reasoning: [
                    "Strong signal with low portfolio correlation",
                    "Within risk budget limits",
                    "Aligns with portfolio diversification goals"
                ],
                portfolio_impact: {
                    position_change: 0.02,
                    weight_change: 0.015,
                    risk_contribution_change: 0.008,
                    correlation_impact: 0.12
                }
            }
        ],
        coordination_summary: {
            total_signals_processed: 5,
            decisions_generated: 3,
            signals_rejected: 2,
            portfolio_utilization: 0.85,
            risk_budget_used: 0.75,
            coordination_quality: 0.88
        }
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table coordination_configurations {
    id: UUID (primary key, auto-generated)
    config_name: String (required, max_length: 100)
    coordination_parameters: JSON (required)
    priority_weights: JSON (required)
    risk_limits: JSON (required)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table coordination_sessions {
    id: UUID (primary key, auto-generated)
    coordination_id: String (required, unique, max_length: 100)
    signals_processed: Integer
    decisions_generated: Integer
    coordination_quality: Float
    processing_time_ms: Float
    created_at: Timestamp (default: now)
}

Table coordination_decisions {
    id: UUID (primary key, auto-generated)
    coordination_id: String (required, max_length: 100)
    decision_id: String (required, max_length: 100)
    instrument_id: String (required, max_length: 20)
    action: String (required, max_length: 10)
    quantity: Float (required)
    priority: Integer
    confidence: Float
    reasoning: JSON
    portfolio_impact: JSON
    created_at: Timestamp (default: now)
}
```

### TimescaleDB (Query Side)
```pseudo
Table coordination_decisions_ts {
    timestamp: Timestamp (required, partition_key)
    coordination_id: String (required, max_length: 100)
    decision_id: String (required, max_length: 100)
    instrument_id: String (required, max_length: 20)
    action: String (required, max_length: 10)
    quantity: Float (required)
    confidence: Float
    portfolio_impact: JSON
    coordination_metadata: JSON

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
    partition_dimension: instrument_id (partitions: 8)
}

Table coordination_performance_ts {
    timestamp: Timestamp (required, partition_key)
    coordination_quality: Float
    avg_processing_time_ms: Float
    decisions_per_minute: Float
    portfolio_utilization: Float
    risk_budget_utilization: Float

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 hour)
}
```

### Redis Caching
```pseudo
Cache coordination_cache {
    // Current portfolio state
    "portfolio_state:current": PortfolioState (TTL: 1m)

    // Coordination parameters
    "coord_params:{config_name}": CoordinationParameters (TTL: 30m)

    // Recent decisions
    "recent_decisions:{instrument_id}": List<CoordinatedTradingDecision> (TTL: 5m)

    // Correlation matrix
    "correlation_matrix:current": CorrelationMatrix (TTL: 15m)

    // Risk metrics
    "risk_metrics:{instrument_id}": RiskMetrics (TTL: 10m)
}
```

## Implementation Estimation

### Priority: **CRITICAL** (Core coordination functionality)
### Estimated Time: **5-6 weeks**

#### Week 1-2: Core Coordination Engine
- Python service setup with FastAPI
- Basic signal-portfolio matching algorithms
- Portfolio state integration and management
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3-4: Advanced Coordination Logic
- Multi-signal prioritization and optimization
- Portfolio impact calculation and assessment
- Risk-aware coordination algorithms
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 5: Optimization & Performance
- Coordination quality scoring and validation
- Performance optimization for real-time processing
- Advanced coordination strategies
- **Effort**: 1 senior developer × 1 week = 1 dev-week

#### Week 6: Integration & Testing
- Integration with all upstream and downstream services
- Performance testing and optimization
- Coordination effectiveness validation
- **Effort**: 2 developers × 1 week = 2 dev-weeks

### Total Effort: **11 dev-weeks**
### Team Size: **2 developers** (1 senior Python developer, 1 mid-level developer)
### Dependencies: Trading Decision workflow, Portfolio Management workflow, correlation data

### Success Criteria:
- P99 coordination latency < 200ms
- Optimal signal-portfolio matching accuracy > 90%
- Support for 100+ signals simultaneously
- Portfolio constraint compliance 100%
- Real-time coordination capability
