# PS_kmp — How It Works

## Overview

PS_kmp is the Kotlin Multiplatform mobile (and desktop) client for ProductiveSocial. It targets Android, iOS, and JVM Desktop from a single shared codebase using Compose Multiplatform for UI. The app is offline-first: all user data is written to a local SQLite database via SQLDelight before any server communication, so the app is fully functional without a network connection.

---

## Authentication

Users sign in with only their email address — no password. The app calls `POST /api/v1/auth/identify` on the selfmanager service with the email, and the server either creates a new account or returns the existing one, along with a JWT access token.

The token and user ID are stored in the local `sync_metadata` table so they survive app restarts. On launch, `UserSessionManager.init()` reads the stored token and restores the session silently — the user is never asked to sign in again unless they explicitly log out. Logout calls `UserSessionManager.logout()`, which wipes all locally stored user data including the token, effectively resetting the app to a clean state.

---

## Projects

Projects are containers that group tasks, habits, and routines. Each project carries a name, description, color (hex), icon name, priority level, and tags. The user can browse projects in a grid view and drill into a project to see all its nested entities.

On first sync, every user has at least a "Default" project provided by the server. Projects can be created locally and synced to the server, and the client maintains a `serverId` alongside the local integer ID so it knows which server entity a local row maps to after sync.

---

## Tasks

Tasks are the primary unit of work. Each task has a name, description, priority (None / Low / Medium / High), urgency, optional due date, recurring flag, and a target. Tasks support multiple **scheduled times** per day (stored as `HH:mm` strings) and multiple **reminder options** (At time, 5 / 10 / 15 / 30 minutes before).

Tasks can have **subtasks**, each with its own name and completed state. The task creation and editing screen manages subtask entries inline — add, rename, and remove subtasks from the same form.

**Tags** are user-scoped string labels. A task can carry multiple tags, and the same tags can be applied across tasks, habits, routines, and projects. Tags are created on reference and stored in a shared `tag` table.

On the self-manager screen, the task list is filtered by selected date and can be sorted by date or priority, and grouped by none, tag, project, or urgency. Completing a task records a timestamped completion entry in the local `completion_log` table, which is pushed to the server on the next sync.

---

## Habits

Habits come in two types: `Start` (building a new behavior) and `Quit` (breaking one). Like tasks, habits have a name, description, project, priority, tags, reminders, and subtasks. What distinguishes habits is that they track **daily completions** — each day the habit is due, the user marks it done, and that completion is recorded in the `habit_completion` table with the completion date.

Habits also have independent **scheduled times** (when the habit is planned to run) and **reminder times** (when the user is notified to do it). Both are stored as separate time lists on the habit.

On the self-manager screen, habits for the currently selected date are shown alongside tasks and routines. Marking a habit complete creates a local completion record with today's date, which syncs to the server and gets a server-side `serverId` mapped back.

---

## Routines

Routines are ordered sequences of steps meant to be completed in a fixed order. Each step has a name, duration in minutes, a position index (for ordering), and an `autoStart` flag that tells the client whether to automatically advance to the next step when the current one finishes.

Routines follow the same recurrency, scheduled-times, and reminder-times pattern as habits. They also support **off-times** — specific days or time windows when the routine should not run — stored in a `routine_off_time` table.

Like tasks and habits, routines appear in the self-manager list for the selected date and can be linked to pomodoro sessions.

---

## Pomodoro Timer

The pomodoro module implements a configurable focus timer with a full session lifecycle managed both locally and server-synced.

**Settings** control the timer behavior: work interval duration, short break duration, long break duration, number of cycles before a long break, and toggles for auto-starting breaks, auto-starting the next session, sound, and notifications. Settings are stored in the local `pomodoro_settings` table and synced to the timer service via `PUT /api/v1/pomodoro/settings`.

Each **session** is a single focus block. Sessions can be standalone or linked to a Task, Habit, or Routine — the link is stored as `entityType` + `entityLocalId`. Within a session, individual work and break blocks are tracked as **intervals** in the `pomodoro_interval` table, each with a type (Work / ShortBreak / LongBreak), duration, and start/end timestamps.

The `PomoViewModel` owns the running timer state: time remaining, whether a break is active, total sessions completed today, and total focused minutes. When an interval ends, the ViewModel decides the next interval type based on cycle count and the user's settings, then auto-starts it if the auto-start flag is set.

The `FocusViewModel` extends the timer for sessions linked to a specific entity. It surfaces the entity's subtasks so the user can check them off during the focus session, and it tracks a list of "completion items" — what was accomplished — that can be reviewed when ending the session. After a session, a task picker lets the user immediately chain into the next task.

Session state transitions are: Active → Paused → Active (resume), Active → Completed, or Active/Paused → Abandoned. All transitions update the local row and are pushed to the timer service on the next sync via `POST /api/v1/pomodoro/sync`.

---

## Self-Manager Screen

The self-manager screen is the main daily view. It shows a horizontal date strip at the top; tapping a date loads the tasks, habits, and routines scheduled for that day. The list supports live sorting (by date or priority) and grouping (by none, tag, project, or urgency) controlled by a sort/group picker in the header.

The `SelfManagerViewModel` combines streams from the Task, Habit, and Routine repositories and re-emits sorted and grouped lists reactively whenever the selected date or a repository changes.

---

## Journal

The journal module has a screen in place that shows entries with date, time, mood, and content. The data layer and server integration are not yet wired — this screen currently displays placeholder structure. Full journal functionality is planned for a future phase.

---

## Analytics Insights

An `AnalyticsApiClient` is set up to call `POST /api/v1/analytics` on the analytics service with an analysis type and optional model ID. The supported types are `PRODUCTIVITY_SUMMARY`, `TASK_PRIORITIZATION`, `HABIT_INSIGHTS`, `ROUTINE_OPTIMIZATION`, and `CUSTOM`. The `InsightsViewModel` and `UserPageScreen` provide the integration point; full display of generated reports is in progress.

---

## Offline Sync

The sync layer is the backbone of the offline-first architecture. Every entity (project, task, habit, routine, habit completion, pomodoro session) is assigned a client-generated UUID (`syncId`) at creation time. This UUID is the idempotency key — if the same entity is synced twice, the server ignores the duplicate.

A bidirectional sync call (`POST /api/v1/sync`) batches all pending creates, updates, and deletes in a single request and returns:
- `idMappings` — client UUID → server-assigned integer ID, so the client can update its local `serverId` columns.
- `serverChanges` — entities changed on the server since the client's last `syncedAt` timestamp (e.g., changes made from another device).
- `errors` — per-item failures, so one bad record doesn't block the rest.
- A new `syncedAt` timestamp to store for the next call.

Pomodoro sessions sync separately through the timer service's own sync endpoint.

The `sync_metadata` table acts as the sync state store: it holds the device ID, the stored server user ID, `lastSyncedAt`, and `lastTimerSyncedAt` as key-value rows.

---

## Navigation & Routing

Navigation is handled by a type-safe `Routes` sealed interface where each screen is a serializable object or data class. The `NavHost` maps routes to composables. Top-level destinations — Self-Manager, Pomodoro, Journal, User Page — are accessible from the bottom navigation bar on mobile and a side navigation rail on desktop.

Deeper routes cover: Project creation and details, Task creation and details, Habit creation and editing and details, Routine creation and details, Pomodoro execution linked to an entity, Pomodoro settings, Pomodoro task picker, and Pomodoro statistics.

---

## Responsive Layout

The app adapts to screen size using a `WindowSize` utility. On compact screens (phones) it uses a bottom navigation bar. On medium and expanded screens (tablets, desktop) it switches to a side navigation rail. The `NavScaffold` composes the appropriate navigation component with the content area.

---

## Dependency Injection

Koin is used for dependency injection across all platforms. `AppModule` registers all repositories, view models, and API clients as singletons or factories. Platform-specific modules provide the SQLDelight driver and Ktor engine appropriate for each target.

---

## Platform Details

The app runs on three platforms from a single shared `commonMain` source set:

**Android** — uses `AndroidSqliteDriver` for SQLDelight and the OkHttp engine for Ktor. The base URL points to `10.0.2.2` (emulator localhost bridge) for local development. Entry point is `MainActivity` with a `PSocialApplication` for Koin initialization.

**iOS** — uses the native SQLite driver and Darwin engine for Ktor. The base URL points to `localhost`. Entry point is `MainViewController` called from the Swift `iosApp` target.

**JVM Desktop** — uses `JdbcSqliteDriver` and the Java engine for Ktor. The base URL points to `localhost`. Entry point is `main.kt` in `jvmMain`.

Service ports in local development: selfmanager on `8080`, timer on `8081`, analytics on `8082`.
