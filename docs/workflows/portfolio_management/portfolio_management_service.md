# Portfolio Management Service

## Purpose
The Portfolio Management Service is responsible for tracking, analyzing, and managing investment portfolios across multiple strategies and asset classes. It provides comprehensive portfolio data management, performance tracking, risk assessment, and compliance monitoring to ensure optimal portfolio management and reporting capabilities.

## Strict Boundaries

### Responsibilities
- Real-time position tracking and reconciliation
- Portfolio performance calculation and attribution
- Compliance monitoring and reporting
- Corporate action processing and impact assessment
- Portfolio data management and storage
- Position reconciliation with broker statements
- Cash and margin management
- Tax lot tracking and tax reporting

### Non-Responsibilities
- Does NOT make trading decisions
- Does NOT execute trades
- Does NOT perform portfolio optimization (handled by Portfolio Optimization Service)
- Does NOT handle order management
- Does NOT perform risk calculations (relies on Risk Analysis Service)
- Does NOT manage user authentication or authorization
- Does NOT handle market data collection

## API Design (API-First)

### REST API

#### GET /api/v1/portfolios
Retrieves a list of portfolios.

**Query Parameters:**
- `user_id` (string, optional): Filter portfolios by user
- `status` (string, optional): Filter by status (active, archived)
- `page` (integer, optional): Page number for pagination
- `size` (integer, optional): Page size for pagination

**Response:**
```json
{
  "portfolios": [
    {
      "id": "port-123456",
      "name": "Growth Portfolio",
      "description": "Long-term growth strategy",
      "base_currency": "USD",
      "status": "active",
      "created_at": "2025-01-15T10:30:00Z",
      "updated_at": "2025-06-10T14:25:00Z",
      "total_value": 1250000.00,
      "cash_balance": 125000.00,
      "performance": {
        "daily": 0.75,
        "weekly": 2.25,
        "monthly": 3.50,
        "ytd": 12.75,
        "inception": 25.50
      }
    }
  ],
  "page": 1,
  "size": 10,
  "total_pages": 3,
  "total_elements": 25
}
```

#### GET /api/v1/portfolios/{portfolio_id}
Retrieves detailed information about a specific portfolio.

**Path Parameters:**
- `portfolio_id` (string, required): Portfolio identifier

**Response:**
```json
{
  "id": "port-123456",
  "name": "Growth Portfolio",
  "description": "Long-term growth strategy",
  "base_currency": "USD",
  "status": "active",
  "created_at": "2025-01-15T10:30:00Z",
  "updated_at": "2025-06-10T14:25:00Z",
  "total_value": 1250000.00,
  "cash_balance": 125000.00,
  "margin_used": 50000.00,
  "margin_available": 500000.00,
  "performance": {
    "daily": 0.75,
    "weekly": 2.25,
    "monthly": 3.50,
    "ytd": 12.75,
    "inception": 25.50
  },
  "risk_metrics": {
    "volatility": 12.5,
    "sharpe_ratio": 1.8,
    "sortino_ratio": 2.2,
    "max_drawdown": 15.3,
    "var_95": 22500.00,
    "cvar_95": 30000.00
  },
  "allocations": {
    "by_asset_class": {
      "equities": 65.0,
      "fixed_income": 20.0,
      "commodities": 10.0,
      "cash": 5.0
    },
    "by_sector": {
      "technology": 30.0,
      "healthcare": 15.0,
      "financials": 12.0,
      "consumer_discretionary": 8.0,
      "other": 35.0
    },
    "by_geography": {
      "north_america": 60.0,
      "europe": 25.0,
      "asia_pacific": 10.0,
      "emerging_markets": 5.0
    }
  }
}
```

#### GET /api/v1/portfolios/{portfolio_id}/positions
Retrieves positions for a specific portfolio.

**Path Parameters:**
- `portfolio_id` (string, required): Portfolio identifier

**Query Parameters:**
- `asset_class` (string, optional): Filter by asset class
- `sector` (string, optional): Filter by sector
- `page` (integer, optional): Page number for pagination
- `size` (integer, optional): Page size for pagination

**Response:**
```json
{
  "portfolio_id": "port-123456",
  "as_of": "2025-06-20T16:00:00Z",
  "positions": [
    {
      "instrument_id": "AAPL",
      "instrument_type": "equity",
      "quantity": 1000,
      "average_cost": 150.25,
      "current_price": 175.50,
      "market_value": 175500.00,
      "unrealized_pnl": 25250.00,
      "unrealized_pnl_percent": 16.8,
      "weight": 14.04,
      "sector": "technology",
      "industry": "consumer_electronics",
      "country": "us",
      "currency": "USD",
      "tax_lots": [
        {
          "lot_id": "lot-789012",
          "acquisition_date": "2025-01-20T10:15:00Z",
          "quantity": 500,
          "cost_basis": 145.50,
          "cost_basis_total": 72750.00
        },
        {
          "lot_id": "lot-789013",
          "acquisition_date": "2025-03-15T11:30:00Z",
          "quantity": 500,
          "cost_basis": 155.00,
          "cost_basis_total": 77500.00
        }
      ]
    }
  ],
  "page": 1,
  "size": 10,
  "total_pages": 5,
  "total_elements": 42
}
```

#### GET /api/v1/portfolios/{portfolio_id}/performance
Retrieves performance data for a specific portfolio.

**Path Parameters:**
- `portfolio_id` (string, required): Portfolio identifier

**Query Parameters:**
- `period` (string, optional): Time period (1d, 1w, 1m, 3m, 6m, 1y, ytd, all)
- `frequency` (string, optional): Data frequency (daily, weekly, monthly)
- `include_benchmark` (boolean, optional): Include benchmark comparison

**Response:**
```json
{
  "portfolio_id": "port-123456",
  "period": "1y",
  "frequency": "daily",
  "base_currency": "USD",
  "start_date": "2024-06-20T00:00:00Z",
  "end_date": "2025-06-20T00:00:00Z",
  "starting_value": 1000000.00,
  "ending_value": 1250000.00,
  "total_return": 25.0,
  "annualized_return": 25.0,
  "risk_metrics": {
    "volatility": 12.5,
    "sharpe_ratio": 1.8,
    "sortino_ratio": 2.2,
    "max_drawdown": 15.3,
    "max_drawdown_date": "2025-03-15T00:00:00Z",
    "var_95": 22500.00,
    "cvar_95": 30000.00
  },
  "performance_series": [
    {
      "date": "2024-06-20T00:00:00Z",
      "value": 1000000.00,
      "daily_return": 0.0,
      "cumulative_return": 0.0
    },
    {
      "date": "2024-06-21T00:00:00Z",
      "value": 1005000.00,
      "daily_return": 0.5,
      "cumulative_return": 0.5
    }
  ],
  "benchmark_comparison": {
    "benchmark_id": "SPY",
    "benchmark_name": "S&P 500 ETF",
    "benchmark_return": 20.0,
    "excess_return": 5.0,
    "tracking_error": 3.5,
    "information_ratio": 1.43,
    "beta": 0.95,
    "alpha": 6.0,
    "r_squared": 0.85,
    "benchmark_series": [
      {
        "date": "2024-06-20T00:00:00Z",
        "value": 100.0,
        "daily_return": 0.0,
        "cumulative_return": 0.0
      },
      {
        "date": "2024-06-21T00:00:00Z",
        "value": 100.4,
        "daily_return": 0.4,
        "cumulative_return": 0.4
      }
    ]
  }
}
```

#### GET /api/v1/portfolios/{portfolio_id}/attribution
Retrieves performance attribution data for a specific portfolio.

**Path Parameters:**
- `portfolio_id` (string, required): Portfolio identifier

**Query Parameters:**
- `period` (string, optional): Time period (1m, 3m, 6m, 1y, ytd, all)
- `level` (string, optional): Attribution level (asset_class, sector, security)

**Response:**
```json
{
  "portfolio_id": "port-123456",
  "period": "1y",
  "base_currency": "USD",
  "start_date": "2024-06-20T00:00:00Z",
  "end_date": "2025-06-20T00:00:00Z",
  "total_return": 25.0,
  "attribution": {
    "by_asset_class": [
      {
        "asset_class": "equities",
        "allocation_effect": 2.5,
        "selection_effect": 10.0,
        "interaction_effect": 0.5,
        "total_effect": 13.0,
        "average_weight": 65.0,
        "return": 30.0
      },
      {
        "asset_class": "fixed_income",
        "allocation_effect": -1.0,
        "selection_effect": 5.0,
        "interaction_effect": 0.2,
        "total_effect": 4.2,
        "average_weight": 20.0,
        "return": 8.0
      }
    ],
    "by_sector": [
      {
        "sector": "technology",
        "allocation_effect": 3.0,
        "selection_effect": 8.0,
        "interaction_effect": 0.5,
        "total_effect": 11.5,
        "average_weight": 30.0,
        "return": 35.0
      }
    ],
    "top_contributors": [
      {
        "instrument_id": "AAPL",
        "instrument_name": "Apple Inc.",
        "contribution": 3.5,
        "average_weight": 14.0,
        "return": 40.0
      }
    ],
    "bottom_contributors": [
      {
        "instrument_id": "XYZ",
        "instrument_name": "Example Corp",
        "contribution": -1.2,
        "average_weight": 2.0,
        "return": -15.0
      }
    ]
  }
}
```

#### GET /api/v1/portfolios/{portfolio_id}/compliance
Retrieves compliance status and alerts for a specific portfolio.

**Path Parameters:**
- `portfolio_id` (string, required): Portfolio identifier

**Query Parameters:**
- `status` (string, optional): Filter by status (compliant, warning, violation)
- `category` (string, optional): Filter by category (concentration, leverage, etc.)

**Response:**
```json
{
  "portfolio_id": "port-123456",
  "as_of": "2025-06-20T16:00:00Z",
  "overall_status": "warning",
  "rules": [
    {
      "rule_id": "rule-123",
      "name": "Sector Concentration",
      "description": "No sector should exceed 35% of portfolio",
      "category": "concentration",
      "status": "warning",
      "current_value": 30.0,
      "limit": 35.0,
      "details": {
        "sector": "technology",
        "current_allocation": 30.0,
        "max_allocation": 35.0
      }
    },
    {
      "rule_id": "rule-124",
      "name": "Maximum Leverage",
      "description": "Portfolio leverage should not exceed 1.5x",
      "category": "leverage",
      "status": "compliant",
      "current_value": 1.2,
      "limit": 1.5
    }
  ]
}
```

### gRPC API

```protobuf
syntax = "proto3";

package portfolio_management.v1;

import "google/protobuf/timestamp.proto";

service PortfolioManagementService {
  // Portfolio operations
  rpc GetPortfolios(GetPortfoliosRequest) returns (GetPortfoliosResponse);
  rpc GetPortfolio(GetPortfolioRequest) returns (GetPortfolioResponse);
  rpc CreatePortfolio(CreatePortfolioRequest) returns (CreatePortfolioResponse);
  rpc UpdatePortfolio(UpdatePortfolioRequest) returns (UpdatePortfolioResponse);

  // Position operations
  rpc GetPositions(GetPositionsRequest) returns (GetPositionsResponse);
  rpc UpdatePosition(UpdatePositionRequest) returns (UpdatePositionResponse);

  // Performance operations
  rpc GetPerformance(GetPerformanceRequest) returns (GetPerformanceResponse);
  rpc GetAttribution(GetAttributionRequest) returns (GetAttributionResponse);

  // Compliance operations
  rpc GetComplianceStatus(GetComplianceStatusRequest) returns (GetComplianceStatusResponse);

  // Streaming updates
  rpc StreamPortfolioUpdates(StreamPortfolioUpdatesRequest) returns (stream PortfolioUpdate);
}

message GetPortfoliosRequest {
  string user_id = 1;
  string status = 2;
  int32 page = 3;
  int32 size = 4;
}

message GetPortfoliosResponse {
  repeated PortfolioSummary portfolios = 1;
  int32 page = 2;
  int32 size = 3;
  int32 total_pages = 4;
  int32 total_elements = 5;
}

message PortfolioSummary {
  string id = 1;
  string name = 2;
  string description = 3;
  string base_currency = 4;
  string status = 5;
  google.protobuf.Timestamp created_at = 6;
  google.protobuf.Timestamp updated_at = 7;
  double total_value = 8;
  double cash_balance = 9;
  PerformanceSummary performance = 10;
}

message PerformanceSummary {
  double daily = 1;
  double weekly = 2;
  double monthly = 3;
  double ytd = 4;
  double inception = 5;
}

message GetPortfolioRequest {
  string portfolio_id = 1;
}

message GetPortfolioResponse {
  Portfolio portfolio = 1;
}

message Portfolio {
  string id = 1;
  string name = 2;
  string description = 3;
  string base_currency = 4;
  string status = 5;
  google.protobuf.Timestamp created_at = 6;
  google.protobuf.Timestamp updated_at = 7;
  double total_value = 8;
  double cash_balance = 9;
  double margin_used = 10;
  double margin_available = 11;
  PerformanceSummary performance = 12;
  RiskMetrics risk_metrics = 13;
  Allocations allocations = 14;
}

message RiskMetrics {
  double volatility = 1;
  double sharpe_ratio = 2;
  double sortino_ratio = 3;
  double max_drawdown = 4;
  double var_95 = 5;
  double cvar_95 = 6;
}

message Allocations {
  map<string, double> by_asset_class = 1;
  map<string, double> by_sector = 2;
  map<string, double> by_geography = 3;
}

message GetPositionsRequest {
  string portfolio_id = 1;
  string asset_class = 2;
  string sector = 3;
  int32 page = 4;
  int32 size = 5;
}

message GetPositionsResponse {
  string portfolio_id = 1;
  google.protobuf.Timestamp as_of = 2;
  repeated Position positions = 3;
  int32 page = 4;
  int32 size = 5;
  int32 total_pages = 6;
  int32 total_elements = 7;
}

message Position {
  string instrument_id = 1;
  string instrument_type = 2;
  double quantity = 3;
  double average_cost = 4;
  double current_price = 5;
  double market_value = 6;
  double unrealized_pnl = 7;
  double unrealized_pnl_percent = 8;
  double weight = 9;
  string sector = 10;
  string industry = 11;
  string country = 12;
  string currency = 13;
  repeated TaxLot tax_lots = 14;
}

message TaxLot {
  string lot_id = 1;
  google.protobuf.Timestamp acquisition_date = 2;
  double quantity = 3;
  double cost_basis = 4;
  double cost_basis_total = 5;
}

message GetPerformanceRequest {
  string portfolio_id = 1;
  string period = 2;
  string frequency = 3;
  bool include_benchmark = 4;
}

message GetPerformanceResponse {
  string portfolio_id = 1;
  string period = 2;
  string frequency = 3;
  string base_currency = 4;
  google.protobuf.Timestamp start_date = 5;
  google.protobuf.Timestamp end_date = 6;
  double starting_value = 7;
  double ending_value = 8;
  double total_return = 9;
  double annualized_return = 10;
  RiskMetrics risk_metrics = 11;
  repeated PerformancePoint performance_series = 12;
  BenchmarkComparison benchmark_comparison = 13;
}

message PerformancePoint {
  google.protobuf.Timestamp date = 1;
  double value = 2;
  double daily_return = 3;
  double cumulative_return = 4;
}

message BenchmarkComparison {
  string benchmark_id = 1;
  string benchmark_name = 2;
  double benchmark_return = 3;
  double excess_return = 4;
  double tracking_error = 5;
  double information_ratio = 6;
  double beta = 7;
  double alpha = 8;
  double r_squared = 9;
  repeated PerformancePoint benchmark_series = 10;
}

message CreatePortfolioRequest {
  string name = 1;
  string description = 2;
  string base_currency = 3;
  map<string, string> metadata = 4;
}

message CreatePortfolioResponse {
  Portfolio portfolio = 1;
}

message UpdatePortfolioRequest {
  string portfolio_id = 1;
  string name = 2;
  string description = 3;
  string status = 4;
  map<string, string> metadata = 5;
}

message UpdatePortfolioResponse {
  Portfolio portfolio = 1;
}

message UpdatePositionRequest {
  string portfolio_id = 1;
  string instrument_id = 2;
  double quantity = 3;
  double average_cost = 4;
}

message UpdatePositionResponse {
  Position position = 1;
}

message GetAttributionRequest {
  string portfolio_id = 1;
  string period = 2;
  string level = 3;
}

message GetAttributionResponse {
  string portfolio_id = 1;
  string period = 2;
  string base_currency = 3;
  google.protobuf.Timestamp start_date = 4;
  google.protobuf.Timestamp end_date = 5;
  double total_return = 6;
  AttributionData attribution = 7;
}

message AttributionData {
  repeated AttributionItem by_asset_class = 1;
  repeated AttributionItem by_sector = 2;
  repeated ContributorItem top_contributors = 3;
  repeated ContributorItem bottom_contributors = 4;
}

message AttributionItem {
  string category = 1;
  double allocation_effect = 2;
  double selection_effect = 3;
  double interaction_effect = 4;
  double total_effect = 5;
  double average_weight = 6;
  double return = 7;
}

message ContributorItem {
  string instrument_id = 1;
  string instrument_name = 2;
  double contribution = 3;
  double average_weight = 4;
  double return = 5;
}

message GetComplianceStatusRequest {
  string portfolio_id = 1;
  string status = 2;
  string category = 3;
}

message GetComplianceStatusResponse {
  string portfolio_id = 1;
  google.protobuf.Timestamp as_of = 2;
  string overall_status = 3;
  repeated ComplianceRule rules = 4;
}

message ComplianceRule {
  string rule_id = 1;
  string name = 2;
  string description = 3;
  string category = 4;
  string status = 5;
  double current_value = 6;
  double limit = 7;
  map<string, string> details = 8;
}

message StreamPortfolioUpdatesRequest {
  repeated string portfolio_ids = 1;
  bool include_positions = 2;
  bool include_performance = 3;
}

message PortfolioUpdate {
  string portfolio_id = 1;
  google.protobuf.Timestamp timestamp = 2;
  double total_value = 3;
  double cash_balance = 4;
  PerformanceSummary performance = 5;
  repeated PositionUpdate position_updates = 6;
}

message PositionUpdate {
  string instrument_id = 1;
  double quantity = 2;
  double current_price = 3;
  double market_value = 4;
  double unrealized_pnl = 5;
}
```

## Data Model

### Core Entities

#### Portfolio
Represents an investment portfolio with positions, performance, and risk metrics.

**Attributes:**
- `id` (string): Unique identifier for the portfolio
- `name` (string): Display name for the portfolio
- `description` (string): Detailed description
- `base_currency` (string): Base currency for the portfolio
- `status` (enum): Portfolio status (active, archived, etc.)
- `created_at` (datetime): Creation timestamp
- `updated_at` (datetime): Last update timestamp
- `total_value` (decimal): Current total market value
- `cash_balance` (decimal): Available cash balance
- `margin_used` (decimal): Amount of margin currently used
- `margin_available` (decimal): Available margin
- `performance` (object): Performance summary metrics
- `risk_metrics` (object): Risk assessment metrics
- `allocations` (object): Current portfolio allocations

#### Position
Represents a holding of a specific instrument within a portfolio.

**Attributes:**
- `portfolio_id` (string): Reference to the portfolio
- `instrument_id` (string): Reference to the instrument
- `instrument_type` (enum): Type of instrument (equity, bond, etc.)
- `quantity` (decimal): Number of units held
- `average_cost` (decimal): Average cost per unit
- `current_price` (decimal): Current market price
- `market_value` (decimal): Current total market value
- `unrealized_pnl` (decimal): Unrealized profit/loss
- `unrealized_pnl_percent` (decimal): Percentage profit/loss
- `weight` (decimal): Position weight in the portfolio
- `sector` (string): Industry sector
- `industry` (string): Specific industry
- `country` (string): Country of listing
- `currency` (string): Trading currency

#### TaxLot
Represents a specific purchase lot for tax tracking purposes.

**Attributes:**
- `lot_id` (string): Unique identifier for the tax lot
- `position_id` (string): Reference to the position
- `acquisition_date` (datetime): Purchase date
- `quantity` (decimal): Number of units in this lot
- `cost_basis` (decimal): Cost per unit
- `cost_basis_total` (decimal): Total cost for this lot
- `tax_status` (enum): Tax status (short-term, long-term)

#### Transaction
Represents a transaction affecting a portfolio.

**Attributes:**
- `transaction_id` (string): Unique identifier
- `portfolio_id` (string): Reference to the portfolio
- `instrument_id` (string): Reference to the instrument
- `transaction_type` (enum): Type of transaction (buy, sell, dividend, etc.)
- `quantity` (decimal): Number of units
- `price` (decimal): Price per unit
- `amount` (decimal): Total transaction amount
- `fees` (decimal): Transaction fees
- `taxes` (decimal): Transaction taxes
- `trade_date` (datetime): Date of the trade
- `settlement_date` (datetime): Settlement date
- `status` (enum): Transaction status

#### PerformanceRecord
Represents a performance snapshot for a portfolio.

**Attributes:**
- `portfolio_id` (string): Reference to the portfolio
- `date` (datetime): Date of the record
- `value` (decimal): Portfolio value on this date
- `cash_flow` (decimal): Net cash flow on this date
- `daily_return` (decimal): Return for this day
- `cumulative_return` (decimal): Cumulative return from inception

#### ComplianceRule
Represents a compliance rule for a portfolio.

**Attributes:**
- `rule_id` (string): Unique identifier
- `portfolio_id` (string): Reference to the portfolio
- `name` (string): Rule name
- `description` (string): Detailed description
- `category` (enum): Rule category (concentration, leverage, etc.)
- `limit_type` (enum): Type of limit (maximum, minimum, range)
- `limit_value` (decimal): Threshold value
- `status` (enum): Current status (compliant, warning, violation)
- `current_value` (decimal): Current measured value

## DB Schema (CQRS Pattern)

### Write Schema (Command Side)

#### portfolios
```sql
CREATE TABLE portfolios (
    id VARCHAR(36) PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    base_currency VARCHAR(3) NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'active',
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX portfolios_status_idx ON portfolios(status);
```

#### positions
```sql
CREATE TABLE positions (
    id VARCHAR(36) PRIMARY KEY,
    portfolio_id VARCHAR(36) NOT NULL,
    instrument_id VARCHAR(20) NOT NULL,
    instrument_type VARCHAR(20) NOT NULL,
    quantity DECIMAL(18, 8) NOT NULL,
    average_cost DECIMAL(18, 8) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT positions_portfolio_instrument_unique UNIQUE (portfolio_id, instrument_id),
    CONSTRAINT positions_portfolio_fk FOREIGN KEY (portfolio_id) REFERENCES portfolios(id)
);

CREATE INDEX positions_portfolio_idx ON positions(portfolio_id);
CREATE INDEX positions_instrument_idx ON positions(instrument_id);
```

#### tax_lots
```sql
CREATE TABLE tax_lots (
    id VARCHAR(36) PRIMARY KEY,
    position_id VARCHAR(36) NOT NULL,
    acquisition_date TIMESTAMP WITH TIME ZONE NOT NULL,
    quantity DECIMAL(18, 8) NOT NULL,
    cost_basis DECIMAL(18, 8) NOT NULL,
    cost_basis_total DECIMAL(18, 8) NOT NULL,
    tax_status VARCHAR(20),
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT tax_lots_position_fk FOREIGN KEY (position_id) REFERENCES positions(id)
);

CREATE INDEX tax_lots_position_idx ON tax_lots(position_id);
CREATE INDEX tax_lots_acquisition_date_idx ON tax_lots(acquisition_date);
```

#### transactions
```sql
CREATE TABLE transactions (
    id VARCHAR(36) PRIMARY KEY,
    portfolio_id VARCHAR(36) NOT NULL,
    instrument_id VARCHAR(20),
    transaction_type VARCHAR(20) NOT NULL,
    quantity DECIMAL(18, 8),
    price DECIMAL(18, 8),
    amount DECIMAL(18, 8) NOT NULL,
    fees DECIMAL(18, 8) NOT NULL DEFAULT 0,
    taxes DECIMAL(18, 8) NOT NULL DEFAULT 0,
    trade_date TIMESTAMP WITH TIME ZONE NOT NULL,
    settlement_date TIMESTAMP WITH TIME ZONE,
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT transactions_portfolio_fk FOREIGN KEY (portfolio_id) REFERENCES portfolios(id)
);

CREATE INDEX transactions_portfolio_idx ON transactions(portfolio_id);
CREATE INDEX transactions_trade_date_idx ON transactions(trade_date);
CREATE INDEX transactions_status_idx ON transactions(status);
```

#### cash_balances
```sql
CREATE TABLE cash_balances (
    id VARCHAR(36) PRIMARY KEY,
    portfolio_id VARCHAR(36) NOT NULL,
    currency VARCHAR(3) NOT NULL,
    amount DECIMAL(18, 8) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT cash_balances_portfolio_currency_unique UNIQUE (portfolio_id, currency),
    CONSTRAINT cash_balances_portfolio_fk FOREIGN KEY (portfolio_id) REFERENCES portfolios(id)
);

CREATE INDEX cash_balances_portfolio_idx ON cash_balances(portfolio_id);
```

#### compliance_rules
```sql
CREATE TABLE compliance_rules (
    id VARCHAR(36) PRIMARY KEY,
    portfolio_id VARCHAR(36) NOT NULL,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    category VARCHAR(50) NOT NULL,
    limit_type VARCHAR(20) NOT NULL,
    limit_value DECIMAL(18, 8) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT compliance_rules_portfolio_fk FOREIGN KEY (portfolio_id) REFERENCES portfolios(id)
);

CREATE INDEX compliance_rules_portfolio_idx ON compliance_rules(portfolio_id);
CREATE INDEX compliance_rules_category_idx ON compliance_rules(category);
```

### Read Schema (Query Side)

#### portfolio_snapshots
```sql
CREATE TABLE portfolio_snapshots (
    portfolio_id VARCHAR(36) NOT NULL,
    snapshot_date DATE NOT NULL,
    total_value DECIMAL(18, 8) NOT NULL,
    cash_balance DECIMAL(18, 8) NOT NULL,
    margin_used DECIMAL(18, 8) NOT NULL DEFAULT 0,
    margin_available DECIMAL(18, 8) NOT NULL DEFAULT 0,
    daily_return DECIMAL(10, 8),
    mtd_return DECIMAL(10, 8),
    ytd_return DECIMAL(10, 8),
    inception_return DECIMAL(10, 8),
    volatility_30d DECIMAL(10, 8),
    sharpe_ratio DECIMAL(10, 8),
    sortino_ratio DECIMAL(10, 8),
    max_drawdown DECIMAL(10, 8),
    var_95 DECIMAL(18, 8),
    cvar_95 DECIMAL(18, 8),
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    PRIMARY KEY (portfolio_id, snapshot_date)
);

CREATE INDEX portfolio_snapshots_date_idx ON portfolio_snapshots(snapshot_date);
```

#### position_snapshots
```sql
CREATE TABLE position_snapshots (
    portfolio_id VARCHAR(36) NOT NULL,
    instrument_id VARCHAR(20) NOT NULL,
    snapshot_date DATE NOT NULL,
    quantity DECIMAL(18, 8) NOT NULL,
    market_price DECIMAL(18, 8) NOT NULL,
    market_value DECIMAL(18, 8) NOT NULL,
    cost_basis DECIMAL(18, 8) NOT NULL,
    unrealized_pnl DECIMAL(18, 8) NOT NULL,
    unrealized_pnl_percent DECIMAL(10, 8) NOT NULL,
    weight DECIMAL(10, 8) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    PRIMARY KEY (portfolio_id, instrument_id, snapshot_date)
);

CREATE INDEX position_snapshots_portfolio_date_idx ON position_snapshots(portfolio_id, snapshot_date);
```

#### portfolio_allocations
```sql
CREATE TABLE portfolio_allocations (
    portfolio_id VARCHAR(36) NOT NULL,
    snapshot_date DATE NOT NULL,
    allocation_type VARCHAR(20) NOT NULL,
    category VARCHAR(50) NOT NULL,
    weight DECIMAL(10, 8) NOT NULL,
    market_value DECIMAL(18, 8) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    PRIMARY KEY (portfolio_id, snapshot_date, allocation_type, category)
);

CREATE INDEX portfolio_allocations_portfolio_date_idx ON portfolio_allocations(portfolio_id, snapshot_date);
```

#### performance_attribution
```sql
CREATE TABLE performance_attribution (
    portfolio_id VARCHAR(36) NOT NULL,
    period_start DATE NOT NULL,
    period_end DATE NOT NULL,
    attribution_level VARCHAR(20) NOT NULL,
    category VARCHAR(50) NOT NULL,
    allocation_effect DECIMAL(10, 8) NOT NULL,
    selection_effect DECIMAL(10, 8) NOT NULL,
    interaction_effect DECIMAL(10, 8) NOT NULL,
    total_effect DECIMAL(10, 8) NOT NULL,
    average_weight DECIMAL(10, 8) NOT NULL,
    return DECIMAL(10, 8) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    PRIMARY KEY (portfolio_id, period_start, period_end, attribution_level, category)
);

CREATE INDEX performance_attribution_portfolio_period_idx ON performance_attribution(portfolio_id, period_start, period_end);
```

#### compliance_status
```sql
CREATE TABLE compliance_status (
    rule_id VARCHAR(36) NOT NULL,
    portfolio_id VARCHAR(36) NOT NULL,
    snapshot_date DATE NOT NULL,
    status VARCHAR(20) NOT NULL,
    current_value DECIMAL(18, 8) NOT NULL,
    limit_value DECIMAL(18, 8) NOT NULL,
    details JSONB,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    PRIMARY KEY (rule_id, snapshot_date)
);

CREATE INDEX compliance_status_portfolio_date_idx ON compliance_status(portfolio_id, snapshot_date);
CREATE INDEX compliance_status_status_idx ON compliance_status(status);
```

## Place in Workflow

The Portfolio Management Service is a central component of the Portfolio Management Workflow and serves as the system of record for portfolio data. It:

1. **Maintains accurate position records** across all instruments and portfolios
2. **Tracks performance** at portfolio, strategy, and position levels
3. **Monitors compliance** with investment mandates and regulatory requirements
4. **Processes corporate actions** and their impact on positions
5. **Provides comprehensive portfolio data** to downstream services

The service interacts with:
- **Order Management Service** (input): Receives trade confirmations to update positions
- **Market Data Service** (input): Receives pricing data for valuation
- **Risk Analysis Service** (input/output): Exchanges risk metrics and portfolio data
- **Portfolio Optimization Service** (output): Provides portfolio data for optimization
- **Trading Strategy Service** (output): Supplies portfolio constraints and allocations
- **Reporting Service** (output): Provides data for comprehensive reporting
- **User Service** (output): Delivers portfolio information to end users

## Project Plan

### Phase 1: Foundation (Weeks 1-4)

#### Week 1: Setup and Infrastructure
- Set up development environment and CI/CD pipeline
- Configure Kubernetes infrastructure for the service
- Set up PostgreSQL database with appropriate schemas
- Implement basic service skeleton with health checks

#### Week 2: Core Data Model Implementation
- Implement portfolio data model and repositories
- Develop position tracking functionality
- Create transaction processing logic
- Build cash management capabilities

#### Week 3: Performance Calculation
- Implement time-weighted return calculation
- Develop performance attribution analysis
- Create benchmark comparison functionality
- Build risk metrics calculation

#### Week 4: Basic API Implementation
- Implement REST API endpoints for data retrieval
- Create gRPC service for internal communication
- Develop basic authentication and authorization
- Write comprehensive API documentation

### Phase 2: Enhancement (Weeks 5-8)

#### Week 5: Compliance Monitoring
- Implement compliance rule engine
- Develop rule evaluation and monitoring
- Create compliance reporting functionality
- Build alert generation for violations

#### Week 6: Tax Lot Management
- Implement tax lot tracking
- Develop lot allocation strategies (FIFO, LIFO, etc.)
- Create tax reporting functionality
- Build tax optimization analysis

#### Week 7: Corporate Actions Processing
- Implement dividend processing
- Develop stock split handling
- Create merger and acquisition processing
- Build corporate action impact analysis

#### Week 8: Integration and Testing
- Integrate with Order Management Service
- Connect with Market Data Service
- Perform load and stress testing
- Develop comprehensive monitoring and alerting

### Phase 3: Production Readiness (Weeks 9-12)

#### Week 9: Documentation and Training
- Create detailed service documentation
- Develop operational runbooks
- Prepare training materials
- Conduct knowledge transfer sessions

#### Week 10: Advanced Features
- Implement multi-currency support
- Develop custom benchmark capabilities
- Create advanced performance analytics
- Build portfolio modeling functionality

#### Week 11: Quality Assurance
- Conduct accuracy testing of calculations
- Perform reconciliation testing
- Validate compliance monitoring
- Address any identified issues

#### Week 12: Final Preparation and Deployment
- Conduct final integration testing
- Prepare production deployment plan
- Execute staged rollout
- Monitor production performance

### Ongoing Maintenance
- Regular reconciliation with broker statements
- Performance calculation validation
- Database optimization and maintenance
- Feature enhancements based on user feedback
