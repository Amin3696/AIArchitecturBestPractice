# Access Control Matrix

## هدف

این سند ماتریس کامل `Role -> Permission`، مرز اختیارات هر نقش، و overrideهای مجاز در MVP را تعریف می‌کند تا در طراحی API، application service، UI و تست‌های authorization تفسیر یکتا وجود داشته باشد.

---

## نقش‌های اصلی

| Role | Purpose | Scope |
| --- | --- | --- |
| `Patient` | رزرو و مدیریت نوبت‌های خود | فقط داده‌ها و عملیات متعلق به خود بیمار |
| `Doctor` | مشاهده برنامه و مدیریت مراجعه بیماران خود | فقط schedule، slotها و appointmentهای منتسب به خود پزشک |
| `Receptionist` | اجرای عملیات روزمره کلینیک روی نوبت‌ها و بیماران | داده‌های عملیاتی کلینیک در محدوده دسترسی تعریف‌شده |
| `Clinic Admin` | اداره کسب‌وکاری سامانه در سطح کلینیک | تمام عملیات کسب‌وکاری MVP در سطح کلینیک |
| `System Admin` | اداره فنی، امنیتی و تنظیماتی سامانه | دسترسی مدیریتی فنی، نه مالکیت عملیات روزمره کسب‌وکاری |

---

## Permission Matrix

Legend:

- `A`: Allowed
- `O`: Allowed with ownership constraint
- `X`: Not allowed
- `OV`: Allowed as override capability

| Capability | Patient | Doctor | Receptionist | Clinic Admin | System Admin |
| --- | --- | --- | --- | --- | --- |
| Register patient | A (self) | X | X | X | X |
| Login / Logout | A | A | A | A | A |
| Reset own password | A | A | A | A | A |
| View own profile | O | O | O | O | O |
| Edit own profile | O | X | X | X | X |
| Search patient | X | X | A | A | X |
| View patient profile | O (self) | X | A | A | X |
| Create doctor | X | X | X | A | X |
| Edit doctor | X | X | X | A | X |
| Deactivate doctor | X | X | X | A | X |
| Manage specialties | X | X | X | A | X |
| View own doctor profile | X | O | A | A | X |
| Define own schedule | X | O | X | A | X |
| View own schedule | X | O | A | A | X |
| Change own schedule | X | O | X | A | X |
| Add schedule exception for own schedule | X | O | X | A | X |
| Create slot | X | O (own) | X | A | X |
| Update slot | X | O (own) | X | A | X |
| Deactivate slot | X | O (own) | X | A | X |
| Reactivate slot | X | O (own) | X | A | X |
| Search doctors | A | A | A | A | X |
| View available slots | A | A | A | A | X |
| Book appointment | O (self only) | X | A | A | X |
| Cancel appointment inside policy | O (self only) | X | A | A | X |
| Cancel appointment outside policy | X | X | OV | OV | X |
| Reschedule appointment inside policy | O (self only) | X | A | A | X |
| Reschedule appointment outside policy | X | X | OV | OV | X |
| View own appointments | O | O (own patients only) | A | A | X |
| View appointment details | O (own) | O (own) | A | A | X |
| View daily clinic schedule | X | O (own only) | A | A | X |
| Filter clinic appointments | X | O (own only) | A | A | X |
| Mark visit as waiting | X | O (own only) | A | A | X |
| Mark visit in progress | X | O (own only) | A | A | X |
| Mark appointment completed | X | O (own only) | A | A | X |
| Mark appointment no-show | X | O (own only) | A | A | X |
| View reports | X | X | X | A | X |
| View audit log | X | X | X | A (business audit) | A |
| Manage clinic users | X | X | X | A | X |
| Manage administrative users | X | X | X | X | A |
| Manage role assignments | X | X | X | A (clinic roles only) | A |
| Manage system configuration | X | X | X | X | A |
| Manage access policies | X | X | X | X | A |

---

## Ownership Rules

### `Patient`

- فقط به پروفایل، نوبت‌ها و داده‌های خود دسترسی دارد.
- اجازه مشاهده یا تغییر اطلاعات بیماران دیگر را ندارد.
- فقط می‌تواند نوبت را برای خودش ایجاد کند.

### `Doctor`

- فقط schedule، slotها و appointmentهای منتسب به خود را می‌بیند.
- اجازه مدیریت پزشکان دیگر، تخصص‌ها یا کاربران را ندارد.
- فقط می‌تواند وضعیت مراجعه appointmentهای مربوط به خود را تغییر دهد.

### `Receptionist`

- می‌تواند برای هر بیمار در محدوده کلینیک نوبت ثبت، لغو یا جابه‌جا کند.
- می‌تواند خارج از `Cancellation Window` عملیات لغو/جابجایی را به‌عنوان override انجام دهد.
- مالک مدیریت پزشک، تخصص و کاربران نیست.

### `Clinic Admin`

- مالک عملیات کسب‌وکاری سامانه در سطح کلینیک است.
- تمام capabilityهای `Receptionist` را دارد.
- می‌تواند پزشکان، تخصص‌ها، scheduleها، slotها، گزارش‌ها و کاربران کلینیک را مدیریت کند.

### `System Admin`

- مالک تنظیمات فنی، امنیتی و مدیریتی سامانه است.
- به‌صورت پیش‌فرض عملیات کسب‌وکاری روزمره مانند booking/cancellation/reschedule انجام نمی‌دهد.
- دسترسی به داده‌های بیمار/نوبت فقط در حد audit، troubleshooting یا policyهای امنیتی مجاز است و نباید به‌عنوان actor عملیاتی مدل شود.

---

## Override Rules

| Scenario | Receptionist | Clinic Admin | System Admin |
| --- | --- | --- | --- |
| Cancel outside `Cancellation Window` | Allowed | Allowed | Not a business actor |
| Reschedule outside `Cancellation Window` | Allowed | Allowed | Not a business actor |
| Deactivate reserved slot | Not owner by default | Allowed | Not a business actor |
| View broad clinic operational data | Allowed | Allowed | Only for audit/troubleshooting |
| Change clinic user roles | Not allowed | Allowed | Allowed for admin roles |

---

## Boundary Rules

- `Receptionist` مجاز به مدیریت roleها و تنظیمات سیستم نیست.
- `Doctor` مجاز به دسترسی به اطلاعات بیماران خارج از appointmentهای خود نیست.
- `Clinic Admin` مجاز به مدیریت administrative access در سطح platform نیست.
- `System Admin` نباید جایگزین actorهای کسب‌وکاری روزمره شود.

---

## Authorization Design Notes

- authorization باید در application layer enforce شود.
- ownership check برای `Patient` و `Doctor` باید در همه queryها و commandها اجباری باشد.
- قابلیت‌های override باید در audit log با actor و reason ثبت شوند.
- تست‌های authorization باید حداقل برای موارد `Patient`, `Doctor`, `Receptionist`, `Clinic Admin`, `System Admin` پوشش داده شوند.