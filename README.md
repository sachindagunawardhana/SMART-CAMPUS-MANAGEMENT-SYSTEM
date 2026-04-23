# 🏫 Smart Campus Management System (SCMS)
## Spring Boot REST API — Complete Implementation

---

## 📐 1. SYSTEM DESIGN OVERVIEW

### Architecture: Layered (Clean) Architecture

```
┌──────────────────────────────────────────────────┐
│              Presentation Layer                   │
│     REST Controllers  ←→  DTOs (Request/Response)│
├──────────────────────────────────────────────────┤
│               Business Logic Layer                │
│  Services (Interfaces + Impls) + Facade + Factory │
├──────────────────────────────────────────────────┤
│               Data Access Layer                   │
│      Spring Data JPA Repositories                 │
├──────────────────────────────────────────────────┤
│               Database Layer                      │
│         H2 (dev) / MySQL (production)             │
└──────────────────────────────────────────────────┘
```

**Cross-cutting:**
- Security Layer: JWT + Spring Security (stateless)
- Exception Layer: GlobalExceptionHandler
- Observer Layer: Notification events across services

---

## 📁 2. PROJECT FOLDER STRUCTURE

```
smart-campus-management/
├── pom.xml
└── src/
    ├── main/
    │   ├── java/com/scms/
    │   │   ├── SmartCampusManagementApplication.java
    │   │   ├── config/
    │   │   │   ├── SecurityConfig.java          # JWT + Spring Security config
    │   │   │   └── DataSeeder.java              # Demo data on startup
    │   │   ├── controller/
    │   │   │   ├── AuthController.java          # POST /api/auth/login, /register
    │   │   │   ├── RoomController.java          # /api/rooms/**
    │   │   │   ├── BookingController.java       # /api/bookings/**
    │   │   │   ├── MaintenanceController.java   # /api/maintenance/**
    │   │   │   ├── NotificationController.java  # /api/notifications/**
    │   │   │   └── AnalyticsController.java     # /api/analytics/**
    │   │   ├── dto/
    │   │   │   ├── AuthDto.java
    │   │   │   ├── RoomDto.java
    │   │   │   ├── BookingDto.java
    │   │   │   ├── MaintenanceDto.java
    │   │   │   ├── NotificationDto.java
    │   │   │   └── AnalyticsDto.java
    │   │   ├── entity/
    │   │   │   ├── User.java        (abstract base)
    │   │   │   ├── Admin.java
    │   │   │   ├── Staff.java
    │   │   │   ├── Student.java
    │   │   │   ├── Room.java
    │   │   │   ├── Booking.java
    │   │   │   ├── MaintenanceRequest.java
    │   │   │   └── Notification.java
    │   │   ├── enums/
    │   │   │   ├── UserRole.java
    │   │   │   ├── RoomType.java
    │   │   │   ├── BookingStatus.java
    │   │   │   ├── MaintenanceStatus.java
    │   │   │   ├── Priority.java
    │   │   │   └── NotificationType.java
    │   │   ├── exception/
    │   │   │   ├── GlobalExceptionHandler.java
    │   │   │   ├── InvalidBookingException.java
    │   │   │   ├── UnauthorizedAccessException.java
    │   │   │   ├── DuplicateDataException.java
    │   │   │   ├── ResourceNotFoundException.java
    │   │   │   └── SystemException.java
    │   │   ├── repository/
    │   │   │   ├── UserRepository.java
    │   │   │   ├── RoomRepository.java
    │   │   │   ├── BookingRepository.java
    │   │   │   ├── MaintenanceRequestRepository.java
    │   │   │   └── NotificationRepository.java
    │   │   ├── security/
    │   │   │   ├── JwtUtils.java
    │   │   │   ├── JwtAuthFilter.java
    │   │   │   └── UserDetailsServiceImpl.java
    │   │   └── service/
    │   │       ├── UserService.java (interface)
    │   │       ├── RoomService.java (interface)
    │   │       ├── BookingService.java (interface)
    │   │       ├── MaintenanceService.java (interface)
    │   │       ├── NotificationService.java (interface)
    │   │       ├── NotificationObserver.java (Observer interface)
    │   │       ├── NotificationPublisher.java (Subject interface)
    │   │       └── impl/
    │   │           ├── UserServiceImpl.java    ← Factory Pattern
    │   │           ├── RoomServiceImpl.java
    │   │           ├── BookingServiceImpl.java
    │   │           ├── MaintenanceServiceImpl.java
    │   │           ├── NotificationServiceImpl.java ← Observer Pattern
    │   │           ├── BookingFacade.java      ← Facade Pattern
    │   │           └── AnalyticsService.java   ← Bonus analytics
    │   └── resources/
    │       └── application.properties
    └── test/
        ├── java/com/scms/
        │   ├── ScmsIntegrationTest.java
        │   └── service/
        │       ├── BookingServiceTest.java     (10 test cases)
        │       ├── MaintenanceServiceTest.java  (6 test cases + 1 intentional fail)
        │       ├── NotificationServiceTest.java (6 test cases)
        │       └── RoomServiceTest.java         (6 test cases)
        └── resources/
            └── application-test.properties
```

---

## 🎨 3. DESIGN PATTERNS

### Creational — Factory Method (UserServiceImpl)
```
UserServiceImpl.createUserByRole(request, role)
  → ADMIN   → new Admin(...)
  → STAFF   → new Staff(...)
  → STUDENT → new Student(...)
```
**Why:** New user types can be added by adding a `case` without touching callers — Open/Closed Principle.

### Structural — Facade (BookingFacade)
```
BookingFacade
  ├── bookRoom(request)       → coordinates RoomService + BookingService
  ├── findAvailableRooms(...) → delegates to RoomService
  └── cancelBooking(...)      → delegates to BookingService
```
**Why:** Controllers stay thin. Multi-step workflows are hidden behind one entry point.

### Behavioral — Observer (NotificationServiceImpl)
```
NotificationPublisher (Subject)
  └── notifyObservers(type, user, title, msg, refId, refType)
        ├── NotificationServiceImpl.onEvent() → saves to DB
        ├── EmailObserver.onEvent()            → sends email
        └── SMSObserver.onEvent()              → sends SMS
```
**Why:** Adding new notification channels (Push, Slack) requires zero changes to existing services.

---

## 🔑 4. API ENDPOINTS REFERENCE

### Authentication (Public)
| Method | Endpoint               | Description          |
|--------|------------------------|----------------------|
| POST   | /api/auth/login        | Get JWT token        |
| POST   | /api/auth/register     | Register new user    |

### Rooms
| Method | Endpoint                        | Role        |
|--------|---------------------------------|-------------|
| GET    | /api/rooms                      | All         |
| GET    | /api/rooms/active               | All         |
| GET    | /api/rooms/{id}                 | All         |
| GET    | /api/rooms/available?start&end  | All         |
| POST   | /api/rooms/available/filter     | All         |
| POST   | /api/rooms                      | ADMIN       |
| PUT    | /api/rooms/{id}                 | ADMIN       |
| PUT    | /api/rooms/{id}/deactivate      | ADMIN       |
| PUT    | /api/rooms/{id}/activate        | ADMIN       |

### Bookings
| Method | Endpoint                     | Role              |
|--------|------------------------------|-------------------|
| POST   | /api/bookings                | All               |
| GET    | /api/bookings/my             | All               |
| GET    | /api/bookings/{id}           | All               |
| GET    | /api/bookings                | ADMIN             |
| GET    | /api/bookings/room/{roomId}  | ADMIN / STAFF     |
| PUT    | /api/bookings/{id}/confirm   | ADMIN             |
| PUT    | /api/bookings/{id}/cancel    | Owner or ADMIN    |

### Maintenance
| Method | Endpoint                          | Role              |
|--------|-----------------------------------|-------------------|
| POST   | /api/maintenance                  | All               |
| GET    | /api/maintenance/my               | All               |
| GET    | /api/maintenance/{id}             | All               |
| GET    | /api/maintenance                  | ADMIN / STAFF     |
| GET    | /api/maintenance/assigned         | ADMIN / STAFF     |
| PUT    | /api/maintenance/{id}/assign      | ADMIN             |
| PUT    | /api/maintenance/{id}/status      | Assignee / ADMIN  |

### Notifications
| Method | Endpoint                         | Role  |
|--------|----------------------------------|-------|
| GET    | /api/notifications               | All   |
| GET    | /api/notifications/unread        | All   |
| GET    | /api/notifications/unread/count  | All   |
| PUT    | /api/notifications/{id}/read     | All   |
| PUT    | /api/notifications/read-all      | All   |

### Analytics (Admin only)
| Method | Endpoint              | Description         |
|--------|-----------------------|---------------------|
| GET    | /api/analytics        | Full dashboard stats |

---

## 🚀 5. QUICK START GUIDE

### Prerequisites
- Java 17+
- Maven 3.8+
- Git

### Step 1 — Clone & Build
```bash
cd smart-campus-management
mvn clean package -DskipTests
```

### Step 2 — Run
```bash
mvn spring-boot:run
# OR
java -jar target/smart-campus-management-1.0.0.jar
```

The application starts on **http://localhost:8080**

H2 Console: **http://localhost:8080/h2-console**
- JDBC URL: `jdbc:h2:mem:scmsdb`
- Username: `sa` | Password: *(blank)*

### Step 3 — Login (Demo Credentials)
```bash
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@scms.com","password":"Admin@123"}'
```
Response: `{"token":"eyJ...", "role":"ADMIN", ...}`

### Step 4 — Use the Token
```bash
TOKEN="eyJ..."

# Create a room
curl -X POST http://localhost:8080/api/rooms \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "roomNumber":"Z100",
    "roomName":"New Room",
    "roomType":"SEMINAR_ROOM",
    "capacity":30
  }'

# Book a room
curl -X POST http://localhost:8080/api/bookings \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "roomId":1,
    "startTime":"2026-06-01T09:00:00",
    "endTime":"2026-06-01T11:00:00",
    "purpose":"Team meeting",
    "attendeesCount":15
  }'
```

### Step 5 — Run Tests
```bash
# All tests
mvn test

# Specific test class
mvn test -Dtest=BookingServiceTest

# With verbose output
mvn test -Dtest=BookingServiceTest -Dsurefire.failIfNoSpecifiedTests=false
```

---

## 🗄️ 6. DATABASE SCHEMA (Auto-Generated by Hibernate)

```sql
-- Users (single-table inheritance)
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_type VARCHAR(31),      -- discriminator: ADMIN / STAFF / STUDENT
    first_name VARCHAR(255) NOT NULL,
    last_name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    role VARCHAR(50) NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    department VARCHAR(255),
    employee_id VARCHAR(255),
    position VARCHAR(255),
    student_id VARCHAR(255),
    faculty VARCHAR(255),
    year_of_study INT,
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

-- Rooms
CREATE TABLE rooms (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    room_number VARCHAR(255) UNIQUE NOT NULL,
    room_name VARCHAR(255) NOT NULL,
    room_type VARCHAR(50) NOT NULL,
    capacity INT NOT NULL,
    building VARCHAR(255),
    floor_number INT,
    has_projector BOOLEAN,
    has_whiteboard BOOLEAN,
    has_air_conditioning BOOLEAN,
    is_active BOOLEAN DEFAULT TRUE,
    description VARCHAR(500),
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

-- Bookings
CREATE TABLE bookings (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    room_id BIGINT REFERENCES rooms(id),
    user_id BIGINT REFERENCES users(id),
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP NOT NULL,
    purpose VARCHAR(300),
    status VARCHAR(50) NOT NULL,
    attendees_count INT,
    cancellation_reason VARCHAR(300),
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

-- Maintenance requests
CREATE TABLE maintenance_requests (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description VARCHAR(1000) NOT NULL,
    priority VARCHAR(50) NOT NULL,
    status VARCHAR(50) NOT NULL,
    room_id BIGINT REFERENCES rooms(id),
    reported_by_id BIGINT REFERENCES users(id),
    assigned_to_id BIGINT REFERENCES users(id),
    resolution_notes VARCHAR(500),
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    resolved_at TIMESTAMP
);

-- Notifications
CREATE TABLE notifications (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    message VARCHAR(500) NOT NULL,
    notification_type VARCHAR(50) NOT NULL,
    recipient_id BIGINT REFERENCES users(id),
    is_read BOOLEAN DEFAULT FALSE,
    reference_id BIGINT,
    reference_type VARCHAR(50),
    created_at TIMESTAMP,
    read_at TIMESTAMP
);
```

---

## 🧪 7. TEST SUMMARY

| Test File                   | Test Cases | Notes                              |
|-----------------------------|------------|------------------------------------|
| BookingServiceTest          | 10         | Includes double-booking & edge cases |
| MaintenanceServiceTest      | 6          | TC06 = **intentional failing test** |
| NotificationServiceTest     | 6          | Tests observer registration        |
| RoomServiceTest             | 6          | CRUD + deactivation tests          |
| ScmsIntegrationTest         | 3          | Full Spring context                |

**Intentional Failing Test:** `MaintenanceServiceTest.TC06`
- Expects status `"ASSIGNED"` but actual is `"OPEN"`
- Purpose: demonstrates that the test harness correctly detects wrong assertions

---

## 🔧 8. SWITCHING TO MySQL (Production)

In `application.properties`, comment out H2 and uncomment MySQL:
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/scmsdb?useSSL=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=yourpassword
spring.jpa.database-platform=org.hibernate.dialect.MySQLDialect
spring.jpa.hibernate.ddl-auto=update
```

Create the database:
```sql
CREATE DATABASE scmsdb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

---

## 👥 9. DEMO USER CREDENTIALS

| Role    | Email                    | Password     |
|---------|--------------------------|--------------|
| Admin   | admin@scms.com           | Admin@123    |
| Staff   | staff@scms.com           | Staff@123    |
| Staff   | jane.staff@scms.com      | Staff@123    |
| Student | student@scms.com         | Student@123  |
| Student | Kiyon.student@scms.com     | Student@123  |

---

## 📊 10. SOLID PRINCIPLES APPLIED

| Principle | Where Applied                                          |
|-----------|--------------------------------------------------------|
| **S**RP   | Each class has one job (Service, Controller, Repository) |
| **O**CP   | Factory Method: new user types without modifying caller |
| **L**SP   | Admin/Staff/Student safely substitute for User          |
| **I**SP   | NotificationObserver and NotificationPublisher are separate narrow interfaces |
| **D**IP   | Controllers depend on service interfaces, not impls     |
