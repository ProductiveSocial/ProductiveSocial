# psocial_analytics — How It Works

## Authentication & Authorization

The analytics service accepts JWT tokens issued by selfmanager without any additional login step — they share the same JWT secret, so a token from selfmanager is valid here immediately. The user's integer `userId` is extracted from the JWT claims on every request and used to identify them across all downstream calls.

Admin access is controlled entirely by the `ADMIN_EMAILS` environment variable — a comma-separated list of email addresses. There is no admin flag in the database. This means granting or revoking admin access is a configuration change, not a database update, and takes effect immediately on the next request.

Service-to-service calls (from other backend services calling analytics) are authenticated via the `X-Internal-Key` header.

---

## Running an Analysis

When a user calls `POST /api/v1/analytics` with an analysis type and model ID, the service orchestrates a multi-step pipeline:

First it pulls the user's productivity data from selfmanager — tasks, habits, and routines — via internal key-authenticated calls. Then it fetches the user's pomodoro stats from the timer service. If either of these upstream calls fails, the service continues with whatever data it has — selfmanager and timer failures are non-blocking and result in those sections being omitted from the prompt rather than failing the whole request.

With the data in hand, the service builds a structured prompt tailored to the requested analysis type and sends it to the billing service's internal predict endpoint along with the user's selfmanager ID. Billing handles the credit check, LLM inference, and credit deduction. If this call fails, the error is surfaced directly to the user — there is no silent fallback when the actual analysis can't run.

On success, the report (analysis type, model used, insight text, credits charged, timestamp) is persisted locally in the analytics database, and the result is returned to the user.

`GET /api/v1/analytics` returns all stored reports for the authenticated user, ordered newest first.

---

## Analysis Types

Each analysis type shapes the prompt in a specific way:

| Type | What it focuses on |
|------|-------------------|
| `PRODUCTIVITY_SUMMARY` | A cross-service overview combining tasks, habits, routines, and focus sessions into a holistic picture |
| `TASK_PRIORITIZATION` | Builds a prioritised focus list from the user's pending tasks and how much time they've already spent on each |
| `HABIT_INSIGHTS` | Looks at consistency across habits, identifies which are at risk of breaking, and surfaces the single highest-impact change |
| `ROUTINE_OPTIMIZATION` | Reviews the user's routines for efficiency and suggests concrete improvements to their structure or ordering |
| `WEEKLY_TIMER_SUMMARY` | A weekly Pomodoro recap covering focus time, completion rate, peak hours and days, and a suggestion for next week |
| `CUSTOM` | A free-form prompt supplied directly by the user — the model receives it as-is alongside the user's data as context |

---

## Credits (Billing Proxy)

Analytics is the only way clients interact with the billing service. Every credit operation passes through here — clients are never given direct access to billing.

When a credit call comes in, the service extracts the authenticated user's `userId` from the JWT and forwards it to billing as the `selfmanager_user_id`. From the client's perspective, these endpoints behave like any other API call; the billing hop is invisible:

- `GET /api/v1/credits/balance` — returns the user's current credit balance
- `GET /api/v1/credits/transactions?limit=50&offset=0` — returns paginated transaction history (deposits, charges, refunds) with a running `credits_balance` field

Users cannot deposit credits themselves. Deposits are admin-only operations performed via the admin dashboard, which calls billing directly with an internal key.

---

## Admin

Admin endpoints let authorized users inspect and manage the system across all users:

- `GET /api/v1/admin/stats` returns system-wide totals — number of users, total reports generated, total credits charged, and a breakdown of report counts by analysis type
- `GET /api/v1/admin/users` lists all users who have ever used the analytics service, with their report counts
- `GET /api/v1/admin/reports` returns every report across all users
- `GET /api/v1/admin/users/{userId}/reports` returns all reports for a specific user
- `DELETE /api/v1/admin/users/{userId}/reports` bulk-deletes all reports for a user
- `PATCH /api/v1/admin/users/{userId}/toggle-admin` flips the admin flag on the user's local analytics record
- `POST /api/v1/admin/seed-reports` inserts sample reports for a user, bypassing billing entirely — useful for testing without spending credits

---

## Internal Endpoint

`GET /internal/users/{userId}/stats` is available for service-to-service use (X-Internal-Key required). It returns the user's total report count and the timestamp of their most recent report — a lightweight call designed for other services that need to know whether a user has used analytics without pulling full report data.

---

## External Service Clients

The analytics service maintains three HTTP clients for communicating with upstream services:

**SelfmanagerClient** calls `GET /internal/users/{id}/tasks`, `/habits`, and `/routines` using the internal key. If selfmanager is unavailable or returns an error, the client returns empty lists and the analysis continues without that data.

**TimerClient** calls `GET /internal/users/{id}/stats` using the internal key. If the timer is unavailable, the client returns null and the pomodoro section is simply omitted from the prompt.

**BillingClient** calls `POST /api/v1/internal/predict` using the internal key. It uses a 2-minute timeout to accommodate LLM cold starts (particularly relevant for Ollama on Render's free tier). If billing fails, the error propagates to the user — a failed inference is not silently swallowed.

---

## Data Model

The analytics database stores two entity types:

**User** — a local mirror of the selfmanager user record. Stores the selfmanager integer user ID, the user's email, and a local admin flag. This record is created the first time a user calls any analytics endpoint.

**AnalyticsReport** — one record per completed analysis. Stores the analysis type, the billing model ID used, the full insight text returned by the LLM, the number of credits charged, and the creation timestamp.
