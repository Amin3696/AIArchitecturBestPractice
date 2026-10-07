# UC-03: جابه‌جایی نوبت (Reschedule Appointment)

## هدف

انتقال اتمیک Appointment به Slot جدید معتبر بدون از دست‌رفتن نوبت قبلی در failure.

## Actorها

- Primary: `Patient` (owner)
- Secondary: `Receptionist`, `Clinic Admin` (override مجاز)

## Preconditions

- Appointment فعلی در وضعیت `BOOKED` باشد.
- Slot جدید فعال، آینده و بدون Appointment فعال باشد.
- برای بیمار همان سیاست زمانی UC-02 اعمال می‌شود.

## Main Success Flow

1. Actor نوبت فعلی را انتخاب می‌کند.
2. Actor Slot جدید را از لیست Slotهای آزاد انتخاب می‌کند.
3. کلاینت `POST /appointments/{appointmentId}/reschedule` را با `X-Idempotency-Key` ارسال می‌کند.
4. سرویس مجوزها و guardها را بررسی می‌کند.
5. سرویس عملیات را در transaction محلی انجام می‌دهد.
6. Slot قبلی آزاد و Slot جدید رزرو می‌شود (طبق guardها).
7. Appointment با زمان جدید در وضعیت `BOOKED` باقی می‌ماند.
8. event جابه‌جایی منتشر می‌شود.
9. پاسخ `200` برگردانده می‌شود.

## Postconditions

- Appointment به Slot جدید منتقل شده است.
- اگر عملیات fail شود، Appointment قبلی بدون تغییر باقی می‌ماند.

## API/خطاهای کلیدی

- Endpoint: `POST /appointments/{appointmentId}/reschedule`
- Success: `200`
- Errors: `APPOINTMENT_SLOT_UNAVAILABLE`, `APPOINTMENT_CANCELLATION_WINDOW_EXCEEDED`, `APPOINTMENT_ALREADY_TERMINAL`

## Traceability

- FR: `FR-34`, `FR-35`, `FR-37`
