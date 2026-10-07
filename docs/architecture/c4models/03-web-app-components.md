# C4 Model - Level 3: Web App Components

```mermaid
C4Component
title Clinic Appointment System - Web App Components

Person(patient, "Patient")
Person(doctor, "Doctor")
Person(receptionist, "Receptionist")
Person(clinicAdmin, "Clinic Admin")
Person(systemAdmin, "System Admin")

Container_Boundary(web, "Web App (SPA)") {
    Component(shell, "App Shell & Router", "SPA shell", "Bootstraps the application, routing, shared layout, and navigation guards")
    Component(session, "Session & Access Client", "Client auth/session component", "Stores session state, applies route guards, and resolves role-aware UI access")
    Component(bookingUi, "Booking & Appointment UI", "Patient/Reception UI", "Searches doctors and slots, books, cancels, and reschedules appointments")
    Component(doctorUi, "Doctor Schedule & Visit UI", "Doctor UI", "Shows doctor schedule and allows visit status updates")
    Component(adminUi, "Clinic Admin UI", "Admin UI", "Manages doctors, schedules, slots, users, and reports")
    Component(apiClient, "API Client", "HTTP client", "Calls backend REST endpoints and normalizes request/response handling")
}

Container(api, "Backend API", "API-centric Modular Monolith", "Exposes REST endpoints")

Rel(patient, shell, "Uses")
Rel(doctor, shell, "Uses")
Rel(receptionist, shell, "Uses")
Rel(clinicAdmin, shell, "Uses")
Rel(systemAdmin, shell, "Uses")

Rel(shell, session, "Checks session and route access")
Rel(shell, bookingUi, "Loads booking flows")
Rel(shell, doctorUi, "Loads doctor flows")
Rel(shell, adminUi, "Loads admin flows")

Rel(bookingUi, apiClient, "Uses")
Rel(doctorUi, apiClient, "Uses")
Rel(adminUi, apiClient, "Uses")
Rel(session, apiClient, "Uses for auth/session endpoints")

Rel(apiClient, api, "Calls REST API over HTTPS")
```

## Components

| Component | Responsibility |
| --- | --- |
| `App Shell & Router` | route composition, layout, navigation, guarded sections |
| `Session & Access Client` | login state, session refresh, logout, client-side role-aware access |
| `Booking & Appointment UI` | patient/reception workflows for search, booking, cancellation, reschedule |
| `Doctor Schedule & Visit UI` | doctor workflows for schedule visibility and visit-state updates |
| `Clinic Admin UI` | doctor/schedule/slot/user/report management |
| `API Client` | common HTTP transport, error mapping, request metadata |

## Notes

- UI-level access control صرفاً برای UX و navigation guard است؛ enforcement نهایی باید در `Backend API` انجام شود.
- `Booking & Appointment UI` هم برای `Patient` و هم برای `Receptionist` استفاده می‌شود، اما capabilityهای قابل‌نمایش بر اساس نقش محدود می‌شود.
- `System Admin` در UI فقط به بخش‌های فنی/امنیتی دسترسی دارد، نه جریان‌های روزمره booking.
