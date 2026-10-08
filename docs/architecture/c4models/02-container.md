
```mermaid

C4Container
title Clinic Appointment System - Container Diagram

Person(patient, "Patient")
Person(doctor, "Doctor")
Person(receptionist, "Receptionist")
Person(clinicAdmin, "Clinic Admin")
Person(systemAdmin, "System Admin")

System_Ext(sms, "SMS Provider")
System_Ext(email, "Email Provider")

Container(web, "Web App", "SPA", "Unified web UI for patients, doctors, receptionists, clinic admins, and system admins")
Container(api, "Backend API", "Java 21 + Spring Boot 3 + Spring Modulith", "Exposes REST endpoints and contains Patient, Doctor, Appointment, and Notification modules")
ContainerDb(db, "PostgreSQL", "Transactional relational database", "Stores business data, audit data, and configuration")
Container(redis, "Redis", "Operational cache", "Caches short-lived operational data such as hot reads, rate-limit windows, and token/session metadata")

Rel(patient, web, "Uses")
Rel(doctor, web, "Uses")
Rel(receptionist, web, "Uses")
Rel(clinicAdmin, web, "Uses")
Rel(systemAdmin, web, "Uses")

Rel(web, api, "Calls REST API over HTTPS")

Rel(api, db, "Reads/Writes transactional data")
Rel(api, redis, "Reads/Writes short-lived cache entries")
Rel(api, sms, "Sends SMS notifications")
Rel(api, email, "Sends email notifications")
```

## Notes

- `Payment Gateway` از container diagram حذف شده چون در MVP خارج از scope است.
- `Redis` در baseline فعلی منبع حقیقت رزرو نیست و فقط برای cache عملیاتی استفاده می‌شود.
- پردازش‌های async مانند reminder/notification داخل همان `Backend API` (runtime مشترک Modular Monolith) اجرا می‌شوند و در MVP به worker مستقل Node.js نیاز نداریم.
- invariantهای جلوگیری از double booking در لایه `Backend API` و پایگاه داده تراکنشی enforce می‌شوند.