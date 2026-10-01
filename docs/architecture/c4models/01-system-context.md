# C4 Model - Level 1: System Context

```mermaid
C4Context
title Clinic Appointment System

Person(patient, "Patient", "Books and manages personal appointments")
Person(doctor, "Doctor", "Views own schedule and updates visit status")
Person(receptionist, "Receptionist", "Handles operational booking, cancellation, rescheduling, and check-in")
Person(clinicAdmin, "Clinic Admin", "Manages doctors, schedules, slots, clinic users, and reports")
Person(systemAdmin, "System Admin", "Manages access policies, admin users, and system settings")

System(system, "Clinic Appointment System", "Web-based modular monolith for clinic appointment booking and operations")

System_Ext(sms, "SMS Provider", "Delivers SMS notifications and reminders")
System_Ext(email, "Email Provider", "Delivers email notifications and reminders")

Rel(patient, system, "Searches doctors, views slots, books/cancels/reschedules own appointments")
Rel(doctor, system, "Views own schedule and appointments, updates visit status")
Rel(receptionist, system, "Books/cancels/reschedules appointments for clinic operations")
Rel(clinicAdmin, system, "Manages clinic business configuration and reports")
Rel(systemAdmin, system, "Manages access control, admin users, and system settings")

Rel(system, sms, "Sends booking/cancellation/reminder SMS")
Rel(system, email, "Sends booking/cancellation/reminder emails")
```

## Notes

- `System Admin` در MVP یک actor فنی/امنیتی است و actor روزمره کسب‌وکاری برای booking محسوب نمی‌شود.
- `SMS Provider` و `Email Provider` تنها external systemهای قطعی در baseline فعلی هستند.
- `Payment Gateway` در MVP خارج از محدوده است و در system context نمایش داده نمی‌شود.