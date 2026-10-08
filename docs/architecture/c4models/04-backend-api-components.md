# C4 Model - Level 3: Backend API Components

```mermaid
C4Component
title Clinic Appointment System - Backend API Components

Container_Boundary(api, "Backend API - Modular Monolith") {
    Component(restApi, "REST Controllers", "API Layer", "Exposes HTTP endpoints, validates request shape, and maps responses/errors")
    Component(access, "Authentication & Access Control", "Application/Security", "Authenticates requests and enforces role/ownership rules")
    Component(patientModule, "Patient Module", "Application + Domain + Infrastructure", "Manages patient registration, lookup, status, and profile data")
    Component(doctorModule, "Doctor Module", "Application + Domain + Infrastructure", "Manages doctors, specialties, schedules, and slot offering")
    Component(appointmentModule, "Appointment Module", "Application + Domain + Infrastructure", "Owns booking, cancellation, reschedule, appointment lifecycle, and visit status")
    Component(notificationModule, "Notification Module", "Application + Domain + Infrastructure", "Creates notification intents, templates, and reminder scheduling")
    Component(asyncOrchestrator, "Async Orchestrator", "Infrastructure", "Executes reminder/notification workflows in-process after transactional commit")
    Component(auditQueries, "Audit & Reporting Queries", "Read/Application", "Builds audit trails and reporting-oriented read queries")
}

ContainerDb(db, "PostgreSQL", "Transactional database", "Stores business data and audit data")
Container(redis, "Redis", "Operational cache", "Supports short-lived cache entries and operational metadata")
System_Ext(sms, "SMS Provider")
System_Ext(email, "Email Provider")

Rel(restApi, access, "Delegates auth and authorization checks")
Rel(restApi, patientModule, "Calls")
Rel(restApi, doctorModule, "Calls")
Rel(restApi, appointmentModule, "Calls")
Rel(restApi, notificationModule, "Calls when needed")
Rel(restApi, auditQueries, "Calls read-oriented queries")

Rel(appointmentModule, patientModule, "Reads patient eligibility via internal contracts")
Rel(appointmentModule, doctorModule, "Validates slot/doctor availability via internal contracts")
Rel(appointmentModule, notificationModule, "Requests booking/cancellation/reschedule notification intents")
Rel(notificationModule, asyncOrchestrator, "Delegates async reminder/notification execution")

Rel(patientModule, db, "Reads/Writes")
Rel(doctorModule, db, "Reads/Writes")
Rel(appointmentModule, db, "Reads/Writes with transactional consistency")
Rel(notificationModule, db, "Reads/Writes notification data")
Rel(auditQueries, db, "Reads")
Rel(asyncOrchestrator, redis, "Reads/Writes operational cache entries")
Rel(asyncOrchestrator, sms, "Sends SMS notifications")
Rel(asyncOrchestrator, email, "Sends email notifications")
```

## Components

| Component | Responsibility |
| --- | --- |
| `REST Controllers` | request validation, endpoint mapping, response contracts, status/error translation |
| `Authentication & Access Control` | actor identity, role checks, ownership checks, security boundaries |
| `Patient Module` | patient profile and business identity ownership |
| `Doctor Module` | doctor, specialty, schedule, and slot ownership |
| `Appointment Module` | booking core, cancellation, reschedule, lifecycle and visit tracking |
| `Notification Module` | notification intent creation, reminder orchestration, delivery preparation |
| `Async Orchestrator` | اجرای async داخلی پس از commit تراکنش و هماهنگی ارسال اعلان |
| `Audit & Reporting Queries` | read models and audit-oriented projections |

## Dependency Rules

- `REST Controllers` به domain داخلی مستقیم متصل نمی‌شوند و فقط از application-facing contracts استفاده می‌کنند.
- `Appointment Module` تنها ماژولی است که outcome نهایی booking را تعیین می‌کند.
- `Doctor Module` منبع حقیقت `Slot` و availability است.
- `Notification Module` downstream است و rule تصمیم‌گیری رزرو را در خود تکرار نمی‌کند.
- `Async Orchestrator` بعد از موفقیت عملیات دامنه فراخوانی می‌شود و نباید موفقیت تراکنش رزرو را قبل از commit نهایی اعلام کند.

## Notes

- invariantهای ضد `double booking` در این container و persistence layer enforce می‌شوند.
- `Redis` lock authority نیست و منبع حقیقت رزرو محسوب نمی‌شود.
- اگر در آینده decomposition رسمی لایه‌ها انجام شود، هر module باید به زیرلایه‌های `API`, `Application`, `Domain`, `Infrastructure` شکسته شود بدون شکستن مرز bounded contextها.
