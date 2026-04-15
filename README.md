<!-- # Do List

## Features

- **User registration & login** with JWT access/refresh token authentication
- **Task management** — create, read, update, delete tasks with priority levels and due dates
- **Task status tracking** — toggle between PENDING and COMPLETED
- **Priority-based grouping** — tasks grouped by HIGH, MEDIUM, LOW with color-coded UI
- **Admin dashboard** — view stats (total users, tasks, completion rate, overdue count)
- **User management** — admins can list, create, search, and delete users
- **Password update** — authenticated users can change their own password
- **Role-based access control** — `ADMIN` and `USER` roles with endpoint-level enforcement
- **Automatic token refresh** — seamless access token renewal via refresh tokens
- **Responsive design** — mobile-first UI with Tailwind CSS
- **Swagger UI** — interactive API documentation

---

## Tech Stack

### Backend

| Component       | Technology                          |
| --------------- | ----------------------------------- |
| Language         | Java 21                             |
| Framework        | Spring Boot 4.0.0                   |
| Security         | Spring Security + JWT (JJWT 0.12.5) |
| Persistence      | Spring Data JPA / Hibernate         |
| Database (dev)   | H2 (in-memory)                      |
| Database (prod)  | PostgreSQL 18                       |
| API Docs         | SpringDoc OpenAPI (Swagger UI)      |
| Build Tool       | Maven                               |
| Utilities        | Lombok                              |

### Frontend

| Component       | Technology                          |
| --------------- | ----------------------------------- |
| Language         | TypeScript 5.9                      |
| Library          | React 19                            |
| Build Tool       | Vite 7                              |
| Styling          | Tailwind CSS 4                      |
| HTTP Client      | Axios                               |
| Forms            | React Hook Form + Zod validation    |
| Routing          | React Router DOM 7                  |

### Infrastructure

| Component       | Technology                          |
| --------------- | ----------------------------------- |
| Containerization | Docker + Docker Compose             |
| Frontend Server  | Nginx (production)                  |
| Database         | PostgreSQL 18 Alpine                |

---

## Architecture

```
┌──────────────┐       ┌──────────────────┐       ┌──────────────┐
│   Frontend   │──────▶│     Backend      │──────▶│  PostgreSQL  │
│  React + TS  │ :3000 │  Spring Boot 4   │ :8080 │   Database   │
│   (Nginx)    │       │   REST API       │       │    :5432     │
└──────────────┘       └──────────────────┘       └──────────────┘
```

- **Frontend** runs on port `3000` (Nginx in production, Vite dev server in development)
- **Backend** runs on port `8080` and exposes a REST API under `/api`
- **PostgreSQL** runs on port `5432` (production) — H2 in-memory is used in development
- In development, Vite proxies `/api` requests to `http://localhost:8080`

---

## Prerequisites

### Docker (production / quick start)

- [Docker](https://docs.docker.com/get-docker/) and Docker Compose

### Local development

- Java 21+
- Node.js 20+
- npm
- Maven (or use the included `./mvnw` wrapper)

---

## Getting Started

### Run with Docker (recommended)

Start all services (PostgreSQL, backend, frontend) with a single command from the project root:

```bash
docker compose up --build
```

| Service    | URL                          |
| ---------- | ---------------------------- |
| Frontend   | http://localhost:3000         |
| Backend    | http://localhost:8080         |
| Swagger UI | http://localhost:8080/swagger-ui/index.html |

To stop all services:

```bash
docker compose down
```

To also remove the database volume:

```bash
docker compose down -v
```

### Run Locally (development)

#### 1. Start the backend

```bash
cd backend
./mvnw spring-boot:run -Dspring-boot.run.profiles=dev
```

This starts the backend with the **dev** profile (H2 in-memory database). No external database needed.

#### 2. Start the frontend

```bash
cd frontend
npm install
npm run dev
```

The frontend dev server starts at http://localhost:3000 and proxies API calls to the backend.

#### Run backend with PostgreSQL locally

If you prefer PostgreSQL during development:

1. Start PostgreSQL using the backend's compose file:

```bash
cd backend
docker compose up -d
```

2. Run the backend with the prod profile:

```bash
./mvnw spring-boot:run -Dspring-boot.run.profiles=prod
```

### Build for production

**Backend:**

```bash
cd backend
./mvnw clean package
java -jar target/backend-0.0.1-SNAPSHOT.jar
```

**Frontend:**

```bash
cd frontend
npm run build
```

The production build outputs to `frontend/dist/` and can be served by any static file server.

---

## Configuration

### Backend Profiles

| Profile | File                          | Database              | JWT Access Expiry |
| ------- | ----------------------------- | --------------------- | ----------------- |
| `dev`   | `application-dev.properties`  | H2 in-memory          | 60 seconds        |
| `prod`  | `application-prod.properties` | PostgreSQL             | 24 hours          |

The active profile defaults to `prod` in `application.properties`. Pass `-Dspring-boot.run.profiles=dev` to use the dev profile (H2 in-memory database) locally.

### Key Backend Properties

```properties
# JWT configuration
jwt.secret=<min-32-character-secret>
jwt.access.expiration=86400000    # access token lifetime (ms)
jwt.refresh.expiration=7          # refresh token lifetime (days)

# Database (prod)
spring.datasource.url=jdbc:postgresql://postgres:5432/db
spring.datasource.username=docker
spring.datasource.password=docker
```

### Docker Environment Variables (PostgreSQL)

| Variable            | Default  |
| ------------------- | -------- |
| `POSTGRES_USER`     | `docker` |
| `POSTGRES_PASSWORD` | `docker` |
| `POSTGRES_DB`       | `db`     |

---

## API Reference

Base URL: `/api`

### Authentication — `/api/auth`

| Method | Endpoint              | Description                | Auth Required |
| ------ | --------------------- | -------------------------- | ------------- |
| POST   | `/api/auth/register`  | Register a new user        | No            |
| POST   | `/api/auth/login`     | Login and receive tokens   | No            |
| POST   | `/api/auth/refresh`   | Refresh access token       | No            |
| POST   | `/api/auth/logout`    | Invalidate refresh token   | No            |

**Register request:**

```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "secret123"
}
```

**Login request:**

```json
{
  "email": "john@example.com",
  "password": "secret123"
}
```

**Auth response:**

```json
{
  "accessToken": "<jwt>",
  "refreshToken": "<uuid>",
  "email": "john@example.com",
  "name": "John Doe",
  "message": "User registered"
}
```

**Refresh request:**

```json
{
  "refreshToken": "<uuid>"
}
```

---

### Tasks — `/api/tasks`

All endpoints require `Authorization: Bearer <accessToken>` header. Users can only access their own tasks.

| Method | Endpoint                    | Description                                  |
| ------ | --------------------------- | -------------------------------------------- |
| POST   | `/api/tasks`                | Create a new task                            |
| GET    | `/api/tasks`                | List authenticated user's tasks              |
| GET    | `/api/tasks/{id}`           | Get a task by ID                             |
| PATCH  | `/api/tasks/{id}/infos`     | Update title, description, priority, due date |
| PATCH  | `/api/tasks/{id}/status`    | Update task status                           |
| DELETE | `/api/tasks/{id}`           | Delete a task                                |

**Create/update task body:**

```json
{
  "title": "Buy groceries",
  "description": "Milk, eggs, bread",
  "priority": "HIGH",
  "status": "PENDING",
  "dueDate": "2026-03-01T10:00:00"
}
```

**Enums:**

| Enum     | Values                        |
| -------- | ----------------------------- |
| Priority | `HIGH`, `MEDIUM`, `LOW`       |
| Status   | `PENDING`, `COMPLETED`        |

---

### Users (Admin) — `/api/users`

| Method | Endpoint                      | Description          | Role Required   |
| ------ | ----------------------------- | -------------------- | --------------- |
| POST   | `/api/users`                  | Create a user        | ADMIN           |
| GET    | `/api/users`                  | List all users       | ADMIN           |
| GET    | `/api/users/email/{email}`    | Find user by email   | ADMIN           |
| DELETE | `/api/users/{id}`             | Delete a user        | ADMIN           |
| PATCH  | `/api/users/{id}/password`    | Update password by ID | Authenticated   |
| PATCH  | `/api/users/me/password`      | Update own password  | Authenticated   |

---

## Authentication Flow

The application uses **stateless JWT authentication** with a refresh token strategy:

```
1. User logs in
   POST /api/auth/login  →  { accessToken (JWT), refreshToken (UUID) }

2. Authenticated requests
   Authorization: Bearer <accessToken>

3. Access token expires
   POST /api/auth/refresh  { refreshToken: "<uuid>" }  →  new accessToken

4. Logout
   POST /api/auth/logout  { refreshToken: "<uuid>" }  →  token invalidated
```

- **Access tokens** are short-lived JWTs signed with HMAC-SHA, carrying `email` and `role` claims
- **Refresh tokens** are UUIDs stored in the database with an expiration date
- The frontend automatically intercepts 401 responses and attempts a silent token refresh before redirecting to login

---

## Default Admin Account

An admin user is seeded automatically on first startup:

| Field    | Value             |
| -------- | ----------------- |
| Email    | `admin@admin.com` |
| Password | `adminadmin`      |
| Role     | `ADMIN`           |

> **Change the default admin password if necessary.**

---

## Frontend Routes

| Path                | Component        | Access           | Description                  |
| ------------------- | ---------------- | ---------------- | ---------------------------- |
| `/login`            | Login            | Public           | User login form              |
| `/register`         | Register         | Public           | User registration form       |
| `/`                 | Dashboard        | Authenticated    | Task list grouped by priority |
| `/update-password`  | UpdatePassword   | Authenticated    | Change password              |
| `/admin/dashboard`  | AdminDashboard   | Admin only       | Stats overview               |
| `/admin/users`      | AdminUsers       | Admin only       | User management              |

### Dashboard Features

- Summary cards showing pending and completed task counts
- Tasks grouped by priority (HIGH / MEDIUM / LOW) with color-coded gradients
- Toggle task completion with one click
- Edit and delete tasks with confirmation dialogs
- Floating action button on mobile for creating tasks

### Admin Dashboard

- Total users (split by role), total tasks, pending tasks (with overdue count)
- Task completion rate progress bar

### Admin Users

- Search users by email, filter by role
- Create new users with assigned roles
- Delete users with confirmation (warns when deleting admin accounts)

---

## Project Structure

```
do-list/
├── docker-compose.yml              # Full-stack Docker orchestration
├── README.md
├── backend/
│   ├── docker-compose.yml          # PostgreSQL only (for local dev)
│   ├── Dockerfile                  # Backend container (Java 21)
│   ├── pom.xml                     # Maven dependencies
│   ├── mvnw / mvnw.cmd            # Maven wrapper
│   └── src/main/java/com/backend/
│       ├── BackendApplication.java
│       ├── config/
│       │   ├── AdminSeed.java      # Seeds default admin on startup
│       │   ├── OpenApiConfig.java  # Swagger configuration
│       │   └── SecurityConfig.java # CORS, JWT filter, role rules
│       ├── controller/
│       │   ├── AuthController.java # Registration, login, token refresh
│       │   ├── TaskController.java # CRUD operations for tasks
│       │   └── UserController.java # Admin user management
│       ├── dto/
│       │   ├── request/            # Incoming request DTOs
│       │   └── response/           # Outgoing response DTOs
│       ├── exception/              # Custom exceptions + global handler
│       ├── model/
│       │   ├── core/               # BaseEntity (audit fields)
│       │   ├── entity/             # User, Task, RefreshToken entities
│       │   └── enums/              # Role, Priority, Status
│       ├── repository/             # Spring Data JPA repositories
│       ├── security/
│       │   └── JwtAuthFilter.java  # JWT request filter
│       └── service/                # Business logic (interfaces + impl)
├── frontend/
│   ├── Dockerfile                  # Multi-stage build (Node → Nginx)
│   ├── package.json
│   ├── vite.config.ts              # Dev server + API proxy config
│   └── src/
│       ├── App.tsx                 # Route definitions
│       ├── components/
│       │   ├── AddTaskModal.tsx    # Task creation modal
│       │   ├── CreateUserModal.tsx # Admin user creation modal
│       │   ├── EditTaskModal.tsx   # Task editing modal
│       │   ├── Login.tsx           # Login form with Zod validation
│       │   ├── Navbar.tsx          # Navigation with role-aware links
│       │   ├── PasswordInput.tsx   # Password field with visibility toggle
│       │   └── Register.tsx        # Registration form
│       ├── context/
│       │   ├── AdminContext.tsx    # Admin state (users list)
│       │   ├── AuthContext.tsx     # Auth state (tokens, user, login/logout)
│       │   └── TaskContext.tsx     # Task CRUD state management
│       ├── pages/
│       │   ├── AdminDashboard.tsx  # Admin stats overview
│       │   ├── AdminUsers.tsx      # User management page
│       │   ├── Dashboard.tsx       # Main task dashboard
│       │   └── UpdatePassword.tsx  # Password change page
│       └── services/
│           ├── adminApi.ts        # Admin API calls
│           ├── api.ts             # Axios instance + interceptors
│           ├── taskApi.ts         # Task API calls
│           └── userApi.ts         # User API calls
```

---

## API Documentation (Swagger)

When the backend is running, interactive API documentation is available at:

| Resource       | URL                                          |
| -------------- | -------------------------------------------- |
| Swagger UI     | http://localhost:8080/swagger-ui/index.html   |
| OpenAPI JSON   | http://localhost:8080/v3/api-docs             |

---

## H2 Console (dev only)

Available when running with the dev profile:

| Property   | Value                  |
| ---------- | ---------------------- |
| URL        | http://localhost:8080/h2-console |
| JDBC URL   | `jdbc:h2:mem:test_db`  |
| Username   | `sa`                   |
| Password   | *(empty)*              |

---

## Testing

Run backend tests:

```bash
cd backend
./mvnw test
```

Test reports are generated in `backend/target/surefire-reports/`.

Run frontend lint:

```bash
cd frontend
npm run lint
``` -->
