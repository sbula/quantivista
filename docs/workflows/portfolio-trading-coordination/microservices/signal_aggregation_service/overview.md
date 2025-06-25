# Signal Aggregation Service

## Overview
The Signal Aggregation Service aggregates and prioritizes trading signals from multiple sources within the QuantiVista platform. It collects signals from various workflows, performs quality-based filtering, resolves conflicts, and provides prioritized signal streams for coordinated trading decisions.

## Technology Stack
- **Primary Language**: Go
- **Data Processing**: Polars (high-performance DataFrames)
- **Analytics Database**: DuckDB (analytical queries)
- **ML Framework**: JAX for mathematical optimization and advanced models
- **API Framework**: Gin (Go web framework)
- **Message Queue**: Apache Pulsar
- **Caching**: Redis
- **Monitoring**: Prometheus + Grafana
- **Containerization**: Docker

## Core Responsibilities

### Multi-Source Signal Collection
- Real-time signal ingestion from multiple workflows
- Signal buffering and temporary storage
- Signal format normalization and standardization
- Source reliability tracking and weighting
- Signal metadata enrichment

### Signal Priority Scoring
- Multi-dimensional signal scoring using DuckDB analytics
- Source credibility and track record weighting
- Signal freshness and timing considerations
- Market condition context adjustment
- Confidence level assessment

### Signal Timing Coordination
- Signal batching for coordinated execution
- Timing optimization based on market conditions
- Cross-signal dependency management
- Execution window coordination
- Market impact timing considerations

### Quality-Based Filtering
- Signal quality assessment and validation
- Outlier detection and filtering
- Consistency checks across signal sources
- Historical performance validation
- Real-time quality monitoring

### Signal Deduplication
- Duplicate signal detection across sources
- Signal correlation analysis
- Redundant signal elimination
- Source attribution for similar signals
- Signal uniqueness scoring

## API Specification

### Input APIs
```pseudo
// Signal ingestion from workflows
POST /api/v1/signals/ingest
{
    source_workflow: string,
    signal_type: string,
    instrument_id: string,
    signal_strength: float,
    confidence: float,
    timestamp: timestamp,
    metadata: object,
    expiry_time: timestamp
}

// Bulk signal ingestion
POST /api/v1/signals/bulk-ingest
{
    signals: [
        {
            source_workflow: string,
            signal_type: string,
            instrument_id: string,
            signal_strength: float,
            confidence: float,
            timestamp: timestamp,
            metadata: object
        }
    ]
}

// Signal priority request
GET /api/v1/signals/prioritized
{
    instrument_filter: [string],
    signal_types: [string],
    min_confidence: float,
    max_age: duration,
    limit: integer
}

// Signal aggregation request
POST /api/v1/signals/aggregate
{
    aggregation_window: duration,
    weighting_scheme: string,
    conflict_resolution: string,
    output_format: string
}
```

### Output APIs
```pseudo
// Prioritized signal stream
{
    signal_id: string,
    aggregated_signals: [
        {
            source_workflow: string,
            signal_type: string,
            instrument_id: string,
            signal_strength: float,
            confidence: float,
            priority_score: float,
            quality_score: float,
            timestamp: timestamp
        }
    ],
    aggregation_metadata: {
        total_signals: integer,
        filtered_signals: integer,
        conflict_resolutions: integer,
        aggregation_timestamp: timestamp
    }
}

// Signal quality metrics
{
    source_workflow: string,
    quality_metrics: {
        signal_count: integer,
        average_quality: float,
        reliability_score: float,
        latency_metrics: object,
        accuracy_metrics: object
    },
    performance_tracking: object
}

// Aggregation statistics
{
    aggregation_period: {start: timestamp, end: timestamp},
    statistics: {
        total_signals_processed: integer,
        signals_by_source: object,
        signals_by_type: object,
        average_priority_score: float,
        conflict_resolution_rate: float
    }
}
```

## Data Sources
- **Trading Decision Workflow**: Trading signals and recommendations
- **Market Prediction Workflow**: Prediction-based signals
- **Market Intelligence Workflow**: News and sentiment signals
- **Instrument Analysis Workflow**: Technical analysis signals
- **Portfolio Management Workflow**: Rebalancing signals

## Data Outputs
- **Coordination Engine Service**: Prioritized and aggregated signals
- **Policy Enforcement Service**: Signals for compliance checking
- **Position Sizing Service**: Signal inputs for sizing decisions
- **Coordination Distribution Service**: Signal aggregation results

## Database Schema

### Signal Buffer (Write Model)
```pseudo
Table: signal_buffer
- signal_id: UUID (Primary Key)
- source_workflow: String
- signal_type: String
- instrument_id: String
- signal_strength: Float
- confidence: Float
- priority_score: Float
- quality_score: Float
- timestamp: Timestamp
- expiry_time: Timestamp
- metadata: JSONB
- processed: Boolean
- created_at: Timestamp

Index: idx_signal_instrument_time ON (instrument_id, timestamp)
Index: idx_signal_source_type ON (source_workflow, signal_type)
Index: idx_signal_priority ON (priority_score DESC, timestamp DESC)
Index: idx_signal_expiry ON (expiry_time)
```

### Aggregated Signals (Read Model)
```pseudo
Table: aggregated_signals_view
- aggregation_id: UUID
- instrument_id: String
- aggregated_strength: Float
- weighted_confidence: Float
- signal_count: Integer
- source_diversity: Float
- aggregation_timestamp: Timestamp
- expiry_time: Timestamp

Index: idx_aggregated_instrument ON (instrument_id)
Index: idx_aggregated_timestamp ON (aggregation_timestamp)
```

### Signal Quality Metrics (Read Model)
```pseudo
Table: signal_quality_metrics_view
- source_workflow: String
- signal_type: String
- date: Date
- signal_count: Integer
- average_quality: Float
- reliability_score: Float
- accuracy_rate: Float
- latency_p95: Float

Index: idx_quality_source_date ON (source_workflow, date)
Index: idx_quality_type_date ON (signal_type, date)
```

## Performance Requirements
- **Signal Ingestion**: Process 10,000+ signals per second
- **Aggregation Latency**: < 100ms for real-time aggregation
- **Signal Buffering**: Support 1M+ active signals
- **Priority Scoring**: < 50ms per signal scoring
- **Deduplication**: < 10ms per signal comparison

## Monitoring and Alerting
- Signal ingestion rates and latency
- Signal quality degradation detection
- Source reliability tracking
- Aggregation performance metrics
- Buffer overflow and capacity alerts
- Signal expiry and cleanup monitoring
