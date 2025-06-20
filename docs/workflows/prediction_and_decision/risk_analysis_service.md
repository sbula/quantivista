# Risk Analysis Service

## Purpose
The Risk Analysis Service is responsible for calculating comprehensive risk metrics for instruments, portfolios, and strategies with real-time monitoring. It provides quantitative risk assessments that enable informed trading decisions, portfolio optimization, and risk management across the platform.

## Strict Boundaries

### Responsibilities
- Multi-dimensional risk calculation for financial instruments
- Stress testing and scenario analysis
- Correlation and dependency modeling
- Risk limit monitoring and alerting
- Value at Risk (VaR) and Conditional VaR (CVaR) computation
- Volatility and drawdown forecasting
- Risk exposure analysis by various dimensions

### Non-Responsibilities
- Does NOT make trading decisions
- Does NOT execute trades
- Does NOT collect or normalize market data
- Does NOT generate price predictions
- Does NOT manage user preferences or authentication
- Does NOT perform portfolio optimization

## API Design (API-First)

### REST API

#### GET /api/v1/risk/instruments/{instrument_id}
Retrieves risk metrics for a specific instrument.

**Path Parameters:**
- `instrument_id` (string, required): Instrument identifier

**Query Parameters:**
- `metrics` (string, optional): Comma-separated list of risk metrics (volatility, var, cvar, beta, etc.)
- `confidence_level` (float, optional): Confidence level for VaR calculations (default: 0.95)
- `time_horizon` (string, optional): Time horizon for risk metrics (1d, 5d, 10d, 20d, default: 1d)
- `include_history` (boolean, optional): Include historical risk metrics (default: false)
- `history_days` (integer, optional): Number of days of historical data (default: 30)

**Response:**
```json
{
  "instrument_id": "AAPL",
  "as_of": "2025-06-20T16:00:00Z",
  "time_horizon": "1d",
  "confidence_level": 0.95,
  "risk_metrics": {
    "volatility": {
      "value": 0.0185,
      "annualized": 0.2925,
      "percentile": 65
    },
    "var": {
      "value": 2.45,
      "percentage": 0.0162,
      "method": "historical"
    },
    "cvar": {
      "value": 3.75,
      "percentage": 0.0248,
      "method": "historical"
    },
    "beta": {
      "value": 1.15,
      "benchmark": "SPY",
      "r_squared": 0.78
    },
    "max_drawdown": {
      "value": 0.0325,
      "days_to_recovery": 4.5
    }
  },
  "correlations": {
    "SPY": 0.82,
    "QQQ": 0.88,
    "MSFT": 0.75
  },
  "risk_decomposition": {
    "market_risk": 0.65,
    "sector_risk": 0.25,
    "specific_risk": 0.10
  },
  "historical_metrics": [
    {
      "date": "2025-06-19",
      "volatility": 0.0182,
      "var": 2.40,
      "cvar": 3.70
    },
    {
      "date": "2025-06-18",
      "volatility": 0.0178,
      "var": 2.35,
      "cvar": 3.65
    }
  ]
}
```

#### GET /api/v1/risk/portfolios/{portfolio_id}
Retrieves risk metrics for a portfolio.

**Path Parameters:**
- `portfolio_id` (string, required): Portfolio identifier

**Query Parameters:**
- `metrics` (string, optional): Comma-separated list of risk metrics
- `confidence_level` (float, optional): Confidence level for VaR calculations (default: 0.95)
- `time_horizon` (string, optional): Time horizon for risk metrics (1d, 5d, 10d, 20d, default: 1d)
- `include_positions` (boolean, optional): Include position-level risk metrics (default: false)
- `include_stress_tests` (boolean, optional): Include stress test results (default: false)

**Response:**
```json
{
  "portfolio_id": "port-12345",
  "as_of": "2025-06-20T16:00:00Z",
  "time_horizon": "1d",
  "confidence_level": 0.95,
  "total_value": 1250000.00,
  "risk_metrics": {
    "volatility": {
      "value": 0.0125,
      "annualized": 0.1975,
      "percentile": 45
    },
    "var": {
      "value": 18750.00,
      "percentage": 0.0150,
      "method": "historical"
    },
    "cvar": {
      "value": 25000.00,
      "percentage": 0.0200,
      "method": "historical"
    },
    "sharpe_ratio": 1.85,
    "diversification_ratio": 1.45,
    "max_drawdown": {
      "value": 0.0275,
      "days_to_recovery": 5.5
    }
  },
  "risk_exposures": {
    "sector": {
      "Technology": 0.35,
      "Healthcare": 0.25,
      "Financials": 0.20,
      "Consumer": 0.15,
      "Other": 0.05
    },
    "geography": {
      "North America": 0.65,
      "Europe": 0.20,
      "Asia": 0.15
    },
    "asset_class": {
      "Equity": 0.80,
      "Fixed Income": 0.15,
      "Cash": 0.05
    }
  },
  "stress_tests": [
    {
      "scenario": "Market Crash",
      "description": "Equity markets down 20%, volatility up 100%",
      "impact": {
        "value": -187500.00,
        "percentage": -0.15
      }
    },
    {
      "scenario": "Interest Rate Spike",
      "description": "Interest rates up 100 basis points",
      "impact": {
        "value": -62500.00,
        "percentage": -0.05
      }
    }
  ],
  "position_risks": [
    {
      "instrument_id": "AAPL",
      "weight": 0.08,
      "contribution_to_risk": 0.12,
      "var": 3750.00,
      "marginal_var": 0.0018
    },
    {
      "instrument_id": "MSFT",
      "weight": 0.07,
      "contribution_to_risk": 0.10,
      "var": 3125.00,
      "marginal_var": 0.0015
    }
  ]
}
```

#### POST /api/v1/risk/scenarios
Creates and runs a custom risk scenario.

**Request Body:**
```json
{
  "name": "Custom Tech Correction",
  "description": "Technology sector down 15%, other sectors down 5%",
  "market_moves": [
    {
      "factor": "sector",
      "name": "Technology",
      "move": -0.15
    },
    {
      "factor": "sector",
      "name": "Healthcare",
      "move": -0.05
    },
    {
      "factor": "volatility",
      "name": "VIX",
      "move": 0.50
    }
  ],
  "apply_to": {
    "portfolios": ["port-12345", "port-67890"],
    "instruments": ["AAPL", "MSFT", "GOOGL"]
  }
}
```

**Response:**
```json
{
  "scenario_id": "scen-12345",
  "name": "Custom Tech Correction",
  "created_at": "2025-06-20T16:15:00Z",
  "results": {
    "portfolios": [
      {
        "portfolio_id": "port-12345",
        "impact": {
          "value": -125000.00,
          "percentage": -0.10
        },
        "risk_metrics_change": {
          "var": {
            "before": 18750.00,
            "after": 25000.00,
            "change_percentage": 0.33
          },
          "volatility": {
            "before": 0.0125,
            "after": 0.0175,
            "change_percentage": 0.40
          }
        }
      }
    ],
    "instruments": [
      {
        "instrument_id": "AAPL",
        "impact": {
          "price_change": -22.50,
          "percentage": -0.15
        },
        "risk_metrics_change": {
          "var": {
            "before": 2.45,
            "after": 3.25,
            "change_percentage": 0.33
          }
        }
      }
    ]
  }
}
```

### gRPC API

```protobuf
syntax = "proto3";

package risk_analysis.v1;

import "google/protobuf/timestamp.proto";

service RiskAnalysisService {
  // Get risk metrics for an instrument
  rpc GetInstrumentRisk(InstrumentRiskRequest) returns (InstrumentRiskResponse);
  
  // Get risk metrics for a portfolio
  rpc GetPortfolioRisk(PortfolioRiskRequest) returns (PortfolioRiskResponse);
  
  // Run a risk scenario
  rpc RunRiskScenario(RiskScenarioRequest) returns (RiskScenarioResponse);
  
  // Stream real-time risk updates
  rpc StreamRiskMetrics(StreamRiskRequest) returns (stream RiskMetricUpdate);
  
  // Check risk limits
  rpc CheckRiskLimits(RiskLimitCheckRequest) returns (RiskLimitCheckResponse);
}

message InstrumentRiskRequest {
  string instrument_id = 1;
  repeated string metrics = 2;
  double confidence_level = 3;
  string time_horizon = 4; // "1d", "5d", "10d", "20d"
  bool include_history = 5;
  int32 history_days = 6;
}

message InstrumentRiskResponse {
  string instrument_id = 1;
  google.protobuf.Timestamp as_of = 2;
  string time_horizon = 3;
  double confidence_level = 4;
  RiskMetrics risk_metrics = 5;
  map<string, double> correlations = 6;
  RiskDecomposition risk_decomposition = 7;
  repeated HistoricalRiskMetric historical_metrics = 8;
}

message RiskMetrics {
  VolatilityMetric volatility = 1;
  VaRMetric var = 2;
  VaRMetric cvar = 3;
  BetaMetric beta = 4;
  DrawdownMetric max_drawdown = 5;
}

message VolatilityMetric {
  double value = 1;
  double annualized = 2;
  int32 percentile = 3;
}

message VaRMetric {
  double value = 1;
  double percentage = 2;
  string method = 3;
}

message BetaMetric {
  double value = 1;
  string benchmark = 2;
  double r_squared = 3;
}

message DrawdownMetric {
  double value = 1;
  double days_to_recovery = 2;
}

message RiskDecomposition {
  double market_risk = 1;
  double sector_risk = 2;
  double specific_risk = 3;
}

message HistoricalRiskMetric {
  google.protobuf.Timestamp date = 1;
  double volatility = 2;
  double var = 3;
  double cvar = 4;
}

message PortfolioRiskRequest {
  string portfolio_id = 1;
  repeated string metrics = 2;
  double confidence_level = 3;
  string time_horizon = 4;
  bool include_positions = 5;
  bool include_stress_tests = 6;
}

message PortfolioRiskResponse {
  string portfolio_id = 1;
  google.protobuf.Timestamp as_of = 2;
  string time_horizon = 3;
  double confidence_level = 4;
  double total_value = 5;
  RiskMetrics risk_metrics = 6;
  RiskExposures risk_exposures = 7;
  repeated StressTestResult stress_tests = 8;
  repeated PositionRisk position_risks = 9;
}

message RiskExposures {
  map<string, double> sector = 1;
  map<string, double> geography = 2;
  map<string, double> asset_class = 3;
}

message StressTestResult {
  string scenario = 1;
  string description = 2;
  Impact impact = 3;
}

message Impact {
  double value = 1;
  double percentage = 2;
}

message PositionRisk {
  string instrument_id = 1;
  double weight = 2;
  double contribution_to_risk = 3;
  double var = 4;
  double marginal_var = 5;
}

message RiskScenarioRequest {
  string name = 1;
  string description = 2;
  repeated MarketMove market_moves = 3;
  ScenarioTarget apply_to = 4;
}

message MarketMove {
  string factor = 1;
  string name = 2;
  double move = 3;
}

message ScenarioTarget {
  repeated string portfolios = 1;
  repeated string instruments = 2;
}

message RiskScenarioResponse {
  string scenario_id = 1;
  string name = 2;
  google.protobuf.Timestamp created_at = 3;
  ScenarioResults results = 4;
}

message ScenarioResults {
  repeated PortfolioScenarioResult portfolios = 1;
  repeated InstrumentScenarioResult instruments = 2;
}

message PortfolioScenarioResult {
  string portfolio_id = 1;
  Impact impact = 2;
  RiskMetricsChange risk_metrics_change = 3;
}

message InstrumentScenarioResult {
  string instrument_id = 1;
  PriceImpact impact = 2;
  RiskMetricsChange risk_metrics_change = 3;
}

message PriceImpact {
  double price_change = 1;
  double percentage = 2;
}

message RiskMetricsChange {
  MetricChange var = 1;
  MetricChange volatility = 2;
}

message MetricChange {
  double before = 1;
  double after = 2;
  double change_percentage = 3;
}

message StreamRiskRequest {
  repeated string instrument_ids = 1;
  repeated string portfolio_ids = 2;
  repeated string metrics = 3;
  double threshold_change_percentage = 4;
}

message RiskMetricUpdate {
  oneof entity {
    string instrument_id = 1;
    string portfolio_id = 2;
  }
  string metric_name = 3;
  double previous_value = 4;
  double current_value = 5;
  double change_percentage = 6;
  google.protobuf.Timestamp timestamp = 7;
}

message RiskLimitCheckRequest {
  string portfolio_id = 1;
  repeated string limit_types = 2;
}

message RiskLimitCheckResponse {
  string portfolio_id = 1;
  google.protobuf.Timestamp as_of = 2;
  repeated RiskLimitStatus limit_statuses = 3;
}

message RiskLimitStatus {
  string limit_type = 1;
  string description = 2;
  double limit_value = 3;
  double current_value = 4;
  double utilization_percentage = 5;
  string status = 6; // "OK", "WARNING", "BREACH"
  string action_required = 7;
}
```

## Data Model

### Core Entities

#### RiskMetric
Represents a calculated risk measure for an instrument or portfolio.

**Attributes:**
- `entity_id` (string): Reference to the instrument or portfolio
- `entity_type` (enum): Type of entity (instrument, portfolio, strategy)
- `metric_type` (enum): Type of risk metric (volatility, var, cvar, beta, etc.)
- `time_horizon` (enum): Time horizon for the metric (1d, 5d, 10d, 20d)
- `confidence_level` (float): Confidence level for the metric (if applicable)
- `value` (decimal): Calculated value of the metric
- `percentage` (decimal): Percentage representation (if applicable)
- `calculation_timestamp` (datetime): When the metric was calculated
- `calculation_method` (string): Method used for calculation
- `parameters` (map): Additional parameters used in calculation

#### RiskScenario
Represents a stress test or scenario analysis.

**Attributes:**
- `id` (string): Unique identifier for the scenario
- `name` (string): Human-readable name
- `description` (string): Detailed description
- `market_moves` (array): Market factor movements defined in the scenario
- `created_by` (string): User who created the scenario
- `created_at` (datetime): When the scenario was created
- `is_system` (boolean): Whether it's a system-defined scenario
- `is_active` (boolean): Whether the scenario is active

#### ScenarioResult
Represents the result of applying a risk scenario.

**Attributes:**
- `scenario_id` (string): Reference to the scenario
- `entity_id` (string): Reference to the affected entity (instrument, portfolio)
- `entity_type` (enum): Type of entity
- `run_timestamp` (datetime): When the scenario was run
- `impact_value` (decimal): Absolute impact value
- `impact_percentage` (decimal): Percentage impact
- `risk_metrics_before` (map): Risk metrics before the scenario
- `risk_metrics_after` (map): Risk metrics after the scenario
- `details` (map): Additional detailed results

#### RiskExposure
Represents a risk exposure along a specific dimension.

**Attributes:**
- `entity_id` (string): Reference to the instrument or portfolio
- `entity_type` (enum): Type of entity (instrument, portfolio, strategy)
- `dimension` (enum): Exposure dimension (sector, geography, asset_class, etc.)
- `category` (string): Specific category within the dimension
- `exposure_value` (decimal): Absolute exposure value
- `exposure_percentage` (decimal): Percentage exposure
- `benchmark_exposure` (decimal): Benchmark exposure for comparison
- `active_exposure` (decimal): Active exposure (difference from benchmark)
- `calculation_timestamp` (datetime): When the exposure was calculated

#### RiskLimit
Represents a risk limit defined for monitoring.

**Attributes:**
- `id` (string): Unique identifier for the limit
- `entity_id` (string): Reference to the entity (instrument, portfolio, strategy)
- `entity_type` (enum): Type of entity
- `limit_type` (enum): Type of limit (var_limit, exposure_limit, concentration_limit, etc.)
- `metric` (string): Risk metric being limited
- `threshold_value` (decimal): Limit threshold value
- `warning_percentage` (decimal): Warning level as percentage of threshold
- `action_required` (string): Action required when limit is breached
- `is_active` (boolean): Whether the limit is active
- `created_at` (datetime): When the limit was created
- `updated_at` (datetime): When the limit was last updated

## DB Schema (CQRS Pattern)

### Write Schema (Command Side)

#### risk_metrics
```sql
CREATE TABLE risk_metrics (
    id SERIAL PRIMARY KEY,
    entity_id VARCHAR(50) NOT NULL,
    entity_type VARCHAR(20) NOT NULL,
    metric_type VARCHAR(30) NOT NULL,
    time_horizon VARCHAR(10) NOT NULL,
    confidence_level DECIMAL(4,3),
    value DECIMAL(18, 8) NOT NULL,
    percentage DECIMAL(8, 6),
    calculation_timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    calculation_method VARCHAR(30) NOT NULL,
    parameters JSONB,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT risk_metrics_entity_metric_horizon_idx UNIQUE (entity_id, entity_type, metric_type, time_horizon, confidence_level, calculation_timestamp)
);

CREATE INDEX risk_metrics_entity_idx ON risk_metrics(entity_id, entity_type);
CREATE INDEX risk_metrics_calculation_timestamp_idx ON risk_metrics(calculation_timestamp);
```

#### risk_correlations
```sql
CREATE TABLE risk_correlations (
    id SERIAL PRIMARY KEY,
    entity_id_1 VARCHAR(50) NOT NULL,
    entity_id_2 VARCHAR(50) NOT NULL,
    entity_type VARCHAR(20) NOT NULL,
    time_horizon VARCHAR(10) NOT NULL,
    correlation_value DECIMAL(5,4) NOT NULL,
    calculation_timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    calculation_method VARCHAR(30) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT risk_correlations_entities_idx UNIQUE (entity_id_1, entity_id_2, entity_type, time_horizon, calculation_timestamp)
);

CREATE INDEX risk_correlations_entity_1_idx ON risk_correlations(entity_id_1);
CREATE INDEX risk_correlations_entity_2_idx ON risk_correlations(entity_id_2);
```

#### risk_scenarios
```sql
CREATE TABLE risk_scenarios (
    id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    market_moves JSONB NOT NULL,
    created_by VARCHAR(50),
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    is_system BOOLEAN NOT NULL DEFAULT FALSE,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX risk_scenarios_is_active_idx ON risk_scenarios(is_active);
```

#### scenario_results
```sql
CREATE TABLE scenario_results (
    id SERIAL PRIMARY KEY,
    scenario_id VARCHAR(50) NOT NULL REFERENCES risk_scenarios(id),
    entity_id VARCHAR(50) NOT NULL,
    entity_type VARCHAR(20) NOT NULL,
    run_timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    impact_value DECIMAL(18, 8) NOT NULL,
    impact_percentage DECIMAL(8, 6) NOT NULL,
    risk_metrics_before JSONB NOT NULL,
    risk_metrics_after JSONB NOT NULL,
    details JSONB,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT scenario_results_scenario_entity_idx UNIQUE (scenario_id, entity_id, entity_type, run_timestamp)
);

CREATE INDEX scenario_results_scenario_id_idx ON scenario_results(scenario_id);
CREATE INDEX scenario_results_entity_idx ON scenario_results(entity_id, entity_type);
```

#### risk_exposures
```sql
CREATE TABLE risk_exposures (
    id SERIAL PRIMARY KEY,
    entity_id VARCHAR(50) NOT NULL,
    entity_type VARCHAR(20) NOT NULL,
    dimension VARCHAR(30) NOT NULL,
    category VARCHAR(50) NOT NULL,
    exposure_value DECIMAL(18, 8) NOT NULL,
    exposure_percentage DECIMAL(8, 6) NOT NULL,
    benchmark_exposure DECIMAL(8, 6),
    active_exposure DECIMAL(8, 6),
    calculation_timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT risk_exposures_entity_dimension_category_idx UNIQUE (entity_id, entity_type, dimension, category, calculation_timestamp)
);

CREATE INDEX risk_exposures_entity_idx ON risk_exposures(entity_id, entity_type);
CREATE INDEX risk_exposures_dimension_idx ON risk_exposures(dimension, category);
```

#### risk_limits
```sql
CREATE TABLE risk_limits (
    id VARCHAR(50) PRIMARY KEY,
    entity_id VARCHAR(50) NOT NULL,
    entity_type VARCHAR(20) NOT NULL,
    limit_type VARCHAR(30) NOT NULL,
    metric VARCHAR(30) NOT NULL,
    threshold_value DECIMAL(18, 8) NOT NULL,
    warning_percentage DECIMAL(5,4) NOT NULL DEFAULT 0.8,
    action_required TEXT,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT risk_limits_entity_type_metric_idx UNIQUE (entity_id, entity_type, limit_type, metric)
);

CREATE INDEX risk_limits_entity_idx ON risk_limits(entity_id, entity_type);
CREATE INDEX risk_limits_is_active_idx ON risk_limits(is_active);
```

#### risk_limit_breaches
```sql
CREATE TABLE risk_limit_breaches (
    id SERIAL PRIMARY KEY,
    limit_id VARCHAR(50) NOT NULL REFERENCES risk_limits(id),
    breach_timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    current_value DECIMAL(18, 8) NOT NULL,
    utilization_percentage DECIMAL(8, 6) NOT NULL,
    severity VARCHAR(20) NOT NULL,
    is_acknowledged BOOLEAN NOT NULL DEFAULT FALSE,
    acknowledged_by VARCHAR(50),
    acknowledged_at TIMESTAMP WITH TIME ZONE,
    resolution_notes TEXT,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX risk_limit_breaches_limit_id_idx ON risk_limit_breaches(limit_id);
CREATE INDEX risk_limit_breaches_breach_timestamp_idx ON risk_limit_breaches(breach_timestamp);
CREATE INDEX risk_limit_breaches_is_acknowledged_idx ON risk_limit_breaches(is_acknowledged);
```

### Read Schema (Query Side)

#### current_instrument_risk
```sql
CREATE TABLE current_instrument_risk (
    instrument_id VARCHAR(50) NOT NULL,
    time_horizon VARCHAR(10) NOT NULL,
    confidence_level DECIMAL(4,3) NOT NULL,
    volatility DECIMAL(8, 6) NOT NULL,
    volatility_annualized DECIMAL(8, 6) NOT NULL,
    var_value DECIMAL(18, 8) NOT NULL,
    var_percentage DECIMAL(8, 6) NOT NULL,
    cvar_value DECIMAL(18, 8) NOT NULL,
    cvar_percentage DECIMAL(8, 6) NOT NULL,
    beta DECIMAL(5, 3),
    beta_benchmark VARCHAR(20),
    max_drawdown DECIMAL(8, 6),
    risk_decomposition JSONB,
    calculation_timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    PRIMARY KEY (instrument_id, time_horizon, confidence_level)
);
```

#### current_portfolio_risk
```sql
CREATE TABLE current_portfolio_risk (
    portfolio_id VARCHAR(50) NOT NULL,
    time_horizon VARCHAR(10) NOT NULL,
    confidence_level DECIMAL(4,3) NOT NULL,
    total_value DECIMAL(18, 2) NOT NULL,
    volatility DECIMAL(8, 6) NOT NULL,
    volatility_annualized DECIMAL(8, 6) NOT NULL,
    var_value DECIMAL(18, 8) NOT NULL,
    var_percentage DECIMAL(8, 6) NOT NULL,
    cvar_value DECIMAL(18, 8) NOT NULL,
    cvar_percentage DECIMAL(8, 6) NOT NULL,
    sharpe_ratio DECIMAL(5, 3),
    diversification_ratio DECIMAL(5, 3),
    max_drawdown DECIMAL(8, 6),
    risk_exposures JSONB,
    calculation_timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    PRIMARY KEY (portfolio_id, time_horizon, confidence_level)
);
```

#### current_correlations
```sql
CREATE TABLE current_correlations (
    entity_id_1 VARCHAR(50) NOT NULL,
    entity_id_2 VARCHAR(50) NOT NULL,
    entity_type VARCHAR(20) NOT NULL,
    time_horizon VARCHAR(10) NOT NULL,
    correlation_value DECIMAL(5,4) NOT NULL,
    calculation_timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    PRIMARY KEY (entity_id_1, entity_id_2, entity_type, time_horizon)
);

CREATE INDEX current_correlations_entity_1_idx ON current_correlations(entity_id_1, correlation_value DESC);
```

#### risk_limit_status
```sql
CREATE TABLE risk_limit_status (
    limit_id VARCHAR(50) NOT NULL,
    entity_id VARCHAR(50) NOT NULL,
    entity_type VARCHAR(20) NOT NULL,
    limit_type VARCHAR(30) NOT NULL,
    metric VARCHAR(30) NOT NULL,
    threshold_value DECIMAL(18, 8) NOT NULL,
    current_value DECIMAL(18, 8) NOT NULL,
    utilization_percentage DECIMAL(8, 6) NOT NULL,
    status VARCHAR(20) NOT NULL,
    last_checked_at TIMESTAMP WITH TIME ZONE NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    PRIMARY KEY (limit_id)
);

CREATE INDEX risk_limit_status_entity_idx ON risk_limit_status(entity_id, entity_type);
CREATE INDEX risk_limit_status_status_idx ON risk_limit_status(status);
```

#### scenario_analysis_results
```sql
CREATE TABLE scenario_analysis_results (
    scenario_id VARCHAR(50) NOT NULL,
    entity_id VARCHAR(50) NOT NULL,
    entity_type VARCHAR(20) NOT NULL,
    scenario_name VARCHAR(100) NOT NULL,
    impact_value DECIMAL(18, 8) NOT NULL,
    impact_percentage DECIMAL(8, 6) NOT NULL,
    metrics_change JSONB NOT NULL,
    run_timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    PRIMARY KEY (scenario_id, entity_id, entity_type)
);

CREATE INDEX scenario_analysis_results_entity_idx ON scenario_analysis_results(entity_id, entity_type);
```

## Place in Workflow

The Risk Analysis Service is a critical component of the Prediction and Decision Workflow and serves as the primary provider of risk assessments. It:

1. **Receives market data** from the Market Data Service
2. **Incorporates technical indicators** from the Technical Analysis Service
3. **Integrates prediction uncertainty** from the ML Prediction Service
4. **Calculates comprehensive risk metrics** for instruments, portfolios, and strategies
5. **Performs stress testing and scenario analysis** to assess potential impacts
6. **Monitors risk limits** and generates alerts when thresholds are breached
7. **Distributes risk assessments** to downstream services

The service interacts with:
- **Market Data Service** (input): Provides market data for risk calculations
- **Technical Analysis Service** (input): Supplies technical indicators and correlations
- **ML Prediction Service** (input): Provides prediction uncertainty metrics
- **Trading Strategy Service** (output): Uses risk metrics for strategy decisions
- **Portfolio Optimization Service** (output): Incorporates risk assessments for portfolio construction
- **Order Management Service** (output): Uses risk checks for pre-trade validation
- **Reporting Service** (output): Includes risk metrics in reports and dashboards
- **Notification Service** (output): Sends alerts for risk limit breaches

## Project Plan

### Phase 1: Foundation (Weeks 1-4)

#### Week 1: Setup and Infrastructure
- Set up development environment and CI/CD pipeline
- Configure Kubernetes infrastructure for the service
- Set up PostgreSQL for data storage
- Implement basic service skeleton with health checks

#### Week 2: Core Risk Metrics
- Implement volatility calculation
- Develop Value at Risk (VaR) computation
- Create Conditional VaR (CVaR) calculation
- Build correlation analysis functionality

#### Week 3: Risk Decomposition and Exposure
- Implement risk factor decomposition
- Develop exposure analysis by dimension
- Create risk contribution calculation
- Build risk attribution functionality

#### Week 4: Basic API Implementation
- Implement REST API endpoints for risk metrics
- Create gRPC service for real-time updates
- Develop basic authentication and rate limiting
- Write comprehensive API documentation

### Phase 2: Enhancement (Weeks 5-8)

#### Week 5: Scenario Analysis
- Implement stress testing framework
- Develop historical scenario analysis
- Create hypothetical scenario modeling
- Build scenario management functionality

#### Week 6: Risk Limits and Monitoring
- Implement risk limit definition and management
- Develop limit monitoring and breach detection
- Create alerting and notification system
- Build limit breach resolution tracking

#### Week 7: Advanced Risk Metrics
- Implement expected shortfall calculation
- Develop tail risk metrics
- Create liquidity risk assessment
- Build concentration risk analysis

#### Week 8: Integration and Testing
- Integrate with upstream data services
- Connect with downstream decision services
- Perform load and stress testing
- Develop comprehensive monitoring and alerting

### Phase 3: Production Readiness (Weeks 9-12)

#### Week 9: Documentation and Training
- Create detailed service documentation
- Develop operational runbooks
- Prepare training materials
- Conduct knowledge transfer sessions

#### Week 10: Performance Optimization
- Optimize risk calculation algorithms
- Implement parallel processing for large portfolios
- Enhance database query performance
- Fine-tune resource utilization

#### Week 11: Advanced Risk Features
- Implement Monte Carlo simulation
- Develop extreme value theory models
- Create regime-switching risk models
- Build advanced correlation modeling

#### Week 12: Final Preparation and Deployment
- Conduct final integration testing
- Prepare production deployment plan
- Execute staged rollout
- Monitor production performance

### Ongoing Maintenance
- Regular algorithm improvements and optimizations
- Risk model calibration and validation
- Performance monitoring and tuning
- Database maintenance and optimization