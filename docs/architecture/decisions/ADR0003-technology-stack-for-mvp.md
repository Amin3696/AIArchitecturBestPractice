# ADR-0003: انتخاب Technology Stack برای MVP

* **وضعیت:** پذیرفته‌شده
* **تاریخ:** 2026-10-08
* **تصمیم‌گیرندگان:** تیم معماری و توسعه

## ۱. مسئله و کانتکست (Context)

پس از تثبیت سبک معماری `Modular Monolith` در ADR-0001 و تعریف الزامات سازگاری رزرو در ADR-0002، لازم است Technology Stack اجرایی MVP به‌صورت رسمی ثبت شود تا:

* تصمیم‌های فنی تیم یکپارچه و قابل ارجاع باشند
* طراحی، پیاده‌سازی و استخدام/تقسیم‌کار بر مبنای یک baseline ثابت انجام شود
* از واگرایی تکنولوژیک در لایه‌های API، Persistence، Caching و Modularization جلوگیری شود

الزامات کلیدی پروژه در این تصمیم:

* توسعه سریع MVP با حفظ کیفیت و تست‌پذیری
* پشتیبانی مناسب از Domain Modeling و ماژول‌بندی داخلی
* سازگاری قوی تراکنشی برای سناریوهای رزرو/لغو/جابجایی
* سادگی عملیاتی قابل‌قبول برای تیم کوچک

## ۲. گزینه‌های بررسی‌شده (Options Considered)

* **گزینه ۱: Java 21 + Spring Boot 3.x + Spring Modulith + PostgreSQL + Redis**
  * هم‌راستا با معماری Modular Monolith
  * اکوسیستم بالغ برای API، Security، Data، Observability و Test

* **گزینه ۲: .NET + ASP.NET Core + PostgreSQL + Redis**
  * گزینه معتبر فنی، اما با اولویت فعلی تیم برای اکوسیستم Java هم‌راستایی کمتری دارد

* **گزینه ۳: Node.js + TypeScript + NestJS + PostgreSQL + Redis**
  * سرعت توسعه مناسب، اما برای قواعد تراکنشی حساس و ماژول‌بندی دامنه‌محور در MVP فعلی نسبت به گزینه ۱ اولویت پایین‌تری دارد

## ۳. تصمیم نهایی (Decision Outcome)

**گزینه ۱ انتخاب می‌شود.**

Technology Stack رسمی MVP:

1. **Programming Language:** `Java 21`
2. **Backend Framework:** `Spring Boot 3.x` (نسخه 3 یا بالاتر در همان major line)
3. **Modular Monolith Framework:** `Spring Modulith`
4. **Primary Database:** `PostgreSQL`
5. **Cache:** `Redis`

قواعد تکمیلی تصمیم:

* `PostgreSQL` منبع حقیقت داده‌های بیزنسی و تراکنش‌ها است.
* `Redis` صرفاً برای cache عملیاتی و داده‌های کوتاه‌عمر استفاده می‌شود و منبع حقیقت رزرو نیست.
* مرز ماژول‌ها با `Spring Modulith` enforce می‌شود و تعامل بین ماژول‌ها باید از طریق قراردادهای مشخص (API داخلی/Domain Event) انجام شود.
* قابلیت‌های Async مانند reminder/notification درون همین monolith پیاده‌سازی می‌شوند (بدون الزام به تفکیک سرویس مستقل در MVP).

## ۴. پیامدها  (Consequences & Trade-offs)

* **مزایا (Pros):**
  * هم‌راستایی کامل با ADR-0001 (`Modular Monolith`)
  * بلوغ بالای اکوسیستم Spring برای API، Security، Validation و Data Access
  * پشتیبانی مناسب از مرزبندی ماژولی با Spring Modulith
  * سادگی عملیات نسبت به معماری توزیع‌شده در فاز MVP
  * استفاده از PostgreSQL برای تضمین invariantهای تراکنشی حیاتی (از جمله ضد double-booking)

* **ریسک‌ها و هزینه‌ها (Cons):**
  * نیاز به discipline بالا برای حفظ مرزهای ماژولی در طول زمان
  * وابستگی بیشتر تیم به اکوسیستم Spring
  * اضافه شدن پیچیدگی cache invalidation و مدیریت TTL در Redis
  * در صورت رشد شدید بار، ممکن است نیاز به تفکیک runtime برخی قابلیت‌ها ایجاد شود
