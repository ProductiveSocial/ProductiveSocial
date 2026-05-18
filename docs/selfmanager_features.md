# psocial_selfmanager — How It Works

## Authentication

Users sign in using only their email address — there are no passwords. When a user submits their email to `POST /api/v1/auth/identify`, the service either creates their account or retrieves the existing one, then issues a JWT access token and a refresh token. The JWT contains the user's integer ID and email as claims, and is valid for 24 hours. The refresh token lasts 30 days and can be rotated via `POST /api/v1/auth/refresh`, which invalidates the old token and issues a new pair.

Logout works at two levels: `POST /api/v1/auth/logout` revokes only the token for the current device, while `POST /api/v1/auth/logout-all` revokes every refresh token the user has, forcing a fresh login on all devices.

---

## Projects

Every user gets a "Default" project created automatically on first login. Projects are containers for tasks, habits, and routines — they carry a name, description, icon, color, priority, and tags. Deleting a project cascades to everything inside it, so removing a project removes all its tasks, habits, and routines at once.

---

## Tasks

Tasks are the primary unit of work. Each task has a name, description, priority level, target, due date, recurring flag, and optional reminders. Tasks support **subtasks** — each subtask tracks its own name, completed state, and time spent in minutes. Tasks also carry **scheduled times** (a list of epoch timestamps for when the task is planned), a many-to-many **tags** relationship (tags are auto-created on reference), and a **completion log** that keeps a timestamped record of every time the task was marked done.

Time tracking accumulates automatically: every time a pomodoro session linked to this task completes, the timer service calls back to selfmanager and adds the work minutes to the task's running total.

---

## Habits

Habits come in two forms: `Start` habits (building a new behavior) and `Quit` habits (breaking an existing one). Like tasks, habits support subtasks with their own time tracking. What makes habits distinct from tasks is that they maintain two independent time lists — **scheduled times** (when the habit is planned) and **reminder times** (when the user gets notified) — and each completion is linked to a specific scheduled time slot. The completion log records not just that the habit was done, but which subtasks were completed within each instance, giving step-level granularity.

---

## Routines

Routines are structured sequences of steps meant to be completed in order. Each step has a name, duration, description, position index, and an auto-start flag that tells the client whether to advance to the next step automatically. Routines share the same scheduled-times and reminder-times structure as habits, and their completion log captures step-level detail — which steps were done and in what order within each routine completion.

---

## Offline Sync

The sync system is designed so that clients can work entirely offline and push changes when connectivity returns. A single `POST /api/v1/sync` call batches all creates, updates, and deletes across projects, tasks, habits, routines, and habit completions in one request.

Clients assign their own UUIDs (`syncId`) to every entity before sending, which makes the operation idempotent — if the same sync request is sent twice, the second call is a no-op. Cross-references work within a single batch, so a new task can reference a new project in the same request and the server resolves them in the right order.

The server responds with `idMappings` (client UUID → server-assigned integer ID), `serverChanges` (what changed on the server since the client's last sync), a per-item `errors` array (so one bad record doesn't block everything else), and a new `syncedAt` timestamp for the client to store. Deleted entities are tracked in a tombstone table so clients know to purge them from their local database on the next sync.

---

## Internal Endpoints

selfmanager exposes two categories of internal endpoints, both protected by `X-Internal-Key`:

The **time-log endpoint** (`POST /internal/time-log`) is called by the timer service after a pomodoro session completes. It receives the entity type, entity ID, minutes spent, and an optional subtask ID, and adds those minutes to the entity's cumulative total. This call is fire-and-forget from the timer's side — it doesn't wait for a response.

The **data pull endpoints** (`GET /internal/users/{id}/tasks|habits|routines`) are called by the analytics service when building a prompt for LLM analysis. They return the user's full structured data so analytics can describe the user's work life to the model.

---

## User Registry

selfmanager owns the canonical `users` table. The timer service connects to this same table directly via a raw JDBC connection — not through the selfmanager HTTP API. This means when a user logs in via the timer for the first time, their account is created in (or matched against) selfmanager's own `users` table, guaranteeing that the same email always resolves to the same integer `user_id` across the whole system, regardless of which service the user first signed into.

---

## Standalone Usage

selfmanager is fully self-contained. Users can manage tasks, habits, routines, and projects without ever using the timer or analytics services. Pomodoro time tracking enriches entities with time-spent data, but it is entirely optional — the service functions completely on its own.
