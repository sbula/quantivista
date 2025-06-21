# Portfolio Management Workflow

## Overview
The Portfolio Management Workflow is responsible for portfolio-level strategy optimization, performance attribution, and rebalancing trigger generation. This workflow focuses on high-level portfolio strategy, risk budgeting, and determining when portfolio adjustments are needed, working in coordination with the Portfolio Trading Coordination Workflow to implement changes.

## Key Challenges Addressed
- **Multi-Strategy Portfolio Optimization**: Optimizing allocation across different trading strategies
- **Risk Budget Management**: Allocating risk budgets across strategies and asset classes
- **Performance Attribution**: Analyzing returns and attributing performance to different factors
- **Rebalancing Trigger Logic**: Determining when portfolio adjustments are needed
- **Strategy Coordination**: Managing interactions between multiple trading strategies
- **Benchmark Tracking**: Maintaining alignment with investment objectives and benchmarks

## Core Responsibilities
- **Portfolio Strategy Optimization**: Long-term strategic asset allocation and strategy weighting
- **Risk Budget Allocation**: Distributing risk budgets across strategies and asset classes
- **Performance Attribution**: Analyzing portfolio performance and identifying sources of returns
- **Rebalancing Trigger Generation**: Determining when and how portfolio should be rebalanced
- **Strategy Coordination**: Managing multiple trading strategies within portfolio constraints
- **Compliance Monitoring**: Ensuring adherence to investment mandates and regulatory requirements

## NOT This Workflow's Responsibilities
- **Individual Trading Decisions**: Making specific buy/sell decisions (belongs to Trading Decision Workflow)
- **Position Sizing**: Calculating specific trade amounts (belongs to Portfolio Trading Coordination Workflow)
- **Order Execution**: Actual trade execution (belongs to Trade Execution Workflow)
- **Signal Generation**: Generating trading signals (belongs to Trading Decision Workflow)
- **Technical Analysis**: Computing indicators (belongs to Instrument Analysis Workflow)

## Workflow Sequence

### 1. Portfolio Strategy Optimization
**Responsibility**: Strategy Optimization Service

#### Multi-Strategy Portfolio Optimization
```python
class PortfolioStrategyOptimizer:
    def __init__(self):
        self.optimization_objectives = {
            'return_maximization': 0.4,
            'risk_minimization': 0.3,
            'diversification': 0.2,
            'cost_minimization': 0.1
        }
        self.rebalancing_thresholds = {
            'strategy_weight_deviation': 0.05,  # 5% deviation triggers rebalance
            'risk_budget_breach': 0.10,  # 10% risk budget breach
            'performance_divergence': 0.15  # 15% performance divergence
        }

    async def optimize_portfolio_strategy(
        self,
        current_portfolio: PortfolioState,
        strategy_performance: Dict[str, StrategyPerformance],
        market_conditions: MarketConditions,
        correlation_data: CorrelationMatrix
    ) -> PortfolioOptimizationResult:
        """Optimize portfolio strategy allocation and risk budgets"""

        # Analyze current strategy performance
        strategy_analysis = self.analyze_strategy_performance(strategy_performance)

        # Assess market regime and adjust strategy weights
        market_regime = await self.detect_market_regime(market_conditions)
        regime_adjustments = self.get_regime_based_adjustments(market_regime)

        # Optimize strategy allocation using modern portfolio theory
        optimal_weights = await self.optimize_strategy_weights(
            strategy_analysis, regime_adjustments, correlation_data
        )

        # Allocate risk budgets across strategies
        risk_budgets = self.allocate_risk_budgets(optimal_weights, current_portfolio)

        # Check if rebalancing is needed
        rebalancing_needed = self.assess_rebalancing_need(
            current_portfolio, optimal_weights, risk_budgets
        )

        return PortfolioOptimizationResult(
            optimal_strategy_weights=optimal_weights,
            risk_budget_allocation=risk_budgets,
            rebalancing_needed=rebalancing_needed,
            optimization_reasoning=self.generate_optimization_reasoning(
                strategy_analysis, market_regime, optimal_weights
            )
        )
```

### 2. Performance Attribution Analysis
**Responsibility**: Performance Attribution Service

#### Multi-Level Performance Analysis
```python
class PerformanceAttributionAnalyzer:
    def __init__(self):
        self.attribution_levels = ['portfolio', 'strategy', 'sector', 'instrument']

    async def analyze_portfolio_performance(
        self,
        portfolio_returns: PortfolioReturns,
        benchmark_returns: BenchmarkReturns,
        strategy_returns: Dict[str, StrategyReturns]
    ) -> PerformanceAttributionResult:
        """Comprehensive performance attribution analysis"""

        # Portfolio-level attribution
        portfolio_attribution = self.calculate_portfolio_attribution(
            portfolio_returns, benchmark_returns
        )

        # Strategy-level attribution
        strategy_attribution = {}
        for strategy_id, returns in strategy_returns.items():
            strategy_attribution[strategy_id] = self.calculate_strategy_attribution(
                returns, portfolio_returns, benchmark_returns
            )

        # Risk-adjusted performance metrics
        risk_metrics = self.calculate_risk_adjusted_metrics(
            portfolio_returns, benchmark_returns
        )

        # Factor attribution (if factor models available)
        factor_attribution = await self.calculate_factor_attribution(
            portfolio_returns, benchmark_returns
        )

        return PerformanceAttributionResult(
            portfolio_attribution=portfolio_attribution,
            strategy_attribution=strategy_attribution,
            risk_adjusted_metrics=risk_metrics,
            factor_attribution=factor_attribution,
            performance_summary=self.generate_performance_summary(
                portfolio_attribution, strategy_attribution, risk_metrics
            )
        )
```

### 3. Rebalancing Trigger Generation
**Responsibility**: Rebalancing Engine Service

#### Intelligent Rebalancing Logic
```python
class RebalancingTriggerEngine:
    def __init__(self):
        self.trigger_conditions = {
            'time_based': {'frequency': 'monthly', 'day_of_month': 1},
            'threshold_based': {
                'weight_deviation': 0.05,
                'risk_budget_breach': 0.10,
                'performance_divergence': 0.15
            },
            'volatility_based': {'volatility_spike': 0.25},
            'correlation_based': {'correlation_regime_change': 0.20}
        }

    async def evaluate_rebalancing_triggers(
        self,
        current_portfolio: PortfolioState,
        target_allocation: PortfolioOptimizationResult,
        market_conditions: MarketConditions,
        correlation_data: CorrelationMatrix
    ) -> List[RebalanceRequestEvent]:
        """Evaluate various rebalancing triggers and generate requests"""

        rebalance_requests = []

        # Time-based rebalancing
        if self.is_time_based_rebalance_due():
            rebalance_requests.append(self.create_time_based_rebalance_request(
                current_portfolio, target_allocation
            ))

        # Threshold-based rebalancing
        weight_deviations = self.calculate_weight_deviations(
            current_portfolio, target_allocation
        )
        if any(abs(dev) > self.trigger_conditions['threshold_based']['weight_deviation']
               for dev in weight_deviations.values()):
            rebalance_requests.append(self.create_threshold_based_rebalance_request(
                current_portfolio, target_allocation, weight_deviations
            ))

        # Risk-based rebalancing
        risk_budget_breaches = self.assess_risk_budget_breaches(
            current_portfolio, target_allocation
        )
        if risk_budget_breaches:
            rebalance_requests.append(self.create_risk_based_rebalance_request(
                current_portfolio, risk_budget_breaches
            ))

        # Market condition-based rebalancing
        if self.is_market_condition_rebalance_needed(market_conditions):
            rebalance_requests.append(self.create_market_condition_rebalance_request(
                current_portfolio, market_conditions
            ))

        return rebalance_requests
```

### 4. Risk Budget Management
**Responsibility**: Risk Budget Service

#### Dynamic Risk Budget Allocation
- **Strategy Risk Budgets**: Allocate risk budgets across different trading strategies
- **Sector Risk Budgets**: Manage sector exposure limits and risk contributions
- **Factor Risk Budgets**: Control exposure to systematic risk factors
- **Tail Risk Management**: Monitor and control extreme risk scenarios
- **Correlation Risk Budgets**: Manage portfolio correlation exposure

### 5. Strategy Coordination and Monitoring
**Responsibility**: Strategy Coordination Service

#### Multi-Strategy Management
- **Strategy Performance Monitoring**: Track individual strategy performance and risk metrics
- **Strategy Interaction Analysis**: Analyze correlations and interactions between strategies
- **Strategy Capacity Management**: Monitor strategy capacity and scalability limits
- **Strategy Risk Monitoring**: Ensure strategies stay within allocated risk budgets
- **Strategy Lifecycle Management**: Handle strategy deployment, scaling, and retirement

### 6. Event-Driven Portfolio Management
**Responsibility**: Portfolio Management Distribution Service
- **Real-time streaming**: Apache Pulsar for immediate rebalancing triggers
- **Performance reporting**: Batch processing for comprehensive performance analysis
- **Strategy coordination**: Event-driven strategy management and monitoring
- **Risk monitoring**: Real-time risk budget tracking and alerts

## Event Contracts

### Events Consumed

#### `CoordinatedTradingDecisionEvent` (from Portfolio Trading Coordination Workflow)
```json
{
  "eventId": "uuid",
  "timestamp": "2025-06-21T10:30:00.300Z",
  "decision": {
    "instrument_id": "AAPL",
    "action": "BUY",
    "position_size": 0.03,
    "trade_amount": 30000.00
  },
  "portfolio_context": {
    "sector_exposure_impact": {
      "sector": "technology",
      "before": 0.15,
      "after": 0.18
    },
    "risk_contribution": 0.008
  }
}
```

#### `TradeExecutedEvent` (from Trade Execution Workflow)
```json
{
  "eventId": "uuid",
  "timestamp": "2025-06-21T10:35:00.000Z",
  "execution": {
    "instrument_id": "AAPL",
    "action": "BUY",
    "quantity": 197,
    "executed_price": 152.28,
    "total_amount": 29999.16
  }
}
```

### Events Produced

#### `RebalanceRequestEvent`
```json
{
  "eventId": "uuid",
  "timestamp": "2025-06-21T11:00:00.000Z",
  "portfolio_id": "main_portfolio",
  "rebalance_type": "STRATEGIC|TACTICAL|RISK_DRIVEN|TIME_BASED",
  "trigger_reason": {
    "type": "WEIGHT_DEVIATION",
    "description": "Technology sector weight exceeded target by 6%",
    "severity": "MEDIUM",
    "urgency": "NORMAL"
  },
  "target_adjustments": [
    {
      "strategy_id": "momentum_strategy",
      "current_weight": 0.35,
      "target_weight": 0.30,
      "adjustment_needed": -0.05,
      "priority": "HIGH"
    },
    {
      "sector": "technology",
      "current_exposure": 0.31,
      "target_exposure": 0.25,
      "adjustment_needed": -0.06,
      "affected_instruments": ["AAPL", "MSFT", "GOOGL"]
    }
  ],
  "constraints": {
    "max_turnover": 0.10,
    "min_trade_size": 1000,
    "execution_timeframe": "1_day",
    "cost_limit": 0.002
  },
  "risk_considerations": {
    "current_portfolio_var": 0.025,
    "target_portfolio_var": 0.022,
    "correlation_impact": "REDUCE_TECH_CORRELATION",
    "liquidity_requirements": "NORMAL"
  }
}
```

#### `PortfolioOptimizationEvent`
```json
{
  "eventId": "uuid",
  "timestamp": "2025-06-21T11:00:00.100Z",
  "portfolio_id": "main_portfolio",
  "optimization_result": {
    "optimal_strategy_weights": {
      "momentum_strategy": 0.30,
      "mean_reversion_strategy": 0.25,
      "trend_following_strategy": 0.20,
      "arbitrage_strategy": 0.15,
      "defensive_strategy": 0.10
    },
    "risk_budget_allocation": {
      "momentum_strategy": 0.35,
      "mean_reversion_strategy": 0.25,
      "trend_following_strategy": 0.20,
      "arbitrage_strategy": 0.10,
      "defensive_strategy": 0.10
    },
    "expected_portfolio_metrics": {
      "expected_return": 0.12,
      "expected_volatility": 0.18,
      "sharpe_ratio": 0.67,
      "max_drawdown": 0.15
    }
  },
  "market_regime": {
    "detected_regime": "TRENDING_MARKET",
    "confidence": 0.82,
    "regime_adjustments": ["INCREASE_MOMENTUM", "REDUCE_MEAN_REVERSION"]
  },
  "optimization_reasoning": {
    "primary_factors": ["strong_momentum_performance", "low_correlation_environment"],
    "adjustments_made": ["increased_momentum_allocation", "reduced_defensive_allocation"],
    "risk_considerations": ["correlation_regime_stable", "volatility_within_targets"]
  }
}
```

#### `PerformanceAttributionEvent`
```json
{
  "eventId": "uuid",
  "timestamp": "2025-06-21T11:00:00.200Z",
  "portfolio_id": "main_portfolio",
  "attribution_period": {
    "start": "2025-06-01T00:00:00.000Z",
    "end": "2025-06-21T00:00:00.000Z"
  },
  "portfolio_performance": {
    "total_return": 0.045,
    "benchmark_return": 0.038,
    "excess_return": 0.007,
    "tracking_error": 0.025,
    "information_ratio": 0.28,
    "sharpe_ratio": 1.85
  },
  "strategy_attribution": {
    "momentum_strategy": {
      "return_contribution": 0.018,
      "risk_contribution": 0.012,
      "weight": 0.32,
      "performance": "OUTPERFORMING"
    },
    "mean_reversion_strategy": {
      "return_contribution": 0.008,
      "risk_contribution": 0.006,
      "weight": 0.24,
      "performance": "NEUTRAL"
    }
  },
  "sector_attribution": {
    "technology": {
      "return_contribution": 0.022,
      "weight_effect": 0.008,
      "selection_effect": 0.014,
      "interaction_effect": 0.000
    }
  },
  "risk_attribution": {
    "systematic_risk": 0.65,
    "idiosyncratic_risk": 0.35,
    "factor_exposures": {
      "market_beta": 1.05,
      "size_factor": -0.15,
      "value_factor": 0.08,
      "momentum_factor": 0.25
    }
  }
}
```

## Microservices Architecture

### 1. Strategy Optimization Service (Python)
**Purpose**: Portfolio-level strategy optimization and allocation
**Technology**: Python + PyPortfolioOpt + cvxpy + NumPy + SciPy
**Scaling**: Horizontal by optimization complexity
**NFRs**: P99 optimization < 5s, optimal allocation quality, multi-objective optimization

### 2. Performance Attribution Service (Python)
**Purpose**: Multi-level performance analysis and attribution
**Technology**: Python + Pandas + NumPy + QuantLib + performance analytics libraries
**Scaling**: Horizontal by attribution complexity
**NFRs**: P99 attribution calculation < 2s, accurate factor attribution, comprehensive analysis

### 3. Rebalancing Engine Service (Python)
**Purpose**: Rebalancing trigger generation and coordination
**Technology**: Python + optimization libraries + asyncio
**Scaling**: Horizontal by portfolio complexity
**NFRs**: P99 trigger evaluation < 1s, optimal rebalancing timing, cost-aware triggers

### 4. Risk Budget Service (Java)
**Purpose**: Risk budget allocation and monitoring across strategies
**Technology**: Java + Spring Boot + risk management libraries
**Scaling**: Horizontal by risk calculation complexity
**NFRs**: P99 risk budget calculation < 500ms, accurate risk attribution, real-time monitoring

### 5. Strategy Coordination Service (Python)
**Purpose**: Multi-strategy management and interaction analysis
**Technology**: Python + asyncio + correlation analysis libraries
**Scaling**: Horizontal by strategy count
**NFRs**: P99 coordination < 300ms, strategy interaction analysis, capacity monitoring

### 6. Portfolio Management Distribution Service (Go)
**Purpose**: Event streaming, reporting, and API management
**Technology**: Go + Apache Pulsar + Redis + gRPC
**Scaling**: Horizontal by topic partitions
**NFRs**: P99 distribution latency < 25ms, 99.99% delivery guarantee, comprehensive reporting

## Messaging Technology Strategy

### Apache Pulsar (Primary for Real-time Management)
**Use Cases**:
- **Rebalancing triggers**: Immediate rebalancing request distribution
- **Performance updates**: Real-time performance attribution updates
- **Risk alerts**: Portfolio risk budget breaches and violations
- **Strategy coordination**: Multi-strategy management and monitoring

**Configuration**:
```yaml
pulsar:
  topics:
    - "portfolio-management/rebalance-requests/{urgency}/{portfolio_id}"
    - "portfolio-management/optimization/{strategy_type}/{portfolio_id}"
    - "portfolio-management/performance/{attribution_level}/{period}"
    - "portfolio-management/risk-alerts/{severity}/{budget_type}"
  retention:
    rebalance_requests: "90 days"
    optimization: "1 year"
    performance: "5 years"
    risk_alerts: "1 year"
  replication:
    clusters: ["us-east", "us-west", "eu-central"]
```

### Apache Kafka (Batch Processing & Analytics)
**Use Cases**:
- **Historical performance analysis**: Long-term performance attribution
- **Strategy backtesting**: Historical strategy performance analysis
- **Risk reporting**: Comprehensive risk and compliance reporting
- **Regulatory reporting**: Audit trails and compliance documentation

## Integration Points with Other Workflows

### Consumes From
- **Portfolio Trading Coordination Workflow**: `CoordinatedTradingDecisionEvent` for portfolio impact tracking
- **Trade Execution Workflow**: `TradeExecutedEvent` for position updates
- **Instrument Analysis Workflow**: `CorrelationMatrixUpdatedEvent` for risk calculations
- **Market Data Workflow**: `NormalizedMarketDataEvent` for portfolio valuation

### Produces For
- **Portfolio Trading Coordination Workflow**: `RebalanceRequestEvent` for portfolio adjustments
- **Reporting Workflow**: `PerformanceAttributionEvent` for comprehensive reporting
- **Risk Management**: Portfolio risk metrics and compliance monitoring

## Data Storage Strategy

### PostgreSQL (Primary Portfolio Data)
- **Portfolio positions**: Current holdings, weights, and allocations
- **Strategy definitions**: Strategy parameters and configurations
- **Performance history**: Historical returns and attribution data
- **Risk budgets**: Strategy and sector risk budget allocations

### TimescaleDB (Time-series Performance & Risk)
- **Portfolio performance**: Time-series portfolio returns and metrics
- **Strategy performance**: Individual strategy performance tracking
- **Risk metrics**: Historical risk measures and attribution
- **Benchmark data**: Benchmark returns and comparison metrics

### Redis (Real-time Caching & State)
- **Current portfolio state**: Real-time positions and exposures
- **Performance cache**: Latest performance metrics and attribution
- **Optimization results**: Cached optimization outcomes
- **Risk calculations**: Current risk budgets and utilization

## Implementation Roadmap

### Phase 1: Core Portfolio Management (Weeks 1-8)
- Deploy Strategy Optimization Service
- Implement Performance Attribution Service
- Set up basic rebalancing trigger logic
- Portfolio state tracking and management

### Phase 2: Advanced Optimization & Risk (Weeks 9-16)
- Deploy Risk Budget Service with advanced allocation
- Implement multi-objective portfolio optimization
- Add comprehensive performance attribution
- Strategy interaction analysis

### Phase 3: Dynamic Management & Coordination (Weeks 17-24)
- Deploy Strategy Coordination Service
- Implement dynamic rebalancing triggers
- Add market regime-based optimization
- Advanced risk budget management

### Phase 4: Advanced Features & Analytics (Weeks 25-32)
- Machine learning-enhanced optimization
- Predictive rebalancing triggers
- Advanced factor attribution models
- Cross-portfolio strategy coordination
```