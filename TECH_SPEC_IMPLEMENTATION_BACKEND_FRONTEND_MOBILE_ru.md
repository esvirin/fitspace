# Техническое ТЗ на реализацию
## Проект: Pilates Space — Backend + Frontend + Mobile

**Версия:** 1.0 (draft)  
**Дата:** 2026-03-02  
**Основание:** бизнес-ТЗ MVP и целевые процессы оплаты/записи через GymMaster

---

## 1. Цель реализации

Реализовать цифровой контур записи и оплаты для Pilates Space, включающий:
- backend интеграционный слой (GymMaster + платежи + уведомления),
- веб-фронтенд (сайт + клиентский кабинет),
- мобильное приложение (iOS/Android) для self-service клиентов.

Ключевой результат: администратор и клиенты работают в едином цифровом процессе, с минимизацией ручных операций.

---

## 2. Архитектура решения (высокоуровнево)

## 2.1 Компоненты
1. **Web Frontend** (публичный сайт + личный кабинет).
2. **Mobile App** (iOS/Android).
3. **Backend API** (BFF + интеграционный слой).
4. **GymMaster API** (клиенты, расписание, бронирования, покупки/балансы).
5. **Payment Connectors**:
   - JCC Smart (карты),
6. **Notification Service** (push/email, при необходимости SMS).
7. **DB/Storage** для служебных данных интеграции, логов, аудита, идемпотентности.

## 2.2 Принципы
- GymMaster — master system по клиентам/посещениям/продуктам.
- Backend хранит только то, что нужно для orchestration, аудита и надежности.
- Любые платежные операции идут через провайдеров, без хранения card PAN/CVV в системе.

---

## 3. Границы ответственности по слоям

## 3.1 Backend
- API для веба и мобайла.
- Интеграция с GymMaster API.
- Интеграция с JCC.
- Webhook endpoints, валидация подписи, обработка событий.
- Синхронизация и reconciliation платежей.
- Бизнес-правила (статусы покупок, отмены, дедупликация запросов).
- Push orchestration.

## 3.2 Frontend (Web)
- Публичный сайт (контентные страницы).
- Личный кабинет клиента:
  - регистрация/логин,
  - расписание,
  - бронирование/отмена,
  - покупка разового/пакета,
  - история посещений/покупок.
- Админ-панель (минимум для оператора, если требуется отдельный UI кроме GymMaster).

## 3.3 Mobile App
- Авторизация клиента.
- Расписание, бронь/отмена.
- Покупка/оплата.
- Push-уведомления.
- Экран профиля и баланса пакета.

---

## 4. Данные и сущности

## 4.1 Основные сущности
- `ClientProfile` (ссылка на GymMaster Member ID)
- `ClassSession`
- `Booking`
- `Product` (single class, package 10/20/30, personal training)
- `Purchase`
- `PaymentTransaction`
- `PaymentAttempt`
- `WebhookEvent`
- `Notification`

## 4.2 Минимальные поля (backend)

### Purchase
- `id`
- `clientId`
- `productType` (single/package/personal)
- `quantity` (1/10/20/30)
- `amount`
- `currency` (EUR)
- `status` (`initiated|pending_payment|paid|failed|cancelled|manual_review`)
- `source` (`web|mobile|admin_desk`)
- `externalGymMasterRef` (optional)

### PaymentTransaction
- `id`
- `purchaseId`
- `provider` (`jcc|cash`)
- `providerPaymentId`
- `status` (`created|authorized|captured|failed|expired|refunded|pending`)
- `rawPayload` (json, masked)
- `confirmedAt`

### WebhookEvent
- `id`
- `provider`
- `eventType`
- `eventId`
- `signatureValid`
- `receivedAt`
- `processedAt`
- `processingStatus`
- `dedupKey`

---

## 5. API-контракты (внутренний backend API)

## 5.1 Auth
- `POST /api/v1/auth/signup`
- `POST /api/v1/auth/login`
- `POST /api/v1/auth/refresh`
- `POST /api/v1/auth/logout`

## 5.2 Расписание/бронирования
- `GET /api/v1/classes?dateFrom=&dateTo=&location=&trainer=`
- `POST /api/v1/bookings` (classId)
- `POST /api/v1/bookings/{id}/cancel`
- `GET /api/v1/bookings/my`

## 5.3 Продукты/покупки
- `GET /api/v1/products`
- `POST /api/v1/purchases/initiate`
- `GET /api/v1/purchases/{id}`
- `GET /api/v1/purchases/my`

## 5.4 Платежи
- `POST /api/v1/payments/jcc/create`
- `POST /api/v1/payments/cash/confirm` (admin only)
- `GET /api/v1/payments/{id}/status`

## 5.5 Webhooks
- `POST /api/v1/webhooks/jcc`

## 5.6 Админ-операции
- `POST /api/v1/admin/clients/create-or-link`
- `POST /api/v1/admin/purchases/create`
- `POST /api/v1/admin/purchases/{id}/mark-paid-cash`

---

## 6. Ключевые бизнес-процессы (реализация)

## 6.1 Покупка пакета на ресепшене — карта (JCC)
1. Админ выбирает/создает клиента.
2. Создает `Purchase` в статусе `initiated`.
3. Backend создает платеж через JCC API (`pending_payment`).
4. Клиент платит на терминале.
5. JCC webhook/status callback → backend верифицирует событие.
6. При `captured/success` backend:
   - ставит `Purchase=paid`,
   - вызывает GymMaster API для фиксации покупки,
   - сохраняет `externalGymMasterRef`.
7. В случае ошибки: `failed`/`manual_review` + алерт админу.

## 6.2 Покупка пакета на ресепшене — наличные
1. Админ создает `Purchase`.
2. После фактического приема наличных жмет «Подтвердить кэш».
3. Backend переводит `Purchase=paid` с `provider=cash`.
4. Backend фиксирует покупку в GymMaster.

---

## 7. Интеграция с GymMaster

## 7.1 Обязательные операции
- Поиск/создание клиента.
- Получение расписания и доступности мест.
- Создание/отмена бронирования.
- Покупка membership/product.
- Получение баланса/истории посещений.

## 7.2 Технические требования
- Retry с backoff для временных ошибок.
- Idempotency key для операций покупки/брони.
- Логирование request/response (без чувствительных данных).
- Мониторинг ошибок API GymMaster.

## 7.3 Доступ клиента к профилю, созданному админом
Реализовать один из штатных GymMaster-потоков (по фактической поддержке):
- invite email,
- reset password,
- claim account.

Статус: **Not confirmed**, проверить в discovery.

---

## 8. Платежный модуль: требования

## 8.1 Общие
- PCI DSS scope минимизируется (hosted fields/redirect/provider-side capture, где возможно).
- Все webhook-и проходят валидацию подписи/секрета.
- Идемпотентная обработка webhook событий.
- Разделение статусов: `payment_status` и `business_purchase_status`.

## 8.2 JCC
- Реализовать API client для create payment / status check.
- Реализовать callback endpoint (если поддерживается).
- Fallback polling по `providerPaymentId` при недоставке callback.

---

## 9. Frontend (Web) — детальные требования

## 9.1 Публичная часть
- Главная (2 направления: Pilates + будущий Lagree).
- О студии, тренеры, контакты, галерея.
- CTA: «Записаться», «Скачать приложение», «Личный кабинет».

## 9.2 Личный кабинет
- Регистрация/логин.
- Просмотр и фильтрация расписания.
- Бронь/отмена.
- Покупка продуктов.
- Экран «Мой баланс» (остаток пакета + история).
- Экран «Мои оплаты» (статусы транзакций).

## 9.3 UX-ограничения
- Не более 3 кликов до брони занятия.
- Ясные статусы платежа: «успешно / в обработке / ошибка».
- Ясные действия при `manual_review`.

---

## 10. Mobile (iOS/Android) — детальные требования

## 10.1 Функционал MVP
- Email/phone login.
- Календарь занятий.
- Бронь/отмена.
- Покупка/оплата.
- Push уведомления.
- Профиль, баланс, история.

## 10.2 Push-события
- booking_confirmed
- class_reminder
- class_changed
- payment_success
- payment_pending_review

## 10.3 Технические требования
- Поддержка последних 2 мажорных версий iOS/Android.
- Crash-free rate > 99.5% (целевой KPI после релиза).

---

## 11. Безопасность и соответствие

- GDPR: lawful basis, privacy policy, consent where required.
- Шифрование in-transit (TLS 1.2+), secrets в vault/env.
- RBAC для админ-операций.
- Audit log неизменяемого типа для финансовых действий.
- Маскирование PII/платежных данных в логах.

---

## 12. Наблюдаемость и эксплуатация

- Метрики:
  - успешность бронирований,
  - конверсия в оплату,
  - доля `manual_review`,
  - latency API,
  - ошибки интеграций.
- Алерты:
  - рост failed платежей,
  - недоступность GymMaster API,
  - webhook lag > N минут.
- Техпанель для поддержки: поиск по purchaseId/clientId/providerPaymentId.

---

## 13. Тестирование

## 13.1 Backend
- Unit + integration tests.
- Contract tests к GymMaster/JCC sandbox.
- Webhook tests (валидная/невалидная подпись, дубликаты, reorder).

## 13.2 Frontend/Mobile
- E2E критических путей:
  1) login → booking,
  2) purchase → payment success,
  3) cancel booking,
  4) pending payment → manual review.

## 13.3 UAT (бизнес)
- Администратор выполняет 20 типовых операций без ручных обходов.
- Клиентские сценарии проходят на реальных тестовых данных.

---

## 14. Этапы реализации (delivery)

## Sprint 0 (1–2 недели)
- Discovery API (GymMaster, JCC).
- Подтверждение контрактов и ограничений.
- Финализация архитектуры и ERD.

## Sprint 1–2
- Backend foundation, auth, classes/bookings.
- Web личный кабинет (без оплаты).

## Sprint 3–4
- Платежи JCC + cash flow + GymMaster purchase sync.
- Admin operations.

## Sprint 5
- Notification service.

## Sprint 6
- Mobile release candidate.
- UAT + bugfix + production readiness.

---

## 15. Критерии готовности (Definition of Done)

1. Все критические API endpoint задокументированы (OpenAPI).
2. Покупка/оплата/начисление занятий работает end-to-end.
3. Webhooks обрабатываются идемпотентно и безопасно.
4. Клиент может записаться и оплатить через web и mobile.
5. Админ может провести продажу в студии по 2 методам: карта/JCC, кэш.
6. Есть логи, алерты, дешборды, инструкции поддержки.

---

## 16. Открытые вопросы (блокеры)

1. GymMaster: точный API-метод для «продажи пакета от имени админа» и подтверждения покупки.
2. JCC Smart: подтвержденный сценарий передачи суммы на терминал из внешней системы.
4. Требуется ли фискализация/инвойсинг на стороне Кипра в рамках MVP.
5. Нужен ли отдельный admin UI или достаточно GymMaster + служебная панель операций.

---

## 17. Рекомендуемый стек (предложение)

_Not confirmed, можно скорректировать под команду._

- Backend: Node.js (NestJS) + PostgreSQL + Redis + BullMQ (reconciliation jobs)
- Web: Next.js
- Mobile: React Native
- Observability: Sentry + Prometheus/Grafana (или managed APM)
- CI/CD: GitHub Actions

---

## 18. Артефакты, которые должны быть подготовлены после утверждения ТЗ

1. OpenAPI spec (backend).
2. ERD + migration plan.
3. Sequence diagrams по 3 платежным сценариям.
4. UX flow map (web/mobile).
5. UAT checklist для администратора.
6. Runbook поддержки инцидентов платежей.
