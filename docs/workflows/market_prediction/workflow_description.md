# Market Prediction Workflow

## Overview
The Market Prediction Workflow is responsible for generating instrument evaluations and price predictions using machine learning models, technical analysis, and market intelligence. This workflow focuses purely on prediction and evaluation without making trading decisions, providing standardized instrument ratings and confidence metrics for downstream decision-making workflows.

## Key Challenges Addressed
- **Quality-Aware Feature Integration**: Consuming features from upstream workflows with quality-based weighting
- **Multi-Timeframe Evaluation**: Consistent instrument rating across different time horizons
- **Model Lifecycle Management**: Automated retraining, A/B testing, and performance monitoring
- **Prediction Accuracy**: Ensemble methods and uncertainty quantification
- **Real-time Prediction Updates**: Streaming predictions as new market data and intelligence arrive

## Core Responsibilities
- **Feature Engineering**: Quality-weighted feature preparation from all upstream workflows
- **ML Model Management**: Model versioning, training, deployment, and lifecycle management
- **Instrument Evaluation**: Independent rating of instruments across multiple timeframes
- **Prediction Generation**: Price predictions with confidence intervals and uncertainty quantification
- **Performance Monitoring**: Model accuracy tracking, drift detection, and retraining triggers

## NOT This Workflow's Responsibilities
- **Trading Decisions**: Making buy/sell/hold decisions (belongs to Trading Decision Workflow)
- **Position Sizing**: Calculating position sizes (belongs to Trading Decision Workflow)
- **Portfolio Analysis**: Portfolio-level risk and optimization (belongs to Portfolio Management Workflow)
- **Order Execution**: Trade execution and order management (belongs to Trade Execution Workflow)
- **Risk Policy Enforcement**: Applying trading rules and constraints (belongs to Trading Decision Workflow)

## Workflow Sequence

### 1. Quality-Aware Feature Engineering
**Responsibility**: ML Feature Engineering Service

#### Multi-Source Feature Integration
- **Market Data Features**: Price, volume, volatility from Market Data workflow
- **Technical Features**: Indicators, patterns, correlations from Instrument Analysis workflow
- **Intelligence Features**: Sentiment, impact assessments from Market Intelligence workflow
- **Quality Weighting**: Apply quality scores from upstream workflows to feature importance
- **Feature Validation**: Cross-validate features across multiple sources

#### Feature Categories by Quality Tier
```python
class FeatureQualityTiers:
    TIER_1_PREMIUM = {
        'sources': ['verified_market_data', 'high_confidence_technical_indicators'],
        'weight_multiplier': 1.0,
        'use_for': 'real_time_predictions'
    }
    
    TIER_2_STANDARD = {
        'sources': ['standard_market_data', 'medium_confidence_indicators'],
        'weight_multiplier': 0.8,
        'use_for': 'medium_term_predictions'
    }
    
    TIER_3_RESEARCH = {
        'sources': ['social_media_sentiment', 'low_confidence_signals'],
        'weight_multiplier': 0.5,
        'use_for': 'long_term_trend_analysis'
    }
```

### 2. ML Model Management and Training
**Responsibility**: ML Model Management Service

#### Model Lifecycle Management
- **Model Versioning**: Semantic versioning with A/B testing capabilities
- **Automated Retraining**: Trigger retraining based on performance degradation
- **Ensemble Management**: Dynamic model weighting based on recent performance
- **Drift Detection**: Monitor feature and prediction drift over time
- **Gradual Rollouts**: Canary deployments for new model versions

#### Model Types and Specialization
- **Short-term Models (1h-4h)**: High-frequency technical and microstructure features
- **Medium-term Models (1d-1w)**: Technical indicators and sentiment analysis
- **Long-term Models (1w-1mo)**: Fundamental analysis and macroeconomic factors
- **Ensemble Models**: Meta-learning across multiple base models
- **Specialized Models**: Sector-specific, volatility-regime specific models

### 3. Prediction Generation Engine
**Responsibility**: ML Prediction Engine Service

#### Multi-Timeframe Prediction Generation
- **Ensemble Aggregation**: Combine predictions across models and timeframes
- **Uncertainty Quantification**: Bayesian confidence intervals and prediction intervals
- **Calibration**: Ensure prediction probabilities match actual outcomes
- **Real-time Updates**: Streaming predictions as new data arrives
- **Batch Predictions**: Efficient batch processing for multiple instruments

### 4. Independent Instrument Evaluation
**Responsibility**: Instrument Evaluation Service

#### Rating Scale (Independent of Portfolio)
- **Strong Buy (5)**: High confidence positive prediction with strong technical confirmation
- **Buy (4)**: Moderate confidence positive prediction with technical support
- **Neutral (3)**: Low confidence or conflicting signals across timeframes
- **Sell (2)**: Moderate confidence negative prediction with technical confirmation
- **Strong Sell (1)**: High confidence negative prediction with strong technical confirmation

#### Multi-Timeframe Rating Engine
```python
class InstrumentEvaluator:
    def __init__(self):
        self.timeframes = ['1h', '4h', '1d', '1w', '1mo']
        self.rating_thresholds = {
            'strong_buy': 0.8,
            'buy': 0.6,
            'neutral': 0.4,
            'sell': 0.2,
            'strong_sell': 0.0
        }
    
    async def evaluate_instrument(self, instrument_id: str) -> InstrumentEvaluation:
        """Generate independent evaluation for an instrument"""
        
        # Collect features from all upstream workflows
        features = await self.collect_quality_weighted_features(instrument_id)
        
        # Generate predictions for all timeframes
        predictions = {}
        for timeframe in self.timeframes:
            prediction = await self.ml_prediction_service.predict(
                instrument_id, timeframe, features
            )
            predictions[timeframe] = prediction
        
        # Apply technical confirmation
        technical_signals = await self.get_technical_confirmation(instrument_id)
        
        # Integrate sentiment analysis
        sentiment_score = await self.get_sentiment_score(instrument_id)
        
        # Calculate composite rating for each timeframe
        ratings = {}
        for timeframe, prediction in predictions.items():
            composite_score = self.calculate_composite_score(
                prediction, technical_signals, sentiment_score, timeframe
            )
            ratings[timeframe] = self.score_to_rating(composite_score)
        
        return InstrumentEvaluation(
            instrument_id=instrument_id,
            timestamp=datetime.utcnow(),
            ratings=ratings,
            predictions=predictions,
            technical_signals=technical_signals,
            sentiment_score=sentiment_score,
            confidence_metrics=self.calculate_confidence_metrics(predictions)
        )
```

### 5. Prediction Quality Assessment
**Responsibility**: Prediction Quality Service

#### Quality Metrics and Validation
- **Prediction Accuracy**: Track accuracy across different timeframes and market conditions
- **Calibration Assessment**: Ensure predicted probabilities match actual outcomes
- **Feature Quality Validation**: Cross-validate features across multiple sources
- **Model Agreement**: Measure consensus across different models
- **Confidence Calibration**: Validate that confidence scores are well-calibrated

### 6. Event-Driven Prediction Distribution
**Responsibility**: Prediction Distribution Service
- **Real-time streaming**: Apache Pulsar for immediate prediction updates
- **Prediction persistence**: Store predictions with full metadata and reasoning
- **Performance tracking**: Monitor prediction outcomes and model performance
- **API gateway**: RESTful and gRPC APIs for prediction consumption
- **Quality routing**: Route high-quality predictions to real-time systems

## Event Contracts

### Events Produced

#### `InstrumentEvaluatedEvent`
```json
{
  "eventId": "uuid",
  "timestamp": "2025-06-21T10:30:00.123Z",
  "instrument_id": "AAPL",
  "evaluation": {
    "ratings": {
      "1h": "buy",
      "4h": "buy", 
      "1d": "strong_buy",
      "1w": "neutral",
      "1mo": "buy"
    },
    "predictions": {
      "1h": {
        "direction": "positive",
        "confidence": 0.78,
        "price_target": 152.50,
        "probability": 0.78,
        "confidence_interval": {
          "lower_95": 151.20,
          "upper_95": 153.80
        }
      },
      "1d": {
        "direction": "positive", 
        "confidence": 0.85,
        "price_target": 155.25,
        "probability": 0.85,
        "confidence_interval": {
          "lower_95": 152.80,
          "upper_95": 157.70
        }
      }
    },
    "technical_signals": {
      "rsi_signal": "oversold_recovery",
      "macd_signal": "bullish_crossover",
      "pattern_signal": "ascending_triangle"
    },
    "sentiment_score": 0.72,
    "overall_confidence": 0.81,
    "quality_metrics": {
      "feature_quality": 0.89,
      "data_completeness": 0.95,
      "model_agreement": 0.87
    }
  },
  "reasoning": {
    "primary_factors": ["strong_technical_momentum", "positive_sentiment", "earnings_beat"],
    "risk_factors": ["sector_volatility", "market_uncertainty"],
    "confidence_drivers": ["high_model_agreement", "strong_technical_confirmation"]
  }
}
```

#### `MarketPredictionEvent`
```json
{
  "eventId": "uuid",
  "timestamp": "2025-06-21T10:30:00.200Z",
  "instrument_id": "AAPL",
  "timeframe": "1d",
  "prediction": {
    "direction": "positive",
    "confidence": 0.85,
    "price_target": 155.25,
    "probability_distribution": {
      "positive": 0.85,
      "neutral": 0.10,
      "negative": 0.05
    },
    "confidence_interval": {
      "lower_95": 152.80,
      "upper_95": 157.70,
      "lower_80": 153.50,
      "upper_80": 157.00
    },
    "expected_return": 0.025,
    "volatility_forecast": 0.18
  },
  "model_metadata": {
    "model_id": "ensemble_v2.1",
    "models_used": ["gradient_boost", "neural_net", "random_forest"],
    "feature_importance": {
      "rsi_14": 0.25,
      "news_sentiment": 0.20,
      "volume_change": 0.15,
      "macd_signal": 0.12,
      "sector_momentum": 0.10
    },
    "prediction_quality": {
      "feature_quality": 0.89,
      "model_agreement": 0.87,
      "historical_accuracy": 0.74
    }
  }
}
```

#### `ModelPerformanceEvent`
```json
{
  "eventId": "uuid",
  "timestamp": "2025-06-21T10:30:00.300Z",
  "model_id": "ensemble_v2.1",
  "timeframe": "1d",
  "performance_period": {
    "start": "2025-06-01T00:00:00.000Z",
    "end": "2025-06-21T00:00:00.000Z"
  },
  "metrics": {
    "accuracy": 0.74,
    "precision": 0.76,
    "recall": 0.72,
    "f1_score": 0.74,
    "calibration_score": 0.91,
    "log_loss": 0.45,
    "brier_score": 0.18
  },
  "performance_by_rating": {
    "strong_buy": {"accuracy": 0.82, "precision": 0.85, "recall": 0.78},
    "buy": {"accuracy": 0.71, "precision": 0.73, "recall": 0.69},
    "neutral": {"accuracy": 0.65, "precision": 0.62, "recall": 0.68},
    "sell": {"accuracy": 0.69, "precision": 0.71, "recall": 0.67},
    "strong_sell": {"accuracy": 0.79, "precision": 0.81, "recall": 0.77}
  },
  "drift_metrics": {
    "feature_drift": 0.12,
    "prediction_drift": 0.08,
    "performance_drift": 0.05,
    "retraining_recommended": false
  }
}
```
