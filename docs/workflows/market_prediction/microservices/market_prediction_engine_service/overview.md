# Market Prediction Engine Service

## Responsibility
Transform synthesized trading signals into market predictions using ensemble machine learning models. Generates directional forecasts, confidence scores, and price targets across multiple timeframes with real-time inference capabilities.

## Technology Stack
- **Language**: Python + FastAPI + Scikit-learn + XGBoost + LightGBM + MLflow
- **Libraries**: XGBoost, LightGBM, TensorFlow, PyTorch, MLflow, Redis client
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **ML Framework**: JAX for custom optimization algorithms and advanced models
- **Scaling**: Horizontal by prediction volume, model type partitioning
- **NFRs**: P99 prediction latency < 500ms, 5K+ predictions/sec throughput, 70% directional accuracy

## API Specification

### Core APIs
```pseudo
// Enumerations
enum PredictionDirection {
    STRONG_BUY,
    BUY,
    NEUTRAL,
    SELL,
    STRONG_SELL
}

enum ModelType {
    XGBOOST,
    LIGHTGBM,
    RANDOM_FOREST,
    NEURAL_NETWORK,
    LSTM,
    ENSEMBLE
}

enum PredictionTimeframe {
    HOUR_1,
    HOUR_4,
    DAY_1,
    WEEK_1,
    MONTH_1
}

// Data Models
struct PredictionRequest {
    instrument_id: String
    timeframes: List<PredictionTimeframe>
    signal_vector: SignalVector
    model_preference: Optional<ModelType>
}

struct MarketPrediction {
    prediction_id: String
    instrument_id: String
    timeframe: PredictionTimeframe
    direction: PredictionDirection
    confidence: Float
    price_target: Float
    probability_distribution: Map<String, Float>
    confidence_interval: ConfidenceInterval
    model_metadata: ModelMetadata
    timestamp: DateTime
}

struct ConfidenceInterval {
    lower_95: Float
    upper_95: Float
    lower_80: Float
    upper_80: Float
}

struct ModelMetadata {
    model_id: String
    model_type: ModelType
    version: String
    ensemble_weight: Float
    feature_importance: Map<String, Float>
}

// REST API Endpoints
POST /api/v1/predictions/generate
    Request: PredictionRequest
    Response: List<MarketPrediction>

POST /api/v1/predictions/batch
    Request: List<PredictionRequest>
    Response: List<MarketPrediction>

GET /api/v1/predictions/{instrument_id}/{timeframe}
    Response: MarketPrediction

GET /api/v1/models/active
    Response: List<ModelMetadata>

POST /api/v1/models/deploy/{model_id}
    Request: DeploymentConfig
    Response: DeploymentResult

GET /api/v1/ensemble/performance
    Response: EnsemblePerformanceMetrics
```

### Event Output
```pseudo
Event market_prediction_generated {
    event_id: String
    timestamp: DateTime
    prediction_generated: PredictionGeneratedData
}

struct PredictionGeneratedData {
    prediction_batch_id: String
    instrument_id: String
    timeframes: List<String>
    predictions_count: Integer
    generation_duration_ms: Integer
    average_confidence: Float
    model_ensemble_used: List<String>
    directional_distribution: DirectionalDistribution
}

struct DirectionalDistribution {
    strong_buy_count: Integer
    buy_count: Integer
    neutral_count: Integer
    sell_count: Integer
    strong_sell_count: Integer
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:20:00.000Z",
    prediction_generated: {
        prediction_batch_id: "prediction_20250621_001",
        instrument_id: "AAPL",
        timeframes: ["1h", "4h", "1d"],
        predictions_count: 3,
        generation_duration_ms: 450,
        average_confidence: 0.78,
        model_ensemble_used: ["XGBOOST", "LIGHTGBM", "NEURAL_NETWORK"],
        directional_distribution: {
            strong_buy_count: 1,
            buy_count: 2,
            neutral_count: 0,
            sell_count: 0,
            strong_sell_count: 0
        }
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table prediction_jobs {
    id: UUID (primary key, auto-generated)
    job_id: String (required, unique, max_length: 100)
    instrument_id: String (required, max_length: 50)
    timeframes: JSON (required)
    status: String (required, max_length: 20)
    predictions_generated: Integer (default: 0)
    generation_duration_ms: Float
    average_confidence: Float
    created_at: Timestamp (default: now)
}

Table model_deployments {
    id: UUID (primary key, auto-generated)
    model_id: String (required, unique, max_length: 100)
    model_type: String (required, max_length: 50)
    version: String (required, max_length: 20)
    status: String (required, max_length: 20)
    ensemble_weight: Float
    performance_metrics: JSON
    deployed_at: Timestamp (default: now)
}

Table prediction_performance {
    id: UUID (primary key, auto-generated)
    prediction_id: String (required, max_length: 100)
    instrument_id: String (required, max_length: 50)
    timeframe: String (required, max_length: 10)
    predicted_direction: String (required, max_length: 20)
    actual_direction: String (max_length: 20)
    confidence: Float (required)
    accuracy: Boolean
    created_at: Timestamp (default: now)
}
```

### Redis Caching
```pseudo
Cache prediction_cache {
    // Generated predictions
    "predictions:{instrument_id}:{timeframe}": MarketPrediction (TTL: 10m)
    
    // Model performance
    "model_performance:{model_id}": PerformanceMetrics (TTL: 1h)
    
    // Ensemble weights
    "ensemble_weights": Map<String, Float> (TTL: 6h)
    
    // Prediction status
    "prediction_status:{job_id}": PredictionStatus (TTL: 1h)
}
```

## Implementation Estimation

### Priority: **CRITICAL** (Core ML prediction engine)
### Estimated Time: **4-5 weeks**

#### Week 1-2: Core Prediction Engine
- Python service setup with ML frameworks
- Basic ensemble model implementation
- Model loading and inference pipeline
- **Effort**: 3 developers × 2 weeks = 6 dev-weeks

#### Week 3: Advanced ML Features
- Multi-timeframe prediction coordination
- Ensemble optimization and weighting
- Performance monitoring and feedback
- **Effort**: 3 developers × 1 week = 3 dev-weeks

#### Week 4-5: Integration & Optimization
- Integration with signal synthesis service
- Performance optimization and caching
- Model deployment and versioning
- **Effort**: 3 developers × 2 weeks = 6 dev-weeks

### Total Effort: **15 dev-weeks**
### Team Size: **3 developers** (2 senior ML engineers, 1 Python developer)
### Dependencies: Trading Indicator Synthesis Service, MLflow infrastructure

### Success Criteria:
- P99 prediction latency < 500ms
- 5K+ predictions per second throughput
- 70% directional accuracy over 30-day periods
- Support for ensemble models
- Multi-timeframe prediction capability
- Real-time model performance tracking
