# 🚗 Vehicle Booking System — Secure REST API

> A production-minded RESTful backend for browsing, booking, and managing vehicles — built with **Spring Boot**, **Spring Security + JWT**, **Spring Data JPA**, and **MySQL**, with **role-based access control** separating public, user, and admin capabilities.

![Java](https://img.shields.io/badge/Java-17+-ED8B00?logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.x-6DB33F?logo=springboot&logoColor=white)
![Spring Security](https://img.shields.io/badge/Spring%20Security-JWT-6DB33F?logo=springsecurity&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.x-4479A1?logo=mysql&logoColor=white)
![Postman](https://img.shields.io/badge/Docs-Postman-FF6C37?logo=postman&logoColor=white)

The API follows a strict **layered architecture** (Controller → Service → Repository → Entity), authenticates every protected request with a **stateless JWT**, and enforces **authorization by role** so that customers can only manage their own bookings while administrators manage inventory and platform-wide data.

---

## 📌 Table of Contents

1. [Features](#-features)
2. [Tech Stack](#-tech-stack)
3. [Project Architecture](#-project-architecture)
4. [Authentication & Authorization](#-authentication--authorization)
5. [Directory Structure](#-directory-structure)
6. [API Endpoints](#-api-endpoints)
7. [Request & Response Examples](#-request--response-examples)
8. [Setup & Installation](#-setup--installation)
9. [Design Decisions](#-design-decisions)

---

## ✨ Features

- **Stateless JWT authentication** — register, log in, and receive a Bearer token for all protected routes
- **Role-based access control (RBAC)** — three access tiers: public, authenticated user, and admin
- **Admin vehicle management**
  - Add new vehicles with pricing, booking amount, image URL, and stock count
  - Update existing vehicle details (including remaining units)
  - View all registered users and all bookings across the platform
- **User booking workflow**
  - Book a vehicle with a chosen delivery date
  - Cancel an existing booking by ID
  - View personal booking history with vehicle details and status
- **Public vehicle catalogue** — browse available vehicles without authentication
- **Inventory tracking** — each vehicle carries a `remainingUnits` count
- **Layered, testable codebase** — business logic isolated from HTTP and persistence concerns
- **Persistent storage** — MySQL via Spring Data JPA with automatic schema management

---

## 🛠 Tech Stack

### Backend
- **Java 17+**
- **Spring Boot** (Spring Web)
- **Spring Security** with **JWT** (JSON Web Tokens)
- **Spring Data JPA / Hibernate** (ORM)
- **Maven** (build & dependency management)

### Database
- **MySQL 8.x**
- **MySQL Connector/J** (JDBC driver)

### API Tooling
- **Postman** (collection and API documentation)

---

## 🏗 Project Architecture

Every request passes through a JWT security filter before reaching the controller layer. Each layer depends only on the layer directly beneath it.

```text
┌──────────────────────────────┐
│  Client (Postman / Frontend) │   HTTP + JSON
└───────────────┬──────────────┘
                │  Authorization: Bearer <JWT>
                ▼
┌──────────────────────────────┐
│  Security Filter Chain       │   Validates JWT, resolves user & role,
│  (JWT Authentication Filter) │   rejects unauthorized requests (401/403)
└───────────────┬──────────────┘
                ▼
┌──────────────────────────────┐
│  Controller Layer            │   Routing, request/response mapping
│  (@RestController)           │   /login  /register  /vehicle  /auth/**
└───────────────┬──────────────┘
                │  delegates
                ▼
┌──────────────────────────────┐
│  Service Layer               │   Business rules: stock checks, booking
│  (@Service)                  │   creation/cancellation, token issuing
└───────────────┬──────────────┘
                │  calls
                ▼
┌──────────────────────────────┐
│  Repository Layer            │   Data access abstraction
│  (JpaRepository)             │   (Spring Data JPA)
└───────────────┬──────────────┘
                │  Hibernate / JDBC
                ▼
┌──────────────────────────────┐
│  MySQL Database              │   users · vehicles · bookings
└──────────────────────────────┘
```

---

## 🔐 Authentication & Authorization

Routes are grouped by access level:

| Access Level | Route Pattern | Requirement |
|:------------:|---------------|-------------|
| 🌐 Public | `/login`, `/register`, `GET /vehicle` | None |
| 👤 User | `/auth/user/**` | Valid JWT |
| 🛡 Admin | `/auth/admin/**` | Valid JWT with admin role |

**Flow**

1. `POST /register` — create an account.
2. `POST /login` — submit email and password; receive a JWT.
3. Send the token on every protected request:

```http
Authorization: Bearer <your_jwt_token>
```

Requests with a missing or invalid token are rejected before reaching any controller, and user-level tokens cannot access admin routes.

---

## 📁 Directory Structure

```text
vehicle-booking-system/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── vehiclebooking/
│   │   │           ├── VehicleBookingApplication.java   # Spring Boot entry point
│   │   │           ├── controller/
│   │   │           │   ├── AuthController.java          # /login, /register
│   │   │           │   ├── AdminController.java         # /auth/admin/**
│   │   │           │   ├── UserController.java          # /auth/user/**
│   │   │           │   └── VehicleController.java       # /vehicle (public)
│   │   │           ├── service/
│   │   │           │   ├── AuthService.java
│   │   │           │   ├── VehicleService.java
│   │   │           │   └── BookingService.java
│   │   │           ├── repository/
│   │   │           │   ├── UserRepository.java
│   │   │           │   ├── VehicleRepository.java
│   │   │           │   └── BookingRepository.java
│   │   │           ├── entity/
│   │   │           │   ├── User.java
│   │   │           │   ├── Vehicle.java
│   │   │           │   └── Booking.java
│   │   │           ├── dto/                             # Request/response payloads
│   │   │           └── security/
│   │   │               ├── SecurityConfig.java          # Route rules & filter chain
│   │   │               ├── JwtUtil.java                 # Token generation & validation
│   │   │               └── JwtAuthenticationFilter.java
│   │   └── resources/
│   │       └── application.properties                   # Server, DB & JWT config
│   └── test/
│       └── java/com/vehiclebooking/                     # Unit / integration tests
├── pom.xml
└── README.md
```

---

## 🔗 API Endpoints

**Base URL:** `http://localhost:8080`

### Public

| Method | Endpoint | Description |
|:------:|----------|-------------|
| `POST` | `/register` | Registers a new user; returns the created user with a confirmation message. |
| `POST` | `/login` | Authenticates credentials and returns a JWT Bearer token. |
| `GET`  | `/vehicle` | Returns all vehicles in the catalogue. |

### User (`ROLE_USER`)

| Method | Endpoint | Description |
|:------:|----------|-------------|
| `POST` | `/auth/user/bookVehicle` | Books a vehicle (`vehicleId`, `deliveryDate`) for the authenticated user; returns booking confirmation. |
| `POST` | `/auth/user/cancelBooking/{bookingId}` | Cancels the booking with the given ID; returns a confirmation message. |
| `GET`  | `/auth/user/bookedVehicles` | Returns all bookings made by the authenticated user, with vehicle details, dates, and status. |

### Admin (`ROLE_ADMIN`)

| Method | Endpoint | Description |
|:------:|----------|-------------|
| `POST` | `/auth/admin/addVehicle` | Adds a new vehicle; returns the created vehicle with its assigned ID. |
| `POST` | `/auth/admin/updateVehicle` | Updates an existing vehicle identified by `id`; returns the updated vehicle. |
| `GET`  | `/auth/admin/users` | Returns all registered users (ID, name, email, role). |
| `GET`  | `/auth/admin/bookings` | Returns all bookings across the platform with user details, vehicle info, dates, and status. |

---

## 📨 Request & Response Examples

### Register

`POST /register`

```json
{
  "name": "John Doe",
  "email": "user@example.com",
  "password": "yourpassword",
  "phoneNumber": "12345678"
}
```

### Login

`POST /login`

```json
{
  "email": "user@example.com",
  "password": "yourpassword"
}
```

Returns a JWT to be sent as `Authorization: Bearer <token>`.

### Add Vehicle (Admin)

`POST /auth/admin/addVehicle`

```json
{
  "make": "Honda",
  "model": "Sp Shine 125",
  "year": 2026,
  "price": 125000,
  "bookingAmount": 5000,
  "imageUrl": "https://example.com/images/sp-shine-125.jpg",
  "remainingUnits": 42
}
```

### Update Vehicle (Admin)

`POST /auth/admin/updateVehicle`

```json
{
  "id": 1,
  "remainingUnits": 34
}
```

### Book Vehicle (User)

`POST /auth/user/bookVehicle`

```json
{
  "vehicleId": 2,
  "deliveryDate": "2026-10-07"
}
```

### cURL quick start

```bash
# 1. Log in and copy the token
curl -X POST http://localhost:8080/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"yourpassword"}'

# 2. Call a protected endpoint
curl http://localhost:8080/auth/user/bookedVehicles \
  -H "Authorization: Bearer <your_jwt_token>"
```

---

## 🚀 Setup & Installation

### Prerequisites

| Requirement | Version |
|-------------|---------|
| JDK         | 17 or higher |
| Maven       | 3.8+ |
| MySQL       | 8.x |
| Git         | any recent version |
| Postman     | optional, for API testing |

```bash
java -version
mvn -version
mysql --version
```

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/<your-repo-name>.git
cd <your-repo-name>
```

### 2. Create the MySQL database

```sql
CREATE DATABASE vehicle_booking_db;
```

> Tables are created automatically by Hibernate on first run.

### 3. Configure `application.properties`

Edit `src/main/resources/application.properties`:

```properties
# ---------- Server ----------
server.port=8080

# ---------- MySQL Datasource ----------
spring.datasource.url=jdbc:mysql://localhost:3306/vehicle_booking_db
spring.datasource.username=root
spring.datasource.password=your_mysql_password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# ---------- JPA / Hibernate ----------
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect

# ---------- JWT ----------
jwt.secret=replace_with_a_long_random_secret_of_at_least_32_characters
jwt.expiration-ms=86400000
```

**Security tip:** never commit real secrets. Externalize them with environment variables:

```properties
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
jwt.secret=${JWT_SECRET}
```

### 4. Build and run

```bash
mvn clean install
mvn spring-boot:run
```

Or run the packaged JAR:

```bash
java -jar target/*.jar
```

The API is now available at **http://localhost:8080**.

### 5. Verify the setup

```bash
curl http://localhost:8080/vehicle
```

An empty array or a list of vehicles confirms the server and database are connected.

### 6. Create an admin account

New accounts are registered through `/register`. To use the admin endpoints, set the role of your account to admin in the `users` table, then log in again to receive an admin-scoped token:

```sql
UPDATE users SET role = 'ROLE_ADMIN' WHERE email = 'your_email@example.com';
```

### Troubleshooting

| Issue | Fix |
|-------|-----|
| `401 Unauthorized` | Token missing, expired, or malformed. Log in again and send `Authorization: Bearer <token>`. |
| `403 Forbidden` | Your role does not allow this route, e.g. a user token on an admin endpoint. |
| `Communications link failure` | Confirm MySQL is running and the datasource URL is correct. |
| `Access denied for user` | Re-check the database username and password. |
| `Port 8080 already in use` | Change `server.port` or stop the conflicting process. |

---

## 🧠 Design Decisions

- **Stateless authentication:** JWT keeps the API horizontally scalable, with no server-side session storage.
- **Route-level authorization:** the `/auth/user/**` and `/auth/admin/**` prefixes let the security configuration enforce roles declaratively instead of repeating checks inside controllers.
- **Separation of concerns:** controllers handle transport, services own business rules (stock validation, booking lifecycle), repositories own persistence.
- **Repository abstraction:** `JpaRepository` removes boilerplate DAO code and keeps the data layer easy to test and swap.
- **Documented contract:** the full request collection is documented in Postman for fast onboarding and manual testing.

---

## 👤 Author

**Ramnivas**
Final-year Engineering Student · Java & Spring Boot Developer

🔗 GitHub: (https://github.com/rampra4)

---

⭐ If you found this project useful, consider giving it a star!
