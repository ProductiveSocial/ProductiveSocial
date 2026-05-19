Конечно! Вот перевод этого документа на русский язык:

---

# ProductiveSocial — Полный дизайн системы

> В этом документе представлен полный дизайн системы ProductiveSocial: как то, что построено сейчас, так и окончательная целевая архитектура, когда платформа будет завершена. Разделы явно обозначены **[CURRENT]** (текущие) или **[PLANNED]** (планируемые), чтобы всегда было понятно, в каком состоянии находится система.

---

## Содержание

1. [Видение продукта](#1-видение-продукта)
2. [Общая архитектура](#2-общая-архитектура)
3. [Инвентарь сервисов](#3-инвентарь-сервисов)
4. [Полная карта взаимодействия сервисов](#4-полная-карта-взаимодействия-сервисов)
5. [Аутентификация и идентификация](#5-аутентификация-и-идентификация)
6. [Текущие сервисы — глубокий анализ](#6-текущие-сервисы--глубокий-анализ)
   - 6.1 psocial_selfmanager
   - 6.2 psocial_timer
   - 6.3 psocial_analytics
   - 6.4 psocial_billing
7. [Планируемые сервисы — дизайн](#7-планируемые-сервисы--дизайн)
   - 7.1 psocial_user
   - 7.2 psocial_journal
   - 7.3 psocial_social
   - 7.4 psocial_notes
8. [Мобильный клиент — PS_KMP](#8-мобильный-клиент--ps_kmp)
9. [Диаграммы потоков данных](#9-диаграммы-потоков-данных)
   - 9.1 Pipeline AI анализа
   - 9.2 Офлайн-синхронизация
   - 9.3 Поток социального взаимодействия
   - 9.4 AI-синтез журнала
10. [Схемы баз данных](#10-схемы-баз-данных)
11. [Обзор API](#11-обзор-api)
12. [Архитектура развертывания](#12-архитектура-развертывания)
13. [Дорожная карта ML и AI](#13-дорожная-карта-ml-и-ai)
14. [Модель безопасности](#14-модель-безопасности)

---

## 1. Видение продукта

ProductiveSocial — это единая платформа для личной продуктивности, которая устраняет фрагментацию существующего рынка инструментов для продуктивности. Сегодня пользователь, который хочет управлять задачами, формировать привычки, отслеживать фокус-сессии, вести дневник и получать AI-инсайты, вынужден использовать 4-5 отдельных приложений, каждое со своим аккаунтом, хранилищем данных и узким AI, которое может рассуждать только о своих данных.

Основное предложение ProductiveSocial — что все эти аспекты личной продуктивности должны находиться в одной, интегрированной платформе, где:

- Каждое действие (завершение задачи, отметка привычки, завершение фокус-сессии, запись в дневник) обогащает единый структурированный контекст данных
- AI-анализ имеет одновременный доступ ко всему этому контексту при генерации инсайтов
- Социальный слой обеспечивает ответственность и сообщество без фрагментации внимания, характерной для мейнстримовых соцсетей
- Платформа аккумулирует структурированные поведенческие данные, которые со временем будут использоваться для собственных предиктивных моделей

Архитектура системы — сегодня 4 сервиса, при полном реализовани — 8, — создана с нуля для поддержки этого полного видения. Каждое решение (секрет JWT, целочисленный userId, внутренние границы API, протокол офлайн-синхронизации) было принято с учетом будущей полной системы.

---

## 2. Общая архитектура

### Текущее состояние [CURRENT]

```
┌─────────────────────────────────────────────────────────────────────┐
│                          КЛИЕНТЫ                                    │
│                                                                     │
│   PS_KMP (Android / iOS / Desktop)    psocial_user_dashboard        │
│   [экран входа + каркас]                [Streamlit, порт 1231]      │
│                                                                     │
│                              psocial_dashboard                      │
│                              [Streamlit, порт 1230]                │
└──────────────┬──────────────────────────┬───────────────────────────┘
               │ JWT (Bearer)             │ JWT + X-Internal-Key
               ▼                          ▼
┌──────────────────────────────────────────────────────────────────────┐
│                       СЕРВИСЫ, ОБРАЩАЮЩИЕСЯ К ПОЛЬЗОВАТЕЛЮ             │
│                                                                     │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────────┐    │
│  │  selfmanager    │  │     timer       │  │    analytics     │    │
│  │  Kotlin/Ktor    │  │  Kotlin/Ktor    │  │  Kotlin/Ktor     │    │
│  │  порт 1226      │  │  порт 1227      │  │  порт 1228       │    │
│  │                 │  │                 │  │                  │    │
│  │ задачи, привычки│  │ Pomodoro-сессии │  │ AI-анализ, кредит│    │
│  │ рутины, проекты │  │ инсайты        │  │ прокси, админ    │    │
│  │ идентификация   │  │                 │  │                  │    │
│  └────────┬────────┘  └────────┬────────┘  └────────┬─────────┘    │
│           │                   │                     │              │
│     ┌─────┴──────┐      ┌─────┴──────┐       ┌─────┴──────┐       │
│     │selfmanager │      │  timer_db   │       │analytics_db│      │
│     │    _db     │      │ (PostgreSQL)│       │ (PostgreSQL)│     │
│     │ (PostgreSQL)│     └─────────────┘       └─────────────┘     │
│     └────────────┘                                                 │
└──────────────────────────────────┬───────────────────────────────────┘
                                   │ X-Internal-Key
                                   ▼
┌──────────────────────────────────────────────────────────────────────┐
│                          ВНУТРЕННИЙ СЕРВИС                            │
│                                                                     │
│          ┌─────────────────────────────────────┐                   │
│          │            billing                  │                   │
│          │         Python/FastAPI               │                   │
│          │           порт 1229                  │                   │
│          │                                     │                   │
│          │  кредитный учет  inference LLM     │                   │
│          │  реестр моделей sklearn             │                   │
│          └──────────────────┬──────────────────┘                   │
│                             │                                       │
│                    ┌────────┴────────┐                              │
│                    │   billing_db    │     ┌──────────────────┐   │
│                    │  (PostgreSQL)   │     │  Ollama / Claude │   │
│                    └─────────────────┘     │  / OpenAI / GPT  │   │
│                                            └──────────────────┘   │
└──────────────────────────────────────────────────────────────────────┘
```

### Планируемое состояние [PLANNED — Полное видение]

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              КЛИЕНТЫ                                    │
│                                                                         │
│    PS_KMP (Android / iOS / Desktop)      Веб-приложение (будущее)       │
│    [полный функционал]                                                 │
└───────────────────────────────┬─────────────────────────────────────────┘
                                │ JWT (Bearer)
                                ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        СЕРВИСЫ, ОБРАЩАЮЩИЕСЯ К ПОЛЬЗОВАТЕЛЮ             │
│                                                                         │
│  ┌─────────────┐ ┌──────────┐ ┌───────────┐ ┌──────────┐ ┌──────────┐  │
│  │selfmanager  │ │  timer   │ │analytics  │ │  user    │ │ journal  │  │
│  │Kotlin/Ktor  │ │Kotlin/Ktor││Ktor       │ │Kotlin/Ktor││Ktor      │  │
│  │             │ │          │ │           │ │          ││          │  │
│  │задачи, привычки│ │Pomodoro  │ │AI-анализ  │ │ профили ││записи     │  │
│  │рутины, заметки│ │сессии    │ │кредит-прокси││ соц.граф││   дневник │  │
│  │списки дел   │ │фокус, статистика│ │          ││          ││          │  │
│  │проекты      │ │          │ │           │ │          ││          │  │
│  │идентификация │ │          │ │           │ │          ││          │  │
│  └──────┬──────┘ └────┬─────┘ └─────┬─────┘ └────┬─────┘ └────┬─────┘  │
│         │             │             │             │            │        │
│     ┌───┴──┐      ┌───┴──┐     ┌───┴──┐     ┌───┴──┐    ┌───┴──┐     │
│     │ DB   │      │ DB   │     │ DB   │     │ DB   │    │ DB   │     │
│     └──────┘      └──────┘     └──────┘     └──────┘    └──────┘     │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                     социализация                                  │   │
│  │                  Kotlin/Ktor                                     │   │
│  │   лента достижений · достижения · шаблоны привычек             │   │
│  │   партнерство по ответственности · комнаты фокуса · профили  │   │
│  │                    ┌──────┐                                       │   │
│  │                    │  DB  │                                       │   │
│  │                    └──────┘                                       │   │
│  └──────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────┬───────────────────────────────┘
                                          │ X-Internal-Key
                                          ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                           ВНУТРЕННИЙ СЕРВИС                              │
│                                                                         │
│              ┌───────────────────────────────────────┐                 │
│              │               billing                 │                 │
│              │            Python/FastAPI             │                 │
│              │  кредитный учет · inference LLM     │                 │
│              │  реестр моделей · sklearn            │                 │
│              │  обученные модели поведения           │                 │
│              └───────────────┬───────────────────────┘                 │
│                              │                   ┌───────────────┐     │
│                        ┌─────┴──────┐            │ LLM Провайдеры │  │
│                        │ billing_db │            │ Ollama/Claude │  │
│                        └────────────┘            │ OpenAI / GPT  │  │
│                                                  └───────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Инвентарь сервисов

| Сервис | Язык/Фреймворк | Порт | Статус | Ответственность |
|---|---|---|---|---|
| `psocial_selfmanager` | Kotlin/Ktor | 1226 | **Работает** | Идентификация пользователя, задачи, привычки, рутины, проекты, теги, офлайн-синхронизация |
| `psocial_timer` | Kotlin/Ktor | 1227 | **Работает** | Pomodoro, интервалы, аналитика фокуса, атрибуция времени |
| `psocial_analytics` | Kotlin/Ktor | 1228 | **Работает** | AI-обработка, прокси кредитов, отчеты админов |
| `psocial_billing` | Python/FastAPI | 1229 | **Работает** | Кредитный учет, inference ML/LLM, реестр моделей |
| `psocial_dashboard` | Python/Streamlit | 1230 | **Работает** | Админский веб-интерфейс (краткосрочно, пока не построен нативный админ) |
| `psocial_user_dashboard` | Python/Streamlit | 1231 | **Работает** | Пользовательский веб-интерфейс (краткосрочно, пока не закончена версия KMP) |
| `PS_KMP` | Kotlin/KMP + Compose | — | **В разработке** | Основной мобильный/десктоп-клиент (Android, iOS, JVM Desktop) |
| `psocial_user` | Kotlin/Ktor | 1232 | **Планируется** | Профили пользователей, аватары, био, социальная сеть |
| `psocial_journal` | Kotlin/Ktor | 1233 | **Планируется** | Записи дневника, отслеживание настроения, AI-обработка ежедневных обзоров |
| `psocial_social` | Kotlin/Ktor | 1234 | **Планируется** | Достижения, сообщество, шаблоны привычек, комнаты фокуса |
| `psocial_notes` | Kotlin/Ktor | 1235 | **Планируется** | Заметки с связями с задачами, привычками, проектами |

---

## 4. Полная карта взаимодействия сервисов

### Текущие взаимодействия [CURRENT]

```
КЛИЕНТ (JWT Bearer)
    │
    ├──▶ selfmanager:1226   ──JDBC──▶  selfmanager_db.users  (UserRegistry)
    │         │                                ▲
    │         │  X-Internal-Key                │ JDBC (UserRegistry)
    │         ▼                                │
    │    [internal/time-log]          timer:1227
    │                                     │
    ├──▶ timer:1227          ─────────────┘
    │
    ├──▶ analytics:1228
    │         │
    │         ├── X-Internal-Key ──▶ selfmanager:1226  (задачи, привычки, рутины)
    │         ├── X-Internal-Key ──▶ timer:1227         (статистика Pomodoro)
    │         └── X-Internal-Key ──▶ billing:1229       (предсказания, баланс, транзакции)
    │
    └──▶ billing:1229  (только для админки, через X-Internal-Key)
              │
              └──▶ Ollama / Claude / GPT-4o
```

### Планируемое взаимодействие [PLANNED — Полное видение]

```
КЛИЕНТ (JWT Bearer)
    │
    ├──▶ selfmanager:1226      задачи, привычки, рутины, заметки, списки дел, синхронизация
    │         │
    │         ├── X-Internal-Key ──▶ [internal/time-log]  ◀── timer:1227
    │         └── JDBC ──────────▶ selfmanager_db.users   ◀── timer:1227 (UserRegistry)
    │
    ├──▶ timer:1227             фокус, сессии, интервалы, аналитика фокуса
    │
    ├──▶ analytics:1228         AI-анализ, кредит-прокси
    │         │
    │         ├── X-Internal-Key ──▶ selfmanager:1226      (задачи/привычки/рутины)
    │         ├── X-Internal-Key ──▶ timer:1227             (статистика фокуса)
    │         ├── X-Internal-Key ──▶ journal:1233           (записи дневника для синтеза)
    │         ├── X-Internal-Key ──▶ social:1234            (социальный контекст)
    │         └── X-Internal-Key ──▶ billing:1229           (инференс + кредиты)
    │
    ├──▶ user:1232              профили, социальная сеть, подписки
    │
    ├──▶ journal:1233           записи, настроения, AI-синтез
    │         │
    │         └── X-Internal-Key ──▶ analytics:1228        (запуск синтеза)
    │
    ├──▶ social:1234            достижения, лента, шаблоны привычек, комнаты
    │         │
    │         ├── X-Internal-Key ──▶ selfmanager:1226      (данные о привычках)
    │         ├── X-Internal-Key ──▶ timer:1227             (данные о фокусе)
    │         └── X-Internal-Key ──▶ user:1232              (данные профиля для ленты)
    │
    └──▶ notes:1235             заметки с связями к сущностям
              │
              └── X-Internal-Key ──▶ selfmanager:1226      (валидация сущностей)

billing:1229 (только внутри, клиентам не вызывается)
    │
    ├──▶ Ollama (локально / ngrok)
    ├──▶ API Anthropic Claude
    ├──▶ API OpenAI GPT-4o
    └──▶ sklearn .pkl модели (загружаются через joblib)
```

---

## 5. Аутентификация и идентификация

### Текущий поток аутентификации [CURRENT]

```
┌─────────────────────────────────────────────────────────────────────┐
│  Шаг 1 — Клиент отправляет email в selfmanager                         │
│                                                                     │
│  POST /api/v1/auth/identify  {"email": "user@example.com"}          │
│                                                                     │
│  selfmanager:                                                       │
│    INSERT INTO users (email) VALUES (?)                              │
│    ON CONFLICT (email) DO UPDATE SET email = EXCLUDED.email          │
│    RETURNING id                          ──▶  userId = 42           │
│                                                                     │
│  Возвращает: { accessToken, refreshToken, userId: 42, email }       │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  Шаг 2 — Структура токена (JWT HS256)                                │
│                                                                     │
│  Заголовок:  { "alg": "HS256", "typ": "JWT" }                         │
│  Полезная нагрузка: { "userId": 42, "email": "user@example.com",     │
│             "iat": 1716000000, "exp": 1716086400 }                  │
│  Подпись: HMAC-SHA256(заголовок + "." + полезная нагрузка, JWT_SECRET)│
│                                                                     │
│  TTL для токена доступа: 24 часа                                      │
│  TTL для токена обновления: 7 дней                                    │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  Шаг 3 — Принятие токенов между сервисами                            │
│                                                                     │
│  selfmanager ─── одинаковый JWT_SECRET ──▶ timer  ✓ принимает токен  │
│  selfmanager ─── одинаковый JWT_SECRET ──▶ analytics  ✓ принимает токен│
│                                                                     │
│  Один вход. Один токен. Все сервисы.                                │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  Шаг 4 — UserRegistry (каноничный целочисленный идентификатор)      │
│                                                                     │
│  Проблема: timer должен присваивать тот же userId, что и selfmanager │
│  Решение: timer подключается к таблице users selfmanager через JDBC   │
│                                                                     │
│  timer ──JDBC──▶ selfmanager_db.users                                │
│    INSERT INTO users (email) ON CONFLICT DO UPDATE RETURNING id     │
│                                                                     │
│  Результат: одинаковый userId = 42 независимо от того, какая служба  │
│  увидела пользователя первой. Единственное подключение к базе — межсистемное.│
└─────────────────────────────────────────────────────────────────────┘
```

### Расширенный поток аутентификации [PLANNED]

Когда появится `psocial_user`, модель идентификации расширится:

```
selfmanager       → владеет каноничной таблицей пользователей (целый userId, email)
psocial_user      → владеет богатым профилем (имя, аватар, био, таймзона)
                   читает userId из JWT, соединения с базой не требуется
Все остальные сервисы → используют тот же самый JWT (userId извлекается из claims)
                   вызывают psocial_user через X-Internal-Key для получения профиля

Целый userId остается универсальным ключом для всех сервисов.
psocial_user НЕ заменяет таблицу пользователей selfmanager —
она накладывает более богатые данные идентичности поверх того же целого ключа.
```

### Аутентификация сервисов друг с другом

```
┌────────────────────────────────────────────────────────┐
│  X-Internal-Key                                              │
│                                                              │
│  Отправитель добавляет заголовок:  X-Internal-Key: <общий секрет> │
│  Получатель проверяет:  если заголовок != INTERNAL_API_KEY  │
│                         вернуть 401 Unauthorized             │
│                                                              │
│  Ключ хранится ТОЛЬКО в переменных окружения.                │
│  Никогда в коде. Никогда в ответах API.                      │
│  Никогда в документации Swagger.                              │
│                                                              │
│  Все сервисы используют один и тот же INTERNAL_API_KEY.      │
└────────────────────────────────────────────────────────┘
```

---

## 6. Текущие сервисы — глубокий анализ

### 6.1 psocial_selfmanager [CURRENT]

**Ответственность:** Ядро идентификации и главный хранилищ данных продуктивности. Владеет таблицей `users`, к которой подключаются все остальные сервисы.

**Стэк технологий:** Kotlin 1.9+, Ktor 2.x, Exposed ORM, PostgreSQL, HikariCP, Koin DI

**Модель сущностей:**

```
users
  └─▶ projects
        ├─▶ tasks
        │     ├─▶ subtasks
        │     ├─▶ task_scheduled_times
        │     ├─▶ task_tags  ──▶  tags
        │     └─▶ completion_log
        │
        ├─▶ habits
        │     ├─▶ habit_subtasks
        │     ├─▶ habit_scheduled_times
        │     ├─▶ habit_reminder_times
        │     ├─▶ habit_completions
        │     │     └─▶ habit_subtask_completions
        │     └─▶ habit_tags  ──▶  tags
        │
        └─▶ routines
              ├─▶ routine_steps
              ├─▶ routine_scheduled_times
              ├─▶ routine_reminder_times
              ├─▶ routine_completions
              │     └─▶ routine_step_completions
              └─▶ routine_tags  ──▶  tags
```

**Ключевые решения:**
- `syncId` UUID у каждой сущности для офлайн-идемпотентной синхронизации
- `ON CONFLICT (sync_id) DO UPDATE` — безопасно при повторе
- Теги создаются автоматически при ссылке; уникальны для пользователя
- Внутренние эндпоинты возвращают минимальные представления для эффективности промптов LLM
- `time_spent_minutes` по задачам и привычкам обновляется атомарно через `POST /internal/time-log`

---

### 6.2 psocial_timer [CURRENT]

**Ответственность:** Жизненный цикл Pomodoro-сессий и аналитика фокуса.

**Стэк технологий:** Kotlin/Ktor, Exposed, PostgreSQL, HikariCP, Koin

**Модель состояния жизненного цикла:**

```
  ┌──────────┐
  │  ACTIVE  │◀──────────────────────────┐
  └────┬─────┘                          │
       │                                │
    pause()                          resume()
       │                                │
       ▼                                │
  ┌──────────┐                    ┌─────┴────┐
  │  PAUSED  │───────────────────▶│  ACTIVE  │
  └────┬─────┘                    └──────────┘
       │
   abandon()
       │
       ▼
  ┌───────────┐
  │ ABANDONED │
  └───────────┘

  Из ACTIVE:
    complete()  —─▶  COMPLETED  —─▶  отправка POST /internal/time-log (fire-and-forget)
    abandon()   —─▶  ABANDONED
```

**Последовательность интервалов:**

```
Создается сессия
    │
    ▼
 Рабочий интервал (по умолчанию 25 мин)
    │
    ▼ завершение
 completedWork % cyclesUntilLongBreak == 0?
    ├── ДА —─▶ Длинный перерыв (15 мин)
    └── НЕТ —▶ Короткий перерыв (5 мин)
    │
    ▼ завершение
 Рабочий интервал
    │
    └── цикл повторяется
```

**Адаптивные сигналы AI (возвращаются при событии):**

```
После завершения работы  →  suggestedNextBreak:
  всего минут работы сегодня >= 90 AND без длинного перерыва
    → ExtendedRest
  цикл достиг порога для длинного перерыва
    → LongBreak
  по умолчанию
    → ShortBreak

При паузе  →  abandonmentRisk:
  pauseCount >= личного порога (среднее число пауз при отказе, минимум 3)
    → Внимание: "Вы поставили на паузу N раз. Обычно так происходит в этот момент."
  возраст сессии > 3× запланированной длительности ИЛИ без завершенных циклов
    → Внимание: "Сессия проста. Рассмотрите возможность бросить."
```

---

### 6.3 psocial_analytics [CURRENT]

**Ответственность:** AI-процессинг и публичный шлюз к биллингу.

**Pipeline анализа:**

```
POST /api/v1/analytics  {type, modelId, customPrompt?}
    │
    ├── Обеспечить существование пользователя в таблице analytics_users
    │
    ├── Параллельные запросы (Kotlin корутины):
    │     async { GET /internal/users/{id}/tasks }   → selfmanager
    │     async { GET /internal/users/{id}/habits }  → selfmanager
    │     async { GET /internal/users/{id}/routines }→ selfmanager
    │     async { GET /internal/users/{id}/stats }    → timer
    │     awaitAll()
    │     (любая ошибка → пропуск данных, не сбой пайплайна)
    │
    ├── Формирование inputData:
    │     задачи, привычки, рутины, статистика Pomodoro + типовой промпт
    │
    ├── POST /api/v1/internal/predict → биллинг
    │     (таймаут 2 мин для запуска LLM)
    │     (ошибка — ошибка пользователю, без скрытого отката)
    │
    ├── Вытаскивание текста инсайта + списанных кредитов
    │
    ├── INSERT INTO analytics_reports (...)
    │
    └── Возврат AnalysisResult пользователю
```

**Типы анализа и стратегия промптов:**

| Тип | Фокус промпта | Общий вывод по данным |
|---|---|---|
| `PRODUCTIVITY_SUMMARY` | Обзор всех аспектов | Корреляция между временем фокуса, завершением привычек и выполнением задач |
| `TASK_PRIORITIZATION` | Задачи в очереди + вложенное время | Выявляет важные задачи, игнорируемые несмотря на фокус |
| `HABIT_INSIGHTS` | Стрик, рисковые привычки, изменение | Связь привычек и сессий фокусировки |
| `ROUTINE_OPTIMIZATION` | Эффективность шагов, пробелы | Выявляет самые "слабые" этапы рутины |
| `WEEKLY_TIMER_SUMMARY` | Время фокуса, завершение, пики | Лучшие часы фокусировки, самые продуктивные сущности |
| `CUSTOM` | Свободный запрос пользователя | Открытый кросс-доменный анализ |
| `DAILY_REFLECTION` *(планируется)* | Запись в дневник + субъективные ощущения | Связь настроения и объективных данных за день |

---

### 6.4 psocial_billing [CURRENT]

**Ответственность:** Внутренний учет кредитов и inference ML/LLM. Не вызывается клиентами напрямую.

**Модель баланса кредитов:**

```
Традиционный подход (ИЗБЕГАТЬ):
  колонка users.balance  — обновляется при каждой транзакции
  Риск: баланс и журнал транзакций могут расходиться при частичных сбоях

Подход ProductiveSocial:
  таблица транзакций с колонкой balance_after
  Текущий баланс — SELECT balance_after FROM transactions
                    WHERE selfmanager_user_id = ?
                    ORDER BY created_at DESC LIMIT 1

  Баланс — это журнал транзакций. Он не может расходиться.
```

**Жизненный цикл предсказания:**

```
POST /api/v1/internal/predict
    │
    ├── Проверка модели (активна, есть)
    ├── ensure_welcome_credits(userId)
    │     ЕСЛИ транзакций нет — вставить депозит 100 кредитов
    ├── Проверка баланса >= цена модели
    │     ЕСЛИ недостаточно — 402 PaymentRequired
    ├── Вставка предсказания (статус=ПОДГОТОВЛЕНО)
    │
    ├── Отправка провайдеру:
    │     OLLAMA     → POST /api/chat
    │     ANTHROPIC  → anthropic SDK
    │     OPENAI     → openai SDK
    │     SKLEARN    → joblib.load(model_path).predict_proba()
    │
    ├── В случае успеха:
    │     Вставка транзакций (тип=СПИСАНИЕ, сумма=-цена, balance_after=новый баланс)
    │     Обновление предсказания (статус=УСПЕХ, результат)
    │     Возврат { результат, списанные кредиты }
    │
    └── В случае ошибки:
          Обновление предсказания (статус=НЕУДАЧА, сообщение об ошибке)
          транзакции не создаются — баланс не меняется
          ошибка передается вызывающему
```

---

## 7. Планируемые сервисы — дизайн

### 7.1 psocial_user [ПЛАНИРУЕТСЯ]

**Ответственность:** Расширенные профили пользователей и социальная сеть (подписки/подписчики).

**Почему отдельно от selfmanager?** selfmanager владеет только минимальными данными для идентификации (email + ID). Расширенный профиль с отображаемым именем, аватаром, био, соц.графом — это другая область. Также профиль читается несколькими сервисами (соц., аналитика, дневник), и единая модель предотвращает дублирование и несогласованность.

**Стэк:** Kotlin/Ktor, PostgreSQL, HikariCP, Koin

**Схема БД:**

```sql
-- Расширенный профиль поверх selfmanager
CREATE TABLE user_profiles (
    id                  SERIAL PRIMARY KEY,
    selfmanager_user_id INTEGER NOT NULL UNIQUE,  -- FK на selfmanager.users.id
    display_name        VARCHAR(100),
    bio                 TEXT,
    avatar_url          TEXT,
    timezone            VARCHAR(50) DEFAULT 'UTC',
    is_public           BOOLEAN NOT NULL DEFAULT TRUE,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- соц.граф
CREATE TABLE follows (
    follower_id INTEGER NOT NULL,   -- selfmanager_user_id
    following_id INTEGER NOT NULL,  -- selfmanager_user_id
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    PRIMARY KEY (follower_id, following_id)
);

-- Настройки уведомлений
CREATE TABLE notification_preferences (
    user_id                 INTEGER NOT NULL UNIQUE,
    habit_reminders         BOOLEAN NOT NULL DEFAULT TRUE,
    routine_reminders       BOOLEAN NOT NULL DEFAULT TRUE,
    social_achievements     BOOLEAN NOT NULL DEFAULT TRUE,
    weekly_summary          BOOLEAN NOT NULL DEFAULT TRUE,
    accountability_updates  BOOLEAN NOT NULL DEFAULT TRUE
);
```

**API:**

```
GET    /api/v1/users/me                  JWT
PATCH  /api/v1/users/me                  JWT
GET    /api/v1/users/{id}                JWT
POST   /api/v1/users/{id}/follow         JWT
DELETE /api/v1/users/{id}/follow         JWT
GET    /api/v1/users/me/followers        JWT
GET    /api/v1/users/me/following        JWT
GET    /internal/users/{id}/profile      X-Internal-Key
```

---

### 7.2 psocial_journal [ПЛАНИРУЕТСЯ]

**Ответственность:** Ежедневные записи, настроение, AI-синтез, связывающий субъективные ощущения и объективные данные.

**Главная ценность:** закрывает цикл между объективной активностью (что сделано) и субъективным ощущением (как было). Ни одно существующее приложение не делает этого автоматом, потому что не имеет доступа к обеим сторонам.

**Стэк:** Kotlin/Ktor, PostgreSQL, HikariCP, Koin

**Схема БД:**

```sql
CREATE TABLE journal_entries (
    id                  SERIAL PRIMARY KEY,
    sync_id             UUID NOT NULL UNIQUE,
    user_id             INTEGER NOT NULL,
    entry_date          DATE NOT NULL,
    written_at          TIMESTAMPTZ DEFAULT NOW(),
    mood                INTEGER,  -- 1–5
    mood_label          VARCHAR(50),
    content             TEXT NOT NULL,
    tasks_completed     JSONB,
    habits_completed    JSONB,
    focus_minutes       INTEGER,
    synthesis_text      TEXT,
    synthesis_model     VARCHAR(100),
    synthesis_at        TIMESTAMPTZ,
    created_at          TIMESTAMPTZ DEFAULT NOW(),
    updated_at          TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE mood_log (
    id          SERIAL PRIMARY KEY,
    user_id     INTEGER NOT NULL,
    logged_at   TIMESTAMPTZ DEFAULT NOW(),
    mood        INTEGER NOT NULL,
    context     VARCHAR(100)
);
```

**API:**

```
POST   /api/v1/journal/entries
GET    /api/v1/journal/entries
GET    /api/v1/journal/entries/{date}
PATCH  /api/v1/journal/entries/{id}
POST   /api/v1/journal/entries/{id}/synthesise
GET    /api/v1/journal/mood/history
POST   /api/v1/journal/mood
GET    /internal/users/{id}/entries
```

**AI синтез:**

```
Пользователь пишет запись
    │
    ▼
Сервис дневника собирает:
    ├── Завершенные задачи сегодня
    ├── Завершенные привычки
    ├── Время фокуса
    │
    ▼
Пользователь или автоматический запуск вызывает синтез:
    analytics → DAILY_REFLECTION
    inputData = {
        content, mood, задачи, привычки, время фокуса
    }
    промпт: "Вы — рефлексивный коуч. На основе дневника, настроения и объективных данных за сегодня напишите тёплый, короткий и проницательный обзор. Определите одну закономерность и одну конкретную цель на завтра."
    │
    ▼
LLM генерирует текст
    │
    ▼
Сервис дневника сохраняет синтез в записи
```

---

### 7.3 psocial_social [ПЛАНИРУЕТСЯ]

**Ответственность:** Сообщество и ответственность. Создано для мотивации без внедрения внимания, характерного для соцсетей.

**Дизайн:**
1. Нет алгоритмической ленты — только хронология
2. Деление на достижения и опциональное участие — без пассивных обновлений статуса
3. Социальные функции — в отдельной вкладке
4. Низкая частота уведомлений

**Стэк:** Kotlin/Ktor, PostgreSQL, HikariCP, Koin

**Схема БД:**

```sql
-- Достижения
CREATE TABLE achievements (
    id              SERIAL PRIMARY KEY,
    user_id         INTEGER NOT NULL,
    type            VARCHAR(50) NOT NULL,
    label           VARCHAR(100) NOT NULL,
    earned_at       TIMESTAMPTZ DEFAULT NOW(),
    is_public       BOOLEAN DEFAULT FALSE,
    shared_at       TIMESTAMPTZ
);

-- Посты в ленте (достижения + вручную)
CREATE TABLE feed_posts (
    id              SERIAL PRIMARY KEY,
    user_id         INTEGER NOT NULL,
    post_type       VARCHAR(30) NOT NULL,
    content         TEXT,
    reference_id    INTEGER,
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

-- Шаблоны привычек
CREATE TABLE habit_blueprints (
    id              SERIAL PRIMARY KEY,
    author_user_id  INTEGER NOT NULL,
    name            VARCHAR(255),
    description     TEXT,
    habit_type      VARCHAR(10),
    recurrency      VARCHAR(20),
    suggested_time  TIME,
    tags            TEXT[],
    adopted_count   INTEGER DEFAULT 0,
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

-- Ответственность
CREATE TABLE accountability_partners (
    user_id         INTEGER NOT NULL,
    partner_id      INTEGER NOT NULL,
    habit_name      VARCHAR(255),
    goal_text       TEXT,
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    PRIMARY KEY (user_id, partner_id)
);

-- Комнаты фокуса
CREATE TABLE focus_rooms (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(100),
    host_user_id    INTEGER NOT NULL,
    is_active       BOOLEAN DEFAULT TRUE,
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE focus_room_participants (
    room_id         UUID REFERENCES focus_rooms(id),
    user_id         INTEGER,
    status          VARCHAR(20) DEFAULT 'IDLE',
    joined_at       TIMESTAMPTZ DEFAULT NOW(),
    PRIMARY KEY (room_id, user_id)
);
```

**API:**

```
GET    /api/v1/social/feed
GET    /api/v1/social/achievements
POST   /api/v1/social/achievements/{id}/share
GET    /api/v1/social/blueprints
POST   /api/v1/social/blueprints
POST   /api/v1/social/blueprints/{id}/adopt
GET    /api/v1/social/accountability
POST   /api/v1/social/accountability
GET    /api/v1/social/rooms
POST   /api/v1/social/rooms
POST   /api/v1/social/rooms/{id}/join
PATCH  /api/v1/social/rooms/{id}/status
```

**Правила триггера достижений:**

| Достижение | Условие триггера |
|---|---|
| `HABIT_STREAK_7` | Любая привычка завершена 7 дней подряд |
| `HABIT_STREAK_30` | Любая привычка завершена 30 дней подряд |
| `HABIT_STREAK_100` | Любая привычка завершена 100 дней подряд |
| `FOCUS_10H` | Накоплено 10 часов фокусировки Pomodoro |
| `FOCUS_100H` | Накоплено 100 часов |
| `TASKS_50` | Выполнено 50 задач |
| `ROUTINE_WEEK` | Одна рутина выполнена каждый день 7 дней подряд |
| `BLUEPRINT_PUBLISHED` | Опубликован первый шаблон привычки |
| `BLUEPRINT_ADOPTED_10` | Шаблон принят 10 другими пользователями |

---

### 7.4 psocial_notes [ПЛАНИРУЕТСЯ]

**Ответственность:** Свободный текст с связями к сущностям (задачи, привычки, рутины, проекты).

**Ключевое отличие:** заметки привязаны к сущностям — например, встреча к задаче "Планировать Q3". Не только по полнотекстовому поиску, а через связи.

**Схема БД:**

```sql
CREATE TABLE notes (
    id              SERIAL PRIMARY KEY,
    sync_id         UUID NOT NULL UNIQUE,
    user_id         INTEGER NOT NULL,
    title           VARCHAR(500),
    content         TEXT NOT NULL,
    -- Связь с сущностью (опционально)
    entity_type     VARCHAR(20),
    entity_id       INTEGER,
    tags            TEXT[],
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    updated_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_notes_entity ON notes (user_id, entity_type, entity_id);
```

**API:**

```
GET    /api/v1/notes
POST   /api/v1/notes
GET    /api/v1/notes/{id}
PATCH  /api/v1/notes/{id}
DELETE /api/v1/notes/{id}
GET    /api/v1/notes?entityType=TASK&entityId=15
POST   /api/v1/sync
```

---

## 8. Мобильный клиент — PS_KMP

### Общая архитектура

```
┌───────────────────────────────────────────────┐
│                   PS_KMP                      │
│                                               │
│  ┌────────────────────────────────────────┐   │
│  │             UI (Compose Multiplatform) │   │
│  │   ViewModels, State Management           │   │
│  │   Screen: SelfManager, Pomodoro, Journal │   │
│  └─────────────┬──────────────────────────┘   │
│                │                                  │
│  ┌─────────────▼─────────────────────────┐       │
│  │           Data Layer                │       │
│  │ Repositories (Task, Habit, Routine,│       │
│  │ Pomodoro, Journal, Notes, Social) │       │
│  │   Local (SQLDelight)  Remote (Ktor)│       │
│  └─────────────────────────────────────┘       │
│ Platforms:                                     │
│  Android → SQLDelight + OkHttp                  │
│  iOS → Native SQLDelight + Darwin engine       │
│  Desktop → JDBC + Java engine                   │
└───────────────────────────────────────────────┘
```

### Навигационная структура [Текущая + Планируемая]

```
NavHost
 ├── /login
 ├── /self-manager
 │    ├── /projects
 │    │    └── /projects/{id}
 │    ├── /tasks/create
 │    ├── /tasks/{id}
 │    ├── /habits/create
 │    ├── /habits/{id}
 │    ├── /routines/create
 │    └── /routines/{id}/run
 ├── /pomodoro
 │    ├── /pomodoro/session/{id}
 │    ├── /pomodoro/focus/{entityType}/{entityId}
 │    ├── /pomodoro/settings
 │    ├── /pomodoro/statistics
 │    └── /pomodoro/picker
 ├── /journal
 │    ├── /journal/entry/{date}
 │    └── /journal/mood
 ├── /notes
 │    └── /notes/{id}
 ├── /social
 │    ├── /social/achievements
 │    ├── /social/blueprints
 │    └── /social/rooms
 └── /user
       ├── /profile
       ├── /insights
       └── /credits
```

### Оффлайн-протокол синхронизации

```
Устройство                                  Сервер (selfmanager)
  │                                                  │
  │  Все записи сначала пишутся в SQLDelight       │
  │  isDirty=1, serverId=NULL                        │
  │                                                  │
  │──── POST /api/v1/sync ──────────────────────────▶│
  │     {                                            │
  │       create: [сущности с syncId]                │
  │       update: [измененные сущности]              │
  │       delete: [syncId для удаления]               │
  │       lastSyncedAt: "2026-05-18T..."             │
  │     }                                            │
  │◀─── Ответ синхронизации ─────────────────────────│
  │     {                                            │
  │       idMappings: {syncId → serverId}            │
  │       serverChanges: [новые/обновленные]        │
  │       errors: [ошибки]                            │
  │       syncedAt: "2026-05-18T..."                  │
  │     }                                            │
  │                                                  │
  │  Обновление локальной базы:                        │
  │    serverId в колонках                            │
  │    isDirty=0                                    │
  │    удаление строк isDeleted                     │
  │    вставка/обновление serverChanges              │
  │    сохранение lastSyncedAt                      │
```

---

## 9. Диаграммы потоков данных

### 9.1 Pipeline AI анализа (Текущий)

```
Пользователь / Клиент
     │
     │  POST /api/v1/analytics {type, modelId}  (JWT)
     ▼
psocial_analytics
     │
     ├── [async] GET /internal/users/{id}/tasks
     ├── [async] GET /internal/users/{id}/habits
     ├── [async] GET /internal/users/{id}/routines
     └── [async] GET /internal/users/{id}/stats
          │
          │  awaitAll() — задержка = max(tasks, habits, routines, stats)
          ▼
     Формирование inputData:
     { tasks: [...], habits: [...], routines: [...],
       pomodoro: {...}, prompt: "..." }
          │
          │  POST /api/v1/internal/predict  (X-Internal-Key)
          ▼
psocial_billing
     │
     ├── Проверка модели (активна, есть)
     ├── ensure_welcome_credits() — первый депозит, если нужно
     ├── Проверка баланса >= цена
     ├── Вставка предсказаний (статус=ПОДГОТОВЛЕНО)
     │
     └── Вызов LLM:
           OLLAMA    → POST /api/chat
           ANTHROPIC → anthropic SDK
           OPENAI    → openai SDK
               │
               ▼
          Текст инсайта
               │
     ├── Вставка транзакций (СПИСАНИЕ)
     └── Обновление предсказаний (УСПЕХ)
          │
          ▼
     Возврат {результат, списанные кредиты}
          │
          ▼
psocial_analytics
     │
     ├── Вставка отчётов
     └── Возврат AnalysisResult клиенту
```

### 9.2 AI Pipeline (Будущее — с дневником)

```
Пользователь / Клиент
     │
     │  POST /api/v1/analytics {type: DAILY_REFLECTION, modelId}  (JWT)
     ▼
psocial_analytics
     │
     ├── [async] GET /internal/users/{id}/tasks
     ├── [async] GET /internal/users/{id}/habits
     ├── [async] GET /internal/users/{id}/stats
     └── [async] GET /internal/users/{id}/entries   [ПЛАНИРУЕТСЯ]
          │
          ▼
Формирование inputData:
{ tasks_completed: [...], habits_completed: [...],
  focus_minutes: N, journal_entry: "...", mood: 4 }
 промпт: "Соедини субъективные ощущения с объективными данными..."
          │
          ▼
billing → LLM → синтез текста
          │
          ▼
psocial_analytics → возвращает синтез
          │
          ▼
psocial_journal → сохраняет синтез в запись [ПЛАНИРУЕТСЯ]
```

### 9.3 Поток социального взаимодействия [ПЛАНИРУЕТСЯ]

```
Пользователь завершает 30-дневный стрик привычки
     │
     ▼
selfmanager срабатывает (или аналитика обнаружит при следующем анализе)
     │
     ▼
psocial_social
  Вставка достижения (тип='HABIT_STREAK_30', user_id=42)
     │
     ▼
Пользователь открывает достижения
  Видит новый бейдж: "30-дневный стрик: утренняя зарядка"
  Нажимает "Поделиться в сообщество"
     │
     ▼
psocial_social
  Вставка feed_posts (тип='ACHIEVEMENT', reference_id=achievement.id)
     │
     ▼
Подписчики видят пост
  GET /api/v1/social/feed
  → [{user: {displayName, avatar}, achievement: {...}}]
```

### 9.4 Оффлайн-синхронизация с конфликтами

```
Сценарий: пользователь редактирует задачу на телефоне (офлайн), затем на вебе

Телефон (офлайн):
  UPDATE tasks SET title='Новое название A', isDirty=1 WHERE syncId='abc-123'

Веб (онлайн):
  PATCH /api/v1/tasks/task/{id}  {title: 'Новое название B'}
  → обновляется название на 'Новое название B', updatedAt=T2

Телефон подключается и синхронизируется:
  POST /api/v1/sync
  { updates: [{syncId: 'abc-123', title: 'Новое название A'}] }

Сервер применяет конфликт-решение:
  Серверное updatedAt (T2) > клиентского (T1)
  → Побеждает сервер: название 'Новое название B'
  → serverChanges включает задачу с названием 'Новое название B'

Телефон получает ответ:
  serverChanges: [{syncId: 'abc-123', title: 'Новое название B'}]
  → локальное название перезаписывается на 'Новое название B'
  → isDirty=0
```

---

## 10. Схемы баз данных

### selfmanager_db — Основные таблицы

```sql
-- Каноническая таблица пользователей (общая с timer)
users (id SERIAL PK, email VARCHAR UNIQUE, created_at TIMESTAMPTZ)

-- Проекты
projects (id, sync_id UUID UNIQUE, user_id → users, name, color, icon,
          priority, created_at, updated_at)

-- Задачи
tasks (id, sync_id UUID UNIQUE, user_id, project_id → projects,
       title, description, priority, urgency, completed BOOL,
       time_spent_minutes INT DEFAULT 0,
       due_date, is_recurring, target,
       created_at, updated_at)

subtasks (id, sync_id UUID UNIQUE, task_id → tasks,
          name, completed BOOL, position INT, created_at)

task_scheduled_times (id, task_id, scheduled_time TIME, created_at)

-- Привычки
habits (id, sync_id UUID UNIQUE, user_id, project_id,
        name, description, habit_type VARCHAR(10),
        recurrency, time_spent_minutes INT DEFAULT 0,
        created_at, updated_at)

habit_completions (id, sync_id UUID UNIQUE, habit_id, user_id,
                   completed_at DATE,
                   created_at TIMESTAMPTZ)

habit_subtask_completions (id, habit_completion_id, subtask_id,
                           completed BOOL, created_at)

-- Рутина
routines (id, sync_id UUID UNIQUE, user_id, project_id,
          name, recurrency, created_at, updated_at)

routine_steps (id, sync_id UUID UNIQUE, routine_id,
               name, duration_minutes INT, position INT,
               auto_start BOOL DEFAULT FALSE)

routine_completions (id, sync_id UUID UNIQUE, routine_id, user_id,
                     completed_at TIMESTAMPTZ)

routine_step_completions (id, routine_completion_id, step_id,
                          completed BOOL, time_taken_minutes INT)

-- Теги
tags (id, user_id, name VARCHAR(100), UNIQUE(user_id, name))
task_tags (task_id, tag_id)
habit_tags (habit_id, tag_id)
routine_tags (routine_id, tag_id)
```

### timer_db — Основные таблицы

```sql
pomodoro_settings (
    id, user_id UNIQUE,
    work_duration_minutes INT DEFAULT 25,
    short_break_minutes   INT DEFAULT 5,
    long_break_minutes    INT DEFAULT 15,
    cycles_until_long_break INT DEFAULT 4,
    auto_start_breaks BOOL, auto_start_sessions BOOL,
    sound BOOL, notifications BOOL,
    updated_at TIMESTAMPTZ
)

pomodoro_sessions (
    id, user_id,
    entity_type VARCHAR(20),
    entity_id   INTEGER,
    status      VARCHAR(20),
    total_work_minutes INT DEFAULT 0,
    completed_cycles   INT DEFAULT 0,
    pause_count        INT DEFAULT 0,
    work_duration_snapshot  INT,
    short_break_snapshot    INT,
    long_break_snapshot     INT,
    cycles_snapshot         INT,
    started_at  TIMESTAMPTZ,
    completed_at TIMESTAMPTZ
)

pomodoro_intervals (
    id, session_id → pomodoro_sessions,
    interval_type   VARCHAR(15),
    status          VARCHAR(15),
    planned_minutes INT,
    actual_minutes  INT,
    started_at TIMESTAMPTZ,
    ended_at   TIMESTAMPTZ
)
```

### billing_db — Основные таблицы

```sql
ml_models (
    id UUID PK DEFAULT gen_random_uuid(),
    name, provider VARCHAR(50),
    model_name, cost_per_use INT DEFAULT 5,
    system_prompt TEXT,
    is_active BOOL DEFAULT TRUE,
    created_at TIMESTAMPTZ
)

transactions (
    id UUID PK,
    selfmanager_user_id INT NOT NULL,
    transaction_type VARCHAR(20),
    amount INT NOT NULL,
    description TEXT,
    prediction_id UUID,
    balance_after INT NOT NULL,
    created_at TIMESTAMPTZ
)

predictions (
    id UUID PK,
    model_id UUID,
    selfmanager_user_id INT,
    status VARCHAR(20),
    input_data JSONB,
    output_data JSONB,
    error_message TEXT,
    credits_charged INT,
    created_at TIMESTAMPTZ,
    completed_at TIMESTAMPTZ
)
```

### analytics_db

```sql
analytics_users (
    id, selfmanager_id INT UNIQUE, email, is_admin BOOL
)

analytics_reports (
    id, user_id → analytics_users,
    analysis_type VARCHAR(50),
    model_id VARCHAR(100),
    insight_text TEXT,
    credits_charged INT,
    created_at TIMESTAMPTZ
)
```

---

## 11. Обзор API

### Аутентификация (selfmanager)

```
POST /api/v1/auth/identify
POST /api/v1/auth/refresh
POST /api/v1/auth/logout
POST /api/v1/auth/logout-all
```

### Данные (требует JWT)

```
POST /api/v1/sync
GET/POST /api/v1/projects/...
GET/POST /api/v1/tasks/...
GET/POST /api/v1/habits/...
GET/POST /api/v1/routines/...
GET /internal/users/{id}/tasks
GET /internal/users/{id}/habits
GET /internal/users/{id}/routines
POST /internal/time-log
```

### timer

```
GET/PUT /api/v1/pomodoro/settings
GET/POST /api/v1/pomodoro/sessions
POST /api/v1/pomodoro/sessions/{id}/start-interval
POST /api/v1/pomodoro/sessions/{id}/complete-interval
PATCH /api/v1/pomodoro/sessions/{id}/pause|resume|complete
DELETE /api/v1/pomodoro/sessions/{id}/abandon
POST /api/v1/pomodoro/sync
GET /api/v1/pomodoro/insights/focus-patterns
GET /api/v1/pomodoro/insights/entity-stats
```

### analytics

```
POST /api/v1/analytics
GET /api/v1/analytics
GET /api/v1/credits/balance
GET /api/v1/credits/transactions
```

### Админка (требует JWT + ADMIN_EMAILS)

```
GET /api/v1/admin/stats
GET /api/v1/admin/users
GET /api/v1/admin/reports
GET/DELETE /api/v1/admin/users/{id}/reports
POST /api/v1/admin/seed-reports
PATCH /api/v1/admin/users/{id}/toggle-admin
```

### Внутренние (X-Internal-Key)

```
GET /internal/users/{id}/stats
GET /api/v1/models
POST /api/v1/internal/models
POST /api/v1/internal/predict
GET /api/v1/internal/users/{id}/balance
GET /api/v1/internal/users/{id}/transactions
POST /api/v1/internal/users/{id}/deposit
```

---

## 12. Архитектура развертывания

### Локальная разработка

```
Colima (macOS)
    │
    └── Docker Compose
          │
          ├── selfmanager (порт 1226) — готовая JAR
          ├── selfmanager_db (порт 5432)
          │
          ├── timer (порт 1227) — готовая JAR
          ├── timer_db (порт 5434)
          │
          ├── analytics (порт 1228) — готовая JAR
          ├── analytics_db (порт 5435)
          │
          ├── billing (порт 1229) — Python, установка зависимостей
          ├── billing_db (порт 5433)
          │
          ├── dashboard (порт 1230) — Streamlit
          └── user_dashboard (порт 1231) — Streamlit
```

Все контейнеры на мосту psocial_network.
Обмен между контейнерами по имени сервиса.

Локальный Ollama: порт 11434, доступен из billing через host.docker.internal.

---

### Продакшн (Текущий)

```
Render.com (бесплатный)
    │
    ├── selfmanager
    ├── timer
    ├── analytics
    ├── billing
    ├── dashboard
    └── user_dashboard

Ollama — через ngrok (локально у разработчика)
```

### Продакшн (Целевое)

```
Render.com (платный, всегда активные)
    │
    ├── Все 6 сервисов (без холодных стартов)
    ├── psocial_user
    ├── psocial_journal
    ├── psocial_social
    └── psocial_notes

Базы данных: Neon PostgreSQL
LLM Провайдеры:
    ├── Anthropic Claude
    └── OpenAI GPT-4o

CDN: для статичных файлов, аватаров (Cloudflare или S3)

Push-уведомления: Firebase, APNs
```

---

## 13. Дорожная карта ML и AI

### Фаза 1 — Prompting LLM (Текущий)

Все анализы используют retrieval-augmented generation:
- данные подгружаются в реальном времени
- промпт формируется по типу
- не требуется обучение, метки или датасеты
- масштабируется сразу для любого пользователя
- качество зависит от промпта, а не от данных обучения

### Фаза 2 — Гибрид LLM + обученные модели (Планируется)

При накоплении данных:
- узкоспециализированные модели
- обучение оффлайн
- загрузка через POST /api/v1/models/{id}/upload
- использование в инференсе через `model.predict_proba()`
- добавление в селектор моделей

### Фаза 3 — Inference на устройстве (Будущее)

Для полностью офлайн AI:
- легкие модели (ONNX / CoreML)
- интеграция в KMP
- inference на устройстве
- отправка результатов в сервер для аналитики

---

## 14. Модель безопасности

### Границы угроз

```
НЕДОВЕРЕННЫЙ ──────────────
  Мобильный клиент / браузер / HTTP клиент
      │
      │ Только JWT Bearer
      │ Проверка подписи и срока
      ▼
УСЛУГИ, ОБРАЩАЮЩИЕСЯ К ПОЛЬЗОВАТЕЛЮ
      │
      │ X-Internal-Key для внутренних эндпоинтов
      │ Хранится только в env
      ▼
ВНУТРЕННИЕ СЕРВИСЫ (биллинг)
      │
      │ Не используют JWT
      │ Все требуют X-Internal-Key
      │ URL биллинга — скрыт
      │ Не документируется публично
```

### Инвентарь секретов

| Секрет | Хранится в | Кто знает | Ротация |
|---|---|---|---|
| `JWT_SECRET` | Env (selfmanager, timer, analytics) | 3 сервиса | Все требуют деплой одновременно |
| `INTERNAL_API_KEY` | Env на всех | Все сервисы | Все требуют деплой |
| `DATABASE_URL` | Env у каждого сервиса | 1 сервис | Отдельный деплой |
| `ANTHROPIC_API_KEY` | Env у billing | billing | billing деплой |
| `OPENAI_API_KEY` | Env у billing | billing | billing деплой |

### Известные уязвимости (до продакшна)

1. Аутентификация по email — без верификации, риск аккаунт-энумерации
2. Общий JWT секрет — при компромате все сервисы под ударом; лучше RS256
3. Нет лимитирования запросов к `/auth/identify`
4. Туннель ngrok для Ollama — не подходит для многопользовательского продакшна
5. Нет аудита — только лог транзакций, вызовы сервисов — без логирования
