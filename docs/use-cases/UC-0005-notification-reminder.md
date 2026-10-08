# UC-0005: Notification و Reminder

## هدف

ارسال اعلان رزرو/لغو/جابجایی و Reminder بدون شکستن تراکنش اصلی رزرو.

## Actorها

- Primary: `System`
- Secondary: `Patient` (دریافت‌کننده)

## Preconditions

- event کسب‌وکاری معتبر تولید شده باشد (`AppointmentBooked`, `AppointmentCancelled`, `AppointmentRescheduled`).
- داده تماس بیمار در دسترس باشد.

## Main Success Flow

1. سیستم event را از context `Appointment` دریافت می‌کند.
2. پیام notification با template مناسب ساخته می‌شود.
3. کانال ارسال انتخاب می‌شود (MVP: SMS/Email؛ Push وابسته به scope نهایی).
4. ارسال انجام یا زمان‌بندی reminder ثبت می‌شود.
5. وضعیت ارسال در `Notification`/`DeliveryAttempt` ثبت می‌شود.

## Postconditions

- اعلان/یادآور در تاریخچه notification قابل پیگیری است.
- failure ارسال، transaction رزرو را invalidate نمی‌کند.

## Traceability

- FR: `FR-25`, `FR-33`, `FR-43`, `FR-44`, `FR-45`
