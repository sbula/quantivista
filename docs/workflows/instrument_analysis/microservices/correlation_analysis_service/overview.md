# Correlation Analysis Service

## Responsibility
Efficient correlation computation with cluster-based optimization. Implements two-tier architecture: daily full correlation matrices and real-time cluster correlations for 100x+ performance improvement.

## Technology Stack
- **Language**: Rust + nalgebra + rayon + Apache Arrow
- **Libraries**: nalgebra (linear algebra), rayon (parallelism), polars (data processing)
- **Scaling**: Horizontal by correlation timeframes, optimized for cluster-based computation
- **NFRs**: Daily full matrix < 10 minutes for 10K instruments, real-time cluster updates < 100ms

## API Specification

### Core APIs
```pseudo
// Enumerations
enum CorrelationMethod {
    PEARSON,
    SPEARMAN,
    KENDALL
}

// Data Models
struct CorrelationRequest {
    instruments: List<String>
    method: CorrelationMethod
    period_days: Integer
    timeframe: String
    use_clusters: Boolean
}

struct CorrelationResponse {
    correlation_id: String
    method: CorrelationMethod
    matrix: CorrelationMatrix
    computation_time_ms: Float
    instruments_count: Integer
    cluster_optimized: Boolean
}

struct CorrelationMatrix {
    instruments: List<String>
    correlations: Matrix<Float>  // 2D matrix
    timestamp: DateTime
    confidence_scores: Matrix<Float>  // 2D matrix
}

// REST API Endpoints
POST /api/v1/correlations/compute
    Request: CorrelationRequest
    Response: CorrelationResponse

GET /api/v1/correlations/daily/{date}
    Response: CorrelationResponse

GET /api/v1/correlations/cluster/{cluster_id}
    Response: CorrelationResponse
```

### Event Output
```pseudo
Event correlation_matrix_updated {
    event_id: String
    timestamp: DateTime
    correlation_update: CorrelationUpdateData
}

struct CorrelationUpdateData {
    correlation_id: String
    update_type: String
    method: String
    period_days: Integer
    instruments_count: Integer
    computation_time_ms: Integer
    cluster_optimized: Boolean
    optimization_ratio: Float
    matrix_summary: MatrixSummaryData
}

struct MatrixSummaryData {
    avg_correlation: Float
    max_correlation: Float
    min_correlation: Float
    high_correlation_pairs: Integer
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    correlation_update: {
        correlation_id: "daily_20250621",
        update_type: "daily_full_matrix",
        method: "pearson",
        period_days: 30,
        instruments_count: 5000,
        computation_time_ms: 450000,
        cluster_optimized: true,
        optimization_ratio: 189.5,
        matrix_summary: {
            avg_correlation: 0.15,
            max_correlation: 0.95,
            min_correlation: -0.45,
            high_correlation_pairs: 234
        }
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table correlation_configurations {
    id: UUID (primary key, auto-generated)
    config_name: String (required, max_length: 100)
    method: String (required, max_length: 20)
    period_days: Integer (required)
    instruments: JSON (required)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table correlation_computations {
    id: UUID (primary key, auto-generated)
    correlation_id: String (required, max_length: 100)
    method: String (required, max_length: 20)
    computation_date: Date (required)
    instruments_count: Integer
    computation_time_ms: Float
    cluster_optimized: Boolean
    created_at: Timestamp (default: now)
}
```

### TimescaleDB (Query Side)
```pseudo
Table correlation_matrices_ts {
    timestamp: Timestamp (required, partition_key)
    correlation_id: String (required, max_length: 100)
    method: String (required, max_length: 20)
    period_days: Integer (required)
    matrix_data: Binary  // Compressed correlation matrix
    instruments: JSON
    computation_metadata: JSON

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
}
```

## Implementation Estimation

### Priority: **HIGH** (Critical for portfolio optimization)
### Estimated Time: **5-6 weeks**

#### Week 1-2: Core Correlation Engine
- Rust service with nalgebra integration
- Basic correlation computation (Pearson, Spearman)
- Matrix storage and compression
- **Effort**: 2 senior Rust developers × 2 weeks = 4 dev-weeks

#### Week 3-4: Cluster Optimization
- Two-tier correlation architecture
- Cluster-based correlation computation
- Real-time incremental updates
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 5-6: Performance & Integration
- SIMD optimizations and parallel processing
- Integration with clustering service
- Performance testing (10K+ instruments)
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **12 dev-weeks**
### Team Size: **2 senior Rust developers**
### Dependencies: Clustering service, market data, TimescaleDB

### Success Criteria:
- Daily full matrix < 10 minutes for 10K instruments
- Real-time cluster updates < 100ms
- 100x+ performance improvement with clustering
- Support multiple correlation methods
- Compressed matrix storage
