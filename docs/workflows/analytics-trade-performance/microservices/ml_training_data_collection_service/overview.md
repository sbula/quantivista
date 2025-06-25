# ML Training Data Collection Service

## Responsibility
Systematic collection and preparation of high-quality ML training datasets from trading decisions, market context, and execution outcomes. Creates properly labeled datasets for training and improving ML models across all trading workflows.

## Technology Stack
- **Language**: Python + Polars + DuckDB + Apache Arrow for high-performance data processing
- **Libraries**: Polars for DataFrame operations, DuckDB for complex queries, Apache Arrow for columnar data
- **Data Processing**: Polars for high-performance feature engineering (10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for mathematical optimization and advanced models
- **Storage**: Apache Parquet for efficient data storage, Apache Arrow for zero-copy transfers
- **ML Pipeline**: MLflow for experiment tracking, DVC for data versioning
- **Scaling**: Horizontal by data volume, vertical for feature engineering
- **NFRs**: P99 data processing < 5s, 100% data capture, high-quality labeled datasets

## API Specification

### Internal APIs

#### Data Collection API
```pseudo
// Enumerations
enum DecisionOutcome {
    EXCELLENT,      // >20 bps outperformance
    GOOD,          // 10-20 bps outperformance  
    NEUTRAL,       // ±10 bps performance
    POOR,          // 10-20 bps underperformance
    BAD            // >20 bps underperformance
}

enum MarketRegime {
    TRENDING_UP,
    TRENDING_DOWN,
    SIDEWAYS,
    HIGH_VOLATILITY,
    LOW_VOLATILITY,
    CRISIS
}

enum DatasetType {
    SIGNAL_GENERATION,
    RISK_PREDICTION,
    EXECUTION_OPTIMIZATION,
    MARKET_PREDICTION,
    PORTFOLIO_OPTIMIZATION
}

// Data Models
struct TrainingDataRequest {
    decision_id: String
    include_tick_data: Boolean
    include_market_context: Boolean
    include_sentiment_data: Boolean
    lookback_minutes: Integer
    lookahead_minutes: Integer
}

struct TrainingDataResponse {
    dataset_id: String
    decision_context: DecisionContext
    market_context: MarketContext
    execution_context: ExecutionContext
    outcome_labels: OutcomeLabels
    feature_vector: FeatureVector
    data_quality_score: Float
}

struct DecisionContext {
    decision_id: String
    signal_strength: Float
    confidence_score: Float
    risk_score: Float
    position_size: Float
    decision_timestamp: DateTime
    decision_reasoning: String
    timeframe_consensus: Map<String, Float>
}

struct MarketContext {
    market_regime: MarketRegime
    volatility_level: Float
    liquidity_score: Float
    sentiment_score: Float
    technical_indicators: Map<String, Float>
    correlation_environment: Map<String, Float>
    economic_indicators: Map<String, Float>
}

struct ExecutionContext {
    order_book_imbalance: Float
    bid_ask_spread: Float
    market_impact_estimate: Float
    venue_liquidity: Map<String, Float>
    execution_algorithm: String
    execution_parameters: Map<String, Float>
}

struct OutcomeLabels {
    decision_outcome: DecisionOutcome
    execution_quality: Float
    cost_efficiency: Float
    timing_quality: Float
    risk_adjusted_return: Float
    benchmark_outperformance: Float
    attribution_factors: Map<String, Float>
}
```

#### Feature Engineering API
```pseudo
// Feature engineering models
struct FeatureEngineeringRequest {
    raw_data: RawMarketData
    decision_context: DecisionContext
    feature_types: List<String>
    normalization_method: String
}

struct FeatureEngineeringResponse {
    feature_vector: FeatureVector
    feature_metadata: FeatureMetadata
    data_quality_metrics: DataQualityMetrics
}

struct FeatureVector {
    market_microstructure_features: Map<String, Float>
    sentiment_features: Map<String, Float>
    technical_features: Map<String, Float>
    risk_features: Map<String, Float>
    temporal_features: Map<String, Float>
    cross_asset_features: Map<String, Float>
}

struct FeatureMetadata {
    feature_names: List<String>
    feature_types: List<String>
    normalization_stats: Map<String, NormalizationStats>
    feature_importance: Map<String, Float>
    correlation_matrix: Matrix
}

// REST API Endpoints
POST /api/v1/training-data/collect
    Request: TrainingDataRequest
    Response: TrainingDataResponse

POST /api/v1/training-data/engineer-features
    Request: FeatureEngineeringRequest
    Response: FeatureEngineeringResponse

GET /api/v1/training-data/datasets/{dataset_type}
    Response: List<DatasetInfo>

POST /api/v1/training-data/label-outcome
    Request: OutcomeLabelingRequest
    Response: OutcomeLabelingResponse
```

### Event Output

#### TrainingDatasetEvent
```pseudo
Event training_dataset_generated {
    event_id: String
    timestamp: DateTime
    training_dataset: TrainingDatasetPayload
    data_quality_metrics: DataQualityPayload
}

struct TrainingDatasetPayload {
    dataset_id: String
    dataset_type: String
    sample_count: Integer
    feature_count: Integer
    label_distribution: Map<String, Integer>
    data_period: DateRange
    quality_score: Float
}

struct DataQualityPayload {
    completeness_score: Float
    accuracy_score: Float
    consistency_score: Float
    timeliness_score: Float
    validity_score: Float
    missing_data_percentage: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-24T11:00:00.000Z",
    training_dataset: {
        dataset_id: "dataset_20250624_001",
        dataset_type: "SIGNAL_GENERATION",
        sample_count: 10000,
        feature_count: 247,
        label_distribution: {
            "EXCELLENT": 1250,
            "GOOD": 2100,
            "NEUTRAL": 4300,
            "POOR": 1850,
            "BAD": 500
        },
        data_period: {
            start_date: "2025-06-01",
            end_date: "2025-06-24"
        },
        quality_score: 0.94
    },
    data_quality_metrics: {
        completeness_score: 0.98,
        accuracy_score: 0.96,
        consistency_score: 0.92,
        timeliness_score: 0.99,
        validity_score: 0.95,
        missing_data_percentage: 0.02
    }
}
```

## Data Model

### Core Entities
```pseudo
// Data Models
struct TrainingDataset {
    dataset_id: String
    dataset_type: DatasetType
    creation_timestamp: DateTime
    sample_count: Integer
    feature_schema: FeatureSchema
    label_schema: LabelSchema
    data_quality_report: DataQualityReport
    storage_location: String
}

struct FeatureSchema {
    feature_definitions: List<FeatureDefinition>
    normalization_parameters: Map<String, NormalizationParams>
    feature_selection_criteria: FeatureSelectionCriteria
}

struct LabelSchema {
    label_definitions: List<LabelDefinition>
    labeling_criteria: LabelingCriteria
    label_validation_rules: List<ValidationRule>
}

struct DataQualityReport {
    quality_metrics: Map<String, Float>
    data_lineage: DataLineage
    validation_results: List<ValidationResult>
    anomaly_detection_results: List<AnomalyResult>
}
```

## Database Schema (CQRS Pattern)

### Command Side (PostgreSQL)
```pseudo
// Training dataset metadata
Table training_datasets {
    id: UUID (primary key, auto-generated)
    dataset_id: String (required, unique, max_length: 100)
    dataset_type: String (required, max_length: 50)
    creation_timestamp: Timestamp (required)
    sample_count: Integer (required)
    feature_count: Integer (required)
    label_distribution: JSON (required)
    quality_score: Float (required)
    storage_location: String (required, max_length: 500)
    feature_schema: JSON (required)
    label_schema: JSON (required)
    created_at: Timestamp (default: now)
}

// Feature definitions
Table feature_definitions {
    id: UUID (primary key, auto-generated)
    feature_name: String (required, unique, max_length: 100)
    feature_type: String (required, max_length: 50)
    feature_category: String (required, max_length: 50)
    calculation_method: Text (required)
    normalization_method: String (max_length: 50)
    data_sources: JSON (required)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

// Outcome labeling rules
Table labeling_rules {
    id: UUID (primary key, auto-generated)
    rule_name: String (required, unique, max_length: 100)
    rule_type: String (required, max_length: 50)
    criteria: JSON (required)
    thresholds: JSON (required)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

// Data quality tracking
Table data_quality_reports {
    id: UUID (primary key, auto-generated)
    dataset_id: String (required, max_length: 100)
    quality_metrics: JSON (required)
    validation_results: JSON (required)
    anomaly_results: JSON
    data_lineage: JSON (required)
    report_timestamp: Timestamp (required)
    created_at: Timestamp (default: now)
}

// Indexes
idx_training_datasets_type_time: (dataset_type, creation_timestamp DESC)
idx_feature_definitions_category: (feature_category, enabled)
idx_labeling_rules_type: (rule_type, enabled)
idx_data_quality_dataset_time: (dataset_id, report_timestamp DESC)
```

### Query Side (TimescaleDB)
```pseudo
// Training data collection metrics
Table training_data_metrics_ts {
    timestamp: Timestamp (required, partition_key)
    dataset_type: String (required, max_length: 50)
    
    // Collection metrics
    samples_collected: Integer
    features_engineered: Integer
    labels_generated: Integer
    processing_time_ms: Float
    
    // Quality metrics
    data_completeness: Float
    data_accuracy: Float
    label_quality: Float
    feature_quality: Float
    
    // Performance metrics
    collection_rate_per_second: Float
    feature_engineering_rate: Float
    labeling_accuracy: Float
    
    created_at: Timestamp (default: now)
    
    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
}

// Feature importance tracking
Table feature_importance_ts {
    timestamp: Timestamp (required, partition_key)
    feature_name: String (required, max_length: 100)
    dataset_type: String (required, max_length: 50)
    importance_score: Float (required)
    correlation_with_outcome: Float
    information_gain: Float
    mutual_information: Float
    
    created_at: Timestamp (default: now)
    
    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 week)
}

// Indexes for fast queries
idx_training_metrics_type_time: (dataset_type, timestamp DESC)
idx_feature_importance_name_time: (feature_name, timestamp DESC)
```

### Data Lake Storage (Apache Parquet)
```pseudo
// Raw training datasets stored in Parquet format
Structure training_data_lake {
    // Partitioned by dataset_type and date
    /data/training_datasets/{dataset_type}/year={YYYY}/month={MM}/day={DD}/
    
    // Files contain:
    - decision_context.parquet
    - market_context.parquet  
    - execution_context.parquet
    - feature_vectors.parquet
    - outcome_labels.parquet
    - metadata.json
}

// Feature store for reusable features
Structure feature_store {
    /data/features/{feature_category}/year={YYYY}/month={MM}/day={DD}/
    
    // Standardized feature format
    - market_microstructure_features.parquet
    - sentiment_features.parquet
    - technical_features.parquet
    - risk_features.parquet
}
```

### Redis Caching Strategy
```pseudo
Cache ml_training_cache {
    // Active datasets
    "datasets:active": List<TrainingDataset> (TTL: 1h)
    
    // Feature schemas
    "features:schema:{dataset_type}": FeatureSchema (TTL: 4h)
    
    // Labeling rules
    "labeling_rules:active": List<LabelingRule> (TTL: 2h)
    
    // Recent quality metrics
    "quality:latest": DataQualityMetrics (TTL: 30m)
    
    // Feature importance
    "feature_importance:{dataset_type}": Map<String, Float> (TTL: 1h)
    
    // Processing status
    "processing:status": ProcessingStatus (TTL: 5m)
}
```

## Implementation Estimation

### Priority: **CRITICAL** (Essential for ML model improvement)
### Estimated Time: **6-8 weeks**

#### Week 1-2: Core Data Collection Engine
- Python service setup with Polars, DuckDB, and Apache Arrow
- Basic tick data collection and preprocessing
- Decision context extraction and storage
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3-4: Feature Engineering Framework
- Advanced feature engineering using Polars
- Market microstructure feature extraction
- Sentiment and technical feature integration
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 5-6: Outcome Labeling System
- Automated outcome labeling based on performance
- Multi-criteria labeling framework
- Label quality validation and verification
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 7-8: Data Quality & Integration
- Comprehensive data quality monitoring
- Integration with all trading workflows
- Dataset generation and export capabilities
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **16 dev-weeks**
### Team Size: **2 developers** (1 senior ML engineer, 1 data engineer)
### Dependencies:
- All trading workflows for decision and outcome data
- Market Data Acquisition for tick data
- High-performance storage infrastructure

### Risk Factors:
- **Medium**: Data volume and processing performance
- **Medium**: Data quality and consistency across sources
- **Low**: Technology stack maturity

### Success Criteria:
- Collect 100% of trading decisions with full context
- Generate high-quality labeled datasets with >95% accuracy
- Process tick data with <5s latency
- Support multiple dataset types for different ML models
- Maintain data quality score >90%
