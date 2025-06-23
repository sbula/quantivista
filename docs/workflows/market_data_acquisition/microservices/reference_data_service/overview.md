# Reference Data Service

## Responsibility
Centralized reference data management for all financial instruments, sectors, currencies, and market metadata. Provides authoritative source for instrument classifications, corporate hierarchies, and static data used across all QuantiVista workflows.

## Technology Stack
- **Language**: Java + Spring Boot + PostgreSQL + Redis caching
- **Libraries**: Spring Boot, JPA, Redis, data validation libraries
- **Scaling**: Horizontal by data volume, heavy caching
- **NFRs**: P99 lookup < 10ms, 99.99% data accuracy, comprehensive coverage

## API Specification

### Core APIs
```pseudo
// Enumerations
enum InstrumentType {
    EQUITY,
    BOND,
    ETF,
    MUTUAL_FUND,
    OPTION,
    FUTURE,
    CURRENCY,
    COMMODITY,
    INDEX
}

enum SectorClassification {
    GICS_SECTOR,
    GICS_INDUSTRY_GROUP,
    GICS_INDUSTRY,
    GICS_SUB_INDUSTRY,
    ICB_SECTOR,
    CUSTOM_SECTOR
}

// Data Models
struct Instrument {
    instrument_id: String
    symbol: String
    name: String
    instrument_type: InstrumentType
    exchange: String
    currency: String
    country: String
    sector_classifications: Map<SectorClassification, String>
    market_cap: Optional<Float>
    shares_outstanding: Optional<Float>
    ipo_date: Optional<Date>
    is_active: Boolean
    metadata: Map<String, Any>
}

struct SectorHierarchy {
    classification_type: SectorClassification
    sector_code: String
    sector_name: String
    parent_sector: Optional<String>
    child_sectors: List<String>
    instruments: List<String>
}

struct CurrencyData {
    currency_code: String
    currency_name: String
    country: String
    is_major: Boolean
    decimal_places: Integer
}

struct ExchangeData {
    exchange_code: String
    exchange_name: String
    country: String
    timezone: String
    trading_hours: TradingHours
    holidays: List<Date>
}

// REST API Endpoints
GET /api/v1/instruments/{instrument_id}
    Response: Instrument

GET /api/v1/instruments/search
    Parameters: symbol, name, sector, exchange
    Response: List<Instrument>

GET /api/v1/sectors/{classification_type}
    Response: List<SectorHierarchy>

GET /api/v1/sectors/{classification_type}/{sector_code}/instruments
    Response: List<String>

GET /api/v1/currencies
    Response: List<CurrencyData>

GET /api/v1/exchanges
    Response: List<ExchangeData>

POST /api/v1/instruments/bulk-lookup
    Request: List<String>
    Response: List<Instrument>
```

### Event Output
```pseudo
Event reference_data_updated {
    event_id: String
    timestamp: DateTime
    reference_data_updated: ReferenceDataUpdatePayload
}

struct ReferenceDataUpdatePayload {
    instrument_id: String
    update_type: String
    changes: ChangeDetailsData
    effective_date: String
    source: String
}

struct ChangeDetailsData {
    gics_industry: FieldChangeData
}

struct FieldChangeData {
    old_value: String
    new_value: String
}

// Example Event Data
{
    event_id: "uuid",
    timestamp: "2025-06-21T10:00:00.000Z",
    reference_data_updated: {
        instrument_id: "AAPL",
        update_type: "sector_classification_change",
        changes: {
            gics_industry: {
                old_value: "Technology Hardware & Equipment",
                new_value: "Technology Hardware, Storage & Peripherals"
            }
        },
        effective_date: "2025-06-21",
        source: "MSCI_GICS_Update"
    }
}
```

## Database Schema

### PostgreSQL (Master Data)
```pseudo
Table instruments {
    id: UUID (primary key, auto-generated)
    instrument_id: String (required, unique, max_length: 20)
    symbol: String (required, max_length: 20)
    name: String (required, max_length: 200)
    instrument_type: String (required, max_length: 50)
    exchange: String (required, max_length: 20)
    currency: String (required, max_length: 3)
    country: String (required, max_length: 50)
    market_cap: Integer
    shares_outstanding: Integer
    ipo_date: Date
    is_active: Boolean (default: true)
    metadata: JSON
    created_at: Timestamp (default: now)
    updated_at: Timestamp (default: now)
}

Table sector_classifications {
    id: UUID (primary key, auto-generated)
    instrument_id: String (required, max_length: 20, foreign_key: instruments.instrument_id)
    classification_type: String (required, max_length: 50)
    sector_code: String (required, max_length: 20)
    sector_name: String (required, max_length: 200)
    level: Integer (required)
    parent_sector: String (max_length: 20)
    effective_date: Date (required)
    expiry_date: Date
}

Table currencies {
    currency_code: String (primary key, max_length: 3)
    currency_name: String (required, max_length: 100)
    country: String (required, max_length: 50)
    is_major: Boolean (default: false)
    decimal_places: Integer (default: 2)
    created_at: Timestamp (default: now)
}

Table exchanges {
    exchange_code: String (primary key, max_length: 20)
    exchange_name: String (required, max_length: 200)
    country: String (required, max_length: 50)
    timezone: String (required, max_length: 50)
    trading_hours: JSON (required)
    holidays: JSON
    created_at: Timestamp (default: now)
}
```

### Redis Caching
```pseudo
struct ReferenceDataCache {
    // Instrument cache: "instrument:{instrument_id}" -> Instrument
    // Symbol lookup: "symbol:{symbol}" -> instrument_id
    // Sector instruments: "sector:{classification}:{code}" -> List<instrument_id>
    // Exchange data: "exchange:{exchange_code}" -> ExchangeData
    // Currency data: "currency:{currency_code}" -> CurrencyData
}
```

## External Data Sources

### **Primary Sources:**
- **Bloomberg Reference Data**: Comprehensive instrument data, corporate actions
- **Refinitiv (LSEG)**: Global reference data, sector classifications
- **MSCI GICS**: Global Industry Classification Standard
- **ICB (Industry Classification Benchmark)**: Alternative sector classification
- **ISO Standards**: Currency codes (ISO 4217), country codes (ISO 3166)

### **Secondary Sources:**
- **Exchange Websites**: Trading hours, holidays, listing requirements
- **Regulatory Filings**: SEC EDGAR, company annual reports
- **Data Vendors**: Morningstar, FactSet for additional metadata

## Implementation Estimation

### Priority: **CRITICAL** (Foundation for all services)
### Estimated Time: **4-5 weeks**

#### Week 1-2: Core Reference Data Engine
- Java service setup with Spring Boot
- Basic instrument and sector data management
- External data source integration
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

#### Week 3: Advanced Features
- Sector hierarchy management
- Currency and exchange data
- Data validation and quality checks
- **Effort**: 2 developers × 1 week = 2 dev-weeks

#### Week 4-5: Integration & Optimization
- Redis caching optimization
- Bulk lookup APIs
- Integration with all QuantiVista services
- **Effort**: 2 developers × 2 weeks = 4 dev-weeks

### Total Effort: **10 dev-weeks**
### Team Size: **2 developers** (1 senior Java developer, 1 data specialist)
### Dependencies: Bloomberg/Refinitiv APIs, MSCI GICS data, PostgreSQL, Redis

### Success Criteria:
- P99 lookup latency < 10ms
- 99.99% data accuracy
- Comprehensive instrument coverage (50K+ instruments)
- Real-time reference data updates
- Support for multiple sector classification systems
