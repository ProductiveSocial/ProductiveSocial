# ProductiveSocial — Complete System Design

> This document covers the full system design of ProductiveSocial: both what is built today and the complete target architecture when the platform is finished. Sections are explicitly labelled **[CURRENT]** or **[PLANNED]** so the current and future states are always distinguishable.

---

## Table of Contents

1. [Product Vision](#1-product-vision)
2. [High-Level Architecture](#2-high-level-architecture)
3. [Service Inventory](#3-service-inventory)
4. [Complete Service Interaction Map](#4-complete-service-interaction-map)
5. [Authentication and Identity](#5-authentication-and-identity)
6. [Current Services — Deep Dive](#6-current-services--deep-dive)
   - 6.1 psocial_selfmanager
   - 6.2 psocial_timer
   - 6.3 psocial_analytics
   - 6.4 psocial_billing
7. [Planned Services — Design](#7-planned-services--design)
   - 7.1 psocial_user
   - 7.2 psocial_journal
   - 7.3 psocial_social
   - 7.4 psocial_notes
8. [Mobile Client — PS_KMP](#8-mobile-client--ps_kmp)
9. [Data Flow Diagrams](#9-data-flow-diagrams)
   - 9.1 AI Analysis Pipeline
   - 9.2 Offline Sync
   - 9.3 Social Engagement Flow
   - 9.4 Journal AI Synthesis
10. [Database Schemas](#10-database-schemas)
11. [API Surface Summary](#11-api-surface-summary)
12. [Deployment Architecture](#12-deployment-architecture)
13. [ML and AI Roadmap](#13-ml-and-ai-roadmap)
14. [Security Model](#14-security-model)

---

## 1. Product Vision

ProductiveSocial is a unified personal productivity platform that eliminates the fragmentation of the current productivity tool market. Today, a user who wants to manage tasks, build habits, track focus sessions, journal their day, and get AI-powered insights must maintain four to five separate applications, each with its own account, its own data store, and its own narrow AI that can only reason about its own data.

ProductiveSocial's core proposition is that all of these dimensions of personal productivity belong in a single, integrated platform where:

- Every productivity action (completing a task, checking off a habit, finishing a focus session, writing a journal entry) enriches a single unified data context
- An AI analysis layer has simultaneous access to all of that context when generating insights
- A social layer enables accountability and community without introducing the attention fragmentation of mainstream social media
- The platform accumulates structured behavioral data that will power proprietary predictive models over time

The architecture of the system — four services today, eight when complete — is designed from the beginning to support this full vision. Every design decision in the current phase (the shared JWT secret, the integer user ID, the internal API boundaries, the offline-first sync protocol) was made with the complete system in mind.

---

## 2. High-Level Architecture

### Current State [CURRENT]

```
┌─────────────────────────────────────────────────────────────────────┐
│                          CLIENTS                                     │
│                                                                     │
│   PS_KMP (Android / iOS / Desktop)    psocial_user_dashboard        │
│   [auth screen + scaffolded]          [Streamlit, port 1231]        │
│                                                                     │
│                              psocial_dashboard                      │
│                              [Streamlit admin, port 1230]           │
└──────────────┬──────────────────────────┬───────────────────────────┘
               │ JWT (Bearer)             │ JWT + X-Internal-Key
               ▼                          ▼
┌──────────────────────────────────────────────────────────────────────┐
│                       USER-FACING SERVICES                          │
│                                                                     │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────────┐    │
│  │  selfmanager    │  │     timer       │  │    analytics     │    │
│  │  Kotlin/Ktor    │  │  Kotlin/Ktor    │  │  Kotlin/Ktor     │    │
│  │  port 1226      │  │  port 1227      │  │  port 1228       │    │
│  │                 │  │                 │  │                  │    │
│  │ tasks habits    │  │ pomodoro        │  │ AI analysis      │    │
│  │ routines        │  │ sessions        │  │ credit proxy     │    │
│  │ projects        │  │ insights        │  │ admin            │    │
│  │ identity        │  │                 │  │                  │    │
│  └────────┬────────┘  └────────┬────────┘  └────────┬─────────┘    │
│           │                   │                     │              │
│     ┌─────┴──────┐      ┌─────┴──────┐       ┌─────┴──────┐       │
│     │selfmanager │      │  timer_db  │       │analytics_db│       │
│     │    _db     │      │ (postgres) │       │ (postgres) │       │
│     │ (postgres) │      └────────────┘       └────────────┘       │
│     └────────────┘                                                 │
└──────────────────────────────────┬───────────────────────────────────┘
                                   │ X-Internal-Key
                                   ▼
┌──────────────────────────────────────────────────────────────────────┐
│                        INTERNAL SERVICE                             │
│                                                                     │
│          ┌─────────────────────────────────────┐                   │
│          │            billing                  │                   │
│          │         Python/FastAPI               │                   │
│          │           port 1229                  │                   │
│          │                                     │                   │
│          │  credit ledger  LLM inference        │                   │
│          │  model registry  sklearn pathway     │                   │
│          └──────────────────┬──────────────────┘                   │
│                             │                                       │
│                    ┌────────┴────────┐                              │
│                    │   billing_db    │     ┌──────────────────┐     │
│                    │  (postgres)     │     │  Ollama / Claude │     │
│                    └─────────────────┘     │  / OpenAI / GPT  │     │
│                                            └──────────────────┘     │
└──────────────────────────────────────────────────────────────────────┘
```

### Target State [PLANNED — Full Vision]

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              CLIENTS                                    │
│                                                                         │
│    PS_KMP (Android / iOS / Desktop)      Web App (future)               │
│    [full feature coverage]                                              │
└───────────────────────────────┬─────────────────────────────────────────┘
                                │ JWT (Bearer)
                                ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        USER-FACING SERVICES                             │
│                                                                         │
│  ┌─────────────┐ ┌──────────┐ ┌───────────┐ ┌──────────┐ ┌──────────┐  │
│  │selfmanager  │ │  timer   │ │analytics  │ │  user    │ │ journal  │  │
│  │Kotlin/Ktor  │ │Kotlin/   │ │Kotlin/    │ │Kotlin/   │ │Kotlin/   │  │
│  │             │ │Ktor      │ │Ktor       │ │Ktor      │ │Ktor      │  │
│  │tasks habits │ │pomodoro  │ │AI analysis│ │profiles  │ │entries   │  │
│  │routines     │ │sessions  │ │credit     │ │social    │ │mood      │  │
│  │notes        │ │focus     │ │proxy      │ │graph     │ │AI synth  │  │
│  │todo lists   │ │rooms     │ │           │ │          │ │          │  │
│  │projects     │ │streaks   │ │           │ │          │ │          │  │
│  │identity     │ │          │ │           │ │          │ │          │  │
│  └──────┬──────┘ └────┬─────┘ └─────┬─────┘ └────┬─────┘ └────┬─────┘  │
│         │             │             │             │            │        │
│     ┌───┴──┐      ┌───┴──┐     ┌───┴──┐     ┌───┴──┐    ┌───┴──┐     │
│     │ DB   │      │ DB   │     │ DB   │     │ DB   │    │ DB   │     │
│     └──────┘      └──────┘     └──────┘     └──────┘    └──────┘     │
│                                                                         │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                     social                                       │   │
│  │                  Kotlin/Ktor                                     │   │
│  │   community feed · achievements · habit blueprints              │   │
│  │   accountability partners · focus rooms · public profiles       │   │
│  │                    ┌──────┐                                      │   │
│  │                    │  DB  │                                      │   │
│  │                    └──────┘                                      │   │
│  └──────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────┬───────────────────────────────┘
                                          │ X-Internal-Key
                                          ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                           INTERNAL SERVICE                              │
│                                                                         │
│              ┌───────────────────────────────────────┐                 │
│              │               billing                 │                 │
│              │            Python/FastAPI             │                 │
│              │  credit ledger · LLM inference        │                 │
│              │  model registry · sklearn pathway     │                 │
│              │  trained behavioral models            │                 │
│              └───────────────┬───────────────────────┘                 │
│                              │                   ┌───────────────┐     │
│                        ┌─────┴──────┐            │ LLM Providers │     │
│                        │ billing_db │            │ Ollama/Claude │     │
│                        └────────────┘            │ OpenAI / GPT  │     │
│                                                  └───────────────┘     │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Service Inventory

| Service | Lang/Framework | Port | Status | Responsibility |
|---|---|---|---|---|
| `psocial_selfmanager` | Kotlin/Ktor | 1226 | **Live** | User identity, tasks, habits, routines, projects, tags, offline sync |
| `psocial_timer` | Kotlin/Ktor | 1227 | **Live** | Pomodoro sessions, intervals, focus analytics, time attribution |
| `psocial_analytics` | Kotlin/Ktor | 1228 | **Live** | AI analysis pipeline, credit proxy, admin reports |
| `psocial_billing` | Python/FastAPI | 1229 | **Live** | Credit ledger, LLM/sklearn inference engine, model registry |
| `psocial_dashboard` | Python/Streamlit | 1230 | **Live** | Admin web interface (short-term, until native admin is built) |
| `psocial_user_dashboard` | Python/Streamlit | 1231 | **Live** | User web interface (short-term, until KMP app is feature-complete) |
| `PS_KMP` | Kotlin/KMP + Compose | — | **In progress** | Primary mobile/desktop client (Android, iOS, JVM Desktop) |
| `psocial_user` | Kotlin/Ktor | 1232 | **Planned** | User profiles, avatars, bios, social graph |
| `psocial_journal` | Kotlin/Ktor | 1233 | **Planned** | Journal entries, mood tracking, AI-assisted daily synthesis |
| `psocial_social` | Kotlin/Ktor | 1234 | **Planned** | Achievements, community feed, habit blueprints, focus rooms |
| `psocial_notes` | Kotlin/Ktor | 1235 | **Planned** | Notes with entity linking to tasks, habits, projects |

---

## 4. Complete Service Interaction Map

### Current Interactions [CURRENT]

```
CLIENT (JWT Bearer)
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
    │         ├── X-Internal-Key ──▶ selfmanager:1226  (tasks, habits, routines)
    │         ├── X-Internal-Key ──▶ timer:1227         (pomodoro stats)
    │         └── X-Internal-Key ──▶ billing:1229       (predict, balance, transactions)
    │
    └──▶ billing:1229  (admin dashboard only, via X-Internal-Key)
              │
              └──▶ Ollama / Anthropic Claude / OpenAI GPT-4o
```

### Target Interactions [PLANNED — Full Vision]

```
CLIENT (JWT Bearer)
    │
    ├──▶ selfmanager:1226      tasks, habits, routines, notes, todo lists, sync
    │         │
    │         ├── X-Internal-Key ──▶ [internal/time-log]  ◀── timer:1227
    │         └── JDBC ──────────▶ selfmanager_db.users   ◀── timer:1227 (UserRegistry)
    │
    ├──▶ timer:1227             pomodoro, sessions, intervals, focus analytics
    │
    ├──▶ analytics:1228         AI analysis, credit proxy
    │         │
    │         ├── X-Internal-Key ──▶ selfmanager:1226      (tasks/habits/routines)
    │         ├── X-Internal-Key ──▶ timer:1227             (focus stats)
    │         ├── X-Internal-Key ──▶ journal:1233           (journal entries for synthesis)
    │         ├── X-Internal-Key ──▶ social:1234            (social context)
    │         └── X-Internal-Key ──▶ billing:1229           (inference + credits)
    │
    ├──▶ user:1232              profiles, social graph, follow/unfollow
    │
    ├──▶ journal:1233           entries, mood logs, AI synthesis
    │         │
    │         └── X-Internal-Key ──▶ analytics:1228        (trigger daily synthesis)
    │
    ├──▶ social:1234            achievements, feed, habit blueprints, focus rooms
    │         │
    │         ├── X-Internal-Key ──▶ selfmanager:1226      (habit streak data for blueprints)
    │         ├── X-Internal-Key ──▶ timer:1227             (focus room session data)
    │         └── X-Internal-Key ──▶ user:1232              (profile data for feed)
    │
    └──▶ notes:1235             notes with entity links
              │
              └── X-Internal-Key ──▶ selfmanager:1226      (entity validation)

billing:1229 (internal only, never called by clients directly)
    │
    ├──▶ Ollama (local / ngrok tunnel)
    ├──▶ Anthropic Claude API
    ├──▶ OpenAI GPT-4o API
    └──▶ sklearn .pkl models (loaded via joblib)
```

---

## 5. Authentication and Identity

### Authentication Flow [CURRENT]

```
┌─────────────────────────────────────────────────────────────────────┐
│  Step 1 — Client sends email to selfmanager                        │
│                                                                     │
│  POST /api/v1/auth/identify  {"email": "user@example.com"}         │
│                                                                     │
│  selfmanager:                                                       │
│    INSERT INTO users (email) VALUES (?)                             │
│    ON CONFLICT (email) DO UPDATE SET email = EXCLUDED.email         │
│    RETURNING id                          ──▶  userId = 42          │
│                                                                     │
│  Returns: { accessToken, refreshToken, userId: 42, email }         │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  Step 2 — Token structure (JWT HS256)                              │
│                                                                     │
│  Header:  { "alg": "HS256", "typ": "JWT" }                         │
│  Payload: { "userId": 42, "email": "user@example.com",             │
│             "iat": 1716000000, "exp": 1716086400 }                  │
│  Signature: HMAC-SHA256(header + "." + payload, JWT_SECRET)        │
│                                                                     │
│  Access token TTL:  24 hours                                        │
│  Refresh token TTL: 7 days                                          │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  Step 3 — Cross-service token acceptance                           │
│                                                                     │
│  selfmanager ─── same JWT_SECRET ───▶ timer      ✓ accepts token  │
│  selfmanager ─── same JWT_SECRET ───▶ analytics  ✓ accepts token  │
│                                                                     │
│  One login. One token. All services.                               │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  Step 4 — UserRegistry (canonical integer identity)                │
│                                                                     │
│  Problem: timer must assign the same userId as selfmanager         │
│  Solution: timer connects to selfmanager's users table via JDBC    │
│                                                                     │
│  timer ──JDBC──▶ selfmanager_db.users                              │
│    INSERT INTO users (email) ON CONFLICT DO UPDATE RETURNING id    │
│                                                                     │
│  Result: same userId = 42 regardless of which service saw          │
│  the user first. Only cross-database connection in the system.     │
└─────────────────────────────────────────────────────────────────────┘
```

### Authentication Flow — Extended [PLANNED]

When `psocial_user` is introduced, the identity model expands:

```
selfmanager       → owns the canonical users table (integer userId, email)
psocial_user      → owns the rich profile (displayName, avatar, bio, timezone)
                    reads userId from JWT, no cross-db connection needed
All other services → still use the same JWT (userId extracted from claims)
                    call psocial_user via X-Internal-Key to get profile data

The integer userId remains the universal key across all services.
psocial_user does NOT replace selfmanager's users table —
it layers richer identity data on top of the same integer key.
```

### Service-to-Service Authentication

```
┌────────────────────────────────────────────────────────┐
│  X-Internal-Key pattern                               │
│                                                       │
│  Caller adds header:  X-Internal-Key: <shared-secret> │
│  Receiver validates:  if header != INTERNAL_API_KEY   │
│                         return 401 Unauthorized       │
│                                                       │
│  Key is stored ONLY in environment variables.         │
│  Never in code. Never in API responses.               │
│  Never in Swagger documentation.                      │
│                                                       │
│  All services share the same INTERNAL_API_KEY value.  │
└────────────────────────────────────────────────────────┘
```

---

## 6. Current Services — Deep Dive

### 6.1 psocial_selfmanager [CURRENT]

**Responsibility:** The identity hub and primary productivity data store. Owns the `users` table that all other services reference.

**Tech stack:** Kotlin 1.9+, Ktor 2.x, Exposed ORM, PostgreSQL, HikariCP, Koin DI

**Entity model:**

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

**Key design decisions:**
- `syncId` UUID on every entity for offline-first idempotent sync
- `ON CONFLICT (sync_id) DO UPDATE` semantics — safe to retry
- Tags auto-created on reference; unique per user
- Internal endpoints return minimal representations for LLM prompt efficiency
- `time_spent_minutes` on tasks and habits updated atomically by timer via `POST /internal/time-log`

---

### 6.2 psocial_timer [CURRENT]

**Responsibility:** Pomodoro session lifecycle and all focus analytics.

**Tech stack:** Kotlin/Ktor, Exposed, PostgreSQL, HikariCP, Koin

**Session lifecycle state machine:**

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

  From ACTIVE:
    complete()  ──▶  COMPLETED  ──▶  fire POST /internal/time-log (fire-and-forget)
    abandon()   ──▶  ABANDONED
```

**Interval sequencing:**

```
Session created
    │
    ▼
 Work interval (25 min default)
    │
    ▼ complete
 completedWork % cyclesUntilLongBreak == 0?
    ├── YES ──▶  Long Break (15 min)
    └── NO  ──▶  Short Break (5 min)
    │
    ▼ complete
 Work interval
    │
    └── repeat
```

**Adaptive intelligence signals (returned on every relevant event):**

```
After work interval complete  →  suggestedNextBreak:
  totalWorkMinutesToday >= 90 AND no LongBreak in this session
    → ExtendedRest
  cycleCount triggers long break
    → LongBreak
  default
    → ShortBreak

On pause  →  abandonmentRisk:
  pauseCount >= personalThreshold (avg pause count at abandonment, min 3)
    → WARNING: "You've paused N times. You typically abandon at this point."
  sessionAge > 3× plannedDuration AND completedCycles == 0
    → WARNING: "Session has been idle. Consider abandoning."
```

---

### 6.3 psocial_analytics [CURRENT]

**Responsibility:** AI analysis pipeline and the only public gateway to billing.

**Analysis pipeline:**

```
POST /api/v1/analytics  {type, modelId, customPrompt?}
    │
    ├── Ensure user exists in analytics_users table
    │
    ├── Parallel upstream fetch (Kotlin coroutines):
    │     async { GET /internal/users/{id}/tasks      → selfmanager }
    │     async { GET /internal/users/{id}/habits     → selfmanager }
    │     async { GET /internal/users/{id}/routines   → selfmanager }
    │     async { GET /internal/users/{id}/stats      → timer       }
    │     awaitAll()
    │     (any failure → empty data, not a pipeline failure)
    │
    ├── Build inputData map:
    │     tasks, habits, routines, pomodoro stats + type-specific prompt
    │
    ├── POST /api/v1/internal/predict → billing
    │     (2-minute timeout for cold LLM starts)
    │     (failure here → error to user, no silent fallback)
    │
    ├── Extract insight text + creditsCharged
    │
    ├── INSERT INTO analytics_reports (...)
    │
    └── Return AnalysisResult to client
```

**Analysis types and prompt strategy:**

| Type | Prompt focus | Key cross-domain insight |
|---|---|---|
| `PRODUCTIVITY_SUMMARY` | Holistic overview of all dimensions | Correlation between focus time, habit completion, and task throughput |
| `TASK_PRIORITIZATION` | Pending tasks + time already invested | Identifies high-priority tasks being neglected despite focus time |
| `HABIT_INSIGHTS` | Streak patterns, at-risk habits, one change | Habit-focus session correlation |
| `ROUTINE_OPTIMIZATION` | Step efficiency, adherence gaps | Identifies which routine steps break adherence most often |
| `WEEKLY_TIMER_SUMMARY` | Focus time, completion rate, peak patterns | Best focus hours, most productive entity types |
| `CUSTOM` | User-supplied free-form prompt | Open-ended cross-domain query |
| `DAILY_REFLECTION` *(planned)* | Journal entry + day's activity data | Connects subjective mood with objective productivity output |

---

### 6.4 psocial_billing [CURRENT]

**Responsibility:** Internal credit ledger and multi-provider LLM/ML inference engine. Never called by clients directly.

**Credit balance model:**

```
Traditional approach (AVOID):
  users.balance column  ← updated on every transaction
  Risk: balance and transaction log can diverge on partial failure

ProductiveSocial approach:
  transactions table with balance_after column
  Current balance = SELECT balance_after FROM transactions
                    WHERE selfmanager_user_id = ?
                    ORDER BY created_at DESC LIMIT 1

  Balance IS the transaction log. Cannot be inconsistent.
```

**Prediction lifecycle:**

```
POST /api/v1/internal/predict
    │
    ├── Validate model (active, exists)
    ├── ensure_welcome_credits(userId)
    │     IF no transactions exist → INSERT DEPOSIT of 100 credits
    ├── Check balance >= model.cost_per_use
    │     IF insufficient → 402 PaymentRequired
    ├── INSERT predictions (status=PENDING)
    │
    ├── Dispatch to provider:
    │     OLLAMA     → POST /api/chat (OpenAI-compatible, ngrok tunnel in prod)
    │     ANTHROPIC  → anthropic SDK (claude-sonnet-4-6 etc.)
    │     OPENAI     → openai SDK (gpt-4o etc.)
    │     SKLEARN    → joblib.load(model_path).predict_proba(features)
    │
    ├── ON SUCCESS:
    │     INSERT transactions (type=CHARGE, amount=-cost, balance_after=new_balance)
    │     UPDATE predictions (status=SUCCESS, output_data=response)
    │     RETURN { result, creditsCharged }
    │
    └── ON FAILURE:
          UPDATE predictions (status=FAILED, error_message)
          NO transaction created — balance unchanged
          PROPAGATE error to caller
```

---

## 7. Planned Services — Design

### 7.1 psocial_user [PLANNED]

**Responsibility:** Rich user profile data and the social graph (follow/follower relationships).

**Why separate from selfmanager?** selfmanager owns productivity data and the minimal identity record needed for that (email + integer ID). A user profile with display name, avatar, bio, and social graph is a distinct domain — social presence and productivity workspace should not be coupled. Additionally, profile data will be read by multiple other services (social, analytics, journal) and having a single owner prevents duplication and consistency issues.

**Tech stack:** Kotlin/Ktor, PostgreSQL, HikariCP, Koin

**Database schema:**

```sql
-- Rich profile on top of selfmanager's integer user ID
CREATE TABLE user_profiles (
    id                  SERIAL PRIMARY KEY,
    selfmanager_user_id INTEGER NOT NULL UNIQUE,  -- FK to selfmanager.users.id (logical, not physical)
    display_name        VARCHAR(100),
    bio                 TEXT,
    avatar_url          TEXT,
    timezone            VARCHAR(50) DEFAULT 'UTC',
    is_public           BOOLEAN NOT NULL DEFAULT TRUE,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Social graph
CREATE TABLE follows (
    follower_id INTEGER NOT NULL,   -- selfmanager_user_id
    following_id INTEGER NOT NULL,  -- selfmanager_user_id
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    PRIMARY KEY (follower_id, following_id)
);

-- Notification preferences
CREATE TABLE notification_preferences (
    user_id                 INTEGER NOT NULL UNIQUE,
    habit_reminders         BOOLEAN NOT NULL DEFAULT TRUE,
    routine_reminders       BOOLEAN NOT NULL DEFAULT TRUE,
    social_achievements     BOOLEAN NOT NULL DEFAULT TRUE,
    weekly_summary          BOOLEAN NOT NULL DEFAULT TRUE,
    accountability_updates  BOOLEAN NOT NULL DEFAULT TRUE
);
```

**API surface:**

```
GET    /api/v1/users/me                  JWT  → own profile
PATCH  /api/v1/users/me                  JWT  → update profile (displayName, bio, avatar)
GET    /api/v1/users/{id}                JWT  → public profile view
POST   /api/v1/users/{id}/follow         JWT  → follow a user
DELETE /api/v1/users/{id}/follow         JWT  → unfollow
GET    /api/v1/users/me/followers        JWT  → list followers
GET    /api/v1/users/me/following        JWT  → list following
GET    /internal/users/{id}/profile      X-Internal-Key → profile for social/analytics
```

---

### 7.2 psocial_journal [PLANNED]

**Responsibility:** Daily journal entries, mood logs, and AI-assisted synthesis that connects subjective reflection with objective productivity data.

**The core value proposition of the journal service** is that it closes the loop between objective activity data (what was done, measured by selfmanager and the timer) and subjective experience (how it felt, written by the user). No existing journaling application performs this synthesis automatically because no existing application has access to both sides of it.

**Tech stack:** Kotlin/Ktor, PostgreSQL, HikariCP, Koin

**Database schema:**

```sql
CREATE TABLE journal_entries (
    id                  SERIAL PRIMARY KEY,
    sync_id             UUID NOT NULL UNIQUE,
    user_id             INTEGER NOT NULL,
    entry_date          DATE NOT NULL,         -- local date, not UTC timestamp
    written_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    mood                INTEGER,               -- 1–5 scale (1=very bad, 5=excellent)
    mood_label          VARCHAR(50),           -- optional emoji/label ("energised", "stressed")
    content             TEXT NOT NULL,         -- user's free-form text
    -- Auto-attached activity context (populated from selfmanager + timer at entry time)
    tasks_completed     JSONB,                 -- [{name, priority}]
    habits_completed    JSONB,                 -- [{name, type}]
    focus_minutes       INTEGER,               -- total Pomodoro work minutes on this date
    -- AI synthesis (populated after user triggers or on schedule)
    synthesis_text      TEXT,                  -- LLM-generated reflection
    synthesis_model     VARCHAR(100),
    synthesis_at        TIMESTAMPTZ,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE mood_log (
    id          SERIAL PRIMARY KEY,
    user_id     INTEGER NOT NULL,
    logged_at   TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    mood        INTEGER NOT NULL,              -- quick check-in without full entry
    context     VARCHAR(100)                   -- "morning", "after_session", "evening"
);
```

**API surface:**

```
POST   /api/v1/journal/entries            JWT → create entry (text + mood)
GET    /api/v1/journal/entries            JWT → list entries (paginated, date range)
GET    /api/v1/journal/entries/{date}     JWT → get entry for specific date
PATCH  /api/v1/journal/entries/{id}       JWT → edit entry
POST   /api/v1/journal/entries/{id}/synthesise  JWT → trigger AI synthesis for this entry
GET    /api/v1/journal/mood/history       JWT → mood log over time
POST   /api/v1/journal/mood              JWT → quick mood check-in
GET    /internal/users/{id}/entries      X-Internal-Key → entries for analytics context
```

**AI synthesis flow:**

```
User writes journal entry (content + mood)
    │
    ▼
Journal service automatically fetches (or receives):
    ├── Tasks completed today       ← selfmanager internal API
    ├── Habits completed today      ← selfmanager internal API
    └── Focus minutes today         ← timer internal API

These are stored as JSONB on the entry row.

User (or scheduled job) triggers synthesis:
    │
    ▼
Analytics service called with type = DAILY_REFLECTION:
    inputData = {
        journal_entry: entry.content,
        mood: entry.mood,
        tasks_completed: entry.tasks_completed,
        habits_completed: entry.habits_completed,
        focus_minutes: entry.focus_minutes
    }
    prompt = "You are a reflective productivity coach. Based on the user's
              journal entry, their mood, and their objective activity data
              for today, write a brief, warm, and insightful end-of-day
              synthesis. Identify one pattern and one concrete intention
              for tomorrow."
    │
    ▼
LLM generates synthesis text
    │
    ▼
Journal service stores synthesis_text on the entry
```

---

### 7.3 psocial_social [PLANNED]

**Responsibility:** The community and accountability layer. Designed to enable the motivational power of social accountability without introducing the attention fragmentation of mainstream social media.

**Design principles:**
1. No algorithmic feed — chronological only, no engagement optimization
2. Sharing is opt-in and achievement-based — no passive status updates
3. Social features are on a dedicated tab, not embedded in the productivity workspace
4. Notifications are low-frequency by default

**Tech stack:** Kotlin/Ktor, PostgreSQL, HikariCP, Koin

**Database schema:**

```sql
-- Achievements earned by reaching milestones
CREATE TABLE achievements (
    id              SERIAL PRIMARY KEY,
    user_id         INTEGER NOT NULL,
    type            VARCHAR(50) NOT NULL,   -- 'HABIT_STREAK_30', 'FOCUS_100H', etc.
    label           VARCHAR(100) NOT NULL,
    earned_at       TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    is_public       BOOLEAN NOT NULL DEFAULT FALSE,
    shared_at       TIMESTAMPTZ
);

-- Community feed posts (achievements + manual milestones)
CREATE TABLE feed_posts (
    id              SERIAL PRIMARY KEY,
    user_id         INTEGER NOT NULL,
    post_type       VARCHAR(30) NOT NULL,   -- 'ACHIEVEMENT', 'MILESTONE', 'BLUEPRINT'
    content         TEXT,
    reference_id    INTEGER,               -- achievement ID or blueprint ID
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Habit blueprints — shared habit templates
CREATE TABLE habit_blueprints (
    id              SERIAL PRIMARY KEY,
    author_user_id  INTEGER NOT NULL,
    name            VARCHAR(255) NOT NULL,
    description     TEXT,
    habit_type      VARCHAR(10) NOT NULL,  -- START / QUIT
    recurrency      VARCHAR(20) NOT NULL,
    suggested_time  TIME,
    tags            TEXT[],
    adopted_count   INTEGER NOT NULL DEFAULT 0,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Accountability partnerships
CREATE TABLE accountability_partners (
    user_id         INTEGER NOT NULL,
    partner_id      INTEGER NOT NULL,
    habit_name      VARCHAR(255),          -- shared habit they are accountable for
    goal_text       TEXT,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    PRIMARY KEY (user_id, partner_id)
);

-- Focus rooms — shared virtual focus spaces
CREATE TABLE focus_rooms (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            VARCHAR(100) NOT NULL,
    host_user_id    INTEGER NOT NULL,
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE focus_room_participants (
    room_id         UUID NOT NULL REFERENCES focus_rooms(id),
    user_id         INTEGER NOT NULL,
    status          VARCHAR(20) NOT NULL DEFAULT 'IDLE',  -- IDLE, FOCUSING, ON_BREAK, DONE
    joined_at       TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    PRIMARY KEY (room_id, user_id)
);
```

**API surface:**

```
GET    /api/v1/social/feed                      JWT → chronological community feed
GET    /api/v1/social/achievements              JWT → own achievements
POST   /api/v1/social/achievements/{id}/share   JWT → share achievement to feed
GET    /api/v1/social/blueprints                JWT → browse habit blueprints
POST   /api/v1/social/blueprints                JWT → publish a blueprint
POST   /api/v1/social/blueprints/{id}/adopt     JWT → adopt a blueprint (creates habit in selfmanager)
GET    /api/v1/social/accountability            JWT → list accountability partners + their progress
POST   /api/v1/social/accountability            JWT → add accountability partner
GET    /api/v1/social/rooms                     JWT → list active focus rooms
POST   /api/v1/social/rooms                     JWT → create a focus room
POST   /api/v1/social/rooms/{id}/join           JWT → join a focus room
PATCH  /api/v1/social/rooms/{id}/status         JWT → update own status in room
```

**Achievement trigger rules:**

| Achievement | Trigger condition |
|---|---|
| `HABIT_STREAK_7` | Any single habit completed 7 days in a row |
| `HABIT_STREAK_30` | Any single habit completed 30 days in a row |
| `HABIT_STREAK_100` | Any single habit completed 100 days in a row |
| `FOCUS_10H` | Accumulated 10 total hours of Pomodoro focus |
| `FOCUS_100H` | Accumulated 100 total hours |
| `TASKS_50` | 50 tasks completed |
| `ROUTINE_WEEK` | Same routine completed every day for 7 consecutive days |
| `BLUEPRINT_PUBLISHED` | First habit blueprint published |
| `BLUEPRINT_ADOPTED_10` | Blueprint adopted by 10 other users |

---

### 7.4 psocial_notes [PLANNED]

**Responsibility:** Free-form text notes with contextual links to productivity entities (tasks, habits, routines, projects).

**Key differentiator from external note apps:** Notes in ProductiveSocial are attached to the entities they belong to. Meeting notes for a project planning session are attached to the task "Plan Q3". Research notes for a habit are attached to the habit itself. Notes are always findable through the entity they relate to, not just through a full-text search.

**Database schema:**

```sql
CREATE TABLE notes (
    id              SERIAL PRIMARY KEY,
    sync_id         UUID NOT NULL UNIQUE,
    user_id         INTEGER NOT NULL,
    title           VARCHAR(500),
    content         TEXT NOT NULL,
    -- Entity link (optional)
    entity_type     VARCHAR(20),    -- 'TASK', 'HABIT', 'ROUTINE', 'PROJECT', NULL
    entity_id       INTEGER,        -- server-side integer ID of linked entity
    tags            TEXT[],
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_notes_entity ON notes (user_id, entity_type, entity_id);
```

**API surface:**

```
GET    /api/v1/notes                           JWT → list all notes (paginated)
POST   /api/v1/notes                           JWT → create note (with optional entity link)
GET    /api/v1/notes/{id}                      JWT → get note
PATCH  /api/v1/notes/{id}                      JWT → update note
DELETE /api/v1/notes/{id}                      JWT → delete note
GET    /api/v1/notes?entityType=TASK&entityId=15  JWT → all notes linked to a specific entity
POST   /api/v1/sync                            JWT → offline sync (same pattern as selfmanager)
```

---

## 8. Mobile Client — PS_KMP

### Architecture Overview

```
┌────────────────────────────────────────────────────┐
│                    PS_KMP                          │
│                                                    │
│  ┌─────────────────────────────────────────────┐   │
│  │               Presentation Layer           │   │
│  │   Compose Multiplatform UI + ViewModels    │   │
│  │   SelfManagerScreen  PomodoroScreen        │   │
│  │   JournalScreen      UserPageScreen        │   │
│  │   SocialScreen       NotesScreen           │   │
│  └───────────────────┬─────────────────────────┘   │
│                      │                             │
│  ┌───────────────────▼─────────────────────────┐   │
│  │                Data Layer                  │   │
│  │  Repositories (Task, Habit, Routine,       │   │
│  │   Pomodoro, Journal, Notes, Social)        │   │
│  │       ┌──────────────┬──────────────┐      │   │
│  │  LocalDS (SQLDelight) │  RemoteDS (Ktor)   │   │
│  └───────────────────────┴──────────────┘      │   │
│         │                        │              │   │
│  ┌──────▼──────┐          ┌──────▼──────┐       │   │
│  │  SQLDelight │          │  Ktor HTTP  │       │   │
│  │  (SQLite)   │          │  Clients    │       │   │
│  └─────────────┘          └─────────────┘       │   │
└────────────────────────────────────────────────────┘

Platforms:
  Android   → AndroidSqliteDriver + OkHttp engine
  iOS       → NativeSqliteDriver  + Darwin engine
  Desktop   → JdbcSqliteDriver    + Java engine
```

### Navigation Structure [Current + Planned]

```
NavHost
 │
 ├── /login                 Authentication screen
 │
 ├── /self-manager          Daily view (tasks + habits + routines)
 │     ├── /projects        Project grid
 │     │     └── /projects/{id}     Project detail
 │     ├── /tasks/create    New task
 │     ├── /tasks/{id}      Task detail / edit
 │     ├── /habits/create   New habit
 │     ├── /habits/{id}     Habit detail / edit
 │     ├── /routines/create New routine
 │     └── /routines/{id}/run  Routine Runner
 │
 ├── /pomodoro              Timer home
 │     ├── /pomodoro/session/{id}   Active session
 │     ├── /pomodoro/focus/{entityType}/{entityId}  Entity-linked session
 │     ├── /pomodoro/settings       Timer settings
 │     ├── /pomodoro/statistics     Focus patterns + entity stats
 │     └── /pomodoro/picker         Post-session task picker
 │
 ├── /journal               Journal entries list      [PLANNED]
 │     ├── /journal/entry/{date}   View / write entry
 │     └── /journal/mood           Mood history chart
 │
 ├── /notes                 Notes list                [PLANNED]
 │     └── /notes/{id}      Note detail / edit
 │
 ├── /social                Community feed            [PLANNED]
 │     ├── /social/achievements    Own achievements
 │     ├── /social/blueprints      Habit blueprints browser
 │     └── /social/rooms           Focus rooms
 │
 └── /user                  User page
       ├── /user/profile    Own profile / settings    [PLANNED]
       ├── /user/insights   AI analysis + reports
       └── /user/credits    Credit balance + history
```

### Offline Sync Protocol

```
Device                              Server (selfmanager)
  │                                        │
  │  All writes go to SQLDelight first     │
  │  isDirty = 1, serverId = NULL          │
  │                                        │
  │──── POST /api/v1/sync ────────────────▶│
  │     {                                  │
  │       creates: [entities with syncId] │
  │       updates: [changed entities]     │
  │       deletes: [syncIds to remove]    │
  │       lastSyncedAt: "2026-05-18T..."  │
  │     }                                  │
  │                                        │
  │◀─── SyncResponse ─────────────────────│
  │     {                                  │
  │       idMappings: {syncId → serverId} │
  │       serverChanges: [new/updated]    │
  │       errors: [per-item failures]     │
  │       syncedAt: "2026-05-18T..."      │
  │     }                                  │
  │                                        │
  │  Update local DB:                      │
  │    serverId columns populated          │
  │    isDirty = 0                         │
  │    isDeleted rows deleted              │
  │    serverChanges upserted              │
  │    lastSyncedAt stored                 │
```

---

## 9. Data Flow Diagrams

### 9.1 AI Analysis Pipeline (Current)

```
User / Client
     │
     │  POST /api/v1/analytics {type, modelId}  (JWT)
     ▼
psocial_analytics
     │
     ├── [async] GET /internal/users/{id}/tasks      ──▶ selfmanager
     ├── [async] GET /internal/users/{id}/habits     ──▶ selfmanager
     ├── [async] GET /internal/users/{id}/routines   ──▶ selfmanager
     └── [async] GET /internal/users/{id}/stats      ──▶ timer
          │
          │  awaitAll() — total latency = max(tasks, habits, routines, stats)
          ▼
     Build inputData:
     { tasks: [...], habits: [...], routines: [...],
       pomodoro: {...}, prompt: "..." }
          │
          │  POST /api/v1/internal/predict  (X-Internal-Key)
          ▼
psocial_billing
     │
     ├── Validate model (active, exists)
     ├── ensure_welcome_credits() — first-use deposit if needed
     ├── Check balance >= cost
     ├── INSERT predictions (PENDING)
     │
     └── Call LLM:
           OLLAMA    → POST /api/chat  (via ngrok in prod)
           ANTHROPIC → anthropic.messages.create()
           OPENAI    → openai.chat.completions.create()
               │
               ▼
          insight text returned
               │
     ├── INSERT transactions (CHARGE)
     └── UPDATE predictions (SUCCESS)
          │
          ▼
     Return {result, creditsCharged}
          │
          ▼
psocial_analytics
     │
     ├── INSERT analytics_reports
     └── Return AnalysisResult to client
```

### 9.2 AI Analysis Pipeline (Future — with Journal)

```
User / Client
     │
     │  POST /api/v1/analytics {type: DAILY_REFLECTION, modelId}  (JWT)
     ▼
psocial_analytics
     │
     ├── [async] GET /internal/users/{id}/tasks     ──▶ selfmanager
     ├── [async] GET /internal/users/{id}/habits    ──▶ selfmanager
     ├── [async] GET /internal/users/{id}/stats     ──▶ timer
     └── [async] GET /internal/users/{id}/entries   ──▶ journal   [PLANNED]
          │
          ▼
     Build inputData:
     { tasks_completed: [...], habits_completed: [...],
       focus_minutes: N, journal_entry: "...", mood: 4 }
     prompt: "Connect this user's subjective reflection with
              their objective productivity data today..."
          │
          ▼
psocial_billing → LLM → synthesis text
          │
          ▼
psocial_analytics → return synthesis
          │
          ▼
psocial_journal → store synthesis_text on entry   [PLANNED]
```

### 9.3 Social Engagement Flow [PLANNED]

```
User completes 30-day habit streak
     │
     ▼
selfmanager fires event (or analytics detects on next analysis)
     │
     ▼
psocial_social
  INSERT achievements (type='HABIT_STREAK_30', user_id=42)
     │
     ▼
User opens Achievements screen
  Sees new badge: "30-Day Streak: Morning Exercise"
  Taps "Share to Community"
     │
     ▼
psocial_social
  INSERT feed_posts (type='ACHIEVEMENT', reference_id=achievement.id)
     │
     ▼
Followers see post in their community feed
  GET /api/v1/social/feed
  → [{user: {displayName, avatar}, achievement: {label, earnedAt}}]
```

### 9.4 Offline Sync with Conflict Resolution

```
SCENARIO: User edits task on phone (offline), then on web dashboard (online)

Phone (offline):
  UPDATE tasks SET title='New title A', isDirty=1 WHERE syncId='abc-123'

Web dashboard (online):
  PATCH /api/v1/tasks/task/{id}  {title: 'New title B'}
  → server updates title to 'New title B', updatedAt = T2

Phone comes online and syncs:
  POST /api/v1/sync
  { updates: [{syncId: 'abc-123', title: 'New title A', updatedAt: T1}] }

Server applies conflict resolution:
  Server's updatedAt (T2) > client's updatedAt (T1)
  → Server wins: title remains 'New title B'
  → serverChanges includes the task with title='New title B'

Phone receives response:
  serverChanges: [{syncId: 'abc-123', title: 'New title B', ...}]
  → local title overwritten with 'New title B'
  → isDirty = 0
```

---

## 10. Database Schemas

### selfmanager_db — Key Tables

```sql
-- Canonical user identity (shared with timer via JDBC)
users (id SERIAL PK, email VARCHAR UNIQUE, created_at TIMESTAMPTZ)

-- Projects
projects (id, sync_id UUID UNIQUE, user_id → users, name, color, icon,
          priority, created_at, updated_at)

-- Tasks
tasks (id, sync_id UUID UNIQUE, user_id, project_id → projects,
       title, description, priority, urgency, completed BOOL,
       time_spent_minutes INT DEFAULT 0,
       due_date, is_recurring, target,
       created_at, updated_at)

subtasks (id, sync_id UUID UNIQUE, task_id → tasks,
          name, completed BOOL, position INT, created_at)

task_scheduled_times (id, task_id, scheduled_time TIME, created_at)

-- Habits
habits (id, sync_id UUID UNIQUE, user_id, project_id,
        name, description, habit_type VARCHAR(10),  -- START | QUIT
        recurrency, time_spent_minutes INT DEFAULT 0,
        created_at, updated_at)

habit_completions (id, sync_id UUID UNIQUE, habit_id, user_id,
                   completed_at DATE,  -- local date
                   created_at TIMESTAMPTZ)

habit_subtask_completions (id, habit_completion_id, subtask_id,
                           completed BOOL, created_at)

-- Routines
routines (id, sync_id UUID UNIQUE, user_id, project_id,
          name, recurrency, created_at, updated_at)

routine_steps (id, sync_id UUID UNIQUE, routine_id,
               name, duration_minutes INT, position INT,
               auto_start BOOL DEFAULT FALSE)

routine_completions (id, sync_id UUID UNIQUE, routine_id, user_id,
                     completed_at TIMESTAMPTZ)

routine_step_completions (id, routine_completion_id, step_id,
                          completed BOOL, time_taken_minutes INT)

-- Tags (shared across entity types)
tags (id, user_id, name VARCHAR(100), UNIQUE(user_id, name))
task_tags (task_id, tag_id, PRIMARY KEY (task_id, tag_id))
habit_tags (habit_id, tag_id, PRIMARY KEY (habit_id, tag_id))
routine_tags (routine_id, tag_id, PRIMARY KEY (routine_id, tag_id))
```

### timer_db — Key Tables

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
    entity_type VARCHAR(20),   -- TASK | HABIT | ROUTINE | NULL
    entity_id   INTEGER,
    status      VARCHAR(20),   -- ACTIVE | PAUSED | COMPLETED | ABANDONED
    total_work_minutes INT DEFAULT 0,
    completed_cycles   INT DEFAULT 0,
    pause_count        INT DEFAULT 0,
    -- Settings snapshot (preserved for historical accuracy)
    work_duration_snapshot  INT,
    short_break_snapshot    INT,
    long_break_snapshot     INT,
    cycles_snapshot         INT,
    started_at  TIMESTAMPTZ,
    completed_at TIMESTAMPTZ
)

pomodoro_intervals (
    id, session_id → pomodoro_sessions,
    interval_type   VARCHAR(15),  -- WORK | SHORT_BREAK | LONG_BREAK
    status          VARCHAR(15),  -- IN_PROGRESS | COMPLETED | ABANDONED
    planned_minutes INT,
    actual_minutes  INT,          -- NULL until interval ends
    started_at TIMESTAMPTZ,
    ended_at   TIMESTAMPTZ
)
```

### billing_db — Key Tables

```sql
ml_models (
    id UUID PK DEFAULT gen_random_uuid(),
    name, provider VARCHAR(50),   -- OLLAMA | ANTHROPIC | OPENAI | SKLEARN
    model_name, cost_per_use INT DEFAULT 5,
    system_prompt TEXT,
    is_active BOOL DEFAULT TRUE,
    created_at TIMESTAMPTZ
)

transactions (
    id UUID PK,
    selfmanager_user_id INT NOT NULL,
    transaction_type VARCHAR(20),  -- DEPOSIT | CHARGE | REFUND
    amount INT NOT NULL,           -- positive = deposit, negative = charge
    description TEXT,
    prediction_id UUID → predictions,
    balance_after INT NOT NULL,    -- running balance after this transaction
    created_at TIMESTAMPTZ
)
-- Index: (selfmanager_user_id, created_at DESC) for O(log n) balance lookup

predictions (
    id UUID PK,
    model_id UUID → ml_models,
    selfmanager_user_id INT,
    status VARCHAR(20),   -- PENDING | SUCCESS | FAILED
    input_data  JSONB,    -- full user context sent to LLM
    output_data JSONB,    -- full LLM response
    error_message TEXT,
    credits_charged INT,
    created_at TIMESTAMPTZ,
    completed_at TIMESTAMPTZ
)
```

### analytics_db

```sql
analytics_users (
    id, selfmanager_id INT UNIQUE, email, is_admin BOOL DEFAULT FALSE,
    created_at TIMESTAMPTZ
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

## 11. API Surface Summary

### Authentication Endpoints (selfmanager)

```
POST /api/v1/auth/identify       → Find-or-create user, return JWT pair
POST /api/v1/auth/refresh        → Rotate refresh token
POST /api/v1/auth/logout         → Revoke single device token
POST /api/v1/auth/logout-all     → Revoke all tokens
```

### selfmanager Data Endpoints (JWT required)

```
Sync:
  POST /api/v1/sync              → Batch offline sync (all entity types)

Projects:
  GET/POST    /api/v1/projects/projects
  GET/PATCH/DELETE  /api/v1/projects/project/{id}

Tasks:
  GET/POST    /api/v1/tasks/tasks
  GET/PATCH/DELETE  /api/v1/tasks/task/{id}

Habits:
  GET/POST    /api/v1/habits/habits
  GET/PATCH/DELETE  /api/v1/habits/habit/{id}
  POST        /api/v1/habits/habit/{id}/complete

Routines:
  GET/POST    /api/v1/routines/routines
  GET/PATCH/DELETE  /api/v1/routines/routine/{id}
  POST        /api/v1/routines/routine/{id}/complete

Internal (X-Internal-Key):
  GET   /internal/users/{id}/tasks
  GET   /internal/users/{id}/habits
  GET   /internal/users/{id}/routines
  POST  /internal/time-log
```

### timer Endpoints (JWT required)

```
GET/PUT    /api/v1/pomodoro/settings
GET/POST   /api/v1/pomodoro/sessions
POST       /api/v1/pomodoro/sessions/{id}/start-interval
POST       /api/v1/pomodoro/sessions/{id}/complete-interval
PATCH      /api/v1/pomodoro/sessions/{id}/pause|resume|complete
DELETE     /api/v1/pomodoro/sessions/{id}/abandon
POST       /api/v1/pomodoro/sync
GET        /api/v1/pomodoro/insights/focus-patterns
GET        /api/v1/pomodoro/insights/entity-stats

Internal (X-Internal-Key):
  GET  /internal/users/{id}/stats
```

### analytics Endpoints (JWT required)

```
POST /api/v1/analytics           → Run AI analysis
GET  /api/v1/analytics           → List own reports
GET  /api/v1/credits/balance     → Credit balance (proxied from billing)
GET  /api/v1/credits/transactions → Transaction history (proxied)

Admin (JWT + ADMIN_EMAILS):
  GET    /api/v1/admin/stats
  GET    /api/v1/admin/users
  GET    /api/v1/admin/reports
  GET/DELETE  /api/v1/admin/users/{id}/reports
  POST   /api/v1/admin/seed-reports
  PATCH  /api/v1/admin/users/{id}/toggle-admin

Internal (X-Internal-Key):
  GET  /internal/users/{id}/stats
```

### billing Endpoints (X-Internal-Key required)

```
GET    /api/v1/models                    → List active models (public, no auth)
POST/GET/PUT  /api/v1/internal/models    → Model registry management
POST   /api/v1/internal/predict          → Run inference + charge credits
GET    /api/v1/internal/users/{id}/balance
GET    /api/v1/internal/users/{id}/transactions
POST   /api/v1/internal/users/{id}/deposit
```

---

## 12. Deployment Architecture

### Local Development

```
Colima (macOS container runtime)
    │
    └── Docker Compose
          │
          ├── selfmanager container     (port 1226) ← pre-built fat JAR
          ├── selfmanager_db container  (port 5432)
          │
          ├── timer container           (port 1227) ← pre-built fat JAR
          ├── timer_db container        (port 5434)
          │
          ├── analytics container       (port 1228) ← pre-built fat JAR
          ├── analytics_db container    (port 5435)
          │
          ├── billing container         (port 1229) ← Python, pip install at start
          ├── billing_db container      (port 5433)
          │
          ├── dashboard container       (port 1230) ← Streamlit
          └── user_dashboard container  (port 1231) ← Streamlit

All containers on psocial_network bridge.
Container-to-container by service name: http://selfmanager:1226

Local Ollama on host machine (port 11434)
  accessed from billing container via host.docker.internal:11434
```

### Production (Current)

```
Render.com (free tier)
    │
    ├── psocial_selfmanager  Web Service  → Neon PostgreSQL (selfmanager_db)
    ├── psocial_timer        Web Service  → Neon PostgreSQL (timer_db)
    ├── psocial_analytics    Web Service  → Neon PostgreSQL (analytics_db)
    ├── psocial_billing      Web Service  → Neon PostgreSQL (billing_db)
    ├── psocial_dashboard    Web Service
    └── psocial_user_dashboard  Web Service

Ollama (developer's local machine)
    └── ngrok HTTPS tunnel → billing OLLAMA_BASE_URL env var

Cold start on free tier: ~50s (JVM), ~8s (Python)
```

### Production (Target)

```
Render.com (paid tier — always-on containers)
    │
    ├── All 6 current services (upgraded to eliminate cold starts)
    ├── psocial_user       (new)
    ├── psocial_journal    (new)
    ├── psocial_social     (new)
    └── psocial_notes      (new)

Databases: Neon PostgreSQL (one cluster per service, paid tier)

LLM Providers (no more ngrok):
    ├── Anthropic Claude API (primary)
    └── OpenAI GPT-4o API (secondary / fallback)

CDN: for static assets and avatar images (Cloudflare or AWS S3)

Push Notifications: Firebase Cloud Messaging (Android) + APNs (iOS)
```

---

## 13. ML and AI Roadmap

### Phase 1 — LLM Prompting (Current)

All AI analysis uses retrieval-augmented generation: live user data fetched at inference time, passed to a large language model with type-specific prompts.

```
No training required.
No labeled data required.
Scales immediately to any new user.
Output quality limited by prompt design, not by training data.
```

### Phase 2 — Hybrid LLM + Trained Models (Planned)

As the platform accumulates real user behavioral data, narrow prediction tasks can be addressed more accurately with trained models:

```
Target predictive tasks:

┌─────────────────────────────────┬─────────────────────┬──────────────────────────┐
│ Task                            │ Model type          │ Training signal          │
├─────────────────────────────────┼─────────────────────┼──────────────────────────┤
│ Habit continuation probability  │ Gradient boosting   │ Completion logs + streaks│
│ Task completion likelihood      │ Logistic regression │ Task history + context   │
│ Optimal session duration        │ Linear regression   │ Completed interval logs  │
│ Abandonment risk score          │ Gradient boosting   │ Pause count at abandon   │
│ Daily productivity score        │ Neural network      │ Composite multi-signal   │
└─────────────────────────────────┴─────────────────────┴──────────────────────────┘

Deployment path (no system changes required):
  1. Train model offline with accumulated data
  2. Serialize to .pkl via joblib
  3. Upload to billing: POST /api/v1/models/{id}/upload
  4. Model appears in model picker alongside LLM options
  5. billing invokes: model.predict_proba(feature_vector)
```

### Phase 3 — On-Device Inference (Future)

For fully offline AI features, lightweight models (ONNX or Core ML format) can be bundled into the KMP app:

```
Small classification models (habit risk, productivity score)
  → converted to ONNX / Core ML
  → bundled in KMP app
  → inference runs on device, no network required
  → syncs outputs back to server for analytics aggregation
```

---

## 14. Security Model

### Threat Boundaries

```
UNTRUSTED ──────────────────────────────────────────────────────────────────
  Mobile client / web browser / any HTTP client
      │
      │  Only JWT Bearer tokens accepted
      │  Token must be signed with correct JWT_SECRET
      │  Expired tokens rejected (24h TTL)
      ▼
USER-FACING SERVICES (selfmanager, timer, analytics)
      │
      │  X-Internal-Key required for all /internal/* endpoints
      │  Key stored only in environment variables, never in code
      ▼
INTERNAL SERVICES (billing)
      │
      │  No JWT auth whatsoever
      │  All endpoints require X-Internal-Key
      │  Billing URL never returned to clients
      │  Not documented in public Swagger
TRUSTED ────────────────────────────────────────────────────────────────────
```

### Secret Inventory

| Secret | Where stored | Who knows it | Rotation impact |
|---|---|---|---|
| `JWT_SECRET` | Env vars on selfmanager, timer, analytics | 3 services | All 3 must redeploy simultaneously |
| `INTERNAL_API_KEY` | Env vars on all services | All services | All must redeploy |
| `DATABASE_URL` per service | Env vars on each service | 1 service each | Individual service redeploy |
| `ANTHROPIC_API_KEY` | Env var on billing | billing only | billing redeploy |
| `OPENAI_API_KEY` | Env var on billing | billing only | billing redeploy |

### Known Security Gaps (Pre-Production)

1. **Email-only auth** — no verification, susceptible to account enumeration
2. **Shared JWT secret** — compromise affects all services simultaneously; RS256 with per-service public keys is the production solution
3. **No rate limiting** on `/auth/identify`
4. **ngrok Ollama tunnel** — not appropriate for multi-user production
5. **No audit log** — credit transactions are logged but service-to-service calls are not
