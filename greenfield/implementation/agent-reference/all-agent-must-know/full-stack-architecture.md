# PEaRLS Specimen Accessioning - Full Stack Architecture  
  
## Overview  
  
**PEaRLS** (Platform EnAbled Reference Laboratory System) Specimen Accessioning is a comprehensive laboratory order management system for Mayo Clinic Reference Laboratory Services (RLS). This document provides a detailed architecture reference for the full stack, from the Angular UI through the API layers to the MongoDB database.  
  
---  
  
## System Architecture Diagram  
  
```  
┌─────────────────────────────────────────────────────────────────────────────────────┐  
│                                  PRESENTATION LAYER                                  │  
│                                                                                       │  
│  ┌─────────────────────────────────────────────────────────────────────────────────┐ │  
│  │                     mcs-products-mono-ui (Angular 18+, Nx Monorepo)             │ │  
│  │  ┌──────────────────────────────────────────────────────────────────────────┐   │ │  
│  │  │    rls-specimen-accessioning (Angular SPA)                               │   │ │  
│  │  │    ├── apps/pearls/rls-specimen-accessioning (Application Shell)         │   │ │  
│  │  │    └── libs/pearls/specimen-accessioning                                 │   │ │  
│  │  │        ├── features/orders-feat (Feature Module)                         │   │ │  
│  │  │        ├── ui/orders-ui (UI Components, Store, Middleware)               │   │ │  
│  │  │        └── common (Shared Utilities)                                     │   │ │  
│  │  └──────────────────────────────────────────────────────────────────────────┘   │ │  
│  │                                      │                                           │ │  
│  │  ┌──────────────────────────────────────────────────────────────────────────┐   │ │  
│  │  │    shared/data-access/orders-data-access                                 │   │ │  
│  │  │    ├── services/ (OrdersService, TestsService, AccountsService, etc.)    │   │ │  
│  │  │    ├── models/ (OrderModel, TestModel, SpecimenModel, PatientModel, etc.)│   │ │  
│  │  │    └── types/ (Enums, Constants, Type Definitions)                       │   │ │  
│  │  └──────────────────────────────────────────────────────────────────────────┘   │ │  
│  └─────────────────────────────────────────────────────────────────────────────────┘ │  
└─────────────────────────────────────────────────────────────────────────────────────┘  
                                          │                                          │ HTTP/REST (Azure AD JWT)                                          ▼┌─────────────────────────────────────────────────────────────────────────────────────┐  
│                                   API GATEWAY LAYER                                  │  
│                                                                                       │  
│  ┌─────────────────────────────────────────────────────────────────────────────────┐ │  
│  │              rls-orders-cnsmr-api (Orders Consumer API) - .NET 8.0             │ │  
│  │                                                                                  │ │  
│  │  Purpose: UI-facing API that orchestrates multiple backend services              │ │  
│  │                                                                                  │ │  
│  │  ┌────────────────────────────────────────────────────────────────────────┐     │ │  
│  │  │  Controllers (API Layer)                                               │     │ │  
│  │  │  ├── OrdersController    (/api/v1/orders)                              │     │ │  
│  │  │  ├── TestsController     (/api/v1/tests)                               │     │ │  
│  │  │  ├── AccountsController  (/api/v1/accounts)                            │     │ │  
│  │  │  ├── DocumentsController (/api/v1/documents)                           │     │ │  
│  │  │  ├── LabelsController    (/api/v1/labels)                              │     │ │  
│  │  │  └── ReportsController   (/api/v1/reports)                             │     │ │  
│  │  └────────────────────────────────────────────────────────────────────────┘     │ │  
│  │                                      │                                           │ │  
│  │  ┌────────────────────────────────────────────────────────────────────────┐     │ │  
│  │  │  BLL (Business Logic Layer)                                            │     │ │  
│  │  │  ├── OrderService (Order operations, change detection, save logic)     │     │ │  
│  │  │  ├── TestService (Test catalog integration)                            │     │ │  
│  │  │  ├── LtcService (Lab Test Catalog)                                     │     │ │  
│  │  │  ├── DocumentService (Document management)                             │     │ │  
│  │  │  ├── LabelService (PDF label generation)                               │     │ │  
│  │  │  ├── ReportService (Report generation)                                 │     │ │  
│  │  │  ├── SpecimenService (Specimen operations)                             │     │ │  
│  │  │  ├── DataverseService (CRM integration)                                │     │ │  
│  │  │  └── Composer/ (PDF composers using QuestPDF)                          │     │ │  
│  │  └────────────────────────────────────────────────────────────────────────┘     │ │  
│  │                                      │                                           │ │  
│  │  ┌────────────────────────────────────────────────────────────────────────┐     │ │  
│  │  │  DAL (Data Access Layer - Repository Pattern)                          │     │ │  
│  │  │  ├── OrdersRepository     → Calls rls-orders-orch-api                  │     │ │  
│  │  │  ├── LtcRepository        → Calls LTC API                              │     │ │  
│  │  │  ├── DataverseRepository  → Calls Dataverse CRM API                    │     │ │  
│  │  │  ├── DocgenRepository     → Calls DocGen API                           │     │ │  
│  │  │  ├── SpecimenRepository   → Calls rls-orders-orch-api                  │     │ │  
│  │  │  └── TestRepository       → Calls rls-orders-orch-api                  │     │ │  
│  │  └────────────────────────────────────────────────────────────────────────┘     │ │  
│  └─────────────────────────────────────────────────────────────────────────────────┘ │  
└─────────────────────────────────────────────────────────────────────────────────────┘  
                                          │                                          │ HTTP/REST (Service-to-Service)                                          ▼┌─────────────────────────────────────────────────────────────────────────────────────┐  
│                                  ORCHESTRATION LAYER                                 │  
│                                                                                       │  
│  ┌─────────────────────────────────────────────────────────────────────────────────┐ │  
│  │            rls-orders-orch-api (Orders Orchestration API) - .NET 8.0           │ │  
│  │                                                                                  │ │  
│  │  Purpose: Core business logic, order lifecycle management, event publishing     │ │  
│  │                                                                                  │ │  
│  │  ┌────────────────────────────────────────────────────────────────────────┐     │ │  
│  │  │  Controllers                                                           │     │ │  
│  │  │  ├── OrdersController         (/api/v1/orders)                         │     │ │  
│  │  │  ├── TestsController          (/api/v1/orders/{id}/tests)              │     │ │  
│  │  │  ├── SpecimensController      (/api/v1/specimens)                      │     │ │  
│  │  │  └── InterfacedOrdersController (HL7 interface handling)               │     │ │  
│  │  └────────────────────────────────────────────────────────────────────────┘     │ │  
│  │                                      │                                           │ │  
│  │  ┌────────────────────────────────────────────────────────────────────────┐     │ │  
│  │  │  BLL (Business Logic Layer)                                            │     │ │  
│  │  │  ├── OrdersBll                                                         │     │ │  
│  │  │  │   ├── GetOrderByOrderUid()                                          │     │ │  
│  │  │  │   ├── CreateOrder()                                                 │     │ │  
│  │  │  │   ├── UpdateOrder()                                                 │     │ │  
│  │  │  │   ├── SearchOrders()                                                │     │ │  
│  │  │  │   ├── AddTestToOrder()                                              │     │ │  
│  │  │  │   ├── CancelTests()                                                 │     │ │  
│  │  │  │   ├── UpdatePatientDemographics()                                   │     │ │  
│  │  │  │   ├── AddNoteToOrder()                                              │     │ │  
│  │  │  │   ├── TestReceive()                                                 │     │ │  
│  │  │  │   └── ResendResults()                                               │     │ │  
│  │  │  ├── SpecimensBll (Specimen lifecycle management)                      │     │ │  
│  │  │  ├── InterfacedOrdersBll (HL7 message processing)                      │     │ │  
│  │  │  └── Helpers/                                                          │     │ │  
│  │  │      ├── OrderEventHelper (Event creation)                             │     │ │  
│  │  │      ├── OrderTopicHelper (Pub/Sub message building)                   │     │ │  
│  │  │      ├── OrderStatusHelper (Status calculation)                        │     │ │  
│  │  │      ├── PopulateOrderModelHelper (Model population)                   │     │ │  
│  │  │      ├── ProductCatalogHelper (Product catalog integration)            │     │ │  
│  │  │      ├── PatientHelper (Patient data mapping)                          │     │ │  
│  │  │      └── TimezoneHelper (Timezone conversions)                         │     │ │  
│  │  └────────────────────────────────────────────────────────────────────────┘     │ │  
│  │                                      │                                           │ │  
│  │  ┌────────────────────────────────────────────────────────────────────────┐     │ │  
│  │  │  DAL (Data Access Layer)                                               │     │ │  
│  │  │  ├── OrdersDataService      → Calls rls-orders-data-api                │     │ │  
│  │  │  ├── SpecimensDataService   → Calls Specimens Data API                 │     │ │  
│  │  │  ├── WheelieDataService     → Order/Specimen number generation         │     │ │  
│  │  │  ├── DataverseService       → CRM account/timezone info                │     │ │  
│  │  │  ├── ProdCatService         → Product Catalog (test info)              │     │ │  
│  │  │  └── OrderTopicDataService  → Google Cloud Pub/Sub publishing          │     │ │  
│  │  └────────────────────────────────────────────────────────────────────────┘     │ │  
│  └─────────────────────────────────────────────────────────────────────────────────┘ │  
└─────────────────────────────────────────────────────────────────────────────────────┘  
                                          │                                          │ HTTP/REST                                          ▼┌─────────────────────────────────────────────────────────────────────────────────────┐  
│                                     DATA LAYER                                       │  
│                                                                                       │  
│  ┌─────────────────────────────────────────────────────────────────────────────────┐ │  
│  │                rls-orders-data-api (Orders Data API) - .NET 8.0                │ │  
│  │                                                                                  │ │  
│  │  Purpose: Direct MongoDB access, CRUD operations, search capabilities           │ │  
│  │                                                                                  │ │  
│  │  ┌────────────────────────────────────────────────────────────────────────┐     │ │  
│  │  │  Controllers                                                           │     │ │  
│  │  │  └── OrdersController    (/api/v1/orders)                              │     │ │  
│  │  │      ├── GetOrderByOrderUid()                                          │     │ │  
│  │  │      ├── GetOrderByOrderNumber()                                       │     │ │  
│  │  │      ├── GetOrderByOrderingSystem()                                    │     │ │  
│  │  │      ├── AddOrder()                                                    │     │ │  
│  │  │      ├── UpdateOrder()                                                 │     │ │  
│  │  │      └── SearchOrders()                                                │     │ │  
│  │  └────────────────────────────────────────────────────────────────────────┘     │ │  
│  │                                      │                                           │ │  
│  │  ┌────────────────────────────────────────────────────────────────────────┐     │ │  
│  │  │  BLL (Business Logic Layer)                                            │     │ │  
│  │  │  └── OrdersBll                                                         │     │ │  
│  │  │      ├── Validation (CreateOrderModelValidator, UpdateOrderValidator)  │     │ │  
│  │  │      ├── Mapping (OrderMappingHelper, EventMappingHelper)              │     │ │  
│  │  │      ├── SearchOrdersHelper (MongoDB query construction)               │     │ │  
│  │  │      └── Optimistic Locking (RevisionNumber tracking)                  │     │ │  
│  │  └────────────────────────────────────────────────────────────────────────┘     │ │  
│  │                                      │                                           │ │  
│  │  ┌────────────────────────────────────────────────────────────────────────┐     │ │  
│  │  │  DAL (Data Access Layer)                                               │     │ │  
│  │  │  └── OrderDataService                                                  │     │ │  
│  │  │      ├── OrdersCollectionHelper (MongoDB collection operations)        │     │ │  
│  │  │      ├── GetOrderDocument()                                            │     │ │  
│  │  │      ├── InsertOrderDocument()                                         │     │ │  
│  │  │      ├── UpdateOrderDocument() (with optimistic locking)               │     │ │  
│  │  │      ├── SearchOrderDocuments()                                        │     │ │  
│  │  │      └── GetOrderEvents()                                              │     │ │  
│  │  └────────────────────────────────────────────────────────────────────────┘     │ │  
│  └─────────────────────────────────────────────────────────────────────────────────┘ │  
│                                          │                                           │  
│                                          │ MongoDB Driver                            │  
│                                          ▼                                           │  
│  ┌─────────────────────────────────────────────────────────────────────────────────┐ │  
│  │                              MongoDB Database                                   │ │  
│  │  ├── Orders Collection (OrderDocumentModel)                                     │ │  
│  │  └── Events Collection (EventDocumentModel - Order history/audit log)          │ │  
│  └─────────────────────────────────────────────────────────────────────────────────┘ │  
└─────────────────────────────────────────────────────────────────────────────────────┘  
```  
  
---  
  
## External Service Integrations  
  
```  
┌─────────────────────────────────────────────────────────────────────────────────────┐  
│                            EXTERNAL SERVICE INTEGRATIONS                             │  
│                                                                                       │  
│  ┌─────────────────────┐  ┌─────────────────────┐  ┌─────────────────────────────┐  │  
│  │   Dataverse CRM     │  │  Lab Test Catalog   │  │     Product Catalog         │  │  
│  │   (Microsoft)       │  │  (LTC API)          │  │     (ProdCat API)           │  │  
│  │                     │  │                     │  │                             │  │  
│  │ • Account info      │  │ • Test details      │  │ • PEaRLS product models     │  │  
│  │ • Client timezones  │  │ • Specimen specs    │  │ • Test configuration        │  │  
│  │ • Contact data      │  │ • Stability info    │  │ • Results configuration     │  │  
│  └─────────────────────┘  └─────────────────────┘  └─────────────────────────────┘  │  
│                                                                                       │  
│  ┌─────────────────────┐  ┌─────────────────────┐  ┌─────────────────────────────┐  │  
│  │   Wheelie API       │  │   DocGen API        │  │   Google Cloud Storage      │  │  
│  │   (Number Gen)      │  │   (Documents)       │  │   (Report Storage)          │  │  
│  │                     │  │                     │  │                             │  │  
│  │ • Order numbers     │  │ • PDF generation    │  │ • Report documents          │  │  
│  │ • Specimen numbers  │  │ • Document storage  │  │ • Signed URLs               │  │  
│  │ • Prefix handling   │  │                     │  │                             │  │  
│  └─────────────────────┘  └─────────────────────┘  └─────────────────────────────┘  │  
│                                                                                       │  
│  ┌──────────────────────────────────────────────────────────────────────────────┐   │  
│  │                       Google Cloud Pub/Sub (Order Topic)                     │   │  
│  │                                                                               │   │  
│  │  Published Events (from Orchestration API):                                   │   │  
│  │  • OrderCreated          • TestAdded           • TestCanceled                │   │  
│  │  • OrderUpdate           • TestUpdate          • OrderCanceled               │   │  
│  │  • DemographicUpdate     • TestReceived        • NoteAdded                   │   │  
│  │  • AccountUpdated        • Resulted            • ResendResults               │   │  
│  └──────────────────────────────────────────────────────────────────────────────┘   │  
└─────────────────────────────────────────────────────────────────────────────────────┘  
```  
  
---  
  
## Layer Responsibilities  
  
### 1. Presentation Layer (mcs-products-mono-ui)  
  
**Repository:** `mcs-products-mono-ui`  
  
**Technology Stack:**  
- Angular 18+ with standalone components  
- Nx Monorepo architecture  
- NGRX Signal Store for state management  
- PrimeNG UI components  
- TypeScript, Sass  
  
**Key Libraries:**  
  
| Library Path | Purpose |  
|--------------|---------|  
| `libs/pearls/specimen-accessioning/features/orders-feat` | Feature module containing order components and resolvers |  
| `libs/pearls/specimen-accessioning/ui/orders-ui` | UI components, services, state management, middleware |  
| `libs/pearls/specimen-accessioning/common` | Shared utilities and constants |  
| `libs/shared/data-access/orders-data-access` | HTTP services and data models for API communication |  
  
**State Management (orders.store.ts):**  
- Signal-based reactive state using `@ngrx/signals`  
- Manages: order, initialOrder, selectedMayoTestId, hasChanged, isLoading, activeSidebar, etc.  
- Methods for order CRUD, test management, specimen handling  
  
**Angular Services (orders-data-access):**  
  
| Service | Purpose | API Target |  
|---------|---------|------------|  
| `OrdersService` | Order CRUD operations | `/api/v1/orders` |  
| `TestsService` | Test catalog lookups | `/api/v1/tests` |  
| `AccountsService` | Account information | `/api/v1/accounts` |  
| `LabelsService` | Label PDF generation | `/api/v1/labels` |  
| `OrdersSearchService` | Order search | `/api/v1/orders/search` |  
  
---  
  
### 2. API Gateway Layer (rls-orders-cnsmr-api)  
  
**Repository:** `rls-orders-cnsmr-api`  
  
**Technology Stack:**  
- .NET 8.0 Web API  
- Azure AD Authentication (JWT)  
- QuestPDF for label generation  
- AutoMapper for object mapping  
- FluentValidation  
  
**Purpose:** UI-facing API that aggregates and orchestrates multiple backend services. Acts as a Backend-for-Frontend (BFF) pattern.  
  
**Controllers & Endpoints:**  
  
| Controller | Base Route | Key Endpoints |  
|------------|------------|---------------|  
| `OrdersController` | `/api/v1/orders` | GET `/{orderId}`, GET `?orderIdentifier=`, POST, PUT `/{orderId}`, GET `/search`, GET `/export` |  
| `TestsController` | `/api/v1/tests` | GET `/{mayoTestId}`, POST `/{orderUid}/tests/resend-results` |  
| `AccountsController` | `/api/v1/accounts` | GET `/{accountNumber}` |  
| `DocumentsController` | `/api/v1/documents` | GET (document retrieval) |  
| `LabelsController` | `/api/v1/labels` | POST `/processing`, `/aliquot`, `/storage`, `/extra` |  
| `ReportsController` | `/api/v1/reports` | GET `/{orderUid}/{mayoTestId}` |  
  
**BLL Services:**  
  
| Service | Responsibility |  
|---------|----------------|  
| `OrderService` | Order CRUD, change detection, update orchestration |  
| `TestService` | Test catalog integration, AOE prompts |  
| `LtcService` | Lab Test Catalog data retrieval |  
| `DocumentService` | Document management |  
| `LabelService` | PDF label generation (using QuestPDF composers) |  
| `ReportService` | Report URL generation |  
| `SpecimenService` | Specimen operations |  
| `DataverseService` | CRM account information |  
  
**DAL Repositories:**  
  
| Repository | External Service |  
|------------|------------------|  
| `OrdersRepository` | rls-orders-orch-api |  
| `LtcRepository` | Lab Test Catalog API |  
| `DataverseRepository` | Dataverse CRM API |  
| `DocgenRepository` | DocGen API |  
| `SpecimenRepository` | rls-orders-orch-api |  
| `TestRepository` | rls-orders-orch-api |  
  
---  
  
### 3. Orchestration Layer (rls-orders-orch-api)  
  
**Repository:** `rls-orders-orch-api`  
  
**Technology Stack:**  
- .NET 8.0 Web API  
- Google Cloud Pub/Sub integration  
- Azure AD Authentication  
- AutoMapper, FluentValidation  
  
**Purpose:** Core business logic layer handling order lifecycle management, validation, and event publishing to Pub/Sub.  
  
**Controllers & Endpoints:**  
  
| Controller | Base Route | Key Operations |  
|------------|------------|----------------|  
| `OrdersController` | `/api/v1/orders` | Order CRUD, search, notes, account updates |  
| `TestsController` | `/api/v1/orders/{orderUid}/tests` | Test add, update, cancel, receipt, receive, resend results |  
| `SpecimensController` | `/api/v1/specimens` | Specimen lookup, triage resolution |  
| `InterfacedOrdersController` | `/api/v1/interfaced-orders` | HL7 interface message handling |  
  
**BLL - OrdersBll Key Methods:**  
  
| Method | Purpose |  
|--------|---------|  
| `GetOrderByOrderUid()` | Retrieve order by internal ID |  
| `CreateOrder()` | Create new order with validation and Pub/Sub event |  
| `UpdateOrder()` | Update order details (not tests/notes) |  
| `AddTestToOrder()` | Add tests to existing order |  
| `CancelTests()` | Cancel tests with proper status handling |  
| `UpdateTests()` | Update test details |  
| `UpdatePatientDemographics()` | Update patient information |  
| `AddNoteToOrder()` | Add order/internal/ref-lab notes |  
| `TestReceive()` | Mark tests as received at lab |  
| `ResendResults()` | Trigger result resend |  
| `UpdateAccountNumber()` | Update account and timezone |  
| `SearchOrders()` | Search with pagination |  
  
**DAL Services:**  
  
| Service | Purpose |  
|---------|---------|  
| `OrdersDataService` | HTTP client for rls-orders-data-api |  
| `SpecimensDataService` | Specimen CRUD operations |  
| `WheelieDataService` | Generate order/specimen numbers |  
| `DataverseService` | Get account timezone from CRM |  
| `ProdCatService` | Get product catalog information |  
| `OrderTopicDataService` | Publish events to Google Cloud Pub/Sub |  
  
**Pub/Sub Event Types:**  
  
| Event Subtype | Trigger |  
|---------------|---------|  
| `OrderCreated` | New order created |  
| `OrderUpdate` | Order details updated |  
| `OrderCanceled` | All tests canceled |  
| `TestAdded` | Tests added to order |  
| `TestUpdate` | Test details updated |  
| `TestCanceled` | Tests canceled |  
| `TestReceived` | Tests received at lab |  
| `DemographicUpdate` | Patient demographics updated |  
| `NoteAdded` | Note added to order |  
| `AccountUpdated` | Account number changed |  
| `Resulted` | Results processed |  
  
---  
  
### 4. Data Layer (rls-orders-data-api)  
  
**Repository:** `rls-orders-data-api`  
  
**Technology Stack:**  
- .NET 8.0 Web API  
- MongoDB Driver  
- Azure AD Authentication  
- AutoMapper, FluentValidation  
  
**Purpose:** Direct database access layer for MongoDB. Handles CRUD operations and complex search queries.  
  
**Controller - OrdersController:**  
  
| Endpoint | Method | Purpose |  
|----------|--------|---------|  
| `/{orderUid:guid}` | GET | Get order by internal GUID |  
| `/{orderNumber}` | GET | Get order by order number |  
| `/?orderNumber=&sourceSystem=&supportingId=` | GET | Get order by ordering system |  
| `/` | POST | Create new order |  
| `/{orderUid}` | PUT | Update order |  
| `/search` | POST | Search orders with filters |  
  
**BLL - OrdersBll:**  
  
| Method | Purpose |  
|--------|---------|  
| `GetOrderByOrderUid()` | Retrieve order + events by GUID |  
| `GetOrderByOrderingSystem()` | Lookup by external system identifiers |  
| `GetOrderByOrderNumber()` | Lookup by PEaRLS order number |  
| `AddOrder()` | Insert order with validation |  
| `UpdateOrder()` | Update with optimistic locking |  
| `SearchOrders()` | Dynamic MongoDB query execution |  
  
**DAL - OrderDataService:**  
  
| Method | Purpose |  
|--------|---------|  
| `GetOrderById()` | MongoDB find by OrderUid |  
| `GetOrderByOrderNumber()` | MongoDB find by order number |  
| `GetOrderByOrderingSystem()` | Complex lookup with ordering systems |  
| `CreateOrder()` | Insert order + event documents |  
| `UpdateOrder()` | FindAndReplace with revision check |  
| `SearchOrders()` | Aggregation pipeline execution |  
| `GetEventsByOrderUid()` | Retrieve order event history |  
| `OrderExists()` | Check for duplicates |  
  
**MongoDB Collections:**  
  
| Collection | Document Model | Purpose |  
|------------|----------------|---------|  
| Orders | `OrderDocumentModel` | Order data with nested tests, specimens, patient, account |  
| Events | `EventDocumentModel` | Audit log of all order changes |  
  
**Key Features:**  
- **Optimistic Locking:** Uses `RevisionNumber` to prevent concurrent update conflicts  
- **Event Sourcing:** All changes logged to Events collection for audit trail  
- **Dynamic Queries:** BsonDocument-based filter, sort, and projection building  
  
---  
  
## Data Models  
  
### Core Order Model Structure  
  
```  
OrderModel/DataOrderModel  
├── OrderUid (Guid)  
├── OrderNumber (string - e.g., "R12345678")  
├── Status (OrderStatus enum)  
├── RevisionNumber (int - optimistic locking)  
├── CollectedOnUtc (DateTime)  
├── Account  
│   ├── AccountNumber  
│   ├── AccountName  
│   └── TimeZone  
├── Patient  
│   ├── FirstName, LastName, MiddleName  
│   ├── DateOfBirth  
│   ├── Sex  
│   ├── PatientId (MRN)  
│   └── Address  
├── OrderingProvider  
│   ├── NPI  
│   ├── FirstName, LastName  
│   └── Credentials  
├── Tests[]  
│   ├── MayoTestId  
│   ├── Status (TestStatus enum)  
│   ├── Specimens[]  
│   │   ├── SpecimenId  
│   │   ├── SpecimenNumber  
│   │   ├── SpecimenType  
│   │   ├── Container  
│   │   └── Temperature  
│   ├── AoeQuestionsAndAnswers[]  
│   ├── ReceivedOn  
│   ├── IsCanceled  
│   └── CanceledReason  
├── OrderingSystem[] (external identifiers)  
│   ├── SourceSystem  
│   ├── OrderNumber  
│   └── SupportingId  
├── Notes  
│   ├── OrderNotes[]  
│   ├── InternalNotes[]  
│   └── RefLabNotes[]  
├── Events[] (history)  
│   ├── EventType  
│   ├── EventSubType  
│   ├── EventData  
│   ├── Actor  
│   └── Timestamp  
└── HoldResults (bool)  
```  
  
### Test Status Workflow  
  
```  
Posted → Received → Partial → Final/Revised  
           ↓        Canceled```  
  
### Order Status Calculation  
  
Order status is computed based on aggregate test statuses:  
- `Posted` - All tests in Posted status  
- `Received` - At least one test Received, none Final  
- `Partial` - Mix of Received and Final tests  
- `Final` - All non-canceled tests are Final/Revised  
- `Canceled` - All tests canceled  
  
---  
  
## Authentication & Authorization  
  
### Azure AD Integration  
  
All APIs use Azure AD JWT Bearer token authentication.  
  
**App Roles:**  
  
| Role | Scope |  
|------|-------|  
| `OrderRead` | Read order data |  
| `OrderWrite` | Create/Update orders |  
| `OrdersConsumerOrderRead` | Consumer API read access |  
| `OrdersConsumerOrderWrite` | Consumer API write access |  
| `SpecimenRead` | Read specimen data |  
| `SpecimenWrite` | Create/Update specimens |  
| `AdminSearch` | Advanced search capabilities |  
| `ReferralsSearch` | Referrals search access |  
| `QualitySearch` | Quality search access |  
| `PearlsResultsList` | Results resend capability |  
  
---  
  
## Deployment Architecture  
  
### Google Cloud Platform (Cloud Run)  
  
```  
┌────────────────────────────────────────────────────────────────┐  
│                    Google Cloud Platform                        │  
│                                                                  │  
│  ┌──────────────────────────────────────────────────────────┐  │  
│  │                      Cloud Run                            │  │  
│  │  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────┐ │  │  
│  │  │ specimen-       │ │ orders-         │ │ orders-     │ │  │  
│  │  │ accessioning-ui │ │ consumer-api    │ │ orch-api    │ │  │  
│  │  │ (Angular)       │ │ (.NET 8)        │ │ (.NET 8)    │ │  │  
│  │  └─────────────────┘ └─────────────────┘ └─────────────┘ │  │  
│  │                                          ┌─────────────┐ │  │  
│  │                                          │ orders-     │ │  │  
│  │                                          │ data-api    │ │  │  
│  │                                          │ (.NET 8)    │ │  │  
│  │                                          └─────────────┘ │  │  
│  └──────────────────────────────────────────────────────────┘  │  
│                                                                  │  
│  ┌──────────────────┐  ┌──────────────────┐  ┌───────────────┐ │  
│  │   Pub/Sub        │  │   Cloud Storage  │  │   MongoDB     │ │  
│  │   (Order Topic)  │  │   (Reports)      │  │   Atlas       │ │  
│  └──────────────────┘  └──────────────────┘  └───────────────┘ │  
└────────────────────────────────────────────────────────────────┘  
```  
  
### Environment Configuration  
  
| Environment | UI Project | API Project | Base URL Pattern |  
|-------------|------------|-------------|------------------|  
| DEV | uiprj-app-dev-fc89 | orch-app-dev-c75e | `*-dev.mcs-rls-n.caf.mccapp.com` |  
| TEST | uiprj-app-test-942f | TBD | `*-test.mcs-rls-n.caf.mccapp.com` |  
| STAGE | uiprj-app-stage-6ecf | TBD | `*-stage.mcs-rls-p.caf.mccapp.com` |  
| PROD | TBD | TBD | `*-prod.mcs-rls-p.caf.mccapp.com` |  
  
---  
  
## Key Patterns & Practices  
  
### 1. Layered Architecture  
Each API follows a consistent three-layer pattern:  
- **Api (Controllers)** - HTTP routing, authorization, request/response mapping  
- **BLL (Business Logic)** - Validation, business rules, orchestration  
- **DAL (Data Access)** - External service communication, database operations  
  
### 2. Dependency Injection  
Uses Mayo ODN DI framework with `[OdnDiRegister]` attribute for automatic service registration.  
  
### 3. AutoMapper  
Extensive use of AutoMapper for model transformations between layers (Api Model ↔ BLL Model ↔ DAL/Document Model).  
  
### 4. FluentValidation  
All input validation implemented using FluentValidation with custom validators.  
  
### 5. Feature Toggles  
Runtime feature flags for enabling/disabling functionality:  
```  
OrdersFeature|DocGenFeature  
```  
  
### 6. Error Handling  
- Custom exceptions (`NotFoundException`, `BadRequestException`, `OrdersConsumerApiException`, etc.)  
- Silent error header (`X-Error-Type: silent`) for UI handling  
- Structured logging with sensitive data masking  
  
### 7. Optimistic Locking  
`RevisionNumber` field prevents concurrent update conflicts at the data layer.  
  
### 8. Event-Driven Architecture  
Order changes published to Google Cloud Pub/Sub for downstream processing.  
  
---  
  
## Quick Reference  
  
### Repository URLs  
  
| Repository | Purpose |  
|------------|---------|  
| `mcs-products-mono-ui` | Angular monorepo (UI) |  
| `rls-orders-cnsmr-api` | Consumer API (BFF) |  
| `rls-orders-orch-api` | Orchestration API |  
| `rls-orders-data-api` | Data API (MongoDB) |  
  
### API Base URLs (Dev)  
  
| Service | URL |  
|---------|-----|  
| Consumer API | `rls-cnsmr-dev.mcs-rls-n.caf.mccapp.com` |  
| Orchestration API | `rls-orch-dev.mcs-rls-n.caf.mccapp.com` |  
| Data API | Internal only |  
  
### Key Configuration Settings  
  
| Setting | Purpose |  
|---------|---------|  
| `rls-orders-consumer-feature-toggle` | Feature toggles |  
| `rls-orders-orchestration-api-base-url` | Orch API URL |  
| `dataverse-api-base-url` | CRM API URL |  
| `ltc-api-base-url` | Lab Test Catalog URL |  
| `gcp-order-topic-id` | Pub/Sub topic |  
  
---  
  
## Document Revision History  
  
| Version | Date | Author | Changes |  
|---------|------|--------|---------|  
| 1.0 | 2026-01-27 | Generated | Initial comprehensive architecture document |