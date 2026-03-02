# ROADMAP — 2 разработчика (Mobile+Web Front / Backend+Admin)
## Pilates Space (16–20 недель)

**Dev A:** Mobile + Web Front  
**Dev B:** Backend + Admin

---

## Week 1–2: Foundation + contract-first

### Dev B
- ERD, доменная модель, OpenAPI v0
- Auth/RBAC skeleton
- CI/CD + staging + базовые миграции

### Dev A
- UI foundation (design tokens, nav, app shell)
- Web public layout + mobile shell
- API client scaffolding по OpenAPI

### Milestone
- Staging поднят, `OpenAPI v1` зафиксирован (contract freeze для core flows).

---

## Week 3–5: Core delivery параллельно

### Dev B
- CRM, schedule, bookings, packages, ledger, audit
- Базовые admin API endpoints

### Dev A
- Web: auth, schedule, booking/cancel, profile/balance
- Mobile: auth, schedule list, booking/cancel

### Milestone
- E2E сценарий: signup → booking → cancel (web+mobile через staging API).

---

## Week 6–8: Admin + payment-ready UX

### Dev B
- Admin v1: dashboard, clients, schedule ops, sales/cash ops
- JCC connector draft + payment statuses

### Dev A
- Web public polishing + client cabinet v1
- Mobile: profile/history + purchase UX states

### Milestone
- Админ может продавать cash, клиент видит баланс/историю на web/mobile.

---

## Week 9–11: JCC POS initiation + waitlist

### Dev B
- `terminal/initiate`, callbacks/webhooks, polling fallback
- payment_events + auto-activation purchase
- waitlist backend (FIFO, timers, override)

### Dev A
- Web/mobile waitlist flows (join/confirm/leave)
- Payment status UX (success/failed/manual_review)

### Milestone
- POS initiation работает end-to-end, waitlist закрыт функционально.

---

## Week 12–14: Push + hardening

### Dev B
- Notification service backend, retry/dedup, delivery logs
- Monitoring + alerts + backup/restore checks

### Dev A
- Push token flow (iOS/Android), deep links
- Final UX polish и crash/perf fixes

### Milestone
- Ключевые push-события доставляются, наблюдаемость включена.

---

## Week 15–16: UAT + release candidate

### Совместно
- Full regression (web/mobile/admin/backend)
- Fix critical/high bugs
- Release checklist + rollback plan

### Milestone
- Go/No-Go решение, RC готов к production rollout.

---

## Week 17–20: Буфер (по факту)

- UAT feedback
- App Store/Play review loops
- UX/reporting polishing
- Hardening payment edge-cases

---

## Контрольные правила команды

1. **Contract-first:** новые поля/эндпоинты только через OpenAPI change log.
2. **Weekly sync A↔B:** минимум 2 API sync в неделю.
3. **Definition of Done:** код + тесты + логи + роли + docs.
4. **Scope control:** новые фичи после Week 2 только в Phase 2 backlog.
