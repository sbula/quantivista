# ML Prediction Service

## Purpose
The ML Prediction Service is responsible for generating price movement predictions using ensemble machine learning models with uncertainty quantification. It transforms various input features into probabilistic forecasts that can be used by other services to make informed trading decisions and risk assessments.

## Strict Boundaries

### Responsibilities
- Multi-model ensemble predictions for financial instruments
- Confidence interval calculation for predictions
- Model performance monitoring and evaluation
- Feature importance analysis and interpretation
- Dynamic model selection based on market conditions
- Distribution of prediction results with uncertainty metrics

### Non-Responsibilities
- Does NOT make trading decisions
- Does NOT execute trades
- Does NOT collect or normalize market data
- Does NOT calculate technical indicators
- Does NOT perform risk calculations
- Does NOT manage user preferences or authentication

## API Design (API-First)

### REST API

#### GET /api/v1/predictions/{instrument_id}
Retrieves price predictions for a specific instrument.

**Path Parameters:**
- `instrument_id` (string, required): Instrument identifier

**Query Parameters:**
- `timeframes` (string, required): Comma-separated list of prediction timeframes (1h, 4h, 1d, 1w, 1mo)
- `include_features` (boolean, optional): Include input features in response (default: false)
- `include_history` (boolean, optional): Include historical predictions (default: false)
- `history_limit` (integer, optional): Maximum number of historical predictions (default: 10)

**Response:**
```json
{
  "instrument_id": "AAPL",
  "generated_at": "2025-06-20T14:30:00Z",
  "predictions": [
    {
      "timeframe": "1d",
      "direction": "positive",
      "confidence": 0.78,
      "price_target": 155.25,
      "confidence_interval": {
        "lower_95": 152.80,
        "upper_95": 157.70
      },
      "probability_distribution": {
        "positive": 0.78,
        "neutral": 0.15,
        "negative": 0.07
      },
      "models_used": ["gradient_boost", "neural_net", "random_forest"],
      "feature_importance": {
        "rsi_14": 0.25,
        "news_sentiment": 0.20,
        "volume_change": 0.15
      }
    },
    {
      "timeframe": "1w",
      "direction": "neutral",
      "confidence": 0.65,
      "price_target": 153.50,
      "confidence_interval": {
        "lower_95": 148.20,
        "upper_95": 158.80
      },
      "probability_distribution": {
        "positive": 0.45,
        "neutral": 0.40,
        "negative": 0.15
      },
      "models_used": ["gradient_boost", "neural_net", "random_forest"],
      "feature_importance": {
        "sector_momentum": 0.30,
        "macd_signal": 0.25,
        "earnings_surprise": 0.15
      }
    }
  ],
  "model_performance": {
    "accuracy_1d": 0.72,
    "accuracy_1w": 0.68,
    "sharpe_ratio": 1.85,
    "calibration_score": 0.92
  }
}
```

#### GET /api/v1/predictions/batch
Retrieves predictions for multiple instruments.

**Query Parameters:**
- `instruments` (string, required): Comma-separated list of instrument IDs
- `timeframe` (string, required): Prediction timeframe (1h, 4h, 1d, 1w, 1mo)
- `min_confidence` (float, optional): Minimum prediction confidence (0.0-1.0, default: 0.6)

**Response:**
```json
{
  "generated_at": "2025-06-20T14:30:00Z",
  "timeframe": "1d",
  "predictions": [
    {
      "instrument_id": "AAPL",
      "direction": "positive",
      "confidence": 0.78,
      "price_target": 155.25
    },
    {
      "instrument_id": "MSFT",
      "direction": "positive",
      "confidence": 0.82,
      "price_target": 320.50
    },
    {
      "instrument_id": "GOOGL",
      "direction": "neutral",
      "confidence": 0.65,
      "price_target": 2750.00
    }
  ]
}
```

#### GET /api/v1/models/performance
Retrieves performance metrics for prediction models.

**Query Parameters:**
- `timeframe` (string, optional): Filter by prediction timeframe (1h, 4h, 1d, 1w, 1mo)
- `model_type` (string, optional): Filter by model type (gradient_boost, neural_net, random_forest, ensemble)
- `from` (string, optional): Start timestamp (ISO 8601)
- `to` (string, optional): End timestamp (ISO 8601)

**Response:**
```json
{
  "performance_metrics": [
    {
      "model_type": "ensemble",
      "timeframe": "1d",
      "accuracy": 0.72,
      "precision": 0.75,
      "recall": 0.70,
      "f1_score": 0.72,
      "sharpe_ratio": 1.85,
      "calibration_score": 0.92,
      "log_loss": 0.45
    },
    {
      "model_type": "gradient_boost",
      "timeframe": "1d",
      "accuracy": 0.70,
      "precision": 0.73,
      "recall": 0.68,
      "f1_score": 0.70,
      "sharpe_ratio": 1.75,
      "calibration_score": 0.90,
      "log_loss": 0.48
    }
  ]
}
```

### gRPC API

```protobuf
syntax = "proto3";

package ml_prediction.v1;

import "google/protobuf/timestamp.proto";

service MLPredictionService {
  // Get predictions for a specific instrument
  rpc GetPredictions(PredictionRequest) returns (PredictionResponse);
  
  // Get batch predictions for multiple instruments
  rpc GetBatchPredictions(BatchPredictionRequest) returns (BatchPredictionResponse);
  
  // Stream real-time prediction updates
  rpc StreamPredictions(StreamPredictionRequest) returns (stream PredictionUpdate);
  
  // Get model performance metrics
  rpc GetModelPerformance(ModelPerformanceRequest) returns (ModelPerformanceResponse);
}

message PredictionRequest {
  string instrument_id = 1;
  repeated string timeframes = 2; // "1h", "4h", "1d", "1w", "1mo"
  bool include_features = 3;
  bool include_history = 4;
  int32 history_limit = 5;
}

message PredictionResponse {
  string instrument_id = 1;
  google.protobuf.Timestamp generated_at = 2;
  repeated Prediction predictions = 3;
  ModelPerformanceMetrics model_performance = 4;
  repeated HistoricalPrediction history = 5;
}

message Prediction {
  string timeframe = 1;
  string direction = 2; // "positive", "neutral", "negative"
  double confidence = 3;
  double price_target = 4;
  ConfidenceInterval confidence_interval = 5;
  ProbabilityDistribution probability_distribution = 6;
  repeated string models_used = 7;
  map<string, double> feature_importance = 8;
  map<string, double> input_features = 9;
}

message ConfidenceInterval {
  double lower_95 = 1;
  double upper_95 = 2;
}

message ProbabilityDistribution {
  double positive = 1;
  double neutral = 2;
  double negative = 3;
}

message HistoricalPrediction {
  google.protobuf.Timestamp timestamp = 1;
  string timeframe = 2;
  string direction = 3;
  double confidence = 4;
  double price_target = 5;
  double actual_price = 6;
  bool was_correct = 7;
}

message BatchPredictionRequest {
  repeated string instrument_ids = 1;
  string timeframe = 2;
  double min_confidence = 3;
}

message BatchPredictionResponse {
  google.protobuf.Timestamp generated_at = 1;
  string timeframe = 2;
  repeated BatchPredictionItem predictions = 3;
}

message BatchPredictionItem {
  string instrument_id = 1;
  string direction = 2;
  double confidence = 3;
  double price_target = 4;
}

message StreamPredictionRequest {
  repeated string instrument_ids = 1;
  repeated string timeframes = 2;
  double min_confidence = 3;
}

message PredictionUpdate {
  string instrument_id = 1;
  string timeframe = 2;
  google.protobuf.Timestamp generated_at = 3;
  string direction = 4;
  double confidence = 5;
  double price_target = 6;
  ConfidenceInterval confidence_interval = 7;
}

message ModelPerformanceRequest {
  string timeframe = 1;
  string model_type = 2;
  google.protobuf.Timestamp from = 3;
  google.protobuf.Timestamp to = 4;
}

message ModelPerformanceResponse {
  repeated ModelPerformanceMetrics performance_metrics = 1;
}

message ModelPerformanceMetrics {
  string model_type = 1;
  string timeframe = 2;
  double accuracy = 3;
  double precision = 4;
  double recall = 5;
  double f1_score = 6;
  double sharpe_ratio = 7;
  double calibration_score = 8;
  double log_loss = 9;
}
```

## Data Model

### Core Entities

#### PredictionModel
Represents a machine learning model used for predictions.

**Attributes:**
- `id` (string): Unique identifier for the model
- `type` (enum): Type of model (gradient_boost, neural_net, random_forest, ensemble)
- `version` (string): Model version
- `timeframe` (enum): Prediction timeframe (1h, 4h, 1d, 1w, 1mo)
- `features` (array): Input features used by the model
- `hyperparameters` (map): Model hyperparameters
- `training_date` (datetime): When the model was trained
- `performance_metrics` (map): Model performance metrics
- `status` (enum): Model status (active, inactive, deprecated)

#### Prediction
Represents a price prediction for a specific instrument and timeframe.

**Attributes:**
- `instrument_id` (string): Reference to the instrument
- `timeframe` (enum): Prediction timeframe (1h, 4h, 1d, 1w, 1mo)
- `model_id` (string): Reference to the model that generated the prediction
- `timestamp` (datetime): When the prediction was generated
- `direction` (enum): Predicted price direction (positive, neutral, negative)
- `confidence` (float): Confidence level of the prediction (0.0-1.0)
- `price_target` (decimal): Predicted price target
- `confidence_interval_lower` (decimal): Lower bound of the confidence interval
- `confidence_interval_upper` (decimal): Upper bound of the confidence interval
- `probability_distribution` (map): Probability distribution of outcomes
- `feature_importance` (map): Importance of input features
- `input_features` (map): Input features used for the prediction

#### PredictionEvaluation
Represents an evaluation of a prediction against actual outcomes.

**Attributes:**
- `prediction_id` (string): Reference to the prediction
- `actual_price` (decimal): Actual price at the end of the prediction timeframe
- `actual_direction` (enum): Actual price direction
- `was_correct` (boolean): Whether the prediction was correct
- `evaluation_timestamp` (datetime): When the evaluation was performed
- `profit_loss` (decimal): Hypothetical profit/loss if traded on this prediction
- `notes` (string): Additional evaluation notes

#### ModelPerformance
Represents performance metrics for a model over a specific period.

**Attributes:**
- `model_id` (string): Reference to the model
- `start_date` (datetime): Start of the evaluation period
- `end_date` (datetime): End of the evaluation period
- `accuracy` (float): Prediction accuracy
- `precision` (float): Precision metric
- `recall` (float): Recall metric
- `f1_score` (float): F1 score
- `sharpe_ratio` (float): Sharpe ratio of predictions
- `calibration_score` (float): Calibration score
- `log_loss` (float): Logarithmic loss
- `confusion_matrix` (map): Confusion matrix data

## DB Schema (CQRS Pattern)

### Write Schema (Command Side)

#### prediction_models
```sql
CREATE TABLE prediction_models (
    id VARCHAR(50) PRIMARY KEY,
    type VARCHAR(20) NOT NULL,
    version VARCHAR(20) NOT NULL,
    timeframe VARCHAR(10) NOT NULL,
    features JSONB NOT NULL,
    hyperparameters JSONB NOT NULL,
    training_date TIMESTAMP WITH TIME ZONE NOT NULL,
    performance_metrics JSONB,
    status VARCHAR(20) NOT NULL DEFAULT 'active',
    model_path VARCHAR(255) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX prediction_models_type_timeframe_idx ON prediction_models(type, timeframe);
CREATE INDEX prediction_models_status_idx ON prediction_models(status);
```

#### model_training_runs
```sql
CREATE TABLE model_training_runs (
    id SERIAL PRIMARY KEY,
    model_id VARCHAR(50) NOT NULL REFERENCES prediction_models(id),
    training_dataset VARCHAR(100) NOT NULL,
    validation_dataset VARCHAR(100) NOT NULL,
    training_parameters JSONB NOT NULL,
    training_metrics JSONB,
    validation_metrics JSONB,
    start_time TIMESTAMP WITH TIME ZONE NOT NULL,
    end_time TIMESTAMP WITH TIME ZONE,
    status VARCHAR(20) NOT NULL DEFAULT 'running',
    error_message TEXT,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX model_training_runs_model_id_idx ON model_training_runs(model_id);
CREATE INDEX model_training_runs_status_idx ON model_training_runs(status);
```

#### predictions
```sql
CREATE TABLE predictions (
    id SERIAL PRIMARY KEY,
    instrument_id VARCHAR(20) NOT NULL,
    timeframe VARCHAR(10) NOT NULL,
    model_id VARCHAR(50) NOT NULL REFERENCES prediction_models(id),
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    direction VARCHAR(10) NOT NULL,
    confidence DECIMAL(4,3) NOT NULL,
    price_target DECIMAL(18, 8) NOT NULL,
    confidence_interval_lower DECIMAL(18, 8) NOT NULL,
    confidence_interval_upper DECIMAL(18, 8) NOT NULL,
    probability_distribution JSONB NOT NULL,
    feature_importance JSONB,
    input_features JSONB,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT predictions_instrument_timeframe_timestamp_idx UNIQUE (instrument_id, timeframe, timestamp)
);

CREATE INDEX predictions_instrument_id_idx ON predictions(instrument_id);
CREATE INDEX predictions_timestamp_idx ON predictions(timestamp);
CREATE INDEX predictions_model_id_idx ON predictions(model_id);
```

#### prediction_evaluations
```sql
CREATE TABLE prediction_evaluations (
    id SERIAL PRIMARY KEY,
    prediction_id INTEGER NOT NULL REFERENCES predictions(id),
    actual_price DECIMAL(18, 8) NOT NULL,
    actual_direction VARCHAR(10) NOT NULL,
    was_correct BOOLEAN NOT NULL,
    evaluation_timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    profit_loss DECIMAL(18, 8),
    notes TEXT,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT prediction_evaluations_prediction_id_idx UNIQUE (prediction_id)
);

CREATE INDEX prediction_evaluations_was_correct_idx ON prediction_evaluations(was_correct);
```

#### model_performance_snapshots
```sql
CREATE TABLE model_performance_snapshots (
    id SERIAL PRIMARY KEY,
    model_id VARCHAR(50) NOT NULL REFERENCES prediction_models(id),
    start_date TIMESTAMP WITH TIME ZONE NOT NULL,
    end_date TIMESTAMP WITH TIME ZONE NOT NULL,
    accuracy DECIMAL(4,3) NOT NULL,
    precision DECIMAL(4,3) NOT NULL,
    recall DECIMAL(4,3) NOT NULL,
    f1_score DECIMAL(4,3) NOT NULL,
    sharpe_ratio DECIMAL(5,3),
    calibration_score DECIMAL(4,3),
    log_loss DECIMAL(5,3),
    confusion_matrix JSONB,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT model_performance_snapshots_model_period_idx UNIQUE (model_id, start_date, end_date)
);

CREATE INDEX model_performance_snapshots_model_id_idx ON model_performance_snapshots(model_id);
CREATE INDEX model_performance_snapshots_date_range_idx ON model_performance_snapshots(start_date, end_date);
```

### Read Schema (Query Side)

#### latest_predictions
```sql
CREATE TABLE latest_predictions (
    instrument_id VARCHAR(20) NOT NULL,
    timeframe VARCHAR(10) NOT NULL,
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    direction VARCHAR(10) NOT NULL,
    confidence DECIMAL(4,3) NOT NULL,
    price_target DECIMAL(18, 8) NOT NULL,
    confidence_interval_lower DECIMAL(18, 8) NOT NULL,
    confidence_interval_upper DECIMAL(18, 8) NOT NULL,
    probability_distribution JSONB NOT NULL,
    feature_importance JSONB,
    model_id VARCHAR(50) NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    PRIMARY KEY (instrument_id, timeframe)
);
```

#### prediction_history
```sql
CREATE TABLE prediction_history (
    id SERIAL PRIMARY KEY,
    instrument_id VARCHAR(20) NOT NULL,
    timeframe VARCHAR(10) NOT NULL,
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL,
    direction VARCHAR(10) NOT NULL,
    confidence DECIMAL(4,3) NOT NULL,
    price_target DECIMAL(18, 8) NOT NULL,
    actual_price DECIMAL(18, 8),
    was_correct BOOLEAN,
    profit_loss DECIMAL(18, 8),
    CONSTRAINT prediction_history_instrument_timeframe_timestamp_idx UNIQUE (instrument_id, timeframe, timestamp)
);

CREATE INDEX prediction_history_instrument_timeframe_idx ON prediction_history(instrument_id, timeframe);
CREATE INDEX prediction_history_timestamp_idx ON prediction_history(timestamp DESC);
```

#### active_models
```sql
CREATE TABLE active_models (
    model_id VARCHAR(50) PRIMARY KEY,
    type VARCHAR(20) NOT NULL,
    version VARCHAR(20) NOT NULL,
    timeframe VARCHAR(10) NOT NULL,
    accuracy DECIMAL(4,3) NOT NULL,
    sharpe_ratio DECIMAL(5,3),
    features JSONB NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    CONSTRAINT active_models_type_timeframe_idx UNIQUE (type, timeframe)
);
```

#### model_performance_metrics
```sql
CREATE TABLE model_performance_metrics (
    model_type VARCHAR(20) NOT NULL,
    timeframe VARCHAR(10) NOT NULL,
    accuracy DECIMAL(4,3) NOT NULL,
    precision DECIMAL(4,3) NOT NULL,
    recall DECIMAL(4,3) NOT NULL,
    f1_score DECIMAL(4,3) NOT NULL,
    sharpe_ratio DECIMAL(5,3),
    calibration_score DECIMAL(4,3),
    log_loss DECIMAL(5,3),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    PRIMARY KEY (model_type, timeframe)
);
```

## Place in Workflow

The ML Prediction Service is a core component of the Prediction and Decision Workflow and serves as the primary generator of price predictions. It:

1. **Receives features** from various upstream services (Market Data Service, Technical Analysis Service, News Intelligence Service)
2. **Selects appropriate models** based on market conditions and instrument characteristics
3. **Generates price predictions** with confidence intervals for multiple timeframes
4. **Evaluates prediction accuracy** and updates model performance metrics
5. **Distributes prediction results** to downstream services

The service interacts with:
- **Market Data Service** (input): Provides market data features
- **Technical Analysis Service** (input): Supplies technical indicators
- **News Intelligence Service** (input): Provides sentiment and impact assessments
- **Instrument Clustering Service** (input): Supplies clustering information
- **Trading Strategy Service** (output): Uses predictions for strategy decisions
- **Risk Analysis Service** (output): Incorporates prediction uncertainty into risk calculations
- **Portfolio Optimization Service** (output): Uses predictions for portfolio adjustments
- **Reporting Service** (output): Includes prediction data in reports and dashboards

## Project Plan

### Phase 1: Foundation (Weeks 1-4)

#### Week 1: Setup and Infrastructure
- Set up development environment and CI/CD pipeline
- Configure Kubernetes infrastructure for the service
- Set up PostgreSQL for data storage
- Implement basic service skeleton with health checks

#### Week 2: Feature Engineering and Data Pipeline
- Develop feature collection and aggregation pipeline
- Implement feature preprocessing and normalization
- Create feature storage and versioning system
- Build data validation and quality checks

#### Week 3: Core ML Models
- Implement gradient boosting models (XGBoost, LightGBM)
- Develop neural network models (JAX/Flax)
- Create random forest models
- Build model evaluation framework

#### Week 4: Basic API Implementation
- Implement REST API endpoints for predictions
- Create gRPC service for real-time updates
- Develop basic authentication and rate limiting
- Write comprehensive API documentation

### Phase 2: Enhancement (Weeks 5-8)

#### Week 5: Ensemble Modeling
- Implement model ensemble techniques
- Develop dynamic model selection
- Create model weighting strategies
- Build model performance tracking

#### Week 6: Uncertainty Quantification
- Implement confidence interval calculation
- Develop probability distribution estimation
- Create uncertainty visualization
- Build calibration assessment tools

#### Week 7: Model Explainability
- Implement feature importance analysis
- Develop SHAP value calculation
- Create counterfactual explanations
- Build explanation visualization tools

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
- Optimize model inference speed
- Implement model caching strategies
- Enhance database query performance
- Fine-tune resource utilization

#### Week 11: Advanced ML Features
- Implement online learning capabilities
- Develop transfer learning techniques
- Create automated hyperparameter optimization
- Build advanced ensemble techniques

#### Week 12: Final Preparation and Deployment
- Conduct final integration testing
- Prepare production deployment plan
- Execute staged rollout
- Monitor production performance

### Ongoing Maintenance
- Regular model retraining and improvement
- Performance monitoring and optimization
- Feature engineering enhancements
- Algorithm updates and improvements