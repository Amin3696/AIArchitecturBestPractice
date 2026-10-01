
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
Container(api, "Backend API", "API-centric Modular Monolith", "Exposes REST endpoints and contains Patient, Doctor, Appointment, and Notification modules")
ContainerDb(db, "PostgreSQL", "Transactional relational database", "Stores business data, audit data, and configuration")
Container(redis, "Redis", "Redis + BullMQ backend", "Supports background job queueing and short-lived operational caching")
Container(worker, "Background Worker", "Node.js worker", "Processes notification and reminder jobs asynchronously")

Rel(patient, web, "Uses")
Rel(doctor, web, "Uses")
Rel(receptionist, web, "Uses")
Rel(clinicAdmin, web, "Uses")
Rel(systemAdmin, web, "Uses")

Rel(web, api, "Calls REST API over HTTPS")

Rel(api, db, "Reads/Writes transactional data")
Rel(api, redis, "Enqueues async jobs and uses operational cache")
Rel(worker, redis, "Consumes BullMQ jobs")
Rel(worker, db, "Reads/Writes job-relevant business data")
Rel(worker, sms, "Sends SMS notifications")
Rel(worker, email, "Sends email notifications")
```

## Notes

- `Payment Gateway` از container diagram حذف شده چون در MVP خارج از scope است.
- `Redis` در baseline فعلی منبع حقیقت رزرو نیست و به‌عنوان queue backend / cache عملیاتی مدل شده است.
- invariantهای جلوگیری از double booking در لایه `Backend API` و پایگاه داده تراکنشی enforce می‌شوند.