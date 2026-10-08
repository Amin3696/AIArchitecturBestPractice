# Data Model & DB Constraints (MVP Baseline)

## هدف

این سند خروجی مرحله ۷ MUST را تکمیل می‌کند:

- ERD منطقی سیستم
- نهایی‌سازی `Aggregate`، `Entity` و `Value Object`
- تعریف جدول‌ها، کلیدها، `Unique Constraint` و `Foreign Key`
- strategy جلوگیری از رزرو همزمان در سطح persistence
- timezone رسمی سیستم و نحوه ذخیره زمان

---

## 1) ERD منطقی سیستم

```mermaid
erDiagram
    PATIENT ||--o{ APPOINTMENT : books
    DOCTOR ||--o{ DOCTOR_SLOT : offers
    SPECIALTY ||--o{ DOCTOR_SPECIALTY : classifies
    DOCTOR ||--o{ DOCTOR_SPECIALTY : has
    DOCTOR ||--o{ DOCTOR_SCHEDULE : owns
    DOCTOR_SCHEDULE ||--o{ SCHEDULE_WORKING_WINDOW : defines
    DOCTOR_SCHEDULE ||--o{ SCHEDULE_EXCEPTION : overrides
    DOCTOR_SLOT ||--o| APPOINTMENT : reserved_by
    APPOINTMENT ||--o{ APPOINTMENT_EVENT : emits
    APPOINTMENT ||--o{ NOTIFICATION : triggers
    NOTIFICATION ||--o{ DELIVERY_ATTEMPT : has

    PATIENT {
      uuid patient_id PK
      string first_name
      string last_name
      string mobile_number UK
      string email
      date date_of_birth
      string status
      timestamptz created_at_utc
      timestamptz updated_at_utc
    }

    DOCTOR {
      uuid doctor_id PK
      string full_name
      string status
      int default_visit_duration_minutes
      timestamptz created_at_utc
      timestamptz updated_at_utc
    }

    SPECIALTY {
      uuid specialty_id PK
      string code UK
      string title
      bool is_active
    }

    DOCTOR_SPECIALTY {
      uuid doctor_id FK
      uuid specialty_id FK
      bool is_primary
      PK doctor_id_specialty_id
    }

    DOCTOR_SCHEDULE {
      uuid schedule_id PK
      uuid doctor_id FK
      date effective_from
      date effective_to
      string recurrence_pattern
      bool is_active
    }

    SCHEDULE_WORKING_WINDOW {
      uuid window_id PK
      uuid schedule_id FK
      int day_of_week
      time local_start_time
      time local_end_time
    }

    SCHEDULE_EXCEPTION {
      uuid exception_id PK
      uuid schedule_id FK
      date exception_date_local
      time local_start_time
      time local_end_time
      string exception_type
      string reason
    }

    DOCTOR_SLOT {
      uuid slot_id PK
      uuid doctor_id FK
      date local_date
      time local_start_time
      time local_end_time
      timestamptz start_at_utc
      timestamptz end_at_utc
      string status
      string created_source
      bool is_active
      timestamptz created_at_utc
      timestamptz updated_at_utc
    }

    APPOINTMENT {
      uuid appointment_id PK
      uuid slot_id FK
      uuid patient_id FK
      uuid doctor_id FK
      uuid specialty_id FK
      timestamptz scheduled_start_utc
      timestamptz scheduled_end_utc
      string appointment_status
      string visit_status
      timestamptz booked_at_utc
      timestamptz cancelled_at_utc
      string cancellation_reason
      string created_by_actor
      string last_modified_by_actor
      int version_no
    }

    APPOINTMENT_EVENT {
      uuid event_id PK
      uuid appointment_id FK
      string event_type
      json payload
      timestamptz occurred_at_utc
      string actor
    }

    NOTIFICATION {
      uuid notification_id PK
      uuid appointment_id FK
      uuid patient_id FK
      string template_type
      string channel
      string status
      timestamptz scheduled_at_utc
      timestamptz sent_at_utc
      string failure_reason
    }

    DELIVERY_ATTEMPT {
      uuid attempt_id PK
      uuid notification_id FK
      int attempt_no
      string provider
      string provider_response_code
      timestamptz attempted_at_utc
      bool is_success
    }
```

---

## 2) Aggregate / Entity / Value Object نهایی

### `Patient` Context

- **Aggregate:** `Patient`
- **Entities:** `Patient`
- **Value Objects:** `PatientContactInfo`, `PatientStatus`

### `Doctor` Context

- **Aggregate:** `Doctor`, `Schedule`, `Slot`
- **Entities:** `Doctor`, `Specialty`, `DoctorSchedule`, `ScheduleWorkingWindow`, `ScheduleException`, `DoctorSlot`
- **Value Objects:** `DoctorStatus`, `SlotStatus`, `TimeWindow`, `ScheduleRecurrencePattern`

### `Appointment` Context

- **Aggregate:** `Appointment`
- **Entities:** `Appointment`, `AppointmentEvent`
- **Value Objects:** `AppointmentStatus`, `VisitStatus`, `CancellationDecision`, `RescheduleDecision`

### `Notification` Context

- **Aggregate:** `Notification`
- **Entities:** `Notification`, `DeliveryAttempt`
- **Value Objects:** `NotificationChannel`, `NotificationStatus`, `ReminderSchedule`

---

## 3) جدول‌ها، کلیدها و قیود پایگاه داده

## 3.1 جدول‌های اصلی

- `patients`
- `doctors`
- `specialties`
- `doctor_specialties`
- `doctor_schedules`
- `schedule_working_windows`
- `schedule_exceptions`
- `doctor_slots`
- `appointments`
- `appointment_events`
- `notifications`
- `delivery_attempts`

## 3.2 Primary/Unique Keys (MVP)

- PK برای همه جدول‌ها مطابق ERD
- `patients.mobile_number` یکتا
- `specialties.code` یکتا
- `doctor_specialties(doctor_id, specialty_id)` یکتا
- `doctor_slots(doctor_id, start_at_utc, end_at_utc)` یکتا
- `delivery_attempts(notification_id, attempt_no)` یکتا

## 3.3 Foreign Keyها (حداقل لازم)

- `doctor_specialties.doctor_id -> doctors.doctor_id`
- `doctor_specialties.specialty_id -> specialties.specialty_id`
- `doctor_schedules.doctor_id -> doctors.doctor_id`
- `schedule_working_windows.schedule_id -> doctor_schedules.schedule_id`
- `schedule_exceptions.schedule_id -> doctor_schedules.schedule_id`
- `doctor_slots.doctor_id -> doctors.doctor_id`
- `appointments.slot_id -> doctor_slots.slot_id`
- `appointments.patient_id -> patients.patient_id`
- `appointments.doctor_id -> doctors.doctor_id`
- `appointments.specialty_id -> specialties.specialty_id`
- `appointment_events.appointment_id -> appointments.appointment_id`
- `notifications.appointment_id -> appointments.appointment_id`
- `notifications.patient_id -> patients.patient_id`
- `delivery_attempts.notification_id -> notifications.notification_id`

## 3.4 Check Constraints کلیدی

- `appointments.appointment_status IN ('BOOKED','CANCELLED','COMPLETED','NO_SHOW')`
- `appointments.visit_status IN ('NOT_STARTED','WAITING','IN_PROGRESS','COMPLETED','NO_SHOW')`
- `doctor_slots.status IN ('AVAILABLE','RESERVED','INACTIVE','EXPIRED')`
- `doctor_slots.start_at_utc < doctor_slots.end_at_utc`
- `doctor_slots.local_start_time < doctor_slots.local_end_time`

---

## 4) Strategy جلوگیری از رزرو همزمان در Persistence

طبق ADR-0002:

1. `BookAppointment` و `RescheduleAppointment` فقط در transaction محلی اجرا می‌شوند.
2. anti-double-booking در DB enforce می‌شود (pre-check به‌تنهایی کافی نیست).

### 4.1 Unique constraint روی Appointment فعال هر Slot

```sql
CREATE UNIQUE INDEX uq_appointments_active_per_slot
ON appointments (slot_id)
WHERE appointment_status = 'BOOKED';
```

### 4.2 قید Appointment فعال بیمار/پزشک/روز

```sql
CREATE UNIQUE INDEX uq_appointments_active_patient_doctor_day
ON appointments (patient_id, doctor_id, (scheduled_start_utc::date))
WHERE appointment_status = 'BOOKED';
```

### 4.3 رفتار تعارض همزمانی

- درخواست برنده commit می‌شود.
- درخواست بازنده rollback می‌شود.
- خطای DB به business error `APPOINTMENT_SLOT_UNAVAILABLE` نگاشت می‌شود.

---

## 5) Timezone رسمی سیستم و ذخیره زمان

## 5.1 Timezone رسمی

- timezone رسمی سیستم: `Asia/Tehran`

## 5.2 Storage Policy

- همه timestampها در DB به‌صورت `timestamptz` و بر مبنای UTC ذخیره می‌شوند.
- برای `Slot` فیلدهای `local_date`، `local_start_time` و `local_end_time` نیز ذخیره می‌شوند.
- تبدیل local/UTC باید timezone-aware باشد (بدون offset ثابت).

## 5.3 Rule Evaluation

- ruleهای `Cancellation Window` و «همان روز» با زمان محلی کلینیک (`Asia/Tehran`) ارزیابی می‌شوند.
- persistence ordering و consistency روی UTC انجام می‌شود.

---

## 6) Traceability به اسناد مرجع

- `docs/architecture/domain/domain-context.md`
- `docs/architecture/domain/state-machines.md`
- `docs/architecture/domain/contexts/appointment/context-spec.md`
- `docs/architecture/domain/contexts/doctor/context-spec.md`
- `docs/architecture/domain/contexts/patient/context-spec.md`
- `docs/architecture/domain/contexts/notification/context-spec.md`
- `docs/architecture/decisions/ADR-0002-booking-consistency-and-concurrency-control.md`
