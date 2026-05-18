# psocial_timer — How It Works

## Authentication

The timer uses the same email-only find-or-create pattern as selfmanager. Submitting an email to `POST /api/v1/auth/identify` either creates the account or retrieves it, then issues a JWT. Because the timer and selfmanager share the same JWT secret and issuer, a token issued by selfmanager is accepted directly by the timer — one login works on both services without any token exchange.

---

## Pomodoro Settings

Every user has one settings record that governs how their sessions behave. It stores the work interval duration, short break duration, long break duration, and how many work cycles must complete before a long break is triggered. It also carries a set of preference flags: whether breaks auto-start, whether sessions auto-start, whether focus mode is enabled, and sound and notification preferences.

When a session is created, the current settings are **snapshotted** into that session record. This means that if the user later changes their settings, historical session data reflects what their settings actually were at the time — past sessions are never retroactively altered.

Once a user has completed at least 5 work intervals, 3 short breaks, and 3 long breaks, the settings response also includes a **recommended durations** block — personalized work, short break, and long break durations calculated from the actual average duration of each interval type across their real session history.

---

## Session Lifecycle

A session is created with an optional link to a Task, Habit, or Routine from selfmanager, or it can run standalone. Creating a session automatically starts the first Work interval.

From there the session progresses through intervals:

**Starting an interval** advances to the next one. The cycle logic works like this: after each Work interval, the service checks how many work intervals have been completed in the current cycle. If that count is a multiple of `cyclesUntilLongBreak`, the next interval is a Long Break; otherwise it's a Short Break.

**Completing an interval** marks it done. If it was a Work interval, its duration is added to `totalWorkMinutes` and, when a full cycle is reached, `completedCycles` increments. Completing a Work interval also returns a **break suggestion** — the service looks at how much focus time the user has accumulated today and what position they're at in their cycle, then recommends one of three things: an Extended Rest (20 minutes, triggered when the user has accumulated 90+ minutes of focus today with no long break in the current session), a Long Break (if the cycle count dictates it), or the default Short Break.

**Pausing** a session increments its `pauseCount` and returns an **abandonment risk signal** — see below. **Resuming** simply moves the session back to active.

**Completing** a session marks it as done. If the session was linked to an entity and had accumulated work minutes, the timer fires a call to selfmanager's internal time-log endpoint to update that entity's `timeSpentMinutes`. This call is fire-and-forget — the session completes normally even if selfmanager is unreachable.

**Abandoning** a session marks it as abandoned without logging time.

---

## Abandonment Risk Detection

Every time a session is paused, the service evaluates whether the session is at risk of being abandoned and returns a nullable `AbandonmentRisk` signal with a message and reason.

Two independent signals can trigger this:

The **pause threshold** signal fires when the current `pauseCount` reaches or exceeds the user's personal threshold. If the user has at least 3 previously abandoned sessions, their threshold is computed as the average number of pauses those sessions had at the point of abandonment. Otherwise a static threshold of 3 is used.

The **stale session** signal fires when the session has been open for more than three times its planned total work duration (work interval × cycles until long break) and has zero completed cycles. A session that's been sitting untouched far longer than planned with nothing to show for it is at high abandonment risk.

---

## Focus Pattern Insights

`GET /api/v1/pomodoro/insights/focus-patterns` aggregates the user's completed Work intervals and groups them by hour of day (0–23, UTC) and by day of week. Each bucket returns the interval count and total work minutes for that slot. The response also surfaces `peakHour` and `peakDayOfWeek` — the hour and day where the user has completed the most focus intervals — giving a clear picture of when they do their best work.

---

## Entity Stats

`GET /api/v1/pomodoro/insights/entity-stats` breaks down session performance by entity type: Task, Habit, Routine, and Standalone. For each type it reports total sessions, completed sessions, abandoned sessions, completion rate, average work minutes per session, and average completed cycles per session. The service also identifies the `strongestEntityType` (highest completion rate) and `weakestEntityType` (lowest), but only for types with at least 3 terminal sessions — types with too little data are excluded to avoid misleading conclusions.

---

## Offline Sync

`POST /api/v1/pomodoro/sync` allows KMP clients to batch sessions and intervals created while offline and push them in a single request. Each session and interval carries a stable client UUID (`clientId`) — if the same sync request is retried, the server recognizes the IDs and skips duplicates rather than creating them twice.

The response contains everything the client needs to reconcile: `sessionIdMappings` maps each client UUID to its server-assigned ID, `serverSessions` contains sessions that changed on the server since the client's `lastSyncedAt` (for multi-device sync), `errors` lists any per-item failures (one bad item doesn't block the rest), and `syncedAt` is the timestamp the client should store for its next sync call.

If any synced sessions that completed are linked to entities and have work minutes, time-log calls fire to selfmanager after the database commit.

---

## Internal Endpoint

`GET /internal/users/{userId}/stats` is called by the analytics service when building an LLM prompt. It returns aggregated pomodoro data for the user: total session count, completed session count, total work minutes, total cycles, average work minutes per session, and per-entity-type breakdowns of session counts and work minutes. All internal endpoints require the `X-Internal-Key` header.

---

## Standalone Usage

The timer is fully self-contained. Entity linking (to selfmanager Tasks, Habits, or Routines) is optional on every session. Users can track focus sessions, view insights, and sync across devices without ever touching selfmanager. The time-log calls back to selfmanager only happen when a session is linked to an entity — standalone sessions complete entirely within the timer.

---

## User Registry

Rather than duplicating user records, the timer connects directly to selfmanager's `users` table via a raw JDBC connection (HikariCP pool). When a user logs into the timer for the first time, the find-or-create runs against the same table selfmanager owns, ensuring the same email always resolves to the same integer `user_id` across both services. This connection works even if the selfmanager HTTP service is down, since it bypasses the API entirely and talks directly to the database.
