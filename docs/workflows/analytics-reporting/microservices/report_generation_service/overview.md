# Report Generation Service

## Responsibility
Automated report generation and formatting service for financial reports, regulatory filings, and client communications. Supports multiple output formats including PDF, Excel, Word, and HTML with professional formatting and branding.

## Technology Stack
- **Language**: Python + ReportLab + Jinja2 + Polars
- **Data Processing**: Polars for high-performance data manipulation (5-10x faster than pandas)
- **Analytics**: DuckDB for complex analytical queries and aggregations
- **Libraries**: ReportLab, Jinja2, Polars, openpyxl, python-docx, WeasyPrint
- **Scaling**: Horizontal by report complexity
- **NFRs**: P99 report generation < 30s, professional formatting, multi-format support

## API Specification

### Core APIs
```pseudo
// Enumerations
enum ReportType {
    PORTFOLIO_PERFORMANCE,
    RISK_ANALYSIS,
    COMPLIANCE_REPORT,
    CLIENT_STATEMENT,
    REGULATORY_FILING,
    EXECUTIVE_SUMMARY,
    CUSTOM_REPORT
}

enum OutputFormat {
    PDF,
    EXCEL,
    WORD,
    HTML,
    CSV,
    JSON
}

enum ReportFrequency {
    ON_DEMAND,
    DAILY,
    WEEKLY,
    MONTHLY,
    QUARTERLY,
    ANNUALLY
}

// Data Models
struct ReportGenerationRequest {
    report_id: String
    report_type: ReportType
    template_id: String
    output_format: OutputFormat
    data_sources: List<String>
    parameters: ReportParameters
    formatting: ReportFormatting
    delivery: DeliveryOptions
}

struct ReportParameters {
    date_range: DateRange
    portfolios: List<String>
    clients: List<String>
    include_charts: Boolean
    include_appendices: Boolean
    custom_sections: List<String>
    filters: Map<String, Any>
}

struct ReportFormatting {
    template_id: String
    branding: BrandingConfig
    page_layout: PageLayout
    chart_styling: ChartStyling
    font_settings: FontSettings
    color_scheme: ColorScheme
}

struct GeneratedReport {
    report_id: String
    report_type: ReportType
    output_format: OutputFormat
    file_path: String
    file_size_bytes: Integer
    generation_time_ms: Float
    page_count: Integer
    sections: List<ReportSection>
    metadata: ReportMetadata
}

struct ReportTemplate {
    template_id: String
    template_name: String
    report_type: ReportType
    template_config: TemplateConfig
    sections: List<SectionTemplate>
    styling: TemplateStyle
}

// REST API Endpoints
POST /api/v1/reports/generate
    Request: ReportGenerationRequest
    Response: GeneratedReport

GET /api/v1/reports/{report_id}/status
    Response: ReportStatus

GET /api/v1/reports/{report_id}/download
    Response: FileDownload

POST /api/v1/reports/schedule
    Request: ScheduledReportRequest
    Response: ScheduledReport

GET /api/v1/reports/templates
    Response: List<ReportTemplate>

POST /api/v1/reports/templates
    Request: ReportTemplate
    Response: ReportTemplate
```

### Event Output
```pseudo
Event report_generated {
    event_id: String
    timestamp: DateTime
    report_generated: ReportGeneratedData
}

struct ReportGeneratedData {
    report_id: String
    report_type: String
    output_format: String
    parameters: ReportParametersData
    file_details: FileDetailsData
    generation_time_ms: Integer
    sections: List<String>
}

struct ReportParametersData {
    portfolio_id: String
    date_range: DateRangeData
    include_charts: Boolean
    include_appendices: Boolean
}

struct DateRangeData {
    start_date: String
    end_date: String
}

struct FileDetailsData {
    file_path: String
    file_size_bytes: Integer
    page_count: Integer
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    report_generated: {
        report_id: "report_20250621_001",
        report_type: "PORTFOLIO_PERFORMANCE",
        output_format: "PDF",
        parameters: {
            portfolio_id: "portfolio_001",
            date_range: {
                start_date: "2025-06-01",
                end_date: "2025-06-21"
            },
            include_charts: true,
            include_appendices: true
        },
        file_details: {
            file_path: "/reports/portfolio_performance_20250621.pdf",
            file_size_bytes: 2048576,
            page_count: 15
        },
        generation_time_ms: 12500,
        sections: [
            "executive_summary",
            "performance_overview",
            "risk_analysis",
            "holdings_detail",
            "appendices"
        ]
    }
}
```

## Database Schema

### PostgreSQL (Command Side)
```pseudo
Table report_templates {
    id: UUID (primary key, auto-generated)
    template_id: String (required, unique, max_length: 100)
    template_name: String (required, max_length: 200)
    report_type: String (required, max_length: 50)
    template_config: JSON (required)
    sections: JSON (required)
    styling: JSON (required)
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}

Table generated_reports {
    id: UUID (primary key, auto-generated)
    report_id: String (required, unique, max_length: 100)
    report_type: String (required, max_length: 50)
    template_id: String (required, max_length: 100)
    output_format: String (required, max_length: 20)
    parameters: JSON (required)
    file_path: String (max_length: 500)
    file_size_bytes: Integer
    generation_time_ms: Integer
    status: String (required, max_length: 20)
    created_at: Timestamp (default: now)
}

Table scheduled_reports {
    id: UUID (primary key, auto-generated)
    schedule_id: String (required, unique, max_length: 100)
    report_type: String (required, max_length: 50)
    template_id: String (required, max_length: 100)
    frequency: String (required, max_length: 20)
    schedule_config: JSON (required)
    delivery_config: JSON (required)
    next_run_time: Timestamp
    enabled: Boolean (default: true)
    created_at: Timestamp (default: now)
}
```

## Implementation Estimation

### Priority: **HIGH** (Critical for client communication)
### Estimated Time: **5-6 weeks**

#### Week 1-2: Core Report Engine
- Python service setup with ReportLab
- Basic PDF and Excel generation
- Template system implementation
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3-4: Advanced Features
- Multi-format support (Word, HTML)
- Chart integration and formatting
- Professional styling and branding
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 5-6: Automation & Integration
- Scheduled report generation
- Integration with visualization service
- Delivery automation and optimization
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **12 dev-weeks**
### Team Size: **2 developers** (1 senior Python developer, 1 report design specialist)
### Dependencies: Analytics engine, visualization service, data sources

### Success Criteria:
- P99 report generation < 30 seconds
- Professional formatting and branding
- Multi-format support (PDF, Excel, Word, HTML)
- Automated scheduling and delivery
- Template customization capability
- Integration with charts and visualizations
