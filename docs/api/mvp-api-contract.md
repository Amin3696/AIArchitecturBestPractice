# MVP API Contract (v1)

## هدف

این سند قرارداد واقعی API برای MVP را مشخص می‌کند و ۵ خروجی مرحله ۶ MUST را پوشش می‌دهد:

1. فهرست endpointهای MVP
2. request/response schema هر endpoint
3. status code و business error code سناریوهای اصلی
4. قرارداد authentication، logout، reset password و session/token
5. قواعد idempotency برای booking/reschedule

> مرجع سبک طراحی REST: `docs/api/rest-api-standards.md`

## خروجی اجرایی قرارداد

- فایل قابل‌اجرا برای ابزارها: `docs/api/openapi-v1.yaml`

---

## 1) Base Contract

- Base URL: `/api/v1`
- Content-Type: `application/json`
- Auth header: `Authorization: Bearer <accessToken>`
- Trace header: `X-Request-Id: <uuid>`

### Envelope موفق

```json
{
  "data": {}
}
```

### Envelope خطا

```json
{
  "error": {
    "code": "APPOINTMENT_SLOT_UNAVAILABLE",
    "message": "The requested slot is no longer available.",
    "details": [
      {
        "field": "slotId",
        "reason": "already_booked"
      }
    ],
    "requestId": "4a53b672-0bc8-4780-b4cc-d0fc18f149a0"
  }
}
```

---

## 2) Authentication, Session & Token Contract

## 2.1 Login

- `POST /auth/login`
- Auth: عمومی

Request:

```json
{
  "mobileNumber": "09121234567",
  "password": "string"
}
```

Response `200`:

```json
{
  "data": {
    "accessToken": "jwt",
    "refreshToken": "opaque-or-jwt",
    "tokenType": "Bearer",
    "expiresIn": 900,
    "refreshExpiresIn": 2592000,
    "sessionId": "01JABCSESSION",
    "user": {
      "userId": "01JUSER",
      "role": "PATIENT",
      "displayName": "Ali Ahmadi"
    }
  }
}
```

Status/Error:

- `200` موفق
- `401` + `AUTH_INVALID_CREDENTIALS`
- `423` + `AUTH_ACCOUNT_LOCKED`

## 2.2 Refresh Token

- `POST /auth/refresh`
- Auth: عمومی (با refresh token)

Request:

```json
{
  "refreshToken": "string"
}
```

Response `200`: مشابه Login با tokenهای جدید.

Errors:

- `401` + `AUTH_INVALID_REFRESH_TOKEN`
- `401` + `AUTH_SESSION_REVOKED`

## 2.3 Logout

- `POST /auth/logout`
- Auth: الزامی

Request:

```json
{
  "sessionId": "01JABCSESSION"
}
```

Response:

- `204` بدون body

Errors:

- `401` + `AUTH_UNAUTHENTICATED`

## 2.4 Password Reset - Request

- `POST /auth/password-reset-requests`
- Auth: عمومی

Request:

```json
{
  "mobileNumber": "09121234567"
}
```

Response:

- `202` (همیشه پاسخ عمومی برای جلوگیری از user enumeration)

## 2.5 Password Reset - Confirm

- `POST /auth/password-resets`
- Auth: عمومی

Request:

```json
{
  "mobileNumber": "09121234567",
  "resetToken": "string",
  "newPassword": "string"
}
```

Response:

- `204`

Errors:

- `400` + `AUTH_RESET_TOKEN_INVALID`
- `410` + `AUTH_RESET_TOKEN_EXPIRED`

---

## 3) Endpointهای MVP به‌تفکیک Context

## 3.1 Patient

### Register Patient

- `POST /patients`
- Roles: عمومی

Request:

```json
{
  "firstName": "Ali",
  "lastName": "Ahmadi",
  "mobileNumber": "09121234567",
  "dateOfBirth": "1994-05-12",
  "email": "ali@example.com",
  "password": "string"
}
```

Response `201`:

```json
{
  "data": {
    "patientId": "01JPAT",
    "firstName": "Ali",
    "lastName": "Ahmadi",
    "mobileNumber": "09121234567",
    "status": "ACTIVE",
    "createdAt": "2026-10-07T10:00:00Z"
  }
}
```

Errors:

- `409` + `PATIENT_MOBILE_ALREADY_EXISTS`
- `422` + `VALIDATION_FAILED`

### Get My Profile

- `GET /patients/me`
- Roles: `PATIENT`

Response `200`:

```json
{
  "data": {
    "patientId": "01JPAT",
    "firstName": "Ali",
    "lastName": "Ahmadi",
    "mobileNumber": "09121234567",
    "dateOfBirth": "1994-05-12",
    "email": "ali@example.com",
    "status": "ACTIVE"
  }
}
```

### Update My Profile

- `PATCH /patients/me`
- Roles: `PATIENT`

Request:

```json
{
  "firstName": "Ali",
  "lastName": "Ahmadi",
  "email": "new-email@example.com"
}
```

Response `200`: همان schema پروفایل.

---

## 3.2 Doctor & Specialty

### Create Doctor

- `POST /doctors`
- Roles: `CLINIC_ADMIN`

Request:

```json
{
  "fullName": "Dr. Sara Mohammadi",
  "defaultVisitDurationMinutes": 20,
  "specialtyIds": ["01JSPC01"],
  "status": "ACTIVE"
}
```

Response `201`:

```json
{
  "data": {
    "doctorId": "01JDOC01",
    "fullName": "Dr. Sara Mohammadi",
    "status": "ACTIVE",
    "specialties": [
      { "specialtyId": "01JSPC01", "title": "Cardiology" }
    ]
  }
}
```

### Update Doctor

- `PATCH /doctors/{doctorId}`
- Roles: `CLINIC_ADMIN`

Request:

```json
{
  "fullName": "Dr. Sara M.",
  "defaultVisitDurationMinutes": 15
}
```

Response `200`: همان schema doctor.

### Deactivate Doctor

- `POST /doctors/{doctorId}/deactivation`
- Roles: `CLINIC_ADMIN`

Request:

```json
{
  "reason": "leave"
}
```

Response: `204`

### Search Doctors

- `GET /doctors?specialtyId=&search=&date=`
- Roles: `PATIENT`, `RECEPTIONIST`, `CLINIC_ADMIN`, `DOCTOR`

Response `200` (collection):

```json
{
  "data": [
    {
      "doctorId": "01JDOC01",
      "fullName": "Dr. Sara Mohammadi",
      "status": "ACTIVE",
      "specialties": ["Cardiology"]
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 1,
    "totalPages": 1,
    "hasNext": false,
    "hasPrevious": false
  }
}
```

### Specialty Endpoints

- `POST /specialties` (`CLINIC_ADMIN`)
- `PATCH /specialties/{specialtyId}` (`CLINIC_ADMIN`)
- `POST /specialties/{specialtyId}/deactivation` (`CLINIC_ADMIN`)
- `GET /specialties` (authenticated)

Request/Response schema:

```json
{
  "code": "CARD",
  "title": "Cardiology",
  "isActive": true
}
```

---

## 3.3 Schedule & Slot

### Define Doctor Schedule

- `POST /doctors/{doctorId}/schedules`
- Roles: `CLINIC_ADMIN`, `DOCTOR` (owner)

Request:

```json
{
  "effectiveFrom": "2026-10-10",
  "effectiveTo": "2026-12-31",
  "recurrencePattern": "WEEKLY",
  "workingWindows": [
    { "dayOfWeek": 6, "start": "09:00", "end": "13:00" },
    { "dayOfWeek": 1, "start": "16:00", "end": "19:00" }
  ]
}
```

Response `201`:

```json
{
  "data": {
    "scheduleId": "01JSCH01",
    "doctorId": "01JDOC01",
    "isActive": true
  }
}
```

### Add Schedule Exception

- `POST /doctors/{doctorId}/schedules/{scheduleId}/exceptions`
- Roles: `CLINIC_ADMIN`, `DOCTOR` (owner)

Request:

```json
{
  "exceptionDate": "2026-11-01",
  "type": "DAY_OFF",
  "reason": "holiday"
}
```

Response: `201`

### Generate Slots

- `POST /doctors/{doctorId}/slots:generate`
- Roles: `CLINIC_ADMIN`, `DOCTOR` (owner)

Request:

```json
{
  "fromDate": "2026-10-10",
  "toDate": "2026-10-31",
  "slotDurationMinutes": 20
}
```

Response `202`:

```json
{
  "data": {
    "jobId": "01JJOB01",
    "status": "ACCEPTED"
  }
}
```

### Create Slot (Manual)

- `POST /slots`
- Roles: `CLINIC_ADMIN`, `DOCTOR` (owner)

Request:

```json
{
  "doctorId": "01JDOC01",
  "startAt": "2026-10-20T06:30:00Z",
  "endAt": "2026-10-20T06:50:00Z"
}
```

Response `201`:

```json
{
  "data": {
    "slotId": "01JSLT01",
    "doctorId": "01JDOC01",
    "startAt": "2026-10-20T06:30:00Z",
    "endAt": "2026-10-20T06:50:00Z",
    "status": "AVAILABLE"
  }
}
```

### Deactivate Slot

- `POST /slots/{slotId}/deactivation`
- Roles: `CLINIC_ADMIN`, `DOCTOR` (owner)

Request:

```json
{
  "reason": "clinic_closure"
}
```

Response: `204`

### Get Available Slots

- `GET /doctors/{doctorId}/available-slots?date=2026-10-20`
- Roles: authenticated users

Response `200`:

```json
{
  "data": [
    {
      "slotId": "01JSLT01",
      "doctorId": "01JDOC01",
      "startAt": "2026-10-20T06:30:00Z",
      "endAt": "2026-10-20T06:50:00Z",
      "status": "AVAILABLE"
    }
  ]
}
```

---

## 3.4 Appointment

### Book Appointment

- `POST /appointments`
- Roles: `PATIENT` (self), `RECEPTIONIST`, `CLINIC_ADMIN`
- Idempotency: **required** (`Idempotency-Key`)

Request:

```json
{
  "patientId": "01JPAT01",
  "slotId": "01JSLT01"
}
```

Response `201`:

```json
{
  "data": {
    "appointmentId": "01JAPT01",
    "patientId": "01JPAT01",
    "doctorId": "01JDOC01",
    "slotId": "01JSLT01",
    "scheduledStart": "2026-10-20T06:30:00Z",
    "scheduledEnd": "2026-10-20T06:50:00Z",
    "appointmentStatus": "BOOKED",
    "visitStatus": "NOT_STARTED",
    "bookedAt": "2026-10-07T11:00:00Z"
  }
}
```

Errors:

- `409` + `APPOINTMENT_SLOT_UNAVAILABLE`
- `409` + `APPOINTMENT_PATIENT_DOCTOR_DAY_CONFLICT`
- `409` + `DOCTOR_INACTIVE`
- `409` + `SLOT_INACTIVE`
- `409` + `SLOT_IN_PAST`
- `404` + `PATIENT_NOT_FOUND`
- `404` + `SLOT_NOT_FOUND`

### Get My Appointments

- `GET /appointments/me?status=&from=&to=&page=&pageSize=`
- Roles: `PATIENT`

Response `200`: collection با pagination.

### Get Appointment Details

- `GET /appointments/{appointmentId}`
- Roles: owner patient, owner doctor, `RECEPTIONIST`, `CLINIC_ADMIN`

Response `200`: schema کامل appointment.

Errors:

- `403` + `AUTH_FORBIDDEN`
- `404` + `APPOINTMENT_NOT_FOUND`

### Cancel Appointment

- `POST /appointments/{appointmentId}/cancellation`
- Roles: owner `PATIENT`, `RECEPTIONIST`, `CLINIC_ADMIN`

Request:

```json
{
  "reason": "patient_request"
}
```

Response `200`:

```json
{
  "data": {
    "appointmentId": "01JAPT01",
    "appointmentStatus": "CANCELLED",
    "cancelledAt": "2026-10-07T11:20:00Z"
  }
}
```

Errors:

- `409` + `APPOINTMENT_CANCELLATION_WINDOW_EXCEEDED` (برای بیمار)
- `409` + `APPOINTMENT_ALREADY_TERMINAL`

### Reschedule Appointment

- `POST /appointments/{appointmentId}/reschedule`
- Roles: owner `PATIENT`, `RECEPTIONIST`, `CLINIC_ADMIN`
- Idempotency: **required** (`Idempotency-Key`)

Request:

```json
{
  "newSlotId": "01JSLT99",
  "reason": "time_change"
}
```

Response `200`:

```json
{
  "data": {
    "appointmentId": "01JAPT01",
    "oldSlotId": "01JSLT01",
    "newSlotId": "01JSLT99",
    "appointmentStatus": "BOOKED",
    "visitStatus": "NOT_STARTED",
    "rescheduledAt": "2026-10-07T12:00:00Z"
  }
}
```

Errors:

- `409` + `APPOINTMENT_SLOT_UNAVAILABLE`
- `409` + `APPOINTMENT_CANCELLATION_WINDOW_EXCEEDED` (برای بیمار)
- `409` + `APPOINTMENT_ALREADY_TERMINAL`

### Update Visit Status

- `POST /appointments/{appointmentId}/visit-status`
- Roles: owner `DOCTOR`, `RECEPTIONIST`, `CLINIC_ADMIN`

Request:

```json
{
  "visitStatus": "WAITING"
}
```

Response `200`:

```json
{
  "data": {
    "appointmentId": "01JAPT01",
    "appointmentStatus": "BOOKED",
    "visitStatus": "WAITING",
    "updatedAt": "2026-10-07T12:10:00Z"
  }
}
```

Errors:

- `409` + `APPOINTMENT_INVALID_STATE_TRANSITION`
- `403` + `AUTH_FORBIDDEN`

### Daily Clinic Appointments

- `GET /appointments/clinic?date=2026-10-20&doctorId=&specialtyId=&status=`
- Roles: `RECEPTIONIST`, `CLINIC_ADMIN`

Response `200`: collection با pagination.

---

## 3.5 Notification

### Get My Notifications

- `GET /notifications/me?page=&pageSize=`
- Roles: `PATIENT`

Response `200`:

```json
{
  "data": [
    {
      "notificationId": "01JNTF01",
      "templateType": "BOOKING_CONFIRMED",
      "channel": "SMS",
      "status": "SENT",
      "scheduledAt": "2026-10-07T11:00:00Z",
      "sentAt": "2026-10-07T11:00:02Z"
    }
  ]
}
```

---

## 4) Status Code Matrix (سناریوهای اصلی)

- `200`: دریافت/تغییر موفق
- `201`: ایجاد موفق (`/patients`, `/doctors`, `/appointments`)
- `202`: jobهای async مثل `slots:generate`
- `204`: عملیات بدون body مثل logout/deactivation
- `400`: schema نامعتبر
- `401`: احراز هویت نامعتبر یا منقضی
- `403`: دسترسی غیرمجاز برای role/ownership
- `404`: resource پیدا نشد
- `409`: conflict بیزنسی یا وضعیت نامعتبر
- `422`: اعتبارسنجی دامنه یا فیلد
- `429`: rate limit
- `500`: خطای داخلی

---

## 5) Business Error Codes (MVP)

## 5.1 Auth

- `AUTH_UNAUTHENTICATED`
- `AUTH_INVALID_CREDENTIALS`
- `AUTH_INVALID_REFRESH_TOKEN`
- `AUTH_SESSION_REVOKED`
- `AUTH_ACCOUNT_LOCKED`
- `AUTH_RESET_TOKEN_INVALID`
- `AUTH_RESET_TOKEN_EXPIRED`

## 5.2 Patient/Doctor/Slot

- `PATIENT_NOT_FOUND`
- `PATIENT_MOBILE_ALREADY_EXISTS`
- `DOCTOR_NOT_FOUND`
- `DOCTOR_INACTIVE`
- `SLOT_NOT_FOUND`
- `SLOT_INACTIVE`
- `SLOT_IN_PAST`
- `SLOT_OVERLAP`

## 5.3 Appointment

- `APPOINTMENT_NOT_FOUND`
- `APPOINTMENT_SLOT_UNAVAILABLE`
- `APPOINTMENT_PATIENT_DOCTOR_DAY_CONFLICT`
- `APPOINTMENT_CANCELLATION_WINDOW_EXCEEDED`
- `APPOINTMENT_ALREADY_TERMINAL`
- `APPOINTMENT_INVALID_STATE_TRANSITION`
- `APPOINTMENT_RESCHEDULE_ATOMICITY_FAILED`

## 5.4 Authorization

- `AUTH_FORBIDDEN`
- `OWNERSHIP_VIOLATION`

---

## 6) Idempotency Rules

## 6.1 Header Contract

- Header: `Idempotency-Key: <uuid-or-random-string>`
- Key length: 8..128
- Scope: `method + path + actorId`
- TTL نگهداری نتیجه: 24 ساعت

## 6.2 عملیات اجباری

- `POST /appointments` → اجباری
- `POST /appointments/{appointmentId}/reschedule` → اجباری

در نبود header:

- `400` + `IDEMPOTENCY_KEY_REQUIRED`

## 6.3 رفتار بازپخش

- اگر همان `idempotency key` با همان payload تکرار شود:
  - همان response قبلی (`201` یا `200`) برگردد.
- اگر همان key با payload متفاوت تکرار شود:
  - `409` + `IDEMPOTENCY_KEY_REUSED_WITH_DIFFERENT_PAYLOAD`

## 6.4 خطاهای idempotency

- `IDEMPOTENCY_KEY_REQUIRED`
- `IDEMPOTENCY_KEY_INVALID`
- `IDEMPOTENCY_KEY_REUSED_WITH_DIFFERENT_PAYLOAD`

---

## 7) هم‌ترازی با قوانین دامنه

- جلوگیری از double booking با `APPOINTMENT_SLOT_UNAVAILABLE` enforce می‌شود.
- `Cancellation Window = 2 hours` برای actor بیمار enforce می‌شود.
- لغو/جابجایی خارج بازه برای `RECEPTIONIST` و `CLINIC_ADMIN` به‌عنوان override مجاز است.
- `Appointment Status` و `Visit Status` جداگانه در schema پاسخ نگه داشته می‌شوند.
