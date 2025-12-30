# PG / Hostel Management System

## High-Level Design (HLD) – Architect View (25+ Years Experience)

---

## 1. Purpose of This Document

This document is written so that **even a fresher or junior developer** can:

* Understand WHAT the system does
* Understand HOW the system works end-to-end
* Know WHERE each responsibility lives
* Start coding confidently without confusion

This HLD intentionally avoids buzzwords and focuses on **clarity, flow, and reasoning**.

---

## 2. Problem Statement (In Simple Words)

Managing PGs/Hostels manually leads to:

* No bed-level visibility
* Confusion in tenant allocation
* Rent leakage and disputes
* Poor complaint tracking

This system digitizes **everything from bed to payment** in a structured, scalable way.

---

## 3. Who Uses the System (Actors)

1. **Owner** – Owns one or more PGs/Hostels
2. **Manager** – Operates PG on behalf of owner (optional)
3. **Tenant** – Stays in a bed
4. **System** – Automates rent cycles, status updates

---

## 4. High-Level Architecture Overview

### 4.1 Architecture Style

**Layered Architecture (Industry Standard)**

```
Client (Web / Mobile / Postman)
        ↓
REST API Layer (Controllers)
        ↓
Business Layer (Services)
        ↓
Persistence Layer (Repositories)
        ↓
Database (RDBMS)
```

### Why this architecture?

* Easy to understand
* Easy to test
* Easy to scale
* Easy for juniors to follow

---

## 5. Component Responsibility (Very Important)

### 5.1 Controller Layer (WHAT APIs DO)

* Accept HTTP requests
* Validate input
* Call service layer
* Return responses

❌ No business logic here

---

### 5.2 Service Layer (BRAIN OF SYSTEM)

* Business rules
* Validations
* Transaction handling
* Workflow orchestration

Example:

* “Assign bed only if available”
* “Mark bed OCCUPIED when tenant joins”

---

### 5.3 Repository Layer (DATABASE ACCESS)

* Only CRUD operations
* No business logic
* Talks only to database

---

## 6. Core Domain Model (Mental Model)

```
Owner
 └── Hostel (PG)
      └── Floor
           └── Room
                └── Bed
                     └── Tenant
```

### Key Design Decision

**Bed is the smallest rentable unit** – this avoids future redesign.

---

## 7. End-to-End System Flow (Big Picture Flowchart)

```
Owner Registers
      ↓
Creates Hostel
      ↓
Creates Floors
      ↓
Creates Rooms
      ↓
Creates Beds
      ↓
Tenant Onboarded
      ↓
Bed Assigned
      ↓
Rent Cycle Starts
      ↓
Complaint / Maintenance (Optional)
```

This is the **entire system lifecycle**.

---

## 8. Detailed Flowcharts

### 8.1 Hostel Setup Flow (Owner Perspective)

```
Start
  ↓
Login as Owner
  ↓
Create Hostel
  ↓
Add Floors
  ↓
Add Rooms per Floor
  ↓
Add Beds per Room
  ↓
Hostel Ready for Tenants
  ↓
End
```

---

### 8.2 Tenant Onboarding Flow

```
Start
  ↓
Register Tenant
  ↓
Check Bed Availability
  ↓
Is Bed Available?
   ├─ No → Show Error
   └─ Yes
        ↓
Assign Bed
        ↓
Update Bed Status = OCCUPIED
        ↓
Create Rent Record
        ↓
Tenant Successfully Onboarded
        ↓
End
```

---

## 9. Sequence Diagram – Tenant Onboarding

```
Client → TenantController : POST /tenants
TenantController → TenantService : onboardTenant()
TenantService → BedService : checkAvailability()
BedService → BedRepository : findById()
BedRepository → BedService : Bed
BedService → TenantService : AVAILABLE
TenantService → TenantRepository : save(Tenant)
TenantService → BedRepository : updateStatus(OCCUPIED)
TenantService → RentRepository : createRent()
TenantService → Controller : Success Response
```

This shows **who calls whom and why**.

---

## 10. Rent Payment Flow

```
System Triggers Monthly Cycle
      ↓
Fetch Active Tenants
      ↓
Generate Rent Due
      ↓
Tenant Pays Rent
      ↓
Update Payment Status
      ↓
Generate Report
```

---

## 11. Complaint Flow

```
Tenant Raises Complaint
      ↓
Complaint Assigned Status = OPEN
      ↓
Manager Updates Status
      ↓
RESOLVED / CLOSED
```

---

## 12. Non-Functional Design (Why This Will Scale)

* Stateless REST APIs
* DB normalization
* Clear transaction boundaries
* Easy horizontal scaling
* Clean separation of concerns

---

## 13. Design Principles Used (Explained Simply)

* **Single Responsibility** – each class does one thing
* **Loose Coupling** – layers don’t depend directly
* **High Cohesion** – related logic stays together
* **Fail Fast** – validate early

---

## 14. How a Junior Developer Should Read This Project

1. Read this HLD once
2. Understand entity hierarchy
3. Follow flows (Section 7–11)
4. Then start coding module by module

---

## 15. Visual ER Diagram (Explained for Beginners)

### 15.1 ER Diagram – Textual Visual Representation

```
USER (Owner / Manager / Tenant)
 └── user_id (PK)
 └── role
 └── name
 └── phone

USER (Owner)
   1
   |
   | owns
   |
   *
HOSTEL
 └── hostel_id (PK)
 └── name
 └── location
 └── owner_id (FK → USER)

HOSTEL
   1
   |
   | has
   |
   *
FLOOR
 └── floor_id (PK)
 └── floor_number
 └── hostel_id (FK)

FLOOR
   1
   |
   | contains
   |
   *
ROOM
 └── room_id (PK)
 └── room_number
 └── capacity
 └── floor_id (FK)

ROOM
   1
   |
   | contains
   |
   *
BED
 └── bed_id (PK)
 └── bed_number
 └── status (AVAILABLE / OCCUPIED)
 └── room_id (FK)

BED
   1
   |
   | assigned to
   |
   0..1
TENANT
 └── tenant_id (PK)
 └── name
 └── phone
 └── bed_id (FK)
 └── joining_date

TENANT
   1
   |
   | has
   |
   *
RENT_PAYMENT
 └── payment_id (PK)
 └── tenant_id (FK)
 └── month
 └── amount
 └── status
```

---

### 15.2 ER Design Rules (Very Important)

* One **Owner → Many Hostels**
* One **Hostel → Many Floors**
* One **Floor → Many Rooms**
* One **Room → Many Beds**
* One **Bed → At most ONE Tenant** (critical rule)
* One **Tenant → Many Rent Payments**

This ensures **no data duplication and no ambiguity**.

---

## 16. Proper Sequence Diagram – Tenant Onboarding (UML-style)

### 16.1 Actors & Components

* Actor: Owner / Manager
* UI Client
* TenantController
* TenantService
* BedService
* TenantRepository
* BedRepository
* RentRepository
* Database

---

### 16.2 UML Sequence Diagram (Readable & Industry-Standard)

```
Owner/Manager
     |
     | 1. Submit Tenant Details + BedId
     v
UI Client
     |
     | 2. POST /api/tenants
     v
TenantController
     |
     | 3. onboardTenant(request)
     v
TenantService
     |
     | 4. validateRequest()
     |
     | 5. checkBedAvailability(bedId)
     v
BedService
     |
     | 6. findBedById(bedId)
     v
BedRepository
     |
     | 7. return Bed(status)
     v
BedService
     |
     | 8. if status != AVAILABLE → throw Exception
     |
     | 9. return AVAILABLE
     v
TenantService
     |
     | 10. save Tenant
     v
TenantRepository
     |
     | 11. Tenant Saved
     v
TenantService
     |
     | 12. update Bed status = OCCUPIED
     v
BedRepository
     |
     | 13. Bed Updated
     v
TenantService
     |
     | 14. create Rent Record
     v
RentRepository
     |
     | 15. Rent Saved
     v
TenantService
     |
     | 16. commit Transaction
     v
TenantController
     |
     | 17. return 201 CREATED
     v
UI Client
```

---

### 16.3 Transaction Boundary (CRITICAL)

* Steps **10 → 15** run in **ONE database transaction**
* If any step fails → rollback everything
* Prevents:

    * Partial tenant creation
    * Bed occupied without tenant

---

### 16.2 Why This Sequence Matters

* Prevents double bed allocation
* Guarantees transactional consistency
* Clear ownership of responsibilities

---

## 17. Visual Sequence – Rent Payment Flow

```
Scheduler / Tenant
      |
      | Generate / Pay Rent
      v
RentController
      |
      v
RentService
      |
      v
RentRepository
      |
      v
Database
```