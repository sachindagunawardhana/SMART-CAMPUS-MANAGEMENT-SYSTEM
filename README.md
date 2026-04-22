# 🏛️ Smart Campus Management System (SCMS)

A full-stack **Spring Boot 3 + HTML/JS** application for managing campus rooms,
bookings, maintenance requests, and notifications.

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│  HTML/JS Frontend (Vanilla JS, Fetch API, CSS Grid)     │
│  index.html · dashboard · rooms · bookings ·            │
│  maintenance · notifications · users · analytics        │
└─────────────────────┬───────────────────────────────────┘
                      │ REST (JSON + JWT Bearer Token)
┌─────────────────────▼───────────────────────────────────┐
│  Spring Boot 3 Backend                                  │
│                                                         │
│  Controllers (REST API)                                 │
│    AuthController · RoomController · BookingController  │
│    MaintenanceController · NotificationController       │
│    AdminController                                      │
│                                                         │
│  Services (Business Logic + Design Patterns)            │
│    RoomService       ← FACTORY Pattern                  │
│    BookingService    ← FACADE Pattern                   │
│    NotificationService ← OBSERVER Pattern               │
│    MaintenanceService · AnalyticsService · UserService  │
│                                                         │
│  Repositories (Spring Data JPA)                         │
│    UserRepository · RoomRepository · BookingRepository  │
│    MaintenanceRequestRepository · NotificationRepository│
│                                                         │
│  Security (JWT Stateless)                               │
│    JwtUtils · JwtAuthFilter · ScmsUserDetailsService    │
└─────────────────────┬───────────────────────────────────┘
                      │ JPA / Hibernate
┌─────────────────────▼───────────────────────────────────┐
│  H2 In-Memory DB (default) | MySQL (production)         │
└─────────────────────────────────────────────────────────┘
```

---

## 🎨 Design Patterns

| Pattern | Type | Class | Why |
|---|---|---|---|
| **Factory** | Creational | `RoomFactory` | Centralises Room creation with type-specific defaults (projector for halls, AC for labs) |
| **Facade** | Structural | `BookingService` | Single entry point hiding conflict detection, validation, persistence + notification dispatch |
| **Observer** | Behavioral | `NotificationService` | Decouples event producers (booking/maintenance) from notification consumers; new channels (email, SMS) just implement `NotificationObserver` |
| **Singleton** | Creational | Spring `@Service` beans | Framework-managed singleton lifecycle |

---

## 🔐 Role-Based Access

| Feature | Admin | Staff | Student |
|---|:---:|:---:|:---:|
| View Rooms | ✅ | ✅ | ✅ |
| Create/Edit Rooms | ✅ | ❌ | ❌ |
| Book Rooms | ✅ | ✅ | ✅ |
| View All Bookings | ✅ | ❌ | ❌ |
| Cancel Any Booking | ✅ | ❌ | ❌ |
| Report Maintenance | ✅ | ✅ | ✅ |
| Assign Maintenance | ✅ | ❌ | ❌ |
| Update Status | ✅ | ✅ | ❌ |
| User Management | ✅ | ❌ | ❌ |
| Analytics | ✅ | ❌ | ❌ |

---

## 🚀 Quick Start

### Prerequisites
- Java 17+
- Maven 3.8+

### 1. Clone / Extract the project
```bash
cd scms
```

### 2. Run the application
```bash
mvn spring-boot:run
```

The app starts on **http://localhost:8080**

### 3. Open in browser
```
http://localhost:8080
```

### 4. Demo Login Credentials
| Role | Username | Password |
|---|---|---|
| Admin | `admin` | `admin123` |
| Staff | `staff` | `staff123` |
| Student | `student` | `student123` |

---

## 🧪 Running Tests

```bash
# Run all tests
mvn test

# Run with verbose output
mvn test -Dtest=ScmsUnitTests -pl . -Dsurefire.useFile=false
```

### Test Cases

| # | Test | Expected |
|---|---|---|
| TC01 | Create booking — happy path | ✅ Pass |
| TC02 | Double booking prevention | ✅ Pass |
| TC03 | Past-time booking rejected | ✅ Pass |
| TC04 | Capacity exceeded rejected | ✅ Pass |
| TC05 | Cancel — unauthorized user | ✅ Pass |
| TC06 | Duplicate room number | ✅ Pass |
| TC07 | Maintenance request creation | ✅ Pass |
| TC08 | Assign — non-admin rejected | ✅ Pass |
| TC09 | Notification fires on booking | ✅ Pass |
| TC10 | **Intentionally Failing** — wrong status | ❌ Fail (by design) |

---

## 📁 Project Structure

```
scms/
├── pom.xml
└── src/
    ├── main/
    │   ├── java/com/scms/
    │   │   ├── SmartCampusApplication.java
    │   │   ├── config/
    │   │   │   ├── SecurityConfig.java       # JWT + CORS security
    │   │   │   └── DataSeeder.java           # Demo data on startup
    │   │   ├── controller/                   # REST endpoints
    │   │   │   ├── AuthController.java
    │   │   │   ├── RoomController.java
    │   │   │   ├── BookingController.java
    │   │   │   ├── MaintenanceController.java
    │   │   │   ├── NotificationController.java
    │   │   │   └── AdminController.java
    │   │   ├── dto/                          # Request/Response objects
    │   │   │   ├── AuthDTO.java
    │   │   │   ├── RoomDTO.java
    │   │   │   ├── BookingDTO.java
    │   │   │   ├── MaintenanceDTO.java
    │   │   │   ├── NotificationDTO.java
    │   │   │   └── AnalyticsDTO.java
    │   │   ├── exception/                    # Custom exceptions
    │   │   │   ├── SystemException.java
    │   │   │   ├── InvalidBookingException.java
    │   │   │   ├── UnauthorizedAccessException.java
    │   │   │   ├── DuplicateDataException.java
    │   │   │   ├── ResourceNotFoundException.java
    │   │   │   └── GlobalExceptionHandler.java
    │   │   ├── model/                        # JPA Entities
    │   │   │   ├── User.java (abstract)
    │   │   │   ├── Room.java
    │   │   │   ├── Booking.java
    │   │   │   ├── MaintenanceRequest.java
    │   │   │   └── Notification.java
    │   │   ├── repository/                   # Spring Data JPA
    │   │   ├── security/                     # JWT classes
    │   │   │   ├── JwtUtils.java
    │   │   │   ├── JwtAuthFilter.java
    │   │   │   └── ScmsUserDetailsService.java
    │   │   └── service/
    │   │       ├── NotificationObserver.java  # Observer interface
    │   │       ├── NotificationSubject.java   # Subject interface
    │   │       └── impl/
    │   │           ├── RoomFactory.java       # Factory Pattern
    │   │           ├── RoomService.java
    │   │           ├── BookingService.java    # Facade Pattern
    │   │           ├── MaintenanceService.java
    │   │           ├── NotificationService.java # Observer Pattern
    │   │           ├── UserService.java
    │   │           └── AnalyticsService.java
    │   └── resources/
    │       ├── application.properties
    │       └── static/
    │           ├── index.html               # Login page
    │           ├── css/app.css              # Shared styles
    │           ├── js/
    │           │   ├── api.js               # API client + helpers
    │           │   └── layout.js            # Sidebar renderer
    │           └── pages/
    │               ├── dashboard.html
    │               ├── rooms.html
    │               ├── bookings.html
    │               ├── maintenance.html
    │               ├── notifications.html
    │               ├── users.html           # Admin only
    │               └── analytics.html       # Admin only
    └── test/
        └── java/com/scms/
            └── ScmsUnitTests.java
```

---

## 🌐 API Reference

### Auth
| Method | Endpoint | Access |
|---|---|---|
| POST | `/api/auth/login` | Public |
| POST | `/api/auth/register` | Public |

### Rooms
| Method | Endpoint | Access |
|---|---|---|
| GET | `/api/rooms` | All |
| GET | `/api/rooms/available` | All |
| POST | `/api/rooms` | Admin |
| PUT | `/api/rooms/{id}` | Admin |
| PATCH | `/api/rooms/{id}/activate` | Admin |
| PATCH | `/api/rooms/{id}/deactivate` | Admin |
| PATCH | `/api/rooms/{id}/maintenance` | Admin |

### Bookings
| Method | Endpoint | Access |
|---|---|---|
| GET | `/api/bookings` | Admin |
| GET | `/api/bookings/my` | Self |
| GET | `/api/bookings/upcoming` | Self |
| POST | `/api/bookings` | All |
| DELETE | `/api/bookings/{id}/cancel` | Owner/Admin |

### Maintenance
| Method | Endpoint | Access |
|---|---|---|
| GET | `/api/maintenance` | Admin |
| GET | `/api/maintenance/my` | Self |
| GET | `/api/maintenance/assigned` | Staff/Admin |
| POST | `/api/maintenance` | All |
| PATCH | `/api/maintenance/{id}/assign` | Admin |
| PATCH | `/api/maintenance/{id}/status` | Staff/Admin |

### Admin
| Method | Endpoint | Access |
|---|---|---|
| GET | `/api/admin/users` | Admin |
| GET | `/api/admin/analytics` | Admin |
| PATCH | `/api/admin/users/{id}/activate` | Admin |

---

## 🗄️ Switch to MySQL (Production)

In `application.properties`, comment H2 and uncomment MySQL:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/scmsdb?useSSL=false
spring.datasource.username=root
spring.datasource.password=yourpassword
spring.datasource.driverClassName=com.mysql.cj.jdbc.Driver
spring.jpa.database-platform=org.hibernate.dialect.MySQL8Dialect
spring.jpa.hibernate.ddl-auto=update
```

Add MySQL dependency to `pom.xml`:
```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
</dependency>
```

---

## 🔧 H2 Console (Development)

Access the in-memory database at:
```
http://localhost:8080/h2-console
JDBC URL:  jdbc:h2:mem:scmsdb
Username:  sa
Password:  (empty)
```

---

## ✅ SOLID Principles Applied

- **S**ingle Responsibility — each service handles one domain
- **O**pen/Closed — `NotificationObserver` can be extended without modifying `NotificationService`
- **L**iskov Substitution — `User` subclasses used interchangeably
- **I**nterface Segregation — `NotificationSubject` and `NotificationObserver` are separate interfaces
- **D**ependency Inversion — services depend on repository interfaces, not implementations
[README.md](https://github.com/user-attachments/files/26979641/README.md)
