# Market Data Service

## Purpose
The Market Data Service is responsible for collecting, normalizing, and distributing market data from various providers with high reliability and low latency. It serves as the authoritative source of market data for the entire QuantiVista platform, ensuring that all downstream services have access to high-quality, standardized market data.

## Strict Boundaries

### Responsibilities
- Real-time and historical data ingestion from multiple sources
- Data quality validation and cleansing
- Format normalization across providers
- Corporate actions processing
- Data distribution via event streams

### Non-Responsibilities
- Does NOT perform analysis or make predictions
- Does NOT make trading decisions
- Does NOT store user preferences or configurations
- Does NOT handle authentication or authorization

## API Design (API-First)

### REST API

#### GET /api/v1/market-data/instruments
Retrieves a list of available financial instruments.

**Query Parameters:**
- `type` (string, optional): Filter by instrument type (stock, option, future, forex, crypto)
- `exchange` (string, optional): Filter by exchange
- `sector` (string, optional): Filter by sector
- `limit` (integer, optional): Maximum number of results (default: 100)
- `offset` (integer, optional): Pagination offset (default: 0)

**Response:**
```json
{
  "instruments": [
    {
      "id": "AAPL",
      "name": "Apple Inc.",
      "type": "stock",
      "exchange": "NASDAQ",
      "sector": "Technology",
      "currency": "USD",
      "metadata": {
        "isin": "US0378331005",
        "cusip": "037833100"
      }
    }
  ],
  "pagination": {
    "total": 10000,
    "limit": 100,
    "offset": 0
  }
}
```

#### GET /api/v1/market-data/ohlc/{instrument_id}
Retrieves OHLC (Open, High, Low, Close) data for a specific instrument.

**Path Parameters:**
- `instrument_id` (string, required): Instrument identifier

**Query Parameters:**
- `interval` (string, required): Time interval (1m, 5m, 15m, 30m, 1h, 4h, 1d, 1w, 1mo)
- `from` (string, required): Start timestamp (ISO 8601)
- `to` (string, required): End timestamp (ISO 8601)
- `adjusted` (boolean, optional): Apply corporate action adjustments (default: true)

**Response:**
```json
{
  "instrument_id": "AAPL",
  "interval": "1d",
  "adjusted": true,
  "data": [
    {
      "timestamp": "2025-06-19T00:00:00Z",
      "open": 150.25,
      "high": 152.75,
      "low": 149.50,
      "close": 151.80,
      "volume": 75000000
    }
  ]
}
```

#### GET /api/v1/market-data/ticks/{instrument_id}
Retrieves tick-by-tick data for a specific instrument.

**Path Parameters:**
- `instrument_id` (string, required): Instrument identifier

**Query Parameters:**
- `from` (string, required): Start timestamp (ISO 8601)
- `to` (string, required): End timestamp (ISO 8601)
- `limit` (integer, optional): Maximum number of ticks (default: 1000)

**Response:**
```json
{
  "instrument_id": "AAPL",
  "data": [
    {
      "timestamp": "2025-06-20T14:30:00.123Z",
      "price": 151.75,
      "volume": 100,
      "side": "buy",
      "exchange": "NASDAQ"
    }
  ]
}
```

### gRPC API

```protobuf
syntax = "proto3";

package market_data.v1;

import "google/protobuf/timestamp.proto";

service MarketDataService {
  // Stream real-time market data updates
  rpc StreamMarketData(StreamMarketDataRequest) returns (stream MarketDataUpdate);

  // Get historical OHLC data
  rpc GetHistoricalOHLC(HistoricalOHLCRequest) returns (HistoricalOHLCResponse);

  // Get instrument details
  rpc GetInstrument(GetInstrumentRequest) returns (InstrumentResponse);
}

message StreamMarketDataRequest {
  repeated string instrument_ids = 1;
  string update_type = 2; // "tick", "ohlc", "orderbook"
  string interval = 3; // For OHLC: "1m", "5m", "15m", "30m", "1h", "4h", "1d"
}

message MarketDataUpdate {
  string instrument_id = 1;
  string update_type = 2;
  google.protobuf.Timestamp timestamp = 3;

  oneof data {
    TickData tick = 4;
    OHLCData ohlc = 5;
    OrderBookData orderbook = 6;
  }
}

message TickData {
  double price = 1;
  double volume = 2;
  string side = 3;
  string exchange = 4;
}

message OHLCData {
  double open = 1;
  double high = 2;
  double low = 3;
  double close = 4;
  double volume = 5;
  string interval = 6;
}

message OrderBookData {
  repeated OrderBookLevel bids = 1;
  repeated OrderBookLevel asks = 2;
}

message OrderBookLevel {
  double price = 1;
  double volume = 2;
  int32 order_count = 3;
}

message HistoricalOHLCRequest {
  string instrument_id = 1;
  string interval = 2; // "1m", "5m", "15m", "30m", "1h", "4h", "1d"
  google.protobuf.Timestamp from = 3;
  google.protobuf.Timestamp to = 4;
  bool adjusted = 5;
}

message HistoricalOHLCResponse {
  string instrument_id = 1;
  string interval = 2;
  bool adjusted = 3;
  repeated OHLCData data = 4;
}

message GetInstrumentRequest {
  string instrument_id = 1;
}

message InstrumentResponse {
  string id = 1;
  string name = 2;
  string type = 3;
  string exchange = 4;
  string sector = 5;
  string currency = 6;
  map<string, string> metadata = 7;
}
```

## Data Model

### Core Entities

#### Instrument
Represents a financial instrument that can be traded.

**Attributes:**
- `id` (string): Unique identifier for the instrument
- `name` (string): Human-readable name
- `type` (enum): Type of instrument (stock, option, future, forex, crypto)
- `exchange` (string): Exchange where the instrument is traded
- `sector` (string): Industry sector (for stocks)
- `currency` (string): Currency in which the instrument is denominated
- `metadata` (map): Additional instrument-specific metadata

#### OHLC
Represents price data aggregated over a specific time interval.

**Attributes:**
- `instrument_id` (string): Reference to the instrument
- `interval` (enum): Time interval (1m, 5m, 15m, 30m, 1h, 4h, 1d, 1w, 1mo)
- `timestamp` (datetime): Start of the time interval
- `open` (decimal): Opening price
- `high` (decimal): Highest price during the interval
- `low` (decimal): Lowest price during the interval
- `close` (decimal): Closing price
- `volume` (decimal): Trading volume
- `adjusted` (boolean): Whether prices are adjusted for corporate actions

#### Tick
Represents an individual trade.

**Attributes:**
- `instrument_id` (string): Reference to the instrument
- `timestamp` (datetime): Exact time of the trade
- `price` (decimal): Trade price
- `volume` (decimal): Trade volume
- `side` (enum): Buy or sell
- `exchange` (string): Exchange where the trade occurred
- `trade_id` (string): Unique identifier for the trade

#### CorporateAction
Represents a corporate event that affects instrument pricing.

**Attributes:**
- `instrument_id` (string): Reference to the instrument
- `type` (enum): Type of action (split, dividend, merger, spinoff)
- `ex_date` (date): Date when the action takes effect
- `factor` (decimal): Adjustment factor
- `details` (map): Additional action-specific details

## DB Schema (CQRS Pattern)

### Write Schema (Command Side)

#### instruments
```sql
CREATE TABLE instruments (
    id VARCHAR(20) PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    type VARCHAR(20) NOT NULL,
    exchange VARCHAR(20) NOT NULL,
    sector VARCHAR(50),
    currency VARCHAR(3) NOT NULL,
    metadata JSONB,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);
```

#### raw_ticks
```sql
CREATE TABLE raw_ticks (
    id BIGSERIAL PRIMARY KEY,
    instrument_id VARCHAR(20) NOT NULL REFERENCES instruments(id),
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    price DECIMAL(18, 8) NOT NULL,
    volume DECIMAL(18, 8) NOT NULL,
    side VARCHAR(4),
    exchange VARCHAR(20) NOT NULL,
    trade_id VARCHAR(50),
    source VARCHAR(20) NOT NULL,
    received_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT raw_ticks_instrument_timestamp_idx UNIQUE (instrument_id, timestamp, trade_id)
);

-- Time-based partitioning
SELECT create_hypertable('raw_ticks', 'timestamp', chunk_time_interval => INTERVAL '1 day');
```

#### corporate_actions
```sql
CREATE TABLE corporate_actions (
    id SERIAL PRIMARY KEY,
    instrument_id VARCHAR(20) NOT NULL REFERENCES instruments(id),
    type VARCHAR(20) NOT NULL,
    ex_date DATE NOT NULL,
    factor DECIMAL(10, 6) NOT NULL,
    details JSONB,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT corporate_actions_instrument_date_type_idx UNIQUE (instrument_id, ex_date, type)
);
```

#### data_quality_events
```sql
CREATE TABLE data_quality_events (
    id SERIAL PRIMARY KEY,
    instrument_id VARCHAR(20) NOT NULL REFERENCES instruments(id),
    event_type VARCHAR(50) NOT NULL,
    severity VARCHAR(10) NOT NULL,
    description TEXT NOT NULL,
    affected_timestamp TIMESTAMP WITH TIME ZONE,
    detected_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    resolved_at TIMESTAMP WITH TIME ZONE,
    resolution TEXT
);
```

### Read Schema (Query Side)

#### ohlc_1m, ohlc_5m, ohlc_15m, ohlc_1h, ohlc_1d
```sql
CREATE TABLE ohlc_1m (
    instrument_id VARCHAR(20) NOT NULL,
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    open DECIMAL(18, 8) NOT NULL,
    high DECIMAL(18, 8) NOT NULL,
    low DECIMAL(18, 8) NOT NULL,
    close DECIMAL(18, 8) NOT NULL,
    volume DECIMAL(18, 8) NOT NULL,
    adjusted BOOLEAN NOT NULL DEFAULT TRUE,
    PRIMARY KEY (instrument_id, timestamp)
);

-- Time-based partitioning
SELECT create_hypertable('ohlc_1m', 'timestamp', chunk_time_interval => INTERVAL '1 day');

-- Create similar tables for other intervals
```

#### latest_prices
```sql
CREATE TABLE latest_prices (
    instrument_id VARCHAR(20) PRIMARY KEY,
    price DECIMAL(18, 8) NOT NULL,
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    daily_change DECIMAL(18, 8),
    daily_change_percent DECIMAL(10, 6),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);
```

#### instrument_stats
```sql
CREATE TABLE instrument_stats (
    instrument_id VARCHAR(20) PRIMARY KEY,
    avg_daily_volume DECIMAL(18, 8),
    fifty_day_ma DECIMAL(18, 8),
    two_hundred_day_ma DECIMAL(18, 8),
    fifty_two_week_high DECIMAL(18, 8),
    fifty_two_week_low DECIMAL(18, 8),
    volatility_30d DECIMAL(10, 6),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);
```

## Place in Workflow

The Market Data Service is the foundation of the Market Data Acquisition and Processing Workflow and serves as the entry point for all market data into the QuantiVista platform. It:

1. **Initiates the workflow** by connecting to external data sources and ingesting raw market data
2. **Performs initial processing** including validation, normalization, and enrichment
3. **Stores both raw and processed data** for audit, replay, and analysis
4. **Distributes processed data** to downstream services via event streams

The service interacts with:
- **External data providers** (input): Alpha Vantage, Finnhub, IEX Cloud, Interactive Brokers, Alpaca, etc.
- **Technical Analysis Service** (output): Provides normalized market data for technical indicator calculation
- **ML Prediction Service** (output): Supplies market data as input features for prediction models
- **Risk Analysis Service** (output): Delivers market data for risk calculations
- **Trading Strategy Service** (output): Provides market data for trading decisions
- **Reporting Service** (output): Supplies historical market data for reports and visualizations

## Project Plan

### Phase 1: Foundation (Weeks 1-4)

#### Week 1: Setup and Infrastructure
- Set up development environment and CI/CD pipeline
- Configure Kubernetes infrastructure for the service
- Set up TimescaleDB for time-series data storage
- Implement basic service skeleton with health checks

#### Week 2: Data Ingestion Framework
- Develop provider-agnostic data ingestion framework
- Implement adapter pattern for different data sources
- Create connection management with retry logic
- Build data validation and quality check framework

#### Week 3: Core Data Processing
- Implement data normalization and standardization
- Develop corporate actions processing
- Create data enrichment pipeline
- Build efficient time-series storage mechanisms

#### Week 4: Basic API Implementation
- Implement REST API endpoints for data retrieval
- Create gRPC service for real-time data streaming
- Develop basic authentication and rate limiting
- Write comprehensive API documentation

### Phase 2: Enhancement (Weeks 5-8)

#### Week 5: Advanced Data Processing
- Implement anomaly detection for data quality
- Develop data lineage tracking
- Create comprehensive data quality metrics
- Build data reconciliation mechanisms

#### Week 6: Scalability and Performance
- Optimize database queries and indexing
- Implement caching strategies
- Develop horizontal scaling capabilities
- Create load balancing across data providers

#### Week 7: Resilience and Reliability
- Implement circuit breakers for unreliable data sources
- Develop fallback mechanisms
- Create comprehensive error handling
- Build automated recovery procedures

#### Week 8: Integration and Testing
- Integrate with downstream services
- Perform load and stress testing
- Conduct security testing
- Develop comprehensive monitoring and alerting

### Phase 3: Production Readiness (Weeks 9-12)

#### Week 9: Documentation and Training
- Create detailed service documentation
- Develop operational runbooks
- Prepare training materials
- Conduct knowledge transfer sessions

#### Week 10: Performance Optimization
- Fine-tune database performance
- Optimize memory usage
- Enhance throughput and latency
- Implement advanced caching strategies

#### Week 11: Security Hardening
- Conduct security review
- Implement additional security measures
- Perform penetration testing
- Address security findings

#### Week 12: Final Preparation and Deployment
- Conduct final integration testing
- Prepare production deployment plan
- Execute staged rollout
- Monitor production performance

### Ongoing Maintenance
- Regular performance monitoring and optimization
- Data provider integration updates
- Schema and API versioning
- Security patches and updates
