# RLS (Reference Lab Systems) - Complete System Architecture

## Overview

The RLS (Reference Lab Systems) ecosystem is a comprehensive laboratory order management platform built on a microservices architecture. It consists of a modern Angular frontend monorepo and three .NET 8.0 backend APIs that orchestrate laboratory order workflows, from order creation through result delivery.

---

## System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                   FRONTEND                                       │
│  ┌───────────────────────────────────────────────────────────────────────────┐  │
│  │                     mcs-products-mono-ui (Nx Monorepo)                    │  │
│  │  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────────────────┐  │  │
│  │  │ PEARLS Apps     │ │ Product Catalog │ │ Shared Libraries            │  │  │
│  │  │ - Accession     │ │ - Nexus App     │ │ - Core (Auth, State)        │  │  │
│  │  │ - Specimen      │ │                 │ │ - Data Access (API clients) │  │  │
│  │  │ - Sendouts      │ │                 │ │ - UI (PrimeNG wrappers)     │  │  │
│  │  │ - Order Viewer  │ │                 │ │ - Pattern Library           │  │  │
│  │  └─────────────────┘ └─────────────────┘ └─────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        │ HTTPS/REST (JWT Bearer)
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              BACKEND SERVICES                                    │
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │                  rls-orders-cnsmr-api (Consumer API)                     │    │
│  │   Primary entry point for UI - aggregates data from multiple services   │    │
│  │   Features: Order CRUD, Labels, Documents, Reports, Search              │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                        │                                         │
│                   ┌────────────────────┼────────────────────┐                   │
│                   │                    │                    │                   │
│                   ▼                    ▼                    ▼                   │
│  ┌──────────────────────┐ ┌─────────────────────┐ ┌─────────────────────────┐   │
│  │ Lab Test Catalog API │ │ Dataverse CRM API   │ │ DocGen API              │   │
│  │ (Test definitions)   │ │ (Account info)      │ │ (Label/Doc generation)  │   │
│  └──────────────────────┘ └─────────────────────┘ └─────────────────────────┘   │
│                                        │                                         │
│                                        ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │                 rls-orders-orch-api (Orchestration API)                  │    │
│  │   Business logic orchestration - coordinates data flow and events        │    │
│  │   Features: Order orchestration, Event publishing, External integrations │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                           │                        │                             │
│                           ▼                        ▼                             │
│  ┌─────────────────────────────────┐    ┌─────────────────────────────────┐     │
│  │     Google Cloud Pub/Sub        │    │        GCS Storage              │     │
│  │   (Order change events)         │    │   (Results PDFs, Documents)     │     │
│  └─────────────────────────────────┘    └─────────────────────────────────┘     │
│                                        │                                         │
│                                        ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │                    rls-orders-data-api (Data API)                        │    │
│  │   Data persistence layer - direct MongoDB operations                     │    │
│  │   Features: CRUD, Search, Event logging, Optimistic locking             │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                        │                                         │
│                                        ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │                           MongoDB (pearls)                               │    │
│  │   Collections: orders, order-events                                      │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 1. Frontend: mcs-products-mono-ui

### Technology Stack
| Technology | Version | Purpose |
|------------|---------|---------|
| Angular | 19.2.14 | Core framework (zoneless change detection) |
| Nx | 20.8.2 | Monorepo management |
| NgRx Signals | 19.0.1 | State management |
| PrimeNG | 19.0.6 | UI component library |
| RxJS | 7.8.0 | Reactive programming |
| TypeScript | 5.x | Type-safe JavaScript |

### Workspace Structure

```
mcs-products-mono-ui/
├── apps/
│   ├── rls-accession-portal-ui     # Main PEARLS portal
│   ├── rls-specimen-accessioning   # Specimen processing
│   ├── rls-sendouts-ui             # Send-out lab management
│   ├── rls-order-system-viewer-app # Order viewing
│   ├── rls-core-home-app           # Home/core module
│   ├── prodcat-nexus-app           # Product catalog
│   └── *-e2e                       # E2E test apps (Cypress)
│
├── libs/
│   ├── pearls/                     # Domain-specific libraries
│   │   ├── auto-accession/         # Barcode/automated specimen entry
│   │   ├── specimen-accessioning/  # Complex state with middleware
│   │   ├── sendouts/               # Outbound lab processing
│   │   ├── core-home/              # Home page features
│   │   └── order-system-viewer/    # Order viewing features
│   │
│   ├── product-catalog/            # Product management
│   │   ├── product-details/
│   │   ├── product-viewer/
│   │   └── common/
│   │
│   └── shared/                     # Cross-cutting concerns
│       ├── core/                   # Auth, state, interceptors
│       ├── data-access/            # API service clients
│       ├── ui/                     # Design system (PrimeNG wrappers)
│       ├── pattern-library/        # Theme, styles (SCSS)
│       ├── utils/                  # Helper functions
│       └── testing/                # Test utilities
│
├── nx.json                         # Nx workspace configuration
├── package.json                    # Dependencies
└── tsconfig.base.json              # TypeScript configuration
```

### State Management: NgRx Signals

The UI uses modern signal-based state management with `@ngrx/signals`:

```typescript
// Example: User Store (libs/shared/core)
export const UserStore = signalStore(
  { providedIn: 'root' },
  withState<UserState>(initialState),
  withMethods((store) => ({
    setUser(user: User): void { patchState(store, { user }); }
  }))
);

// Complex Example: Orders Store (libs/pearls/specimen-accessioning)
// Uses middleware pattern for side effects:
// - order.middleware, tests.middleware, save.middleware
// State includes: order, selectedTest, changes tracking, UI state
```

### Authentication & Authorization

| Component | Implementation |
|-----------|----------------|
| Protocol | OAuth2/OIDC |
| Library | angular-oauth2-oidc v17.0.2 |
| Flow | Authorization Code with PKCE |
| Provider | Azure AD (Mayo Clinic tenant) |
| Token Refresh | Automatic silent refresh |
| Cross-tab | Custom CrossTabStorage |

**User Model** (from identity claims):
- firstName, lastName, email (upn), lanId, personId
- Roles from access token via IdentityService
- Profile photo: `https://quarterly.mayo.edu/qtphotos/{personId}.jpg`

### Routing Pattern

```typescript
// Lazy-loaded feature modules with role-based guards
export const appRoutes: Route[] = [
  ...authRoutes,
  {
    path: 'search',
    loadComponent: () => import('@rls/auto-accession/features/search-feat'),
    canActivate: [AuthGuard],
    data: {
      requiredRoles: [Roles.SPECIMENS_SEARCH],
      headerTitle: 'Automated Accessioning'
    }
  }
];
```

### API Integration Pattern

```typescript
// Service pattern (libs/shared/data-access)
@Injectable({ providedIn: 'root' })
export class OrdersService {
  private apiUrl = getFullApiUrl(
    this.configService,
    'rlsOrdersConsumerApi',    // Config key
    EndpointUrl.ORDERS_V1      // API version
  );

  getOrder(orderId: string): Observable<Order> {
    return this.http.get<Order>(`${this.apiUrl}/${orderId}`);
  }
}
```

### UI Component Library

| Component | Source | Usage |
|-----------|--------|-------|
| Core Components | PrimeNG 19.0.6 | Tables, Forms, Dialogs |
| Icons | primeicons + @mayoclinic/icons | Icon system |
| Design System | libs/shared/ui/design-system | PrimeNG wrappers |
| Theme | pearls-theme.css | Custom Mayo styling |

**Design System Components:**
- FormWrapper, Input, Select, MultiSelect, DatePicker
- PrimeMcsTable (advanced data table)
- Drawer, ConfirmDialog (overlays)
- BaseHeaderComponent, FooterComponent, SidebarComponent

---

## 2. Backend: rls-orders-cnsmr-api (Consumer API)

### Purpose
Primary entry point for the UI. Aggregates data from multiple backend services and provides a unified API for frontend consumption.

### Technology Stack
| Technology | Version | Purpose |
|------------|---------|---------|
| .NET | 8.0 | Runtime |
| ASP.NET Core | 8.0 | Web API framework |
| QuestPDF | Community | PDF label generation |
| FluentValidation | 12.x | Request validation |
| AutoMapper | 12.x | Object mapping |
| Serilog | 4.x | Structured logging |

### Project Structure

```
Mayo.MCS.RLS.OrdersConsumer.Api/          # Controllers & endpoints
Mayo.MCS.RLS.OrdersConsumer.Api.Model/    # DTOs & request/response models
Mayo.MCS.RLS.OrdersConsumer.BLL/          # Business logic services
Mayo.MCS.RLS.OrdersConsumer.DAL/          # HTTP repositories
Mayo.MCS.RLS.OrdersConsumer.Model/        # Domain models & exceptions
```

### API Endpoints

| Controller | Base Route | Purpose |
|------------|------------|---------|
| OrdersController | `/api/v1/orders` | Order CRUD, search, notes |
| DocumentsController | `/api/v1/documents` | Document generation |
| ReportsController | `/api/v1/reports` | Result PDF signed URLs |
| LabelsController | `/api/v1/labels` | PDF label generation |
| TestsController | `/api/v1/tests` | Test catalog, AOE prompts |
| AccountsController | `/api/v1/accounts` | Client account info |

**Key Order Endpoints:**
```
GET    /api/v1/orders/{orderId}           # Get by internal ID
GET    /api/v1/orders?orderIdentifier=    # Search by identifier
POST   /api/v1/orders/search              # Advanced search with filters
POST   /api/v1/orders                     # Create manual order
PATCH  /api/v1/orders/{orderId}           # Update order
POST   /api/v1/orders/{orderId}/notes     # Add order note
```

### Service Layer

| Service | Purpose |
|---------|---------|
| IOrderService | Order management (CRUD, search, notes) |
| ITestService | Test information and AOE prompts |
| ILabelService | PDF label generation via QuestPDF |
| IDocumentService | Document export (PDF/CSV) |
| IReportService | Report retrieval with signed URLs |
| IDataverseService | CRM account information |
| ILtcService | Lab Test Catalog integration |

### External Service Integration

| Repository | Target Service | Purpose |
|------------|----------------|---------|
| OrdersRepository | rls-orders-orch-api | Order CRUD operations |
| LtcRepository | Lab Test Catalog API | Test definitions, AOE |
| DataverseRepository | Dataverse CRM | Account info, timezones |
| DocgenRepository | DocGen API | Label/document generation |

### Authorization Roles

| Role | Access Level |
|------|--------------|
| `order.read`, `ordersconsumer.order.read` | Read orders |
| `order.write`, `ordersconsumer.order.write` | Modify orders |
| `label.read`, `ordersconsumer.label.read` | Access labels |
| `account.read`, `ordersconsumer.account.read` | Read CRM accounts |
| `ordersconsumer.document.read` | Access documents |
| `pearls.results.list` | View lab results |

---

## 3. Backend: rls-orders-orch-api (Orchestration API)

### Purpose
Business logic orchestration layer. Coordinates data flow between services and publishes events for downstream systems.

### Technology Stack
| Technology | Version | Purpose |
|------------|---------|---------|
| .NET | 8.0 | Runtime |
| ASP.NET Core | 8.0 | Web API framework |
| Google Cloud Pub/Sub | - | Event messaging |
| Google Cloud Storage | - | Document storage |
| FluentValidation | - | Request validation |
| AutoMapper | - | Object mapping |

### Project Structure

```
Mayo.MCS.RLS.Orders.Orch.Api/             # Controllers & endpoints
Mayo.MCS.RLS.Orders.Orch.BLL/             # Business logic layer
├── OrdersBll.cs                          # Order orchestration
├── InterfacedOrdersBll.cs                # EDI/HL7 message processing
├── SpecimensBll.cs                       # Specimen logic
├── Helpers/
│   ├── Order/Topic/OrderTopicHelper      # Pub/Sub message creation
│   └── PopulateOrderModelHelper          # Model enrichment
└── Validators/                           # FluentValidation rules
Mayo.MCS.RLS.Orders.Orch.DAL/             # Data access services
├── OrdersDataService                     # → Orders Data API
├── ProdCatService                        # → Product Catalog API
├── DataverseService                      # → CRM/Dataverse
├── OrderTopicDataService                 # → GCP Pub/Sub
└── Helpers/ConfigurationHelper           # Config management
```

### API Endpoints

**Orders Controller** (`/api/v1/orders`):
```
GET    /{orderUid}                  # Retrieve order details
POST   /search                      # Search orders (paged)
POST   /                            # Create manual order
PUT    /{orderUid}                  # Update order
PUT    /{orderUid}/patient          # Update patient demographics
PUT    /{orderUid}/accountnumber    # Update account number
POST   /{orderUid}/notes/{noteType} # Add notes
```

**InterfacedOrders Controller** (`/api/v1/orders/interfaced`):
```
POST   /                            # Upsert interfaced orders (NW messages)
PUT    /cancel                      # Cancel tests (CA messages)
POST   /results                     # Post test results (ORU messages)
```

### Event Publishing (Google Cloud Pub/Sub)

| Event Type | Trigger |
|------------|---------|
| Order Created | New order saved |
| Order Updated | Order modified |
| Test Added | New test added to order |
| Test Canceled | Test removed from order |
| Results Received | Lab results processed |
| Patient Updated | Demographics changed |
| Notes Added | New notes attached |

**Message Flow:**
```
OrdersBll.CreateOrder()
    → OrdersDataService.CreateOrder()      # Save to MongoDB
    → PopulateOrderModelHelper()            # Enrich with related data
    → OrderTopicHelper.CreateMessage()      # Build Pub/Sub message
    → OrderTopicDataService.Publish()       # Send to topic
```

### External Service Integration

| Service | Type | Purpose |
|---------|------|---------|
| Orders Data API | REST | CRUD orders (MongoDB) |
| Product Catalog | REST | Test validation, metadata |
| Dataverse/CRM | REST | Account timezone info |
| GCP Pub/Sub | Event | Order change notifications |
| GCP Storage | Cloud | Results/PDF storage |

---

## 4. Backend: rls-orders-data-api (Data API)

### Purpose
Data persistence layer. Direct MongoDB operations with optimistic locking and event auditing.

### Technology Stack
| Technology | Version | Purpose |
|------------|---------|---------|
| .NET | 8.0 | Runtime |
| ASP.NET Core | 8.0 | Web API framework |
| MongoDB Driver | 3.5.2 | Database access |
| FluentValidation | 12.1.1 | Request validation |

### Project Structure

```
Mayo.MCS.RLS.Orders.Data.Api/             # Controllers & endpoints
Mayo.MCS.RLS.Orders.Data.Api.Model/       # Data Transfer Objects
Mayo.MCS.RLS.Orders.Data.BLL/             # Business logic layer
Mayo.MCS.RLS.Orders.Data.DAL/             # Data access layer
├── OrderDataService                      # Service abstraction
└── OrdersCollectionHelper                # Direct MongoDB operations
Mayo.MCS.RLS.Orders.Data.Model/           # Shared models & exceptions
```

### API Endpoints

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/{orderUid:guid}` | Get order by internal UID |
| GET | `/{orderNumber}` | Get order by PEARLS order number |
| GET | `/?orderNumber=X&sourceSystem=Y` | Get by ordering system |
| POST | `/` | Create new order |
| PUT | `/{orderUid}` | Update existing order |
| POST | `/search` | Search with pagination & filtering |

### MongoDB Schema

**Database:** `pearls`

**Collections:**
| Collection | Purpose |
|------------|---------|
| `orders` | Main order documents |
| `order-events` | Event audit trail |

**Serialization Conventions:**
- GUIDs → BSON String type
- CamelCase element naming
- Enums → String representation
- Ignore null values
- Ignore extra elements
- TLS 1.2 SSL encryption

### CRUD Operations

| Operation | Transactions | Concurrency |
|-----------|--------------|-------------|
| Create | ✅ Multi-document | None |
| Read | ❌ | N/A |
| Update | ✅ Multi-document | ✅ Optimistic locking |
| Search | ❌ | N/A (Pagination) |

**Optimistic Locking:**
```csharp
// RevisionNumber incremented on each update
// 409 Conflict returned if revision mismatch detected
if (existingOrder.RevisionNumber != request.RevisionNumber)
    throw new OptimisticLockingException();
```

---

## 5. Cross-Cutting Concerns

### Authentication & Authorization

All services use Azure AD with JWT Bearer tokens:

```
┌─────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   User      │───▶│   Azure AD      │───▶│ JWT Bearer      │
│   (Browser) │    │   (OAuth2/OIDC) │    │ Token           │
└─────────────┘    └─────────────────┘    └─────────────────┘
                                                   │
       ┌───────────────────────────────────────────┼───────┐
       │                                           ▼       │
       │  ┌─────────────────────────────────────────────┐  │
       │  │         All Backend APIs                    │  │
       │  │  - Validate JWT signature & audience        │  │
       │  │  - Extract roles from token                 │  │
       │  │  - Apply [AuthorizeRoles] attributes        │  │
       │  └─────────────────────────────────────────────┘  │
       └───────────────────────────────────────────────────┘
```

### Error Handling

**Custom Exception Hierarchy:**
```
BaseOdnException
├── NotFoundException (404)
├── BadRequestException (400)
├── UnauthorizedException (401)
├── ForbiddenException (403)
├── OptimisticLockingException (409)
└── ServerException (500)

BaseOdnPassthroughException
└── *ApiException (wraps downstream HTTP errors)
```

### Logging & Monitoring

| Component | Implementation |
|-----------|----------------|
| Structured Logging | Serilog |
| Cloud Logging | Google Cloud Logging |
| APM | AppDynamics |
| Entry/Exit Logging | OdnExceptionFilter |
| Sensitive Data Masking | Automatic (SSN, DOB, etc.) |

### Configuration Management

**Sources (Priority Order):**
1. `appsettings.json` - Static defaults
2. `appsettings.{Environment}.json` - Environment overrides
3. Google Cloud Metadata Service - Runtime secrets
4. Environment Variables - Container runtime

---

## 6. Data Flow Examples

### Create Manual Order

```
┌─────────────────┐
│   UI (Angular)  │
│   OrdersService │
└────────┬────────┘
         │ POST /api/v1/orders
         ▼
┌─────────────────────┐
│   Consumer API      │
│   OrdersController  │
│   → OrderService    │
│   → OrdersRepository│
└────────┬────────────┘
         │ POST /api/v1/orders
         ▼
┌─────────────────────┐
│   Orchestration API │
│   OrdersController  │
│   → OrdersBll       │
│   → Validation      │
│   → OrdersDataSvc   │
└────────┬────────────┘
         │ POST /api/v1/orders
         ▼
┌─────────────────────┐
│   Data API          │
│   OrdersController  │
│   → OrderDataService│
│   → MongoDB Insert  │
│   → Event Logging   │
└────────┬────────────┘
         │ Response
         ▼
┌─────────────────────┐
│   Orchestration API │
│   → Enrich Model    │
│   → Pub/Sub Publish │
└────────┬────────────┘
         │ Response
         ▼
┌─────────────────────┐
│   Consumer API      │
│   → Return to UI    │
└─────────────────────┘
```

### Search Orders

```
┌─────────────────┐
│   UI (Angular)  │
│   Search Form   │
└────────┬────────┘
         │ POST /api/v1/orders/search
         ▼
┌─────────────────────┐
│   Consumer API      │
│   → Build Query     │
│   → OrdersRepository│
└────────┬────────────┘
         │ POST /api/v1/orders/search
         ▼
┌─────────────────────┐
│   Orchestration API │
│   → OrdersDataSvc   │
└────────┬────────────┘
         │ POST /api/v1/orders/search
         ▼
┌─────────────────────┐
│   Data API          │
│   → MongoDB Query   │
│   → Pagination      │
│   → Return Results  │
└─────────────────────┘
```

---

## 7. Deployment Architecture

### Environments

| Environment | UI Domain | API Domain |
|-------------|-----------|------------|
| Development | rls-ui-dev.mcs-rls-n.caf.mccapp.com | rls-*-dev.mcs-rls-n.caf.mccapp.com |
| Test | rls-ui-test.mcs-rls-n.caf.mccapp.com | rls-*-test.mcs-rls-n.caf.mccapp.com |
| Staging | rls-ui-stage.mcs-rls-p.caf.mccapp.com | rls-*-stage.mcs-rls-p.caf.mccapp.com |
| Production | Google Cloud Run URLs | Google Cloud Run URLs |

### Container Infrastructure

```
┌─────────────────────────────────────────────────────────────┐
│                    Google Cloud Run                          │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐     │ │
│  │  │ Consumer API │ │ Orch API     │ │ Data API     │     │ │
│  │  │ (Container)  │ │ (Container)  │ │ (Container)  │     │ │
│  │  └──────────────┘ └──────────────┘ └──────────────┘     │ │
│  │        ▲                ▲                ▲               │ │
│  │        │                │                │               │ │
│  │  ┌─────┴────────────────┴────────────────┴─────┐         │ │
│  │  │          AppDynamics APM Agent              │         │ │
│  │  └─────────────────────────────────────────────┘         │ │
│  └─────────────────────────────────────────────────────────┘ │
│                              │                                │
│                              ▼                                │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │                    MongoDB Atlas                         │ │
│  │                    Database: pearls                      │ │
│  └─────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Docker Base Images

| Service | Base Image |
|---------|------------|
| All .NET APIs | mcr.microsoft.com/dotnet/aspnet:8.0-alpine3.22-amd64 |
| UI (build) | node:lts |
| UI (serve) | nginx:alpine |

---

## 8. Key Architectural Patterns

| Pattern | Implementation |
|---------|----------------|
| **Microservices** | Separate APIs for orchestration, data, consumption |
| **API Gateway** | Consumer API aggregates backend services |
| **Event-Driven** | Pub/Sub for order change notifications |
| **Repository Pattern** | Data services abstract external APIs |
| **CQRS-lite** | Search operations separated from mutations |
| **Optimistic Locking** | RevisionNumber for concurrent update handling |
| **Feature Toggles** | Enable/disable features without deployment |
| **Layered Architecture** | API → BLL → DAL separation |
| **Signal-based State** | NgRx Signals for frontend state |
| **Lazy Loading** | Angular route-based code splitting |

---

## 9. Security Considerations

| Area | Implementation |
|------|----------------|
| Authentication | Azure AD OAuth2/OIDC with PKCE |
| Authorization | Role-based access control (RBAC) |
| Transport | HTTPS/TLS 1.2+ everywhere |
| Token Management | Automatic silent refresh, secure storage |
| Sensitive Data | Automatic masking in logs (PII fields) |
| CORS | Configured per environment |
| Headers | X-Frame-Options, secure cookie flags |
| Secrets | Google Cloud Metadata Service |
