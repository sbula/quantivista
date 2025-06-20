# Technical Analysis Service

## Purpose
The Technical Analysis Service is responsible for computing technical indicators and performing statistical analysis on market data with high performance and accuracy. It provides a comprehensive suite of technical analysis tools that can be used by other services to make informed trading decisions and risk assessments.

## Strict Boundaries

### Responsibilities
- Technical indicator calculation for various timeframes
- Statistical analysis and pattern recognition
- Cross-instrument correlation analysis
- Volatility modeling and forecasting
- Support and resistance level identification
- Distribution of technical analysis results

### Non-Responsibilities
- Does NOT make trading decisions
- Does NOT execute trades
- Does NOT perform fundamental analysis
- Does NOT handle instrument metadata management
- Does NOT perform instrument clustering
- Does NOT manage user preferences or authentication

## API Design (API-First)

### REST API

#### GET /api/v1/indicators/{instrument_id}
Retrieves technical indicators for a specific instrument.

**Path Parameters:**
- `instrument_id` (string, required): Instrument identifier

**Query Parameters:**
- `indicators` (string, required): Comma-separated list of indicators (e.g., "sma,ema,rsi,macd")
- `interval` (string, required): Time interval (1m, 5m, 15m, 30m, 1h, 4h, 1d, 1w, 1mo)
- `from` (string, required): Start timestamp (ISO 8601)
- `to` (string, required): End timestamp (ISO 8601)
- `params` (string, optional): JSON-encoded parameters for indicators (e.g., periods, deviations)

**Response:**
```json
{
  "instrument_id": "AAPL",
  "interval": "1d",
  "from": "2025-05-01T00:00:00Z",
  "to": "2025-06-01T00:00:00Z",
  "indicators": {
    "sma": {
      "periods": [20, 50, 200],
      "data": [
        {
          "timestamp": "2025-05-20T00:00:00Z",
          "values": {
            "20": 152.75,
            "50": 148.32,
            "200": 142.18
          }
        }
      ]
    },
    "rsi": {
      "period": 14,
      "data": [
        {
          "timestamp": "2025-05-20T00:00:00Z",
          "value": 65.42
        }
      ]
    }
  }
}
```

#### GET /api/v1/patterns/{instrument_id}
Retrieves pattern recognition results for a specific instrument.

**Path Parameters:**
- `instrument_id` (string, required): Instrument identifier

**Query Parameters:**
- `patterns` (string, required): Comma-separated list of patterns (e.g., "head_and_shoulders,double_top,triangle")
- `interval` (string, required): Time interval (1h, 4h, 1d, 1w)
- `from` (string, required): Start timestamp (ISO 8601)
- `to` (string, required): End timestamp (ISO 8601)
- `min_confidence` (float, optional): Minimum confidence level (0.0-1.0, default: 0.7)

**Response:**
```json
{
  "instrument_id": "AAPL",
  "interval": "1d",
  "from": "2025-05-01T00:00:00Z",
  "to": "2025-06-01T00:00:00Z",
  "patterns": [
    {
      "type": "head_and_shoulders",
      "start_timestamp": "2025-05-10T00:00:00Z",
      "end_timestamp": "2025-05-20T00:00:00Z",
      "confidence": 0.85,
      "target_price": 145.50,
      "completion_percentage": 100
    },
    {
      "type": "triangle",
      "start_timestamp": "2025-05-25T00:00:00Z",
      "end_timestamp": "2025-06-01T00:00:00Z",
      "confidence": 0.72,
      "target_price": 155.25,
      "completion_percentage": 75
    }
  ]
}
```

#### GET /api/v1/correlations
Retrieves correlation analysis between instruments.

**Query Parameters:**
- `instruments` (string, required): Comma-separated list of instrument IDs
- `interval` (string, required): Time interval (1d, 1w, 1mo)
- `from` (string, required): Start timestamp (ISO 8601)
- `to` (string, required): End timestamp (ISO 8601)
- `method` (string, optional): Correlation method (pearson, spearman, kendall, default: pearson)

**Response:**
```json
{
  "interval": "1d",
  "from": "2025-05-01T00:00:00Z",
  "to": "2025-06-01T00:00:00Z",
  "method": "pearson",
  "correlations": {
    "AAPL": {
      "MSFT": 0.82,
      "GOOGL": 0.75,
      "AMZN": 0.68
    },
    "MSFT": {
      "AAPL": 0.82,
      "GOOGL": 0.79,
      "AMZN": 0.71
    },
    "GOOGL": {
      "AAPL": 0.75,
      "MSFT": 0.79,
      "AMZN": 0.80
    },
    "AMZN": {
      "AAPL": 0.68,
      "MSFT": 0.71,
      "GOOGL": 0.80
    }
  }
}
```

### gRPC API

```protobuf
syntax = "proto3";

package technical_analysis.v1;

import "google/protobuf/timestamp.proto";

service TechnicalAnalysisService {
  // Calculate technical indicators
  rpc CalculateIndicators(IndicatorRequest) returns (IndicatorResponse);
  
  // Stream real-time indicator updates
  rpc StreamIndicators(StreamIndicatorRequest) returns (stream IndicatorUpdate);
  
  // Detect patterns in price data
  rpc DetectPatterns(PatternRequest) returns (PatternResponse);
  
  // Calculate correlations between instruments
  rpc CalculateCorrelations(CorrelationRequest) returns (CorrelationResponse);
}

message IndicatorRequest {
  string instrument_id = 1;
  repeated string indicator_types = 2; // "sma", "ema", "rsi", "macd", etc.
  string interval = 3; // "1m", "5m", "15m", "30m", "1h", "4h", "1d", "1w"
  google.protobuf.Timestamp from = 4;
  google.protobuf.Timestamp to = 5;
  map<string, IndicatorParams> params = 6;
}

message IndicatorParams {
  repeated int32 periods = 1;
  repeated double deviations = 2;
  map<string, double> custom_params = 3;
}

message IndicatorResponse {
  string instrument_id = 1;
  string interval = 2;
  map<string, IndicatorData> indicators = 3;
}

message IndicatorData {
  repeated IndicatorPoint data = 1;
  map<string, string> metadata = 2;
}

message IndicatorPoint {
  google.protobuf.Timestamp timestamp = 1;
  map<string, double> values = 2;
}

message StreamIndicatorRequest {
  repeated string instrument_ids = 1;
  repeated string indicator_types = 2;
  string interval = 3;
  map<string, IndicatorParams> params = 4;
}

message IndicatorUpdate {
  string instrument_id = 1;
  string indicator_type = 2;
  string interval = 3;
  google.protobuf.Timestamp timestamp = 4;
  map<string, double> values = 5;
}

message PatternRequest {
  string instrument_id = 1;
  repeated string pattern_types = 2; // "head_and_shoulders", "double_top", etc.
  string interval = 3; // "1h", "4h", "1d", "1w"
  google.protobuf.Timestamp from = 4;
  google.protobuf.Timestamp to = 5;
  double min_confidence = 6;
}

message PatternResponse {
  string instrument_id = 1;
  string interval = 2;
  repeated PatternDetection patterns = 3;
}

message PatternDetection {
  string type = 1;
  google.protobuf.Timestamp start_timestamp = 2;
  google.protobuf.Timestamp end_timestamp = 3;
  double confidence = 4;
  double target_price = 5;
  int32 completion_percentage = 6;
}

message CorrelationRequest {
  repeated string instrument_ids = 1;
  string interval = 2; // "1d", "1w", "1mo"
  google.protobuf.Timestamp from = 3;
  google.protobuf.Timestamp to = 4;
  string method = 5; // "pearson", "spearman", "kendall"
}

message CorrelationResponse {
  string interval = 1;
  string method = 2;
  repeated CorrelationPair correlations = 3;
}

message CorrelationPair {
  string instrument_id_1 = 1;
  string instrument_id_2 = 2;
  double correlation = 3;
  double p_value = 4;
}
```

## Data Model

### Core Entities

#### Indicator
Represents a technical indicator calculated for a specific instrument and timeframe.

**Attributes:**
- `instrument_id` (string): Reference to the instrument
- `indicator_type` (enum): Type of indicator (SMA, EMA, RSI, MACD, etc.)
- `interval` (enum): Time interval (1m, 5m, 15m, 30m, 1h, 4h, 1d, 1w, 1mo)
- `timestamp` (datetime): Timestamp for the indicator value
- `parameters` (map): Parameters used for calculation (periods, deviations, etc.)
- `values` (map): Calculated values (may include multiple values for some indicators)
- `metadata` (map): Additional indicator-specific metadata

#### Pattern
Represents a detected chart pattern.

**Attributes:**
- `instrument_id` (string): Reference to the instrument
- `pattern_type` (enum): Type of pattern (head and shoulders, double top, triangle, etc.)
- `interval` (enum): Time interval (1h, 4h, 1d, 1w)
- `start_timestamp` (datetime): Start of the pattern
- `end_timestamp` (datetime): End of the pattern
- `confidence` (float): Confidence level of the pattern detection
- `target_price` (decimal): Projected price target based on the pattern
- `completion_percentage` (integer): How complete the pattern is (0-100%)

#### Correlation
Represents a correlation between two instruments.

**Attributes:**
- `instrument_id_1` (string): First instrument
- `instrument_id_2` (string): Second instrument
- `interval` (enum): Time interval (1d, 1w, 1mo)
- `start_timestamp` (datetime): Start of the correlation period
- `end_timestamp` (datetime): End of the correlation period
- `method` (enum): Correlation method (pearson, spearman, kendall)
- `correlation` (float): Correlation coefficient (-1.0 to 1.0)
- `p_value` (float): Statistical significance of the correlation

#### SupportResistance
Represents support and resistance levels for an instrument.

**Attributes:**
- `instrument_id` (string): Reference to the instrument
- `level_type` (enum): Type of level (support, resistance)
- `interval` (enum): Time interval (1h, 4h, 1d, 1w)
- `price` (decimal): Price level
- `strength` (float): Strength of the level (0.0-1.0)
- `touches` (integer): Number of times the price has touched this level
- `start_timestamp` (datetime): When the level was first identified
- `last_test_timestamp` (datetime): When the level was last tested

## DB Schema (CQRS Pattern)

### Write Schema (Command Side)

#### indicator_calculations
```sql
CREATE TABLE indicator_calculations (
    id SERIAL PRIMARY KEY,
    instrument_id VARCHAR(20) NOT NULL,
    indicator_type VARCHAR(20) NOT NULL,
    interval VARCHAR(10) NOT NULL,
    parameters JSONB NOT NULL,
    calculation_timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    error_message TEXT,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT indicator_calculations_unique_idx UNIQUE (instrument_id, indicator_type, interval, parameters)
);

CREATE INDEX indicator_calculations_status_idx ON indicator_calculations(status);
```

#### pattern_detections
```sql
CREATE TABLE pattern_detections (
    id SERIAL PRIMARY KEY,
    instrument_id VARCHAR(20) NOT NULL,
    pattern_type VARCHAR(30) NOT NULL,
    interval VARCHAR(10) NOT NULL,
    start_timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    end_timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    confidence DECIMAL(5,4) NOT NULL,
    target_price DECIMAL(18, 8),
    completion_percentage INTEGER NOT NULL,
    detection_timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX pattern_detections_instrument_idx ON pattern_detections(instrument_id);
CREATE INDEX pattern_detections_timestamp_idx ON pattern_detections(end_timestamp);
```

#### correlation_calculations
```sql
CREATE TABLE correlation_calculations (
    id SERIAL PRIMARY KEY,
    instrument_id_1 VARCHAR(20) NOT NULL,
    instrument_id_2 VARCHAR(20) NOT NULL,
    interval VARCHAR(10) NOT NULL,
    start_timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    end_timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    method VARCHAR(20) NOT NULL DEFAULT 'pearson',
    correlation DECIMAL(5,4) NOT NULL,
    p_value DECIMAL(7,6) NOT NULL,
    calculation_timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT correlation_calculations_unique_idx UNIQUE (instrument_id_1, instrument_id_2, interval, start_timestamp, end_timestamp, method)
);

CREATE INDEX correlation_calculations_instruments_idx ON correlation_calculations(instrument_id_1, instrument_id_2);
```

#### support_resistance_levels
```sql
CREATE TABLE support_resistance_levels (
    id SERIAL PRIMARY KEY,
    instrument_id VARCHAR(20) NOT NULL,
    level_type VARCHAR(10) NOT NULL,
    interval VARCHAR(10) NOT NULL,
    price DECIMAL(18, 8) NOT NULL,
    strength DECIMAL(4,3) NOT NULL,
    touches INTEGER NOT NULL DEFAULT 1,
    start_timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    last_test_timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX support_resistance_levels_instrument_idx ON support_resistance_levels(instrument_id);
```

### Read Schema (Query Side)

#### technical_indicators
```sql
CREATE TABLE technical_indicators (
    instrument_id VARCHAR(20) NOT NULL,
    indicator_type VARCHAR(20) NOT NULL,
    interval VARCHAR(10) NOT NULL,
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    parameters JSONB NOT NULL,
    values JSONB NOT NULL,
    metadata JSONB,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    PRIMARY KEY (instrument_id, indicator_type, interval, timestamp, parameters)
);

-- Time-based partitioning
SELECT create_hypertable('technical_indicators', 'timestamp', chunk_time_interval => INTERVAL '1 day');

CREATE INDEX technical_indicators_lookup_idx ON technical_indicators(instrument_id, indicator_type, interval);
```

#### latest_indicators
```sql
CREATE TABLE latest_indicators (
    instrument_id VARCHAR(20) NOT NULL,
    indicator_type VARCHAR(20) NOT NULL,
    interval VARCHAR(10) NOT NULL,
    parameters JSONB NOT NULL,
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    values JSONB NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    PRIMARY KEY (instrument_id, indicator_type, interval, parameters)
);
```

#### active_patterns
```sql
CREATE TABLE active_patterns (
    id SERIAL PRIMARY KEY,
    instrument_id VARCHAR(20) NOT NULL,
    pattern_type VARCHAR(30) NOT NULL,
    interval VARCHAR(10) NOT NULL,
    start_timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    end_timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    confidence DECIMAL(5,4) NOT NULL,
    target_price DECIMAL(18, 8),
    completion_percentage INTEGER NOT NULL,
    is_completed BOOLEAN NOT NULL DEFAULT FALSE,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX active_patterns_instrument_idx ON active_patterns(instrument_id);
CREATE INDEX active_patterns_completion_idx ON active_patterns(completion_percentage);
```

#### correlation_matrix
```sql
CREATE TABLE correlation_matrix (
    instrument_id_1 VARCHAR(20) NOT NULL,
    instrument_id_2 VARCHAR(20) NOT NULL,
    interval VARCHAR(10) NOT NULL,
    correlation DECIMAL(5,4) NOT NULL,
    p_value DECIMAL(7,6) NOT NULL,
    start_timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    end_timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    PRIMARY KEY (instrument_id_1, instrument_id_2, interval)
);
```

## Place in Workflow

The Technical Analysis Service is a core component of the Instrument Analysis Workflow and serves as the primary processor of market data for technical analysis. It:

1. **Receives normalized market data** from the Market Data Service
2. **Calculates technical indicators** for various timeframes and instruments
3. **Detects chart patterns** and identifies support/resistance levels
4. **Performs correlation analysis** between instruments
5. **Distributes technical analysis results** to downstream services

The service interacts with:
- **Market Data Service** (input): Provides normalized market data for analysis
- **ML Prediction Service** (output): Uses technical indicators as input features
- **Trading Strategy Service** (output): Incorporates technical analysis into trading strategies
- **Risk Analysis Service** (output): Utilizes correlation data for risk calculations
- **Portfolio Optimization Service** (output): Uses clustering and correlation data for portfolio construction
- **Reporting Service** (output): Includes technical analysis in reports and dashboards

## Project Plan

### Phase 1: Foundation (Weeks 1-4)

#### Week 1: Setup and Infrastructure
- Set up development environment and CI/CD pipeline
- Configure Kubernetes infrastructure for the service
- Set up TimescaleDB for time-series data storage
- Implement basic service skeleton with health checks

#### Week 2: Core Indicator Calculation
- Implement moving average indicators (SMA, EMA, WMA)
- Develop momentum indicators (RSI, MACD, Stochastic)
- Create volatility indicators (Bollinger Bands, ATR)
- Build trend indicators (ADX, Parabolic SAR)

#### Week 3: Advanced Analysis Features
- Implement pattern recognition algorithms
- Develop support and resistance detection
- Create correlation analysis functionality
- Build statistical analysis components

#### Week 4: Basic API Implementation
- Implement REST API endpoints for data retrieval
- Create gRPC service for real-time updates
- Develop basic authentication and rate limiting
- Write comprehensive API documentation

### Phase 2: Enhancement (Weeks 5-8)

#### Week 5: Performance Optimization
- Optimize indicator calculation algorithms
- Implement parallel processing for multiple instruments
- Create caching mechanisms for frequently used indicators
- Develop incremental update capabilities

#### Week 6: Advanced Technical Analysis
- Implement Fibonacci retracement analysis
- Develop Elliott Wave pattern recognition
- Create harmonic pattern detection
- Build candlestick pattern recognition

#### Week 7: Real-time Processing
- Implement streaming updates for indicators
- Develop real-time pattern detection
- Create alert generation for pattern completions
- Build websocket API for live updates

#### Week 8: Integration and Testing
- Integrate with Market Data Service
- Connect with downstream services
- Perform load and stress testing
- Develop comprehensive monitoring and alerting

### Phase 3: Production Readiness (Weeks 9-12)

#### Week 9: Documentation and Training
- Create detailed service documentation
- Develop operational runbooks
- Prepare training materials
- Conduct knowledge transfer sessions

#### Week 10: Advanced Optimization
- Implement GPU acceleration for complex calculations
- Optimize database queries and indexing
- Enhance memory management
- Fine-tune resource utilization

#### Week 11: Quality Assurance
- Conduct accuracy testing against industry standards
- Perform backtesting of pattern recognition
- Validate correlation calculations
- Address any identified issues

#### Week 12: Final Preparation and Deployment
- Conduct final integration testing
- Prepare production deployment plan
- Execute staged rollout
- Monitor production performance

### Ongoing Maintenance
- Regular algorithm improvements and optimizations
- Addition of new technical indicators
- Performance monitoring and tuning
- Database maintenance and optimization