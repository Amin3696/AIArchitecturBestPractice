# C4 Model - Level 3: Background Worker Components

```mermaid
C4Component
title Clinic Appointment System - Background Worker Components

Container_Boundary(worker, "Background Worker") {
    Component(jobConsumer, "BullMQ Job Consumer", "Worker runtime", "Consumes reminder and notification jobs from queues")
    Component(notificationOrchestrator, "Notification Orchestrator", "Application", "Loads job data, decides execution path, and coordinates delivery")
    Component(templateComposer, "Template Composer", "Application", "Builds message payloads from notification templates and business facts")
    Component(channelPolicy, "Channel Policy Resolver", "Domain/Application", "Resolves SMS/email channel selection and fallback policy")
    Component(smsAdapter, "SMS Provider Adapter", "Infrastructure", "Sends SMS through the configured SMS provider")
    Component(emailAdapter, "Email Provider Adapter", "Infrastructure", "Sends email through the configured email provider")
    Component(retryHandler, "Retry & Failure Handler", "Infrastructure/Application", "Applies retry policy, failure recording, and dead-letter behavior")
}

Container(redis, "Redis", "Redis + BullMQ backend", "Stores queued jobs")
ContainerDb(db, "PostgreSQL", "Transactional database", "Stores notification definitions, delivery attempts, and supporting business data")
System_Ext(sms, "SMS Provider", "External SMS service")
System_Ext(email, "Email Provider", "External email service")

Rel(redis, jobConsumer, "Supplies queued jobs")
Rel(jobConsumer, notificationOrchestrator, "Delegates job execution")
Rel(notificationOrchestrator, db, "Reads notification/job/business data")
Rel(notificationOrchestrator, templateComposer, "Builds message payload")
Rel(notificationOrchestrator, channelPolicy, "Resolves channel and fallback")
Rel(channelPolicy, smsAdapter, "Uses when SMS is selected")
Rel(channelPolicy, emailAdapter, "Uses when email is selected")
Rel(notificationOrchestrator, retryHandler, "Reports failure or retry need")
Rel(retryHandler, db, "Stores attempts and failure state")
Rel(retryHandler, redis, "Requeues retryable jobs")
Rel(smsAdapter, sms, "Sends SMS")
Rel(emailAdapter, email, "Sends email")
```

## Components

| Component | Responsibility |
| --- | --- |
| `BullMQ Job Consumer` | receives async jobs from queue |
| `Notification Orchestrator` | central execution path for notification and reminder jobs |
| `Template Composer` | transforms template + data into final message payload |
| `Channel Policy Resolver` | applies channel selection and fallback decisions |
| `SMS Provider Adapter` | isolated adapter for SMS integration |
| `Email Provider Adapter` | isolated adapter for email integration |
| `Retry & Failure Handler` | retry scheduling, failure persistence, and operational safety |

## Notes

- worker درباره valid بودن رزرو تصمیم نمی‌گیرد؛ فقط event/job معتبر را پردازش می‌کند.
- failure در ارسال اعلان نباید integrity رزرو را بر هم بزند، اما باید traceable و retryable باشد.
- policyهای دقیق retry/timeout/fallback هنوز در checklist بخش integration باقی مانده‌اند و بعداً باید روی این Level 3 sync شوند.
