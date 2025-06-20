# Order Management Service

## Purpose and Boundaries

### Purpose
The Order Management Service (OMS) manages the complete lifecycle of orders from creation to settlement with comprehensive audit trails. It serves as the central hub for all order-related operations within the QuantiVista platform, ensuring reliable order processing, tracking, and compliance.

### Strict Boundaries
- **Responsible for**: Order lifecycle management, pre-trade risk checks, order validation, order history tracking, and post-trade processing
- **Not responsible for**: Making trading decisions, executing orders with brokers, portfolio management, or strategy implementation
- **Interfaces with**: Trading Strategy Service (receives trade decisions), Broker Integration Service (sends orders for execution), Risk Analysis Service (for pre-trade checks), Portfolio Management Service (for position updates)

## Place in Workflow

The Order Management Service is a critical component in the Trade Execution Workflow:

1. **Receives trade decisions** from the Trading Strategy Service
2. **Performs pre-trade risk checks** and compliance validation
3. **Creates and validates orders** based on trade decisions
4. **Routes orders** to the Broker Integration Service for execution
5. **Monitors order status** and handles execution reports
6. **Processes trade confirmations** and settlement instructions
7. **Updates positions** and provides order history
8. **Maintains comprehensive audit trails** for compliance and reporting

## API Design (API-First)

### REST API

#### Order Management Endpoints

```yaml
openapi: 3.0.3
info:
  title: Order Management Service API
  version: 1.0.0
  description: API for managing the complete lifecycle of orders
paths:
  /orders:
    post:
      summary: Create a new order
      operationId: createOrder
      tags: [Orders]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateOrderRequest'
      responses:
        '201':
          description: Order created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/OrderResponse'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'
    get:
      summary: List orders with filtering
      operationId: listOrders
      tags: [Orders]
      parameters:
        - name: status
          in: query
          schema:
            type: string
            enum: [PENDING, OPEN, FILLED, PARTIALLY_FILLED, CANCELED, REJECTED]
        - name: accountId
          in: query
          schema:
            type: string
        - name: symbol
          in: query
          schema:
            type: string
        - name: fromDate
          in: query
          schema:
            type: string
            format: date-time
        - name: toDate
          in: query
          schema:
            type: string
            format: date-time
        - name: page
          in: query
          schema:
            type: integer
            default: 0
        - name: size
          in: query
          schema:
            type: integer
            default: 20
      responses:
        '200':
          description: List of orders
          content:
            application/json:
              schema:
                type: object
                properties:
                  content:
                    type: array
                    items:
                      $ref: '#/components/schemas/OrderResponse'
                  page:
                    type: integer
                  size:
                    type: integer
                  totalElements:
                    type: integer
                  totalPages:
                    type: integer
        '401':
          $ref: '#/components/responses/Unauthorized'
  
  /orders/{orderId}:
    get:
      summary: Get order by ID
      operationId: getOrder
      tags: [Orders]
      parameters:
        - name: orderId
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Order details
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/OrderResponse'
        '404':
          description: Order not found
        '401':
          $ref: '#/components/responses/Unauthorized'
    
    put:
      summary: Update order (cancel, modify)
      operationId: updateOrder
      tags: [Orders]
      parameters:
        - name: orderId
          in: path
          required: true
          schema:
            type: string
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UpdateOrderRequest'
      responses:
        '200':
          description: Order updated successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/OrderResponse'
        '400':
          $ref: '#/components/responses/BadRequest'
        '404':
          description: Order not found
        '401':
          $ref: '#/components/responses/Unauthorized'

components:
  schemas:
    CreateOrderRequest:
      type: object
      required: [accountId, symbol, side, type, quantity]
      properties:
        accountId:
          type: string
          description: Account identifier
        symbol:
          type: string
          description: Instrument symbol
        side:
          type: string
          enum: [BUY, SELL]
          description: Order side
        type:
          type: string
          enum: [MARKET, LIMIT, STOP, STOP_LIMIT]
          description: Order type
        quantity:
          type: number
          format: double
          description: Order quantity
        price:
          type: number
          format: double
          description: Limit price (required for LIMIT and STOP_LIMIT orders)
        stopPrice:
          type: number
          format: double
          description: Stop price (required for STOP and STOP_LIMIT orders)
        timeInForce:
          type: string
          enum: [DAY, GTC, IOC, FOK]
          default: DAY
          description: Time in force
        strategyId:
          type: string
          description: ID of the strategy that generated this order
        notes:
          type: string
          description: Additional notes or comments
    
    UpdateOrderRequest:
      type: object
      properties:
        action:
          type: string
          enum: [CANCEL, MODIFY]
          description: Action to perform on the order
        price:
          type: number
          format: double
          description: New limit price (for MODIFY action)
        quantity:
          type: number
          format: double
          description: New quantity (for MODIFY action)
        stopPrice:
          type: number
          format: double
          description: New stop price (for MODIFY action)
        notes:
          type: string
          description: Additional notes or comments
    
    OrderResponse:
      type: object
      properties:
        id:
          type: string
          description: Order ID
        accountId:
          type: string
          description: Account identifier
        symbol:
          type: string
          description: Instrument symbol
        side:
          type: string
          enum: [BUY, SELL]
          description: Order side
        type:
          type: string
          enum: [MARKET, LIMIT, STOP, STOP_LIMIT]
          description: Order type
        status:
          type: string
          enum: [PENDING, OPEN, FILLED, PARTIALLY_FILLED, CANCELED, REJECTED]
          description: Order status
        quantity:
          type: number
          format: double
          description: Order quantity
        filledQuantity:
          type: number
          format: double
          description: Filled quantity
        price:
          type: number
          format: double
          description: Limit price
        stopPrice:
          type: number
          format: double
          description: Stop price
        averageExecutionPrice:
          type: number
          format: double
          description: Average execution price
        timeInForce:
          type: string
          enum: [DAY, GTC, IOC, FOK]
          description: Time in force
        strategyId:
          type: string
          description: ID of the strategy that generated this order
        brokerOrderId:
          type: string
          description: Broker's order ID
        rejectionReason:
          type: string
          description: Reason for rejection (if status is REJECTED)
        notes:
          type: string
          description: Additional notes or comments
        createdAt:
          type: string
          format: date-time
          description: Order creation timestamp
        updatedAt:
          type: string
          format: date-time
          description: Last update timestamp
        executionReports:
          type: array
          items:
            $ref: '#/components/schemas/ExecutionReport'
    
    ExecutionReport:
      type: object
      properties:
        id:
          type: string
          description: Execution report ID
        executionId:
          type: string
          description: Execution ID
        quantity:
          type: number
          format: double
          description: Executed quantity
        price:
          type: number
          format: double
          description: Execution price
        timestamp:
          type: string
          format: date-time
          description: Execution timestamp
        fees:
          type: number
          format: double
          description: Execution fees
        venue:
          type: string
          description: Execution venue
  
  responses:
    BadRequest:
      description: Invalid request
      content:
        application/json:
          schema:
            type: object
            properties:
              message:
                type: string
              errors:
                type: array
                items:
                  type: object
                  properties:
                    field:
                      type: string
                    message:
                      type: string
    
    Unauthorized:
      description: Authentication required
      content:
        application/json:
          schema:
            type: object
            properties:
              message:
                type: string
                example: Authentication required
```

### gRPC API (Internal Services)

```protobuf
syntax = "proto3";

package trading.order.v1;

import "google/protobuf/timestamp.proto";

service OrderService {
  // Create a new order
  rpc CreateOrder(CreateOrderRequest) returns (OrderResponse);
  
  // Get order by ID
  rpc GetOrder(GetOrderRequest) returns (OrderResponse);
  
  // Update order status based on execution reports
  rpc UpdateOrderStatus(UpdateOrderStatusRequest) returns (OrderResponse);
  
  // Cancel an existing order
  rpc CancelOrder(CancelOrderRequest) returns (OrderResponse);
  
  // Stream order status updates
  rpc StreamOrderUpdates(StreamOrderUpdatesRequest) returns (stream OrderUpdate);
}

message CreateOrderRequest {
  string account_id = 1;
  string symbol = 2;
  OrderSide side = 3;
  OrderType type = 4;
  double quantity = 5;
  double price = 6;
  double stop_price = 7;
  TimeInForce time_in_force = 8;
  string strategy_id = 9;
  string notes = 10;
}

message GetOrderRequest {
  string order_id = 1;
}

message UpdateOrderStatusRequest {
  string order_id = 1;
  OrderStatus status = 2;
  ExecutionReport execution_report = 3;
}

message CancelOrderRequest {
  string order_id = 1;
  string reason = 2;
}

message StreamOrderUpdatesRequest {
  string account_id = 1;
  repeated string symbols = 2;
  repeated OrderStatus statuses = 3;
}

message OrderResponse {
  string id = 1;
  string account_id = 2;
  string symbol = 3;
  OrderSide side = 4;
  OrderType type = 5;
  OrderStatus status = 6;
  double quantity = 7;
  double filled_quantity = 8;
  double price = 9;
  double stop_price = 10;
  double average_execution_price = 11;
  TimeInForce time_in_force = 12;
  string strategy_id = 13;
  string broker_order_id = 14;
  string rejection_reason = 15;
  string notes = 16;
  google.protobuf.Timestamp created_at = 17;
  google.protobuf.Timestamp updated_at = 18;
  repeated ExecutionReport execution_reports = 19;
}

message OrderUpdate {
  string order_id = 1;
  OrderStatus status = 2;
  double filled_quantity = 3;
  double average_execution_price = 4;
  google.protobuf.Timestamp timestamp = 5;
  ExecutionReport latest_execution = 6;
}

message ExecutionReport {
  string id = 1;
  string execution_id = 2;
  double quantity = 3;
  double price = 4;
  google.protobuf.Timestamp timestamp = 5;
  double fees = 6;
  string venue = 7;
}

enum OrderSide {
  ORDER_SIDE_UNSPECIFIED = 0;
  ORDER_SIDE_BUY = 1;
  ORDER_SIDE_SELL = 2;
}

enum OrderType {
  ORDER_TYPE_UNSPECIFIED = 0;
  ORDER_TYPE_MARKET = 1;
  ORDER_TYPE_LIMIT = 2;
  ORDER_TYPE_STOP = 3;
  ORDER_TYPE_STOP_LIMIT = 4;
}

enum OrderStatus {
  ORDER_STATUS_UNSPECIFIED = 0;
  ORDER_STATUS_PENDING = 1;
  ORDER_STATUS_OPEN = 2;
  ORDER_STATUS_FILLED = 3;
  ORDER_STATUS_PARTIALLY_FILLED = 4;
  ORDER_STATUS_CANCELED = 5;
  ORDER_STATUS_REJECTED = 6;
}

enum TimeInForce {
  TIME_IN_FORCE_UNSPECIFIED = 0;
  TIME_IN_FORCE_DAY = 1;
  TIME_IN_FORCE_GTC = 2;
  TIME_IN_FORCE_IOC = 3;
  TIME_IN_FORCE_FOK = 4;
}
```

## Data Model

### Core Entities

#### Order
The central entity representing a trading order.

```java
@Entity
@Table(name = "orders")
public class Order {
    @Id
    private String id;
    
    private String accountId;
    private String symbol;
    
    @Enumerated(EnumType.STRING)
    private OrderSide side;
    
    @Enumerated(EnumType.STRING)
    private OrderType type;
    
    @Enumerated(EnumType.STRING)
    private OrderStatus status;
    
    private BigDecimal quantity;
    private BigDecimal filledQuantity;
    private BigDecimal price;
    private BigDecimal stopPrice;
    private BigDecimal averageExecutionPrice;
    
    @Enumerated(EnumType.STRING)
    private TimeInForce timeInForce;
    
    private String strategyId;
    private String brokerOrderId;
    private String rejectionReason;
    private String notes;
    
    @CreatedDate
    private Instant createdAt;
    
    @LastModifiedDate
    private Instant updatedAt;
    
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<ExecutionReport> executionReports = new ArrayList<>();
    
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<OrderEvent> events = new ArrayList<>();
}
```

#### ExecutionReport
Represents an execution report received from a broker.

```java
@Entity
@Table(name = "execution_reports")
public class ExecutionReport {
    @Id
    private String id;
    
    @ManyToOne
    @JoinColumn(name = "order_id")
    private Order order;
    
    private String executionId;
    private BigDecimal quantity;
    private BigDecimal price;
    private Instant timestamp;
    private BigDecimal fees;
    private String venue;
}
```

#### OrderEvent
Represents an event in the order lifecycle for event sourcing.

```java
@Entity
@Table(name = "order_events")
public class OrderEvent {
    @Id
    private String id;
    
    @ManyToOne
    @JoinColumn(name = "order_id")
    private Order order;
    
    @Enumerated(EnumType.STRING)
    private OrderEventType type;
    
    @Column(columnDefinition = "jsonb")
    private String payload;
    
    private String userId;
    private Instant timestamp;
    private String correlationId;
}
```

## Database Schema (CQRS Pattern)

### Write Model (Command Side)

#### orders Table
```sql
CREATE TABLE orders (
    id VARCHAR(36) PRIMARY KEY,
    account_id VARCHAR(36) NOT NULL,
    symbol VARCHAR(20) NOT NULL,
    side VARCHAR(10) NOT NULL,
    type VARCHAR(15) NOT NULL,
    status VARCHAR(20) NOT NULL,
    quantity DECIMAL(20, 8) NOT NULL,
    filled_quantity DECIMAL(20, 8) NOT NULL DEFAULT 0,
    price DECIMAL(20, 8),
    stop_price DECIMAL(20, 8),
    average_execution_price DECIMAL(20, 8),
    time_in_force VARCHAR(10) NOT NULL,
    strategy_id VARCHAR(36),
    broker_order_id VARCHAR(100),
    rejection_reason TEXT,
    notes TEXT,
    created_at TIMESTAMP NOT NULL,
    updated_at TIMESTAMP NOT NULL,
    
    CONSTRAINT fk_account FOREIGN KEY (account_id) REFERENCES accounts(id),
    CONSTRAINT chk_order_type CHECK (
        (type = 'MARKET' AND price IS NULL) OR
        (type = 'LIMIT' AND price IS NOT NULL) OR
        (type = 'STOP' AND stop_price IS NOT NULL) OR
        (type = 'STOP_LIMIT' AND price IS NOT NULL AND stop_price IS NOT NULL)
    )
);

CREATE INDEX idx_orders_account_id ON orders(account_id);
CREATE INDEX idx_orders_symbol ON orders(symbol);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_created_at ON orders(created_at);
```

#### execution_reports Table
```sql
CREATE TABLE execution_reports (
    id VARCHAR(36) PRIMARY KEY,
    order_id VARCHAR(36) NOT NULL,
    execution_id VARCHAR(100) NOT NULL,
    quantity DECIMAL(20, 8) NOT NULL,
    price DECIMAL(20, 8) NOT NULL,
    timestamp TIMESTAMP NOT NULL,
    fees DECIMAL(20, 8) NOT NULL DEFAULT 0,
    venue VARCHAR(50),
    
    CONSTRAINT fk_order FOREIGN KEY (order_id) REFERENCES orders(id)
);

CREATE INDEX idx_execution_reports_order_id ON execution_reports(order_id);
CREATE INDEX idx_execution_reports_timestamp ON execution_reports(timestamp);
```

#### order_events Table (Event Sourcing)
```sql
CREATE TABLE order_events (
    id VARCHAR(36) PRIMARY KEY,
    order_id VARCHAR(36) NOT NULL,
    type VARCHAR(50) NOT NULL,
    payload JSONB NOT NULL,
    user_id VARCHAR(36),
    timestamp TIMESTAMP NOT NULL,
    correlation_id VARCHAR(36),
    
    CONSTRAINT fk_order FOREIGN KEY (order_id) REFERENCES orders(id)
);

CREATE INDEX idx_order_events_order_id ON order_events(order_id);
CREATE INDEX idx_order_events_timestamp ON order_events(timestamp);
CREATE INDEX idx_order_events_correlation_id ON order_events(correlation_id);
```

### Read Model (Query Side)

#### order_summary View
```sql
CREATE VIEW order_summary AS
SELECT 
    o.id,
    o.account_id,
    o.symbol,
    o.side,
    o.type,
    o.status,
    o.quantity,
    o.filled_quantity,
    o.price,
    o.stop_price,
    o.average_execution_price,
    o.time_in_force,
    o.strategy_id,
    o.broker_order_id,
    o.created_at,
    o.updated_at,
    a.name as account_name,
    i.name as instrument_name,
    i.asset_class,
    i.currency,
    (CASE 
        WHEN o.side = 'BUY' THEN o.filled_quantity * o.average_execution_price
        ELSE -o.filled_quantity * o.average_execution_price
    END) as notional_value,
    (SELECT SUM(fees) FROM execution_reports WHERE order_id = o.id) as total_fees
FROM 
    orders o
JOIN 
    accounts a ON o.account_id = a.id
JOIN 
    instruments i ON o.symbol = i.symbol;
```

#### daily_order_statistics Materialized View
```sql
CREATE MATERIALIZED VIEW daily_order_statistics AS
SELECT 
    DATE_TRUNC('day', created_at) as trade_date,
    account_id,
    COUNT(*) as total_orders,
    SUM(CASE WHEN status = 'FILLED' THEN 1 ELSE 0 END) as filled_orders,
    SUM(CASE WHEN status = 'CANCELED' THEN 1 ELSE 0 END) as canceled_orders,
    SUM(CASE WHEN status = 'REJECTED' THEN 1 ELSE 0 END) as rejected_orders,
    SUM(CASE WHEN side = 'BUY' AND status = 'FILLED' THEN filled_quantity * average_execution_price ELSE 0 END) as buy_notional,
    SUM(CASE WHEN side = 'SELL' AND status = 'FILLED' THEN filled_quantity * average_execution_price ELSE 0 END) as sell_notional
FROM 
    orders
GROUP BY 
    DATE_TRUNC('day', created_at), account_id;

CREATE UNIQUE INDEX idx_daily_order_statistics_date_account ON daily_order_statistics(trade_date, account_id);
```

## Project Plan

### Phase 1: Foundation (Weeks 1-2)

#### Week 1: Setup and Core Domain Model
- Set up project structure and build system
- Implement core domain model (Order, ExecutionReport, etc.)
- Create database schema with migrations
- Implement basic repository layer
- Set up testing framework and write initial tests

#### Week 2: Basic API Implementation
- Implement REST API endpoints for order management
- Implement gRPC service definitions
- Create service layer with basic business logic
- Implement validation rules for orders
- Write integration tests for API endpoints

### Phase 2: Core Functionality (Weeks 3-4)

#### Week 3: Order Lifecycle Management
- Implement order state machine
- Create event sourcing infrastructure
- Implement order creation and validation
- Add order modification and cancellation logic
- Implement execution report processing

#### Week 4: Integration with Other Services
- Implement integration with Broker Integration Service
- Add pre-trade risk check integration
- Implement position update notifications
- Create order event publishing mechanism
- Write integration tests for service interactions

### Phase 3: Advanced Features (Weeks 5-6)

#### Week 5: Performance and Scalability
- Implement CQRS pattern with read models
- Add caching for frequently accessed data
- Optimize database queries and indexes
- Implement batch processing for high-volume scenarios
- Conduct performance testing and optimization

#### Week 6: Monitoring and Resilience
- Add comprehensive logging and metrics
- Implement circuit breakers for external dependencies
- Create health check endpoints
- Add retry mechanisms for transient failures
- Implement rate limiting for API endpoints

### Phase 4: Finalization (Weeks 7-8)

#### Week 7: Security and Compliance
- Implement authentication and authorization
- Add audit logging for compliance
- Create data retention policies
- Implement data encryption for sensitive information
- Conduct security review and testing

#### Week 8: Documentation and Deployment
- Create comprehensive API documentation
- Write operational runbooks
- Prepare deployment configurations
- Conduct final testing and bug fixes
- Prepare for production deployment

### Milestones and Deliverables

1. **End of Week 2**: Basic API and domain model implemented
2. **End of Week 4**: Complete order lifecycle management with service integration
3. **End of Week 6**: Performance-optimized service with monitoring
4. **End of Week 8**: Production-ready service with documentation

### Risk Management

1. **Integration Complexity**: Start integration testing early with mock services
2. **Performance Bottlenecks**: Conduct regular performance testing throughout development
3. **Data Consistency**: Implement robust transaction management and event sourcing
4. **Security Vulnerabilities**: Regular security reviews and automated scanning
5. **Regulatory Compliance**: Involve compliance team early in the design process