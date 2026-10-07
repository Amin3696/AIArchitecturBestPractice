# چک‌لیست تکمیل AKB برای آمادگی تولید سورس

این چک‌لیست برای تکمیل knowledge base پروژه نوشته شده تا مستندات به سطحی برسند که بتوان بر اساس آن‌ها source code را با ریسک پایین‌تری تولید کرد.

معیار اولویت‌ها:

- `Must`: قبل از شروع تولید سورس باید تکمیل شوند.
- `Should`: بلافاصله بعد از موارد حیاتی تکمیل شوند تا طراحی و پیاده‌سازی قابل اتکا بماند.
- `Later`: برای بلوغ معماری، عملیات و توسعه‌پذیری بعدی لازم‌اند.

---

## Must

### 1) پاکسازی ساختار و baseline مستندات

- [x] نام‌گذاری فایل‌ها و پوشه‌های اشتباه اصلاح شود (`appointmnet`, `contex-spec`, `system-contex`, فایل‌های `.md.md`).
- [x] مشخص شود کدام اسناد authoritative هستند و در تعارض بین اسناد، کدام مرجع نهایی تصمیم است.
- [x] نسخه و وضعیت هر سند کلیدی (`draft`, `reviewed`, `approved`) ثبت شود.

### 2) تکمیل مدل دامنه و مرزها

- [x] `context-map` تکمیل شود.
- [x] `domain-context` تکمیل شود.
- [x] `ubiquitous-language` تکمیل شود.
- [x] مشخصات همه bounded contextها تکمیل شود: `appointment`, `doctor`, `patient`, `notification`.
- [x] مسئولیت هر context و owner هر business capability نهایی شود.
- [x] نوع ارتباط بین contextها مشخص شود: upstream/downstream, ACL, shared kernel, published language.

### 3) نهایی‌سازی business rules حیاتی

- [x] قانون جلوگیری از double booking به‌صورت دقیق و قابل تست تعریف شود.
- [x] رفتار دقیق `Cancellation Window` نهایی شود.
- [x] قواعد `Reschedule` و atomic بودن آن نهایی شود.
- [x] رفتار Slot بعد از لغو نوبت مشخص شود.
- [x] قواعد مربوط به پزشک غیرفعال، Slot گذشته و Slot غیرفعال نهایی شود.
- [x] قواعد چند نوبت فعال برای یک بیمار نهایی شود.

### 4) مدل وضعیت‌ها و جریان‌های اصلی

- [x] state machine کامل `Appointment` تعریف شود.
- [x] state machine کامل `Slot` تعریف شود.
- [x] مشخص شود `Visit Status` از `Appointment Status` جداست یا خیر.
- [x] transitionها، guardها و actor مجاز برای هر تغییر وضعیت ثبت شود.

### 5) نقش‌ها و مجوزها

- [x] ماتریس کامل `Role -> Permission` تهیه شود.
- [x] مرز اختیارات `Patient`, `Doctor`, `Receptionist`, `Clinic Admin`, `System Admin` مشخص شود.
- [x] موارد override مانند لغو/جابجایی توسط پذیرش یا مدیر شفاف شود.

### 6) API contractهای واقعی

- [x] فهرست endpointهای MVP تعریف شود.
- [x] request/response schema برای هر endpoint مشخص شود.
- [x] status code و business error code هر سناریوی اصلی تعریف شود.
- [x] قرارداد احراز هویت، logout، reset password و session/token مشخص شود.
- [x] قواعد idempotency برای عملیات حساس مثل booking/reschedule مشخص شود.

### 7) مدل داده و قیود پایگاه داده

- [ ] ERD منطقی سیستم تهیه شود.
- [ ] aggregateها، entityها و value objectها نهایی شوند.
- [ ] جدول‌ها، کلیدها، unique constraintها و foreign keyها تعریف شوند.
- [ ] strategy جلوگیری از رزرو همزمان در سطح persistence مشخص شود.
- [ ] timezone رسمی سیستم و نحوه ذخیره زمان‌ها مشخص شود.

### 8) use caseهای end-to-end

- [ ] use case رزرو نوبت مستند شود.
- [ ] use case لغو نوبت مستند شود.
- [ ] use case جابه‌جایی نوبت مستند شود.
- [ ] use case مدیریت برنامه پزشک مستند شود.
- [ ] use case notification و reminder مستند شود.
- [ ] exception flowهای حیاتی برای concurrency و authorization مستند شوند.

---

## Should

### 9) تکمیل نیازمندی‌های ناقص یا مبهم

- [ ] نیازمندی‌های مربوط به `Patient Profile` کامل شوند.
- [ ] نیازمندی‌های `User Management` برای نقش‌های مدیریتی کامل شوند.
- [ ] نیازمندی‌های `Doctor Schedule Management` و actor مجاز آن نهایی شوند.
- [ ] نیازمندی‌های `Reporting` با فیلترها، KPIها و خروجی‌ها دقیق شوند.
- [ ] تفاوت قابلیت‌های MVP و Future Scope در مستندات به‌وضوح برچسب‌گذاری شود.

### 10) قراردادهای integration

- [ ] قرارداد `SMS Provider` مشخص شود.
- [ ] قرارداد `Email Provider` مشخص شود.
- [ ] retry policy، timeout و fallback برای ارسال اعلان‌ها تعریف شود.
- [ ] مشخص شود در MVP کدام کانال‌ها واقعی‌اند و کدام شبیه‌سازی می‌شوند.

### 11) ADRهای تکمیلی

- [ ] ADR مربوط به authentication/authorization ثبت شود.
- [ ] ADR مربوط به concurrency control و booking consistency ثبت شود.
- [ ] ADR مربوط به notification architecture ثبت شود.
- [ ] ADR مربوط به data model / storage choices ثبت شود.
- [ ] ADR مربوط به background jobs / queue / worker ثبت شود.

### 12) معماری اجرایی قابل پیاده‌سازی

- [ ] decomposition ماژول‌های modular monolith به سطح `API`, `Application`, `Domain`, `Infrastructure` مشخص شود.
- [ ] قرارداد تعامل داخلی ماژول‌ها تعریف شود.
- [ ] application serviceها و responsibility هر کدام استخراج شوند.
- [ ] policyهای transaction boundary و consistency boundary ثبت شوند.

### 13) audit و security

- [ ] eventهای audit‌شونده دقیقاً فهرست شوند.
- [ ] فیلدهای audit شامل actor, action, timestamp, before/after نهایی شوند.
- [ ] policy نگه‌داری audit مشخص شود.
- [ ] طبقه‌بندی داده‌های حساس و سطح حفاظت آن‌ها مشخص شود.
- [ ] قواعد logging امن و masking داده‌های حساس تعریف شود.

### 14) traceability و testability

- [ ] نگاشت `Interview -> SRS -> FR/NFR -> Domain -> API -> Test` ایجاد شود.
- [ ] acceptance criteria هر capability به سناریوهای تست قابل تبدیل شود.
- [ ] matrix تست برای رزرو همزمان، لغو، reschedule و authorization تهیه شود.

---

## Later

### 15) مستندات عملیات و استقرار

- [ ] deployment view تکمیل شود.
- [ ] environmentها (`dev`, `test`, `prod`) تعریف شوند.
- [ ] secret/config management مشخص شود.
- [ ] backup/restore و disaster recovery به سطح مناسب MVP مستند شود.

### 16) observability و قابلیت بهره‌برداری

- [ ] logging standard تکمیل شود.
- [ ] metrics کلیدی سیستم تعریف شوند.
- [ ] health check و readiness/liveness policy تعریف شود.
- [ ] alertهای حیاتی کسب‌وکار و فنی مشخص شوند.

### 17) تکامل‌پذیری محصول

- [ ] سناریوی توسعه به `multi-clinic` تحلیل و محدودیت‌های فعلی مستند شود.
- [ ] featureهای خارج از MVP مثل `payment`, `push notification`, `advanced reporting` در roadmap معماری قرار گیرند.
- [ ] rule engine یا configuration strategy برای policyهای قابل تغییر در آینده بررسی شود.

### 18) استانداردها و referenceهای مکمل

- [ ] security standard پروژه اضافه شود.
- [ ] coding/documentation standards در صورت نیاز اضافه شوند.
- [ ] referenceهای domain/business برای اصطلاحات درمانگاه و نوبت‌دهی تکمیل شوند.

---

## Definition of Ready برای شروع تولید سورس

وقتی موارد زیر تکمیل شوند، AKB برای شروع تولید سورس در وضعیت قابل قبول قرار می‌گیرد:

- [ ] contextها و context map نهایی شده باشند.
- [ ] business ruleهای رزرو، لغو و reschedule بدون ابهام باشند.
- [ ] state machineها کامل و قابل تست باشند.
- [ ] ماتریس نقش/مجوز نهایی شده باشد.
- [ ] API contractهای MVP آماده باشند.
- [ ] مدل داده و constraintهای ضد double-booking مشخص باشند.
- [ ] use caseهای end-to-end و exception flowهای اصلی ثبت شده باشند.
- [ ] ADRهای تصمیمات حیاتی فنی ثبت شده باشند.

---

## پیشنهاد ترتیب اجرا

1. پاکسازی ساختار اسناد
2. تکمیل دامنه و context map
3. نهایی‌سازی business rules و state machine
4. نهایی‌سازی role/permission matrix
5. تعریف API contract و data model
6. ثبت ADRهای فنی حیاتی
7. تکمیل traceability و test matrix
