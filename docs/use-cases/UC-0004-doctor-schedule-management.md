# UC-0004: مدیریت برنامه پزشک (Doctor Schedule Management)

## هدف

تعریف/تغییر برنامه کاری پزشک و عرضه Slot معتبر برای رزرو.

## Actorها

- Primary: `Clinic Admin`
- Secondary: `Doctor` (برای schedule خودش)

## Preconditions

- Actor مجاز بر اساس نقش/ownership باشد.
- پزشک فعال باشد.

## Main Success Flow

1. Actor برنامه کاری پزشک را تعریف می‌کند (`POST /doctors/{doctorId}/schedules`).
2. در صورت نیاز exception ثبت می‌کند (`POST /doctors/{doctorId}/schedules/{scheduleId}/exceptions`).
3. سیستم تولید Slot را اجرا می‌کند (`POST /doctors/{doctorId}/slots:generate`) یا Slot دستی ایجاد می‌شود (`POST /slots`).
4. سیستم هم‌پوشانی و اعتبار بازه‌ها را اعتبارسنجی می‌کند.
5. Slotهای معتبر با وضعیت `AVAILABLE` عرضه می‌شوند.

## Alternative Flows

- غیرفعال‌سازی Slot با `POST /slots/{slotId}/deactivation` باعث خروج Slot از عرضه رزرو جدید می‌شود.

## Postconditions

- برنامه پزشک و Slotهای قابل رزرو با قواعد دامنه هم‌راستا هستند.

## API/خطاهای کلیدی

- Errors: `SLOT_OVERLAP`, `DOCTOR_INACTIVE`, `AUTH_FORBIDDEN`

## Traceability

- FR: `FR-14`, `FR-15`, `FR-16`, `FR-18`
