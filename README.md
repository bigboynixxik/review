# Зона ответственности

**вклад в проект: серверная часть, ядро бизнес-логики, хранилища, проектная документация**

Заложил каркас backend-сервиса, спроектировал хранилища и реализовал основной сценарий очереди на Go, а также подготовил аналитику и архитектурную документацию проекта.

### Документация и проектирование

- Описал контекст проекта, бизнес-цели и user stories (`README.md`, `docs/requirements.md`).
- Составил бизнес-, пользовательские, функциональные и нефункциональные требования (`docs/requirements.md`).
- Нарисовал C4-диаграммы уровня Context и Container (`docs/c4_context.md`, `docs/c4_container.md`).
- Описал работу redis и postgres в проекте (`docs/storage/redis.md`,`docs/storage/redis.md`)

### Каркас проекта и инфраструктура сборки

- Инициализировал структуру репозитория и Go-модуль (`backend/go.mod`), `.gitignore`, `.dockerignore`.
- Написал `Dockerfile` для сборки backend-образа (Go 1.26.4).
- Настроил статический анализ: конфигурация `golangci-lint` (`backend/.golangci.yaml`).
- Реализовал служебные пакеты:
    - `pkg/logger` — настройка структурного логгера на `slog`
    - `pkg/closer` — упорядоченное завершение работы компонентов
    - `pkg/migrator` — прогон миграций на `goose`
    - `pkg/postgres_settings` — пул соединений к Postgres на `pgx`
    - `pkg/redis_settings` — подключение к Redis
- Написал middleware логирования, прокидывающее логгер в `context` (`internal/transport/mw/logger.go`).
- Настроил парсинг конфигурации из переменных окружения: инфраструктурные и бизнес-параметры (`internal/config`, `.env.example`).

### Слой данных

- Спроектировал схему БД и написал первую миграцию (`internal/migrations/001_init_schema.sql`): таблицы `rights`, `queue_memberships`, `product_stock`, ENUM-типы статусов, индексы и ограничения целостности.
- Описал доменные модели и типизированные ошибки (`internal/models`): `Right`, `QueueMembership`, `ProductStock`, `Status`, `errors; покрыл статусы тестами.
- Реализовал Postgres-репозиторий (`internal/repository/postgres/durable.go`): сохранение прав, upsert членства в очереди, транзакционное списание остатка вместе с погашением права. Покрыл интеграционными тестами на `testcontainers`.
- Реализовал Redis-репозиторий как горячий путь с защитой от гонок через атомарные Lua-скрипты (`internal/repository/redis/cache.go`):
    - `TryAllocate` / `CommitPurchase` / `InitStock` — атомарная работа с остатком товара
    - `Enqueue` / `RemoveFromQueue` — FIFO-очередь на монотонном счётчике
    - кэш членства и выданных прав для быстрой валидации перед оплатой
    - `AddToExpiryTimer` / `RemoveFromExpiryTimer` — отслеживание дедлайнов офферов и прав
    - `PublishEvent` — публикация изменений статуса для realtime-клиентов
    - `GetQueueMetrics` — получение позиции и доступных единиц за один pipeline
  - Покрыл интеграционными и race-condition тестами на `testcontainers`.

### Бизнес-логика очереди

- Реализовал ядро `QueueService` (`internal/service/queue_service.go`):
    - `JoinQueue` и `processAllocation` — вход в очередь с идемпотентностью и немедленной выдачей права, если товар доступен
    - `AcceptOffer` / `DeclineOffer` — принятие и отклонение частичного оффера
    - `AdvanceQueue` — продвижение очереди после освобождения остатка
    - `ValidateRight` / `ProcessPayment` — проверка права перед checkout и завершение покупки
    - `ProcessExpirations` — обработка истёкших офферов и прав: возврат остатка, смена статуса, продвижение очереди
- Выделил интерфейс для фонового воркера экспираций (`internal/service/worker/interfaces.go`).
- Реализовал `CalculateETA` (`internal/service/membership.go`) — позиция пользователя в очереди и прогноз времени ожидания на основе метрик Redis и параметра `AvgPaymentTime` из конфигурации.
- Описал контракты репозиториев и сервисов (`internal/service/interfaces.go`, `internal/transport/interfaces.go`).
- Покрыл бизнес-логику unit-тестами с `gomock` и `testify`-сьютами: `queue_service_test`, `join_queue_test.go`, `accept_offer_test.go`, `decline_offer_test.go`, `advance_queue_test.go`, `validate_right_test.go`, `process_payment_test.go`, `process_expirations_test.go`, `membership_test.go`.

### Остальное влияние на проект

- Добавил логирование ошибок в общий хелпер ответов транспортного слоя (`internal/transport/api/response.go`).
- Вёл репозиторий как владелец: ревью и приём пул-реквестов команды в `develop`.
- Активное обсуждение архитектуры и дополнительного функционала
