graph TD
    %% Clients Layer
    Clients["Clients<br/>- KMP (Android/iOS/Desktop)<br/>- Web App (planned)"]
    Clients -->|JWT Bearer Tokens| Services

    %% User-Facing Services
    Services["User Services<br/>- Selfmanager<br/>- Timer<br/>- Analytics<br/>- User Profiles<br/>- Journal (planned)"]
    Services -->|X-Internal-Key| Internal_API

    %% Databases for each service
    Selfmanager_DB["Selfmanager DB"]
    Timer_DB["Timer DB"]
    Analytics_DB["Analytics DB"]
    UserProfile_DB["User Profile DB"]
    Journal_DB["Journal DB"]
    Social_DB["Social Service DB"]
    Services --> Selfmanager_DB
    Services --> Timer_DB
    Services --> Analytics_DB
    Services --> UserProfile_DB
    Services --> Journal_DB
    %% Social Service
    Social["Social Module<br/>- Community Feed<br/>- Achievements<br/>- Habit Blueprints<br/>- Accountability<br/>- Focus Rooms"]
    Social --> Social_DB

    %% Internal Microservice Layer
    Internal_Service["Internal Services<br/>- Billing & ML inference<br/>- Credit Ledger<br/>- Model Registry<br/>- Behavior Models"]
    Internal_Service -->|API Calls| Billing_Providers

    %% LLM Providers
    Billing_Providers["LLM Providers<br/>- Ollama<br/>- Claude<br/>- OpenAI/GPT"]
    Internal_Service --> Billing_Providers