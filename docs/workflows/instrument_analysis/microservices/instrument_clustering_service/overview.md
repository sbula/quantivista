# Instrument Clustering Service

## Responsibility
Multi-dimensional instrument clustering with dynamic re-clustering for correlation optimization. Groups instruments by sector, market cap, volatility, and trading patterns to enable efficient cluster-based correlation analysis.

## Technology Stack
- **Language**: Python + scikit-learn + JAX + NetworkX
- **Libraries**: scikit-learn, UMAP, HDBSCAN, NetworkX, JAX for GPU acceleration
- **Scaling**: Horizontal by clustering algorithms, GPU acceleration for large datasets
- **NFRs**: P99 clustering latency < 30s for 10K instruments, silhouette score > 0.7

## API Specification

### Core APIs
```pseudo
// Enumerations
enum ClusteringMethod {
    KMEANS,
    HIERARCHICAL,
    DBSCAN,
    HDBSCAN,
    SPECTRAL
}

enum FeatureType {
    SECTOR,
    MARKET_CAP,
    VOLATILITY,
    CORRELATION,
    VOLUME,
    MOMENTUM,
    VALUE_METRICS
}

// Data Models
struct ClusteringRequest {
    instruments: List<String>
    method: ClusteringMethod
    features: List<FeatureType>
    num_clusters: Optional<Integer>
    min_cluster_size: Integer
    max_cluster_size: Integer
    rebalance_threshold: Float
}

struct ClusteringResponse {
    clustering_id: String
    method: ClusteringMethod
    clusters: Map<String, List<String>>  // cluster_id -> instrument_list
    cluster_metrics: Map<String, ClusterMetrics>
    silhouette_score: Float
    processing_time_ms: Float
    feature_importance: Map<String, Float>
}

struct ClusterMetrics {
    cluster_id: String
    size: Integer
    cohesion: Float
    separation: Float
    representative_instrument: String
    avg_market_cap: Float
    avg_volatility: Float
    dominant_sector: String
}

struct ClusterRebalanceRequest {
    clustering_id: String
    threshold: Float
    force_rebalance: Boolean
}

// REST API Endpoints
POST /api/v1/cluster
    Request: ClusteringRequest
    Response: ClusteringResponse

GET /api/v1/clusters/current
    Response: ClusteringResponse

POST /api/v1/clusters/rebalance
    Request: ClusterRebalanceRequest
    Response: ClusteringResponse

GET /api/v1/clusters/{cluster_id}/instruments
    Response: List<String>

GET /api/v1/instruments/{instrument_id}/cluster
    Response: ClusterAssignment
```

### Event Output
```pseudo
Event clustering_updated {
    event_id: String
    timestamp: DateTime
    clustering_update: ClusteringUpdateData
}

struct ClusteringUpdateData {
    clustering_id: String
    method: String
    clusters: ClustersData
    cluster_metrics: ClusterMetricsData
    silhouette_score: Float
    feature_importance: FeatureImportanceData
}

struct ClustersData {
    tech_large_cap: List<String>
    financial_banks: List<String>
    energy_oil: List<String>
}

struct ClusterMetricsData {
    tech_large_cap: ClusterMetricInfo
}

struct ClusterMetricInfo {
    size: Integer
    cohesion: Float
    separation: Float
    representative_instrument: String
    avg_market_cap: Integer
    avg_volatility: Float
    dominant_sector: String
}

struct FeatureImportanceData {
    sector: Float
    market_cap: Float
    volatility: Float
    correlation: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    clustering_update: {
        clustering_id: "clustering_20250621_001",
        method: "hdbscan",
        clusters: {
            tech_large_cap: ["AAPL", "MSFT", "GOOGL", "AMZN"],
            financial_banks: ["JPM", "BAC", "WFC", "C"],
            energy_oil: ["XOM", "CVX", "COP", "EOG"]
        },
        cluster_metrics: {
            tech_large_cap: {
                size: 4,
                cohesion: 0.85,
                separation: 0.72,
                representative_instrument: "AAPL",
                avg_market_cap: 2500000000000,
                avg_volatility: 0.25,
                dominant_sector: "technology"
            }
        },
        silhouette_score: 0.78,
        feature_importance: {
            sector: 0.45,
            market_cap: 0.30,
            volatility: 0.15,
            correlation: 0.10
        }
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table clustering_configurations {
    id: UUID (primary key, auto-generated)
    config_name: String (required, max_length: 100)
    method: String (required, max_length: 20)
    features: JSON (required)
    parameters: JSON (required)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table instrument_clusters {
    id: UUID (primary key, auto-generated)
    clustering_id: String (required, max_length: 100)
    cluster_id: String (required, max_length: 100)
    instrument_id: String (required, max_length: 20)
    cluster_assignment_confidence: Float
    created_at: Timestamp (default: now)

    // Constraints
    unique_clustering_instrument: (clustering_id, instrument_id)
}

Table cluster_metrics {
    id: UUID (primary key, auto-generated)
    clustering_id: String (required, max_length: 100)
    cluster_id: String (required, max_length: 100)
    size: Integer (required)
    cohesion: Float
    separation: Float
    representative_instrument: String (max_length: 20)
    metrics: JSON
    created_at: Timestamp (default: now)
}
```

### Redis Caching
```pseudo
Cache clustering_cache {
    // Current clusters
    "clusters:current": ClusteringResponse (TTL: 1h)

    // Instrument to cluster mapping
    "instrument_cluster:{instrument_id}": String (TTL: 1h)

    // Cluster representatives
    "cluster_rep:{cluster_id}": String (TTL: 1h)

    // Clustering history
    "clustering_history:{date}": String (TTL: 7d)
}
```

## Implementation Estimation

### Priority: **HIGH** (Enables correlation optimization)
### Estimated Time: **4-5 weeks**

#### Week 1-2: Core Clustering Engine
- Python service with scikit-learn integration
- Multiple clustering algorithm implementations
- Feature engineering for financial instruments
- **Effort**: 2 ML engineers × 2 weeks = 4 dev-weeks

#### Week 3: Advanced Features
- Dynamic re-clustering and optimization
- Cluster quality metrics and validation
- GPU acceleration with JAX
- **Effort**: 1 ML engineer × 1 week = 1 dev-week

#### Week 4-5: Integration & Testing
- Integration with market data and correlation services
- Performance testing with 10K+ instruments
- Cluster stability and quality validation
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **9 dev-weeks**
### Team Size: **2 ML engineers + 1 developer**
### Dependencies: Market data, fundamental data, GPU infrastructure

### Success Criteria:
- Cluster 10K+ instruments in < 30 seconds
- Achieve silhouette score > 0.7
- Support 5+ clustering algorithms
- Dynamic re-clustering capability
- Correlation computation optimization > 100x
