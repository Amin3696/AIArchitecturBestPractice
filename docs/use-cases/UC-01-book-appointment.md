# UC-01: رزرو نوبت (Book Appointment)

## هدف

تبدیل یک `Slot` آزاد به یک `Appointment` معتبر بدون double booking.

## Actorها

- Primary: `Patient`
- Secondary: `Receptionist`, `Clinic Admin`

## Preconditions

- کاربر احراز هویت شده باشد.
- `Patient` معتبر باشد.
- `Slot` موجود، فعال و آینده باشد.
- `Doctor` مرتبط با Slot فعال باشد.
- بیمار برای همان پزشک در همان روز Appointment فعال دیگری نداشته باشد.

## Trigger

- کاربر روی اقدام رزرو کلیک می‌کند.

## Main Success Flow

1. Actor لیست Slotهای آزاد را مشاهده می‌کند.
2. Actor یک Slot را انتخاب می‌کند.
3. کلاینت درخواست `POST /appointments` را با `X-Idempotency-Key` ارسال می‌کند.
4. سرویس قوانین دامنه را validate می‌کند.
5. سرویس داخل transaction محلی، Appointment را ایجاد می‌کند.
6. قید persistence برای جلوگیری از رزرو همزمان enforce می‌شود.
7. وضعیت‌ها ثبت می‌شوند: `AppointmentStatus=BOOKED` و `VisitStatus=NOT_STARTED`.
8. event رزرو منتشر می‌شود.
9. پاسخ `201` با جزئیات Appointment بازگردانده می‌شود.

## Postconditions

- Appointment با وضعیت `BOOKED` ایجاد شده است.
- Slot در خروجی observable غیرقابل رزرو است (`RESERVED` یا equivalent derived).

## API/خطاهای کلیدی

- Endpoint: `POST /appointments`
- Success: `201`
- Errors: `APPOINTMENT_SLOT_UNAVAILABLE`, `APPOINTMENT_PATIENT_DOCTOR_DAY_CONFLICT`, `SLOT_INACTIVE`, `SLOT_IN_PAST`, `DOCTOR_INACTIVE`

## Traceability

- FR: `FR-21`, `FR-22`, `FR-23`, `FR-24`
