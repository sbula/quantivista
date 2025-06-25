# Benchmark Data Service

## Responsibility
Comprehensive benchmark and index data management for performance attribution, risk analysis, and portfolio comparison. Provides historical and real-time benchmark prices, compositions, and performance metrics for major indices and custom benchmarks.

## Technology Stack
- **Language**: Python + FastAPI + TimescaleDB
- **Libraries**: FastAPI, numpy, TimescaleDB connector, financial libraries
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for mathematical optimization and advanced models
- **Scaling**: Horizontal by benchmark complexity and historical data volume
- **NFRs**: P99 benchmark query < 200ms, comprehensive index coverage, daily updates

## API Specification

### Core APIs
```pseudo
// Enumerations
enum BenchmarkType {
    EQUITY_INDEX,
    BOND_INDEX,
    COMMODITY_INDEX,
    CURRENCY_INDEX,
    SECTOR_INDEX,
    CUSTOM_BENCHMARK
}

enum BenchmarkProvider {
    SP_DOW_JONES,
    MSCI,
    FTSE_RUSSELL,
    BLOOMBERG_BARCLAYS,
    NASDAQ,
    CUSTOM
}

// Data Models
struct Benchmark {
    benchmark_id: String
    benchmark_name: String
    benchmark_type: BenchmarkType
    provider: BenchmarkProvider
    currency: String
    inception_date: Date
    methodology: String
    rebalance_frequency: String
    composition: List<BenchmarkConstituent>
    metadata: Map<String, Any>
}

struct BenchmarkConstituent {
    instrument_id: String
    weight: Float
    shares: Optional<Float>
    effective_date: Date
}

struct BenchmarkPrice {
    benchmark_id: String
    date: Date
    price: Float
    total_return_index: Float
    net_return_index: Float
    dividend_yield: Optional<Float>
    market_cap: Optional<Float>
}

struct BenchmarkPerformance {
    benchmark_id: String
    period: String
    total_return: Float
    price_return: Float
    dividend_return: Float
    volatility: Float
    sharpe_ratio: Float
    max_drawdown: Float
}

// REST API Endpoints
GET /api/v1/benchmarks
    Parameters: benchmark_type, provider, currency
    Response: List<Benchmark>

GET /api/v1/benchmarks/{benchmark_id}
    Response: Benchmark

GET /api/v1/benchmarks/{benchmark_id}/prices
    Parameters: start_date, end_date
    Response: List<BenchmarkPrice>

GET /api/v1/benchmarks/{benchmark_id}/composition
    Parameters: as_of_date
    Response: List<BenchmarkConstituent>

GET /api/v1/benchmarks/{benchmark_id}/performance
    Parameters: start_date, end_date, frequency
    Response: BenchmarkPerformance

POST /api/v1/benchmarks/custom
    Request: CustomBenchmarkRequest
    Response: Benchmark

GET /api/v1/benchmarks/{benchmark_id}/correlation
    Parameters: other_benchmark_id, period
    Response: CorrelationAnalysis
```

### Event Output
```pseudo
Event benchmark_updated {
    event_id: String
    timestamp: DateTime
    benchmark_updated: BenchmarkUpdatePayload
}

struct BenchmarkUpdatePayload {
    benchmark_id: String
    benchmark_name: String
    update_type: String
    price_data: PriceDataInfo
    composition_changes: List<CompositionChangeData>
}

struct PriceDataInfo {
    date: String
    price: Float
    total_return_index: Float
    daily_return: Float
}

struct CompositionChangeData {
    instrument_id: String
    action: String
    old_weight: Float
    new_weight: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    benchmark_updated: {
        benchmark_id: "SPX",
        benchmark_name: "S&P 500 Index",
        update_type: "daily_price_update",
        price_data: {
            date: "2025-06-21",
            price: 4250.75,
            total_return_index: 8542.30,
            daily_return: 0.0125
        },
        composition_changes: [
            {
                instrument_id: "TSLA",
                action: "weight_change",
                old_weight: 0.0185,
                new_weight: 0.0192
            }
        ]
    }
}
```

## Database Schema

### PostgreSQL (Master Data)
```pseudo
Table benchmarks {
    id: UUID (primary key, auto-generated)
    benchmark_id: String (required, unique, max_length: 20)
    benchmark_name: String (required, max_length: 200)
    benchmark_type: String (required, max_length: 50)
    provider: String (required, max_length: 50)
    currency: String (required, max_length: 3)
    inception_date: Date (required)
    methodology: String
    rebalance_frequency: String (max_length: 20)
    metadata: JSON
    is_active: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table benchmark_constituents {
    id: UUID (primary key, auto-generated)
    benchmark_id: String (required, max_length: 20, foreign_key: benchmarks.benchmark_id)
    instrument_id: String (required, max_length: 20)
    weight: Decimal (required, precision: 10, scale: 6)
    shares: Integer
    effective_date: Date (required)
    expiry_date: Date
}
```

### TimescaleDB (Price Data)
```pseudo
Table benchmark_prices_ts {
    timestamp: Timestamp (required, partition_key)
    benchmark_id: String (required, max_length: 20)
    price: Decimal (required, precision: 15, scale: 6)
    total_return_index: Decimal (precision: 15, scale: 6)
    net_return_index: Decimal (precision: 15, scale: 6)
    dividend_yield: Decimal (precision: 8, scale: 6)
    market_cap: Integer
    volume: Integer

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 month)
    partition_dimension: benchmark_id (partitions: 8)
}

Table benchmark_performance_ts {
    timestamp: Timestamp (required, partition_key)
    benchmark_id: String (required, max_length: 20)
    period: String (required, max_length: 10)
    total_return: Decimal (precision: 10, scale: 6)
    price_return: Decimal (precision: 10, scale: 6)
    dividend_return: Decimal (precision: 10, scale: 6)
    volatility: Decimal (precision: 8, scale: 6)
    sharpe_ratio: Decimal (precision: 8, scale: 4)
    max_drawdown: Decimal (precision: 8, scale: 6)

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
}
```

## External Data Sources

### **Primary Benchmark Providers:**
- **S&P Dow Jones Indices**: S&P 500, S&P 400, S&P 600, Dow Jones indices
- **MSCI**: MSCI World, MSCI Emerging Markets, MSCI EAFE, sector indices
- **FTSE Russell**: Russell 1000, Russell 2000, Russell 3000, FTSE indices
- **NASDAQ**: NASDAQ Composite, NASDAQ 100, technology indices
- **Bloomberg Barclays**: Fixed income indices, bond benchmarks

### **Secondary Sources:**
- **Morningstar**: Category benchmarks, style box indices
- **CRSP**: Academic research indices, historical data
- **Wilshire**: Wilshire 5000, broad market indices
- **Custom Calculations**: Client-specific benchmarks, peer group comparisons

## Implementation Estimation

### Priority: **HIGH** (Critical for performance analysis)
### Estimated Time: **4-5 weeks**

#### Week 1-2: Core Benchmark Engine
- Python service setup with FastAPI
- Basic benchmark data management
- External data source integration
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3: Advanced Features
- Performance calculation engine
- Custom benchmark creation
- Correlation analysis
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 4-5: Integration & Optimization
- TimescaleDB optimization
- Integration with performance attribution
- Historical data backfill
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **10 dev-weeks**
### Team Size: **2 developers** (1 senior Python developer, 1 quantitative analyst)
### Dependencies: S&P, MSCI, Russell data feeds, TimescaleDB, Reference Data Service

### Success Criteria:
- P99 benchmark query < 200ms
- Comprehensive index coverage (500+ benchmarks)
- Daily benchmark updates
- Accurate performance calculations
- Support for custom benchmark creation
