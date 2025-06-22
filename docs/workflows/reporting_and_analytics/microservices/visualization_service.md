# Visualization Service

## Responsibility
Interactive data visualization and charting service for financial analytics. Generates dynamic charts, dashboards, and interactive visualizations for portfolio performance, risk analysis, and market intelligence.

## Technology Stack
- **Language**: Node.js + TypeScript + D3.js + React
- **Libraries**: D3.js, Chart.js, Plotly.js, React, Express.js
- **Scaling**: Horizontal by visualization complexity, CDN for static assets
- **NFRs**: P99 chart generation < 2s, interactive real-time updates, mobile responsive

## API Specification

### Core APIs
```pseudo
// Enumerations
enum ChartType {
    LINE_CHART,
    BAR_CHART,
    CANDLESTICK_CHART,
    HEATMAP,
    SCATTER_PLOT,
    PIE_CHART,
    TREEMAP,
    CORRELATION_MATRIX,
    PERFORMANCE_ATTRIBUTION,
    RISK_DASHBOARD
}

enum DataFrequency {
    REAL_TIME,
    MINUTE,
    HOURLY,
    DAILY,
    WEEKLY,
    MONTHLY
}

enum OutputFormat {
    SVG,
    PNG,
    PDF,
    INTERACTIVE_HTML,
    JSON_CONFIG
}

// Data Models
struct VisualizationRequest {
    chart_id: String
    chart_type: ChartType
    data_source: String
    parameters: ChartParameters
    styling: ChartStyling
    output_format: OutputFormat
    interactive: Boolean
}

struct ChartParameters {
    date_range: DateRange
    instruments: List<String>
    portfolios: List<String>
    metrics: List<String>
    aggregation_level: String
    filters: Map<String, Any>
}

struct ChartStyling {
    theme: String
    color_palette: List<String>
    dimensions: ChartDimensions
    annotations: List<Annotation>
    branding: BrandingConfig
}

struct VisualizationResponse {
    chart_id: String
    chart_type: ChartType
    chart_data: ChartData
    chart_config: ChartConfig
    generation_time_ms: Float
    cache_key: Optional<String>
}

struct ChartData {
    datasets: List<Dataset>
    labels: List<String>
    metadata: DataMetadata
}

struct Dashboard {
    dashboard_id: String
    dashboard_name: String
    layout: DashboardLayout
    charts: List<ChartReference>
    filters: List<DashboardFilter>
    refresh_interval: Integer
}

// REST API Endpoints
POST /api/v1/visualizations/generate
    Request: VisualizationRequest
    Response: VisualizationResponse

GET /api/v1/visualizations/{chart_id}
    Response: VisualizationResponse

POST /api/v1/dashboards/create
    Request: DashboardRequest
    Response: Dashboard

GET /api/v1/dashboards/{dashboard_id}
    Response: Dashboard

GET /api/v1/visualizations/templates
    Response: List<ChartTemplate>

WebSocket /api/v1/visualizations/stream
    Parameters: chart_id, update_frequency
    Response: Stream<ChartUpdate>
```

### Event Output
```pseudo
Event visualization_generated {
    event_id: String
    timestamp: DateTime
    visualization_generated: VisualizationData
}

struct VisualizationData {
    chart_id: String
    chart_type: String
    parameters: VisualizationParametersData
    chart_data: ChartDataInfo
    output_format: String
    generation_time_ms: Integer
    interactive_features: List<String>
}

struct VisualizationParametersData {
    portfolio_id: String
    date_range: DateRangeData
    metrics: List<String>
}

struct DateRangeData {
    start_date: String
    end_date: String
}

struct ChartDataInfo {
    datasets_count: Integer
    data_points: Integer
    time_series_length: Integer
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    visualization_generated: {
        chart_id: "chart_20250621_001",
        chart_type: "PERFORMANCE_ATTRIBUTION",
        parameters: {
            portfolio_id: "portfolio_001",
            date_range: {
                start_date: "2025-01-01",
                end_date: "2025-06-21"
            },
            metrics: ["total_return", "attribution_breakdown"]
        },
        chart_data: {
            datasets_count: 3,
            data_points: 150,
            time_series_length: 172
        },
        output_format: "INTERACTIVE_HTML",
        generation_time_ms: 1250,
        interactive_features: [
            "zoom",
            "pan",
            "tooltip",
            "drill_down"
        ]
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table chart_templates {
    id: UUID (primary key, auto-generated)
    template_id: String (required, unique, max_length: 100)
    template_name: String (required, max_length: 200)
    chart_type: String (required, max_length: 50)
    default_config: JSON (required)
    styling_config: JSON (required)
    category: String (max_length: 50)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table dashboards {
    id: UUID (primary key, auto-generated)
    dashboard_id: String (required, unique, max_length: 100)
    dashboard_name: String (required, max_length: 200)
    owner_id: String (required, max_length: 100)
    layout_config: JSON (required)
    chart_references: JSON (required)
    filters: JSON
    refresh_interval: Integer (default: 300)
    public: Boolean (default: false)
    created_at: Timestamp (default: now)
}

Table visualization_cache {
    id: UUID (primary key, auto-generated)
    cache_key: String (required, unique, max_length: 255)
    chart_type: String (required, max_length: 50)
    parameters_hash: String (required, max_length: 64)
    chart_data: JSON (required)
    chart_config: JSON (required)
    expiry_time: Timestamp (required)
    created_at: Timestamp (default: now)
}
```

### Redis Caching
```pseudo
Cache visualization_cache {
    // Chart cache
    "chart:{cache_key}": ChartData (TTL: 5m)

    // Dashboard cache
    "dashboard:{dashboard_id}": Dashboard (TTL: 15m)

    // Template cache
    "template:{template_id}": ChartTemplate (TTL: 1h)

    // Real-time data
    "realtime:{chart_id}": LiveData (TTL: 30s)
}
```

## Implementation Estimation

### Priority: **MEDIUM** (Important for user experience)
### Estimated Time: **5-6 weeks**

#### Week 1-2: Core Visualization Engine
- Node.js service setup with D3.js
- Basic chart generation capabilities
- Chart template system
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3-4: Interactive Features
- Real-time chart updates
- Interactive dashboard creation
- Advanced chart types (heatmaps, treemaps)
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 5-6: Integration & Optimization
- Integration with analytics services
- Performance optimization and caching
- Mobile responsiveness and accessibility
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **12 dev-weeks**
### Team Size: **2 developers** (1 senior frontend developer, 1 data visualization specialist)
### Dependencies: Analytics engine, data ingestion service

### Success Criteria:
- P99 chart generation < 2 seconds
- Interactive real-time updates
- Mobile responsive design
- Support for 10+ chart types
- Dashboard creation and management
- Real-time data streaming capability
