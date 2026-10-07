# C4 Model - Level 3: Component Views

این فایل نمای راهنمای Level 3 در C4 را برای containerهای اصلی سامانه ارائه می‌کند. در این سطح، هر container به componentهای اصلی شکسته می‌شود تا برای طراحی و تولید سورس، مرز مسئولیت‌ها شفاف‌تر شود.

## Scope

در baseline فعلی، سه container نیاز به Level 3 component view دارند:

- `Web App`
- `Backend API`
- `Background Worker`

Containerهای `PostgreSQL` و `Redis` در این سطح به component داخلی شکسته نمی‌شوند، چون در AKB فعلی به‌عنوان زیرساخت استاندارد مدل شده‌اند.

## Level 3 Files

- [03-web-app-components.md](03-web-app-components.md)
- [04-backend-api-components.md](04-backend-api-components.md)
- [05-background-worker-components.md](05-background-worker-components.md)

## Component Principles

- componentها باید با bounded contextهای تاییدشده در AKB هم‌راستا بمانند.
- `Appointment` مرکز تصمیم‌گیری رزرو و lifecycle نوبت است.
- `Doctor` مالک schedule و `Slot` است.
- `Notification` downstream است و تصمیم رزرو نمی‌گیرد.
- access control باید هم در سطح UI و هم در سطح API enforce شود.
- invariantهای ضد `double booking` در `Backend API` و persistence boundary enforce می‌شوند، نه در `Redis`.

## Mapping to C4 Level 2

| Level 2 Container | Level 3 View | Purpose |
| --- | --- | --- |
| `Web App` | [03-web-app-components.md](03-web-app-components.md) | تفکیک صفحات/ماژول‌های UI و client-side orchestration |
| `Backend API` | [04-backend-api-components.md](04-backend-api-components.md) | تفکیک ماژول‌های API/Application/Domain/Infrastructure در modular monolith |
| `Background Worker` | [05-background-worker-components.md](05-background-worker-components.md) | تفکیک consumerها، orchestration و channel adapterهای اعلان |

## Notes

- این Level 3ها در حد design-for-generation نوشته شده‌اند؛ یعنی برای تولید سورس مرزها و dependency direction را روشن می‌کنند، نه اینکه implementation class-level بدهند.
- اگر بعداً item شماره 12 در checklist اجرا شود، این فایل‌ها باید با decomposition رسمی `API / Application / Domain / Infrastructure` دقیق‌تر sync شوند.
