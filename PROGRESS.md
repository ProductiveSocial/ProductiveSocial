# ProductiveSocial — Progress Tracker

## Status Legend
- ✅ Done
- 🚧 In progress
- 📋 Planned

---

## Backend Services

### psocial_selfmanager (Kotlin/Ktor :1226)
- ✅ Email-only auth (find-or-create) with JWT + refresh tokens
- ✅ Projects, Tasks, Habits, Routines — full CRUD
- ✅ Habit completion logging
- ✅ Subtasks and tags on tasks/habits/routines
- ✅ Paginated list endpoints
- ✅ Offline sync endpoint (`POST /api/v1/sync`)
- ✅ Internal endpoints for analytics (tasks, habits, routines by user ID)
- ✅ UserRegistry — shared user identity via raw JDBC (device_id + created_at fixed)
- ✅ Dockerised and running locally
- 📋 Deploy updated image to Render (UserRegistry `created_at` fix)

### psocial_timer (Kotlin/Ktor :1227)
- ✅ Pomodoro session lifecycle (create → start interval → complete → finish/abandon)
- ✅ Work / ShortBreak / LongBreak interval sequencing
- ✅ Sessions linked to Task, Habit, Routine, or standalone
- ✅ Pause / resume with pause count tracking
- ✅ Smart duration recommendations (based on completed interval history)
- ✅ Focus pattern insights (by hour-of-day and day-of-week)
- ✅ Abandonment risk signal on pause
- ✅ Adaptive break suggestions on work interval completion
- ✅ Entity-level stats (completion rate, avg work minutes by entity type)
- ✅ Offline sync endpoint
- ✅ Internal endpoint for analytics (aggregated user stats)
- ✅ Dockerised and running locally

### psocial_analytics (Kotlin/Ktor :1228)
- ✅ Pulls tasks, habits, routines from selfmanager (internal key)
- ✅ Pulls pomodoro stats from timer (internal key)
- ✅ Sends structured data to billing for LLM inference
- ✅ Persists analysis reports per user
- ✅ 6 analysis types: PRODUCTIVITY_SUMMARY, TASK_PRIORITIZATION, HABIT_INSIGHTS, ROUTINE_OPTIMIZATION, WEEKLY_TIMER_SUMMARY, CUSTOM
- ✅ Admin routes gated by ADMIN_EMAILS env var
- ✅ Dockerised and running locally
- ✅ Fixed: stale Docker image rebuilt from correct analytics JAR

### psocial_billing (Python/FastAPI :1229)
- ✅ Email-only auth with JWT + refresh tokens
- ✅ Credit balance per user (100 credits on registration)
- ✅ Deposit and transaction history
- ✅ ML model registry (LLM + sklearn .pkl)
- ✅ LLM inference via Ollama (default), Anthropic Claude, OpenAI GPT-4o
- ✅ Credit deduction per prediction with full audit log
- ✅ Internal predict endpoint (called by analytics with X-Internal-Key)
- ✅ Internal balance check endpoint
- ✅ Admin: user list, credit top-up, toggle active, all predictions, system stats
- ✅ Link billing account to selfmanager user ID
- ✅ Dockerised and running locally
- ✅ Fixed: LLM prompt now includes full structured user data (tasks/habits/routines/pomodoro)
- ✅ Switched default LLM provider to Ollama (llama3.2 via host.docker.internal)

---

## Dashboards

### psocial_dashboard — Admin (Streamlit :1230)
- ✅ Email login with admin guard (is_admin check)
- ✅ Overview: system-wide billing stats
- ✅ Users: list, activate/deactivate, top up credits
- ✅ Models: register and delete ML/LLM models
- ✅ Predictions: view all predictions system-wide
- ✅ Analytics: view all AI reports, filter by type and user, delete user reports
- ✅ Dockerised and running locally

### psocial_user_dashboard — User (Streamlit :1231)
- ✅ Email login (any registered user)
- ✅ Productivity page: tasks, habits, routines
- ✅ Focus page: session history, insights (focus patterns, entity stats, recommendations)
- ✅ Analysis page: trigger analysis, view previous reports
- ✅ Credits page: balance, deposit, transaction history
- ✅ Dual-token auth (selfmanager JWT + billing JWT)
- ✅ selfmanager user ID linked to billing account on login
- ✅ Dockerised and running locally

---

## Infrastructure

- ✅ Docker Compose: all 10 containers on shared `psocial_network`
- ✅ Inter-service URLs use Docker network names (not Render URLs)
- ✅ Per-service PostgreSQL databases with healthchecks
- ✅ Ollama reachable from containers via `host.docker.internal`
- ✅ `docker-compose.yml` tracked in this repo
- ✅ All services deployed on Render (free tier, cold start ~50s)
- 📋 Re-deploy selfmanager on Render with UserRegistry fix

---

## KMP Mobile App (ProductiveSocial)

- ✅ Project scaffolded (Kotlin Multiplatform, Compose Multiplatform)
- ✅ Android + iOS + Desktop targets
- ✅ SQLDelight offline-first local DB
- 🚧 Auth screens
- 📋 Tasks / Habits / Routines screens
- 📋 Pomodoro timer UI
- 📋 Offline sync with selfmanager
- 📋 Analytics / insights screens

---

## Test Data (local)

User: `test@productivesocial.com`
- ✅ 3 projects (Work, Personal, Learning)
- ✅ 8 tasks across all projects
- ✅ 6 habits (5 daily, 1 weekly)
- ✅ 4 routines with steps (Morning, Deep Work, Evening Learning, Weekly Review)
- ✅ 32 pomodoro sessions (31 completed) linked to tasks, habits, routines, and standalone
- ✅ 55 habit completions across 14 days of history
- ✅ AI analysis tested end-to-end locally (PRODUCTIVITY_SUMMARY via Ollama llama3.2)
- ✅ AI analysis tested end-to-end on Render (via ngrok tunnel to local Ollama)

---

## Known Issues / Next Steps

- 📋 Render selfmanager re-deploy (UserRegistry created_at fix — current prod fails on first login for new users)
- ✅ AI analysis working end-to-end on Render via ngrok → local Ollama
- 📋 Add ANTHROPIC_API_KEY / OPENAI_API_KEY to Render billing env vars for cloud LLM support
- 📋 Admin Dashboard: make-admin UI, model CRUD, user search/filter, date range filters, CSV export
- 📋 User Dashboard: task/habit/routine CRUD, live pomodoro timer, timer settings editor, report filtering
- 📋 KMP app: auth, data screens, timer UI, sync
