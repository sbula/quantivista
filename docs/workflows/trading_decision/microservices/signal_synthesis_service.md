# Signal Synthesis Service

## Responsibility
Multi-timeframe signal analysis and synthesis to create unified trading signals. Combines signals from different timeframes, resolves conflicts, and produces coherent trading recommendations with optimal timing.

## Technology Stack
- **Language**: Python + scikit-learn + NumPy + Pandas
- **Libraries**: scikit-learn, numpy, pandas, scipy for statistical analysis
- **Scaling**: Horizontal by signal complexity
- **NFRs**: P99 synthesis < 150ms, optimal signal quality, timeframe consistency

## API Specification

### Core APIs
```pseudo
// Enumerations
enum SynthesisMethod {
    WEIGHTED_CONSENSUS,
    DOMINANT_TIMEFRAME,
    MAJORITY_VOTE
}

enum ConflictResolution {
    CONSERVATIVE,
    AGGRESSIVE,
    NEUTRAL
}

// Data Models
struct TimeframeSynthesisRequest {
    instrument_id: String
    signals: List<TradingSignal>  // Signals from different timeframes
    synthesis_method: SynthesisMethod
    conflict_resolution: ConflictResolution
}

struct SynthesizedSignal {
    signal_id: String
    instrument_id: String
    direction: SignalDirection
    strength: SignalStrength
    confidence: Float
    timeframe_consensus: TimeframeConsensus
    optimal_entry_timing: OptimalTiming
    synthesis_metadata: SynthesisMetadata
}

struct TimeframeConsensus {
    short_term_signals: List<String>  // 1m, 5m, 15m
    medium_term_signals: List<String>  // 1h, 4h
    long_term_signals: List<String>   // 1d, 1w
    consensus_score: Float
    conflicting_timeframes: List<String>
    dominant_timeframe: String
}

struct OptimalTiming {
    recommended_entry_time: String
    timing_confidence: Float
    expected_delay_minutes: Optional<Integer>
}

// REST API Endpoints
POST /api/v1/synthesize
    Request: TimeframeSynthesisRequest
    Response: SynthesizedSignal
```

### Event Output
```pseudo
Event signal_synthesized {
    event_id: String
    timestamp: DateTime
    signal_synthesized: SynthesizedSignalData
}

struct SynthesizedSignalData {
    signal_id: String
    instrument_id: String
    direction: String
    strength: String
    confidence: Float
    timeframe_consensus: TimeframeConsensusData
    optimal_entry_timing: OptimalTimingData
}

struct TimeframeConsensusData {
    consensus_score: Float
    dominant_timeframe: String
}

struct OptimalTimingData {
    recommended_entry_time: String
    timing_confidence: Float
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    signal_synthesized: {
        signal_id: "synth_20250621_001",
        instrument_id: "AAPL",
        direction: "BUY",
        strength: "STRONG",
        confidence: 0.82,
        timeframe_consensus: {
            consensus_score: 0.85,
            dominant_timeframe: "1h"
        },
        optimal_entry_timing: {
            recommended_entry_time: "WAIT_FOR_PULLBACK",
            timing_confidence: 0.78
        }
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table synthesis_rules {
    id: UUID (primary key, auto-generated)
    rule_name: String (required, max_length: 100)
    timeframe_weights: JSON (required)
    conflict_resolution_method: String (required, max_length: 50)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table timeframe_configurations {
    id: UUID (primary key, auto-generated)
    instrument_id: String (max_length: 20)
    timeframe: String (required, max_length: 10)
    weight: Float (required)
    priority: Integer (default: 1)
    created_at: Timestamp (default: now)
}
```

### TimescaleDB (Query Side)
```pseudo
Table synthesized_signals_ts {
    timestamp: Timestamp (required, partition_key)
    signal_id: String (required, max_length: 100)
    instrument_id: String (required, max_length: 20)
    direction: String (required, max_length: 10)
    confidence: Float (required)
    timeframe_consensus: JSON
    synthesis_metadata: JSON

    // Hypertable Configuration
    partition_by: timestamp (chunk_interval: 1 day)
}
```

## Implementation Estimation

### Priority: **HIGH** (Critical for signal quality)
### Estimated Time: **3-4 weeks**

#### Week 1-2: Core Synthesis Engine
- Multi-timeframe signal analysis algorithms
- Consensus building and conflict resolution
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3-4: Integration & Testing
- Optimal timing detection algorithms
- Integration with Signal Generation Service
- Performance testing and optimization
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **8 dev-weeks**
### Team Size: **2 developers**
### Dependencies: Signal Generation Service, market data

### Success Criteria:
- P99 synthesis latency < 150ms
- 95% timeframe consensus accuracy
- Support for 6+ timeframes simultaneously
- Optimal timing prediction accuracy > 80%
