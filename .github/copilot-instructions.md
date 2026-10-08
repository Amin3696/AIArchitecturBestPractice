# Copilot Instructions - Clinic Appointment System MVP

**Last Updated:** 2026-10-08  
**Version:** 1.0  
**Status:** Active Baseline

> This document provides AI assistants (Copilot) with comprehensive context for implementing, reviewing, or extending the clinic appointment system MVP. All code generation, documentation updates, and architectural decisions should align with this baseline.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Technology Stack](#technology-stack)
3. [Architectural Patterns](#architectural-patterns)
4. [API Contract & Standards](#api-contract--standards)
5. [Database & Concurrency Model](#database--concurrency-model)
6. [Access Control & Security](#access-control--security)
7. [Module Structure](#module-structure)
8. [Engineering Delivery Flow](#engineering-delivery-flow)
9. [Known Ambiguities](#known-ambiguities)
10. [Documentation Rules](#documentation-rules)
11. [Common Code Generation Prompts](#common-code-generation-prompts)

---

## Project Overview

### What

A clinic appointment scheduling system enabling patients to book, reschedule, and cancel appointments; doctors to manage schedules; and clinic staff to oversee operations.

### Core Business Value

Eliminate double-booking of appointment slots, enforce role-based access control, and maintain data integrity under concurrent user load using ACID transactions.

### Key Constraints

- **Team Size:** 3-5 developers
- **Deployment:** Single monolithic JAR (modular internally)
- **Initial Scope:** MVP with 4 bounded contexts (Patient, Doctor, Appointment, Notification)
- **Primary Database:** PostgreSQL (source of truth)
- **Cache:** Redis (operational cache only, NOT authority for bookings)

### Success Criteria

1. ✓ Double-booking prevention via unique constraints
2. ✓ Role-based access control enforced at application layer
3. ✓ Idempotent booking operations (via `Idempotency-Key` header)
4. ✓ Audit trail for sensitive operations
5. ✓ Modular code structure that can evolve to microservices

---

## Technology Stack

### Official Stack (ADR-0003)

| Component | Version | Rationale |
|-----------|---------|-----------|
| **Language** | Java 21 | Modern, LTS, strong typing, mature tooling |
| **Framework** | Spring Boot 3.x | Ecosystem maturity, Spring Modulith support |
| **Modularity** | Spring Modulith | Enforce internal boundaries without microservices |
| **Primary DB** | PostgreSQL 15+ | ACID guarantees, unique constraints, JSON support |
| **Cache** | Redis 7.x | Operational cache, TTL management, in-memory operations |
| **Build Tool** | Maven 3.9.x (or Gradle 8.x) | Dependency management, CI integration |
| **Testing** | JUnit 5 + Testcontainers | Container-based integration tests |

### Key Rules

- **PostgreSQL is the source of truth** for all persistent data (appointments, users, slots).
- **Redis is operational cache only**—never rely on Redis as authority for booking/reservation data.
- **No separate worker process** in MVP; async operations (reminders, notifications) execute in-process post-commit.
- **Java 21 features** (records, sealed classes, text blocks) are encouraged for clean code.

### Dependency Exclusions

- ❌ Do NOT introduce Node.js, TypeScript, or BullMQ
- ❌ Do NOT use .NET or other JVM languages without ADR update
- ❌ Do NOT add microservice patterns (Kafka, gRPC) without explicit decision

---

## Architectural Patterns

### Modular Monolith (ADR-0001)

The system is **one deployable artifact** with **four internal modules**:

```
clinic-appointment-system/
├── patient/           # Patient identity & profile
├── doctor/            # Doctor schedule & availability
├── appointment/       # Core business: bookings & lifecycle
└── notification/      # Async notifications & reminders
```

### Module Interaction Rules

**NO direct dependency on internal implementations.** Modules communicate via:

1. **Domain Events** (async, publish-subscribe)
2. **Query Contracts** (read-only interfaces)
3. **Command Contracts** (well-defined APIs)
4. **Anti-Corruption Layer** (if data translation needed)

#### Example: Appointment booking calls Doctor module

```java
// ❌ WRONG: Direct dependency on Doctor internals
DoctorRepository doctorRepo = doctorContext.getRepository();
Doctor doctor = doctorRepo.findById(doctorId);

// ✅ CORRECT: Use published Doctor contract
SlotQuery slotQuery = doctorContext.getSlotQueryService();
AvailableSlotDto slot = slotQuery.findSlot(slotId);
```

### Bounded Contexts

| Context | Responsibility | Upstream Dependencies | Events Published |
|---------|---------------|-----------------------|-------------------|
| **Patient** | Profile mgmt, identity | None | `PatientRegistered`, `PatientUpdated` |
| **Doctor** | Schedule, slot offering | None | `ScheduleChanged`, `SlotDeactivated` |
| **Appointment** | Booking, cancellation, rescheduling | Patient, Doctor | `AppointmentBooked`, `AppointmentCancelled`, `AppointmentRescheduled` |
| **Notification** | SMS/Email sending | Appointment, Doctor | None (terminal context) |

### Shared Kernel (Minimal)

Only these types are shared across modules:

- `PatientId`, `DoctorId`, `AppointmentId`, `SlotId` (ULID or UUID)
- Enum constants: `AppointmentStatus`, `SlotStatus`, `VisitStatus`, `Role`

**Location:** `clinic-shared/` or `shared-kernel/` package (centralized, version-controlled)

---

## API Contract & Standards

### Base URL & Authentication

```
Base URL: /api/v1
Auth: Bearer <accessToken> (JWT)
Trace: X-Request-Id: <uuid>
```

### Response Envelope

**Success:**
```json
{
  "data": { /* endpoint-specific object */ }
}
```

**Error:**
```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable message",
    "details": [
      {
        "field": "fieldName",
        "reason": "validation_failed"
      }
    ],
    "requestId": "4a53b672-0bc8-4780-b4cc-d0fc18f149a0"
  }
}
```

### Idempotency Contract

For sensitive operations (`POST /appointments`, `POST /appointments/{id}/cancel`):

- **Header:** `Idempotency-Key: <uuid>`
- **Requirement:** Server must store the key and return same response for duplicate requests
- **TTL:** 24 hours (configurable)
- **Response:** `200 OK` (retry of same key) or `201 Created` (first attempt)

### Business Error Codes

| Code | Meaning | HTTP Status |
|------|---------|-------------|
| `APPOINTMENT_SLOT_UNAVAILABLE` | Slot already booked or inactive | 409 |
| `APPOINTMENT_CANCELLATION_WINDOW_CLOSED` | Cancellation deadline passed | 422 |
| `AUTH_INVALID_CREDENTIALS` | Login failed | 401 |
| `AUTH_RESET_TOKEN_EXPIRED` | Password reset token expired | 410 |
| `PATIENT_MOBILE_ALREADY_EXISTS` | Phone number registered | 409 |
| `VALIDATION_FAILED` | Request payload invalid | 422 |
| `AUTHORIZATION_DENIED` | User lacks permission | 403 |

**Full list:** See `docs/api/mvp-api-contract.md` and `docs/api/openapi-v1.yaml`

### API Standards Compliance

All endpoints must follow `docs/api/rest-api-standards.md`:

- Versioned paths (`/api/v1/...`)
- Consistent error envelope
- Idempotency where applicable
- Proper HTTP status codes
- Optional fields in request body
- Pagination for list endpoints (`limit`, `offset`)

---

## Database & Concurrency Model

### Primary Data Model

**Key Entities:**

- `Patient`: User identity, contact info, status
- `Doctor`: Profile, specialties, contact info
- `Schedule`: Recurring doctor availability (week patterns, exceptions)
- `Slot`: Individual booking capacity (`AVAILABLE` → `RESERVED` → `INACTIVE` or `EXPIRED`)
- `Appointment`: Booking record (`BOOKED` → `COMPLETED` / `CANCELLED` / `NO_SHOW`)
- `Visit`: In-clinic check-in (`NOT_STARTED` → `WAITING` → `IN_PROGRESS` → `COMPLETED` / `NO_SHOW`)

### Double-Booking Prevention

**Mechanism:** Unique constraint on `(slot_id, appointment_status = 'BOOKED')`

```sql
CREATE UNIQUE INDEX idx_slot_booked_appointment 
  ON appointment(slot_id) 
  WHERE status = 'BOOKED';
```

**Database Transaction Level:** `READ_COMMITTED` (PostgreSQL default)

**Java Implementation:** Use `@Transactional(isolation = Isolation.READ_COMMITTED)` with explicit pessimistic locking if needed:

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("SELECT s FROM Slot s WHERE s.id = :slotId")
Optional<Slot> findSlotForUpdate(@Param("slotId") String slotId);
```

### State Machine Rules

#### Appointment Status
```
BOOKED → { COMPLETED, CANCELLED, NO_SHOW }
(No revisiting BOOKED after transition)
```

#### Slot Status
```
AVAILABLE → { RESERVED (appointment created), INACTIVE (doctor canceled), EXPIRED (time passed) }
```

#### Visit Status
```
NOT_STARTED → WAITING → IN_PROGRESS → { COMPLETED, NO_SHOW }
```

### Timezone Handling

- **Storage:** All datetimes in UTC in PostgreSQL (`timestamp with time zone`)
- **Interpretation:** System timezone = `Asia/Tehran` (configurable in `application.properties`)
- **Display:** Convert to user's timezone on API response
- **Slot Slot time:** Interpreted in clinic timezone (Asia/Tehran)

```java
// Example
LocalDateTime slotTime = appointment.getSlotTime();  // UTC in DB
ZonedDateTime tehranTime = slotTime
  .atZone(ZoneId.of("Asia/Tehran"));  // Convert for display
```

### Cache Invalidation Policy (Redis)

- **Appointment data:** Do NOT cache (always read from PostgreSQL)
- **Slot availability:** Cache with 5-minute TTL (readable cache, not authoritative)
- **Doctor schedule:** Cache with 1-hour TTL
- **Patient profile:** Cache with 30-minute TTL
- **Clear on mutation:** Always invalidate related keys after INSERT/UPDATE/DELETE

```java
@CacheEvict(value = "doctorSchedule", key = "#doctorId")
public void updateSchedule(String doctorId, Schedule schedule) {
  // ...
}
```

---

## Access Control & Security

### Role-Based Access Control (RBAC)

**Five Roles:**

| Role | Responsibilities | Ownership Rules |
|------|-----------------|-----------------|
| `PATIENT` | View/edit own profile, manage own appointments | Only own data |
| `DOCTOR` | Define schedule, manage own patients' visits | Only own schedule/appointments |
| `RECEPTIONIST` | Book/cancel/reschedule for any patient, override policies | Can override time windows |
| `CLINIC_ADMIN` | Full operational + user management | All clinic data |
| `SYSTEM_ADMIN` | Technical, security, audit access | Platform-wide, not business actor |

### Authorization Enforcement Points

**All endpoints must enforce authorization at application layer:**

```java
@GetMapping("/appointments/{id}")
public AppointmentDto getAppointment(@PathVariable String id) {
  Appointment appointment = appointmentService.findById(id);
  
  // Enforce ownership
  if (!appointment.ownedBy(currentUser.getId())) {
    throw new AuthorizationException("Unauthorized");
  }
  
  return toDto(appointment);
}
```

### Override Capability

- **Receptionist** can cancel/reschedule outside policy windows (logged as override)
- **Clinic Admin** inherits all Receptionist overrides
- **All overrides must be audited** with actor, reason, timestamp, before/after state

### Password Reset & Session Management

- **Session TTL:** 15 minutes for access token, 30 days for refresh token
- **Password Reset:** OTP via SMS (TBD: see FR-04 ambiguity)
- **Logout:** Invalidate both tokens and session record
- **Rate Limit:** Max 5 login attempts per phone number per 15 minutes

---

## Module Structure

### Recommended Layout (Spring Modulith Convention)

```
src/main/java/com/clinic/appointment/
├── patient/                    # Patient Module
│   ├── api/                    # REST controllers
│   │   └── PatientController.java
│   ├── application/            # Use cases
│   │   ├── RegisterPatientUseCase.java
│   │   └── GetPatientUseCase.java
│   ├── domain/                 # Domain model
│   │   ├── Patient.java        (entity/aggregate)
│   │   ├── PatientStatus.java  (value object)
│   │   └── PatientEvents.java  (domain events)
│   ├── infrastructure/         # DB/external adapters
│   │   ├── PatientRepository.java
│   │   └── PatientJpaRepository.java
│   └── package-info.java       # Spring Modulith metadata
│
├── doctor/                     # Doctor Module (similar structure)
│   ├── api/
│   ├── application/
│   ├── domain/
│   ├── infrastructure/
│   └── package-info.java
│
├── appointment/                # Appointment Module (CORE domain)
│   ├── api/
│   ├── application/
│   ├── domain/
│   ├── infrastructure/
│   └── package-info.java
│
├── notification/               # Notification Module
│   ├── api/
│   ├── application/
│   ├── domain/
│   ├── infrastructure/
│   └── package-info.java
│
├── shared/                     # Shared Kernel
│   ├── IDs.java               (PatientId, DoctorId, etc.)
│   ├── Enums.java             (Status, Role, etc.)
│   └── SharedEvents.java
│
└── config/                     # Cross-cutting config
    ├── SecurityConfig.java
    ├── CacheConfig.java
    └── AuditConfig.java
```

### Spring Modulith Package Declaration

Each module's `package-info.java`:

```java
@org.springframework.modulith.NamedModule
package com.clinic.appointment.patient;

import org.springframework.modulith.NamedModule;
```

### No Cross-Module Entity References

**Rule:** No `@ManyToOne` to entity from another module.

**Instead:** Use IDs + Query Contract

```java
// ❌ WRONG
@ManyToOne
Doctor doctor;

// ✅ CORRECT
String doctorId;

// Use service to fetch
DoctorQueryService doctorQueryService; // Injected from Doctor module
DoctorDto doctorInfo = doctorQueryService.getDoctorInfo(doctorId);
```

---

## Engineering Delivery Flow

### Phase 0: Governance & Setup

**Gate:** All analytical ambiguities must be closed.  
**Deliverables:**
- ✓ Baseline documentation reviewed and signed off
- ✓ Technology stack installed locally
- ✓ PostgreSQL + Redis running (Docker Compose)
- ✓ Build tool (Maven/Gradle) configured
- ✓ Ambiguities checklist: all 10 items resolved

**Checklist:** See `docs/requirements/ambiguities-resolution-checklist.md`

### Phase 1: Contract-First API Design

**Gate:** OpenAPI spec matches API Contract, all endpoints defined.  
**Deliverables:**
- OpenAPI 3.0.3 complete for all MVP endpoints
- Request/response schemas frozen
- Error codes & status mappings defined
- Idempotency header documented
- Mock API server generated for frontend parallel work

### Phase 2: Database Schema & Migrations

**Gate:** Schema passes validation against ADR-0002 concurrency rules.  
**Deliverables:**
- `schema.sql` or Flyway migrations created
- Unique constraints for double-booking prevention verified
- Indexes for common queries defined
- Audit table structure finalized

### Phase 3: Modular Structure & Skeleton

**Gate:** Module compilation succeeds, no cross-module package imports.  
**Deliverables:**
- Module directories created (patient, doctor, appointment, notification)
- `package-info.java` for each module
- Spring Modulith configuration in application.yml
- Empty domain model classes created
- Test structure in place

### Phase 4: Security-First Authentication & Authorization

**Gate:** Login/logout/role checks pass automated tests.  
**Deliverables:**
- JWT token generation & validation
- Password hashing (bcrypt)
- Role-based endpoint guards
- Authorization test suite
- Audit logging infrastructure

### Phase 5: Domain Modeling & State Machines

**Gate:** State transitions validated, all invariants covered by tests.  
**Deliverables:**
- Entity and Value Object implementation
- State machine logic (Appointment, Slot, Visit status)
- Domain events published correctly
- Aggregate boundary enforcement

### Phase 6: Concurrency Safety & Booking Logic

**Gate:** Double-booking prevention verified under load test.  
**Deliverables:**
- Unique constraint enforcement in code
- Transactional booking operation
- Idempotency key handling
- Pessimistic/optimistic lock strategy
- Concurrency test suite (multiple threads, same slot)

### Phase 7: Observability & Audit

**Gate:** All sensitive operations appear in audit log with actor & reason.  
**Deliverables:**
- Structured logging (SLF4J + Logback)
- Audit event capture
- Distributed tracing ready (X-Request-Id propagation)
- Operational metrics dashboard skeleton

### Phase 8: CI/CD & Release Readiness

**Gate:** Full test suite passes, deployment artifact created.  
**Deliverables:**
- GitHub Actions / GitLab CI pipeline
- Automated schema migration on startup
- Docker image build & push
- Integration test stage
- Release checklist verified

---

## Known Ambiguities

**Status:** Open — must be resolved before code generation begins.

### 1) FR-04: Password Reset Mechanism

**Question:** Which method?
- OTP via SMS + email confirmation?
- Email link only?
- Both options?

**Impact:** Authentication module design, notification templates, Twilio/email provider integration.

**Resolution:** Update `docs/requirements/functional-requirements.md` + `docs/api/mvp-api-contract.md` with final decision.

### 2) FR-06: Patient Profile Editability

**Question:** Which fields are mutable?
- Name, email, phone?
- Phone requires re-verification?

**Impact:** Patient API design, validation rules.

**Resolution Location:** `docs/requirements/functional-requirements.md`

### 3) FR-14: Doctor Schedule Definition

**Question:** Schedule model details?
- Recurrence pattern (weekly, monthly)?
- Exception override rules?
- Timezone priority?

**Impact:** Doctor domain model, Schedule entity design.

**Resolution Location:** `docs/architecture/domain/data-model-and-db-constraints.md`

### 4) FR-16: Slot Creation Policy

**Question:** How are slots created?
- Only from Schedule (auto-generated)?
- Manual creation also allowed?
- Precedence if both?

**Impact:** Slot lifecycle, Doctor module API.

**Resolution Location:** `docs/requirements/functional-requirements.md`

### 5) FR-36: Receptionist Booking Preconditions

**Question:** Can receptionist book for new (unregistered) patients?
- Must patient be pre-registered?
- Can receptionist register on-the-fly?

**Impact:** Patient registration flow, Receptionist authorization.

**Resolution Location:** `docs/api/mvp-api-contract.md` + `docs/architecture/domain/access-control-matrix.md`

### 6) FR-40: Visit Status Transitions

**Question:** Exact transition matrix?
- Who can trigger which transitions?
- Can status revert?

**Impact:** Visit state machine, API endpoints.

**Resolution Location:** `docs/architecture/domain/state-machines.md`

### 7) FR-43: Reminder Notification Timing

**Question:** When are reminders sent?
- T-24h, T-2h, T-15m?
- How many retries?
- Cutoff time for late cancellations?

**Impact:** Notification scheduling, background job design.

**Resolution Location:** `docs/requirements/functional-requirements.md`

### 8) FR-51: Audit Event Details

**Question:** What fields are audited?
- `actor`, `action`, `timestamp`, `before/after`, `reason`, `requestId`?
- What's the retention policy?
- Sensitive data masking?

**Impact:** Audit model, logging infrastructure, compliance.

**Resolution Location:** `docs/requirements/functional-requirements.md` + new `docs/architecture/domain/audit-model.md`

### 9) FR-46: Push Notifications in MVP Scope

**Question:** Are push notifications (app-based) in scope?
- Or SMS/email only?

**Impact:** Notification module capabilities, third-party integrations.

**Resolution Location:** `docs/requirements/srs.md`

### 10) FR-47/49/50: Reporting & KPI Boundaries

**Question:** What reports are MVP?
- Patient report range?
- KPI definitions?

**Impact:** Reporting module design, query optimization.

**Resolution Location:** `docs/requirements/functional-requirements.md`

---

## Documentation Rules

### Mutable vs. Immutable Documents

#### ✅ Mutable (Update freely with each iteration)

- `docs/requirements/functional-requirements.md` (resolve ambiguities)
- `docs/requirements/srs.md` (refine wording)
- `docs/api/mvp-api-contract.md` (endpoint details)
- `docs/api/openapi-v1.yaml` (schema evolution)
- `docs/architecture/domain/state-machines.md` (transition additions)
- `docs/akb-source-generation-checklist.md` (progress tracking)
- `docs/requirements/ambiguities-resolution-checklist.md` (mark items complete)

#### ❌ Immutable (Change only via formal ADR)

- `docs/architecture/decisions/ADR-*.md` (approved architecture decisions)
- `docs/akb-baseline.md` (version control document, update only for baseline version bump)
- `docs/architecture/c4models/*.md` (system architecture diagrams, change via ADR)
- `docs/architecture/domain/access-control-matrix.md` (role definitions, change via formal decision)
- `docs/architecture/domain/context-map.md` (bounded context boundaries, change via ADR)

#### 🔄 Semi-Mutable (Update with care, document rationale)

- `docs/api/rest-api-standards.md` (standards, update rare; justify deviations)
- `README.md` (public documentation, clear changelog)

### Documentation Conflict Resolution Order

(From `docs/akb-baseline.md`)

1. **ADRs** (Architecture Decision Records) — highest authority for their scope
2. **API Contract + OpenAPI** — authoritative for endpoint behavior
3. **Access Control Matrix** — authoritative for role rules
4. **State Machines** — authoritative for state transitions
5. **Baseline (akb-baseline.md)** — confirms document versions and status
6. **SRS & Functional Requirements** — source of truth for requirements
7. **Supporting documentation** (README, guides, comments) — informative, not normative

---

## Common Code Generation Prompts

### 1) Generate a Domain Entity with State Machine

**Use this prompt** when implementing Appointment or Visit entity:

```
Generate a Spring Boot entity for [Entity Name] with the following:

Attributes: [list attributes]
Status enum: [list statuses]
Allowed transitions: [state machine]
Constraints: [invariants]

Requirements:
- Use Jakarta Persistence annotations (jakarta.persistence.*)
- Emit domain events for state transitions
- Validate invariants in setters or use aggregate pattern
- Include Ulid for ID type
- Add @CreationTimestamp, @UpdateTimestamp

Reference: docs/architecture/domain/state-machines.md
```

### 2) Generate an Authorization Guard for Endpoint

**Use for endpoint security:**

```
Generate a Spring controller method with authorization guard:

Endpoint: [GET/POST/PUT/DELETE] /api/v1/[path]
Allowed roles: [PATIENT, DOCTOR, RECEPTIONIST, CLINIC_ADMIN]
Ownership check: [none, patient own record, doctor own schedule, etc.]

Requirements:
- Use Spring Security annotations (@PreAuthorize, @PostAuthorize)
- Check role first, then ownership
- Throw AuthorizationException with error code
- Log the access attempt

Reference: docs/architecture/domain/access-control-matrix.md
```

### 3) Generate Idempotent POST Endpoint

**For booking operations:**

```
Generate an idempotent POST endpoint for [operation]:

Method: POST /api/v1/[path]
Idempotency header: Idempotency-Key: <uuid>
Storage: [Redis, Database]
TTL: [24 hours, custom]

Requirements:
- Extract Idempotency-Key from request header
- Store request + response with TTL
- Return 201 on first attempt, 200 on retry
- Use same business logic regardless of attempt count

Reference: docs/api/mvp-api-contract.md
```

### 4) Generate Double-Booking Prevention Test

**For concurrency testing:**

```
Generate a JUnit 5 test verifying double-booking prevention:

Scenario: [describe concurrent booking attempts]
Slot: [slot ID]
Attempts: [number of concurrent threads]
Expected: Only one appointment succeeds, others fail with SLOT_UNAVAILABLE

Requirements:
- Use Testcontainers for PostgreSQL
- Use CountDownLatch or CyclicBarrier for thread synchronization
- Verify unique constraint violation is caught and mapped to business error
- Assert exactly one BOOKED appointment exists post-test

Reference: docs/architecture/decisions/ADR-0002-booking-consistency-and-concurrency-control.md
```

### 5) Generate Module Interface (No Internal Implementation)

**For module boundaries:**

```
Generate a public interface for [Module Name] module:

Published language: [list data types/DTOs]
Query operations: [list read-only methods]
Command operations: [list state-changing methods]
Events published: [list domain events]

Requirements:
- Use package-private implementation (@Component, not public class)
- Use public interface for contracts
- DTOs only, no entity leakage
- Document in module's package-info.java

Reference: docs/architecture/domain/context-map.md
```

---

## Quick Reference

### Key Files

| Document | Purpose | Authority |
|----------|---------|-----------|
| `docs/architecture/decisions/ADR-0001-modular-monolith-architecture.md` | Why Modular Monolith | Immutable |
| `docs/architecture/decisions/ADR-0002-booking-consistency-and-concurrency-control.md` | Booking safety model | Immutable |
| `docs/architecture/decisions/ADR-0003-technology-stack-for-mvp.md` | Tech choices & rationale | Immutable |
| `docs/api/rest-api-standards.md` | API style guide | Semi-Mutable |
| `docs/api/mvp-api-contract.md` | Endpoint definitions | Mutable |
| `docs/api/openapi-v1.yaml` | Executable API spec | Mutable |
| `docs/architecture/domain/access-control-matrix.md` | Role & permission rules | Immutable |
| `docs/architecture/domain/state-machines.md` | Status transitions | Mutable (new states) |
| `docs/architecture/domain/data-model-and-db-constraints.md` | Database schema logic | Mutable (schema evolves) |
| `docs/architecture/domain/context-map.md` | Module boundaries | Immutable |
| `docs/requirements/functional-requirements.md` | Business capabilities | Mutable (close ambiguities) |
| `docs/requirements/ambiguities-resolution-checklist.md` | Open decisions | Mutable (check off as resolved) |
| `docs/akb-baseline.md` | Version baseline | Mutable (version bumps only) |

### Common Commands

```bash
# Build with Spring Modulith checks
mvn clean verify -Dmodulith.enforceModulitarity=true

# Run tests with coverage
mvn clean test jacoco:report

# Start dev environment
docker-compose -f docker-compose.dev.yml up

# Generate OpenAPI from code
mvn springdoc-openapi-maven-plugin:generate

# Build Docker image
mvn clean package -Dskip.tests -Ddocker.build

# View module graph
mvn modulith:generate
```

### Error Code Quick Reference

| Business Scenario | Error Code | HTTP Status |
|---|---|---|
| Slot already booked | `APPOINTMENT_SLOT_UNAVAILABLE` | 409 |
| Past deadline for cancellation | `APPOINTMENT_CANCELLATION_WINDOW_CLOSED` | 422 |
| Login failed | `AUTH_INVALID_CREDENTIALS` | 401 |
| Insufficient permissions | `AUTHORIZATION_DENIED` | 403 |
| Phone already registered | `PATIENT_MOBILE_ALREADY_EXISTS` | 409 |
| Validation error | `VALIDATION_FAILED` | 422 |

---

## Getting Help

When implementing a feature:

1. **Clarify requirements:** Check `docs/requirements/functional-requirements.md` and `docs/requirements/ambiguities-resolution-checklist.md`
2. **Review architecture:** Check `docs/architecture/decisions/ADR-*.md` for design constraints
3. **Check API contract:** `docs/api/mvp-api-contract.md` for endpoint specs
4. **Verify access control:** `docs/architecture/domain/access-control-matrix.md`
5. **Test concurrency:** Use patterns from `docs/architecture/decisions/ADR-0002-booking-consistency-and-concurrency-control.md`

---

## Sign-Off

This document is the authoritative baseline for AI-assisted development on the clinic appointment system MVP. All code generation, documentation updates, and architectural decisions must align with the principles and standards documented here.

**Baseline Version:** 1.0  
**Last Updated:** 2026-10-08  
**Next Review:** Upon completion of Phase 0 (Governance & Ambiguity Resolution)

---

*End of Copilot Instructions*
