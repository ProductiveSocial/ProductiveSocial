sequenceDiagram
    participant Client as CLIENTS
    participant Services as USER-FACING SERVICES
    participant Internal as INTERNAL SERVICE
    participant Billing as BILLING (Python/FastAPI)
    participant LLMs as LLM Providers

    %% Client initiates request
    Client ->> Services: JWT (Bearer Token)

    %% User-facing services interact
    Services ->> Selfmanager: Request tasks, habits, routines, notes, projects
    Services ->> Timer: Request pomodoro sessions, focus rooms, streaks
    Services ->> Analytics: Request AI analysis
    Services ->> UserProfiles: Request profile data
    Services ->> Journal: Request journal entries

    %% Internal Service interactions
    Services ->> Internal: X-Internal-Key (service-to-service request)
    Internal ->> Billing: Request credit ledger, LLM inference, model registry
    Billing ->> LLMs: Invoke LLM provider (Ollama, Claude, GPT)
    LLMs -->> Billing: Return inference results
    Billing -->> Internal: Return AI analysis, update credit ledger
    Internal -->> Services: Send processed data / responses

    %% Data persists in respective databases
    Selfmanager -->> Selfmanager DB: Store tasks, habits, routines, notes, projects
    Timer -->> Timer DB: Store pomodoro sessions, streaks, focus rooms
    Analytics -->> Analytics DB: Store AI reports
    UserProfiles -->> User Profile DB
    Journal -->> Journal DB
    Social -->> Social DB

    %% Final response to client
    Services -->> Client: AI analysis report / data
