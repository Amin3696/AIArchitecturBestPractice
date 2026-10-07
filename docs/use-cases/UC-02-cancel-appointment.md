# UC-02: لغو نوبت (Cancel Appointment)

## هدف

تغییر وضعیت Appointment از `BOOKED` به `CANCELLED` با رعایت policyهای نقش و زمان.

## Actorها

- Primary: `Patient` (owner)
- Secondary: `Receptionist`, `Clinic Admin` (override مجاز)

## Preconditions

- Appointment در وضعیت `BOOKED` باشد.
- اگر actor بیمار است، حداقل ۲ ساعت تا زمان شروع نوبت باقی مانده باشد.

## Main Success Flow

1. Actor جزئیات Appointment را مشاهده می‌کند.
2. Actor درخواست لغو را ثبت می‌کند.
3. سرویس ownership/role را بررسی می‌کند.
4. policy زمانی بررسی می‌شود.
5. وضعیت Appointment به `CANCELLED` تغییر می‌کند.
6. event لغو منتشر می‌شود.
7. امکان بازشدن مجدد Slot با guardهای Slot ارزیابی می‌شود.
8. پاسخ `200` با وضعیت جدید بازگردانده می‌شود.

## Postconditions

- Appointment لغو شده و تاریخچه حفظ شده است.
- Slot فقط در صورت فعال/آینده بودن Slot و فعال بودن Doctor می‌تواند دوباره قابل رزرو شود.

## API/خطاهای کلیدی

- Endpoint: `POST /appointments/{appointmentId}/cancellation`
- Success: `200`
- Errors: `APPOINTMENT_CANCELLATION_WINDOW_EXCEEDED`, `APPOINTMENT_ALREADY_TERMINAL`, `AUTH_FORBIDDEN`

## Traceability

- FR: `FR-29`, `FR-30`, `FR-31`, `FR-32`, `FR-33`
