# Trading Strategy Service

## Purpose
The Trading Strategy Service is responsible for implementing trading strategies and generating trade decisions with comprehensive backtesting and optimization capabilities. It transforms predictions, risk metrics, and market data into actionable trading signals with appropriate position sizing and timing recommendations.

## Strict Boundaries

### Responsibilities
- Strategy implementation and execution
- Signal generation and filtering
- Position sizing and risk management
- Entry and exit point determination
- Stop-loss and take-profit calculation
- Strategy performance tracking
- Strategy parameter optimization
- Trade decision generation with reasoning

### Non-Responsibilities
- Does NOT execute trades
- Does NOT collect or normalize market data
- Does NOT calculate technical indicators
- Does NOT generate price predictions
- Does NOT perform risk calculations
- Does NOT manage user preferences or authentication
- Does NOT handle order routing or broker integration

## API Design (API-First)

### REST API

#### GET /api/v1/strategies
Retrieves available trading strategies.

**Query Parameters:**
- `status` (string, optional): Filter by strategy status (active, inactive, testing)
- `category` (string, optional): Filter by strategy category (trend, mean_reversion, momentum, etc.)
- `instrument_type` (string, optional): Filter by supported instrument type (stock, forex, crypto, etc.)
- `limit` (integer, optional): Maximum number of results (default: 20)
- `offset` (integer, optional): Pagination offset (default: 0)

**Response:**
```json
{
  "strategies": [
    {
      "id": "strat-12345",
      "name": "Adaptive Momentum",
      "description": "Momentum strategy with adaptive lookback period based on market volatility",
      "category": "momentum",
      "status": "active",
      "supported_instruments": ["stock", "etf"],
      "risk_profile": "moderate",
      "performance_metrics": {
        "sharpe_ratio": 1.85,
        "sortino_ratio": 2.25,
        "max_drawdown": 0.15,
        "win_rate": 0.62
      },
      "created_at": "2025-01-15T10:00:00Z",
      "last_updated": "2025-06-10T14:30:00Z"
    },
    {
      "id": "strat-67890",
      "name": "Mean Reversion with Bollinger Bands",
      "description": "Mean reversion strategy using Bollinger Bands with dynamic width",
      "category": "mean_reversion",
      "status": "active",
      "supported_instruments": ["stock", "etf", "forex"],
      "risk_profile": "moderate",
      "performance_metrics": {
        "sharpe_ratio": 1.65,
        "sortino_ratio": 2.10,
        "max_drawdown": 0.12,
        "win_rate": 0.58
      },
      "created_at": "2025-02-20T09:15:00Z",
      "last_updated": "2025-06-12T11:45:00Z"
    }
  ],
  "pagination": {
    "total": 12,
    "limit": 20,
    "offset": 0
  }
}
```

#### GET /api/v1/strategies/{strategy_id}
Retrieves detailed information about a specific strategy.

**Path Parameters:**
- `strategy_id` (string, required): Strategy identifier

**Query Parameters:**
- `include_parameters` (boolean, optional): Include strategy parameters (default: false)
- `include_performance` (boolean, optional): Include detailed performance metrics (default: false)

**Response:**
```json
{
  "id": "strat-12345",
  "name": "Adaptive Momentum",
  "description": "Momentum strategy with adaptive lookback period based on market volatility",
  "category": "momentum",
  "status": "active",
  "supported_instruments": ["stock", "etf"],
  "risk_profile": "moderate",
  "timeframes": ["1d", "1w"],
  "min_history_required": "60d",
  "parameters": [
    {
      "name": "base_lookback_period",
      "type": "integer",
      "default_value": 20,
      "min_value": 5,
      "max_value": 60,
      "description": "Base lookback period for momentum calculation"
    },
    {
      "name": "volatility_adjustment_factor",
      "type": "float",
      "default_value": 0.5,
      "min_value": 0.1,
      "max_value": 1.0,
      "description": "Factor to adjust lookback period based on volatility"
    },
    {
      "name": "entry_threshold",
      "type": "float",
      "default_value": 0.02,
      "min_value": 0.005,
      "max_value": 0.05,
      "description": "Minimum momentum for entry"
    }
  ],
  "performance_metrics": {
    "overall": {
      "sharpe_ratio": 1.85,
      "sortino_ratio": 2.25,
      "max_drawdown": 0.15,
      "win_rate": 0.62,
      "profit_factor": 1.75,
      "average_trade": 0.0125,
      "max_consecutive_losses": 4
    },
    "by_market_regime": {
      "trending": {
        "sharpe_ratio": 2.35,
        "win_rate": 0.72
      },
      "mean_reverting": {
        "sharpe_ratio": 1.25,
        "win_rate": 0.48
      },
      "volatile": {
        "sharpe_ratio": 1.45,
        "win_rate": 0.55
      }
    },
    "monthly_returns": [
      {
        "year": 2025,
        "month": 1,
        "return": 0.035
      },
      {
        "year": 2025,
        "month": 2,
        "return": -0.012
      },
      {
        "year": 2025,
        "month": 3,
        "return": 0.028
      }
    ]
  },
  "created_at": "2025-01-15T10:00:00Z",
  "last_updated": "2025-06-10T14:30:00Z",
  "created_by": "user-12345"
}
```

#### POST /api/v1/strategies/{strategy_id}/backtest
Runs a backtest for a specific strategy.

**Path Parameters:**
- `strategy_id` (string, required): Strategy identifier

**Request Body:**
```json
{
  "instruments": ["AAPL", "MSFT", "GOOGL"],
  "start_date": "2024-01-01T00:00:00Z",
  "end_date": "2025-01-01T00:00:00Z",
  "initial_capital": 100000.00,
  "parameters": {
    "base_lookback_period": 25,
    "volatility_adjustment_factor": 0.6,
    "entry_threshold": 0.015
  },
  "position_sizing": {
    "method": "percent_risk",
    "risk_per_trade": 0.01
  },
  "include_trade_list": true
}
```

**Response:**
```json
{
  "backtest_id": "bt-12345",
  "strategy_id": "strat-12345",
  "start_date": "2024-01-01T00:00:00Z",
  "end_date": "2025-01-01T00:00:00Z",
  "initial_capital": 100000.00,
  "final_capital": 125000.00,
  "total_return": 0.25,
  "annualized_return": 0.25,
  "performance_metrics": {
    "sharpe_ratio": 1.92,
    "sortino_ratio": 2.35,
    "max_drawdown": 0.12,
    "win_rate": 0.64,
    "profit_factor": 1.85,
    "average_trade": 0.0135,
    "max_consecutive_losses": 3
  },
  "monthly_returns": [
    {
      "year": 2024,
      "month": 1,
      "return": 0.032
    },
    {
      "year": 2024,
      "month": 2,
      "return": -0.008
    }
  ],
  "equity_curve": [
    {
      "date": "2024-01-01T00:00:00Z",
      "equity": 100000.00
    },
    {
      "date": "2024-01-02T00:00:00Z",
      "equity": 100250.00
    }
  ],
  "trades": [
    {
      "instrument": "AAPL",
      "entry_date": "2024-01-05T00:00:00Z",
      "entry_price": 185.25,
      "exit_date": "2024-01-12T00:00:00Z",
      "exit_price": 192.50,
      "direction": "long",
      "size": 100,
      "profit_loss": 725.00,
      "profit_loss_percent": 0.0392,
      "reason": "Take profit hit"
    }
  ],
  "parameters_used": {
    "base_lookback_period": 25,
    "volatility_adjustment_factor": 0.6,
    "entry_threshold": 0.015
  },
  "completed_at": "2025-06-20T16:30:00Z"
}
```

#### GET /api/v1/decisions
Retrieves trade decisions generated by strategies.

**Query Parameters:**
- `instruments` (string, optional): Comma-separated list of instrument IDs
- `strategies` (string, optional): Comma-separated list of strategy IDs
- `decision_types` (string, optional): Comma-separated list of decision types (buy, sell, hold, close)
- `min_confidence` (float, optional): Minimum confidence level (0.0-1.0, default: 0.6)
- `from` (string, optional): Start timestamp (ISO 8601)
- `to` (string, optional): End timestamp (ISO 8601)
- `limit` (integer, optional): Maximum number of results (default: 20)

**Response:**
```json
{
  "decisions": [
    {
      "id": "dec-12345",
      "instrument_id": "AAPL",
      "strategy_id": "strat-12345",
      "decision_type": "buy",
      "confidence": 0.85,
      "generated_at": "2025-06-20T15:30:00Z",
      "timeframe": "1d",
      "price_at_decision": 150.25,
      "target_price": 165.50,
      "stop_loss": 145.00,
      "risk_reward_ratio": 3.05,
      "position_sizing": {
        "method": "percent_risk",
        "risk_percent": 1.0,
        "suggested_position": 200
      },
      "reasoning": [
        "Strong momentum signal (0.035) above threshold (0.015)",
        "Price above all major moving averages",
        "Positive sentiment from recent earnings",
        "Low volatility environment favors momentum strategies"
      ],
      "execution_window": {
        "start": "2025-06-20T15:30:00Z",
        "end": "2025-06-20T20:00:00Z",
        "optimal_time": "2025-06-20T16:15:00Z"
      }
    },
    {
      "id": "dec-67890",
      "instrument_id": "MSFT",
      "strategy_id": "strat-67890",
      "decision_type": "sell",
      "confidence": 0.75,
      "generated_at": "2025-06-20T15:35:00Z",
      "timeframe": "1d",
      "price_at_decision": 320.50,
      "target_price": 305.00,
      "stop_loss": 328.00,
      "risk_reward_ratio": 2.06,
      "position_sizing": {
        "method": "percent_risk",
        "risk_percent": 1.0,
        "suggested_position": 150
      },
      "reasoning": [
        "Price at upper Bollinger Band (2.5 standard deviations)",
        "RSI in overbought territory (78.5)",
        "Negative divergence on MACD",
        "Historical resistance level at 322.50"
      ],
      "execution_window": {
        "start": "2025-06-20T15:35:00Z",
        "end": "2025-06-20T20:00:00Z",
        "optimal_time": "2025-06-20T16:30:00Z"
      }
    }
  ],
  "pagination": {
    "total": 8,
    "limit": 20,
    "offset": 0
  }
}
```

### gRPC API

```protobuf
syntax = "proto3";

package trading_strategy.v1;

import "google/protobuf/timestamp.proto";

service TradingStrategyService {
  // Get available strategies
  rpc GetStrategies(GetStrategiesRequest) returns (GetStrategiesResponse);
  
  // Get strategy details
  rpc GetStrategy(GetStrategyRequest) returns (GetStrategyResponse);
  
  // Run strategy backtest
  rpc RunBacktest(BacktestRequest) returns (BacktestResponse);
  
  // Get trade decisions
  rpc GetDecisions(GetDecisionsRequest) returns (GetDecisionsResponse);
  
  // Stream real-time trade decisions
  rpc StreamDecisions(StreamDecisionsRequest) returns (stream TradeDecision);
  
  // Optimize strategy parameters
  rpc OptimizeStrategy(OptimizeStrategyRequest) returns (OptimizeStrategyResponse);
}

message GetStrategiesRequest {
  string status = 1;
  string category = 2;
  string instrument_type = 3;
  int32 limit = 4;
  int32 offset = 5;
}

message GetStrategiesResponse {
  repeated StrategyInfo strategies = 1;
  Pagination pagination = 2;
}

message StrategyInfo {
  string id = 1;
  string name = 2;
  string description = 3;
  string category = 4;
  string status = 5;
  repeated string supported_instruments = 6;
  string risk_profile = 7;
  PerformanceMetricsSummary performance_metrics = 8;
  google.protobuf.Timestamp created_at = 9;
  google.protobuf.Timestamp last_updated = 10;
}

message PerformanceMetricsSummary {
  double sharpe_ratio = 1;
  double sortino_ratio = 2;
  double max_drawdown = 3;
  double win_rate = 4;
}

message Pagination {
  int32 total = 1;
  int32 limit = 2;
  int32 offset = 3;
}

message GetStrategyRequest {
  string strategy_id = 1;
  bool include_parameters = 2;
  bool include_performance = 3;
}

message GetStrategyResponse {
  string id = 1;
  string name = 2;
  string description = 3;
  string category = 4;
  string status = 5;
  repeated string supported_instruments = 6;
  string risk_profile = 7;
  repeated string timeframes = 8;
  string min_history_required = 9;
  repeated StrategyParameter parameters = 10;
  PerformanceMetrics performance_metrics = 11;
  google.protobuf.Timestamp created_at = 12;
  google.protobuf.Timestamp last_updated = 13;
  string created_by = 14;
}

message StrategyParameter {
  string name = 1;
  string type = 2;
  oneof default_value {
    int32 int_value = 3;
    double float_value = 4;
    string string_value = 5;
    bool bool_value = 6;
  }
  double min_value = 7;
  double max_value = 8;
  string description = 9;
}

message PerformanceMetrics {
  PerformanceMetricSet overall = 1;
  map<string, PerformanceMetricSet> by_market_regime = 2;
  repeated MonthlyReturn monthly_returns = 3;
}

message PerformanceMetricSet {
  double sharpe_ratio = 1;
  double sortino_ratio = 2;
  double max_drawdown = 3;
  double win_rate = 4;
  double profit_factor = 5;
  double average_trade = 6;
  int32 max_consecutive_losses = 7;
}

message MonthlyReturn {
  int32 year = 1;
  int32 month = 2;
  double return = 3;
}

message BacktestRequest {
  string strategy_id = 1;
  repeated string instruments = 2;
  google.protobuf.Timestamp start_date = 3;
  google.protobuf.Timestamp end_date = 4;
  double initial_capital = 5;
  map<string, double> parameters = 6;
  PositionSizing position_sizing = 7;
  bool include_trade_list = 8;
}

message PositionSizing {
  string method = 1;
  double risk_per_trade = 2;
}

message BacktestResponse {
  string backtest_id = 1;
  string strategy_id = 2;
  google.protobuf.Timestamp start_date = 3;
  google.protobuf.Timestamp end_date = 4;
  double initial_capital = 5;
  double final_capital = 6;
  double total_return = 7;
  double annualized_return = 8;
  PerformanceMetricSet performance_metrics = 9;
  repeated MonthlyReturn monthly_returns = 10;
  repeated EquityPoint equity_curve = 11;
  repeated Trade trades = 12;
  map<string, double> parameters_used = 13;
  google.protobuf.Timestamp completed_at = 14;
}

message EquityPoint {
  google.protobuf.Timestamp date = 1;
  double equity = 2;
}

message Trade {
  string instrument = 1;
  google.protobuf.Timestamp entry_date = 2;
  double entry_price = 3;
  google.protobuf.Timestamp exit_date = 4;
  double exit_price = 5;
  string direction = 6;
  double size = 7;
  double profit_loss = 8;
  double profit_loss_percent = 9;
  string reason = 10;
}

message GetDecisionsRequest {
  repeated string instruments = 1;
  repeated string strategies = 2;
  repeated string decision_types = 3;
  double min_confidence = 4;
  google.protobuf.Timestamp from = 5;
  google.protobuf.Timestamp to = 6;
  int32 limit = 7;
}

message GetDecisionsResponse {
  repeated TradeDecision decisions = 1;
  Pagination pagination = 2;
}

message TradeDecision {
  string id = 1;
  string instrument_id = 2;
  string strategy_id = 3;
  string decision_type = 4;
  double confidence = 5;
  google.protobuf.Timestamp generated_at = 6;
  string timeframe = 7;
  double price_at_decision = 8;
  double target_price = 9;
  double stop_loss = 10;
  double risk_reward_ratio = 11;
  PositionSizingRecommendation position_sizing = 12;
  repeated string reasoning = 13;
  ExecutionWindow execution_window = 14;
}

message PositionSizingRecommendation {
  string method = 1;
  double risk_percent = 2;
  double suggested_position = 3;
}

message ExecutionWindow {
  google.protobuf.Timestamp start = 1;
  google.protobuf.Timestamp end = 2;
  google.protobuf.Timestamp optimal_time = 3;
}

message StreamDecisionsRequest {
  repeated string instruments = 1;
  repeated string strategies = 2;
  repeated string decision_types = 3;
  double min_confidence = 4;
}

message OptimizeStrategyRequest {
  string strategy_id = 1;
  repeated string instruments = 2;
  google.protobuf.Timestamp start_date = 3;
  google.protobuf.Timestamp end_date = 4;
  double initial_capital = 5;
  repeated ParameterRange parameter_ranges = 6;
  string optimization_metric = 7;
  int32 max_iterations = 8;
}

message ParameterRange {
  string name = 1;
  double min_value = 2;
  double max_value = 3;
  double step = 4;
}

message OptimizeStrategyResponse {
  string optimization_id = 1;
  string strategy_id = 2;
  repeated ParameterSet top_parameter_sets = 3;
  ParameterSet best_parameters = 4;
  PerformanceMetricSet best_performance = 5;
  int32 iterations_completed = 6;
  google.protobuf.Timestamp completed_at = 7;
}

message ParameterSet {
  map<string, double> parameters = 1;
  PerformanceMetricSet performance = 2;
  double optimization_score = 3;
}
```

## Data Model

### Core Entities

#### Strategy
Represents a trading strategy with its configuration and parameters.

**Attributes:**
- `id` (string): Unique identifier for the strategy
- `name` (string): Human-readable name
- `description` (string): Detailed description
- `category` (enum): Strategy category (trend, mean_reversion, momentum, etc.)
- `status` (enum): Current status (active, inactive, testing)
- `supported_instruments` (array): Types of instruments supported
- `risk_profile` (enum): Risk profile (conservative, moderate, aggressive)
- `timeframes` (array): Supported timeframes (1h, 4h, 1d, 1w)
- `min_history_required` (string): Minimum history required for the strategy
- `parameters` (array): Strategy parameters with constraints
- `created_at` (datetime): When the strategy was created
- `updated_at` (datetime): When the strategy was last updated
- `created_by` (string): User who created the strategy
- `implementation` (string): Reference to the strategy implementation code

#### StrategyParameter
Represents a configurable parameter for a strategy.

**Attributes:**
- `id` (string): Unique identifier for the parameter
- `strategy_id` (string): Reference to the strategy
- `name` (string): Parameter name
- `type` (enum): Parameter type (integer, float, string, boolean)
- `default_value` (any): Default value
- `min_value` (number): Minimum allowed value (for numeric parameters)
- `max_value` (number): Maximum allowed value (for numeric parameters)
- `description` (string): Parameter description
- `is_required` (boolean): Whether the parameter is required
- `created_at` (datetime): When the parameter was created
- `updated_at` (datetime): When the parameter was last updated

#### TradeDecision
Represents a trading decision generated by a strategy.

**Attributes:**
- `id` (string): Unique identifier for the decision
- `instrument_id` (string): Reference to the instrument
- `strategy_id` (string): Reference to the strategy
- `decision_type` (enum): Type of decision (buy, sell, hold, close)
- `confidence` (float): Confidence level of the decision (0.0-1.0)
- `generated_at` (datetime): When the decision was generated
- `timeframe` (enum): Timeframe for the decision (1h, 4h, 1d, 1w)
- `price_at_decision` (decimal): Instrument price when decision was made
- `target_price` (decimal): Price target for the trade
- `stop_loss` (decimal): Stop-loss price
- `risk_reward_ratio` (decimal): Risk-reward ratio
- `position_sizing` (object): Position sizing recommendation
- `reasoning` (array): Reasoning behind the decision
- `execution_window` (object): Recommended execution time window
- `metadata` (map): Additional decision-specific metadata

#### Backtest
Represents a backtest run for a strategy.

**Attributes:**
- `id` (string): Unique identifier for the backtest
- `strategy_id` (string): Reference to the strategy
- `instruments` (array): Instruments included in the backtest
- `start_date` (datetime): Start date of the backtest
- `end_date` (datetime): End date of the backtest
- `initial_capital` (decimal): Initial capital for the backtest
- `final_capital` (decimal): Final capital after the backtest
- `parameters` (map): Strategy parameters used
- `position_sizing` (object): Position sizing method used
- `performance_metrics` (object): Performance metrics from the backtest
- `trades` (array): Trades executed during the backtest
- `equity_curve` (array): Equity curve data points
- `created_at` (datetime): When the backtest was created
- `completed_at` (datetime): When the backtest was completed
- `created_by` (string): User who created the backtest
- `status` (enum): Backtest status (running, completed, failed)

#### StrategyOptimization
Represents a parameter optimization run for a strategy.

**Attributes:**
- `id` (string): Unique identifier for the optimization
- `strategy_id` (string): Reference to the strategy
- `instruments` (array): Instruments included in the optimization
- `start_date` (datetime): Start date for optimization data
- `end_date` (datetime): End date for optimization data
- `parameter_ranges` (array): Parameter ranges to explore
- `optimization_metric` (string): Metric to optimize for
- `max_iterations` (integer): Maximum number of iterations
- `iterations_completed` (integer): Number of iterations completed
- `best_parameters` (map): Best parameter set found
- `best_performance` (object): Performance with best parameters
- `top_parameter_sets` (array): Top performing parameter sets
- `created_at` (datetime): When the optimization was created
- `completed_at` (datetime): When the optimization was completed
- `created_by` (string): User who created the optimization
- `status` (enum): Optimization status (running, completed, failed)

## DB Schema (CQRS Pattern)

### Write Schema (Command Side)

#### strategies
```sql
CREATE TABLE strategies (
    id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    category VARCHAR(30) NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'inactive',
    supported_instruments JSONB NOT NULL,
    risk_profile VARCHAR(20) NOT NULL,
    timeframes JSONB NOT NULL,
    min_history_required VARCHAR(20) NOT NULL,
    implementation_path VARCHAR(255) NOT NULL,
    created_by VARCHAR(50),
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX strategies_category_idx ON strategies(category);
CREATE INDEX strategies_status_idx ON strategies(status);
```

#### strategy_parameters
```sql
CREATE TABLE strategy_parameters (
    id VARCHAR(50) PRIMARY KEY,
    strategy_id VARCHAR(50) NOT NULL REFERENCES strategies(id),
    name VARCHAR(50) NOT NULL,
    type VARCHAR(20) NOT NULL,
    default_value_int INTEGER,
    default_value_float DECIMAL(18, 8),
    default_value_string TEXT,
    default_value_bool BOOLEAN,
    min_value DECIMAL(18, 8),
    max_value DECIMAL(18, 8),
    description TEXT,
    is_required BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT strategy_parameters_strategy_name_idx UNIQUE (strategy_id, name)
);

CREATE INDEX strategy_parameters_strategy_id_idx ON strategy_parameters(strategy_id);
```

#### trade_decisions
```sql
CREATE TABLE trade_decisions (
    id VARCHAR(50) PRIMARY KEY,
    instrument_id VARCHAR(20) NOT NULL,
    strategy_id VARCHAR(50) NOT NULL REFERENCES strategies(id),
    decision_type VARCHAR(10) NOT NULL,
    confidence DECIMAL(4,3) NOT NULL,
    generated_at TIMESTAMP WITH TIME ZONE NOT NULL,
    timeframe VARCHAR(10) NOT NULL,
    price_at_decision DECIMAL(18, 8) NOT NULL,
    target_price DECIMAL(18, 8),
    stop_loss DECIMAL(18, 8),
    risk_reward_ratio DECIMAL(8, 4),
    position_sizing JSONB NOT NULL,
    reasoning JSONB,
    execution_window JSONB,
    metadata JSONB,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX trade_decisions_instrument_id_idx ON trade_decisions(instrument_id);
CREATE INDEX trade_decisions_strategy_id_idx ON trade_decisions(strategy_id);
CREATE INDEX trade_decisions_generated_at_idx ON trade_decisions(generated_at);
```

#### backtests
```sql
CREATE TABLE backtests (
    id VARCHAR(50) PRIMARY KEY,
    strategy_id VARCHAR(50) NOT NULL REFERENCES strategies(id),
    instruments JSONB NOT NULL,
    start_date TIMESTAMP WITH TIME ZONE NOT NULL,
    end_date TIMESTAMP WITH TIME ZONE NOT NULL,
    initial_capital DECIMAL(18, 2) NOT NULL,
    final_capital DECIMAL(18, 2),
    parameters JSONB NOT NULL,
    position_sizing JSONB NOT NULL,
    performance_metrics JSONB,
    created_by VARCHAR(50),
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    completed_at TIMESTAMP WITH TIME ZONE,
    status VARCHAR(20) NOT NULL DEFAULT 'running'
);

CREATE INDEX backtests_strategy_id_idx ON backtests(strategy_id);
CREATE INDEX backtests_status_idx ON backtests(status);
```

#### backtest_trades
```sql
CREATE TABLE backtest_trades (
    id SERIAL PRIMARY KEY,
    backtest_id VARCHAR(50) NOT NULL REFERENCES backtests(id),
    instrument VARCHAR(20) NOT NULL,
    entry_date TIMESTAMP WITH TIME ZONE NOT NULL,
    entry_price DECIMAL(18, 8) NOT NULL,
    exit_date TIMESTAMP WITH TIME ZONE,
    exit_price DECIMAL(18, 8),
    direction VARCHAR(10) NOT NULL,
    size DECIMAL(18, 8) NOT NULL,
    profit_loss DECIMAL(18, 8),
    profit_loss_percent DECIMAL(8, 6),
    reason VARCHAR(100),
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX backtest_trades_backtest_id_idx ON backtest_trades(backtest_id);
```

#### backtest_equity_curve
```sql
CREATE TABLE backtest_equity_curve (
    id SERIAL PRIMARY KEY,
    backtest_id VARCHAR(50) NOT NULL REFERENCES backtests(id),
    date TIMESTAMP WITH TIME ZONE NOT NULL,
    equity DECIMAL(18, 2) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT backtest_equity_curve_backtest_date_idx UNIQUE (backtest_id, date)
);

CREATE INDEX backtest_equity_curve_backtest_id_idx ON backtest_equity_curve(backtest_id);
```

#### strategy_optimizations
```sql
CREATE TABLE strategy_optimizations (
    id VARCHAR(50) PRIMARY KEY,
    strategy_id VARCHAR(50) NOT NULL REFERENCES strategies(id),
    instruments JSONB NOT NULL,
    start_date TIMESTAMP WITH TIME ZONE NOT NULL,
    end_date TIMESTAMP WITH TIME ZONE NOT NULL,
    parameter_ranges JSONB NOT NULL,
    optimization_metric VARCHAR(30) NOT NULL,
    max_iterations INTEGER NOT NULL,
    iterations_completed INTEGER NOT NULL DEFAULT 0,
    best_parameters JSONB,
    best_performance JSONB,
    created_by VARCHAR(50),
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    completed_at TIMESTAMP WITH TIME ZONE,
    status VARCHAR(20) NOT NULL DEFAULT 'running'
);

CREATE INDEX strategy_optimizations_strategy_id_idx ON strategy_optimizations(strategy_id);
CREATE INDEX strategy_optimizations_status_idx ON strategy_optimizations(status);
```

#### optimization_results
```sql
CREATE TABLE optimization_results (
    id SERIAL PRIMARY KEY,
    optimization_id VARCHAR(50) NOT NULL REFERENCES strategy_optimizations(id),
    parameters JSONB NOT NULL,
    performance JSONB NOT NULL,
    optimization_score DECIMAL(10, 6) NOT NULL,
    iteration INTEGER NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT optimization_results_optimization_iteration_idx UNIQUE (optimization_id, iteration)
);

CREATE INDEX optimization_results_optimization_id_idx ON optimization_results(optimization_id);
CREATE INDEX optimization_results_score_idx ON optimization_results(optimization_score DESC);
```

### Read Schema (Query Side)

#### active_strategies
```sql
CREATE TABLE active_strategies (
    id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    category VARCHAR(30) NOT NULL,
    supported_instruments JSONB NOT NULL,
    risk_profile VARCHAR(20) NOT NULL,
    timeframes JSONB NOT NULL,
    parameters JSONB,
    performance_summary JSONB,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX active_strategies_category_idx ON active_strategies(category);
```

#### strategy_performance
```sql
CREATE TABLE strategy_performance (
    strategy_id VARCHAR(50) PRIMARY KEY,
    sharpe_ratio DECIMAL(5, 3) NOT NULL,
    sortino_ratio DECIMAL(5, 3) NOT NULL,
    max_drawdown DECIMAL(5, 3) NOT NULL,
    win_rate DECIMAL(5, 3) NOT NULL,
    profit_factor DECIMAL(5, 3) NOT NULL,
    average_trade DECIMAL(8, 6) NOT NULL,
    max_consecutive_losses INTEGER NOT NULL,
    monthly_returns JSONB,
    performance_by_regime JSONB,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);
```

#### latest_trade_decisions
```sql
CREATE TABLE latest_trade_decisions (
    id VARCHAR(50) PRIMARY KEY,
    instrument_id VARCHAR(20) NOT NULL,
    strategy_id VARCHAR(50) NOT NULL,
    decision_type VARCHAR(10) NOT NULL,
    confidence DECIMAL(4,3) NOT NULL,
    generated_at TIMESTAMP WITH TIME ZONE NOT NULL,
    timeframe VARCHAR(10) NOT NULL,
    price_at_decision DECIMAL(18, 8) NOT NULL,
    target_price DECIMAL(18, 8),
    stop_loss DECIMAL(18, 8),
    risk_reward_ratio DECIMAL(8, 4),
    position_sizing JSONB NOT NULL,
    reasoning JSONB,
    execution_window JSONB,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX latest_trade_decisions_instrument_id_idx ON latest_trade_decisions(instrument_id);
CREATE INDEX latest_trade_decisions_strategy_id_idx ON latest_trade_decisions(strategy_id);
CREATE INDEX latest_trade_decisions_decision_type_idx ON latest_trade_decisions(decision_type);
```

#### backtest_summaries
```sql
CREATE TABLE backtest_summaries (
    id VARCHAR(50) PRIMARY KEY,
    strategy_id VARCHAR(50) NOT NULL,
    instruments JSONB NOT NULL,
    start_date TIMESTAMP WITH TIME ZONE NOT NULL,
    end_date TIMESTAMP WITH TIME ZONE NOT NULL,
    initial_capital DECIMAL(18, 2) NOT NULL,
    final_capital DECIMAL(18, 2) NOT NULL,
    total_return DECIMAL(8, 6) NOT NULL,
    annualized_return DECIMAL(8, 6) NOT NULL,
    sharpe_ratio DECIMAL(5, 3) NOT NULL,
    max_drawdown DECIMAL(5, 3) NOT NULL,
    win_rate DECIMAL(5, 3) NOT NULL,
    parameters JSONB NOT NULL,
    trade_count INTEGER NOT NULL,
    completed_at TIMESTAMP WITH TIME ZONE NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX backtest_summaries_strategy_id_idx ON backtest_summaries(strategy_id);
CREATE INDEX backtest_summaries_sharpe_ratio_idx ON backtest_summaries(sharpe_ratio DESC);
```

#### optimization_summaries
```sql
CREATE TABLE optimization_summaries (
    id VARCHAR(50) PRIMARY KEY,
    strategy_id VARCHAR(50) NOT NULL,
    instruments JSONB NOT NULL,
    date_range VARCHAR(50) NOT NULL,
    optimization_metric VARCHAR(30) NOT NULL,
    iterations_completed INTEGER NOT NULL,
    best_parameters JSONB NOT NULL,
    best_performance JSONB NOT NULL,
    improvement_percentage DECIMAL(5, 2) NOT NULL,
    completed_at TIMESTAMP WITH TIME ZONE NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX optimization_summaries_strategy_id_idx ON optimization_summaries(strategy_id);
```

## Place in Workflow

The Trading Strategy Service is a critical component of the Prediction and Decision Workflow and serves as the primary generator of trade decisions. It:

1. **Receives predictions** from the ML Prediction Service
2. **Incorporates risk metrics** from the Risk Analysis Service
3. **Applies trading strategies** to the incoming data
4. **Generates trade decisions** with confidence scores and reasoning
5. **Calculates position sizing** based on risk parameters
6. **Determines optimal execution timing**
7. **Distributes trade decisions** to downstream services

The service interacts with:
- **Market Data Service** (input): Provides market data for strategy execution
- **Technical Analysis Service** (input): Supplies technical indicators for strategy rules
- **ML Prediction Service** (input): Provides price predictions with confidence intervals
- **Risk Analysis Service** (input): Supplies risk metrics for position sizing
- **Order Management Service** (output): Receives trade decisions for execution
- **Portfolio Management Service** (output): Uses trade decisions for portfolio adjustments
- **Reporting Service** (output): Includes strategy performance in reports and dashboards
- **Notification Service** (output): Sends alerts for significant trade signals

## Project Plan

### Phase 1: Foundation (Weeks 1-4)

#### Week 1: Setup and Infrastructure
- Set up development environment and CI/CD pipeline
- Configure Kubernetes infrastructure for the service
- Set up PostgreSQL for data storage
- Implement basic service skeleton with health checks

#### Week 2: Strategy Framework
- Develop strategy interface and base classes
- Implement strategy registration and discovery
- Create parameter management system
- Build strategy execution engine

#### Week 3: Core Strategies
- Implement trend-following strategies
- Develop momentum-based strategies
- Create mean-reversion strategies
- Build multi-factor strategies

#### Week 4: Basic API Implementation
- Implement REST API endpoints for strategies
- Create gRPC service for real-time updates
- Develop basic authentication and rate limiting
- Write comprehensive API documentation

### Phase 2: Enhancement (Weeks 5-8)

#### Week 5: Backtesting Engine
- Implement historical data backtesting
- Develop performance metrics calculation
- Create visualization components
- Build parameter sensitivity analysis

#### Week 6: Position Sizing and Risk Management
- Implement position sizing algorithms
- Develop stop-loss and take-profit management
- Create portfolio-level risk controls
- Build dynamic risk adjustment

#### Week 7: Strategy Optimization
- Implement grid search optimization
- Develop genetic algorithm optimization
- Create walk-forward optimization
- Build optimization performance analysis

#### Week 8: Integration and Testing
- Integrate with upstream data and prediction services
- Connect with downstream execution services
- Perform load and stress testing
- Develop comprehensive monitoring and alerting

### Phase 3: Production Readiness (Weeks 9-12)

#### Week 9: Documentation and Training
- Create detailed service documentation
- Develop operational runbooks
- Prepare training materials
- Conduct knowledge transfer sessions

#### Week 10: Performance Optimization
- Optimize strategy execution speed
- Implement parallel backtesting
- Enhance database query performance
- Fine-tune resource utilization

#### Week 11: Advanced Strategy Features
- Implement machine learning-enhanced strategies
- Develop adaptive parameter adjustment
- Create regime-switching strategies
- Build ensemble strategy combinations

#### Week 12: Final Preparation and Deployment
- Conduct final integration testing
- Prepare production deployment plan
- Execute staged rollout
- Monitor production performance

### Ongoing Maintenance
- Regular strategy performance monitoring
- Strategy improvement and optimization
- New strategy development and testing
- Performance benchmarking and tuning