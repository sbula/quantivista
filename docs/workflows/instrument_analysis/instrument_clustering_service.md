# Instrument Clustering Service

## Purpose
The Instrument Clustering Service is responsible for grouping financial instruments based on various characteristics and behaviors using advanced machine learning techniques. It identifies relationships between instruments, creates dynamic clusters, and provides similarity metrics that can be used for portfolio diversification, risk management, and trading strategy development.

## Strict Boundaries

### Responsibilities
- Multi-dimensional clustering analysis of financial instruments
- Dynamic cluster updates based on changing market conditions
- Similarity scoring and ranking between instruments
- Cluster stability monitoring and quality assessment
- Distribution of clustering results to downstream services

### Non-Responsibilities
- Does NOT collect or normalize market data
- Does NOT calculate technical indicators
- Does NOT make trading decisions
- Does NOT execute trades
- Does NOT perform fundamental analysis
- Does NOT manage user preferences or authentication

## API Design (API-First)

### REST API

#### GET /api/v1/clusters
Retrieves current instrument clusters.

**Query Parameters:**
- `clustering_method` (string, optional): Clustering method (kmeans, hierarchical, dbscan, default: kmeans)
- `feature_set` (string, optional): Feature set used for clustering (price_based, fundamental, mixed, default: price_based)
- `num_clusters` (integer, optional): Number of clusters (for k-means, default: 10)
- `min_cluster_size` (integer, optional): Minimum cluster size (for DBSCAN, default: 5)
- `include_features` (boolean, optional): Include feature values in response (default: false)

**Response:**
```json
{
  "clustering_id": "2025-06-20-price-based-kmeans",
  "clustering_method": "kmeans",
  "feature_set": "price_based",
  "num_clusters": 10,
  "created_at": "2025-06-20T12:00:00Z",
  "stability_score": 0.85,
  "silhouette_score": 0.72,
  "clusters": [
    {
      "cluster_id": 1,
      "size": 42,
      "centroid_features": {
        "volatility_30d": 0.15,
        "beta": 1.2,
        "momentum_90d": 0.08
      },
      "characteristic_instruments": ["AAPL", "MSFT", "GOOGL"],
      "instruments": ["AAPL", "MSFT", "GOOGL", "AMZN", "FB"]
    },
    {
      "cluster_id": 2,
      "size": 38,
      "centroid_features": {
        "volatility_30d": 0.22,
        "beta": 1.5,
        "momentum_90d": 0.12
      },
      "characteristic_instruments": ["TSLA", "NVDA", "AMD"],
      "instruments": ["TSLA", "NVDA", "AMD", "SHOP", "SQ"]
    }
  ]
}
```

#### GET /api/v1/clusters/history
Retrieves historical clustering results.

**Query Parameters:**
- `from` (string, required): Start timestamp (ISO 8601)
- `to` (string, required): End timestamp (ISO 8601)
- `clustering_method` (string, optional): Clustering method (kmeans, hierarchical, dbscan)
- `feature_set` (string, optional): Feature set used for clustering (price_based, fundamental, mixed)
- `limit` (integer, optional): Maximum number of results (default: 10)

**Response:**
```json
{
  "history": [
    {
      "clustering_id": "2025-06-20-price-based-kmeans",
      "clustering_method": "kmeans",
      "feature_set": "price_based",
      "num_clusters": 10,
      "created_at": "2025-06-20T12:00:00Z",
      "stability_score": 0.85,
      "silhouette_score": 0.72
    },
    {
      "clustering_id": "2025-06-19-price-based-kmeans",
      "clustering_method": "kmeans",
      "feature_set": "price_based",
      "num_clusters": 10,
      "created_at": "2025-06-19T12:00:00Z",
      "stability_score": 0.83,
      "silhouette_score": 0.71
    }
  ]
}
```

#### GET /api/v1/instruments/{instrument_id}/similar
Retrieves instruments similar to the specified instrument.

**Path Parameters:**
- `instrument_id` (string, required): Instrument identifier

**Query Parameters:**
- `feature_set` (string, optional): Feature set for similarity calculation (price_based, fundamental, mixed, default: price_based)
- `limit` (integer, optional): Maximum number of similar instruments to return (default: 10)
- `min_similarity` (float, optional): Minimum similarity score (0.0-1.0, default: 0.7)

**Response:**
```json
{
  "instrument_id": "AAPL",
  "feature_set": "price_based",
  "cluster_id": 1,
  "similar_instruments": [
    {
      "instrument_id": "MSFT",
      "similarity_score": 0.92,
      "common_features": {
        "sector": "Technology",
        "market_cap_category": "Large Cap",
        "volatility_category": "Medium"
      }
    },
    {
      "instrument_id": "GOOGL",
      "similarity_score": 0.85,
      "common_features": {
        "sector": "Technology",
        "market_cap_category": "Large Cap",
        "volatility_category": "Medium"
      }
    }
  ]
}
```

### gRPC API

```protobuf
syntax = "proto3";

package instrument_clustering.v1;

import "google/protobuf/timestamp.proto";

service InstrumentClusteringService {
  // Get current clustering results
  rpc GetClusters(GetClustersRequest) returns (GetClustersResponse);
  
  // Get historical clustering results
  rpc GetClusteringHistory(GetClusteringHistoryRequest) returns (GetClusteringHistoryResponse);
  
  // Get similar instruments
  rpc GetSimilarInstruments(GetSimilarInstrumentsRequest) returns (GetSimilarInstrumentsResponse);
  
  // Stream cluster updates
  rpc StreamClusterUpdates(StreamClusterUpdatesRequest) returns (stream ClusterUpdate);
}

message GetClustersRequest {
  string clustering_method = 1; // "kmeans", "hierarchical", "dbscan"
  string feature_set = 2; // "price_based", "fundamental", "mixed"
  int32 num_clusters = 3;
  int32 min_cluster_size = 4;
  bool include_features = 5;
}

message GetClustersResponse {
  string clustering_id = 1;
  string clustering_method = 2;
  string feature_set = 3;
  int32 num_clusters = 4;
  google.protobuf.Timestamp created_at = 5;
  double stability_score = 6;
  double silhouette_score = 7;
  repeated Cluster clusters = 8;
}

message Cluster {
  int32 cluster_id = 1;
  int32 size = 2;
  map<string, double> centroid_features = 3;
  repeated string characteristic_instruments = 4;
  repeated string instruments = 5;
}

message GetClusteringHistoryRequest {
  google.protobuf.Timestamp from = 1;
  google.protobuf.Timestamp to = 2;
  string clustering_method = 3;
  string feature_set = 4;
  int32 limit = 5;
}

message GetClusteringHistoryResponse {
  repeated ClusteringMetadata history = 1;
}

message ClusteringMetadata {
  string clustering_id = 1;
  string clustering_method = 2;
  string feature_set = 3;
  int32 num_clusters = 4;
  google.protobuf.Timestamp created_at = 5;
  double stability_score = 6;
  double silhouette_score = 7;
}

message GetSimilarInstrumentsRequest {
  string instrument_id = 1;
  string feature_set = 2;
  int32 limit = 3;
  double min_similarity = 4;
}

message GetSimilarInstrumentsResponse {
  string instrument_id = 1;
  string feature_set = 2;
  int32 cluster_id = 3;
  repeated SimilarInstrument similar_instruments = 4;
}

message SimilarInstrument {
  string instrument_id = 1;
  double similarity_score = 2;
  map<string, string> common_features = 3;
}

message StreamClusterUpdatesRequest {
  string clustering_method = 1;
  string feature_set = 2;
  repeated string instrument_ids = 3;
}

message ClusterUpdate {
  string clustering_id = 1;
  google.protobuf.Timestamp timestamp = 2;
  repeated ClusterAssignment assignments = 3;
}

message ClusterAssignment {
  string instrument_id = 1;
  int32 cluster_id = 2;
  double distance_to_centroid = 3;
}
```

## Data Model

### Core Entities

#### ClusteringRun
Represents a single execution of the clustering algorithm.

**Attributes:**
- `id` (string): Unique identifier for the clustering run
- `method` (enum): Clustering method used (kmeans, hierarchical, dbscan)
- `feature_set` (enum): Feature set used for clustering (price_based, fundamental, mixed)
- `parameters` (map): Algorithm-specific parameters
- `created_at` (datetime): When the clustering was performed
- `stability_score` (float): Measure of cluster stability compared to previous runs
- `silhouette_score` (float): Measure of cluster quality
- `num_clusters` (integer): Number of clusters generated

#### Cluster
Represents a group of similar instruments.

**Attributes:**
- `id` (integer): Cluster identifier
- `clustering_run_id` (string): Reference to the clustering run
- `size` (integer): Number of instruments in the cluster
- `centroid_features` (map): Feature values at the cluster centroid
- `characteristic_instruments` (array): Most representative instruments in the cluster
- `stability_score` (float): Measure of cluster stability over time

#### ClusterMembership
Represents an instrument's membership in a cluster.

**Attributes:**
- `instrument_id` (string): Reference to the instrument
- `cluster_id` (integer): Reference to the cluster
- `clustering_run_id` (string): Reference to the clustering run
- `distance_to_centroid` (float): Distance from the instrument to the cluster centroid
- `membership_strength` (float): Strength of membership (for fuzzy clustering)

#### InstrumentFeature
Represents a feature value for an instrument.

**Attributes:**
- `instrument_id` (string): Reference to the instrument
- `feature_name` (string): Name of the feature
- `feature_value` (float): Value of the feature
- `timestamp` (datetime): When the feature was calculated
- `feature_set` (enum): Feature set this feature belongs to

#### SimilarityScore
Represents a similarity measure between two instruments.

**Attributes:**
- `instrument_id_1` (string): First instrument
- `instrument_id_2` (string): Second instrument
- `feature_set` (enum): Feature set used for similarity calculation
- `similarity_score` (float): Similarity score (0.0-1.0)
- `common_features` (map): Features that contribute to the similarity
- `timestamp` (datetime): When the similarity was calculated

## DB Schema (CQRS Pattern)

### Write Schema (Command Side)

#### clustering_runs
```sql
CREATE TABLE clustering_runs (
    id VARCHAR(50) PRIMARY KEY,
    method VARCHAR(20) NOT NULL,
    feature_set VARCHAR(20) NOT NULL,
    parameters JSONB NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    status VARCHAR(20) NOT NULL DEFAULT 'running',
    error_message TEXT,
    num_clusters INTEGER,
    stability_score DECIMAL(4,3),
    silhouette_score DECIMAL(4,3),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX clustering_runs_created_at_idx ON clustering_runs(created_at);
CREATE INDEX clustering_runs_method_feature_set_idx ON clustering_runs(method, feature_set);
```

#### clusters
```sql
CREATE TABLE clusters (
    id SERIAL PRIMARY KEY,
    clustering_run_id VARCHAR(50) NOT NULL REFERENCES clustering_runs(id),
    cluster_number INTEGER NOT NULL,
    size INTEGER NOT NULL,
    centroid_features JSONB NOT NULL,
    characteristic_instruments JSONB,
    stability_score DECIMAL(4,3),
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT clusters_run_number_idx UNIQUE (clustering_run_id, cluster_number)
);

CREATE INDEX clusters_clustering_run_id_idx ON clusters(clustering_run_id);
```

#### cluster_memberships
```sql
CREATE TABLE cluster_memberships (
    id SERIAL PRIMARY KEY,
    instrument_id VARCHAR(20) NOT NULL,
    cluster_id INTEGER NOT NULL REFERENCES clusters(id),
    clustering_run_id VARCHAR(50) NOT NULL REFERENCES clustering_runs(id),
    distance_to_centroid DECIMAL(10,6) NOT NULL,
    membership_strength DECIMAL(4,3) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT cluster_memberships_unique_idx UNIQUE (instrument_id, clustering_run_id)
);

CREATE INDEX cluster_memberships_instrument_id_idx ON cluster_memberships(instrument_id);
CREATE INDEX cluster_memberships_cluster_id_idx ON cluster_memberships(cluster_id);
```

#### instrument_features
```sql
CREATE TABLE instrument_features (
    id SERIAL PRIMARY KEY,
    instrument_id VARCHAR(20) NOT NULL,
    feature_name VARCHAR(50) NOT NULL,
    feature_value DECIMAL(18, 8) NOT NULL,
    feature_set VARCHAR(20) NOT NULL,
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT instrument_features_unique_idx UNIQUE (instrument_id, feature_name, feature_set, timestamp)
);

CREATE INDEX instrument_features_instrument_id_idx ON instrument_features(instrument_id);
CREATE INDEX instrument_features_feature_set_idx ON instrument_features(feature_set);
CREATE INDEX instrument_features_timestamp_idx ON instrument_features(timestamp);
```

#### similarity_calculations
```sql
CREATE TABLE similarity_calculations (
    id SERIAL PRIMARY KEY,
    instrument_id_1 VARCHAR(20) NOT NULL,
    instrument_id_2 VARCHAR(20) NOT NULL,
    feature_set VARCHAR(20) NOT NULL,
    similarity_score DECIMAL(4,3) NOT NULL,
    common_features JSONB,
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT similarity_calculations_unique_idx UNIQUE (instrument_id_1, instrument_id_2, feature_set, timestamp)
);

CREATE INDEX similarity_calculations_instrument_1_idx ON similarity_calculations(instrument_id_1);
CREATE INDEX similarity_calculations_instrument_2_idx ON similarity_calculations(instrument_id_2);
CREATE INDEX similarity_calculations_score_idx ON similarity_calculations(similarity_score DESC);
```

### Read Schema (Query Side)

#### current_clusters
```sql
CREATE TABLE current_clusters (
    id SERIAL PRIMARY KEY,
    clustering_id VARCHAR(50) NOT NULL,
    method VARCHAR(20) NOT NULL,
    feature_set VARCHAR(20) NOT NULL,
    cluster_number INTEGER NOT NULL,
    size INTEGER NOT NULL,
    centroid_features JSONB NOT NULL,
    characteristic_instruments JSONB,
    instruments JSONB NOT NULL,
    stability_score DECIMAL(4,3),
    created_at TIMESTAMP WITH TIME ZONE NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT current_clusters_method_feature_cluster_idx UNIQUE (method, feature_set, cluster_number)
);

CREATE INDEX current_clusters_method_feature_set_idx ON current_clusters(method, feature_set);
```

#### instrument_cluster_assignments
```sql
CREATE TABLE instrument_cluster_assignments (
    instrument_id VARCHAR(20) NOT NULL,
    clustering_method VARCHAR(20) NOT NULL,
    feature_set VARCHAR(20) NOT NULL,
    cluster_number INTEGER NOT NULL,
    distance_to_centroid DECIMAL(10,6) NOT NULL,
    membership_strength DECIMAL(4,3) NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    PRIMARY KEY (instrument_id, clustering_method, feature_set)
);

CREATE INDEX instrument_cluster_assignments_cluster_idx ON instrument_cluster_assignments(clustering_method, feature_set, cluster_number);
```

#### instrument_similarities
```sql
CREATE TABLE instrument_similarities (
    instrument_id_1 VARCHAR(20) NOT NULL,
    instrument_id_2 VARCHAR(20) NOT NULL,
    feature_set VARCHAR(20) NOT NULL,
    similarity_score DECIMAL(4,3) NOT NULL,
    common_features JSONB,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    PRIMARY KEY (instrument_id_1, instrument_id_2, feature_set)
);

CREATE INDEX instrument_similarities_instrument_1_score_idx ON instrument_similarities(instrument_id_1, similarity_score DESC);
```

#### clustering_history
```sql
CREATE TABLE clustering_history (
    clustering_id VARCHAR(50) PRIMARY KEY,
    method VARCHAR(20) NOT NULL,
    feature_set VARCHAR(20) NOT NULL,
    parameters JSONB NOT NULL,
    num_clusters INTEGER NOT NULL,
    stability_score DECIMAL(4,3) NOT NULL,
    silhouette_score DECIMAL(4,3) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL,
    summary_stats JSONB
);

CREATE INDEX clustering_history_created_at_idx ON clustering_history(created_at DESC);
CREATE INDEX clustering_history_method_feature_set_idx ON clustering_history(method, feature_set);
```

## Place in Workflow

The Instrument Clustering Service is a key component of the Instrument Analysis Workflow and works alongside the Technical Analysis Service to provide comprehensive instrument analysis. It:

1. **Receives instrument features** from various sources (Market Data Service, Technical Analysis Service)
2. **Performs clustering analysis** to group similar instruments
3. **Calculates similarity scores** between instruments
4. **Monitors cluster stability** over time
5. **Distributes clustering results** to downstream services

The service interacts with:
- **Market Data Service** (input): Provides basic instrument metadata and price data
- **Technical Analysis Service** (input): Supplies technical indicators as features for clustering
- **ML Prediction Service** (output): Uses clustering information for prediction models
- **Trading Strategy Service** (output): Incorporates instrument relationships into trading strategies
- **Risk Analysis Service** (output): Utilizes clustering for risk assessment
- **Portfolio Optimization Service** (output): Leverages clustering for diversification strategies
- **Reporting Service** (output): Includes clustering analysis in reports and dashboards

## Project Plan

### Phase 1: Foundation (Weeks 1-4)

#### Week 1: Setup and Infrastructure
- Set up development environment and CI/CD pipeline
- Configure Kubernetes infrastructure for the service
- Set up PostgreSQL for data storage
- Implement basic service skeleton with health checks

#### Week 2: Feature Engineering Framework
- Develop feature extraction pipeline
- Implement feature normalization and standardization
- Create feature selection mechanisms
- Build feature storage and retrieval system

#### Week 3: Core Clustering Algorithms
- Implement K-means clustering
- Develop hierarchical clustering
- Create DBSCAN density-based clustering
- Build cluster evaluation metrics

#### Week 4: Basic API Implementation
- Implement REST API endpoints for data retrieval
- Create gRPC service for real-time updates
- Develop basic authentication and rate limiting
- Write comprehensive API documentation

### Phase 2: Enhancement (Weeks 5-8)

#### Week 5: Advanced Clustering Features
- Implement fuzzy clustering (soft assignments)
- Develop dynamic time-based clustering
- Create anomaly detection within clusters
- Build cluster visualization capabilities

#### Week 6: Similarity Calculation
- Implement multiple similarity metrics
- Develop efficient similarity calculation
- Create similarity-based recommendations
- Build similarity visualization

#### Week 7: Cluster Stability Analysis
- Implement cluster stability metrics
- Develop change detection algorithms
- Create historical cluster tracking
- Build cluster transition analysis

#### Week 8: Integration and Testing
- Integrate with Market Data Service
- Connect with Technical Analysis Service
- Perform load and stress testing
- Develop comprehensive monitoring and alerting

### Phase 3: Production Readiness (Weeks 9-12)

#### Week 9: Documentation and Training
- Create detailed service documentation
- Develop operational runbooks
- Prepare training materials
- Conduct knowledge transfer sessions

#### Week 10: Performance Optimization
- Optimize clustering algorithms
- Implement distributed clustering for large datasets
- Enhance database query performance
- Fine-tune resource utilization

#### Week 11: Advanced Analytics
- Implement cluster interpretation algorithms
- Develop feature importance analysis
- Create cluster-based forecasting
- Build advanced visualization tools

#### Week 12: Final Preparation and Deployment
- Conduct final integration testing
- Prepare production deployment plan
- Execute staged rollout
- Monitor production performance

### Ongoing Maintenance
- Regular algorithm improvements and optimizations
- Feature engineering enhancements
- Performance monitoring and tuning
- Database maintenance and optimization