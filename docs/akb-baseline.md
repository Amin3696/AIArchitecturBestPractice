# AKB Baseline

**Baseline Version:** 1.1  
**Status:** Reviewed Baseline  
**Last Updated:** 2026-10-08  
**Purpose:** تعیین baseline اولیه برای ساختار مستندات، اسناد authoritative، ترتیب رفع تعارض و وضعیت/نسخه اسناد کلیدی برای تولید سورس.

---

## 1. Normalized Documentation Structure

در این baseline، نام‌گذاری فایل‌ها و پوشه‌ها در `docs/` با قواعد زیر نرمال شده است:

- نام فایل‌ها و پوشه‌ها باید بدون غلط املایی باشند.
- هر سند فقط یک پسوند `.md` داشته باشد.
- نام‌گذاری فایل‌ها در مسیرهای معماری و دامنه باید یکنواخت و قابل پیش‌بینی باشد.

اصلاحات ساختاری انجام‌شده در این baseline:

- `appointmnet` → `appointment`
- `contex-spec.md.md` → `context-spec.md`
- `01-system-contex.md` → `01-system-context.md`
- همه فایل‌های `*.md.md` → `*.md`

---

## 2. Authoritative Documents

اسناد زیر در وضعیت فعلی به‌عنوان مراجع authoritative یا source-of-truth هر حوزه شناخته می‌شوند.

| حوزه | سند authoritative | نقش |
| --- | --- | --- |
| ساختار مستندات | `docs/README.md` | مرجع ساختار پوشه‌ها، طبقه‌بندی اسناد و اصول نگه‌داری مستندات |
| baseline دانش پروژه | `docs/akb-baseline.md` | مرجع baseline فعلی برای مرجعیت اسناد، نسخه/وضعیت و رفع تعارض |
| نیازمندی‌های رسمی | `docs/requirements/srs/srs.md` | مرجع اصلی نیازمندی‌های رسمی سیستم |
| نیازمندی‌های عملکردی | `docs/requirements/functional-requirements.md` | فهرست کاری FRها و ابهام‌های تحلیلی |
| نیازمندی‌های غیرعملکردی | `docs/requirements/non-functional-requirements.md` | فهرست کاری NFRها و معیارهای پذیرش |
| کاربران و ذی‌نفعان | `docs/requirements/users-stakeholders.md` | مرجع بازیگران اصلی و ذی‌نفعان |
| استاندارد API | `docs/api/rest-api-standards.md` | مرجع استانداردهای طراحی REST API |
| تصمیمات معماری | `docs/architecture/decisions/ADR-0001-modular-monolith-architecture.md`، `docs/architecture/decisions/ADR-0002-booking-consistency-and-concurrency-control.md`، `docs/architecture/decisions/ADR-0003-technology-stack-for-mvp.md` | مراجع تصمیم‌های معماری ثبت‌شده (سبک معماری، سازگاری رزرو، استک فناوری) |
| نمای سطح بالا معماری | `docs/architecture/c4models/01-system-context.md` | مرجع نمای Context سیستم |
| نمای کانتینر معماری | `docs/architecture/c4models/02-container.md` | مرجع نمای Container سیستم |

اسناد زیر فعلاً authoritative نیستند و باید به‌عنوان working/derived artefact دیده شوند تا بعد از تکمیل و review ارتقا بگیرند:

- `docs/architecture/domain/context-map.md`
- `docs/architecture/domain/domain-context.md`
- `docs/architecture/domain/ubiquitous-language.md`
- همه اسناد `docs/architecture/domain/contexts/**`
- خروجی‌های `docs/ai/outputs/**`
- پرامپت‌های `docs/ai/prompts/**`

---

## 3. Conflict Resolution Order

در صورت تعارض بین اسناد، ترتیب مرجع نهایی به شکل زیر است:

1. ADRهای تأییدشده یا آخرین ADR معتبر در همان موضوع
2. `docs/requirements/srs/srs.md`
3. `docs/requirements/functional-requirements.md` و `docs/requirements/non-functional-requirements.md`
4. `docs/api/rest-api-standards.md`
5. C4 modelها و اسناد معماری تکمیلی
6. `docs/requirements/interviews/**` به‌عنوان منبع خام و upstream
7. `docs/ai/outputs/**` به‌عنوان artefact مشتق‌شده و غیرنهایی

قواعد رفع تعارض:

- اگر بین مصاحبه و SRS تعارض وجود داشت، `SRS` مرجع رسمی فعلی است مگر اینکه نیاز به بازنگری رسمی ثبت شود.
- اگر بین SRS و ADR تعارض وجود داشت، ADR فقط در حوزه تصمیم معماری خودش مقدم است.
- اگر بین FR/NFR و SRS تعارض وجود داشت، SRS مرجع نهایی است و FR/NFR باید هم‌راستا شوند.
- اگر بین خروجی AI و اسناد انسانی تعارض وجود داشت، خروجی AI صرفاً کمکی است و authoritative نیست.

---

## 4. Key Document Registry

| سند                                                                                  | نسخه | وضعیت             | نوع مرجع               | توضیح                                                        |
| ------------------------------------------------------------------------------------ | ---- | ----------------- | ---------------------- | ------------------------------------------------------------ |
| `docs/README.md`                                                                     | 1.0  | Reviewed          | Authoritative          | ساختار و سیاست نگه‌داری مستندات                              |
| `docs/akb-baseline.md`                                                               | 1.1  | Reviewed Baseline | Authoritative          | baseline فعلی AKB و مرجع رفع تعارض                           |
| `docs/requirements/srs/srs.md`                                                       | 1.0  | Draft             | Authoritative          | نیازمندی‌های رسمی سیستم                                      |
| `docs/requirements/functional-requirements.md`                                       | 0.1  | Working Draft     | Supporting             | FRهای کاری و ابهام‌ها                                        |
| `docs/requirements/non-functional-requirements.md`                                   | 0.1  | Working Draft     | Supporting             | NFRهای کاری و معیارهای اولیه                                 |
| `docs/requirements/users-stakeholders.md`                                            | 0.1  | Working Draft     | Supporting             | بازیگران و ذی‌نفعان                                          |
| `docs/api/rest-api-standards.md`                                                     | 1.0  | Draft Standard    | Authoritative          | استاندارد طراحی REST API                                     |
| `docs/architecture/decisions/ADR-0001-modular-monolith-architecture.md`               | 1.0  | Proposed          | Authoritative in Scope | تصمیم معماری ثبت‌شده برای سبک سیستم                          |
| `docs/architecture/decisions/ADR-0002-booking-consistency-and-concurrency-control.md` | 1.0  | Accepted          | Authoritative in Scope | تصمیم معماری برای جلوگیری از double booking و atomicity رزرو |
| `docs/architecture/decisions/ADR-0003-technology-stack-for-mvp.md`                    | 1.0  | Accepted          | Authoritative in Scope | تصمیم معماری برای انتخاب استک فناوری MVP                     |
| `docs/architecture/c4models/01-system-context.md`                                    | 0.1  | Working Draft     | Supporting             | نمای سطح بالا از بازیگران و سیستم                            |
| `docs/architecture/c4models/02-container.md`                                         | 0.1  | Working Draft     | Supporting             | نمای کانتینرها و اجزای اجرایی                                |
| `docs/architecture/domain/context-map.md`                                            | 0.5  | Working Draft     | Supporting             | نقشه فعلی bounded contextها و روابط آن‌ها                    |
| `docs/architecture/domain/domain-context.md`                                         | 0.5  | Working Draft     | Supporting             | دامنه، capabilityها و مرز مسئولیت‌ها                         |
| `docs/architecture/domain/access-control-matrix.md`                                  | 0.5  | Working Draft     | Supporting             | ماتریس نقش/مجوز و overrideهای مجاز                           |
| `docs/architecture/domain/state-machines.md`                                         | 0.5  | Working Draft     | Supporting             | مدل وضعیت‌های Appointment، Visit و Slot                      |
| `docs/architecture/domain/ubiquitous-language.md`                                    | 0.5  | Working Draft     | Supporting             | واژگان مشترک پروژه                                           |
| `docs/architecture/domain/contexts/appointment/context-spec.md`                      | 0.5  | Working Draft     | Supporting             | مشخصات context رزرو و lifecycle نوبت                         |
| `docs/architecture/domain/contexts/doctor/context-spec.md`                           | 0.5  | Working Draft     | Supporting             | مشخصات context پزشک، schedule و slot                         |
| `docs/architecture/domain/contexts/patient/context-spec.md`                          | 0.5  | Working Draft     | Supporting             | مشخصات context بیمار و پروفایل                               |
| `docs/architecture/domain/contexts/notification/context-spec.md`                     | 0.5  | Working Draft     | Supporting             | مشخصات context اعلان و reminder                              |

---

## 5. Usage Rules for Source Generation

برای تولید سورس، ترتیب استفاده از دانش پروژه باید به شکل زیر باشد:

1. ابتدا ADRها و SRS خوانده شوند.
2. سپس FR/NFR و users/stakeholders برای extraction جزئیات استفاده شوند.
3. بعد از آن API standards و C4 modelها برای شکل‌دهی لایه application و interface استفاده شوند.
4. اسناد دامنه فقط زمانی مبنای اصلی تولید سورس باشند که از حالت `Empty Draft` یا `Working Draft` خارج شوند.
5. خروجی‌های AI فقط برای تسریع تحلیل استفاده شوند و مبنای نهایی تولید کد نباشند.

---

## 6. Next Baseline Upgrade Conditions

baseline بعدی زمانی منتشر شود که حداقل موارد زیر تکمیل شده باشند:

- `context-map`, `domain-context`, `ubiquitous-language` تکمیل شوند.
- `context-spec`های چهار bounded context اصلی تکمیل شوند.
- role/permission matrix نهایی شود.
- state machineهای `Appointment` و `Slot` ثبت شوند.
- API contractهای MVP از سطح standard به سطح product-specific contract برسند.
