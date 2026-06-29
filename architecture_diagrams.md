# ProductiveSocial — Architecture Diagrams

---

## 1. Overall System Architecture

```mermaid
graph TD
    subgraph Clients
        KMP[PS_kmp\nAndroid / iOS / Desktop]
        AdminDash[psocial_dashboard\nAdmin Dashboard - 1230]
    end

    subgraph Backend
        SM[psocial_selfmanager\n1226 - Kotlin / Ktor]
        Timer[psocial_timer\n1227 - Kotlin / Ktor]
        Analytics[psocial_analytics\n1228 - Kotlin / Ktor]
        Billing[psocial_billing\n1229 - Python / FastAPI]
    end

    subgraph Databases
        SM_DB[(selfmanager_db\nPostgreSQL 5432)]
        Timer_DB[(timer_db\nPostgreSQL 5434)]
        Analytics_DB[(analytics_db\nPostgreSQL 5435)]
        Billing_DB[(billing_db\nPostgreSQL 5433)]
    end

    subgraph LLM
        Ollama[Ollama\nllama3.2 local]
        Anthropic[Anthropic API]
        OpenAI[OpenAI API]
    end

    KMP -->|JWT Bearer| SM
    KMP -->|JWT Bearer| Timer
    KMP -->|JWT Bearer| Analytics

    AdminDash -->|JWT| SM
    AdminDash -->|JWT| Analytics
    AdminDash -->|X-Internal-Key| Billing

    Analytics -->|X-Internal-Key| SM
    Analytics -->|X-Internal-Key| Timer
    Analytics -->|X-Internal-Key| Billing

    Timer -->|X-Internal-Key fire-and-forget| SM
    Timer -->|JDBC shared users table| SM_DB

    SM --- SM_DB
    Timer --- Timer_DB
    Analytics --- Analytics_DB
    Billing --- Billing_DB

    Billing --> Ollama
    Billing --> Anthropic
    Billing --> OpenAI
```

---

## 2. psocial_selfmanager - 1226

```mermaid
graph TD
    subgraph Incoming
        KMP_SM[KMP Client]
        Analytics_SM[psocial_analytics\nX-Internal-Key]
        Timer_SM[psocial_timer\nX-Internal-Key / JDBC]
        Dash_SM[Dashboards\nJWT]
    end

    subgraph Routes
        Auth["/api/v1/auth\nidentify - refresh - logout"]
        Projects["/api/v1/projects\nCRUD"]
        Tasks["/api/v1/tasks\nCRUD - subtasks - time-log"]
        Habits["/api/v1/habits\nCRUD - scheduling"]
        Routines["/api/v1/routines\nCRUD - steps"]
        Sync["/api/v1/sync\nbatched offline sync"]
        Internal["/internal/users/id\ntasks - habits - routines - time-log"]
    end

    subgraph selfmanager_db
        Users[(users)]
        ProjectsT[(projects)]
        TasksT[(tasks - subtasks - tags)]
        HabitsT[(habits)]
        RoutinesT[(routines - steps)]
    end

    subgraph Outgoing
        Billing_SM[psocial_billing\nwelcome deposit on first user]
    end

    KMP_SM --> Auth
    KMP_SM --> Projects
    KMP_SM --> Tasks
    KMP_SM --> Habits
    KMP_SM --> Routines
    KMP_SM --> Sync
    Dash_SM --> Projects
    Dash_SM --> Tasks
    Analytics_SM --> Internal
    Timer_SM --> Internal

    Auth --> Users
    Projects --> ProjectsT
    Tasks --> TasksT
    Habits --> HabitsT
    Routines --> RoutinesT
    Sync --> Users
    Sync --> ProjectsT
    Sync --> TasksT
    Sync --> HabitsT
    Sync --> RoutinesT
    Internal --> TasksT
    Internal --> HabitsT
    Internal --> RoutinesT

    Auth --> Billing_SM
```

---

## 3. psocial_timer - 1227

```mermaid
graph TD
    subgraph Incoming
        KMP_T[KMP Client\nJWT Bearer]
        Analytics_T[psocial_analytics\nX-Internal-Key]
    end

    subgraph Routes
        AuthT["/api/v1/auth/identify"]
        Settings["/api/v1/pomodoro/settings\nGET - PUT"]
        Sessions["/api/v1/pomodoro/sessions\ncreate - list - detail"]
        Lifecycle["/api/v1/pomodoro/sessions/id\nstart - pause - resume - complete - abandon"]
        Insights["/api/v1/pomodoro/insights\nfocus patterns - entity stats"]
        SyncT["/api/v1/pomodoro/sync\nbatched offline sync"]
        InternalT["/internal/users/id/stats\naggregate pomodoro stats"]
    end

    subgraph timer_db
        SessionsT[(sessions\nstate machine)]
        IntervalsT[(intervals\nwork - break chunks)]
        SettingsT[(pomodoro_settings\nper-user)]
    end

    subgraph Outgoing
        SM_T[psocial_selfmanager\nPOST /internal/time-log\nfire-and-forget]
        SM_DB_T[(selfmanager_db\nusers table - JDBC)]
    end

    KMP_T --> AuthT
    KMP_T --> Settings
    KMP_T --> Sessions
    KMP_T --> Lifecycle
    KMP_T --> Insights
    KMP_T --> SyncT
    Analytics_T --> InternalT

    AuthT --> SM_DB_T
    Sessions --> SessionsT
    Lifecycle --> SessionsT
    Lifecycle --> IntervalsT
    Lifecycle --> SM_T
    Settings --> SettingsT
    Insights --> SessionsT
    Insights --> IntervalsT
    SyncT --> SessionsT
    SyncT --> IntervalsT
    InternalT --> SessionsT
    InternalT --> IntervalsT
```

---

## 4. psocial_analytics - 1228

```mermaid
graph TD
    subgraph Incoming
        KMP_A[KMP Client\nJWT Bearer]
        AdminDash_A[Admin Dashboard\nJWT / X-Internal-Key]
    end

    subgraph Routes
        AnalyticsR["/api/v1/analytics\nPOST trigger - GET list"]
        Admin_A["/api/v1/admin\nstats - users - reports"]
        Credits_A["/api/v1/credits\nbalance - transactions - proxy"]
        Parse["/api/v1/analytics\nparse-task - parse-habit - parse-routine"]
        Prioritize["/api/v1/analytics/prioritize-tasks"]
        Coaching["/api/v1/analytics/habit-coaching"]
        Optimization["/api/v1/analytics/routine-optimization"]
        InternalA["/internal/users/id/stats"]
    end

    subgraph analytics_db
        Reports[(analytics_reports)]
        UsersA[(users - isAdmin flag)]
    end

    subgraph Outgoing
        SM_A[psocial_selfmanager\nGET /internal/users/id\ntasks - habits - routines]
        Timer_A[psocial_timer\nGET /internal/users/id/stats]
        Billing_A[psocial_billing\nPOST /api/v1/internal/predict\nX-Internal-Key]
    end

    KMP_A --> AnalyticsR
    KMP_A --> Credits_A
    KMP_A --> Parse
    KMP_A --> Prioritize
    KMP_A --> Coaching
    KMP_A --> Optimization
    AdminDash_A --> Admin_A
    AdminDash_A --> AnalyticsR

    AnalyticsR --> SM_A
    AnalyticsR --> Timer_A
    AnalyticsR --> Billing_A
    AnalyticsR --> Reports
    Parse --> Billing_A
    Prioritize --> SM_A
    Prioritize --> Billing_A
    Coaching --> Billing_A
    Optimization --> Billing_A
    Credits_A --> Billing_A
    Admin_A --> UsersA
    Admin_A --> Reports
    InternalA --> Reports
```

---

## 5. psocial_billing - 1229

```mermaid
graph TD
    subgraph Incoming
        Analytics_B[psocial_analytics\nX-Internal-Key]
        AdminDash_B[Admin Dashboard\nX-Internal-Key]
    end

    subgraph Routes
        AuthB["/api/v1/auth/identify\nfind-or-create by email"]
        UsersB["/api/v1/users/me\nprofile - balance - transactions"]
        Models["/api/v1/models\nCRUD - LLM model registry"]
        Predictions["/api/v1/predictions\nlist - stats"]
        InternalB["/api/v1/internal/predict\nX-Internal-Key"]
        InternalUsers["/api/v1/internal/users/id\nbalance - transactions - deposit"]
        AdminB["/api/v1/admin\nuser management - system stats"]
    end

    subgraph billing_db
        MLModels[(ml_models\nname - provider - cost)]
        PredictionsT[(predictions\nstatus - input - output - credits)]
        Transactions[(transactions\ndebit - credit - balance_after)]
    end

    subgraph LLMProviders
        Ollama_B[Ollama\nllama3.2]
        Anthropic_B[Anthropic API]
        OpenAI_B[OpenAI API]
    end

    Analytics_B --> InternalB
    Analytics_B --> InternalUsers
    AdminDash_B --> AdminB
    AdminDash_B --> InternalUsers
    AdminDash_B --> Models

    InternalB --> MLModels
    InternalB --> PredictionsT
    InternalB --> Transactions
    InternalB --> Ollama_B
    InternalB --> Anthropic_B
    InternalB --> OpenAI_B

    AuthB --> Transactions
    UsersB --> Transactions
    Models --> MLModels
    Predictions --> PredictionsT
    InternalUsers --> Transactions
    AdminB --> Transactions
    AdminB --> PredictionsT
```

---

## 6. psocial_dashboard - Admin - 1230

```mermaid
graph TD
    subgraph Pages
        Login[Login\nemail via selfmanager]
        Overview[Overview\nsystem stats]
        Users[Users\nbalances - top-up - promote]
        ModelsPage[Models\nregister - delete]
        PredictionsPage[Predictions\nsystem-wide]
        AnalyticsPage[Analytics\nall reports]
    end

    subgraph Outgoing
        SM_D[psocial_selfmanager 1226\nJWT auth - user verification]
        Analytics_D[psocial_analytics 1228\nJWT - admin endpoints - reports]
        Billing_D[psocial_billing 1229\nX-Internal-Key - users - models - transactions]
    end

    Login --> SM_D
    Login --> Analytics_D
    Overview --> Billing_D
    Overview --> Analytics_D
    Users --> Billing_D
    ModelsPage --> Billing_D
    PredictionsPage --> Billing_D
    AnalyticsPage --> Analytics_D
```

---

## 7. Authentication Flow (Client)

```mermaid
graph TD
    A[User enters email] --> B["POST /api/v1/auth/identify\nbody: email"]
    B --> C[selfmanager\nfind-or-create user]
    C --> D["response:\naccessToken - refreshToken - userId"]
    D --> E["sync_metadata\naccessToken - refreshToken\nserverId - email"]
    E --> F[AppSession\nloaded in memory]

    F --> G{token expired\n24h?}
    G -->|no| H[requests continue\nBearer accessToken]
    G -->|yes| I["POST /api/v1/auth/refresh\nbody: refreshToken"]
    I --> J["response:\nnew accessToken - refreshToken"]
    J --> E

    F --> K[user logs out]
    K --> L["logout\nwipe all local tables\nclear sync_metadata\nclear AppSession"]
```

---

## 8. JWT Authentication Flow (Inter-Service)

```mermaid
sequenceDiagram
    participant Client as PS_kmp / Dashboard
    participant SM as psocial_selfmanager
    participant Timer as psocial_timer
    participant Analytics as psocial_analytics

    Client->>SM: POST /api/v1/auth/identify (email)
    SM-->>Client: access_token (24h) + refresh_token (7d)

    Client->>SM: GET /api/v1/tasks (Bearer access_token)
    SM-->>Client: 200 OK

    Client->>Timer: GET /api/v1/pomodoro/sessions (Bearer access_token)
    Note over Timer: Validates token using shared JWT_SECRET
    Timer-->>Client: 200 OK

    Client->>Analytics: POST /api/v1/analytics (Bearer access_token)
    Note over Analytics: Validates token using shared JWT_SECRET
    Analytics-->>Client: 200 OK

    Note over Client,SM: access_token expires after 24h

    Client->>SM: POST /api/v1/auth/refresh (Bearer refresh_token)
    SM-->>Client: new access_token (24h)

    Client->>SM: POST /api/v1/auth/identify (invalid token)
    SM-->>Client: 401 Unauthorized
```

---

## 8. Project Flow

```mermaid
graph TD
    A[User opens Projects screen] --> B[ProjectGrid\n2-column card layout]
    B --> C[ProjectItem card\ncolor gradient - icon - name]

    C --> D[User taps project]
    D --> E[ProjectDetailsScreen\ntasks - habits - routines\ntags - priority]

    B --> F[User creates project]
    F --> G["INSERT into project\nsyncId UUID - no serverId yet\nvisible immediately"]

    G --> H{network\navailable?}
    H -->|yes| I["POST /api/v1/projects\nbody: name - description\niconName - colorHex\npriority - tags"]
    I --> J["response: serverId\nUPDATE project SET serverId"]
    H -->|no| K[WorkManager\nqueues sync for later]
    K --> I

    G --> L[Default project\ncreated by selfmanager\non first registration]
```

---

## 9. Offline Sync Flow

```mermaid
sequenceDiagram
    participant Client as PS_kmp
    participant LocalDB as SQLDelight
    participant SM as psocial_selfmanager
    participant Timer as psocial_timer

    Note over Client,LocalDB: User is offline — all writes go to local DB

    Client->>LocalDB: create task (clientId: abc-123)
    Client->>LocalDB: update habit (clientId: def-456)
    Client->>LocalDB: complete pomodoro session (clientId: ghi-789)

    Note over Client,Timer: Connection restored — sync triggered

    Client->>SM: POST /api/v1/sync
    Note over SM: Processes creates / updates / deletes
    Note over SM: clientId already exists — skip duplicate
    SM-->>Client: idMappings (abc-123 → serverId 42)\nserverChanges + syncedAt

    Client->>LocalDB: replace clientId with serverId 42
    Client->>LocalDB: apply serverChanges (new data from other devices)

    Client->>Timer: POST /api/v1/pomodoro/sync
    Note over Timer: clientId already exists — returns same mapping
    Timer-->>Client: sessionIdMappings (ghi-789 → serverId 17)\nserverSessions + syncedAt

    Client->>LocalDB: replace clientId with serverId 17
    Client->>LocalDB: update sync_metadata.lastSyncedAt

    Note over Client,Timer: Next sync sends only changes after lastSyncedAt
```

---

## 9. PS_kmp - KMP Client

```mermaid
graph TD
    subgraph Screens
        Auth[Auth Screen\nemail identify]
        SelfManager[SelfManager Screen\ndaily dashboard - AI prioritize]
        TaskScreen[Task Screen\nCRUD - NLP parsing]
        HabitScreen[Habit Screen\nCRUD - NLP parsing - AI coaching]
        RoutineScreen[Routine Screen\nCRUD - NLP parsing - AI optimization]
        TimerScreen[Timer Screen\nsession lifecycle - insights]
        UserPage[User Page\nanalysis - credits - reports]
    end

    subgraph NetworkClients
        ServicesClient[ServicesApiClient\nselfmanager 1226]
        TimerClient[TimerApiClient\ntimer 1227]
        AnalyticsClient[AnalyticsApiClient\nanalytics 1228]
    end

    subgraph LocalDB ["Local Database - SQLDelight"]
        TasksDB[(tasks - subtasks - tags)]
        HabitsDB[(habits)]
        RoutinesDB[(routines - steps)]
        SessionsDB[(pomodoro_sessions - intervals)]
        SyncMeta[(sync_metadata\ndevice_id - lastSyncedAt - JWT)]
    end

    subgraph BackendServices
        SM_K[psocial_selfmanager 1226]
        Timer_K[psocial_timer 1227]
        Analytics_K[psocial_analytics 1228]
    end

    Auth --> ServicesClient
    SelfManager --> ServicesClient
    SelfManager --> AnalyticsClient
    TaskScreen --> ServicesClient
    TaskScreen --> AnalyticsClient
    HabitScreen --> ServicesClient
    HabitScreen --> AnalyticsClient
    RoutineScreen --> ServicesClient
    RoutineScreen --> AnalyticsClient
    TimerScreen --> TimerClient
    UserPage --> AnalyticsClient

    ServicesClient --> TasksDB
    ServicesClient --> HabitsDB
    ServicesClient --> RoutinesDB
    ServicesClient --> SyncMeta
    TimerClient --> SessionsDB
    TimerClient --> SyncMeta

    ServicesClient -->|JWT - sync| SM_K
    TimerClient -->|JWT - sync| Timer_K
    AnalyticsClient -->|JWT| Analytics_K
```

---

## 10. LLM Analysis Pipeline — 4-Step Flow (section 2.4)

```mermaid
flowchart TD
    A([Client\nPOST /api/v1/analytics\nBearer JWT]) --> B[Analytics Service\nvalidate JWT]

    B --> C1[GET /internal/users/id/tasks\nX-Internal-Key → selfmanager]
    B --> C2[GET /internal/users/id/habits\nX-Internal-Key → selfmanager]
    B --> C3[GET /internal/users/id/routines\nX-Internal-Key → selfmanager]
    B --> C4[GET /internal/users/id/stats\nX-Internal-Key → timer]

    C1 --> D[Step 2: Build Prompt\nassemble structured JSON payload\nanalysis_type + tasks + habits + routines + pomodoro]
    C2 --> D
    C3 --> D
    C4 --> D

    D --> E[Step 3: LLM Inference\nPOST /api/v1/internal/predict\nX-Internal-Key → billing]

    E --> F{Route by model type}
    F -->|Ollama| G[llama3.2\nlocal via ngrok]
    F -->|Anthropic| H[Claude API]
    F -->|OpenAI| I[GPT API]
    F -->|sklearn| J[joblib .pkl\npredict]

    G --> K[billing: deduct credits\nINSERT prediction + transaction]
    H --> K
    I --> K
    J --> K

    K --> L[Step 4: Persist Report\nINSERT analytics_reports\nreturn insight + creditsCharged]
    L --> M([Client receives\ninsight text])
```

---

## 11. JWT Token Structure (section 2.2.6)

```mermaid
flowchart LR
    subgraph JWT Token
        H["HEADER\n─────────────\nalgorithm: HS256\ntype: JWT\n─────────────\nbase64url encoded"]
        P["PAYLOAD\n─────────────\nuserId: 30\nemail: user@example.com\niat: 1716000000\nexp: 1716086400\n─────────────\nbase64url encoded"]
        S["SIGNATURE\n─────────────\nHMAC-SHA256\nheader + payload\n+ JWT_SECRET\n─────────────\ncannot be forged"]
    end

    H -->|"."| P -->|"."| S

    subgraph Validation
        V1[1. Decode header — check algorithm]
        V2[2. Decode payload]
        V3[3. Recompute HMAC — compare signature]
        V4[4. Check exp > now]
        V5[5. Extract userId + email]
        V1 --> V2 --> V3 --> V4 --> V5
    end

    S --> V1
```

---

## 12. Production Deployment Architecture — Render + Neon + ngrok (section 3.11)

```mermaid
graph TD
    subgraph Local["Local Machine (Developer)"]
        Ollama[Ollama\nllama3.2\nlocalhost:11434]
        Ngrok[ngrok\nhttps://xxx.ngrok.io]
        Ollama --> Ngrok
    end

    subgraph Render["Render — Cloud Hosting"]
        SM[psocial-selfmanager\nKotlin / Ktor]
        Timer[psocial-timer\nKotlin / Ktor]
        Analytics[psocial-analytics\nKotlin / Ktor]
        Billing[psocial-billing\nPython / FastAPI]
        Dashboard[psocial-dashboard\nStreamlit]
    end

    subgraph Neon["Neon — Managed PostgreSQL"]
        SM_DB[(selfmanager_db)]
        Timer_DB[(timer_db)]
        Analytics_DB[(analytics_db)]
        Billing_DB[(billing_db)]
    end

    subgraph Clients["Clients"]
        KMP[KMP Mobile App\nAndroid / iOS]
        Admin[Admin Browser]
    end

    Ngrok -->|OLLAMA_BASE_URL| Billing

    SM --- SM_DB
    Timer --- Timer_DB
    Analytics --- Analytics_DB
    Billing --- Billing_DB
    Timer -->|JDBC cross-DB UserRegistry| SM_DB

    KMP -->|JWT Bearer| SM
    KMP -->|JWT Bearer| Timer
    KMP -->|JWT Bearer| Analytics
    Admin --> Dashboard
    Dashboard -->|JWT| SM
    Dashboard -->|JWT| Analytics
    Dashboard -->|X-Internal-Key| Billing
```

---

## 13. Local Development — Docker Compose Network (section 2.2.2)

```mermaid
graph TD
    subgraph compose["docker compose up — bridge network"]
        subgraph svc["Services"]
            SM[psocial_selfmanager\nhost:1226]
            Timer[psocial_timer\nhost:1227]
            Analytics[psocial_analytics\nhost:1228]
            Billing[psocial_billing\nhost:1229]
            Dashboard[psocial_dashboard\nhost:1230]
        end

        subgraph db["Databases"]
            SM_DB[(selfmanager_db\n:5432)]
            Timer_DB[(timer_db\n:5434)]
            Analytics_DB[(analytics_db\n:5435)]
            Billing_DB[(billing_db\n:5433)]
        end
    end

    SM --- SM_DB
    Timer --- Timer_DB
    Analytics --- Analytics_DB
    Billing --- Billing_DB
    Timer -.->|JDBC cross-DB\nUserRegistry| SM_DB

    Analytics -->|X-Internal-Key| SM
    Analytics -->|X-Internal-Key| Timer
    Analytics -->|X-Internal-Key| Billing
    Timer -->|X-Internal-Key fire-and-forget| SM
    Dashboard -->|JWT| SM
    Dashboard -->|X-Internal-Key| Billing
```

---

## 14. LLM vs Rule-based Baseline — Quality Scores (section 2.8)

```mermaid
xychart-beta
    title "LLM vs Rule-based Baseline — Expert Scores (1–5)"
    x-axis ["PRODUCTIVITY_SUMMARY", "HABIT_INSIGHTS", "WEEKLY_TIMER_SUMMARY"]
    y-axis "Score" 0 --> 5
    bar [1.5, 1.0, 1.5]
    bar [4.2, 3.9, 4.0]
```

*Bar 1 — Rule-based baseline. Bar 2 — LLM (Ollama llama3.2)*

---

## 15. Cold Start vs Warm Response Time (section 2.9)

```mermaid
xychart-beta
    title "Cold Start vs Warm Response Time by Service (seconds)"
    x-axis ["selfmanager (JVM)", "timer (JVM)", "analytics (JVM)", "billing (Python)"]
    y-axis "Seconds" 0 --> 60
    bar [52, 52, 52, 8]
    bar [1, 1, 1, 1]
```

*Bar 1 — cold start (after 15 min idle on Render free tier). Bar 2 — warm response*

---

## 16. Pomodoro Sessions by Entity Type — Production Data (section 2.1 / 3.6.9)

```mermaid
pie title Pomodoro Sessions by Entity Type (17 total)
    "Task-linked" : 7
    "Habit-linked" : 4
    "Routine-linked" : 4
    "Standalone" : 2
```

---

## 17. Focus Minutes by Entity Type — Production Data (section 2.1 / 3.6.9)

```mermaid
pie title Focus Minutes by Entity Type (425 min total)
    "Task-linked (175 min)" : 175
    "Habit-linked (100 min)" : 100
    "Routine-linked (100 min)" : 100
    "Standalone (50 min)" : 50
```

---

## 18. Task Completion by Priority — Production Data (section 2.1)

```mermaid
xychart-beta
    title "Test Dataset: Tasks by Priority and Completion Status"
    x-axis ["High", "Medium", "Low"]
    y-axis "Tasks" 0 --> 4
    bar [0, 2, 0]
    bar [3, 2, 1]
```

*Bar 1 — completed. Bar 2 — pending*

---

## 19. Habit Distribution by Type — Production Data (section 2.1 / 3.6.7)

```mermaid
pie title Habits by Type (6 total)
    "Start — building positive behaviour" : 5
    "Quit — breaking negative behaviour" : 1
```

---

## 20. Credit Balance Timeline — Production Data (section 2.8)

```mermaid
xychart-beta
    title "Credit Balance Over Testing Period (test@productivesocial.com)"
    x-axis ["Welcome +100", "After 20 charges", "Admin deposit +100", "After 1 charge", "Admin deposit +100", "After 16 charges", "Current"]
    y-axis "Credits" 0 --> 220
    line [100, 0, 100, 95, 195, 115, 115]
```

---

## 21. Локальная база данных KMP — ER-диаграмма (раздел 2.2.8)

```mermaid
erDiagram
    sync_metadata {
        TEXT key PK
        TEXT value
    }

    tag {
        INTEGER localId PK
        INTEGER serverId
        TEXT name
    }

    project {
        INTEGER localId PK
        INTEGER serverId
        TEXT syncId
        TEXT name
        TEXT description
        TEXT iconName
        TEXT colorHex
        TEXT priority
        INTEGER createdAt
        INTEGER updatedAt
        INTEGER isDeleted
    }

    project_tag {
        INTEGER projectLocalId FK
        INTEGER tagLocalId FK
    }

    task {
        INTEGER localId PK
        INTEGER serverId
        TEXT syncId
        INTEGER projectLocalId FK
        TEXT name
        TEXT description
        TEXT priority
        TEXT urgency
        INTEGER recurring
        INTEGER sendReminder
        INTEGER date
        TEXT target
        INTEGER completed
        INTEGER timeSpentMinutes
        INTEGER createdAt
        INTEGER updatedAt
        INTEGER isDeleted
    }

    task_time {
        INTEGER id PK
        INTEGER taskLocalId FK
        TEXT time
    }

    task_reminder {
        INTEGER id PK
        INTEGER taskLocalId FK
        TEXT option
    }

    task_tag {
        INTEGER taskLocalId FK
        INTEGER tagLocalId FK
    }

    subtask {
        INTEGER localId PK
        INTEGER serverId
        TEXT syncId
        INTEGER taskLocalId FK
        TEXT name
        INTEGER completed
        INTEGER timeSpentMinutes
        INTEGER isDeleted
    }

    habit {
        INTEGER localId PK
        INTEGER serverId
        TEXT syncId
        INTEGER projectLocalId FK
        TEXT name
        TEXT description
        TEXT habitType
        TEXT recurrency
        TEXT target
        INTEGER sendReminder
        INTEGER completed
        INTEGER timeSpentMinutes
        INTEGER createdAt
        INTEGER updatedAt
        INTEGER isDeleted
    }

    habit_time {
        INTEGER id PK
        INTEGER habitLocalId FK
        TEXT time
    }

    habit_reminder_time {
        INTEGER id PK
        INTEGER habitLocalId FK
        TEXT time
    }

    habit_tag {
        INTEGER habitLocalId FK
        INTEGER tagLocalId FK
    }

    habit_subtask {
        INTEGER localId PK
        INTEGER serverId
        TEXT syncId
        INTEGER habitLocalId FK
        TEXT name
        INTEGER completed
        INTEGER timeSpentMinutes
        INTEGER isDeleted
    }

    habit_completion {
        INTEGER id PK
        INTEGER habitLocalId FK
        TEXT completedDate
        INTEGER serverCompletionId
        TEXT syncId
        INTEGER deleted
    }

    routine {
        INTEGER localId PK
        INTEGER serverId
        TEXT syncId
        INTEGER projectLocalId FK
        TEXT name
        TEXT description
        TEXT recurrency
        TEXT target
        INTEGER sendReminder
        INTEGER completed
        INTEGER createdAt
        INTEGER updatedAt
        INTEGER isDeleted
    }

    routine_time {
        INTEGER id PK
        INTEGER routineLocalId FK
        TEXT time
    }

    routine_off_time {
        INTEGER id PK
        INTEGER routineLocalId FK
        TEXT time
    }

    routine_reminder_time {
        INTEGER id PK
        INTEGER routineLocalId FK
        TEXT time
    }

    routine_tag {
        INTEGER routineLocalId FK
        INTEGER tagLocalId FK
    }

    routine_step {
        INTEGER localId PK
        INTEGER serverId
        TEXT syncId
        INTEGER routineLocalId FK
        TEXT name
        TEXT description
        INTEGER autoStart
        INTEGER durationMinutes
        INTEGER position
        INTEGER completed
        INTEGER isDeleted
    }

    routine_completion {
        INTEGER localId PK
        INTEGER routineLocalId FK
        TEXT completedDate
        TEXT syncId
    }

    pomodoro_settings {
        INTEGER localId PK
        INTEGER serverId
        INTEGER workDurationMinutes
        INTEGER shortBreakMinutes
        INTEGER longBreakMinutes
        INTEGER cyclesUntilLongBreak
        INTEGER autoStartBreaks
        INTEGER autoStartSession
        INTEGER focusModeEnabled
        INTEGER soundEnabled
        INTEGER notificationsEnabled
    }

    pomodoro_session {
        INTEGER localId PK
        INTEGER serverId
        TEXT syncId
        INTEGER userId
        TEXT entityType
        INTEGER entityLocalId
        INTEGER entityServerId
        TEXT status
        INTEGER workDurationMinutes
        INTEGER shortBreakMinutes
        INTEGER longBreakMinutes
        INTEGER cyclesUntilLongBreak
        INTEGER completedCycles
        INTEGER totalWorkMinutes
        INTEGER startedAt
        INTEGER completedAt
    }

    pomodoro_interval {
        INTEGER localId PK
        INTEGER serverId
        TEXT syncId
        INTEGER sessionLocalId FK
        TEXT type
        INTEGER plannedDurationMinutes
        INTEGER startedAt
        INTEGER endedAt
        INTEGER completed
    }

    project ||--o{ task : "содержит"
    project ||--o{ habit : "содержит"
    project ||--o{ routine : "содержит"
    project ||--o{ project_tag : "тегируется"
    tag ||--o{ project_tag : ""
    tag ||--o{ task_tag : ""
    tag ||--o{ habit_tag : ""
    tag ||--o{ routine_tag : ""
    task ||--o{ task_time : "расписание"
    task ||--o{ task_reminder : "напоминание"
    task ||--o{ task_tag : "тегируется"
    task ||--o{ subtask : "имеет"
    habit ||--o{ habit_time : "расписание"
    habit ||--o{ habit_reminder_time : "напоминание"
    habit ||--o{ habit_tag : "тегируется"
    habit ||--o{ habit_subtask : "имеет"
    habit ||--o{ habit_completion : "выполнено в дату"
    routine ||--o{ routine_time : "расписание"
    routine ||--o{ routine_off_time : "время завершения"
    routine ||--o{ routine_reminder_time : "напоминание"
    routine ||--o{ routine_tag : "тегируется"
    routine ||--o{ routine_step : "имеет шаги"
    routine ||--o{ routine_completion : "выполнено в дату"
    pomodoro_session ||--o{ pomodoro_interval : "содержит"
```

**Рисунок 2.X. ER-диаграмма локальной SQLite базы данных клиентского приложения (25 таблиц).**

---

## 22. Ollama + ngrok: маршрут вывода LLM (разделы 2.2.10–2.2.11)

```mermaid
flowchart TD
    User([Пользователь\nзапускает анализ])

    subgraph KMP ["Мобильное приложение (KMP)"]
        App[AnalyticsScreen]
    end

    subgraph Render ["Render (облако)"]
        Billing["psocial_billing\nFastAPI · порт 1229"]
    end

    subgraph ngrok_cloud ["Инфраструктура ngrok"]
        NgrokEdge["ngrok Edge\nhttps://xxxx.ngrok-free.app"]
    end

    subgraph Dev ["Машина разработчика (localhost)"]
        NgrokAgent["ngrok agent\n→ localhost:11434"]
        Ollama["Ollama\nlocalhost:11434"]
        Model["llama3.2 (3B)\nлокальная модель"]
    end

    User --> App
    App -->|"POST /api/v1/analytics/analyze"| Billing

    Billing -->|"OLLAMA_BASE_URL\nngrok-skip-browser-warning\nследование перенаправлениям"| NgrokEdge
    NgrokEdge -->|"зашифрованный туннель"| NgrokAgent
    NgrokAgent -->|"localhost:11434\n/api/generate"| Ollama
    Ollama --> Model
    Model -->|"ответ LLM"| Ollama
    Ollama -->|"JSON stream"| NgrokAgent
    NgrokAgent --> NgrokEdge
    NgrokEdge -->|"HTTP ответ"| Billing
    Billing -->|"аналитический отчёт"| App
    App --> User

    style Dev fill:#f0f4ff,stroke:#4a6cf7
    style Render fill:#fff4e6,stroke:#f59e0b
    style ngrok_cloud fill:#f0fff4,stroke:#10b981
    style KMP fill:#fdf4ff,stroke:#a855f7
```

**Рисунок 2.X. Маршрут запроса на вывод LLM: от пользователя через облако до локального Ollama через туннель ngrok.**

---

## 23. NLP-разбор пользовательского ввода — поток (раздел 2.6)

```mermaid
flowchart TD
    A([Пользователь нажимает\n«Ввод на естественном языке»])
    B["checkModelStatus()\n→ аналитическая служба"]

    B --> C{Модель доступна?}
    C -- "null" --> D["Отображается «Checking…»"]
    D --> B
    C -- "false" --> E["Отображается «AI model is unavailable»\n+ кнопка «Retry»"]
    E -- "Retry" --> B
    C -- "true" --> F["Отображается текстовое поле\n+ кнопка «Generate»"]

    A --> B

    F --> G([Пользователь вводит\nпроизвольный текст])
    G --> H["Запрос отправляется в\nAnalyticsApiClient\nparseTask() / parseHabit() / parseRoutine()"]

    H --> I["Аналитическая служба\n→ LLM-вывод\n→ структурированный JSON"]

    I --> J{Тип сущности}

    J -- "Задача / Привычка" --> K["Карточка предпросмотра:\nназвание, дата, время, приоритет,\nнапоминание, теги, проект, подзадачи"]
    J -- "Рутин" --> L["Карточка предпросмотра:\nназвание, шаги с длительностью\nи флагом autoStart, рекуррентность"]

    K --> M{Пользователь проверяет}
    L --> M

    M -- "Применить" --> N["Данные переносятся в форму создания\n→ пользователь сохраняет"]
    M -- "Отменить" --> G

    N --> O([Сущность сохранена\nв локальной БД и синхронизирована])

    style D fill:#fff3cd,stroke:#f59e0b
    style E fill:#fee2e2,stroke:#ef4444
    style F fill:#dcfce7,stroke:#22c55e
    style O fill:#dbeafe,stroke:#3b82f6
```

**Рисунок 2.X. Поток NLP-разбора пользовательского ввода при создании задачи, привычки или рутина.**
