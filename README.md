# RetryRhythm - Temporal Workflow Orchestration with NestJS

A production-ready NestJS application demonstrating workflow orchestration using Temporal.io. This implementation showcases a complete order processing system with fault tolerance, compensation logic, and real-time progress tracking.

## Overview

RetryRhythm demonstrates enterprise patterns for building reliable, long-running business processes:

- **Durable Execution**: Workflows that survive failures and continue execution
- **Automatic Retries**: Built-in retry policies for failed activities
- **Compensation Logic**: Automatic rollback on failures (refunds, inventory release)
- **Real-time Tracking**: Signal and query patterns for workflow interaction
- **Type Safety**: Full TypeScript support with strict typing

## Tech Stack

- **Runtime**: Node.js 18+
- **Framework**: NestJS
- **Workflow Engine**: Temporal.io
- **Package**: nestjs-temporal-core
- **Database**: PostgreSQL
- **API**: REST with Swagger/OpenAPI
- **Container**: Docker & Docker Compose

## Architecture

```
src/
├── workflows/              # Pure Temporal workflow definitions
│   ├── order.workflow.ts   # Order processing workflow
│   └── index.ts
├── activities/            # NestJS services with @Activity decorator
│   ├── payment.activities.ts     # Payment processing & refunds
│   ├── inventory.activities.ts   # Inventory management
│   ├── email.activities.ts       # Email notifications
│   └── notification.activities.ts
├── services/              # Business logic layer
│   └── order.service.ts
├── controllers/           # REST API endpoints
│   └── order.controller.ts
└── app.module.ts         # TemporalModule configuration
```

## Features

- Complete order processing workflow with 8 steps
- Automatic retry with exponential backoff
- Compensation/rollback on failures
- Real-time order progress tracking
- Signal-based order cancellation
- REST API with Swagger documentation
- Docker Compose for local development

## Order Workflow Lifecycle

The order processing workflow executes 8 sequential steps:

```
Order Received → Validate → Check Inventory → Reserve Inventory
  → Process Payment → Confirm Payment → Send Email → Prepare Shipment → Ship
```

### Failure Handling

**Inventory Unavailable**:

- Release any partial reservations
- Update order status to FAILED
- Log failure reason

**Payment Failure**:

- Refund any processed payments
- Release inventory reservations
- Update order status to FAILED
- Send failure notification

**Order Cancellation** (via Signal):

- Refund payment (if processed)
- Release inventory reservations
- Update order status to CANCELLED

## API Endpoints

| Method | Endpoint                     | Description        |
| ------ | ---------------------------- | ------------------ |
| POST   | /orders                      | Create new order   |
| POST   | /orders/demo                 | Create demo order  |
| GET    | /orders/:workflowId/status   | Get order status   |
| GET    | /orders/:workflowId/progress | Get progress       |
| DELETE | /orders/:workflowId          | Cancel order       |
| PATCH  | /orders/:workflowId          | Update order       |
| GET    | /orders                      | List active orders |

## Getting Started

### Prerequisites

- Node.js 18+
- Docker & Docker Compose
- npm

### Installation

```bash
npm install
```

### Start Temporal Server

```bash
npm run temporal:up
```

This starts:

- Temporal Server: `localhost:7233`
- Temporal Web UI: http://localhost:8088
- PostgreSQL: `localhost:5433`

### Configure Environment

```bash
cp .env.example .env
```

Default configuration:

```env
NODE_ENV=development
PORT=3232
TEMPORAL_ADDRESS=localhost:7233
TEMPORAL_NAMESPACE=default
TEMPORAL_TASK_QUEUE=order-processing
```

### Start Application

```bash
npm run start:dev
```

### Access Points

- REST API: http://localhost:3232
- Swagger UI: http://localhost:3232/api
- Temporal Web UI: http://localhost:8088

## Example Usage

### Create Demo Order

```bash
curl -X POST http://localhost:3232/orders/demo
```

Response:

```json
{
  "orderId": "ORD-1699564829123-ABC123XYZ",
  "workflowId": "order-workflow-ORD-1699564829123-ABC123XYZ",
  "message": "Demo order created successfully"
}
```

### Check Order Status

```bash
curl http://localhost:3232/orders/order-workflow-ORD-xxx/status
```

Response:

```json
{
  "orderId": "ORD-xxx",
  "status": "PAYMENT_CONFIRMED",
  "currentStep": "payment_confirmation",
  "paymentId": "pay_xxx",
  "percentComplete": 62.5
}
```

### Cancel Order

```bash
curl -X DELETE http://localhost:3232/orders/order-workflow-ORD-xxx \
  -H "Content-Type: application/json" \
  -d '{"reason": "Customer requested cancellation"}'
```

## Activity Timeouts & Retries

| Activity     | Timeout   | Max Retries | Policy              |
| ------------ | --------- | ----------- | ------------------- |
| Payment      | 5 minutes | 5           | Exponential backoff |
| Inventory    | 1 minute  | 3           | Exponential backoff |
| Email        | 2 minutes | 3           | Exponential backoff |
| Notification | 2 minutes | 3           | Exponential backoff |

## Development Scripts

```bash
npm run start:dev          # Hot reload development
npm run start:debug        # Debug mode
npm run build              # TypeScript compilation
npm run temporal:up        # Start Temporal
npm run temporal:down      # Stop Temporal
npm run temporal:logs      # View logs
npm run lint               # Lint code
npm run format             # Format code
npm test                   # Run tests
```

## Temporal Integration

```typescript
TemporalModule.registerAsync({
  connection: {
    address: "localhost:7233",
    namespace: "default",
  },
  taskQueue: "order-processing",
  worker: {
    workflowsPath: require.resolve("./workflows"),
    activityClasses: [
      PaymentActivityService,
      InventoryActivityService,
      EmailActivityService,
    ],
    autoStart: true,
  },
});
```

## Why Temporal?

- **Reliability**: Automatic retry and persistence
- **Observability**: Complete workflow history
- **Scalability**: Distributed workflow execution
- **Debugging**: Time-travel debugging with replay

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
