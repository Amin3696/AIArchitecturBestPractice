# Exception Flowهای حیاتی (Concurrency / Authorization)

## EX-01: Concurrency در رزرو یک Slot

### سناریو

دو درخواست همزمان برای یک `slotId` ارسال می‌شود.

### انتظار رفتاری

1. فقط یک transaction commit می‌شود.
2. درخواست دیگر با conflict رد می‌شود.
3. کد خطای برگشتی: `APPOINTMENT_SLOT_UNAVAILABLE`.

### Trace

- ADR: `ADR0002`
- FR: `FR-22`

---

## EX-02: Replayed Request با Idempotency Key یکسان

### سناریو

کلاینت به‌علت timeout همان درخواست را با همان `X-Idempotency-Key` تکرار می‌کند.

### انتظار رفتاری

- اگر payload یکسان باشد: همان نتیجه قبلی بازگردانده شود.
- اگر payload متفاوت باشد: `409` با `IDEMPOTENCY_KEY_REUSED_WITH_DIFFERENT_PAYLOAD`.

---

## EX-03: Authorization Failure (Ownership)

### سناریو

بیمار A تلاش می‌کند Appointment بیمار B را ببیند/لغو کند.

### انتظار رفتاری

- پاسخ `403` با `AUTH_FORBIDDEN` یا `OWNERSHIP_VIOLATION`.
- هیچ تغییری در وضعیت Appointment ایجاد نشود.

---

## EX-04: Authorization Failure (Role)

### سناریو

`Doctor` تلاش می‌کند عملیات مدیریت تخصص یا ایجاد پزشک انجام دهد.

### انتظار رفتاری

- پاسخ `403` با `AUTH_FORBIDDEN`.
- عملیات رد و audit امنیتی ثبت شود.

---

## EX-05: Invalid State Transition

### سناریو

تلاش برای `COMPLETED` کردن یک Appointment که قبلاً `CANCELLED` شده است.

### انتظار رفتاری

- پاسخ `409` با `APPOINTMENT_INVALID_STATE_TRANSITION` یا `APPOINTMENT_ALREADY_TERMINAL`.
- داده قبلی حفظ شود.
