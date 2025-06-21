# Portfolio Trading Coordination Workflow

## Overview
The Portfolio Trading Coordination Workflow is responsible for matching trading signals with portfolio needs, applying portfolio policies and constraints, and generating coordinated trading decisions with specific amounts and volumes. This workflow serves as the critical coordination layer between pure trading signals and portfolio-level requirements.

## Key Challenges Addressed
- **Signal-Portfolio Matching**: Matching trading signals with portfolio rebalancing needs
- **Multi-Constraint Optimization**: Balancing trading opportunities with portfolio constraints
- **Position Sizing Optimization**: Calculating optimal position sizes using Kelly criterion and risk limits
- **Risk Policy Enforcement**: Applying position limits, sector constraints, and correlation limits
- **Conflict Resolution**: Resolving conflicts between signals and portfolio requirements

## Core Responsibilities
- **Signal-Portfolio Coordination**: Match trading signals with portfolio rebalancing requests
- **Portfolio Policy Enforcement**: Apply position limits, sector constraints, and risk policies
- **Position Sizing**: Calculate optimal position sizes and trade amounts
- **Risk-Aware Trade Coordination**: Integrate correlation matrices and risk metrics
- **Coordinated Decision Generation**: Generate execution-ready trading decisions with amounts

## NOT This Workflow's Responsibilities
- **Signal Generation**: Generating trading signals (belongs to Trading Decision Workflow)
- **Portfolio Strategy**: Long-term portfolio strategy optimization (belongs to Portfolio Management Workflow)
- **Order Execution**: Actual trade execution (belongs to Trade Execution Workflow)
- **Risk Calculation**: Computing risk metrics (belongs to Instrument Analysis Workflow)
- **Performance Attribution**: Portfolio performance analysis (belongs to Portfolio Management Workflow)

## Workflow Sequence

### 1. Signal and Request Processing
**Responsibility**: Coordination Engine Service

#### Input Processing
- **Trading Signals**: Consume `TradingSignalEvent` from Trading Decision Workflow
- **Rebalance Requests**: Consume `RebalanceRequestEvent` from Portfolio Management Workflow
- **Portfolio State**: Real-time portfolio positions and exposures
- **Risk Metrics**: Correlation matrices and risk data from Instrument Analysis Workflow

### 2. Portfolio Policy Enforcement
**Responsibility**: Policy Enforcement Service

#### Risk Policy Validation
```python
class PortfolioPolicyEnforcer:
    def __init__(self):
        self.policy_rules = {
            'max_position_size': 0.05,  # 5% max per instrument
            'max_sector_exposure': 0.25,  # 25% max per sector
            'max_correlation_exposure': 0.60,  # 60% max correlated exposure
            'max_portfolio_volatility': 0.20,  # 20% max portfolio volatility
            'max_leverage': 2.0  # 2x max leverage
        }
        
    async def validate_signal_against_portfolio(
        self, 
        signal: TradingSignalEvent, 
        portfolio_state: PortfolioState,
        correlation_data: CorrelationMatrix
    ) -> PolicyValidationResult:
        """Validate trading signal against portfolio policies"""
        
        violations = []
        warnings = []
        adjustments = {}
        
        # Check if signal aligns with rebalancing needs
        rebalance_alignment = self.check_rebalance_alignment(signal, portfolio_state)
        
        # Validate position size constraints
        max_allowed_size = self.calculate_max_allowed_position_size(
            signal.instrument_id, portfolio_state
        )
        
        # Check sector exposure limits
        sector_impact = await self.calculate_sector_impact(signal, portfolio_state)
        if sector_impact.new_exposure > self.policy_rules['max_sector_exposure']:
            violations.append(f"Sector exposure would exceed {self.policy_rules['max_sector_exposure']:.1%}")
            adjustments['max_position_size'] = self.calculate_sector_constrained_size(
                signal, portfolio_state
            )
        
        # Check correlation exposure
        correlation_impact = self.calculate_correlation_impact(
            signal, portfolio_state, correlation_data
        )
        if correlation_impact > self.policy_rules['max_correlation_exposure']:
            warnings.append(f"High correlation exposure: {correlation_impact:.1%}")
        
        return PolicyValidationResult(
            is_valid=len(violations) == 0,
            violations=violations,
            warnings=warnings,
            adjustments=adjustments,
            max_allowed_size=max_allowed_size,
            rebalance_alignment=rebalance_alignment
        )
```

### 3. Position Sizing and Optimization
**Responsibility**: Position Sizing Service

#### Kelly Criterion with Portfolio Constraints
```python
class PortfolioAwarePositionSizer:
    def __init__(self):
        self.max_kelly_fraction = 0.25  # Cap Kelly at 25%
        self.min_position_size = 0.01   # Minimum 1% position
        
    async def calculate_optimal_position_size(
        self, 
        signal: TradingSignalEvent,
        portfolio_state: PortfolioState,
        policy_constraints: PolicyValidationResult,
        correlation_data: CorrelationMatrix
    ) -> PositionSizingResult:
        """Calculate optimal position size considering portfolio context"""
        
        # Base Kelly calculation
        kelly_size = self.calculate_kelly_fraction(signal)
        
        # Apply portfolio constraints
        constrained_size = min(kelly_size, policy_constraints.max_allowed_size)
        
        # Adjust for correlation risk
        correlation_adjustment = self.calculate_correlation_adjustment(
            signal.instrument_id, portfolio_state, correlation_data
        )
        adjusted_size = constrained_size * correlation_adjustment
        
        # Consider rebalancing needs
        if policy_constraints.rebalance_alignment.is_needed:
            rebalance_size = policy_constraints.rebalance_alignment.suggested_size
            # Blend signal-driven and rebalance-driven sizing
            final_size = self.blend_sizing_approaches(adjusted_size, rebalance_size)
        else:
            final_size = adjusted_size
        
        # Apply minimum size threshold
        if final_size > 0 and final_size < self.min_position_size:
            final_size = self.min_position_size
        
        return PositionSizingResult(
            position_size=final_size,
            kelly_fraction=kelly_size,
            constraint_adjustments=constrained_size,
            correlation_adjustment=correlation_adjustment,
            rebalance_component=rebalance_size if policy_constraints.rebalance_alignment.is_needed else 0,
            reasoning=self.generate_sizing_reasoning(signal, kelly_size, final_size)
        )
```

### 4. Coordinated Decision Generation
**Responsibility**: Decision Coordination Service

#### Decision Synthesis
```python
class CoordinatedDecisionGenerator:
    def __init__(self):
        self.policy_enforcer = PortfolioPolicyEnforcer()
        self.position_sizer = PortfolioAwarePositionSizer()
        
    async def generate_coordinated_decision(
        self, 
        signal: TradingSignalEvent,
        portfolio_state: PortfolioState,
        correlation_data: CorrelationMatrix,
        rebalance_requests: List[RebalanceRequestEvent]
    ) -> Optional[CoordinatedTradingDecisionEvent]:
        """Generate coordinated trading decision with amounts and portfolio context"""
        
        # Validate signal against portfolio policies
        policy_validation = await self.policy_enforcer.validate_signal_against_portfolio(
            signal, portfolio_state, correlation_data
        )
        
        if not policy_validation.is_valid:
            # Try with policy adjustments
            if policy_validation.adjustments:
                adjusted_validation = await self.retry_with_adjustments(
                    signal, portfolio_state, policy_validation.adjustments
                )
                if not adjusted_validation.is_valid:
                    return None  # Cannot create compliant decision
                policy_validation = adjusted_validation
            else:
                return None
        
        # Calculate optimal position size
        sizing_result = await self.position_sizer.calculate_optimal_position_size(
            signal, portfolio_state, policy_validation, correlation_data
        )
        
        if sizing_result.position_size <= 0:
            return None  # No viable position size
        
        # Calculate trade amount in dollars
        current_price = await self.get_current_price(signal.instrument_id)
        portfolio_value = portfolio_state.total_value
        trade_amount = sizing_result.position_size * portfolio_value
        share_quantity = int(trade_amount / current_price)
        
        # Determine execution strategy
        execution_strategy = self.determine_execution_strategy(
            signal, sizing_result, portfolio_state
        )
        
        # Calculate risk metrics for this decision
        decision_risk_metrics = await self.calculate_decision_risk_metrics(
            signal.instrument_id, sizing_result.position_size, portfolio_state, correlation_data
        )
        
        return CoordinatedTradingDecisionEvent(
            instrument_id=signal.instrument_id,
            action=signal.action,
            position_size=sizing_result.position_size,
            trade_amount=trade_amount,
            share_quantity=share_quantity,
            signal_basis=signal,
            portfolio_context=self.create_portfolio_context(portfolio_state, sizing_result),
            risk_metrics=decision_risk_metrics,
            execution_strategy=execution_strategy,
            policy_compliance=policy_validation,
            coordination_reasoning=self.generate_coordination_reasoning(
                signal, sizing_result, policy_validation
            )
        )
```

### 5. Conflict Resolution and Prioritization
**Responsibility**: Conflict Resolution Service

#### Multi-Signal Coordination
- **Signal Prioritization**: Rank signals by confidence, urgency, and portfolio impact
- **Resource Allocation**: Distribute available capital across multiple opportunities
- **Timing Coordination**: Sequence trades to minimize market impact
- **Risk Budget Management**: Ensure total risk exposure stays within limits

### 6. Event-Driven Coordination Distribution
**Responsibility**: Coordination Distribution Service
- **Real-time streaming**: Apache Pulsar for immediate coordinated decisions
- **Decision persistence**: Store decisions with full coordination metadata
- **Performance tracking**: Monitor coordination effectiveness and outcomes
- **API gateway**: RESTful and gRPC APIs for decision consumption

## Event Contracts

### Events Consumed

#### `TradingSignalEvent` (from Trading Decision Workflow)
```json
{
  "eventId": "uuid",
  "timestamp": "2025-06-21T10:30:00.200Z",
  "signal": {
    "instrument_id": "AAPL",
    "action": "BUY",
    "confidence": 0.81,
    "strength": "STRONG",
    "signal_score": 0.78
  },
  "expected_outcomes": {
    "price_target": 155.25,
    "expected_return": 0.025,
    "success_probability": 0.78
  }
}
```

#### `RebalanceRequestEvent` (from Portfolio Management Workflow)
```json
{
  "eventId": "uuid",
  "timestamp": "2025-06-21T10:30:00.100Z",
  "portfolio_id": "main_portfolio",
  "rebalance_type": "STRATEGIC|TACTICAL|RISK_DRIVEN",
  "target_adjustments": [
    {
      "instrument_id": "AAPL",
      "current_weight": 0.04,
      "target_weight": 0.06,
      "adjustment_needed": 0.02,
      "urgency": "NORMAL"
    }
  ],
  "constraints": {
    "max_turnover": 0.10,
    "min_trade_size": 1000,
    "execution_timeframe": "1_day"
  }
}
```

### Events Produced

#### `CoordinatedTradingDecisionEvent`
```json
{
  "eventId": "uuid",
  "timestamp": "2025-06-21T10:30:00.300Z",
  "decision": {
    "instrument_id": "AAPL",
    "action": "BUY",
    "position_size": 0.03,
    "trade_amount": 30000.00,
    "share_quantity": 197,
    "confidence": 0.81,
    "priority": "HIGH"
  },
  "signal_basis": {
    "signal_id": "signal-uuid-123",
    "signal_confidence": 0.81,
    "signal_strength": "STRONG",
    "expected_return": 0.025
  },
  "portfolio_context": {
    "current_position": 0.01,
    "target_position": 0.04,
    "sector_exposure_impact": {
      "sector": "technology",
      "before": 0.15,
      "after": 0.18,
      "limit": 0.25
    },
    "correlation_impact": 0.12,
    "risk_contribution": 0.008
  },
  "position_sizing": {
    "kelly_fraction": 0.045,
    "policy_constrained": 0.035,
    "correlation_adjusted": 0.032,
    "final_size": 0.030,
    "sizing_reasoning": "Kelly reduced by correlation risk and sector limits"
  },
  "risk_metrics": {
    "position_var_1d": 900.00,
    "portfolio_var_impact": 0.003,
    "max_loss_estimate": 1500.00,
    "risk_reward_ratio": 2.5
  },
  "execution_strategy": {
    "order_type": "LIMIT",
    "execution_algorithm": "TWAP",
    "time_horizon": "4_hours",
    "urgency": "NORMAL"
  },
  "policy_compliance": {
    "position_limit_check": "PASSED",
    "sector_limit_check": "PASSED",
    "correlation_limit_check": "WARNING",
    "violations": [],
    "warnings": ["high_correlation_with_existing_positions"]
  }
}
```

## Microservices Architecture

### 1. Coordination Engine Service (Python)
**Purpose**: Core coordination logic matching signals with portfolio needs
**Technology**: Python + FastAPI + asyncio + optimization libraries
**Scaling**: Horizontal by coordination complexity
**NFRs**: P99 coordination latency < 200ms, optimal signal-portfolio matching

### 2. Policy Enforcement Service (Python)
**Purpose**: Portfolio policy validation and constraint enforcement
**Technology**: Python + Pydantic + NumPy + constraint solvers
**Scaling**: Horizontal by policy complexity
**NFRs**: P99 validation < 100ms, 100% policy compliance, configurable rules

### 3. Position Sizing Service (Python)
**Purpose**: Kelly criterion and portfolio-aware position sizing
**Technology**: Python + NumPy + SciPy + optimization libraries
**Scaling**: Horizontal by calculation complexity
**NFRs**: P99 sizing calculation < 150ms, mathematically optimal sizing

### 4. Conflict Resolution Service (Python)
**Purpose**: Multi-signal prioritization and resource allocation
**Technology**: Python + optimization libraries + linear programming
**Scaling**: Horizontal by conflict complexity
**NFRs**: P99 resolution < 300ms, optimal resource allocation

### 5. Portfolio State Service (Java)
**Purpose**: Real-time portfolio state tracking and management
**Technology**: Java + Spring Boot + PostgreSQL + Redis
**Scaling**: Horizontal by portfolio segments
**NFRs**: P99 state update < 50ms, 99.99% data consistency

### 6. Coordination Distribution Service (Go)
**Purpose**: Event streaming, decision persistence, and API management
**Technology**: Go + Apache Pulsar + Redis + gRPC
**Scaling**: Horizontal by topic partitions
**NFRs**: P99 distribution latency < 25ms, 99.99% delivery guarantee

## Integration Points with Other Workflows

### Consumes From
- **Trading Decision Workflow**: `TradingSignalEvent` for pure trading signals
- **Portfolio Management Workflow**: `RebalanceRequestEvent` for portfolio rebalancing needs
- **Instrument Analysis Workflow**: `CorrelationMatrixUpdatedEvent` for risk calculations
- **Trade Execution Workflow**: `TradeExecutedEvent` for portfolio state updates

### Produces For
- **Trade Execution Workflow**: `CoordinatedTradingDecisionEvent` for order execution
- **Portfolio Management Workflow**: Portfolio impact and coordination feedback
- **Reporting Workflow**: Coordination effectiveness and decision outcomes

## Implementation Roadmap

### Phase 1: Core Coordination Engine (Weeks 1-8)
- Deploy Coordination Engine Service
- Implement Policy Enforcement Service with basic rules
- Set up Portfolio State Service
- Basic signal-portfolio matching

### Phase 2: Advanced Position Sizing (Weeks 9-16)
- Deploy Position Sizing Service with Kelly criterion
- Implement correlation-aware sizing adjustments
- Add portfolio constraint optimization
- Multi-objective position sizing

### Phase 3: Conflict Resolution & Optimization (Weeks 17-24)
- Deploy Conflict Resolution Service
- Implement multi-signal prioritization
- Add resource allocation optimization
- Advanced coordination algorithms

### Phase 4: Advanced Features (Weeks 25-32)
- Machine learning-enhanced coordination
- Predictive conflict resolution
- Dynamic policy adjustment
- Cross-portfolio coordination
