graph TD
    User([User Client: PS_KMP / Web App])
    User -- JWT (Bearer) --> Selfmanager
    User -- JWT (Bearer) --> Timer
    User -- JWT (Bearer) --> Analytics
    User -- JWT (Bearer) --> UserProfiles
    User -- JWT (Bearer) --> Journal

    subgraph User-Facing Services
        SM[selfmanager : Kotlin/Ktor]
        TM[timer : Kotlin/Ktor]
        AN[analytics : Kotlin/Ktor]
        UP[user profiles : Kotlin/Ktor]
        JD[journal : Kotlin/Ktor]
    end

    subgraph Databases
        SM_DB[(selfmanager_db :5432)]
        TM_DB[(timer_db :5434)]
        AN_DB[(analytics_db :5435)]
        UP_DB[(user_profile_db)]
        JD_DB[(journal_db)]
    end

    %% Client requests
    User -->|JWT (Bearer)| SM
    User -->|JWT (Bearer)| TM
    User -->|JWT (Bearer)| AN
    User -->|JWT (Bearer)| UP
    User -->|JWT (Bearer)| JD

    %% Internal service interactions with internal key
    AN -- X-Internal-Key --> Billing
    TM -- X-Internal-Key --> Billing
    SM -- X-Internal-Key --> Billing
    UP -- X-Internal-Key --> Billing
    JD -- X-Internal-Key --> Billing

    %% Service to database
    SM --> SM_DB
    TM --> TM_DB
    AN --> AN_DB
    UP --> UP_DB
    JD --> JD_DB

    %% Billing and LLM inference
    Billing -- API call --> LLM([LLM Providers: Ollama / Claude / GPT])
