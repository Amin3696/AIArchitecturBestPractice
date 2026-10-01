# سیستم نوبت‌دهی کلینیک

این سند خلاصه فشرده و به‌روز پروژه برای استفاده ابزارهای هوش مصنوعی و توسعه‌دهندگان است. هرگونه تولید کد، بازبینی طراحی یا پیشنهاد پیاده‌سازی باید با اسناد authoritative پروژه هم‌راستا باشد.

## وضعیت کلی

- سبک معماری: `Modular Monolith`
- دامنه MVP: نوبت‌دهی کلینیک برای یک کلینیک، بدون پرداخت آنلاین و بدون multi-clinic
- مرجع رسمی نیازمندی‌ها: `docs/requirements/srs/srs.md`
- مرجع baseline: `docs/akb-baseline.md`

## Bounded Contextهای فعلی

- `Patient`: ثبت‌نام و پروفایل بیمار، اطلاعات تماس و lookup بیمار
- `Doctor`: پزشک، تخصص، schedule، schedule exception و slotهای عرضه‌شده
- `Appointment`: رزرو، لغو، جابه‌جایی، lifecycle نوبت و visit status
- `Notification`: اعلان‌ها و reminderها به‌صورت downstream event consumer

## قوانین معماری و دامنه که باید رعایت شوند

1. **جلوگیری از Double Booking**
   - برای هر `Slot` حداکثر یک `Appointment` فعال مجاز است.
   - منبع حقیقت نهایی برای این invariant پایگاه داده تراکنشی و transaction محلی است.
   - در تعارض همزمانی، فقط یک درخواست موفق می‌شود و بقیه با `APPOINTMENT_SLOT_UNAVAILABLE` رد می‌شوند.
   - مرجع تصمیم: `docs/architecture/decisions/ADR0002-booking-consistency-and-concurrency-control.md`

2. **مرز ماژول‌ها**
   - هیچ ماژولی نباید مستقیماً به entity یا repository داخلی ماژول دیگر دسترسی داشته باشد.
   - تعامل میان contextها فقط از طریق contract داخلی، query contract یا event داخلی مجاز است.
   - `Appointment` مالک `Slot` نیست و فقط از contractهای `Doctor` و `Patient` استفاده می‌کند.

3. **مدل وضعیت‌ها**
   - `Appointment Status` و `Visit Status` دو مفهوم جدا هستند.
   - `Appointment Status` در MVP: `BOOKED`, `CANCELLED`, `COMPLETED`, `NO_SHOW`
   - `Visit Status`: `NOT_STARTED`, `WAITING`, `IN_PROGRESS`, `COMPLETED`, `NO_SHOW`
   - `Slot Status`: `AVAILABLE`, `RESERVED`, `INACTIVE`, `EXPIRED`
   - مرجع: `docs/architecture/domain/state-machines.md`

4. **قواعد کسب‌وکاری حیاتی**
   - `Cancellation Window = 2 hours` بر اساس timezone رسمی کلینیک
   - بیمار فقط تا قبل از پایان این بازه می‌تواند لغو یا جابه‌جایی انجام دهد
   - `Receptionist` و `Clinic Admin` می‌توانند خارج از این بازه override انجام دهند
   - بیمار می‌تواند چند نوبت فعال داشته باشد، اما نه بیش از یک نوبت فعال با یک پزشک در یک روز تقویمی

5. **کنترل دسترسی**
   - `Patient` فقط به داده‌ها و نوبت‌های خودش دسترسی دارد
   - `Doctor` فقط به schedule، slotها و appointmentهای خودش دسترسی دارد
   - `Receptionist` actor عملیاتی کلینیک است
   - `Clinic Admin` actor مدیریتی کسب‌وکاری کلینیک است
   - `System Admin` actor فنی/امنیتی است و actor روزمره booking محسوب نمی‌شود
   - مرجع: `docs/architecture/domain/access-control-matrix.md`

## نکات اجرایی برای تولید سورس

- از فرض کردن `Payment Gateway`, `Multi-Clinic`, `Push Notification`, یا `Identity Context` مستقل در MVP خودداری شود مگر اینکه سند جدید اضافه شود.
- اگر لازم است تصمیم معماری جدیدی گرفته شود، باید ADR جدید ایجاد شود.
- اگر بین اسناد تعارض وجود داشت، ترتیب مرجع از `docs/akb-baseline.md` پیروی می‌کند.
- اگر قراردادی هنوز دقیق نشده باشد، باید از اسناد authoritative نقل شود و از اختراع API یا schema قطعی خودداری شود.

## مستندات مرجع مهم

- `docs/akb-baseline.md`
- `docs/requirements/srs/srs.md`
- `docs/requirements/functional-requirements.md`
- `docs/requirements/non-functional-requirements.md`
- `docs/architecture/decisions/ADR0001-modular-monolith-architecture.md`
- `docs/architecture/decisions/ADR0002-booking-consistency-and-concurrency-control.md`
- `docs/architecture/domain/context-map.md`
- `docs/architecture/domain/domain-context.md`
- `docs/architecture/domain/state-machines.md`
- `docs/architecture/domain/access-control-matrix.md`
- `docs/architecture/c4models/01-system-context.md`
- `docs/architecture/c4models/02-container.md`