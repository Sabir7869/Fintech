# TrackMyFin

TrackMyFin is a full-stack personal finance tracker with:
- Spring Boot backend (REST API + JWT authentication)
- React + TypeScript frontend
- PostgreSQL/H2 persistence via Spring Data JPA
- AI-assisted expense extraction from voice and bill images (Gemini)

This README is a technical reference for:
- project setup
- database schema
- backend API endpoints
- request flow
- DTO and data mapping flow

## 1) Project Structure

```text
TrackMyFin/
|-- TrackMyFin_Backend/      # Spring Boot API
|   |-- src/main/java/com/financetracker/
|   |   |-- controller/
|   |   |-- service/
|   |   |-- repository/
|   |   |-- entity/
|   |   |-- dto/
|   |   |-- security/
|   |   `-- config/
|   |-- src/main/resources/
|   |   |-- application.properties
|   |   `-- application-dev.properties
|   `-- pom.xml
|-- TrackMyFin_UI/           # React frontend
|   |-- src/
|   |-- public/
|   `-- package.json
`-- README.md
```

## 2) Tech Stack

### Backend
- Java 21
- Spring Boot 3.3.3
- Spring Security + JWT
- Spring Data JPA + Hibernate
- PostgreSQL (main config) and H2 (dev profile)
- Maven

### Frontend
- React 19 + TypeScript
- React Router
- Recharts
- Tailwind CSS

## 3) Quick Start

## Prerequisites
- JDK 21
- Maven 3.6+
- Node.js 18+
- npm

### Backend run (dev profile, H2)

```powershell
cd TrackMyFin_Backend
$env:SPRING_PROFILES_ACTIVE='dev'
$env:SERVER_PORT='8081'
mvn spring-boot:run
```

API base URL:
- `http://localhost:8081` (dev profile default)

### Backend run (default profile)

```powershell
cd TrackMyFin_Backend
mvn spring-boot:run
```

API base URL:
- `http://localhost:8080`

### Frontend run

```powershell
cd TrackMyFin_UI
npm install
npm start
```

UI URL:
- `http://localhost:3000`

## 4) Configuration

Backend properties are in:
- `TrackMyFin_Backend/src/main/resources/application.properties`
- `TrackMyFin_Backend/src/main/resources/application-dev.properties`

Important settings:
- `server.port`
- `spring.datasource.*`
- `jwt.secret`
- `jwt.expiration`
- `gemini.api.key` (or env var `GEMINI_API_KEY`)
- `gemini.model`

Recommended security practice:
- Do not commit real DB credentials and secrets.
- Use environment variables or an external secret manager.

## 5) Backend Architecture and Request Flow

Standard request flow:

```text
Client (React/Postman)
  -> Controller (@RestController)
     -> Service (business logic)
        -> Repository (Spring Data JPA)
           -> Database
        <- Entity
     <- DTO/Entity response
  <- JSON response
```

Authentication flow:

```text
Login/Register
  -> /api/auth/*
  <- JWT token

For protected endpoints:
Authorization: Bearer <token>
  -> JwtAuthenticationFilter validates token
  -> SecurityContext is populated with User
  -> Controller uses Authentication principal
```

Security behavior summary:
- Public: `/api/auth/**`, `/api/health/**`, `/api/dashboard/health`, category endpoints currently open in `SecurityConfig`
- Protected: all other routes by default

## 6) Database Schema (From JPA Entities)

Primary tables:
- `users`
- `transactions`
- `categories`
- `budgets`
- `salaries`
- `expenses`

### ER Diagram (logical)

```mermaid
erDiagram
    USERS ||--o{ TRANSACTIONS : has
    USERS ||--o{ BUDGETS : has
    USERS ||--o{ SALARIES : has
    USERS ||--o{ EXPENSES : has
    CATEGORIES ||--o{ TRANSACTIONS : classifies
    CATEGORIES ||--o{ BUDGETS : optional

    USERS {
      bigint id PK
      string email UNIQUE
      string password
      string first_name
      string last_name
      string phone_number
      string address
      string date_of_birth
      string bio
      string currency
      string role
      boolean is_enabled
      datetime created_at
      datetime updated_at
    }

    TRANSACTIONS {
      bigint id PK
      decimal amount
      string description
      string type
      bigint category_id FK
      bigint user_id FK
      datetime transaction_date
      string notes
      datetime created_at
      datetime updated_at
    }

    CATEGORIES {
      bigint id PK
      string name
      string description
      string color
      string icon
      string type
      boolean is_default
      datetime created_at
      datetime updated_at
    }

    BUDGETS {
      bigint id PK
      string name
      decimal amount
      decimal spent_amount
      bigint category_id FK
      bigint user_id FK
      string period
      date start_date
      date end_date
      boolean is_active
      datetime created_at
      datetime updated_at
    }

    SALARIES {
      bigint id PK
      decimal amount
      string description
      datetime salary_date
      bigint user_id FK
      datetime created_at
      datetime updated_at
    }

    EXPENSES {
      bigint id PK
      string title
      decimal amount
      datetime date
      string category
      string source
      datetime created_at
      bigint user_id FK
    }
```

### Notes
- `transactions.type`: `INCOME | EXPENSE`
- `categories.type`: `INCOME | EXPENSE`
- `budgets.period`: `WEEKLY | MONTHLY | QUARTERLY | YEARLY`
- `users.currency`: enum from `Currency` (USD, EUR, INR, ...)

## 7) Backend API Endpoints

Base URL examples:
- `http://localhost:8080`
- `http://localhost:8081` (dev profile)

Auth header for protected routes:
- `Authorization: Bearer <JWT_TOKEN>`

### 7.1 Auth

| Method | Endpoint | Auth | Request DTO | Response DTO |
|---|---|---|---|---|
| GET | `/api/auth/currencies` | No | - | `List<Map<String,String>>` |
| POST | `/api/auth/register` | No | `RegisterRequest` | `AuthResponse` |
| POST | `/api/auth/login` | No | `LoginRequest` | `AuthResponse` |

### 7.2 Profile

| Method | Endpoint | Auth | Request DTO | Response DTO |
|---|---|---|---|---|
| GET | `/api/profile` | Yes | - | `ProfileResponse` |
| PUT | `/api/profile` | Yes | `ProfileRequest` | `ProfileResponse` |

### 7.3 Transactions

| Method | Endpoint | Auth | Request DTO | Response DTO |
|---|---|---|---|---|
| POST | `/api/transactions` | Yes | `TransactionRequest` | `TransactionDTO` |
| GET | `/api/transactions` | Yes | - | `List<TransactionDTO>` |
| GET | `/api/transactions/date-range?startDate=&endDate=` | Yes | Query params | `List<TransactionDTO>` |
| GET | `/api/transactions/{id}` | Yes | Path param | `TransactionDTO` |
| PUT | `/api/transactions/{id}` | Yes | `TransactionRequest` | `TransactionDTO` |
| DELETE | `/api/transactions/{id}` | Yes | Path param | Empty body |
| GET | `/api/transactions/summary` | Yes | - | `Map<String,BigDecimal>` |

### 7.4 Salaries

| Method | Endpoint | Auth | Request DTO | Response DTO |
|---|---|---|---|---|
| POST | `/api/salaries` | Yes | `SalaryRequest` | `SalaryResponse` |
| GET | `/api/salaries` | Yes | - | `List<SalaryResponse>` |
| GET | `/api/salaries/{id}` | Yes | Path param | `SalaryResponse` |
| PUT | `/api/salaries/{id}` | Yes | `SalaryRequest` | `SalaryResponse` |
| DELETE | `/api/salaries/{id}` | Yes | Path param | Empty body |

### 7.5 Categories

| Method | Endpoint | Auth (current config) | Request | Response |
|---|---|---|---|---|
| POST | `/api/categories` | No | `Category` entity JSON | `Category` |
| GET | `/api/categories` | No | - | `List<Category>` |
| GET | `/api/categories/type/{type}` | No | Path `INCOME/EXPENSE` | `List<Category>` |
| GET | `/api/categories/default` | No | - | `List<Category>` |
| GET | `/api/categories/{id}` | No | Path param | `Category` |
| PUT | `/api/categories/{id}` | No | `Category` entity JSON | `Category` |
| DELETE | `/api/categories/{id}` | No | Path param | status/message map |
| POST | `/api/categories/initialize-defaults` | No | - | String message |

### 7.6 Budgets

| Method | Endpoint | Auth | Request | Response |
|---|---|---|---|---|
| POST | `/api/budgets` | Yes | `Budget` entity JSON | `Budget` |
| GET | `/api/budgets` | Yes | - | `List<Budget>` |
| GET | `/api/budgets/active` | Yes | - | `List<Budget>` |
| GET | `/api/budgets/current` | Yes | - | `List<Budget>` |
| GET | `/api/budgets/{id}` | Yes | Path param | `Budget` |
| PUT | `/api/budgets/{id}` | Yes | `Budget` entity JSON | `Budget` |
| DELETE | `/api/budgets/{id}` | Yes | Path param | Empty body |

### 7.7 Dashboard and Analytics

| Method | Endpoint | Auth | Request | Response DTO |
|---|---|---|---|---|
| GET | `/api/dashboard/stats` | Yes | Optional `startDate`,`endDate` (YYYY-MM-DD) | `DashboardStatsResponse` |
| GET | `/api/dashboard/expenses-chart` | Yes | Query `range` (`6m`,`12m`,`ytd`) | `ExpenseChartResponse` |
| GET | `/api/dashboard/insights` | Yes | - | `Map<String,List<String>>` |
| GET | `/api/dashboard/health` | No | - | String |

### 7.8 Expense Extraction (Bill Upload)

| Method | Endpoint | Auth | Request DTO | Response |
|---|---|---|---|---|
| POST | `/api/bills` or `/api/bills/extract` | Yes | `BillExtractRequest` | `BillExtractResponse` |
| GET | `/api/expenses/summary/current-month` | Yes | - | `ExpenseSummaryResponse` |
| POST | `/api/expenses/extract-and-save` | Yes | `BillExtractRequest` | `Map<String,Object>` |

### 7.9 Voice Expense

| Method | Endpoint | Auth | Request DTO | Response DTO |
|---|---|---|---|---|
| POST | `/api/voice-expense` | Yes | `VoiceExpenseRequest` | `TransactionDTO` |

### 7.10 Health

| Method | Endpoint | Auth | Response |
|---|---|---|---|
| GET | `/api/health` | No | status map |
| GET | `/api/health/gemini` | No | Gemini status map |

## 8) DTO Reference and Data Flow

### Auth DTOs
- `RegisterRequest`: `email`, `password`, `firstName`, `lastName`, `phoneNumber`, `currency`
- `LoginRequest`: `email`, `password`
- `AuthResponse`: `token`, `email`, `firstName`, `lastName`, `currency`, `message`

Flow:

```text
RegisterRequest/LoginRequest
  -> AuthController
  -> UserService + AuthenticationManager
  -> User entity
  -> JwtUtil.generateToken(...)
  -> AuthResponse
```

### Transaction DTOs
- `TransactionRequest`: `amount`, `description`, `type`, `categoryId`, `transactionDate`, `notes`
- `TransactionDTO`: `id`, `amount`, `description`, `notes`, `transactionDate`, `type`, `categoryId`, `categoryName`, `userId`, `createdAt`, `updatedAt`

Flow:

```text
TransactionRequest
  -> TransactionController
  -> Transaction entity (categoryId -> Category ref)
  -> TransactionService (loads Category, saves via TransactionRepository)
  -> Transaction entity
  -> mapToDTO(...)
  -> TransactionDTO
```

### Salary DTOs
- `SalaryRequest`: `amount`, `description`, `date`
- `SalaryResponse`: `id`, `amount`, `description`, `date`

### Profile DTOs
- `ProfileRequest`: profile fields + optional password change fields
- `ProfileResponse`: profile fields + optional `message`

### Dashboard DTOs
- `DashboardStatsResponse`: `totalBalance`, `monthlyIncome`, `monthlyExpenses`, `savingsRate`
- `ExpenseChartResponse`: `monthlyData`, `categoryData`
- `MonthlyData`: `month`, `amount`
- `CategoryExpenseData`: `name`, `amount`, `percentage`
- `ExpenseSummaryResponse`: `totalSpending`, `categoryWiseBreakdown`

### AI/Bill/Voice DTOs
- `BillExtractRequest`: `billImage` (base64)
- `ExtractedExpense`: `description`, `amount`, `category`, `date`, `merchant`
- `BillExtractResponse`: `expenses`, `merchant`, `message`, `success`
- `VoiceExpenseRequest`: `text`
- `VoiceTransactionDTO`: intermediate structure parsed from Gemini (`amount`, `type`, `category`, `description`, `date`)

Voice flow:

```text
VoiceExpenseRequest(text)
  -> VoiceExpenseController
  -> GeminiService.parseVoiceTransaction(...) -> VoiceTransactionDTO
  -> VoiceExpenseService maps to Transaction entity
  -> TransactionService.save
  -> TransactionDTO
```

Bill flow:

```text
BillExtractRequest(billImage)
  -> BillController / ExpenseController
  -> GeminiService.extractExpensesFromBill(...)
  -> BillExtractResponse / ExtractedExpense list
  -> map to Transaction or Expense entities
  -> save via repositories
  -> API response
```

## 9) Error Handling

Global exception handling is centralized in `GlobalExceptionHandler`:
- Validation errors (`MethodArgumentNotValidException`) -> `400`
- `InvalidGeminiResponseException` -> `422`
- `IllegalStateException` -> `503`
- Generic runtime errors -> `400`
- Unknown errors -> `500`

## 10) Build and Test

### Backend

```powershell
cd TrackMyFin_Backend
mvn clean test
mvn clean package
```

### Frontend

```powershell
cd TrackMyFin_UI
npm test
npm run build
```

## 11) Important Implementation Notes

- Category endpoints are currently publicly accessible due to current `SecurityConfig` matchers.
- Budget endpoints accept and return entity payloads directly (`Budget`), not `BudgetRequest` DTO.
- Category endpoints also use entity payloads (`Category`) directly.
- Dashboard total income includes both `transactions(INCOME)` and `salaries`.
- There are two bill-related persistence paths:
  - `/api/bills*` saves as `transactions` (expense type)
  - `/api/expenses/extract-and-save` saves as `expenses`

## 12) Additional Docs

- Backend README: `TrackMyFin_Backend/README.md`
- UI README: `TrackMyFin_UI/README.md`
- UI quick start: `TrackMyFin_UI/QUICK_START.md`
