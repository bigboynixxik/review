# Зона ответственности

**вклад в проект: серверная часть, ядро бизнес-логики, хранилища, проектная документация**

Заложил каркас backend-сервиса, спроектировал хранилища и реализовал основной сценарий очереди на Go, а также подготовил аналитику и архитектурную документацию проекта.

### Документация и проектирование

- Описал контекст проекта, бизнес-цели и user stories (`README.md`, `docs/requirements.md`). ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/c6c8a959b659138c01388d2c15a9f6f6b3a0ab53))
- Составил бизнес-, пользовательские, функциональные и нефункциональные требования (`docs/requirements.md`). ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/d1807dc48565567d64c530775e23d26055f64cf0))
- Нарисовал C4-диаграммы уровня Context и Container (`docs/c4_context.md`, `docs/c4_container.md`). ([Context](https://github.com/bigboynixxik/team-10-queue-serivce/commit/746c26f634bffbeb41912740a2ce87384f1deaf9), [Container](https://github.com/bigboynixxik/team-10-queue-serivce/commit/8e6d6657c3baf0d0796f2abda8303134073f58c0))
- Описал работу redis и postgres в проекте (`docs/storage/redis.md`, `docs/storage/postgres.md`)
- Описал метрики продавцов, котоыре собираются с помощью продавца: какие, зачем и как их собирать. Добавил эндпоинт в api.yaml (`docs/product_metrics.md`, `docs/api.yml`) [PR](https://github.com/bigboynixxik/team-10-queue-serivce/pull/23)

### Каркас проекта и инфраструктура сборки

- Инициализировал структуру репозитория и Go-модуль (`backend/go.mod`), `.gitignore`, `.dockerignore`. ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/0c9208267d3f31b7d3c9dab8b43b65217b50960d), [Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/e8c50523268221ae4dadcba353b0ba2cd52f5d79))
- Написал изначальный `Dockerfile` для сборки backend-образа (Go 1.26.4). ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/2fb0bfb7d97a8bd0ccef37851338c40b13991a82), [Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/70c252136a6f67d3a740074d88dc69634dcbf42f))
- Настроил статический анализ: конфигурация `golangci-lint` (`backend/.golangci.yaml`). ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/7e579106dea78275359d50c230546d3904d56a02), [Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/5b220d7d4561eba1de0b3a0d6abc39f2039a0dc8))
- Реализовал служебные пакеты:
- `pkg/logger` — настройка структурного логгера на `slog` ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/7bb6b0ed1b66ec39183ef8ea49c5918772ba8ca2))
- `pkg/closer` — упорядоченное завершение работы компонентов ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/f5dc9fb99e6f9e4455a636fe1674565601216016))
- `pkg/migrator` — прогон миграций на `goose` ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/69dc34a40fa479338845ed4bf18db5cc409734c7))
- `pkg/postgres_settings` — пул соединений к Postgres на `pgx` ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/845ceaf67b60319430a6304ba804fd11f4410fad))
- `pkg/redis_settings` — подключение к Redis ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/24607f5e228cc72510cc66c08bbebb00e1c7ff10))
- Написал middleware логирования, прокидывающее логгер в `context` (`internal/transport/mw/logger.go`). ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/d4022e2a71646ab0e76ac671f39cb7e3e15b5f13))
- Настроил парсинг конфигурации из переменных окружения: инфраструктурные и бизнес-параметры (`internal/config`, `.env.example`). ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/5788598b8444524908a0d047f9917b47f4fd2560), [Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/84ae928700f032fb5689c8594a0700ae0921109d), [Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/57756ada72a38ecf4cb5bd7d3feda05acecd8165))

### Слой данных

- Спроектировал схему БД и написал первую миграцию (`internal/migrations/001_init_schema.sql`): таблицы `rights`, `queue_memberships`, `product_stock`, ENUM-типы статусов, индексы и ограничения целостности. ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/69dc34a40fa479338845ed4bf18db5cc409734c7))
- Описал доменные модели и типизированные ошибки (`internal/models`): `Right`, `QueueMembership`, `ProductStock`, `Status`, `errors`; покрыл статусы тестами. ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/932a5681b7c8181264333c117e710252f44d9798), [Tests](https://github.com/bigboynixxik/team-10-queue-serivce/commit/b7977d3dd1f0c2f2fec826a7520b6e2686c16fd5))
- Реализовал Postgres-репозиторий (`internal/repository/postgres/durable.go`): сохранение прав, upsert членства в очереди, транзакционное списание остатка вместе с погашением права. Покрыл интеграционными тестами на `testcontainers`. ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/b81b5ffc1cfde2bc9e70c8711cfc52a500804805), [Tests](https://github.com/bigboynixxik/team-10-queue-serivce/commit/73fd8eed747f9ead10e8a9334baebf0bf62e3623))
- Реализовал Redis-репозиторий как горячий путь с защитой от гонок через атомарные Lua-скрипты (`internal/repository/redis/cache.go`): ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/52902dfd30b5aa103fece319987cc832a4c17d85))
- `TryAllocate` / `CommitPurchase` / `InitStock` — атомарная работа с остатком товара
- `Enqueue` / `RemoveFromQueue` — FIFO-очередь на монотонном счётчике
- кэш членства и выданных прав для быстрой валидации перед оплатой
- `AddToExpiryTimer` / `RemoveFromExpiryTimer` — отслеживание дедлайнов офферов и прав
- `PublishEvent` — публикация изменений статуса для realtime-клиентов
- `GetQueueMetrics` — получение позиции и доступных единиц за один pipeline ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/93adedcba5ca4b5f9894237bcd26f2eb19bad7c9))
- Дополнил репозиторий методами для продвижения очереди. ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/944d12a0600c81f9b00173077042cc32a29faa9d), [Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/80c3fae617e98de715185c1c81e3fa8cd297c078))
- Покрыл интеграционными и race-condition тестами на `testcontainers`. ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/a29f660a7a9a5f70a62be69f6b3acfad57b5b2f3), [Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/4b1ebfab292bc5a025b0d365711202b4fdd3bded), [Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/a2f19ce0a200588b963e78e447288d03fadf41ff), [Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/8ab07bf70e23e7cdfb346cff031425d20bbd7376))

### Бизнес-логика очереди

- Реализовал ядро `QueueService` (`internal/service/queue_service.go`):
- `JoinQueue` и `processAllocation` — вход в очередь с идемпотентностью и немедленной выдачей права, если товар доступен ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/aa647c81d662c34a6db511bd2a318151459284c1))
- `AcceptOffer` / `DeclineOffer` — принятие и отклонение частичного оффера ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/f3fd462c013742ff8896ba2a83645867f2286b56))
- `AdvanceQueue` — продвижение очереди после освобождения остатка ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/3b403bc321b0ad8aa8ed97c27c918e59f2bc59fb))
- `ValidateRight` / `ProcessPayment` — проверка права перед checkout и завершение покупки ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/28547e0ed91b57482159015d6511f2fb18fd7dfd))
- `ProcessExpirations` — обработка истёкших офферов и прав: возврат остатка, смена статуса, продвижение очереди ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/abf543e4973448bf08633c895fadbeab54b0d3e4))
- Выделил интерфейс для фонового воркера экспираций (`internal/service/worker/interfaces.go`). ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/697420502b42932cc63d0e30888087cefab12346))
- Реализовал `CalculateETA` (`internal/service/membership.go`) — позиция пользователя в очереди и прогноз времени ожидания на основе метрик Redis и параметра `AvgPaymentTime` из конфигурации. ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/84853bf0d21341402b1bd9712c8d39387b32b6bc), [Tests](https://github.com/bigboynixxik/team-10-queue-serivce/commit/38c9d53277885dea1485108f579be39c5b37d0d1))
- Описал контракты репозиториев и сервисов (`internal/service/interfaces.go`, `internal/transport/interfaces.go`). ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/25c766e23f7d69d18ea8be9ee290ebeacc240551), [Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/27c91cb75ab8da62c5f92ea5fca8c393b9efcfdb), [Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/2d4d8ed4859ded83392684bfd1bbe81dcd3c7dd9))
- Покрыл бизнес-логику unit-тестами с `go.uber.org/mock` и `testify`-сьютами: `queue_service_test`, `join_queue_test.go`, `accept_offer_test.go`, `decline_offer_test.go`, `advance_queue_test.go`, `validate_right_test.go`, `process_payment_test.go`, `process_expirations_test.go`, `membership_test.go`. ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/86f312bc4589e440fed3455e8793c82abf76fb38), [Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/0ecbc9ec51e8dec006e702cf500346495aa33b4c), [Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/272c8ec2d07d1da1084fb63dc5fb92cff7428ec6), [Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/a32128b4b6eb32c951ef4b48945c145e2020e8ee))

### Остальное влияние на проект

- Добавил сбор метрик для продавца ([PR](https://github.com/bigboynixxik/team-10-queue-serivce/pull/25))

- Добавил логирование ошибок в общий хелпер ответов транспортного слоя (`internal/transport/api/response.go`). ([Commit](https://github.com/bigboynixxik/team-10-queue-serivce/commit/e745667ad0b6f6cf7b56338c839cdb51acb13f64))
- Вёл репозиторий как владелец: ревью и приём пул-реквестов команды в `develop`. ([PR #9](https://github.com/bigboynixxik/team-10-queue-serivce/pull/9), [PR #10](https://github.com/bigboynixxik/team-10-queue-serivce/pull/10) и остальные PR)
- Активное обсуждение архитектуры и дополнительного функционала.
