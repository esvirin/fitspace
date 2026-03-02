# Оценка в часах — 2 разработчика (split ролей)
## Pilates Space

**Команда:**
- **Dev A:** Mobile App + Web Front (публичный сайт + кабинет клиента)
- **Dev B:** Backend + Admin (кастомная админка + интеграции)

**Предпосылка:** full-time, 35–40 продуктивных часов/неделя на человека.

---

## 1) Распределение scope

### Dev A (Mobile + Web Front)
- Web public + client cabinet UI
- Mobile app (iOS/Android, RN)
- Интеграция с API, UX-состояния, клиентские push/deeplink
- Front E2E/manual regression

### Dev B (Backend + Admin)
- API, доменная логика, БД/миграции
- Auth/RBAC, bookings, packages, waitlist
- Payments (JCC + cash), webhooks, reconciliation
- Admin panel и отчеты
- Worker/queues/notification backend

---

## 2) Оценка по ролям

## Dev B — Backend + Admin

### Discovery + архитектура + contracts
- Требования, ERD, OpenAPI, env/CI baseline: **62–92ч**

### Backend Core
- Auth/RBAC, CRM, schedule, bookings, packages, ledger, audit: **136–202ч**

### Payments + JCC POS initiation
- JCC client, callbacks/polling, idempotency, auto-activation: **74–122ч**

### Waitlist engine + notification backend
- FIFO/confirm timers/admin override + события push: **64–110ч**

### Admin panel
- Dashboard, clients, schedule ops, sales, cash ops, audit/report views: **104–166ч**

### QA/Hardening (backend/admin)
- Integration/E2E, monitoring, backup/restore, bugfix: **66–114ч**

**Итого Dev B:** **506–806ч**

---

## Dev A — Mobile + Web Front

### Web (public + client cabinet)
- Public pages, auth UI, schedule/booking, profile/balance/history, i18n: **80–128ч**

### Mobile app (RN)
- Foundation + auth/profile + schedule/booking + waitlist + purchases + push + release prep: **260–460ч**

### QA/Hardening (front/mobile)
- Регрессии, UX polishing, crash/perf fixes: **40–80ч**

**Итого Dev A:** **380–668ч**

---

## 3) Суммарные трудозатраты (часы)

- **Общие человеко-часы:** **886–1474ч**
- **Реалистичный коридор (практически):** **1020–1240ч**

---

## 4) Сроки календарно (за счет параллельной работы)

При хорошем параллелизме (контракты API стабилизированы к концу Week 2):

- **Оптимистично:** **14–16 недель**
- **Реалистично:** **16–18 недель**
- **С буфером/UAT:** **18–20 недель**

> Сокращение календаря относительно 1 разработчика достигается за счет параллели, но не в 2 раза из-за зависимостей API/contracts/payments.

---

## 5) Ключевые зависимости и риски

- Стабильность OpenAPI и ранний contract freeze (иначе простой Dev A)
- Неопределенности JCC терминального сценария
- Объем кастомной админки (часто растет в середине)
- App Store/Play Console и review latency

### Антирисковые меры
- Freeze contracts после Week 2
- Еженедельные API sync-сессии A↔B
- Feature flags для частичных релизов
- Отдельный буфер на payment edge-cases

---

## 6) Формат сметы

`Budget = (Dev A hours + Dev B hours) × blended rate`

Пример при blended rate 50€/ч:
- 1020ч → 51,000€
- 1240ч → 62,000€
- 1474ч → 73,700€
