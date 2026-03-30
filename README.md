# TrackMyFin

TrackMyFin is a full-stack personal finance tracker with JWT authentication, analytics, budgeting, and AI-assisted expense capture (bill image and voice text).

## What Is Inside

- Backend: Spring Boot REST API
- Frontend: React + TypeScript UI
- Databases: PostgreSQL (default profile), H2 in-memory DB (dev profile)
- AI support: Gemini-based expense extraction

## Version Details

These versions are taken from this repository configuration files.

### Backend

- Java: 21
- Spring Boot: 3.3.3
- Maven Compiler Plugin: 3.11.0
- JWT library (jjwt): 0.12.3
- Lombok: 1.18.42

### Frontend

- React: 19.1.1
- React DOM: 19.1.1
- React Scripts: 5.0.1
- TypeScript: 4.9.5
- React Router DOM: 7.9.1
- Recharts: 3.2.1
- Tailwind CSS: 3.4.17

## Repository Structure

```text
TrackMyFin/
|-- TrackMyFin_Backend/
|   |-- pom.xml
|   |-- src/main/java/com/financetracker/
|   `-- src/main/resources/
|-- TrackMyFin_UI/
|   |-- package.json
|   |-- src/
|   `-- public/
`-- README.md
```

## Run Locally (Step-by-Step)

## 1) Prerequisites

Install these first:

- Git
- JDK 21
- Maven 3.8+
- Node.js 18+ (Node.js 20 recommended)
- npm 9+

## 2) Clone This Repository

```bash
git clone https://github.com/Sabir7869/Fintech.git
cd Fintech
```

If your folder name is TrackMyFin instead of Fintech, use that folder name in the next commands.

## 3) Start Backend (Recommended: Dev Profile With H2)

PowerShell:

```powershell
cd TrackMyFin_Backend
$env:SPRING_PROFILES_ACTIVE='dev'
$env:SERVER_PORT='8081'
mvn clean spring-boot:run
```

Backend URL:

- http://localhost:8081

H2 console (dev profile only):

- http://localhost:8081/h2-console

## 4) Start Frontend

Open a new terminal:

```powershell
cd TrackMyFin_UI
npm install
npm start
```

Frontend URL:

- http://localhost:3000

## 5) Verify Everything Works

- Open the frontend and register/login.
- The UI should call backend APIs from the local backend URL.
- Optional backend health checks:
  - http://localhost:8081/api/health
  - http://localhost:8081/api/dashboard/health

## Configuration

Backend config files:

- TrackMyFin_Backend/src/main/resources/application.properties
- TrackMyFin_Backend/src/main/resources/application-dev.properties

Important properties you may want to change:

- server.port
- spring.datasource.url
- spring.datasource.username
- spring.datasource.password
- jwt.secret
- jwt.expiration
- gemini.api.key (or GEMINI_API_KEY env var)
- gemini.model

## Profiles

- dev profile:
  - Uses in-memory H2 database
  - Good for quick local development
- default profile:
  - Uses PostgreSQL settings from application.properties

Run backend with default profile:

```powershell
cd TrackMyFin_Backend
mvn clean spring-boot:run
```

Default backend URL:

- http://localhost:8080

## API Access Basics

- Public endpoints: authentication and health routes
- Protected endpoints: require JWT token in Authorization header

Header format:

```text
Authorization: Bearer <your_jwt_token>
```

## Troubleshooting

### Java version error

- This project enforces Java 21. If build fails, check java -version.

### Port already in use

- Change backend port with SERVER_PORT.
- Or stop the process using 8080/8081.

### npm install issues

- Remove node_modules and package-lock.json, then reinstall:

```powershell
cd TrackMyFin_UI
Remove-Item -Recurse -Force node_modules
Remove-Item -Force package-lock.json
npm install
```

### CORS/API connection issues

- Ensure backend is running first.
- Ensure frontend points to the correct backend URL.

## Security Note

If you are using this repository for your own deployment:

- Replace any sample/default secrets and database credentials.
- Use environment variables or a secret manager.
- Do not commit real production secrets.

## Useful Commands

Backend tests:

```powershell
cd TrackMyFin_Backend
mvn test
```

Frontend tests:

```powershell
cd TrackMyFin_UI
npm test
```

Frontend production build:

```powershell
cd TrackMyFin_UI
npm run build
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
