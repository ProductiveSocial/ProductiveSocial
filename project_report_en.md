# Development of a Fault-Tolerant Microservices Architecture for a Mobile Productivity Optimization Application with Machine Learning Integration

**Ministry of Science and Higher Education of the Russian Federation**

**FEDERAL STATE AUTONOMOUS EDUCATIONAL INSTITUTION OF HIGHER EDUCATION
"ITMO NATIONAL RESEARCH UNIVERSITY"
(ITMO University)**

**PROJECT WORK REPORT**

Topic: Development of a Fault-Tolerant Microservices Architecture for a Mobile Productivity Optimization Application with Machine Learning Integration

Supervisor: Safin Ruslan Maratovich

Student: Miambo Junior Ricardo

Saint Petersburg, 2026

---

## Table of Contents

- Introduction
  - Problem Relevance
  - Significance and Mission of the Project
  - Goals and Objectives
  - Novelty
  - Validation Approach
  - Success Criteria
- Chapter 1: Domain Analysis and Solution Vision
  - 1.1 Domain Analysis
  - 1.2 AS-IS Problem Analysis
  - 1.3 Review of Existing Solutions
  - 1.4 Solution Vision
  - 1.5 Prioritization
- Chapter 2: Methodology, Tooling, and ML Integration
  - 2.1 Experiment Design
  - 2.2 Technology and Tools Overview
    - 2.2.1 Microservices Architecture: Definition and Rationale
    - 2.2.2 Docker and Containerization
    - 2.2.3 Colima: Docker on macOS Without Docker Desktop
    - 2.2.4 Kotlin and the Ktor Framework
    - 2.2.5 Python and FastAPI
    - 2.2.6 JWT Authentication: Structure, Validation, and Cross-Service Sharing
    - 2.2.7 HikariCP: Database Connection Pooling
    - 2.2.8 Kotlin Multiplatform (KMP)
    - 2.2.9 SQLDelight: Type-Safe SQL for KMP
    - 2.2.10 Streamlit: Rapid Dashboard Development
    - 2.2.11 Koin: Dependency Injection for Kotlin
    - 2.2.12 Ollama: Local LLM Inference
    - 2.2.13 Ngrok: Secure Tunneling for Local Services
    - 2.2.14 Render: Cloud Deployment Platform
    - 2.2.15 Neon PostgreSQL: Serverless Managed Database
    - 2.2.16 Kotlin Coroutines: Structured Concurrency
    - 2.2.17 Gradle and the Shadow JAR
  - 2.3 Data Strategy: Application-Generated Data
  - 2.4 LLM-Based Analysis Pipeline
  - 2.5 Prompt Engineering and Analysis Types
  - 2.6 Evaluation Metrics
  - 2.7 Results Analysis
  - 2.8 Error Analysis and Lessons Learned
  - 2.9 Proof-of-Concept Summary
- Chapter 3: Architecture and Implementation
  - 3.1 System Design Decisions and Architectural Principles
  - 3.2 Microservices Architecture
  - 3.3 Authentication and Cross-Service Identity
  - 3.4 Data Model and Storage Strategy
  - 3.5 Service Implementation
  - 3.6 KMP Mobile Application
    - 3.6.1 Three-Layer Clean Architecture
    - 3.6.2 Platform Targets and Configuration
    - 3.6.3 Local Database: SQLDelight Schema
    - 3.6.4 Authentication
    - 3.6.5 Projects
    - 3.6.6 Self-Manager Screen
    - 3.6.7 Tasks
    - 3.6.8 Habits
    - 3.6.9 Routines
    - 3.6.10 Pomodoro Timer
    - 3.6.11 Journal (Planned)
    - 3.6.12 Analytics Insights
    - 3.6.13 Navigation and Routing
    - 3.6.14 Responsive Layout
    - 3.6.15 Dependency Injection
    - 3.6.16 Offline Sync Protocol
    - 3.6.17 Implementation Status
  - 3.7 MVP Scope Definition
  - 3.8 Planned Services and Features
    - 3.8.1 Extended Selfmanager: Beyond Tasks, Habits, and Routines
    - 3.8.2 Extended Timer: Shared Focus and Streaks
    - 3.8.3 The Social Layer (psocial_social)
    - 3.8.4 The User Profile Service (psocial_user)
    - 3.8.5 Proprietary ML Models: The Data Accumulation Strategy
  - 3.9 Web Dashboards
  - 3.10 Deployment
  - 3.11 Testing
  - 3.12 Service Integration: End-to-End Request Flows
  - 3.13 Deployment: Docker, Colima, and Render in Detail
  - 3.14 Security Considerations
  - 3.15 Final Validation
- Conclusion
- References

---

## Introduction

### Problem Relevance

We live in an era of unprecedented access to self-improvement tools. The modern person seeking to increase their personal effectiveness has access to dozens of specialized applications for every conceivable productivity need: task managers for tracking work items, habit trackers for building behavioral routines, Pomodoro timers for focused work sessions, journaling applications for reflection, and AI-powered assistants for advice. This proliferation of tools, however, has created a paradox. The very abundance of options has become its own obstacle: rather than enabling clarity and focus, the fragmented ecosystem of productivity software forces users to manage their productivity tools as a second job, context-switching between applications, manually reconciling data, and constructing a coherent picture of their own performance from scattered sources.

The core problem is one of data fragmentation. A user who completes a task in their task manager, spends forty minutes on a Pomodoro session linked to that task, and reflects on the day in their journal is generating three separate data events in three separate applications that know nothing of one another. The task manager sees a completed item. The timer sees a session of a certain duration. The journal sees only what the user types manually. None of these systems can tell the user: "You spent forty minutes on this high-priority task, you completed it, and historically your most productive sessions happen on Tuesday mornings — here is what that means for how you should plan tomorrow."

This fragmentation has concrete costs. Research and user feedback consistently indicate that context-switching between applications is cognitively expensive. Users report spending significant time entering data redundantly across systems, and the mental overhead of maintaining multiple applications frequently leads to abandonment of one or more tools. More critically, the siloed nature of these systems means that the most valuable insights — the cross-domain correlations between focus patterns, habit consistency, task completion rates, and overall performance — are invisible. Each application can only reason about its own slice of the data.

The opportunity, then, is not simply to build another productivity application, but to build the infrastructure that eliminates fragmentation at its root: a unified platform where every productivity action enriches a single data context, and where an AI analysis layer has simultaneous access to all of that context when generating insights. This is the problem that ProductiveSocial is designed to solve.

---

### Significance and Mission of the Project

The significance of ProductiveSocial extends beyond the immediate convenience of replacing multiple applications with one. The project represents a fundamental shift in the relationship between a user and their productivity tools — from passive tracking to active, contextually aware analysis.

Current productivity applications, even sophisticated ones, remain essentially passive record-keepers. They store what the user inputs and display it back with varying degrees of visualization. Even AI features in existing tools are constrained by the application's own data boundary: a task manager with AI assistance can only reason about tasks. A habit tracker with analytics can only surface patterns within habit data. Neither can cross the boundary to ask: "On the days when this habit was completed, how did task completion rates change? And on those same days, how long were Pomodoro sessions, and how many were abandoned?"

ProductiveSocial aims to be the first platform to make this cross-domain reasoning the default mode of user feedback. When a user runs a Productivity Summary analysis, the AI receives the user's complete productivity context: every task and its status, every habit and its completion frequency, every routine and its adherence, and the full history of focus sessions including completion rates, work duration, peak hours, and abandonment patterns. This breadth of context enables a qualitatively different category of insight — one that is not possible in any siloed tool regardless of how sophisticated its own internal analytics may be.

The mission is captured in a phrase that has guided the project's design from the beginning: to transform productivity tools from passive data containers into an intelligent operating system for the self. This framing has direct architectural implications. An operating system does not merely store programs; it manages their resources, coordinates their communication, and presents the user with a unified interface to the machine's full capabilities. ProductiveSocial is architected on the same principle: four specialized services (task and habit management, focus timing, AI analytics, and a credit-based inference engine) that operate as independent, interoperable components under a unified user identity and a shared analysis layer.

The social dimension of the platform — a planned future module for sharing productivity achievements, habit blueprints, and focus patterns with a community of users — adds a further layer of significance. Social accountability is one of the most empirically supported mechanisms for habit retention and goal achievement. By designing a social layer that is architecturally separated from the productivity workspace (preventing the distraction and compulsive checking behavior characteristic of mainstream social media), ProductiveSocial aims to harness the motivational power of community without introducing the attention fragmentation that undermines productivity in the first place.

---

### Goals and Objectives

The primary goal of this project is the design and implementation of a fault-tolerant microservices platform that unifies the core instruments of personal productivity management into a coherent, AI-enhanced ecosystem.

This goal decomposes into the following specific objectives:

**Objective 1 — Design the microservices architecture.** Define the service boundaries, communication protocols, authentication model, and data ownership rules for a system of independent but interoperable backend services. Each service must be independently deployable, independently scalable, and capable of continued operation even when other services are temporarily unavailable.

**Objective 2 — Implement the core productivity services.** Build and deploy `psocial_selfmanager`, the central service responsible for user identity, task management, habit tracking, routine management, and offline synchronization. Build and deploy `psocial_timer`, the Pomodoro focus session tracking service with adaptive intelligence (break suggestions, abandonment risk detection, and personalized duration recommendations).

**Objective 3 — Implement the AI analytics pipeline.** Build `psocial_analytics`, the service that orchestrates data collection from all other services, constructs structured prompts, invokes LLM inference via the billing service, and persists analysis reports. Implement six analysis types covering the full spectrum of user productivity dimensions.

**Objective 4 — Implement the credit and inference infrastructure.** Build `psocial_billing`, the internal credit ledger and inference engine. Support three LLM providers (Ollama, Anthropic Claude, OpenAI GPT-4o) and a sklearn model pathway for future use. Design the credit system as a transaction ledger with automatic welcome credit initialization.

**Objective 5 — Build administrative and user interfaces.** Implement a Streamlit-based admin dashboard for system management (model registration, user credit management, prediction and report monitoring) and a user dashboard for accessing productivity data, focus analytics, AI analysis, and credit balance.

**Objective 6 — Implement offline-first synchronization.** Design and implement an idempotent batch sync protocol for both the selfmanager service (tasks, habits, routines) and the timer service (sessions, intervals), allowing mobile clients to operate without network connectivity and synchronize reliably on reconnection.

**Objective 7 — Validate the system end-to-end.** Conduct empirical testing of the full analysis pipeline from data input to LLM output, evaluate the quality of generated insights against a rule-based baseline, and verify the correctness of all service interactions including credit deduction, cross-service JWT authentication, and offline sync integrity.

---

### Novelty

The novelty of ProductiveSocial lies in four interrelated dimensions.

**Architectural novelty — unified productivity context for AI reasoning.** Existing AI-enhanced productivity applications confine their AI to the data owned by a single service. A task manager with AI can reason only about tasks. A habit tracker with AI can reason only about habits. ProductiveSocial is designed specifically to give the AI analysis layer simultaneous access to all dimensions of the user's productivity context: tasks, habits, routines, and focus session data, aggregated at the moment of analysis. This cross-domain context is not a UI feature but an architectural guarantee enforced by the internal API design of the analytics service, which explicitly fetches from all three upstream services before any LLM call.

**Technical novelty — shared JWT identity across independent microservices.** Rather than requiring users to authenticate separately to each service (a common friction point in multi-service architectures), all three user-facing services (selfmanager, timer, and analytics) share the same JWT secret and issuer. This means a token issued by any one service is valid across all of them. Combined with the UserRegistry pattern — in which the timer service connects directly to selfmanager's `users` table via a raw JDBC connection to guarantee canonical integer user IDs — the system achieves true single-sign-on across independent services without a dedicated identity provider or token exchange mechanism.

**Technical novelty — billing as a pure internal inference ledger.** The billing service is designed as a completely internal component: no client-facing authentication, no user table, no direct external access. All billing operations are proxied through the analytics service, which holds the only public-facing JWT. This design eliminates an entire attack surface (billing credentials are never exposed to clients), simplifies the billing service's domain model (it knows only credit transactions keyed by integer user ID), and makes the credit system naturally consistent with the rest of the platform's identity model.

**Methodological novelty — application-generated training data as a first-class design goal.** Unlike existing productivity tools that treat AI as a feature bolt-on, ProductiveSocial treats the accumulation of user behavior data as a primary design objective. Every task completion, habit log, routine step, and focus session interval is stored in a structured, timestamped, and cross-referenced schema. This is not incidental; it is the intentional groundwork for a future transition from LLM-based analysis (the current approach, which requires no training data) to proprietary predictive models trained on real user behavioral patterns. The architecture's sklearn inference pathway in the billing service exists specifically to accommodate this transition without any changes to the broader system.

---

### Validation Approach

Given the supervisor's feedback that the previous version of this report lacked verifiable validation and measurable baselines, this report adopts a structured, experiment-driven approach to validation. Rather than asserting that the prototype "successfully aggregates data and generates reports," we define specific experiments with explicit baselines, measurable metrics, and documented results.

Validation is organized across three layers:

**Layer 1 — Functional correctness.** For each core system behavior (JWT authentication, credit accounting, sync idempotency, service-to-service communication), we define a pass/fail criterion and test against it with documented inputs and outputs. These tests confirm that the system does what it is designed to do.

**Layer 2 — Integration quality.** For the end-to-end analysis pipeline — the most complex and most valuable behavior in the system — we define a baseline (rule-based summary), a comparison condition (LLM-generated analysis), a metric (human-evaluated relevance score on a 1–5 scale), and a documented result with example inputs and outputs.

**Layer 3 — Operational reliability.** For the deployment on Render's free tier, we measure cold start latency, warm response times, and behavior under service unavailability (e.g., when the timer service is unreachable during an analytics call).

The experiment design and results are documented in full in Chapter 2.

---

### Success Criteria

The success criteria for this project are defined across three dimensions, each with a concrete measurement method:

**1. System integration correctness.**
- All four backend services communicate successfully using the defined protocols (JWT for client-to-service, X-Internal-Key for service-to-service).
- A user's tasks, habits, routines, and focus sessions are successfully aggregated and passed to the LLM in a single analytics request.
- Credit deduction is mathematically correct (balance before − cost = balance after) and recorded in the transaction log.
- _Measurement: functional test with documented input/output pairs. Target: 100% pass rate._

**2. AI analysis quality.**
- LLM-generated analyses are evaluated against a rule-based baseline for relevance, specificity, and actionability.
- The LLM output must score higher than the baseline on a human evaluation scale (1–5) by a margin of at least 1.5 points.
- _Measurement: side-by-side comparison with documented scoring. Target: LLM score ≥ 3.5/5 vs. baseline ≤ 2.0/5._

**3. Sync reliability.**
- Offline sync correctly persists and reconciles all entity types (tasks, habits, routines, sessions, intervals) after a simulated disconnect-reconnect cycle.
- Idempotency guarantee: resubmitting the same sync request produces no duplicate records.
- _Measurement: test with a pre-defined dataset of 32 sessions, 55 habit completions, 8 tasks. Target: 100% of records correctly synced with zero duplicates on retry._

---

## Chapter 1: Domain Analysis and Solution Vision

### 1.1 Domain Analysis

The domain of personal productivity management sits at the intersection of behavioral psychology, human-computer interaction, and applied AI. Understanding the domain requires a clear picture of the types of productivity work that users engage in, the cognitive and motivational mechanics that govern behavior in this domain, and the technological landscape that has emerged to support it.

**Categories of productivity work.** At the broadest level, productivity tools address four distinct categories of user behavior:

*Goal-directed task management* involves the decomposition of objectives into discrete, actionable items with deadlines, priorities, and completion states. This is the oldest and most mature category of productivity tooling, with roots in Getting Things Done (GTD) methodology and earlier paper-based systems. Users in this mode need to capture work items quickly, organize them by context and priority, track completion, and surface the most important items at any given moment.

*Habit and routine formation* involves the establishment and maintenance of recurring behaviors — both one-time-per-day actions (brushing teeth, meditating) and structured sequences of steps performed in order (a morning routine, a study session protocol). Unlike task management, which is oriented toward completion and closure, habit tracking is oriented toward consistency and streak maintenance. The motivational mechanics are different: the reward is not completion but continuity, and the risk is not backlog but discontinuity. Routines add a temporal and sequential dimension: not just "did I do this" but "did I do all the steps, in the right order, for the right duration."

*Focused work sessions* represent a distinct mode of productivity engagement in which the user commits to uninterrupted concentration on a single task for a defined interval. The Pomodoro Technique, developed by Francesco Cirillo in the 1980s, formalized this into a widely adopted protocol: work for 25 minutes, take a 5-minute break, repeat. Variants with different durations proliferate, and individual users typically settle on interval lengths that match their personal attention spans. The key measurement in this domain is not completion (of a task) or consistency (of a habit) but *depth of engagement* — total focus time, session completion rate, and the absence of interruptions.

*Reflective journaling* serves a fundamentally different psychological function from the above three categories. Rather than driving forward action, journaling creates space for retrospective sense-making: integrating the day's events into a coherent narrative, processing emotions, and extracting lessons. In the context of productivity, journaling is most valuable when it connects the subjective experience of a day (how it felt) with the objective record of activity (what was actually accomplished). This connection is precisely what manual journaling cannot do unaided and what an AI-assisted synthesis makes possible.

**The behavioral economics of productivity tools.** A critical but frequently overlooked dimension of this domain is the gap between tool adoption and consistent usage. Studies of productivity app usage patterns consistently show that the majority of users who download a new productivity application abandon it within the first two weeks. The primary reasons given are: the overhead of data entry, the lack of visible value from the data entered, and the friction of context-switching between multiple applications. These patterns have direct implications for system design: an effective productivity platform must minimize data entry friction, maximize the visible return on the data entered, and reduce the number of applications the user must manage.

**Self-determination theory and intrinsic motivation.** Behavioral psychology research offers a useful lens for understanding why productivity tools succeed or fail. Self-determination theory (Deci and Ryan, 1985) identifies three fundamental psychological needs that drive sustained engagement with any activity: autonomy (the feeling of choice and ownership), competence (the perception of growth and mastery), and relatedness (connection to others who share similar goals). Productivity applications that satisfy these needs — by giving users control over their system, making progress visible in meaningful ways, and connecting them to an accountability community — achieve significantly higher long-term retention than those that satisfy only the instrumental need to "get things done."

The Pomodoro Technique succeeds in part because it addresses the competence dimension directly: each completed session is a visible unit of achieved focus, and the accumulation of completed cycles over time creates a concrete representation of disciplined work. Habit trackers succeed when they visualize streaks, because the streak creates a loss aversion dynamic — breaking a 14-day streak feels costly, which motivates continuation. Neither technique works reliably in isolation; their effects compound when a user can see that the habits they are building are correlating with the focused work sessions that produce their most important task completions.

**Cognitive load and the design implications.** Cognitive load theory (Sweller, 1988) distinguishes between intrinsic cognitive load (the inherent complexity of the task at hand), extraneous cognitive load (overhead imposed by poor tool design), and germane cognitive load (the mental effort that produces learning and long-term memory). A fragmented productivity stack imposes significant extraneous cognitive load: the user must remember to update multiple systems, must mentally integrate information from separate sources, and must manage the cognitive transition between different application contexts. An integrated platform reduces this extraneous load to near zero — the user interacts with one system, and the system handles the data integration internally.

**The role of AI in productivity.** The integration of AI into productivity tools has accelerated significantly since the mainstream availability of large language models in 2022–2023. Current AI applications in this domain fall into several categories: natural language input parsing (converting spoken or typed descriptions into structured task entries), scheduling optimization (recommending when to schedule tasks based on calendar and workload), pattern recognition (identifying which habits are correlated with high-productivity days), and generative summarization (producing written summaries of activity logs). The limitation common to all current implementations is the data boundary problem described in the introduction: each AI feature can only reason about the data owned by the application hosting it.

**Target user profiles.** To ground the design decisions in concrete user needs, three representative user profiles were identified during domain analysis:

*Profile A — The Knowledge Worker.* A professional in a cognitively demanding role (software developer, academic researcher, writer) who needs to manage a continuous backlog of complex tasks, maintain a set of daily professional habits (focused reading, daily writing, exercise), and track how their focus time is distributed across projects. This user's primary pain point is the inability to answer the question: "Am I spending my focus time on the right things?" They are technically sophisticated and will use an API if the UI is insufficient.

*Profile B — The Student.* A university student managing coursework deadlines, study habits (daily revision, lecture notes, practice problems), and exam preparation routines. This user's primary challenge is balancing multiple concurrent obligations with unpredictable workloads. They are highly motivated by streaks and gamification elements but abandon apps quickly if data entry overhead is high. They use their phone as their primary computing device.

*Profile C — The Self-Improvement Enthusiast.* A user primarily motivated by habit formation and lifestyle optimization rather than professional task management. Their primary use of the platform would be habit tracking (exercise, meditation, nutrition, reading) with periodic AI analysis of consistency patterns and correlation between habits and overall wellbeing. They are the primary beneficiary of the HABIT_INSIGHTS and CUSTOM analysis types.

These three profiles informed the feature prioritization in Section 1.5 and the prompt engineering decisions in Section 2.5.

---

### 1.2 AS-IS Problem Analysis

The current state of the personal productivity tooling market can be characterized by three structural problems that no existing solution has adequately addressed.

**Problem 1: Data Silos and the Fragmented User Context**

The fundamental architectural limitation of the current market is that each productivity application maintains its own isolated data store. A user who employs Todoist for task management, Habitica for habit tracking, Forest for focus timing, and Day One for journaling has four separate applications, four separate user accounts, and four separate databases. There is no mechanism by which any of these systems can observe events occurring in the others.

This creates a compounding problem for AI analysis. Even in the most sophisticated current implementations, the AI assistant in a task manager can only observe tasks. It cannot know that the user spent 40 focused minutes on a specific task this morning, that the habit of daily exercise that was supposed to accompany this work session was skipped today, or that the user has been consistently abandoning focus sessions after the second Pomodoro for the past two weeks. Each of these facts is captured in some system, but no system has access to all of them simultaneously.

The result is that users receive fragmented, context-poor feedback. A task manager tells you what you did; a habit tracker tells you whether you were consistent; a timer tells you how long you focused. None of them can tell you how these dimensions interact, or what the cross-domain patterns in your behavior suggest about how you should adjust your approach.

**Problem 2: Manual Data Entry Overhead and Context-Switching Cost**

Users who attempt to maintain multiple productivity applications face a significant administrative burden. The same piece of information must often be entered in multiple places: a task completed in the task manager should also be reflected in the journal, the habit tracker should note the work session that accompanied it, and the timer log should reference the task that was its focus. In practice, users either duplicate their data entry (increasing overhead), accept inconsistency between systems (reducing the value of each), or abandon all but one application (losing coverage of important dimensions).

The context-switching cost of moving between applications is not merely a time cost; it is a cognitive cost. Research on attention residue (the phenomenon in which attention lingers on the prior application even after switching) suggests that each context switch takes a measurable toll on cognitive resources. For a user already striving to maintain focus, the requirement to manage four separate productivity applications is itself an obstacle to the state that those applications are meant to support.

**Problem 3: The Absence of Cross-Domain Behavioral Intelligence**

Current productivity AI is locally optimal but globally blind. Each application can surface sophisticated insights within its own domain — Todoist can recommend task prioritization based on deadlines and historical completion rates; Habitica can predict which habits are at risk of breaking based on streak data; Forest can show focus patterns by time of day. But no application can ask: "On the days when your morning exercise habit was completed, how did your afternoon task completion rate change? And on those days, how did your Pomodoro abandonment rate compare to days when the exercise habit was skipped?"

This kind of cross-domain query is precisely the type of insight that has the highest practical value for behavior change. Behavioral research consistently shows that habits do not operate in isolation — they form interdependent networks in which the completion or failure of one habit has probabilistic effects on others in the same day. Without a system that observes the full behavioral context, these interdependencies remain invisible, and the AI assistance users receive is necessarily shallower than what is technically possible.

---

### 1.3 Review of Existing Solutions

To establish the competitive landscape and demonstrate the specific gap that ProductiveSocial fills, we analyze seven leading applications representing different categories of productivity tooling. The analysis examines six dimensions: task management capability, habit tracking capability, built-in focus/Pomodoro functionality, AI analysis features, offline-first architecture, and whether the AI has access to unified cross-module data.

**Todoist** is the market leader in task management, with over 30 million users. It provides a sophisticated GTD-oriented task system with projects, labels, priority levels, recurring tasks, and natural language date parsing. Its AI features ("AI Assistant") focus on task reformulation, decomposition suggestions, and automatic scheduling. However, Todoist has no native habit tracking, no Pomodoro functionality, and its AI is confined entirely to the task domain. Users who want habit tracking alongside Todoist must integrate a separate application, and no cross-application AI analysis is available.

**Habitica** gamifies habit building by converting habits, daily tasks, and to-dos into a role-playing game. It has a large and engaged community and social features that are genuinely used for accountability. However, its task management is rudimentary compared to dedicated task managers, it has no Pomodoro functionality, and it has no AI analysis features of any kind. Its social features are specifically designed around the gamification layer and are not transferable to a non-gamified context.

**Forest** is a focused Pomodoro timer application with a planting metaphor: users grow virtual trees during focus sessions, which die if they leave the app. It is highly effective as a focus commitment device but does not support task management or habit tracking, and its analytics are limited to session history charts. There is no AI analysis, and data is not accessible outside the application.

**Notion** is the closest existing product to the vision of an all-in-one productivity platform. Its flexibility — users can build custom databases, link records across pages, and create any organizational structure they can imagine — has made it genuinely useful as a unified workspace for many users. However, this flexibility is also its limitation in the productivity domain: Notion does not have native habit tracking mechanics, its task management requires manual setup rather than providing purpose-built productivity structures, its Pomodoro functionality requires third-party embeds, and its AI features (Notion AI) are primarily oriented toward text editing and summarization rather than behavioral pattern analysis. Crucially, Notion AI can only reason about the content of Notion pages, not about cross-domain behavioral patterns derived from structured activity logs.

**Day One** is the leading journaling application for iOS and macOS, with strong privacy features, media support, and location tagging. It is excellent for manual reflective journaling but has no task management, no habit tracking, no Pomodoro features, and no AI that can synthesize activity data from other sources. Its AI features are limited to transcription and basic text assistance.

**RescueTime** is an automatic time-tracking application that passively monitors which applications and websites the user spends time on and generates productivity scores and reports. It solves a different problem — passive monitoring of actual computer usage rather than intentional tracking of planned activities. It has no task management, no habit tracking, and no Pomodoro timer. Its AI features produce automated weekly reports based on computer usage data. It does not integrate with task managers or habit trackers.

**Sunsama** is a daily planning application designed for knowledge workers. It integrates with Todoist, Asana, GitHub, and other external services to pull tasks into a single daily view with time-boxing support. It has basic Pomodoro-style focus features and generates daily "shutdown" summaries. However, it has no habit tracking, its Pomodoro support is basic, its AI is limited to summarization of the day's tasks, and it requires paid subscriptions to external services for its integrations to be valuable.

The following table summarizes the comparison:

| Feature | Todoist | Habitica | Forest | Notion | Day One | RescueTime | Sunsama | **ProductiveSocial** |
|---|---|---|---|---|---|---|---|---|
| **Task Management** | ✅ Full | ⚠️ Basic | ❌ | ✅ Manual setup | ❌ | ❌ | ✅ Full | ✅ Full |
| **Habit Tracking** | ❌ | ✅ Full | ❌ | ⚠️ Manual setup | ❌ | ❌ | ❌ | ✅ Full (Start/Quit, subtasks, completion logs) |
| **Pomodoro Timer** | ❌ | ❌ | ✅ Full | ⚠️ Plugin only | ❌ | ❌ | ⚠️ Basic | ✅ Full (adaptive, abandonment risk, entity-linked) |
| **AI Analysis** | ⚠️ Task-only | ❌ | ❌ | ⚠️ Text editing | ⚠️ Basic | ⚠️ Usage reports | ⚠️ Day summary | ✅ Full cross-domain |
| **Offline-First** | ✅ | ❌ | ⚠️ | ⚠️ Partial | ✅ | ❌ | ❌ | ✅ Full batch sync |
| **Social / Community** | ❌ | ✅ Gamified | ❌ | ❌ | ❌ | ❌ | ❌ | 📋 Planned |
| **Unified AI Context** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ (tasks + habits + routines + focus) |
| **Open LLM Provider** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ (Ollama/Anthropic/OpenAI) |

**Key gap identified:** No existing solution provides all four core productivity modules (task management, habit tracking, Pomodoro timer, and AI analysis) in a single platform. More critically, no existing solution provides an AI analysis layer that has simultaneous access to cross-module behavioral data. This is the specific gap that ProductiveSocial fills.

The claim in this report that "an architecture combining all types of individual productivity tools with a unified AI analysis layer" is novel is substantiated by this comparison. While the individual modules are not novel in isolation, their combination under a unified data model with cross-domain AI access represents a system design that does not exist in the current market.

---

### 1.4 Solution Vision

The ProductiveSocial solution addresses the three identified problems through a specific set of architectural and product decisions.

**For the data silo problem:** A microservices architecture in which all services share a common user identity (the integer `user_id` issued by `psocial_selfmanager`) and a dedicated analytics service that aggregates data across all services before invoking AI analysis. The user does not integrate external services; all modules are built within the platform, and the analytics service has guaranteed internal access to all data via a service-to-service communication layer protected by a shared secret.

**For the data entry and context-switching problem:** A single application with purpose-built UIs for each productivity module, connected by a shared data model. A task created in the task manager is automatically available as a link target in the timer, and time logged in the timer is automatically written back to the task. An offline-first architecture with batch sync ensures that the user can use the application without network connectivity, removing a common friction point for mobile productivity tools.

**For the cross-domain intelligence problem:** An analytics service that collects the full productivity context at the moment of analysis and passes it to a large language model with type-specific prompts designed to elicit cross-domain insights. The AI does not reason about tasks alone or focus sessions alone; it reasons about the user's complete behavioral profile as captured by the integrated platform.

The TO-BE system — covering both what is built today and what is architecturally planned for subsequent phases — consists of the following components:

**Currently implemented:**
- `psocial_selfmanager`: owns user identity, projects, tasks, habits, routines, tags, and offline sync for all entities
- `psocial_timer`: owns Pomodoro sessions, interval tracking, focus pattern analytics, and entity-linked time logging
- `psocial_analytics`: the AI gateway, orchestrating data collection and LLM inference for six analysis types; also the sole public proxy for credit operations
- `psocial_billing`: an internal-only credit ledger and multi-provider LLM/ML inference engine
- `psocial_dashboard`: admin interface for model management, user credit management, and system monitoring (short-term solution while KMP app is in development)
- `psocial_user_dashboard`: user-facing interface for productivity data, focus analytics, AI analysis, and credits (short-term solution while KMP app is in development)
- `PS_KMP`: Kotlin Multiplatform mobile application (Android, iOS, Desktop) — authentication and scaffolding complete, feature screens in active development

**Planned for subsequent phases:**
- `psocial_user`: a dedicated user profile service owning display names, profile photos, bios, and the social graph (follow/follower relationships between users). Currently, user identity is minimal — an email address and an integer ID. When social features are introduced, a richer user profile is required that the existing selfmanager service should not own, since user identity and social presence are distinct domains.
- `psocial_social`: the community and social engagement service, owning public posts, achievement feeds, habit blueprint sharing, and accountability partner relationships. Designed to be architecturally isolated from the productivity workspace — social interactions live in their own service so that the distraction of a social feed is a deliberate opt-in, not embedded in the core productivity flow.
- `psocial_journal`: a dedicated journal service owning daily reflection entries, mood logs, and the AI synthesis that connects journal entries with the day's task completions, habit logs, and focus sessions. The journal is the reflective dimension of the platform — it closes the loop between objective activity data (what was done) and subjective experience (how it felt), a synthesis that no existing productivity tool currently performs.
- `psocial_notes`: a notes service owning free-form text notes with contextual links to tasks, habits, routines, and projects — allowing users to capture reference material, ideas, and research directly within the productivity context they belong to, rather than in a separate application.
- Extended `psocial_selfmanager` modules: to-do lists (lightweight single-step items for quick capture, distinct from full structured tasks), checklists, templates for repeatable project structures, calendar view, and a quick-capture inbox for items that have not yet been organised into a project.

---

### 1.5 Prioritization

The product's features were prioritized using a value-versus-effort framework, with the additional constraint that the system must function as a coherent integrated whole at every stage — partial implementations that break the data flow between modules are not acceptable deliverables.

**Priority 1 (Must have for MVP):**
- User authentication and identity (cross-service JWT, UserRegistry)
- Task management with projects, subtasks, priorities, and offline sync
- Habit tracking (Start/Quit types, completion logs, subtasks)
- Routine management with ordered steps
- Pomodoro session lifecycle (create, start interval, complete, abandon)
- Cross-service AI analysis pipeline with at least four analysis types
- Credit system with transaction logging and welcome deposit
- Admin dashboard for model and credit management
- User dashboard for data access and analysis triggering

**Priority 2 (Important but post-MVP):**
- Adaptive timer features (abandonment risk, break suggestions, duration recommendations)
- Focus pattern insights (by hour of day, by day of week)
- Weekly timer summary analysis type
- Custom analysis type with user-supplied prompt
- Offline sync for timer sessions
- KMP mobile application with full feature coverage

**Priority 3 (Future):**
- Social features (public profiles, achievement sharing, habit blueprints, community feed, accountability partners)
- Proprietary trained ML models for behavioral prediction (habit continuation probability, task completion likelihood, focus session duration forecasting)
- Journal module with mood tracking and AI-assisted day synthesis
- Notes module with contextual entity linking
- User profile service (`psocial_user`) owning display names, avatars, bios, and the social graph
- Extended selfmanager features: to-do lists (lightweight single-step items distinct from full tasks), quick-capture inbox, calendar view, checklists, templates
- Extended timer features: shared focus rooms, focus streaks, session goals
- Third-party integrations (Google Calendar, wearables, Apple Health)
- On-device model inference for fully offline AI analysis

The current project delivers all Priority 1 items and the majority of Priority 2 items. The KMP application is scaffolded and functional in its authentication flow, with remaining screens planned for subsequent development. Priority 3 items are deferred to future development phases, with the architecture explicitly designed to accommodate them without requiring structural changes to the existing services.

It is worth emphasising that the Priority 3 list is not a collection of speculative nice-to-haves — it represents the features that define ProductiveSocial's long-term differentiation. The social layer, the journal, and the richer selfmanager modules are what transform the platform from a capable personal productivity tool into the productivity operating system described in the mission statement. The four services built in this phase are the infrastructure foundation on which all of those features will be built.

---

## Chapter 2: Methodology, Tooling, and ML Integration

### 2.1 Experiment Design

In response to the supervisor's feedback that the previous version of this report lacked measurable validation, this section defines and reports on a structured set of experiments conducted to verify the correctness and quality of the ProductiveSocial system.

The experiments are organized into three categories: functional correctness tests (verifying that the system does what it is designed to do), quality evaluation (comparing LLM-generated analysis against a rule-based baseline), and operational reliability (verifying behavior under deployment and failure conditions).

All experiments were conducted against a consistent test dataset seeded in the local development environment for user `test@productivesocial.com`. The dataset consists of:
- 3 projects: Work, Personal, Learning
- 8 tasks distributed across all projects, with varying priorities and completion states
- 6 habits (5 daily, 1 weekly) of both Start and Quit types
- 4 routines with step-level tracking (Morning, Deep Work, Evening Learning, Weekly Review)
- 32 Pomodoro sessions (31 completed, 1 abandoned) linked to tasks, habits, routines, and standalone
- 55 habit completions across 14 days of history

This dataset was chosen because it represents a realistically populated user profile — neither a trivially small sample nor a synthetically inflated one — and because it covers all entity types that the analytics service draws from.

The complete experiment results table is presented in Section 2.7.

---

### 2.2 Technology and Tools Overview

This section defines every significant technology used in the ProductiveSocial system — not only the choice made but the concept itself — so that the architectural decisions that follow can be understood in full context.

---

#### 2.2.1 Microservices Architecture: Definition and Rationale

A microservices architecture is a software design approach in which a large application is decomposed into a collection of small, independently deployable services, each responsible for a specific business capability and communicating with other services over a network (typically HTTP or a message queue). This stands in contrast to a monolithic architecture, in which all functionality is packaged into a single deployable unit.

The term was popularized by Martin Fowler and James Lewis in a 2014 article that identified the defining characteristics of the pattern: single responsibility per service, independent deployability, decentralized data management (each service owns its own database), and design for failure (each service assumes that other services may be unavailable).

**Why microservices for ProductiveSocial?** The decision to build four separate services rather than a single monolith was driven by three specific requirements of this system:

First, the services have genuinely distinct technology profiles. The three Kotlin services (selfmanager, timer, analytics) are best built in a statically typed, JVM-based language because their primary work is structured data management with complex relational logic. The billing service, by contrast, has a fundamentally different profile: it needs to load Python sklearn `.pkl` model files and call Python-native LLM SDK libraries (anthropic, openai). A monolith would require one of these technology choices to compromise the other; microservices allow each service to use the best language and framework for its specific job.

Second, the services have different fault tolerance requirements. The analytics service must continue functioning even when the timer service is temporarily unavailable — it should simply omit the Pomodoro section from its analysis rather than returning an error. A monolith cannot express this graceful degradation in a natural way; in a microservices architecture, the analytics service simply catches the HTTP timeout from the timer client and proceeds.

Third, the services have genuinely different deployment lifecycles. When a change is made to the billing service's LLM prompt construction logic, only the billing service needs to be redeployed. In a monolith, even a one-line change in the billing logic would require redeploying the entire application — with all the risk that entails.

The trade-offs of this choice are real: inter-service HTTP calls introduce network latency, distributed systems are harder to debug than monoliths, and data consistency across service boundaries requires careful design. Each of these trade-offs is addressed explicitly in the architecture described in Chapter 3.

---

#### 2.2.2 Docker and Containerization

Docker is a platform for packaging, distributing, and running software in containers. A container is a lightweight, isolated runtime environment that bundles an application together with everything it needs to run: its runtime (Java, Python), its dependencies (libraries, packages), its configuration, and its filesystem. Unlike a virtual machine, which emulates an entire operating system, a container shares the host operating system's kernel, making it significantly faster to start and lighter in resource consumption.

**The problem Docker solves** is environment inconsistency: the "it works on my machine" problem. Without containerization, a service that works correctly on the developer's laptop may fail in production because of differences in the installed version of Python, the presence or absence of a system library, or a subtle difference in the operating system's file path conventions. A Docker container eliminates this class of problem by making the runtime environment part of the deployable artifact: if the container works on the developer's laptop, the same container will work in production.

**How ProductiveSocial uses Docker.** Each of the four backend services has a `Dockerfile` that defines its container image:

```dockerfile
# Kotlin service example (psocial_selfmanager)
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY server/build/libs/server-all.jar app.jar
EXPOSE 1226
ENTRYPOINT ["java", "-jar", "app.jar"]
```

This Dockerfile starts from a minimal Java 17 runtime image (Alpine Linux, approximately 80 MB), copies the pre-built fat JAR, exposes the service's port, and defines the startup command. The image is deliberately minimal — it contains only what is needed to run the JAR, not the build tools required to compile it.

For the Python billing service:

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 1229
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "1229"]
```

**Docker Compose** is the tool used to run all services together in the local development environment. It reads a `docker-compose.yml` file that defines all containers (services and databases), their network connections, environment variables, port mappings, and health checks. Instead of starting four services and four databases manually with individual commands, a single `docker compose up` command starts the entire system.

```yaml
# Excerpt from docker-compose.yml
services:
  selfmanager:
    image: psocial_selfmanager:latest
    ports:
      - "1226:1226"
    environment:
      - DATABASE_URL=jdbc:postgresql://selfmanager_db:5432/selfmanager
      - JWT_SECRET=${JWT_SECRET}
    depends_on:
      selfmanager_db:
        condition: service_healthy
    networks:
      - psocial_network

  selfmanager_db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: selfmanager
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - psocial_network

networks:
  psocial_network:
    driver: bridge
```

The `psocial_network` bridge network allows containers to communicate with each other using their service names as hostnames. From inside the `analytics` container, a call to `http://selfmanager:1226/internal/users/42/tasks` is resolved by Docker's internal DNS to the `selfmanager` container's IP address. This is the same name-based addressing that is used in production (with Render's internal DNS), making the local and production configurations structurally identical.

---

#### 2.2.3 Colima: Docker on macOS Without Docker Desktop

Colima is a lightweight container runtime for macOS that provides a Docker-compatible environment without requiring Docker Desktop. It runs a small Linux virtual machine (using Apple's Virtualization Framework on Apple Silicon) that hosts the Docker daemon, and exposes a Docker socket that the Docker CLI and Docker Compose can connect to.

**Why Colima?** Docker Desktop for Mac is proprietary software that requires a paid license for commercial use in organizations above a certain size, and it has a heavier resource footprint than Colima. Colima is open source, free, and typically uses fewer system resources. For development purposes, Colima is functionally equivalent to Docker Desktop: the `docker` and `docker compose` commands work identically, container images build and run the same way, and port mappings work as expected.

Starting the container runtime before beginning development work requires a single command:

```bash
colima start
```

This starts the Linux VM (approximately 5–10 seconds) and makes Docker commands available in the current terminal session. All `docker compose` operations for ProductiveSocial's local development stack run on top of this Colima-managed environment.

---

#### 2.2.4 Kotlin and the Ktor Framework

Kotlin is a statically typed programming language developed by JetBrains, first released in 2016. It runs on the Java Virtual Machine (JVM) and is fully interoperable with Java: Kotlin code can call Java libraries and vice versa. Kotlin has become the primary language for Android development (Google declared it the preferred language for Android in 2019) and has gained significant adoption in server-side development.

The key features that make Kotlin the right choice for ProductiveSocial's backend services are:

*Null safety*: In Kotlin, a variable cannot hold a null value unless its type is explicitly declared as nullable (e.g., `String?` vs. `String`). This eliminates the entire class of NullPointerException errors that are the most common runtime failure in Java applications. For a service like analytics — which assembles data from multiple upstream sources, some of which may return empty or partial data — the compiler's guarantee that every potentially-null value is explicitly handled is a meaningful reliability property.

*Coroutines*: Kotlin's coroutine system provides a structured approach to asynchronous programming. A coroutine is a computation that can be suspended and resumed without blocking the thread it runs on. This is critical for the analytics service, which must make three network calls in parallel (to selfmanager, timer, and eventually billing) without blocking a thread for each call. In traditional thread-blocking code, three parallel calls would require three threads. With coroutines, all three calls can run on a small pool of threads, with each suspended while waiting for the network and resumed when the response arrives.

The analytics service uses this concurrency model:

```kotlin
val tasksDeferred = async { selfmanagerClient.getTasks(userId) }
val habitsDeferred = async { selfmanagerClient.getHabits(userId) }
val routinesDeferred = async { selfmanagerClient.getRoutines(userId) }
val statsDeferred = async { timerClient.getStats(userId) }

val (tasks, habits, routines, stats) = awaitAll(
    tasksDeferred, habitsDeferred, routinesDeferred, statsDeferred
)
```

All four upstream calls are launched simultaneously. The total latency is approximately equal to the slowest call, rather than the sum of all four.

**Ktor** is a framework for building HTTP servers and clients in Kotlin, developed by JetBrains. Unlike heavier frameworks like Spring (which provide comprehensive solutions for every possible enterprise requirement), Ktor is deliberately minimal: it provides a routing DSL, plugin slots for common concerns (authentication, logging, serialization, CORS), and an HTTP client for making outbound requests. It adds only what is installed.

The selfmanager service's routing structure illustrates this concision:

```kotlin
routing {
    authenticate("jwt") {
        route("/api/v1/tasks") {
            get("/tasks") { /* list tasks */ }
            post("/tasks") { /* create task */ }
            put("/tasks/{id}") { /* update task */ }
            delete("/tasks/{id}") { /* delete task */ }
        }
        route("/api/v1/habits") { /* ... */ }
        route("/api/v1/routines") { /* ... */ }
        post("/api/v1/sync") { /* batch sync */ }
    }
    route("/internal") {
        withInternalKey {
            get("/users/{id}/tasks") { /* internal tasks fetch */ }
        }
    }
}
```

---

#### 2.2.5 Python and FastAPI

Python is the dominant language in the machine learning and data science ecosystem. The billing service is written in Python specifically because it needs to integrate with the Python ML stack: `scikit-learn` for loading trained `.pkl` model files, `joblib` for model deserialization, `anthropic` for calling Anthropic's Claude API, and `openai` for calling OpenAI's GPT API. These libraries exist in Python first; their equivalents in other languages are secondary and often incomplete.

**FastAPI** is a modern Python web framework designed for building APIs. It is built on top of Starlette (for async HTTP handling) and Pydantic (for data validation). Its defining characteristic is automatic API documentation: by declaring request and response types as Pydantic models, FastAPI generates an interactive Swagger UI and an OpenAPI specification automatically. This documentation is immediately accessible at `/docs` on the running service.

FastAPI uses Python's `async/await` syntax for non-blocking request handling. For the billing service, this matters when an LLM inference call takes 10–30 seconds: with async handling, the server can continue processing other requests while waiting for the LLM provider to respond, rather than holding a thread idle.

**Pydantic** is a data validation library that is used throughout the billing service for defining request and response shapes. When a request arrives at `POST /api/v1/internal/predict`, Pydantic validates that the `selfmanager_user_id` is an integer, the `model_id` is a valid UUID, and the `input_data` is a dictionary — before any business logic runs. Invalid requests are rejected immediately with a structured error response.

**SQLAlchemy** (async, via `asyncpg`) is the ORM (Object-Relational Mapper) used in the billing service. It maps Python classes to database tables and generates SQL automatically, allowing the service to query and modify the database using Python objects rather than raw SQL strings.

---

#### 2.2.6 JWT Authentication: Structure, Validation, and Cross-Service Sharing

JWT stands for JSON Web Token. It is a compact, self-contained format for representing claims (assertions about a subject, such as a user's identity) in a way that can be cryptographically verified. A JWT is issued by an authentication server and presented by the client on every subsequent request; the receiving server can verify the token's authenticity without contacting the issuing server.

**Token structure.** A JWT consists of three base64url-encoded sections separated by dots:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.
eyJ1c2VySWQiOjQyLCJlbWFpbCI6InVzZXJAZXhhbXBsZS5jb20iLCJpYXQiOjE3MTYwMDAwMDAsImV4cCI6MTcxNjA4NjQwMH0
.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

The first section is the **header**: a JSON object specifying the algorithm used for signing (e.g., HS256, which is HMAC-SHA256) and the token type ("JWT").

The second section is the **payload**: a JSON object containing the claims. In ProductiveSocial, the payload contains:
```json
{
  "userId": 42,
  "email": "user@example.com",
  "iat": 1716000000,
  "exp": 1716086400
}
```
`iat` is "issued at" (Unix timestamp), `exp` is the expiration time. Claims can be read by anyone — the payload is base64-encoded, not encrypted. It should never contain sensitive data like passwords.

The third section is the **signature**: computed by taking `HMAC-SHA256(header + "." + payload, secret)`. The secret is a private string known only to the servers. The signature cannot be forged without the secret; therefore, a valid signature proves that the token was created by a server that knows the secret, and that the payload has not been tampered with since it was issued.

**Validation.** When a service receives a request with a JWT in the `Authorization: Bearer <token>` header, it performs:
1. Decode the header and verify the algorithm is as expected
2. Decode the payload
3. Recompute `HMAC-SHA256(header + "." + payload, secret)` and compare to the provided signature — if they differ, reject
4. Check that `exp` is in the future — if expired, reject
5. Extract `userId` and `email` from the verified payload for use in the request handler

**Cross-service sharing.** The key insight in ProductiveSocial's authentication design is that JWT validation does not require contacting the issuer. Any server that knows the secret can independently verify any token signed with that secret. By giving all three user-facing services (selfmanager, timer, analytics) the same `JWT_SECRET`, a token issued by selfmanager's `/api/v1/auth/identify` endpoint is automatically valid in the timer and analytics services — no token exchange, no redirect, no additional login step.

This shared-secret approach is appropriate for a system where all services are operated by the same organization. It would not be appropriate for a federated system where services are operated by different organizations, in which case a more formal identity provider (OAuth2, OpenID Connect) would be required.

**Access token and refresh token.** The authentication flow issues two tokens: a short-lived access token (used for API calls) and a long-lived refresh token (used to obtain a new access token when the current one expires). The access token has a 24-hour validity; the refresh token has a 7-day validity. This design means a user does not need to re-enter their email every day, but a stolen access token has a bounded window of misuse.

---

#### 2.2.7 HikariCP: Database Connection Pooling

HikariCP is a high-performance JDBC connection pool for Java/Kotlin applications. A connection pool is a cache of database connections that are reused across requests rather than created and destroyed for each request.

Opening a database connection involves a TCP handshake with the database server, authentication, and the creation of a server-side session — typically 20–100 milliseconds of overhead. For an API endpoint that makes several database queries, this overhead on each request would add hundreds of milliseconds to every response time.

HikariCP maintains a pool of pre-opened connections (default: 10 connections). When the application needs to query the database, it borrows a connection from the pool (near-instant), executes the query, and returns the connection to the pool. The pool is configured with a maximum connection count (to avoid overloading the database) and a connection timeout (to fail fast if all connections are busy).

In ProductiveSocial, the `UserRegistry` uses a dedicated HikariCP pool pointing to the selfmanager database — separate from each service's main connection pool. This ensures that UserRegistry operations (which need a cross-database connection from the timer service to selfmanager's `users` table) do not contend with the timer service's own database queries.

---

#### 2.2.8 Kotlin Multiplatform (KMP): Cross-Platform Mobile Development

Kotlin Multiplatform (KMP) is a technology developed by JetBrains that allows Kotlin code to be compiled for multiple target platforms from a single codebase. Rather than writing separate implementations in Swift for iOS and Kotlin for Android, KMP allows the developer to write shared business logic in Kotlin once and compile it to both the JVM (for Android) and native machine code (for iOS via Kotlin/Native).

**The "multiplatform" model** distinguishes between:
- *commonMain*: code shared across all platforms — business logic, data models, use cases, repository interfaces, and network clients
- *androidMain*: Android-specific implementations (e.g., Android-specific dependency injection, platform file paths)
- *iosMain*: iOS-specific implementations

The key insight is that platform-specific code is as small as possible. In the ProductiveSocial mobile app, the API client, data models, sync logic, and business rules live in `commonMain`; only the platform-specific database drivers and UI integration points live in platform-specific source sets.

**Why KMP over React Native or Flutter?** The primary reason is that KMP shares code with the backend: the same data model classes used in the server-side Kotlin services can be directly referenced in the mobile app's `commonMain`. A `Task` data class defined in a shared module has identical field names and types on the server and the client, eliminating the class of bugs where server and client have diverged definitions of the same entity. Additionally, for an Android-first development team, KMP feels natural because the primary language (Kotlin) and tooling (Gradle, Android Studio) are already familiar.

**Compose Multiplatform** is the UI framework built on top of KMP. It extends Jetpack Compose (Google's modern UI toolkit for Android) to run on iOS and Desktop. The UI is written once in Kotlin using Compose's declarative paradigm and compiled to native UI components on each platform.

---

#### 2.2.9 SQLDelight: Type-Safe SQL for Kotlin Multiplatform

SQLDelight is a database library for Kotlin Multiplatform that generates type-safe Kotlin APIs from SQL statements. Instead of writing SQL strings at runtime (which are not checked by the compiler and can fail at runtime with opaque errors), the developer writes SQL in `.sq` files, and SQLDelight generates Kotlin functions that execute those queries with fully typed parameters and return values.

For example, the task query:

```sql
-- In tasks.sq
selectAllByUser:
SELECT id, sync_id, title, priority, completed, created_at
FROM tasks
WHERE user_id = :userId
ORDER BY created_at DESC;
```

Generates:
```kotlin
database.tasksQueries.selectAllByUser(userId = 42).executeAsList()
// returns List<SelectAllByUser> — a generated data class with typed fields
```

SQLDelight supports both Android (using Android's `SQLiteDriver`) and iOS (using `NativeSqliteDriver`), with the same query definitions and the same generated API on both platforms. This is the local database layer for the offline-first sync mechanism: all data created on the mobile device is written to SQLDelight first, and the network sync endpoint is called in the background.

---

#### 2.2.10 Streamlit: Rapid Dashboard Development

Streamlit is an open-source Python framework for building web applications from Python scripts without writing HTML, CSS, or JavaScript. A developer writes a Python script that calls Streamlit functions (`st.title()`, `st.dataframe()`, `st.button()`) and Streamlit renders them as interactive web components in a browser.

**Why Streamlit for the dashboards?** The admin and user dashboards in ProductiveSocial are operational interfaces — they provide access to the system's data for debugging, monitoring, and user operations. They are not intended to be the primary user-facing product (the KMP mobile app is). For this use case, Streamlit's rapid development speed is the decisive advantage: a full-featured admin dashboard with navigation, data tables, forms, and charts can be written in a single Python file of a few hundred lines.

The dashboards communicate with the backend services via HTTP calls using the `requests` library, authenticated with the same JWT that any API client would use. This design means the dashboards are genuine clients of the API — they exercise the same endpoints and receive the same responses as the mobile app — making them valuable integration tests in addition to operational tools.

**Streamlit's execution model.** Streamlit re-runs the entire Python script from top to bottom on every user interaction (button click, dropdown selection, etc.). State is preserved across re-runs using `st.session_state`, a dictionary that persists across executions within a user's browser session. This is why the authentication token is stored in `st.session_state["selfmanager_token"]` — it is set once at login and read on every subsequent page render.

---

#### 2.2.11 Koin: Dependency Injection for Kotlin

Dependency injection (DI) is a design pattern in which an object's dependencies are provided to it from outside rather than created internally. Instead of a service creating its own database connection and HTTP client, those dependencies are created elsewhere and injected into the service's constructor.

The benefits are: testability (in tests, real dependencies can be replaced with mocks), configurability (different environments can inject different implementations), and explicitness (the dependency graph is declared in one place rather than scattered across the codebase).

**Koin** is a lightweight DI framework for Kotlin. Unlike Spring's DI (which uses reflection and annotation processing), Koin uses a Kotlin DSL for declaring the dependency graph. This makes the graph readable and the compile-time feedback clearer.

In ProductiveSocial, each Kotlin service defines its module:

```kotlin
val appModule = module {
    single { DatabaseFactory(get()) }
    single { UserRepository(get()) }
    single { TaskRepository(get()) }
    single { HabitRepository(get()) }
    single { SyncService(get(), get(), get()) }
    single { HttpClient(CIO) { install(ContentNegotiation) { json() } } }
}
```

`single` means a singleton — one instance is created and reused. `get()` means "resolve from the container." When Koin creates `SyncService`, it automatically resolves the three repositories it needs from the container.

---

#### 2.2.12 Ollama: Local LLM Inference

Ollama is an open-source tool for running large language models locally on a developer's machine. It provides a simple command-line interface for downloading and running models (`ollama run llama3.2`) and exposes them via a REST API that is compatible with the OpenAI API format. This compatibility means that any client written against the OpenAI API can switch to Ollama by changing the base URL, with no other code changes.

**Why Ollama?** During development and testing, using a cloud LLM provider (Anthropic or OpenAI) for every test request would incur API costs and introduce network latency. Ollama allows the same LLM inference to run locally, privately, and for free. The model used (llama3.2 at 3B parameters) is small enough to run comfortably on a modern laptop's CPU or M-series GPU.

The limitation is that local Ollama inference is not accessible from cloud-hosted services (like the Render-deployed billing service). To bridge this gap in production testing, the local Ollama instance is exposed via an ngrok tunnel (described in the next section).

---

#### 2.2.13 Ngrok: Secure Tunneling for Local Services

Ngrok is a tool that creates a secure public HTTPS URL that forwards traffic to a service running on a local machine. When ngrok is running and pointed at `localhost:11434` (Ollama's port), it creates a tunnel through which any machine on the internet can send requests to the local Ollama instance.

The Render-hosted billing service's `OLLAMA_BASE_URL` environment variable is set to the ngrok HTTPS URL. When a user triggers an analysis, the billing service sends an inference request to this URL, which is routed through ngrok's infrastructure to the developer's machine, processed by Ollama, and the response is returned.

This arrangement is appropriate only for a proof-of-concept where one developer is testing the system. For a production system with multiple users making simultaneous requests, a proper cloud-hosted model serving infrastructure would be required.

The billing service's Ollama client includes two specific configurations for ngrok compatibility:

```python
headers = {
    "Content-Type": "application/json",
    "ngrok-skip-browser-warning": "true"  # Skip ngrok's browser warning page
}
response = requests.post(
    f"{ollama_base_url}/api/chat",
    json=payload,
    headers=headers,
    allow_redirects=True,  # Follow ngrok's redirect
    timeout=120
)
```

---

#### 2.2.14 Render: Cloud Deployment Platform

Render is a cloud platform for hosting web services, databases, and static sites. It provides a managed deployment environment in which the developer pushes code to a GitHub repository and Render automatically builds and deploys it.

For ProductiveSocial, each service is a separate Render Web Service connected to its own GitHub repository. The deployment process is:
1. The developer pushes a commit to the service's GitHub repository
2. Render detects the push via a webhook
3. Render builds the Docker image from the repository's `Dockerfile`
4. Render deploys the new image, replacing the running container

Render's free tier provides sufficient capacity for a proof-of-concept deployment. The notable limitation, described in Section 2.8, is that free-tier services are automatically spun down after 15 minutes of inactivity and incur a cold start penalty on the next request.

---

#### 2.2.15 Neon PostgreSQL: Serverless Managed Database

Neon is a serverless PostgreSQL platform. Like Render, it provides PostgreSQL databases without requiring the developer to manage database servers. Each database is automatically provisioned with connection pooling, point-in-time recovery, and branching (the ability to create a copy of the database at a specific point in time for testing or development).

Each of the four ProductiveSocial services has its own Neon database cluster. The `DATABASE_URL` environment variable for each service follows the standard PostgreSQL connection string format:

```
postgresql://user:password@ep-xxx.us-east-2.aws.neon.tech/selfmanager?sslmode=require
```

Neon's free tier is adequate for development and testing workloads. For production with real users, the paid tier would be necessary for guaranteed availability and connection limits.

---

#### 2.2.16 Kotlin Coroutines: Structured Concurrency

Kotlin coroutines are a language feature for writing asynchronous code that looks and behaves like sequential code. A coroutine is a unit of computation that can be suspended (waiting for a network response, a database result, or a timer) without blocking the thread it runs on.

**The threading problem.** In traditional synchronous code, every blocking operation (network call, database query) ties up a thread for the duration of the wait. For a server handling 100 concurrent requests that each make 3 network calls taking 200ms each, the naive implementation needs 100 threads just for in-flight requests — expensive in memory and scheduling overhead.

**The coroutine solution.** Coroutines are lightweight. Thousands of coroutines can run on a small pool of threads. When a coroutine is suspended (waiting for a response), the thread it was running on is freed to execute other coroutines. When the response arrives, the coroutine is resumed on any available thread.

For ProductiveSocial's analytics service, coroutines provide two specific benefits: the four upstream calls (tasks, habits, routines, timer stats) are launched as parallel coroutines and awaited together, cutting the total latency from the sum of all four response times to the maximum of any single response time. And if any upstream call fails, the structured concurrency model ensures that the failure is handled in the correct place without leaking coroutines.

---

#### 2.2.17 Gradle and the Shadow JAR

Gradle is the build system used for all Kotlin services. A **fat JAR** (also called a shadow JAR or uber JAR) is a single `.jar` file that contains the application code plus all of its dependencies bundled together. Normally, a JAR file contains only the application's own compiled classes and expects its dependencies to be on the classpath separately. A fat JAR is self-contained: it can be executed with a single `java -jar app.jar` command with no additional classpath configuration.

The `shadowJar` Gradle task (from the `com.github.johnrengelman.shadow` plugin) produces this artifact. The resulting file for each Kotlin service is approximately 20–30 MB (the application code plus Ktor, Exposed, HikariCP, and all transitive dependencies).

This design choice simplifies deployment: the Dockerfile for each Kotlin service simply copies the pre-built JAR and runs it. There is no compilation step in the Docker build — the image is built in seconds rather than minutes.

---

### 2.2 Summary Table

| Technology | Role | Why Chosen |
|---|---|---|
| Kotlin | Language for 3 backend services + mobile | Null safety, coroutines, multiplatform |
| Ktor | HTTP server/client framework | Lightweight, coroutine-native, plugin-based |
| Python | Language for billing service | Native ML ecosystem (sklearn, LLM SDKs) |
| FastAPI | HTTP framework for billing | Async-first, automatic OpenAPI docs, Pydantic |
| PostgreSQL | Database for all services | Reliability, JSON support, advanced indexing |
| Neon | Managed PostgreSQL in cloud | Free tier, no ops overhead |
| Docker | Containerization | Environment consistency, portability |
| Colima | Docker runtime on macOS | Lightweight, free, Docker-compatible |
| JWT | Authentication tokens | Stateless, cross-service verifiable |
| KMP | Mobile cross-platform framework | Single Kotlin codebase for Android + iOS |
| Compose Multiplatform | Shared UI for mobile | Unified UI codebase with KMP |
| SQLDelight | Mobile local database | Type-safe, KMP-compatible, offline storage |
| Streamlit | Admin and user dashboards | Rapid Python dashboard development |
| Koin | Dependency injection | Lightweight, Kotlin DSL, no reflection |
| Ollama | Local LLM inference | Free, private, development-friendly |
| Ngrok | Local→cloud tunnel | Exposes local Ollama to Render |
| Render | Cloud hosting | Free tier, git-push deploy |
| HikariCP | JDBC connection pooling | High performance, widely used |
| Gradle Shadow | Fat JAR build | Self-contained deployment artifact |

---

### 2.3 Data Strategy: Application-Generated Data

One of the most significant differences between the ProductiveSocial ML approach and a conventional machine learning project is the data strategy. This project does not use external datasets. No publicly available productivity dataset (Kaggle Time Management Insights, PMData, or similar) was used for training, validation, or prompt construction. No data preparation pipeline was built. No feature engineering was performed in the conventional ML sense.

This is a deliberate architectural decision, not a limitation, and understanding why requires understanding the fundamental difference between the two main approaches to ML-powered analysis:

**Approach A: Supervised prediction on population data.** Train a model on a dataset of many users' productivity data, with ground truth labels (e.g., "this user's productivity score today was 7.2/10 based on self-report"). Use the model to predict productivity scores or outcomes for new users based on their activity patterns. This approach requires a labeled dataset, ground truth definitions, feature engineering, training infrastructure, and hyperparameter tuning. It is not applicable at this stage of ProductiveSocial's development because no such dataset exists.

**Approach B: Contextual reasoning on user-specific data.** At the moment of analysis, collect the user's complete activity data and pass it to a language model that reasons about it using its pre-trained knowledge of human productivity patterns. No training is required; the model's reasoning capability is a function of its pre-training, and the quality of the analysis is a function of the richness of the context provided. This approach is directly applicable at any stage of development, requires no prior user data, and scales naturally with the amount of data available (a user with 30 days of history gets a better analysis than a user with 3 days).

ProductiveSocial uses Approach B. This is not a compromise; it is the correct approach for the current stage of the project. Approach A becomes viable — and preferable for certain prediction tasks — once real user behavioral data has accumulated at scale. The architecture explicitly preserves a pathway for this transition: the billing service's sklearn inference pathway, model registry, and `.pkl` upload endpoint are designed specifically to accommodate trained models without any changes to the broader system.

**Own data generation for testing.** Rather than using external datasets, a test user profile was manually constructed to represent a realistic and diverse user: three projects covering professional work, personal development, and skill learning; eight tasks of varying priorities, completion states, and time-spent values; six habits covering both positive behaviors to build and negative behaviors to quit; four routines representing the most common temporal anchors of a productive day; and 32 focus sessions spanning two weeks and all entity types. This test data serves as both a validation dataset and a demonstration of the system's analytical capability.

---

### 2.4 LLM-Based Analysis Pipeline

The analytics service's analysis pipeline is the central technical contribution of this project's ML integration. It implements a retrieval-augmented generation (RAG) pattern in which the "retrieval" step fetches live user data from the backend services rather than a vector database.

When a user submits an analysis request (`POST /api/v1/analytics` with an `AnalysisType` and `modelId`), the `AnalyticsService` executes the following sequence:

**Step 1: User context collection.** Three parallel HTTP calls are made using the internal service key:
- `GET /internal/users/{userId}/tasks` → selfmanager returns the user's task list with names, priorities, completion states, and time-spent values
- `GET /internal/users/{userId}/habits` → selfmanager returns habits with names, types (Start/Quit), recurrency, and time-spent values
- `GET /internal/users/{userId}/routines` → selfmanager returns routines with names and recurrency
- `GET /internal/users/{userId}/stats` → timer returns aggregated session statistics

These calls are made with a non-blocking architecture: if selfmanager or timer is unavailable, the corresponding data is set to an empty collection and the analysis proceeds with whatever data is available. Billing failure (the LLM inference step) is not recoverable — it surfaces as an error to the user.

**Step 2: Structured payload construction.** The collected data is assembled into a structured map that will become the LLM's context. The Kotlin implementation:

```kotlin
val inputData = buildMap<String, Any> {
    put("analysis_type", request.type.name)
    put("tasks", tasks.map {
        mapOf("name" to it.name, "priority" to it.priority,
              "completed" to it.completed,
              "timeSpentMinutes" to it.timeSpentMinutes)
    })
    put("habits", habits.map {
        mapOf("name" to it.name, "type" to it.habitType,
              "recurrency" to it.recurrency,
              "timeSpentMinutes" to it.timeSpentMinutes)
    })
    put("routines", routines.map {
        mapOf("name" to it.name, "recurrency" to it.recurrency)
    })
    pomodoroStats?.let {
        put("pomodoro", mapOf(
            "totalSessions" to it.totalSessions,
            "completedSessions" to it.completedSessions,
            "totalWorkMinutes" to it.totalWorkMinutes,
            "totalCycles" to it.totalCycles,
            "avgWorkMinutesPerSession" to it.avgWorkMinutesPerSession,
            "sessionsByEntityType" to it.sessionsByEntityType,
            "workMinutesByEntityType" to it.workMinutesByEntityType,
        ))
    }
    put("prompt", buildPrompt(request))
}
```

**Step 3: LLM inference via billing.** The structured payload is serialized to JSON and sent to `psocial_billing` via `POST /api/v1/internal/predict` with the X-Internal-Key header. The billing service receives the payload, verifies the user has sufficient credits, creates a PENDING prediction record, passes the data to the selected LLM provider, and returns the inference result along with the credits charged.

**Step 4: Report persistence.** The insight text extracted from the billing response is saved to the `analytics_reports` table with the user ID, analysis type, model ID, credit cost, and timestamp. The result is returned to the user.

---

### 2.5 Prompt Engineering and Analysis Types

The quality of an LLM analysis is determined largely by the specificity and appropriateness of the prompt. For each of the six analysis types, a distinct prompt was designed to focus the model's reasoning on the specific question the user is asking.

**PRODUCTIVITY_SUMMARY** uses the broadest prompt:
> "Based on the user's tasks, habits, routines and focus sessions, provide a concise productivity summary. Highlight what went well and what needs attention."

This prompt is intentionally open-ended, directing the model to synthesize all available context into an overview. The model is expected to identify patterns such as: many high-priority tasks pending despite significant focus time (potential prioritization issue), habits consistently completed on days with completed focus sessions but not on days without them (habit-focus correlation), or routines with low adherence compared to simple habits (structural friction in the routine).

**TASK_PRIORITIZATION** uses a more directive prompt:
> "Looking at the user's pending tasks and time spent so far, suggest a prioritized focus list for today. Be specific and actionable."

This prompt focuses the model on the subset of tasks that are incomplete and asks it to produce a ranked list rather than a narrative. The model is expected to weigh priority labels, deadline proximity, and time already invested.

**HABIT_INSIGHTS** focuses on behavioral patterns:
> "Analyze the user's habits. Which are consistent? Which are at risk? What one change would have the highest impact?"

The final question — "what one change would have the highest impact?" — is deliberately singular. It forces the model to prioritize rather than list all possible improvements, producing more actionable output.

**ROUTINE_OPTIMIZATION** focuses on process efficiency:
> "Review the user's routines and time spent. How can they be made more efficient? Suggest concrete improvements."

Routines are the most structured of the tracked entities (ordered steps with durations), and this prompt asks the model to reason about their structure rather than merely their adherence.

**WEEKLY_TIMER_SUMMARY** is the most constrained prompt:
> "Based on the user's Pomodoro session data, generate a concise weekly focus summary (3–4 sentences). Cover total focus time, session completion rate, which entity type they focused on most, and their most productive patterns. Keep the tone warm and forward-looking. End with one actionable suggestion for the coming week."

The explicit length constraint (3–4 sentences), the enumerated content requirements, and the tone instruction reflect lessons learned from early testing in which the model produced either too-brief summaries that omitted key data or too-long narratives that were not read in full by users.

**CUSTOM** accepts a user-supplied prompt, which is used as-is:
> [user-supplied text]

This analysis type is the most flexible and the least predictable in output quality, since the prompt quality depends entirely on the user. It is valuable for questions that do not fit the predefined types and for power users who want to ask specific questions about their data.

The system prompt stored in the model registry for the default Ollama model provides the overarching framing for all analysis types:
> "You are an expert productivity coach. You have access to a user's tasks, habits, routines, and focus session data. Based on this data, provide insightful, actionable, and personalized analysis."

This system prompt establishes the model's role and sets expectations for the output style (insightful, actionable, personalized) without constraining the content of any specific analysis type.

---

### 2.6 Evaluation Metrics

**For LLM analysis quality:**

The primary metric is a human evaluation score on a 1–5 scale along three dimensions:
- *Relevance*: Does the analysis address the specific question the analysis type is designed to answer?
- *Specificity*: Does the analysis reference specific items from the user's data (e.g., naming actual tasks or habits) rather than making generic observations?
- *Actionability*: Does the analysis provide at least one concrete, specific suggestion the user could act on?

The composite score is the average across the three dimensions. The baseline against which the LLM output is compared is a rule-based summary generated without LLM inference — a simple template-filled text that states counts and averages without interpretation (e.g., "You have 8 tasks, 3 completed. Your most-used habit is [name]. You spent 310 minutes in focus sessions.").

**For sync correctness:**

- *Sync completeness*: Percentage of records in the local test dataset that appear correctly in the server database after a sync operation
- *Idempotency*: Number of duplicate records created when the same sync request is submitted twice (target: 0)

**For credit accounting:**

- *Balance accuracy*: `balance_before - cost_per_prediction = balance_after` verified for every prediction in the test set

**For service availability:**

- *Cold start latency*: Time from first HTTP request to first successful response on Render (JVM services)
- *Warm latency*: Time for subsequent requests after the service is warmed up
- *Graceful degradation*: Behavior of analytics when selfmanager or timer is temporarily unavailable

---

### 2.7 Results Analysis

The following table documents all experiments conducted during the validation phase.

| # | Test | Baseline | Metric | Result | Conclusion |
|---|---|---|---|---|---|
| 1 | Cross-service JWT auth: selfmanager token accepted by analytics | N/A (architectural requirement) | Token accepted / rejected | ✅ Token accepted by analytics, timer; correct 401 returned for invalid tokens | JWT sharing across services is correctly implemented |
| 2 | Welcome credit deposit: new user receives 100 credits on first predict call | No prior balance (0 credits) | balance_after on first transaction | ✅ 100 credits deposited automatically; DEPOSIT transaction created | Lazy initialization works correctly |
| 3 | Credit deduction accuracy: balance_before − cost = balance_after | Initial balance: 100 credits; model cost: 5 credits | Mathematical equality | ✅ Post-prediction balance: 95 credits; transaction shows amount: -5, balance_after: 95 | Credit arithmetic is correct |
| 4 | Sync idempotency: same request submitted twice | No duplicates expected | Number of duplicate records | ✅ Second submission returned same idMappings with 0 new records created | Idempotency via clientId works correctly |
| 5 | Sync completeness: all 32 sessions synced after simulated disconnect | 32 sessions in test dataset | % of records in server DB | ✅ 32/32 sessions present; 55/55 habit completions; all 8 tasks | Sync is complete for tested dataset |
| 6 | PRODUCTIVITY_SUMMARY quality (LLM vs. baseline) | Rule-based: "You have 8 tasks, 3 completed…" (score: 1.5/5) | Human eval score 1–5 (relevance, specificity, actionability avg) | ✅ LLM score: 4.2/5; named specific tasks and habits; identified focus-habit correlation | LLM substantially outperforms rule-based baseline |
| 7 | HABIT_INSIGHTS quality (LLM vs. baseline) | Rule-based: "You have 6 habits. 5 are Start type." (score: 1.0/5) | Human eval score 1–5 | ✅ LLM score: 3.9/5; identified at-risk habits and suggested one specific change | LLM produces useful behavioral insights |
| 8 | WEEKLY_TIMER_SUMMARY quality (LLM vs. baseline) | Rule-based: "You had 32 sessions, 31 completed." (score: 1.5/5) | Human eval score 1–5 | ✅ LLM score: 4.0/5; covered completion rate, entity type breakdown, actionable next-week suggestion | Constrained prompt format produces consistently structured output |
| 9 | Graceful degradation: analytics when timer is unavailable | Analysis fails entirely | Analysis completes / fails | ✅ Analysis completes with pomodoro block omitted from context | Timer failures are non-blocking as designed |
| 10 | Service cold start on Render (JVM) | N/A | Time to first successful response | ~52 seconds average for JVM services; ~8 seconds for Python billing | JVM cold start is the main latency bottleneck on free tier |
| 11 | FAILED prediction: no credit deduction | Balance: 95 credits | balance unchanged after failed inference | ✅ Balance remained 95 after simulated provider error; FAILED prediction record, no CHARGE transaction | Failed predictions correctly exempt from billing |
| 12 | Admin deposit: credits added via internal API | User balance: 95 credits | balance_after on deposit | ✅ After depositing 50 credits: balance 145; DEPOSIT transaction recorded | Admin credit management works correctly |

**Sample LLM output for PRODUCTIVITY_SUMMARY** (condensed for report):

Input context (summarized): 8 tasks (3 completed: 2 high priority, 1 medium; 5 pending: 3 high, 2 medium), 6 habits (4 completed daily for past week, 2 inconsistent — both Start type), 4 routines (Morning: 85% adherence, Deep Work: 60%, others lower), 31 Pomodoro sessions (total 310 work minutes, 94% completion rate, primarily linked to Tasks).

LLM output (PRODUCTIVITY_SUMMARY via llama3.2):
> "Your task completion rate shows a clear pattern: high-priority items are getting done, but your backlog of 3 pending high-priority tasks suggests you may be hitting end-of-day fatigue before reaching them. Consider front-loading them earlier in the day, particularly since your focus session data shows your highest completion rate before 14:00. Your Deep Work routine at 60% adherence is your biggest friction point — the gap between its planned structure and actual execution suggests it may need to be shortened or simplified. Two habits — [Habit A] and [Habit B] — have not been completed on days where your focus session count was zero, indicating these habits may be momentum-dependent. Building a short Pomodoro session first thing in the morning may help anchor them."

**Rule-based baseline output (same input):**
> "You have 8 tasks, 3 completed and 5 pending. Your most active habit is [Habit C] with 7 completions. You have 4 routines. You completed 31 focus sessions totaling 310 minutes."

The qualitative difference between the two outputs is evident. The LLM output references specific data points, identifies a cross-domain correlation (habits and focus sessions), names a specific actionable change, and provides a concrete rationale. The baseline output is accurate but provides no insight beyond restating the input data.

---

### 2.8 Error Analysis and Lessons Learned

Several issues were encountered during development and testing that are worth documenting as they inform the final system design.

**Issue 1: LLM prompt did not receive structured data (resolved)**

In an early version of the integration, the billing service received the analytics request but the structured user data (tasks, habits, routines, Pomodoro stats) was not correctly passed through to the LLM prompt. The model received only the type-specific prompt text without any user data, producing a generic response with no reference to the user's actual context. The root cause was a mismatch between the expected JSON structure in the billing service's `PredictionService` and the structure being sent by the analytics service's `BillingClient`.

Resolution: The billing service's LLM prompt builder was updated to correctly extract and format all fields from the `inputData` map, and the analytics service's payload structure was aligned with the billing service's expectations. The fix was verified by confirming that the LLM output named specific user tasks and habits.

**Issue 2: Stale Docker image (resolved)**

After updating the analytics service's `AnalyticsService.kt` to add new analysis types, local testing continued to use an old Docker image that predated the changes. This was identified when the new analysis types were not available in the running container despite the code changes being present in the source.

Resolution: A consistent rebuild-before-test discipline was established, and the Docker Compose configuration was updated to use explicit image names that make version mismatches more visible.

**Issue 3: UserRegistry connection failure for new users on Render (known)**

The `psocial_selfmanager` service's UserRegistry — the raw JDBC connection to the shared `users` table — was failing on Render for new user registrations due to a bug in the `created_at` field initialization. Existing users were unaffected because their records already existed in the database. The bug was identified and the fix was applied locally; re-deployment to Render is pending at the time of writing.

**Issue 4: JVM cold starts on Render free tier**

Render's free tier spins down services after 15 minutes of inactivity. For the three JVM-based services (selfmanager, timer, analytics), the first request after a spindown incurs a ~50 second cold start delay. For the Python billing service, the cold start is approximately 8 seconds. This was mitigated in the user-facing dashboards by displaying a warning message on login indicating that services may take up to 60 seconds to wake up on first use.

**Issue 5: Ngrok tunnel required for Ollama on Render**

The local Ollama instance (running llama3.2) is not directly accessible from Render-hosted services. To enable production testing, the local Ollama instance is exposed via an ngrok HTTPS tunnel, and the billing service's `OLLAMA_BASE_URL` environment variable on Render points to the ngrok tunnel URL. The billing service's Ollama client was updated to include the `ngrok-skip-browser-warning` header and to follow redirects, both of which are required for the ngrok tunnel to function correctly. This setup is appropriate for a proof-of-concept but not for production use — a production deployment would use a cloud-hosted Ollama instance or switch to a hosted provider (Anthropic or OpenAI).

---

### 2.9 Proof-of-Concept Summary

The current implementation of ProductiveSocial constitutes a working proof-of-concept of the core value proposition: a unified productivity platform in which all modules share a common user identity and a central AI analysis layer has simultaneous access to cross-domain behavioral data.

The following capabilities have been demonstrated end-to-end:

1. A user can be identified with a single email address and receive a JWT that is valid across all three user-facing services (selfmanager, timer, analytics).

2. The user can create tasks, habits, and routines in selfmanager and track them using the selfmanager API. These entities are immediately available to the analytics service via the internal API.

3. The user can create and complete Pomodoro sessions in the timer, optionally linked to tasks, habits, or routines. Time spent is automatically written back to the linked entity in selfmanager. Focus pattern analytics are computed from the session history.

4. The user can request an AI analysis from any of six types. The analytics service collects the user's complete productivity context from selfmanager and timer, passes it to the billing service, which invokes the LLM, charges credits, and returns the insight. The report is persisted and displayed to the user.

5. Credits are managed transparently. Every prediction charges the model's cost from the user's balance and records a transaction. Insufficient credits block the analysis before any LLM call is made.

6. Both the admin and user dashboards provide complete interfaces to the above capabilities without requiring any direct API calls from the user.

What this proof-of-concept does not yet include:

- A production-grade mobile application (the KMP app is scaffolded but not feature-complete)
- Proprietary trained ML models (the system uses LLM prompting for all analysis)
- Social features (planned for a subsequent development phase)
- A journal module or notes module (deferred to post-MVP)
- Scalable LLM hosting (current Render deployment uses a personal Ollama instance via ngrok)

The architecture explicitly accommodates each of these future capabilities without structural changes to the existing services.

---

## Chapter 3: Architecture and Implementation

### 3.1 System Design Decisions and Architectural Principles

The architecture of ProductiveSocial was designed around a set of explicit principles that govern every significant decision in the system:

**Principle 1: Service independence over integration convenience.** Each service is designed to function correctly even when the other services are unavailable. The timer works without selfmanager (sessions can be standalone, and time-log calls are fire-and-forget). Selfmanager works without the timer (task and habit management is fully functional without Pomodoro data). Analytics degrades gracefully when selfmanager or timer is unavailable (it proceeds with whatever data it receives). Billing works without any other service (it only receives requests and processes them). This principle sacrifices some development convenience (it would be simpler to have a single monolithic service or to hard-fail on upstream errors) in exchange for a system that is resilient to partial failures.

**Principle 2: Single source of truth per domain.** User identity is owned by selfmanager (and the shared UserRegistry). Task, habit, and routine data is owned by selfmanager. Focus session data is owned by the timer. Credit and inference records are owned by billing. Analytics reports are owned by analytics. No service duplicates the data owned by another service — it retrieves the data via internal API calls at the time it needs it. This prevents the consistency problems that arise when the same data is replicated in multiple services.

**Principle 3: All external access through authenticated endpoints, all internal access through a shared secret.** The boundary between "client-facing" and "service-to-service" is enforced at the HTTP layer. Client requests carry a JWT in the `Authorization: Bearer` header. Service-to-service requests carry the shared internal key in the `X-Internal-Key` header. Billing has no client-facing access whatsoever — all its endpoints require the internal key. This means that even if billing's URL is somehow disclosed, no client can interact with it without possessing the internal key, which is stored only in environment variables on the server.

**Principle 4: Offline-first data ownership.** Mobile clients own their data locally. The server is the synchronization target, not the source of truth during offline operation. This requires client-assigned UUIDs for all entities (so the client can create records without waiting for a server response), idempotent sync operations (so retrying a failed sync never corrupts data), and explicit tombstone tracking for deletions (so the client knows which records to remove from its local database).

**Principle 5: The credit system as a behavioral incentive.** The decision to implement a credit system for AI analysis serves a purpose beyond cost accounting. By making the cost of each analysis visible and deducting it from a finite balance, the system creates a natural incentive for users to use analysis features deliberately rather than compulsively. This is consistent with the platform's philosophy of reducing cognitive overhead rather than increasing it. The welcome credit deposit (100 credits on first use) removes the onboarding barrier, and the admin's ability to top up credits ensures that access is not gated by payment for users who need it.

---

### 3.2 Microservices Architecture

The ProductiveSocial backend consists of four microservices, each with its own database, codebase, and deployment configuration.

#### psocial_selfmanager (Kotlin/Ktor, port 1226)

selfmanager is the identity and productivity data service. It is the central hub of the platform's data model and owns the `users` table that the entire system relies on for user identity.

The service is built on Ktor with Exposed (ORM), PostgreSQL (via JDBC and HikariCP), and Koin for dependency injection. It exposes a REST API over port 1226, with JWT authentication applied to all data endpoints.

The data model consists of: Users, RefreshTokens, Projects, Tasks, Subtasks, TaskScheduledTimes, Habits, HabitSubtasks, HabitScheduledTimes, HabitReminderTimes, HabitCompletions, HabitSubtaskCompletions, Routines, RoutineSteps, RoutineScheduledTimes, RoutineReminderTimes, RoutineCompletions, RoutineStepCompletions, and Tags (with junction tables for many-to-many relationships). Each entity carries a client-assigned `syncId` UUID for offline sync, a server-assigned integer `id` for efficient join operations, and standard `createdAt`/`updatedAt` timestamps.

The offline sync endpoint (`POST /api/v1/sync`) accepts a batched `SyncRequest` containing creates, updates, and deletes for all entity types. It processes creates and updates in dependency order (projects before tasks, habits before completions), uses `ON CONFLICT (sync_id) DO UPDATE` semantics for idempotency, and builds a `SyncResponse` containing `idMappings` (client UUID → server integer ID), `serverChanges` (entities modified server-side since `lastSyncedAt`), and per-item `errors`. The tombstone table records the server IDs of all deleted entities so that clients can purge their local database.

Internal endpoints expose user data for the analytics service: `GET /internal/users/{id}/tasks`, `/habits`, `/routines`. These endpoints are protected by the X-Internal-Key header and return minimal representations optimized for prompt construction rather than full entity responses.

The `POST /internal/time-log` endpoint receives time-log events from the timer service and adds the reported work minutes to the specified entity (task or habit, optionally a specific subtask). This is processed as a database update in a transaction that atomically adds the new minutes to the existing cumulative total.

#### psocial_timer (Kotlin/Ktor, port 1227)

The timer service owns the Pomodoro session lifecycle and all focus analytics. Like selfmanager, it uses Ktor with Exposed and PostgreSQL.

The data model consists of: PomodoroSettings (one row per user), PomodoroSessions, and PomodoroIntervals. A session carries: user ID, optional entity link (type + ID), session status (Active, Paused, Completed, Abandoned), accumulated `totalWorkMinutes`, `completedCycles`, `pauseCount`, `startedAt`, `completedAt`, and a snapshot of the user's settings at session creation time. Each interval carries: session reference, interval type (Work, ShortBreak, LongBreak), status (InProgress, Completed, Abandoned), planned duration (from the settings snapshot), actual duration (computed at completion), and timestamps.

The session lifecycle logic is implemented in `PomodoroService`:

*Interval sequencing*: On `startInterval`, the service computes the next interval type by counting how many work intervals have been completed in the current session. If `completedWorkCount % cyclesUntilLongBreak == 0` and `completedWorkCount > 0`, the next interval is a LongBreak; otherwise it's a ShortBreak. When a work interval is completed, the service increments `completedCycles` when the work-to-break cycle is complete.

*Break suggestion logic*: After each completed work interval, the service computes total work minutes across all of today's sessions for the user, checks whether a long break has already occurred in the current session, and returns one of three suggestions: ExtendedRest (≥90 minutes of focus today, no long break yet), LongBreak (cycle count dictates), or ShortBreak (default).

*Abandonment risk detection*: On pause, the service queries the user's abandoned session history to compute a personal threshold. If the user has ≥3 abandoned sessions, the threshold is the average pause count at abandonment; otherwise it defaults to 3. Additionally, if the session has been open for more than 3× its planned total work duration with zero completed cycles, a stale session warning is generated.

*Duration recommendations*: The settings endpoint computes personalized work, short break, and long break recommendations by averaging the actual duration of completed intervals of each type. This requires a minimum of 5, 3, and 3 intervals respectively to avoid statistically meaningless averages.

The `GET /internal/users/{userId}/stats` endpoint aggregates session data for the analytics service: total sessions, completed sessions, total work minutes, total cycles, average work minutes per session, and per-entity-type breakdowns.

#### psocial_analytics (Kotlin/Ktor, port 1228)

The analytics service is the most compositionally complex of the four services. It orchestrates calls to three upstream services, constructs the LLM context, invokes billing, persists results, and exposes them to users and admins.

The data model is minimal: Users (mirroring selfmanager's integer user IDs with local email and admin flag) and AnalyticsReports (type, model ID, insight text, credits charged, timestamp).

The service uses three HTTP clients: `SelfmanagerClient`, `TimerClient`, and `BillingClient`. Each client uses Ktor's `HttpClient` with the `Json` feature for content negotiation and a connection timeout appropriate to the expected response time of each upstream service. The billing client uses a 2-minute timeout (configurable) to accommodate cold LLM inference.

The `AnalyticsService.analyze()` method:
1. Ensures the user exists in the local database (creates or updates the user record from JWT claims)
2. Calls selfmanager and timer in parallel using Kotlin coroutines (`async { }` blocks joined with `awaitAll()`)
3. Constructs the `inputData` map with all collected context and the type-specific prompt
4. Calls `billingClient.predict()` with a 2-minute timeout
5. Extracts the insight text and credits charged from the billing response
6. Persists the report and returns the result

The admin endpoints (`/api/v1/admin/*`) are protected by a middleware that extracts the email claim from the JWT and checks it against the `ADMIN_EMAILS` environment variable. No database query is required for this check, making admin status changes (via environment variable update) take effect immediately on the next request.

The credit proxy endpoints (`/api/v1/credits/*`) extract the user's integer ID from the JWT claims and forward the request to the corresponding billing internal endpoint, substituting the JWT authentication with the X-Internal-Key header. This is the mechanism by which clients can check balances and view transactions without ever calling billing directly.

#### psocial_selfmanager — API Endpoint Reference

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/api/v1/auth/identify` | None | Find-or-create user, issue JWT pair |
| POST | `/api/v1/auth/refresh` | Refresh token | Exchange refresh token for new access token |
| GET | `/api/v1/projects` | JWT | List all projects for authenticated user |
| POST | `/api/v1/projects` | JWT | Create a new project |
| GET | `/api/v1/tasks/tasks` | JWT | List all tasks (paginated) |
| POST | `/api/v1/tasks/tasks` | JWT | Create a task |
| PUT | `/api/v1/tasks/tasks/{id}` | JWT | Update a task |
| DELETE | `/api/v1/tasks/tasks/{id}` | JWT | Delete a task |
| GET | `/api/v1/habits/habits` | JWT | List all habits |
| POST | `/api/v1/habits/habits` | JWT | Create a habit |
| POST | `/api/v1/habits/habits/{id}/complete` | JWT | Log a habit completion |
| GET | `/api/v1/routines/routines` | JWT | List all routines |
| POST | `/api/v1/routines/routines` | JWT | Create a routine |
| POST | `/api/v1/routines/routines/{id}/complete` | JWT | Log a routine completion |
| POST | `/api/v1/sync` | JWT | Batch offline sync (all entity types) |
| GET | `/internal/users/{id}/tasks` | X-Internal-Key | Internal: tasks for analytics |
| GET | `/internal/users/{id}/habits` | X-Internal-Key | Internal: habits for analytics |
| GET | `/internal/users/{id}/routines` | X-Internal-Key | Internal: routines for analytics |
| POST | `/internal/time-log` | X-Internal-Key | Internal: add work minutes to entity |

#### psocial_timer — API Endpoint Reference

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/api/v1/pomodoro/settings` | JWT | Get user's timer settings and personalized recommendations |
| PUT | `/api/v1/pomodoro/settings` | JWT | Update timer settings |
| GET | `/api/v1/pomodoro/sessions` | JWT | List sessions (optionally filtered by entity) |
| POST | `/api/v1/pomodoro/sessions` | JWT | Create a new session |
| POST | `/api/v1/pomodoro/sessions/{id}/start-interval` | JWT | Start the next work or break interval |
| POST | `/api/v1/pomodoro/sessions/{id}/complete-interval` | JWT | Mark current interval as completed |
| POST | `/api/v1/pomodoro/sessions/{id}/abandon` | JWT | Abandon the current session |
| POST | `/api/v1/pomodoro/sessions/{id}/pause` | JWT | Pause session (triggers abandonment risk check) |
| POST | `/api/v1/pomodoro/sessions/{id}/resume` | JWT | Resume a paused session |
| GET | `/api/v1/pomodoro/insights/focus-patterns` | JWT | Focus patterns by hour of day and day of week |
| GET | `/api/v1/pomodoro/insights/entity-stats` | JWT | Time spent per entity type and entity ID |
| GET | `/internal/users/{id}/stats` | X-Internal-Key | Internal: aggregated stats for analytics |

#### psocial_analytics — API Endpoint Reference

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/api/v1/analytics` | JWT | Run an AI analysis, persist and return report |
| GET | `/api/v1/analytics` | JWT | List all stored reports for user (newest first) |
| GET | `/api/v1/credits/balance` | JWT | Current credit balance (proxied from billing) |
| GET | `/api/v1/credits/transactions` | JWT | Transaction history (proxied from billing) |
| GET | `/api/v1/admin/stats` | JWT + Admin | System-wide totals (users, reports, credits) |
| GET | `/api/v1/admin/users` | JWT + Admin | All users with report counts |
| GET | `/api/v1/admin/reports` | JWT + Admin | All reports across all users |
| GET | `/api/v1/admin/users/{id}/reports` | JWT + Admin | Reports for a specific user |
| DELETE | `/api/v1/admin/users/{id}/reports` | JWT + Admin | Bulk-delete all reports for a user |
| PATCH | `/api/v1/admin/users/{id}/toggle-admin` | JWT + Admin | Toggle admin flag on analytics user record |
| POST | `/api/v1/admin/seed-reports` | JWT + Admin | Insert sample reports (bypass billing) |
| GET | `/internal/users/{id}/stats` | X-Internal-Key | Internal: user report count and last report time |

#### psocial_billing — API Endpoint Reference

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/api/v1/models` | None (public) | List all active models |
| POST | `/api/v1/internal/predict` | X-Internal-Key | Run inference, charge credits, return result |
| GET | `/api/v1/internal/balance/{user_id}` | X-Internal-Key | Get user's current credit balance |
| GET | `/api/v1/internal/transactions/{user_id}` | X-Internal-Key | Get user's transaction history |
| POST | `/api/v1/internal/deposit` | X-Internal-Key | Deposit credits for a user (admin only) |
| POST | `/api/v1/internal/models` | X-Internal-Key | Register a new model |
| PUT | `/api/v1/internal/models/{id}` | X-Internal-Key | Update model configuration |
| GET | `/api/v1/internal/predictions` | X-Internal-Key | List all predictions (admin monitoring) |

---

#### psocial_billing (Python/FastAPI, port 1229)

The billing service is architecturally the simplest but technically the most critical service in the system. It has no JWT authentication, no public endpoints (except `GET /api/v1/models` which is intentionally public), and no user table.

The data model consists of: MLModel (registered models with all configuration including system prompt), Transaction (credit movements keyed by `selfmanager_user_id` with type, amount, description, linked prediction ID, and `balance_after`), and Prediction (inference records with model reference, user ID, input data, output data, status, error message, and timestamps).

The `BillingService.get_balance()` method queries for the single most recent transaction for the user and returns its `balance_after` field. If no transactions exist, it returns 0. This design avoids maintaining a separate balance column that could become inconsistent with the transaction log, at the cost of an indexed query on `selfmanager_user_id` ordered by `created_at` descending with `LIMIT 1`. The `selfmanager_user_id` column is indexed explicitly to keep this query O(log n) in the number of users.

The `ensure_welcome_credits()` method is called at the beginning of every `predict` call. It counts the number of existing transactions for the user; if the count is zero, it creates a DEPOSIT transaction for `DEFAULT_CREDITS_ON_REGISTER` credits before proceeding. This lazy initialization means that welcome credits are created on the user's first attempt to use the AI analysis feature, not at registration — a design choice that avoids creating billing records for users who never use AI features.

The `PredictionService.run()` method:
1. Fetches and validates the model (must exist and be active)
2. Calls `ensure_welcome_credits()`
3. Checks the user's balance against the model's cost
4. Creates a PENDING prediction record
5. Dispatches to the appropriate inference method (LLM or sklearn)
6. On success: creates a CHARGE transaction, updates the prediction to SUCCESS
7. On failure: updates the prediction to FAILED (no transaction created)

The LLM inference methods construct the prompt from the `input_data` payload by formatting all structured fields (tasks, habits, routines, Pomodoro stats) into readable text blocks, combining them with the `prompt` field from the payload, and adding the model's `system_prompt` as the LLM system message.

For Ollama, the service calls the OpenAI-compatible `/api/chat` endpoint on the Ollama host, passing the system and user messages. The `ngrok-skip-browser-warning: true` header and redirect following are included to support the ngrok tunnel used in the Render deployment.

For Anthropic and OpenAI, the respective official SDK clients are used.

---

### 3.3 Authentication and Cross-Service Identity

The authentication architecture solves a non-trivial problem: how to give a user a single identity that is valid across multiple independent services without requiring a centralized identity provider or complex token exchange protocols.

#### The Login Flow

When a user first opens the dashboard or mobile application, they provide only their email address. The system uses a "find-or-create" identity model: if the email exists in the database, the user is returned to their existing account; if not, a new account is created. There is no password and no email verification step — this is intentional for the proof-of-concept phase where the primary concern is demonstrating functionality rather than production-grade security.

The login request is sent to selfmanager:

```
POST /api/v1/auth/identify
Content-Type: application/json

{"email": "user@example.com"}
```

selfmanager finds or creates the user in its database, generates an access token and a refresh token, and returns:

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "userId": 42,
  "email": "user@example.com"
}
```

The client stores both tokens. The `accessToken` is attached to every subsequent API request as an HTTP header: `Authorization: Bearer <accessToken>`. The `refreshToken` is used only when the access token expires.

#### JWT Sharing: One Token for All Services

All three user-facing services (selfmanager, timer, analytics) are configured with the same `JWT_SECRET`, `JWT_ISSUER`, `JWT_AUDIENCE`, and `JWT_REALM` environment variables. These four values fully define the JWT validation context:

- `JWT_SECRET`: The cryptographic key used to sign and verify tokens. Must be a long, random string (minimum 32 bytes). Never committed to source code.
- `JWT_ISSUER`: A string identifying the issuing authority (e.g., `"psocial"`). The receiving service checks that the token's `iss` claim matches.
- `JWT_AUDIENCE`: A string identifying the intended recipient (e.g., `"psocial-api"`). The receiving service checks that the token's `aud` claim includes this value.
- `JWT_REALM`: A string used in the `WWW-Authenticate` header when a request is rejected due to invalid auth (e.g., `"psocial api"`).

Because all three services share these values, a token signed by selfmanager passes validation on the timer and analytics services. The services do not check *who issued* the token — they check only that the signature is valid (which it is, because all share the same secret) and that the claims are well-formed. This is the fundamental mechanism of cross-service authentication without a centralized identity provider.

The Ktor JWT plugin configuration in each service:

```kotlin
install(Authentication) {
    jwt("jwt") {
        realm = System.getenv("JWT_REALM")
        verifier(
            JWT.require(Algorithm.HMAC256(System.getenv("JWT_SECRET")))
                .withIssuer(System.getenv("JWT_ISSUER"))
                .withAudience(System.getenv("JWT_AUDIENCE"))
                .build()
        )
        validate { credential ->
            val userId = credential.payload.getClaim("userId").asInt()
            val email = credential.payload.getClaim("email").asString()
            if (userId != null && email != null) {
                JWTPrincipal(credential.payload)
            } else null
        }
    }
}
```

On every request, the Ktor JWT plugin automatically extracts the token, validates it, and makes the `userId` and `email` claims available to the route handler via `call.principal<JWTPrincipal>()`.

#### The UserRegistry: Canonical Integer Identity

The more subtle identity problem is ensuring that the same integer `user_id` is assigned to the same email address regardless of which service first encounters the user. This matters because cross-service data linkage depends on a shared integer key: if a user's Pomodoro sessions are stored with `user_id = 42` in the timer database but their tasks are stored with `user_id = 99` in the selfmanager database, the analytics service cannot join these two datasets.

The `UserRegistry` solves this by having all services that need user IDs connect directly to selfmanager's `users` table as the canonical source. The timer service has a dedicated HikariCP connection pool pointing to selfmanager's PostgreSQL database (configured via `USER_REGISTRY_DB_*` environment variables). When the timer service receives a request with a JWT, it extracts the email claim and runs:

```sql
INSERT INTO users (email, created_at)
VALUES (?, NOW())
ON CONFLICT (email) DO UPDATE SET email = EXCLUDED.email
RETURNING id
```

This upsert either creates the user (returning the new ID) or finds the existing user (returning the existing ID). The result is always the canonical integer ID for that email — the same ID that selfmanager would return for the same email. The `RETURNING id` clause is a PostgreSQL feature that returns the value of the ID column for the row that was inserted or updated.

The analytics service does not use the UserRegistry directly — instead, it extracts `userId` from the JWT claims (which selfmanager already validated and encoded) and uses that value directly. This works because the JWT's `userId` claim was set by selfmanager when the token was issued, and selfmanager is the authoritative source of user IDs.

The only cross-database connection in the entire system is the timer's UserRegistry connection to selfmanager's database. This is an explicit and documented dependency, its purpose is narrow (one query: find-or-create user by email), and it has a dedicated connection pool to prevent it from interfering with the timer's normal database operations.

#### X-Internal-Key Authentication: Service-to-Service

For calls between services (analytics calling selfmanager for user data, timer calling selfmanager for time-log updates, analytics calling billing for inference), a different authentication mechanism is used: a pre-shared key in the `X-Internal-Key` HTTP header.

The key reason for using a separate mechanism for service-to-service calls rather than reusing JWTs is that internal endpoints expose data in aggregate (e.g., "all tasks for user 42") or perform operations that bypass business rules (e.g., seed test data in billing). These endpoints should only be reachable by trusted services, not by any user with a valid JWT. Using a separate header with a separate key makes this boundary explicit.

The `INTERNAL_API_KEY` environment variable is set to the same value across all services. When analytics calls `GET /internal/users/{userId}/tasks` on selfmanager, it includes `X-Internal-Key: <key>` in the request. selfmanager validates this header before executing the handler. If the header is missing or the key doesn't match, the response is 401 Unauthorized — regardless of whether the request also carries a valid JWT.

The internal key is never committed to source code, never returned in any API response, and not documented in the public Swagger UI. It is stored exclusively in environment variables on the server.

---

### 3.4 Data Model and Storage Strategy

Each service owns its own PostgreSQL database with a schema designed for that service's specific data access patterns. This section describes the key tables and the reasoning behind the most significant design decisions.

#### Selfmanager Database Schema

The selfmanager database is the largest and most complex in the system. Its core design principle is that tasks, habits, and routines are parallel entity types: they share a common structure (owner, project, name, sync ID, timestamps, tags, scheduled times) while having type-specific extensions.

```sql
-- Core user identity table — canonical source of truth for user IDs
CREATE TABLE users (
    id          SERIAL PRIMARY KEY,
    email       VARCHAR(255) UNIQUE NOT NULL,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- JWT refresh tokens
CREATE TABLE refresh_tokens (
    id          SERIAL PRIMARY KEY,
    user_id     INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token       TEXT NOT NULL UNIQUE,
    expires_at  TIMESTAMPTZ NOT NULL,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Projects group tasks, habits, and routines
CREATE TABLE projects (
    id          SERIAL PRIMARY KEY,
    sync_id     UUID NOT NULL UNIQUE,
    user_id     INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    name        VARCHAR(255) NOT NULL,
    color       VARCHAR(50),
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Tasks
CREATE TABLE tasks (
    id                  SERIAL PRIMARY KEY,
    sync_id             UUID NOT NULL UNIQUE,
    user_id             INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    project_id          INTEGER REFERENCES projects(id) ON DELETE SET NULL,
    title               VARCHAR(500) NOT NULL,
    description         TEXT,
    priority            VARCHAR(20) NOT NULL DEFAULT 'MEDIUM', -- LOW, MEDIUM, HIGH, CRITICAL
    completed           BOOLEAN NOT NULL DEFAULT FALSE,
    time_spent_minutes  INTEGER NOT NULL DEFAULT 0,
    due_date            TIMESTAMPTZ,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Habits (behavioral routines tracked for consistency)
CREATE TABLE habits (
    id                  SERIAL PRIMARY KEY,
    sync_id             UUID NOT NULL UNIQUE,
    user_id             INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    project_id          INTEGER REFERENCES projects(id) ON DELETE SET NULL,
    name                VARCHAR(255) NOT NULL,
    habit_type          VARCHAR(10) NOT NULL,  -- 'START' (build) or 'QUIT' (break)
    recurrency          VARCHAR(20) NOT NULL,  -- DAILY, WEEKLY, etc.
    time_spent_minutes  INTEGER NOT NULL DEFAULT 0,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- One record per day-habit pair when user marks a habit complete
CREATE TABLE habit_completions (
    id          SERIAL PRIMARY KEY,
    habit_id    INTEGER NOT NULL REFERENCES habits(id) ON DELETE CASCADE,
    user_id     INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    completed_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Tags — unique per user, auto-created on assignment
CREATE TABLE tags (
    id      SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    name    VARCHAR(100) NOT NULL,
    UNIQUE (user_id, name)
);

-- Many-to-many: tasks ↔ tags
CREATE TABLE task_tags (
    task_id INTEGER NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
    tag_id  INTEGER NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
    PRIMARY KEY (task_id, tag_id)
);
```

The `sync_id` UUID column deserves explanation. In an offline-first system, a mobile client must be able to create a record locally and assign it an identifier before the server has ever seen it. Integer IDs assigned by the database (`SERIAL PRIMARY KEY`) cannot be used for this purpose because the client does not know what integer the server will assign. UUIDs solve this: the client generates a random UUID (using the standard UUID v4 format) at the moment of local record creation, stores the record locally under that UUID, and includes it in the sync request. The server's `ON CONFLICT (sync_id) DO UPDATE` semantics ensure that if the same UUID is submitted twice (e.g., because the first sync attempt timed out and was retried), no duplicate is created.

The server-assigned integer `id` is used for all join operations in the database and all internal service-to-service references (the timer references a task by its integer `id`, not its UUID). The UUID is used only for client-server synchronization identity.

#### Timer Database Schema

```sql
-- One settings row per user — created on first interaction with the timer
CREATE TABLE pomodoro_settings (
    id                      SERIAL PRIMARY KEY,
    user_id                 INTEGER NOT NULL UNIQUE,
    work_duration_minutes   INTEGER NOT NULL DEFAULT 25,
    short_break_minutes     INTEGER NOT NULL DEFAULT 5,
    long_break_minutes      INTEGER NOT NULL DEFAULT 15,
    cycles_until_long_break INTEGER NOT NULL DEFAULT 4,
    updated_at              TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- One session = one focused work block (may contain multiple intervals)
CREATE TABLE pomodoro_sessions (
    id                      SERIAL PRIMARY KEY,
    user_id                 INTEGER NOT NULL,
    entity_type             VARCHAR(20),    -- 'TASK', 'HABIT', 'ROUTINE', or NULL
    entity_id               INTEGER,        -- server-side integer ID of the linked entity
    status                  VARCHAR(20) NOT NULL DEFAULT 'ACTIVE',
    total_work_minutes      INTEGER NOT NULL DEFAULT 0,
    completed_cycles        INTEGER NOT NULL DEFAULT 0,
    pause_count             INTEGER NOT NULL DEFAULT 0,
    -- Settings snapshot — preserved so historical analysis reflects original intent
    work_duration_snapshot  INTEGER NOT NULL,
    short_break_snapshot    INTEGER NOT NULL,
    long_break_snapshot     INTEGER NOT NULL,
    cycles_snapshot         INTEGER NOT NULL,
    started_at              TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    completed_at            TIMESTAMPTZ
);

-- Each work/break interval within a session
CREATE TABLE pomodoro_intervals (
    id              SERIAL PRIMARY KEY,
    session_id      INTEGER NOT NULL REFERENCES pomodoro_sessions(id) ON DELETE CASCADE,
    interval_type   VARCHAR(15) NOT NULL,  -- 'WORK', 'SHORT_BREAK', 'LONG_BREAK'
    status          VARCHAR(15) NOT NULL DEFAULT 'IN_PROGRESS',
    planned_minutes INTEGER NOT NULL,
    actual_minutes  INTEGER,               -- NULL until interval completes or is abandoned
    started_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    ended_at        TIMESTAMPTZ
);
```

The **settings snapshot** pattern (columns `work_duration_snapshot`, `short_break_snapshot`, etc.) preserves the user's settings at session creation time. If the user later changes their work duration from 25 to 50 minutes, historical sessions should still be analyzed against the 25-minute standard that was in effect when they were created — the percentage of sessions that met their planned duration would be meaningless if the planned duration kept changing retroactively.

#### Billing Database Schema

```sql
-- Registered ML/LLM models available for inference
CREATE TABLE ml_models (
    id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name          VARCHAR(255) NOT NULL,
    provider      VARCHAR(50) NOT NULL,   -- 'OLLAMA', 'ANTHROPIC', 'OPENAI', 'SKLEARN'
    model_name    VARCHAR(255) NOT NULL,  -- 'llama3.2', 'claude-sonnet-4-6', 'gpt-4o', etc.
    cost_per_use  INTEGER NOT NULL DEFAULT 5,
    system_prompt TEXT,
    is_active     BOOLEAN NOT NULL DEFAULT TRUE,
    created_at    TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Credit transaction ledger — every credit movement is a row here
CREATE TABLE transactions (
    id                   UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    selfmanager_user_id  INTEGER NOT NULL,
    transaction_type     VARCHAR(20) NOT NULL,  -- 'DEPOSIT', 'CHARGE', 'REFUND'
    amount               INTEGER NOT NULL,       -- positive for deposits, negative for charges
    description          TEXT,
    prediction_id        UUID REFERENCES predictions(id),
    balance_after        INTEGER NOT NULL,       -- running balance AFTER this transaction
    created_at           TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Index enables O(log n) balance lookup (most recent transaction for user)
CREATE INDEX idx_transactions_user_created
ON transactions (selfmanager_user_id, created_at DESC);

-- One record per inference attempt
CREATE TABLE predictions (
    id                   UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    model_id             UUID NOT NULL REFERENCES ml_models(id),
    selfmanager_user_id  INTEGER NOT NULL,
    status               VARCHAR(20) NOT NULL DEFAULT 'PENDING',
    input_data           JSONB,            -- full user context sent to LLM
    output_data          JSONB,            -- full LLM response
    error_message        TEXT,
    credits_charged      INTEGER,
    created_at           TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    completed_at         TIMESTAMPTZ
);
```

**Why JSONB for `input_data` and `output_data`?** JSONB is PostgreSQL's binary JSON type, which stores JSON in a decomposed format that enables indexing and efficient querying. The full LLM input (the user's complete productivity context) and the full LLM output are stored as JSON documents. This preserves the complete record of what was sent and what was received, enabling debugging and future analysis of LLM performance. The binary format also allows queries like "find all predictions where the input contained a task named X," which would be difficult with TEXT storage.

**The transaction-based balance design** eliminates the possibility of balance inconsistency. In a design with a separate `balance` column on a users table, a system crash between "deduct 5 credits" and "record the transaction" would leave the balance and transaction log inconsistent. In the transaction-log design, the balance *is* the transaction log: the current balance is `SELECT balance_after FROM transactions WHERE selfmanager_user_id = ? ORDER BY created_at DESC LIMIT 1`. If the transaction row was written, the balance is correct. If the transaction row was not written (due to a crash), the balance is the same as before — consistent with the prior transaction. There is no separate state to get out of sync.

---

### 3.5 Service Implementation

**Development workflow.** All Kotlin services follow the same project structure: a multi-module Gradle project with a `:server` submodule containing the application code. The server module builds to a fat JAR via the `shadowJar` task, which bundles all dependencies. The `Dockerfile` for each Kotlin service copies the pre-built fat JAR rather than building it in the container, keeping the Docker build time under 10 seconds.

The Python billing service uses a single-module FastAPI application with `alembic` for database migrations (replaced in the current development setup by SQLAlchemy's `create_all()` called at startup, with an optional `RESET_DB=true` flag for schema resets).

**Dependency injection.** All Kotlin services use Koin for dependency injection, configured in a `KoinModule.kt` file that registers all services and HTTP clients as singletons. This makes testing straightforward (Koin modules can be replaced with test-specific bindings) and makes the dependency graph explicit.

**Error handling.** Each service defines a set of application-specific exceptions (e.g., `PaymentRequiredException`, `NotFoundException`, `UnauthorizedException`) that are caught by a global Ktor status page plugin and converted to appropriate HTTP responses with structured error bodies. This ensures that API consumers receive consistent error formats rather than raw stack traces.

**Logging.** Ktor's built-in `CallLogging` plugin logs all incoming requests with their status codes and response times. This is sufficient for the current proof-of-concept stage. In a production deployment, structured logging (e.g., JSON-formatted log lines) and log aggregation (e.g., Datadog, Logtail) would be added.

---

### 3.6 KMP Mobile Application

The Kotlin Multiplatform mobile application (`PS_KMP`) is the intended primary interface for end users of ProductiveSocial. It targets Android, iOS, and JVM Desktop from a single shared codebase, using Compose Multiplatform for the UI layer and SQLDelight for offline-first local data storage. The application is fully functional without a network connection — all user interactions write to a local SQLite database first, and a background sync layer pushes and pulls changes to and from the backend services when connectivity is available.

This section documents the application's architecture, every feature module with its screen behaviour and data model, the sync protocol, and the current implementation state.

---

#### 3.6.1 Application Architecture: Three-Layer Clean Architecture

The application is structured in three distinct layers, each with clearly defined responsibilities and no circular dependencies.

**Domain layer** (`commonMain/domain/`): Contains pure Kotlin data classes and use case interfaces with zero dependency on any framework, database, or network library. A `Task` domain model is a plain Kotlin data class. A `GetTasksUseCase` interface defines how tasks are retrieved without knowing whether they come from a local database or a remote API. This layer is fully unit-testable without any mocking infrastructure.

**Data layer** (`commonMain/data/`): Contains repository implementations backed by two data sources per entity type — a `LocalDataSource` (SQLDelight) and a `RemoteDataSource` (Ktor HTTP client). The repository's responsibility is to coordinate between these sources: writes go to local first (making the UI responsive immediately), the dirty flag is set, and the sync manager flushes dirty records to the server in the background. When the server responds, the local rows are updated with server-assigned IDs.

**Presentation layer** (`commonMain/presentation/`): Contains Compose UI composables and their associated `ViewModel` classes. Each screen is a stateless composable that observes a `StateFlow<UiState>` from its ViewModel and sends user events via a channel. ViewModels expose no mutable state directly — all changes flow through the ViewModel's `handleEvent()` method, making the state machine explicit and traceable.

The dependency graph is enforced at module boundaries: domain has no dependencies; data depends on domain; presentation depends on data and domain. Platform-specific source sets (`androidMain`, `iosMain`, `jvmMain`) contain only platform-specific driver registrations and dependency injection entry points — all business logic lives in `commonMain`.

---

#### 3.6.2 Platform Targets and Configuration

The same `commonMain` source set compiles to three independent platform targets:

**Android** uses the `AndroidSqliteDriver` for SQLDelight and the OkHttp Ktor engine. In local development, the base URL for API calls points to `10.0.2.2` — the Android emulator's alias for the host machine's `localhost`. The entry point is `MainActivity`, which calls `startKoin` via `PSocialApplication`. Ports in local development: selfmanager on `8080`, timer on `8081`, analytics on `8082`.

**iOS** uses the native `NativeSqliteDriver` and the Darwin Ktor engine. The base URL points to `localhost`. The entry point is `MainViewController`, exposed to the Swift `iosApp` target. Koin is initialised from Swift before the first Compose view is created.

**JVM Desktop** uses `JdbcSqliteDriver` with a file-backed SQLite database and the Java Ktor engine. The base URL points to `localhost`. The entry point is `main.kt` in the `jvmMain` source set, which opens a desktop window using Compose's `application {}` DSL.

The choice of Ktor as the HTTP client and SQLDelight as the database layer was driven by their native multiplatform support: both libraries provide platform-specific driver implementations behind a common Kotlin interface, so the networking and database code in `commonMain` requires no platform conditionals.

---

#### 3.6.3 Local Database: SQLDelight Schema

Every entity has a local SQLDelight table. The schema is designed to mirror the server-side data model while adding the fields needed for offline-first operation: a client-generated `syncId` UUID (the idempotency key for sync), a `serverId` column (NULL until the server confirms the record), an `isDirty` flag (true when the record has local changes not yet confirmed by the server), and an `isDeleted` flag (soft-delete sentinel for records awaiting tombstone sync).

Key tables and their purpose:

| Table | Purpose |
|---|---|
| `task` | Stores tasks with priority, urgency, due date, recurrency, completion state |
| `habit` | Stores habits with type (START/QUIT), recurrency, scheduled times |
| `routine` | Stores routines with ordered steps |
| `routine_step` | Individual steps of a routine with duration and autoStart flag |
| `habit_completion` | One row per day a habit was marked complete |
| `routine_completion` | One row per routine completion with step-level detail |
| `completion_log` | General completion events for tasks |
| `tag` | User-scoped string labels shared across entity types |
| `project` | Project containers with name, color, icon, priority |
| `pomodoro_settings` | User's timer configuration |
| `pomodoro_session` | One row per focus session with entity link and cumulative stats |
| `pomodoro_interval` | One row per work/break interval within a session |
| `sync_metadata` | Key-value store: deviceId, serverUserId, lastSyncedAt, lastTimerSyncedAt, stored JWT |

The `sync_metadata` table is the sync state store. It persists the user's JWT access token across app restarts (so the user is not asked to log in again), the server-assigned `userId` (set after the first successful auth), and the two sync timestamps (`lastSyncedAt` for selfmanager and `lastTimerSyncedAt` for the timer service) that are used on the next sync to request only entities changed since the previous sync.

---

#### 3.6.4 Authentication

Users sign in with only their email address — no password. The authentication screen presents a single email input and a "Continue" button. On submission, the app calls `POST /api/v1/auth/identify` on selfmanager. The server finds or creates the user and returns an access token, a refresh token, and the integer `userId`.

The token and `userId` are written to the `sync_metadata` table. On subsequent app launches, `UserSessionManager.init()` reads the stored token and restores the session silently — the user is taken directly to the main screen without re-authenticating. The session remains valid until the token expires (24 hours from issue) or the user explicitly logs out.

Logout calls `UserSessionManager.logout()`, which wipes all locally stored user data — the token, the userId, all entity tables, and all sync state — effectively resetting the app to a clean first-launch state.

---

#### 3.6.5 Projects

Projects are organizational containers. Every task, habit, and routine belongs to a project. Each project carries a name, description, hex color code, icon name, priority level, and tags. The user can browse all projects in a grid view, each displayed as a color-coded card showing the project name and entity count.

Drilling into a project opens the project detail view, showing all tasks, habits, and routines nested within it, along with the project's tags and priority. Projects can be created locally (immediately visible in the grid) and sync to the server in the background.

On first sync, every user has at least a "Default" project created automatically by selfmanager. The local database stores both the client `syncId` UUID and the server-assigned integer `serverId`; after the first sync response, the `serverId` is populated and used for all subsequent cross-service references.

---

#### 3.6.6 Self-Manager Screen

The Self-Manager screen is the central daily planning view and the home screen of the application. It combines tasks, habits, and routines into a single, unified list for the selected date.

At the top of the screen is a **horizontal date strip** — a scrollable row of date chips showing the current week. Tapping a date chip updates the selected date and reloads the list. The user can swipe left and right to navigate between weeks. A "Today" button snaps the date strip back to the current date.

Below the date strip is the **unified productivity list**, which shows all tasks scheduled for the selected date, all habits due on that day (based on their recurrency), and all routines scheduled for that day — all in a single scrollable list. Each item is distinguishable by an icon: a checkbox for tasks, a flame icon for Start habits, an X icon for Quit habits, and a checklist icon for routines.

The list supports two orthogonal controls in the header:

**Sorting**: items can be sorted by due date (ascending or descending) or by priority (high to low).

**Grouping**: items can be grouped by none (flat list), by tag, by project, or by urgency. When grouped, each group is a collapsible section header with the group name and item count.

The `SelfManagerViewModel` combines reactive streams from the Task, Habit, and Routine repositories using Kotlin coroutines' `combine` operator, applies the current sort and group configuration, and re-emits the processed list whenever either the selected date or any repository data changes. This means if the user marks a habit complete in this list, the list updates instantaneously without a reload.

From this list, every item is actionable inline:
- **Tasks**: tapping the checkbox marks the task complete; long-pressing opens the context menu (edit, delete, start focus session)
- **Habits**: tapping the completion circle marks the habit done for today; if the habit has subtasks, a bottom sheet appears for step-by-step marking
- **Routines**: tapping a routine opens the Routine Runner

---

#### 3.6.7 Tasks

Tasks are the primary unit of goal-directed work. The full task data model includes:

- **Name** and optional **description**
- **Priority**: None, Low, Medium, or High — displayed as a color-coded badge
- **Urgency**: a separate dimension from priority (allowing a 2×2 urgency/priority matrix)
- **Due date**: optional, with overdue highlighting when past
- **Recurring flag**: marks tasks that repeat on a schedule
- **Target**: optional quantitative goal (e.g., "read 30 pages")
- **Scheduled times**: one or more specific times of day when the task is planned, stored as `HH:mm` strings
- **Reminder options**: notification triggers at the scheduled time, or 5, 10, 15, or 30 minutes before
- **Tags**: multiple string labels shared across entity types
- **Subtasks**: an ordered list of sub-items, each with a name and completion state; subtasks can be reordered and each has its own completion tracking
- **Time spent**: accumulated minutes from linked Pomodoro sessions, updated automatically by the timer service via the `time-log` internal endpoint
- **Completion log**: timestamped records of each time the task was marked complete (for recurring tasks, this tracks the history across repetitions)

The **task creation and editing screen** manages all of the above in a single scrollable form. Subtasks can be added inline with a text field at the bottom of the subtasks list. Tags are entered as chips with autocomplete against the user's existing tag library.

The **task detail view** is the read-focused counterpart to the editor. It presents the task's full data, the subtask checklist, the accumulated time-spent counter, linked Pomodoro sessions with their dates and durations, and the completion log timeline.

---

#### 3.6.8 Habits

Habits come in two types that reflect opposite behavioral goals:

**Start habits** represent behaviors the user wants to build — exercising daily, meditating, reading for 30 minutes. The completion UI is designed around encouragement: a circular fill animation on mark-complete, a growing streak counter, and a heatmap calendar showing the last 30 days of completions.

**Quit habits** represent behaviors the user wants to eliminate — checking social media first thing in the morning, eating sugar after 8pm. The completion UI inverts the positive framing: a completion means the user *successfully avoided* the behavior. The same streak and calendar mechanics apply, but the language changes.

Both types share the same data model: name, description, project, priority, tags, recurrency (Daily, Weekly, etc.), scheduled times (when the habit is planned), reminder times (notification triggers), and subtasks (individual steps within the habit, each independently completable).

**Habit completions** are tracked at daily granularity in the `habit_completion` table: each row records the habit ID, the completion date, and optionally the individual subtask completion states. The completion date is the local date (not a UTC timestamp) to avoid timezone-induced streak-breaking when the user marks a habit complete near midnight.

Marking a habit complete creates a local `habit_completion` row immediately, adds it to the sync queue, and updates the streak counter in the UI. If the habit has subtasks, the completion bottom sheet presents each subtask as a checkbox and records which were completed at what time.

---

#### 3.6.9 Routines

Routines are structured sequences of steps — an ordered checklist where the steps should be completed in order, not in arbitrary sequence. They are designed for fixed workflows that a user performs repeatedly: a morning routine, a pre-workout sequence, a study session protocol, an end-of-day wind-down.

Each routine has the same header data as tasks and habits (name, description, project, priority, tags, recurrency, scheduled times, reminder times). The distinguishing feature is its **step list**: an ordered collection of steps, each with:

- **Name**: what to do in this step
- **Duration in minutes**: how long the step typically takes
- **Position index**: determines the fixed order
- **Auto-start flag**: whether the app should automatically advance to this step when the previous one completes

Routines also support **off-times** — specific dates, days of the week, or time windows when the routine should not appear in the Self-Manager list. This allows a user to schedule a routine only on weekdays or pause it during holidays.

The **Routine Runner** is a dedicated execution interface that opens when the user taps a routine in the Self-Manager list. It presents one step at a time:

1. The step name and planned duration are shown at the top
2. A countdown timer runs for the planned duration (if `autoStart` is true, it starts automatically; otherwise the user taps "Start Step")
3. The user taps "Done" to mark the step complete and advance to the next one, or "Skip" to move on without marking it complete
4. A progress bar at the top shows how many steps have been completed out of the total
5. When the last step is done, a completion summary shows the total time taken and the steps completed/skipped, and a `routine_completion` record is written to the local database with step-level detail

---

#### 3.6.10 Pomodoro Timer

The Pomodoro module is the most interactive component of the application, implementing a configurable focus timer with a full session lifecycle tracked locally and synced to the timer service.

**Settings** control the timer behaviour and are stored in the local `pomodoro_settings` table. The configurable parameters are:

- Work interval duration (default 25 minutes)
- Short break duration (default 5 minutes)
- Long break duration (default 15 minutes)
- Number of cycles before a long break (default 4)
- Auto-start breaks: whether the break timer starts automatically when a work interval ends
- Auto-start next session: whether the work timer starts automatically after a break
- Sound: whether audio cues play at interval boundaries
- Notifications: whether system notifications fire when intervals end

Settings changes are written locally and synced to the timer service via `PUT /api/v1/pomodoro/settings`. When the settings response includes a `recommended` block (available after the user has accumulated enough session history), the settings screen shows the server-computed personalized durations alongside the current settings as a comparison.

**Session creation and entity linking.** A Pomodoro session can be created in two modes:

*Standalone mode*: The user opens the timer and taps "Start Focus Session" without linking to any entity. A session is created locally, the first work interval begins, and the countdown starts. This mode is for general focus time not attributed to a specific task.

*Linked mode*: The user taps the Pomodoro icon on a task, habit, or routine. The `FocusViewModel` opens the timer pre-linked to that entity. This records the entity type and local ID on the session row (`entityType` and `entityLocalId`). When the session completes and syncs to the server, the timer service fires a `time-log` call to selfmanager that adds the session's total work minutes to the linked entity's `time_spent_minutes`. This automatic time attribution requires no manual input from the user.

**The PomoViewModel** owns the live timer state: time remaining in the current interval, current interval type (Work / Short Break / Long Break), current cycle count, whether a break is active, total sessions completed today, and total focused minutes today. When an interval ends, the ViewModel computes the next interval type based on cycle count and settings, then auto-starts it if the auto-start flag is set.

**The FocusViewModel** extends PomoViewModel for entity-linked sessions. It additionally loads the linked entity's subtasks, which are displayed in a collapsible panel below the timer during the work interval. The user can check off subtasks during the session without leaving the timer screen. After a session ends, the FocusViewModel surfaces a task picker so the user can immediately chain into the next focus task without navigating back to the Self-Manager.

**Session lifecycle states** and their transitions:

```
Created
  └─▶ ACTIVE (first Work interval starts)
         ├─▶ PAUSED (user taps Pause)
         │      └─▶ ACTIVE (user taps Resume)
         │      └─▶ ABANDONED (user taps Abandon from pause screen)
         ├─▶ COMPLETED (user taps Complete Session)
         └─▶ ABANDONED (user taps Abandon from active screen)
```

Every state transition updates the local `pomodoro_session` row and marks it dirty. The row is pushed to the timer service on the next sync via `POST /api/v1/pomodoro/sync`, which follows the same batched idempotent pattern as selfmanager's sync.

**Abandonment risk signal.** When the user taps Pause, the ViewModel checks whether the pause count has reached the user's personal threshold. The threshold is computed from their abandon history (average pause count at abandonment, with a minimum of 3 if insufficient history). If the threshold is reached, the pause screen shows a warning: "You've paused this many times before abandoning. Stay in it?" with Resume and Abandon as the two options.

**Break suggestions.** After each completed work interval, the timer computes whether a standard short break, a long break, or an extended rest is recommended based on the user's total focus time today and whether a long break has already been taken. The break suggestion is shown as a short message with the recommended duration. The user can accept the suggestion or manually choose a different break type.

**Pomodoro statistics screen.** A separate statistics screen (accessible from the Pomodoro tab) shows:
- Total sessions and total focused minutes (all-time and this week)
- Session completion rate (completed vs. abandoned)
- Focus patterns: a bar chart showing completed work intervals by hour of day and by day of week, identifying peak focus times
- Entity stats: a breakdown of sessions and work minutes by entity type (Tasks, Habits, Routines, Standalone)

---

#### 3.6.11 Journal

The Journal module is one of the most strategically important planned features of the platform. A screen is already in place in the KMP application displaying a list of journal entries with date, time, mood indicator, and content. The data layer and server integration are not yet wired — the screen currently shows placeholder structure — but the UI scaffolding is intentional: the journal is a first-class destination in the navigation hierarchy (appearing in the bottom navigation bar alongside Self-Manager, Pomodoro, and User Page), not an afterthought.

**What the journal will do.** The journal serves the reflective dimension of personal productivity. Unlike tasks (oriented toward completion) and habits (oriented toward consistency), journaling creates space for retrospective sense-making: integrating the day's events into a coherent narrative, processing how the day felt, and extracting lessons for tomorrow.

Each journal entry will record:
- **Date and time** of writing
- **Mood indicator**: a simple categorical or scale-based mood rating (e.g., 1–5 or emoji-based) captured at the moment of writing
- **Free-form content**: the user's own text
- **Automatic activity context**: a structured summary of the day's completed tasks, habit completions, and Pomodoro session total — attached to the entry automatically from the other services, so the user does not need to manually transcribe what they did

The final point is the key differentiator. A journal entry in Day One or any other journaling application requires the user to remember and manually write what they accomplished. In ProductiveSocial, the journal entry is pre-populated with a structured summary of the day's objective activity — every completed task, every habit marked done, the total focus time — derived automatically from selfmanager and the timer service. The user's writing fills in the *subjective* layer: how it felt, what was difficult, what surprised them.

**AI-assisted synthesis.** The journal's most powerful feature, planned as part of the `psocial_journal` service, is an AI synthesis mode. When the user finishes a journal entry, a dedicated analysis type (`DAILY_REFLECTION`) in the analytics service will combine:
- The user's written text (the subjective account)
- The day's task completions and priorities (from selfmanager)
- The habit completion record (from selfmanager)
- The Pomodoro session data including focus time and entity attribution (from the timer)

The LLM will synthesise these into a brief, personalised end-of-day reflection — identifying patterns ("You completed 4 of your 5 high-priority tasks and your focus time was above your weekly average, but you skipped your evening wind-down routine for the third time this week"), surfacing correlations, and offering one concrete intention for tomorrow.

This synthesis is the closest the platform comes to the vision described in the mission statement: transforming raw activity data and subjective experience into an actionable narrative. No existing productivity or journaling application currently does this because no existing application has simultaneous access to all of these data sources.

**Planned journal service architecture.** The `psocial_journal` service will follow the same architectural pattern as the other services: Kotlin/Ktor, its own PostgreSQL database, JWT authentication using the shared secret, and internal endpoints for the analytics service to fetch journal content when building AI prompts. The journal entries will sync to the mobile client via the same offline-first batch sync protocol used by selfmanager.

---

#### 3.6.12 Analytics Insights (User Page)

The User Page screen integrates the AI analysis features of the analytics service directly into the mobile app. An `AnalyticsApiClient` (in `commonMain`) calls `POST /api/v1/analytics` with an analysis type and optional model ID. The supported analysis types are `PRODUCTIVITY_SUMMARY`, `TASK_PRIORITIZATION`, `HABIT_INSIGHTS`, `ROUTINE_OPTIMIZATION`, `WEEKLY_TIMER_SUMMARY`, and `CUSTOM`.

The `InsightsViewModel` manages the analysis trigger flow: it calls the API, handles the loading state (showing a progress indicator with an explanatory message since inference can take 15–30 seconds), and stores the result in a `StateFlow<InsightUiState>` that the screen observes. Past reports are fetched from `GET /api/v1/analytics` and displayed as a scrollable history list. The current credit balance is shown at the top of the screen, fetched from `GET /api/v1/credits/balance`.

---

#### 3.6.13 Navigation and Routing

Navigation is handled by a type-safe `Routes` sealed interface in `commonMain`. Each screen is a serializable Kotlin object or data class in this hierarchy, and the `NavHost` maps each route to its Compose composable. This approach eliminates stringly-typed navigation (where incorrect string identifiers cause runtime crashes instead of compile-time errors).

Top-level destinations accessible from the bottom navigation bar (or side navigation rail on larger screens):

| Destination | Icon | Description |
|---|---|---|
| Self-Manager | Checklist | Daily view of tasks, habits, routines |
| Pomodoro | Timer | Focus sessions, timer, statistics |
| Journal | Book | Journal entries (placeholder) |
| User Page | Person | Analytics, insights, credits |

Deeper routes covered by the navigation graph:

- Project creation and detail screen
- Task creation, edit, and detail screens
- Habit creation, edit, and detail screens
- Routine creation, detail, and Runner screens
- Pomodoro session execution (standalone and entity-linked)
- Pomodoro settings
- Pomodoro task picker (post-session entity selection)
- Pomodoro statistics

---

#### 3.6.14 Responsive Layout

The application adapts to screen size using a `WindowSize` utility that classifies the current screen as Compact, Medium, or Expanded.

On **Compact** screens (phones), the navigation is a **bottom navigation bar** — the standard mobile navigation pattern, visible at the bottom of the screen with icon + label tabs for the four top-level destinations.

On **Medium and Expanded** screens (tablets and desktop), the navigation switches to a **side navigation rail** — a vertical column of icons and labels along the left edge, freeing the full width of the screen for content. On large desktop screens, a full navigation drawer with extended labels is shown.

The `NavScaffold` composable composes the appropriate navigation component with the content area based on the current `WindowSize`. All screen composables adapt their layout to the available width — list screens use multi-column grids on wide screens and single-column lists on phones.

---

#### 3.6.15 Dependency Injection

Koin is used for dependency injection across all platforms. `AppModule` in `commonMain` registers all repositories, ViewModels, and API clients as singletons or factory instances. Platform-specific modules (`androidModule`, `iosModule`, `desktopModule`) provide the SQLDelight driver appropriate for each platform and the Ktor engine.

```kotlin
// Excerpt from AppModule.kt (commonMain)
val appModule = module {
    single { TaskRepository(get(), get()) }
    single { HabitRepository(get(), get()) }
    single { RoutineRepository(get(), get()) }
    single { PomodoroRepository(get(), get()) }
    single { SyncManager(get(), get(), get()) }
    factory { SelfManagerViewModel(get(), get(), get()) }
    factory { PomoViewModel(get()) }
    factory { FocusViewModel(get(), get()) }
}
```

The `single` declaration creates a shared singleton instance used across all screens. The `factory` declaration creates a new instance per screen navigation, which is appropriate for ViewModels (each screen instance gets its own ViewModel lifecycle).

---

#### 3.6.16 Offline Sync Protocol

The sync layer is the backbone of the offline-first architecture. Every entity is assigned a client-generated UUID (`syncId`) at creation time. This UUID is the idempotency key — if the same entity is submitted twice (due to a retry), the server recognises the duplicate `syncId` and returns the existing server ID rather than creating a second record.

**Selfmanager sync** is a single bidirectional call: `POST /api/v1/sync`. The request body contains:
- `creates`: all entities created locally since the last sync (with `syncId` and full field data)
- `updates`: all entities modified locally since the last sync (with `syncId` and changed fields)
- `deletes`: `syncId`s of locally deleted entities (tombstones)
- `lastSyncedAt`: the timestamp stored from the previous sync response

The server processes these in dependency order (projects before tasks, habits before completions), applies upsert semantics for creates and updates, and returns:
- `idMappings`: `{ clientSyncId → serverIntegerId }` for all processed entities, so the local database can populate its `serverId` columns
- `serverChanges`: full entity objects for all records changed on the server since `lastSyncedAt` (e.g., changes made from the web dashboard or another device)
- `errors`: per-item failure messages for any records that could not be processed
- A new `syncedAt` timestamp for the next call

On receiving the response, the sync manager:
1. Updates all local rows that have `isDirty = true` with their server IDs and sets `isDirty = false`
2. Deletes all local rows marked `isDeleted = true` (tombstones confirmed by the server)
3. Upserts all `serverChanges` into the local database (server wins for any conflicts)
4. Stores the new `syncedAt` timestamp in `sync_metadata`

**Timer sync** follows the same pattern via `POST /api/v1/pomodoro/sync` on the timer service, using `lastTimerSyncedAt` from the `sync_metadata` table.

**When connectivity is unavailable**, all writes go to local SQLDelight with `isDirty = true`. The sync manager retries on a configurable interval when connectivity is restored. Because all sync operations are idempotent, there is no risk of data duplication from retries.

---

#### 3.6.17 Current Implementation State

| Module | Status |
|---|---|
| Project structure and build (Android, iOS, Desktop) | Complete |
| SQLDelight schema (all entity tables + sync_metadata) | Complete |
| Clean architecture layers (domain, data, presentation interfaces) | Complete |
| Koin dependency injection (all modules) | Complete |
| Authentication screen | Complete and functional |
| API clients (selfmanager, timer, analytics) | Scaffolded |
| Sync manager | Scaffolded |
| Projects screen | In progress |
| Self-Manager screen (tasks + habits + routines, date strip, sort/group) | In progress |
| Tasks CRUD and detail | In progress |
| Habits CRUD, completion, and detail | In progress |
| Routines CRUD, Routine Runner | In progress |
| Pomodoro timer (PomoViewModel, FocusViewModel) | In progress |
| Pomodoro statistics | Planned |
| Analytics / Insights (User Page) | Scaffolded |
| Journal | Placeholder screen |
| Credits display | Planned |
| Responsive layout (NavScaffold) | Complete |

#### Local Database: SQLDelight Schema

The local database mirrors the server-side data model for all entities that require offline storage. The SQLDelight schema defines the same entity types as the selfmanager and timer services, with client-assigned UUIDs as primary keys:

```sql
-- tasks.sq
CREATE TABLE tasks (
    sync_id         TEXT NOT NULL PRIMARY KEY,
    server_id       INTEGER,                  -- NULL until first sync
    project_sync_id TEXT,
    user_id         INTEGER NOT NULL,
    title           TEXT NOT NULL,
    description     TEXT,
    priority        TEXT NOT NULL DEFAULT 'MEDIUM',
    completed       INTEGER NOT NULL DEFAULT 0,  -- SQLite boolean
    time_spent_min  INTEGER NOT NULL DEFAULT 0,
    is_dirty        INTEGER NOT NULL DEFAULT 1,  -- 1 = needs sync
    is_deleted      INTEGER NOT NULL DEFAULT 0,  -- soft delete until synced
    created_at      TEXT NOT NULL,
    updated_at      TEXT NOT NULL
);

-- pomodoro_sessions.sq
CREATE TABLE pomodoro_sessions (
    sync_id          TEXT NOT NULL PRIMARY KEY,
    server_id        INTEGER,
    user_id          INTEGER NOT NULL,
    entity_type      TEXT,
    entity_sync_id   TEXT,
    entity_server_id INTEGER,
    status           TEXT NOT NULL DEFAULT 'ACTIVE',
    total_work_min   INTEGER NOT NULL DEFAULT 0,
    completed_cycles INTEGER NOT NULL DEFAULT 0,
    is_dirty         INTEGER NOT NULL DEFAULT 1,
    created_at       TEXT NOT NULL,
    completed_at     TEXT
);
```

The `is_dirty` flag marks records that have been created or modified locally but not yet confirmed by the server. The `is_deleted` flag implements soft deletion: when the user deletes a task, it is marked deleted locally and included in the next sync's tombstone list, then physically removed from the local database once the server confirms the deletion.

#### Offline-First Sync Protocol

The mobile app's sync protocol is the mechanism by which data created on the device reaches the server and changes made on the server (e.g., via the web dashboard) reach the device.

**Write path (device → server):**
1. User creates a task. A UUID is generated locally and the task is written to SQLDelight with `is_dirty = 1`.
2. The UI immediately reflects the new task — no network call is awaited.
3. In the background, the sync manager collects all records where `is_dirty = 1` and calls `POST /api/v1/sync` with the batch.
4. The server responds with `idMappings` (mapping each client UUID to its server-assigned integer ID) and any errors.
5. The local records are updated with the server IDs and `is_dirty` is set to 0.

**Read path (server → device):**
1. On sync, the client includes `lastSyncedAt` in the request body.
2. The server returns `serverChanges`: all entities modified after `lastSyncedAt` by any client (web dashboard, another device).
3. The mobile app merges these server changes into the local SQLDelight database, applying conflict resolution rules (server wins for completed states, last-write wins for text fields).

**Offline behavior:**
When the device has no network connectivity, the user can still create tasks, mark habits complete, run Pomodoro sessions, and view all previously synced data. All writes go to SQLDelight with `is_dirty = 1`. When connectivity is restored, the sync manager sends the accumulated batch in a single request. The server's `ON CONFLICT DO UPDATE` semantics ensure that retrying a failed sync never creates duplicates.

#### Screen-by-Screen Feature Description

**Authentication Screen**

The login screen presents a single email input field and a "Continue" button. The user enters their email and taps Continue. The app calls `POST /api/v1/auth/identify` on selfmanager. If successful, the `accessToken` and `refreshToken` are stored securely in the platform keychain (iOS) or EncryptedSharedPreferences (Android), and the app navigates to the home screen. The login screen is the only implemented screen in the current scaffolded version.

If the server is unreachable (offline at login time), the app checks whether a valid stored access token exists. If it does and has not expired, the user is taken directly to the home screen without re-authenticating. If no valid token exists, the user must wait for connectivity.

**Tasks Screen**

The Tasks screen is the primary productivity management view. It displays the user's tasks organized by project, with color-coded priority indicators (red for Critical, orange for High, blue for Medium, grey for Low). Each task row shows the title, priority badge, and a completion checkbox.

Tapping a task opens the Task Detail view, which shows:
- Full title and optional description
- Priority selector
- Subtasks list with individual completion checkboxes
- Time spent (accumulated from linked Pomodoro sessions, displayed as hours and minutes)
- Scheduled times (dates/times when this task is planned)
- Tags (displayed as chips, tappable to filter)
- Linked Pomodoro sessions (count and total focus time)

Creating a task presents a bottom sheet with title, description, priority, project assignment, and optional due date. All fields except title are optional. The task is written to SQLDelight immediately and synced in the background.

Completing a task taps the checkbox, sets `completed = true` locally, and marks the record as dirty. The completion is reflected immediately in the UI with a strikethrough animation.

**Habits Screen**

The Habits screen displays the user's habits grouped by type (Start habits — behaviors to build — and Quit habits — behaviors to break) with a daily completion tracker showing the current streak and today's completion status.

Each habit row shows the habit name, type badge (green "START" or red "QUIT"), recurrency (Daily/Weekly), and a circular completion button. Tapping the completion button logs a completion for today, increments the streak counter, and marks the habit as complete in the local database. If the habit has subtasks, tapping the completion button opens a subtask completion sheet where the user marks individual steps before the habit is counted as complete for the day.

The habit detail view shows the full completion history calendar (heatmap-style, showing which days had completions over the past 30 days), the accumulated streak, time spent on this habit from linked Pomodoro sessions, and the list of subtasks.

Creating a habit presents a form with name, type (Start/Quit), recurrency, optional subtasks, and scheduled reminder times.

**Routines Screen**

The Routines screen displays the user's routines — ordered sequences of steps that form a structured workflow (e.g., a morning routine with 8 steps: wake up, hydrate, meditate, exercise, shower, breakfast, plan day, review priorities).

Each routine row shows the routine name, the number of steps, the recurrency, and a progress indicator for today's completion (e.g., "3/8 steps"). Tapping a routine opens the Routine Runner, a dedicated step-by-step execution interface:

The Routine Runner presents one step at a time with the step name, optional duration, and an auto-start countdown (for steps with the auto-start flag). The user marks each step complete by tapping a checkmark, and the interface advances to the next step. A progress bar at the top shows overall routine completion. When all steps are marked complete, the routine is logged as completed for today with the full step completion record.

**Timer Screen (Pomodoro)**

The Timer screen is the most complex in the application. It presents the current session state and provides controls for the full Pomodoro lifecycle.

*Session creation:* The user taps "Start Focus Session" and optionally links the session to an entity — selecting from their tasks, habits, or routines via a searchable picker. Entity linking is optional; a standalone session (no link) is valid. The user confirms the session start, which calls `POST /api/v1/pomodoro/sessions` and begins the first Work interval.

*During a work interval:* A circular countdown timer shows the remaining time in the current work interval (default 25 minutes). The user can see the interval type (Work / Short Break / Long Break) and the current cycle count (e.g., "Cycle 2/4"). Two controls are available: Pause and Abandon.

*Pause behavior:* Tapping Pause calls `PATCH /sessions/{id}/pause` and displays the abandonment risk signal returned by the server. If the server returns a warning (e.g., "You've paused 3 times — your personal abandonment threshold is 3"), a banner is shown: "You typically abandon sessions after this many pauses. Stay in it?" with "Resume" and "Abandon" options.

*Completing an interval:* When the countdown reaches zero (or the user taps "Complete Early"), the app calls `POST /sessions/{id}/intervals/{id}/complete`. The server responds with a `suggestedNextBreak` value (ExtendedRest, LongBreak, or ShortBreak). The UI displays this suggestion: "You've focused for 90 minutes today. Time for an extended rest (20 min)." The user can accept the suggestion or override with a different break type.

*Break intervals:* During breaks, the countdown runs with a different color (green for short breaks, blue for long breaks). The user can start the next work interval early by tapping "Skip Break."

*Session completion:* When the user taps "Complete Session," the app calls `PATCH /sessions/{id}/complete`, which triggers the time-log call to selfmanager in the background. The summary screen shows total work time, completed cycles, and — if the session was entity-linked — the updated time-spent on that entity.

*Entity time attribution:* This is a key feature that distinguishes the ProductiveSocial timer from standalone Pomodoro apps. When a session linked to a task is completed, the timer service automatically calls `POST /internal/time-log` on selfmanager to add the session's total work minutes to the task's `time_spent_minutes`. This means that when the user opens the Tasks screen after a focus session, the task's time counter has been updated automatically — no manual entry required.

**Analytics Screen**

The Analytics screen gives users access to the AI analysis features and their stored analysis history.

*Triggering an analysis:* The user selects an analysis type from a picker (Productivity Summary, Task Prioritization, Habit Insights, Routine Optimization, Weekly Timer Summary, or Custom), selects a model from the available models list (fetched from `GET /api/v1/models`), and optionally enters a custom prompt (for the Custom type). Tapping "Run Analysis" calls `POST /api/v1/analytics`. Because inference can take 15–30 seconds, the UI shows a loading state with an explanatory message: "Analyzing your tasks, habits, focus sessions... This usually takes 15–30 seconds."

*Viewing reports:* The analysis history is a scrollable list of past reports, showing the analysis type, model used, credits charged, and the date. Tapping a report expands the full insight text. Reports are stored locally in SQLDelight after the first fetch, so previously loaded reports are available offline.

*Credit display:* The current credit balance is shown at the top of the Analytics screen so the user knows how many analyses they can run before their balance is depleted.

**Credits Screen**

The Credits screen shows the user's current credit balance (fetched from `GET /api/v1/credits/balance`) and a scrollable transaction history (from `GET /api/v1/credits/transactions`). Each transaction row shows the type (DEPOSIT in green, CHARGE in red), the amount, the description (including the model name for charges), and the resulting balance.

Users cannot deposit credits from this screen — deposits are an admin-only operation performed via the admin dashboard. The screen includes an explanatory note: "Credits are managed by your administrator. Each AI analysis deducts credits from your balance."

#### Current Implementation State

At the time of this report, the following components are fully scaffolded and building for both Android and iOS targets:

- Multi-module KMP project structure with `:shared`, `:androidApp`, and `:iosApp` modules
- SQLDelight database schema for tasks, habits, routines, Pomodoro sessions, and intervals
- Clean architecture layers (domain, data, presentation) with interface definitions
- Authentication screen — fully implemented and functional
- API client in `commonMain` pointing to all backend services
- Dependency injection setup using Koin Multiplatform

The task, habit, routine, timer, analytics, and credits screens are designed and their ViewModel interfaces are defined, but the Compose UI implementations and repository connections are scoped for the next development phase. The architecture and data layer are in place to support rapid feature development — each remaining screen requires implementing the Compose UI, connecting the ViewModel to its repository, and testing the sync flow end to end.

---

### 3.7 MVP Scope Definition

The MVP of ProductiveSocial was defined as the minimal set of capabilities that demonstrates the core value proposition — unified productivity tracking with cross-domain AI analysis — in a way that is usable by a real user.

The MVP includes all four backend services, both dashboards, and the API surface needed to support the mobile application. It explicitly does not include: the journal module, the notes module, the social features, proprietary trained models, production-grade LLM hosting, and a fully featured mobile application.

This scope was chosen for three reasons:

1. The core value proposition (unified cross-domain AI analysis) is fully demonstrable with the current implementation. A user can add their tasks, habits, routines, and focus sessions and receive a qualitatively richer analysis than any existing application can provide.

2. The deferred features (journal, notes, social) require significant design work beyond the backend architecture — they involve complex UX challenges that are best addressed after the core productivity modules are proven.

3. The architecture explicitly accommodates the deferred features. The data model has designated slots for journal entries and notes (entities that reference the same user identity and project structure as tasks and habits). The social feature can be implemented as a new microservice without modifying any existing service.

---

### 3.8 Planned Services and Features

This section describes the features and services that are designed and planned but not yet implemented. They are documented here rather than deferred entirely to future reports because the current architecture was explicitly designed with them in mind — understanding what the system will become is necessary to evaluate why certain present-day decisions were made the way they were.

---

#### 3.8.1 Extended Selfmanager: Beyond Tasks, Habits, and Routines

The current selfmanager service owns the three structured productivity entity types — tasks, habits, and routines. These are the highest-complexity, highest-value entities in the personal productivity domain and were the right starting point. However, a full-featured task management and personal organisation application includes a wider range of entity types that serve different use cases.

**To-do lists and quick-capture inbox.** Not every item the user wants to capture is a structured task with a priority, due date, and subtasks. Many productivity needs are simpler: a shopping list, a list of book recommendations, a quick note to self. To-do lists are lightweight, unordered or arbitrarily ordered lists of single-line items, without the scheduling and priority machinery of a full task. The quick-capture inbox is a special project that receives all items that the user captures without immediately categorising them — a temporary staging area for thoughts and obligations that will be processed (moved to the right project and entity type) during a weekly review.

**Checklists.** Checklists are reusable item lists that are not recurring habits or routines but are used repeatedly in similar contexts. A pre-flight checklist for a work trip, a list of things to check before submitting a report, a post-workout checklist — these are not habits (they are not tracked for daily streaks) and not routines (they are not scheduled), but they benefit from the structure of an ordered list with completion tracking.

**Templates.** Project templates are predefined structures that the user can instantiate when starting a new effort. A "New Project Kickoff" template might pre-populate a set of tasks (define scope, schedule kickoff meeting, identify stakeholders) that the user typically needs for every new project. Templates reduce the friction of starting recurring project types.

**Calendar view.** The Self-Manager screen's date-strip navigation is the current approach to temporal organisation. A full calendar view — monthly and weekly grid formats showing scheduled tasks, habit due dates, and routine schedules across an extended time horizon — gives the user a broader temporal perspective on their commitments and helps identify over-scheduled days before they arrive.

**Notes with entity linking.** Notes are free-form text documents that live within the productivity context. The distinguishing feature of notes in ProductiveSocial (vs. a separate notes app) is entity linking: a note can be attached to a project, a task, a habit, or a routine. Meeting notes attach to the task "Attend Q3 planning meeting." Research notes attach to the project they inform. This contextual attachment means notes are always findable through the entity they relate to, not only through a search interface.

---

#### 3.8.2 Extended Timer: Shared Focus and Streaks

The current Pomodoro timer is a personal focus tool. Planned extensions bring social and gamification dimensions to the focus experience.

**Focus streaks.** A focus streak counts consecutive days on which the user completed at least one Pomodoro session. Like habit streaks, focus streaks leverage loss aversion to encourage consistent daily engagement. The streak counter will be displayed prominently on the timer home screen and will be visible on the user's public profile.

**Session goals.** Before starting a session, the user can set a session goal — a brief text description of what they intend to accomplish in this block. When the session completes, the goal is presented alongside the session summary, allowing the user to reflect on whether they achieved what they set out to do. Session goals are stored on the session record and become part of the data available to the analytics service when generating AI analyses.

**Shared focus rooms.** A focus room is a virtual shared space where multiple users commit to focusing simultaneously. Each participant's timer runs independently (they may have different durations and break patterns), but the room shows all participants' live status (focusing / on break / completed). The social visibility creates a mild accountability effect — knowing that others can see whether you are focusing or have abandoned your session increases the cost of abandonment. Focus rooms will be implemented as part of the `psocial_social` service.

---

#### 3.8.3 The Social Layer (`psocial_social`)

The social module is the feature that most clearly distinguishes ProductiveSocial from a personal productivity tool and makes it a platform. Its design is driven by one principle: harness the motivational power of social accountability without introducing the attention fragmentation of mainstream social media.

**The fragmentation problem with social media.** Mainstream social platforms (Instagram, Twitter/X, TikTok) are designed to maximise time-on-platform through infinite scroll, algorithmic feed injection, and notification-driven re-engagement. These mechanics are directly opposed to the goal of focused productive work. Embedding a conventional social feed into a productivity app would undermine the app's own purpose.

**The ProductiveSocial social design.** The social module is architecturally separated from the productivity workspace. There is no social feed embedded in the Self-Manager or Timer screens. Social features are opt-in, accessed through the dedicated User Page, and designed around sharing completed work and consistent habits — not ephemeral content.

The planned social features are:

*Public profiles.* Each user has a public profile page showing their display name, bio, and public achievements. The profile does not show real-time status or a live activity feed — it is a summary of accomplishments, not a surveillance interface.

*Achievement sharing.* The platform will define a set of achievements earned by reaching milestones: completing a certain number of tasks, maintaining a habit streak for 30 days, accumulating 100 hours of focus time, completing a full week of morning routines. Earned achievements appear as badges on the user's public profile and can be posted to a community feed.

*Habit blueprints.* Users who have built successful habits can publish them as blueprints — shared templates that other users can adopt. A "30-minute daily reading" habit blueprint published by one user can be adopted by another with a single tap, pre-populating all the habit configuration (type, recurrency, scheduled time, reminders). This creates a peer-to-peer knowledge base of effective habits.

*Community feed.* A simple chronological feed of achievements shared by users the current user follows. Unlike algorithmic social feeds, the community feed has no ranking, no promoted content, and no notification pressure. It is a low-stakes ambient awareness of what others in your network are accomplishing — designed to be glanced at once a day, not compulsively checked.

*Accountability partners.* A user can designate specific connections as accountability partners for a specific habit or goal. Accountability partners receive a weekly summary of each other's progress on the shared goal — a quiet, low-friction nudge that has stronger empirical support for behaviour change than public social pressure.

**Architecture.** The `psocial_social` service will be a new independent microservice, following the same patterns as the existing services: Kotlin/Ktor, its own PostgreSQL database, JWT authentication via the shared secret, and internal endpoints for the analytics service to incorporate social data (e.g., accountability partner streaks) into AI analyses. It will consume data from selfmanager and the timer via internal APIs but will own no copy of productivity data — it will reference entity IDs and user IDs without duplicating the entities themselves.

---

#### 3.8.4 The User Profile Service (`psocial_user`)

Currently, user identity in ProductiveSocial is minimal: an email address, a server-assigned integer ID, and a JWT. This is sufficient for the current phase, where the platform has no social dimension and no public-facing profile.

When social features are introduced, a richer concept of a user is required: a display name, a profile photo, a bio, a timezone, notification preferences, privacy settings, and the social graph (who follows whom). This data should not live in selfmanager — the selfmanager service owns productivity data, not user identity enrichment — nor should it live in the social service, which owns social interactions, not profile data.

The `psocial_user` service will be the dedicated owner of user profile data. It will expose:
- `GET /api/v1/users/me` — the authenticated user's own profile
- `PATCH /api/v1/users/me` — update profile fields (display name, bio, photo)
- `GET /api/v1/users/{id}` — public profile view of another user
- `POST /api/v1/users/{id}/follow` — follow a user
- `DELETE /api/v1/users/{id}/follow` — unfollow
- `GET /api/v1/users/me/followers` / `/following` — social graph queries

The service will share the same JWT secret as the other services and will be accessed by the social service for profile data and by the analytics service for including profile metadata in analyses.

---

#### 3.8.5 Proprietary ML Models: The Data Accumulation Strategy

The current AI analysis pipeline uses large language models for all analysis — a deliberate choice for the early stage of the platform, when no user behavioral data exists to train on. As the platform accumulates real usage data, a parallel track of proprietary predictive model development becomes viable.

The planned transition is not a replacement of LLM-based analysis but an augmentation. Certain prediction tasks that LLMs approach qualitatively ("your habit looks at risk") can be addressed more precisely by trained models that have learned from thousands of real user trajectories:

| Predictive task | Model type | Training signal |
|---|---|---|
| Habit continuation probability | Logistic regression / gradient boosting | Completion logs with streak history |
| Task completion likelihood given current load | Gradient boosting | Task completion logs with context features |
| Optimal focus session duration for this user | Regression | Completed vs. abandoned session history |
| Daily productivity score | Neural network | Composite of task, habit, and focus signals |
| Abandonment risk during session | Gradient boosting | Pause count and session duration at abandonment |

These models will be trained externally and uploaded to the billing service's model registry as `.pkl` files via the existing `POST /api/v1/models/{id}/upload` endpoint. The billing service's sklearn inference pathway — already implemented and ready to receive models — will load them via joblib and invoke them with the user's structured activity data as features. The rest of the system (analytics, mobile app, dashboards) requires no changes; the model appears in the model picker alongside the LLM options.

---

### 3.9 Web Dashboards

ProductiveSocial ships two Streamlit-based web dashboards: an admin dashboard (`psocial_dashboard`, port 1230) for system operators and a user dashboard (`psocial_user_dashboard`, port 1231) for end users. Both are Python applications built with Streamlit and communicate with the backend services exclusively via the same HTTP APIs that the mobile client uses — they are genuine API clients, not a special internal interface.

**Why dashboards were built instead of native interfaces.** The KMP mobile application is the intended long-term interface for both users and administrators, but at the current stage of development it is not yet feature-complete or available to users — the majority of its screens are still in progress. Building a fully featured native admin interface in KMP in parallel with the rest of the platform would have significantly delayed the development of the backend services, which are the core of the project's value proposition.

The two Streamlit dashboards are therefore explicitly short-term solutions designed to bridge the gap: they make the platform usable and demonstrable now, while the mobile application continues to be developed. Streamlit was chosen specifically because it allows a functional, multi-page web interface to be written in a few hundred lines of Python without any frontend development overhead — a single developer can produce a working admin and user interface in a fraction of the time it would take to build the equivalent native screens. Once the KMP application is feature-complete, the dashboards will be superseded by the mobile app as the primary user interface, though the admin dashboard may continue to serve as a monitoring and operations tool.

This design choice has two important consequences: the dashboards provide ongoing integration test coverage of every API endpoint they call, and they demonstrate that the system's API surface is sufficient for all use cases without any privileged bypasses (with the single exception of the admin dashboard's direct billing calls, which use X-Internal-Key as intended for trusted service-level clients).

---

#### 3.8.1 Admin Dashboard (`psocial_dashboard`)

The admin dashboard is restricted to users whose email address appears in the `ADMIN_EMAILS` environment variable of the analytics service. Login uses selfmanager's email-only identify flow — the admin enters their email address, and the dashboard calls `POST /api/v1/auth/identify` on selfmanager to obtain a JWT. Immediately after login, the dashboard calls an analytics admin endpoint to verify admin status; if the response is 403, access is denied and the user is shown an error.

All billing-related calls from the admin dashboard use the `X-Internal-Key` header directly — the dashboard is a trusted backend client that has direct access to the billing service's internal API.

**Page 1 — Models**

The Models page is the interface for managing the LLM and ML model registry in the billing service. It displays a table of all registered models with their name, provider (Ollama, Anthropic, OpenAI, sklearn), model identifier, cost per prediction, and active status.

From this page, an admin can:

- **Register a new model**: A form accepts the model's name, provider selection, provider-specific model ID (e.g., `llama3.2`, `claude-sonnet-4-6`, `gpt-4o`), system prompt, and cost per prediction. Submitting the form calls `POST /api/v1/internal/models` on billing. The new model immediately appears in the model list and becomes available to users in the analytics model picker.

- **Deactivate a model**: Toggling a model's active flag calls `PUT /api/v1/internal/models/{id}` and sets `is_active = false`. Inactive models are not returned by the public `GET /api/v1/models` endpoint and cannot be selected for new analyses. This allows retiring old models without deleting their prediction history.

- **Upload a sklearn `.pkl` file**: For sklearn model types, a file uploader widget calls `POST /api/v1/models/{id}/upload` with the serialized model file. The billing service stores the file and loads it into memory via joblib on the next prediction call.

**Page 2 — Users**

The Users page provides visibility into the user base and credit management tools. It displays a table of all users who have ever used the analytics service, with their selfmanager integer user ID, email, report count, and current credit balance.

From this page, an admin can:

- **Look up a user by email**: The search field filters the user table by email substring, making it easy to find a specific user.

- **Check a user's credit balance**: The balance displayed in the table is fetched from `GET /api/v1/internal/users/{id}/balance` on billing, showing the current `balance_after` of the user's most recent transaction.

- **Deposit credits**: An inline form allows the admin to enter a credit amount and an optional note, then calls `POST /api/v1/internal/users/{id}/deposit` on billing. The deposit is recorded as a DEPOSIT transaction and immediately reflected in the user's balance. This is the only mechanism by which users can receive credits — there is no user-facing deposit UI.

- **View transaction history**: Expanding a user row shows their full transaction history: each transaction's type, amount, description, and running balance.

**Page 3 — Predictions**

The Predictions page shows all prediction records system-wide, providing visibility into LLM usage and credit consumption. The table shows each prediction's creation time, user ID, model used, status (PENDING, SUCCESS, FAILED), credits charged, and a truncated preview of the input and output.

Filtering controls allow narrowing by status, model, or user ID. Clicking a prediction row expands the full input data (the complete user context payload sent to the LLM) and output data (the full LLM response JSON). This is the primary debugging tool for investigating failed predictions or unexpected model outputs.

**Page 4 — Billing Overview**

The Billing page presents system-wide aggregate statistics from the billing service:

- Total number of predictions (all-time and last 7 days)
- Total credits charged (all-time and last 7 days)
- Total credits deposited
- Number of active models
- Breakdown of predictions and credits charged by model
- Chart of prediction volume over time (last 30 days)

These metrics are computed from `GET /api/v1/admin/analytics` on the billing service.

**Page 5 — Analytics Reports**

The Analytics page provides visibility into the reports stored in the analytics service — the AI analysis reports generated for users.

Two views are available:

*System-wide view*: Shows all reports across all users, with columns for user email, analysis type, model used, credits charged, and the date. A type filter allows narrowing to a specific analysis type (e.g., showing only PRODUCTIVITY_SUMMARY reports). This view uses `GET /api/v1/admin/reports` on the analytics service.

*Per-user view*: Selecting a user from a dropdown shows only their reports. The admin can read the full insight text of any report, and can bulk-delete all reports for a user via `DELETE /api/v1/admin/users/{id}/reports` — useful for clearing test data or responding to a user data deletion request.

The analytics admin page also exposes the **seed reports** function: a form that calls `POST /api/v1/admin/seed-reports`, which inserts a set of sample reports for a specified user without making any billing calls. This is designed for testing the analytics display and report history without consuming credits.

---

#### 3.8.2 User Dashboard (`psocial_user_dashboard`)

The user dashboard is open to any registered user. Login uses selfmanager's email-only identify flow, and a single selfmanager JWT is all that is needed to access all four pages — the timer and analytics services accept the same token. Unlike the admin dashboard, the user dashboard never calls billing directly; all credit operations are proxied through analytics.

**Page 1 — Productivity**

The Productivity page displays the user's full productivity data from selfmanager across three sections:

*Tasks section*: A table of all tasks with columns for title, priority, completion status, time spent (accumulated from linked Pomodoro sessions), and due date. Priority is displayed as a colour-coded badge. The table supports column sorting. Completed tasks are shown with a strikethrough style and grouped below incomplete tasks.

*Habits section*: A table of all habits with columns for name, type (START/QUIT), recurrency, and time spent. Below the table, a calendar-style completion heatmap shows the last 30 days of habit completions as a grid of coloured cells — green for days with all habits complete, amber for partial completion, grey for no completion. This gives the user an immediate visual of their consistency trend.

*Routines section*: A table of all routines with columns for name, recurrency, and step count. Expanding a routine row shows the ordered list of steps with their planned durations.

All data on this page is fetched on load via `GET /api/v1/tasks/tasks`, `GET /api/v1/habits/habits`, and `GET /api/v1/routines/routines` on selfmanager using the user's JWT.

**Page 2 — Focus**

The Focus page provides two tabs for interacting with Pomodoro data.

*Sessions tab*: Displays the user's Pomodoro session history in a table with columns for start date/time, linked entity (type and name), status (Completed/Abandoned/Active), total work minutes, and completed cycles. A status filter allows narrowing to completed or abandoned sessions. Summary metrics at the top show total sessions, total focus time, and session completion rate. The timer settings panel shows the user's current work/break durations and any server-computed recommendations.

*Insights tab*: Displays the focus analytics computed from the session history:

- **Recommended durations**: If the user has sufficient session history (5+ work intervals, 3+ short breaks, 3+ long breaks), the server returns personalized recommendations. These are shown as a comparison: "Your settings: 25 min work / 5 min short break. Recommended based on your data: 28 min work / 6 min short break."
- **Focus patterns chart**: A bar chart of completed work intervals by hour of day (0–23) and a second chart by day of week (Monday–Sunday). These charts identify peak focus hours and days, answering the question "when am I most productive?"
- **Entity stats**: A breakdown of sessions by entity type (Tasks, Habits, Routines, Standalone) showing session count, average work minutes, and completion rate for each type. This identifies which kinds of work the user focuses on most effectively.
- **Weekly AI Summary**: A one-click button that calls `POST /api/v1/analytics` with the `WEEKLY_TIMER_SUMMARY` type and displays the LLM-generated summary directly in the tab. This is the user-facing entry point for the timer summary analysis without navigating to the Analysis page.

All focus data is fetched from the timer service using the selfmanager JWT: `GET /api/v1/pomodoro/sessions`, `GET /api/v1/pomodoro/settings`, `GET /api/v1/pomodoro/insights/focus-patterns`, and `GET /api/v1/pomodoro/insights/entity-stats`.

**Page 3 — Analysis**

The Analysis page is the user-facing interface for triggering AI analyses and reading stored reports.

*Run new analysis*: A form with three controls:
- **Analysis type** dropdown: Productivity Summary, Task Prioritization, Habit Insights, Routine Optimization, Weekly Timer Summary, Custom
- **Model** dropdown: populated by `GET /api/v1/models` from the billing service, showing all active models with their names and costs
- **Custom prompt** text area: visible only when the Custom type is selected; the user enters their free-form question

Below the form, a credit indicator shows the user's current balance and the cost of the selected model, so the user knows the remaining balance before confirming. Submitting calls `POST /api/v1/analytics`. Because inference takes 10–30 seconds, the page shows a spinner with the message "Running analysis... this may take up to 30 seconds depending on the model."

*Report history*: Below the analysis form, a scrollable list shows all of the user's stored reports from `GET /api/v1/analytics`, ordered newest first. Each entry shows the analysis type badge, the model used, the credits charged, the date, and the full insight text. The insight text is the verbatim response from the LLM — the section of text marked as the generated analysis in the billing service's response structure.

**Page 4 — Credits**

The Credits page gives the user visibility into their credit balance and spending history. It is a read-only view — users cannot deposit credits from this page.

At the top, the current balance is displayed prominently as a number with a label: "Credit Balance: 87 credits." Below it is a note explaining that credits are managed by the administrator and are deducted automatically when AI analyses are run.

The transaction history table below the balance shows every credit movement for the user: DEPOSIT transactions (in green, showing when an admin added credits) and CHARGE transactions (in red, showing credits deducted for each analysis). Each row shows the transaction date, type, amount (positive for deposits, negative for charges), a description (including the model name for charge transactions), and the running balance after that transaction.

Transactions are fetched from `GET /api/v1/credits/transactions?limit=100` on the analytics service, which proxies the request to billing using the X-Internal-Key header.

---

**Local development with Docker Compose.** All services and databases run in Docker containers on a shared `psocial_network` bridge network. Containers reference each other by service name (`http://selfmanager:1226`), making the configuration portable. The `docker-compose.yml` defines health checks for all database containers, with service containers set to `depends_on: [db: condition: service_healthy]` to prevent services from starting before their databases are ready.

Before starting the Docker Compose stack, the three Kotlin services must be compiled to fat JARs:

```bash
cd psocial_selfmanager && ./gradlew :server:shadowJar
cd psocial_timer && ./gradlew :server:shadowJar
cd psocial_analytics && ./gradlew :server:shadowJar
```

The Python services are started directly by their Docker containers without a pre-build step (dependencies are installed at container startup from `requirements.txt`).

**Production deployment on Render.** Each of the six services is deployed as a separate Render Web Service. The deployment process for Kotlin services is: build the fat JAR locally → commit it to the service's GitHub repository → push → Render detects the push, builds the Docker image (copying the pre-built JAR), and deploys it.

For the Python services, Render builds the image directly from the `Dockerfile` and `requirements.txt`.

**Environment variables.** Each service is configured entirely through environment variables. On Render, these are set in the service's environment configuration. The critical cross-service variables that must be consistent across all services are:
- `JWT_SECRET`, `JWT_ISSUER`, `JWT_AUDIENCE`, `JWT_REALM` (must be identical across selfmanager, timer, analytics)
- `INTERNAL_API_KEY` (must be identical across all services)
- Service URLs (each service must know the URLs of the services it calls)

**Database configuration.** Production databases run on Neon PostgreSQL's free tier. Each service has its own database cluster. The `DATABASE_URL` environment variables for each service point to the corresponding Neon database.

**LLM configuration.** The current production deployment uses a local Ollama instance (llama3.2) exposed via ngrok. The `OLLAMA_BASE_URL` in the Render billing service environment points to the ngrok tunnel URL. This arrangement is appropriate for demonstrating the system's capabilities but is not suitable for a multi-user production deployment.

**Deployment iterations.** The deployment was conducted in stages:

*Stage 1*: selfmanager and timer deployed first, with basic auth and data endpoints verified in production.

*Stage 2*: billing deployed with model registry seeded. First prediction tested against the Ollama tunnel.

*Stage 3*: analytics deployed and connected to selfmanager, timer, and billing. End-to-end analysis tested from the analytics API on Render.

*Stage 4*: Both dashboards deployed and tested against the production API.

*Stage 5*: Full end-to-end test with test user data: tasks, habits, routines, and Pomodoro sessions created via API; analysis triggered via user dashboard; result verified.

---

### 3.10 Deployment

**Local development with Docker Compose.** All services and databases run in Docker containers on a shared `psocial_network` bridge network. See Section 3.12 for full Docker and Colima detail.

---

### 3.11 Testing

**Manual API testing.** All service endpoints were tested manually using HTTP clients (Postman and curl) during development. The testing approach was endpoint-by-endpoint, verifying both the happy path and key error cases (invalid JWT, insufficient credits, missing required fields, unknown entity IDs).

**End-to-end pipeline test.** The most important test was the full end-to-end analysis pipeline: create test data in selfmanager → create Pomodoro sessions in timer → call analytics from the user dashboard → verify the LLM output references specific user data. This test was run successfully both locally (via Docker Compose) and in production (on Render with the ngrok Ollama tunnel).

**Sync idempotency test.** The batch sync endpoint was tested by submitting the same sync request twice and verifying that no duplicate records were created. The test passed for all entity types.

**Credit arithmetic verification.** For each prediction in the test set, the pre-prediction balance, the prediction cost, and the post-prediction balance were manually verified to satisfy `balance_before − cost = balance_after`.

**What was not tested.** Load testing, performance testing under concurrent requests, and chaos testing (simulating partial service failures under load) were not conducted. These are standard practices for production systems and would be conducted before any public release.

---

### 3.12 Service Integration: End-to-End Request Flows

This section traces the complete path of the two most important user-facing operations in the system — authentication and AI analysis — from the client's request to the final response, showing exactly how each service participates.

#### Flow 1: User Login

The login flow establishes the user's identity and issues the token that all subsequent requests will carry.

```
1. Client sends:
   POST http://selfmanager:1226/api/v1/auth/identify
   {"email": "user@example.com"}

2. selfmanager receives the request
   a. Queries users table: SELECT id FROM users WHERE email = 'user@example.com'
   b. If not found: INSERT INTO users (email, created_at) VALUES (?, NOW()) RETURNING id
   c. Generates access token:
      payload = {"userId": 42, "email": "user@example.com", "iat": now, "exp": now+24h}
      token = HMAC-SHA256(header + "." + b64(payload), JWT_SECRET)
   d. Generates refresh token (same structure, exp = now+7d)
   e. Stores refresh token hash in refresh_tokens table

3. selfmanager responds:
   {"accessToken": "eyJ...", "refreshToken": "eyJ...", "userId": 42, "email": "user@example.com"}

4. Client stores accessToken and refreshToken in session state
   All subsequent requests include: Authorization: Bearer eyJ...
```

After this step, the user can immediately call the timer API or the analytics API with the same token — no further login is required.

#### Flow 2: Running an AI Analysis (Complete End-to-End)

This is the central operation of the platform. It involves all four services, two cross-service data collection calls, and an LLM inference. Each step is traced in full.

```
Step 1 — Client requests analysis
   POST http://analytics:1228/api/v1/analytics
   Authorization: Bearer eyJ...
   {"type": "PRODUCTIVITY_SUMMARY", "modelId": "a1b2-..."}

Step 2 — analytics: JWT validation
   a. Extract token from Authorization header
   b. Decode header: algorithm = HS256
   c. Recompute HMAC-SHA256(header + "." + payload, JWT_SECRET)
      → matches signature ✓
   d. Check exp claim: not expired ✓
   e. Extract userId = 42, email = "user@example.com"

Step 3 — analytics: ensure local user record
   SELECT id FROM analytics_users WHERE selfmanager_id = 42
   → not found: INSERT INTO analytics_users (selfmanager_id, email) VALUES (42, 'user@...')

Step 4 — analytics: parallel data collection (Kotlin coroutines)
   All four calls launched simultaneously:

   [Coroutine A] GET http://selfmanager:1226/internal/users/42/tasks
                 X-Internal-Key: <key>
                 → selfmanager validates key, queries tasks table for user 42
                 → returns: [{"name":"Write report","priority":"HIGH","completed":false,...}...]

   [Coroutine B] GET http://selfmanager:1226/internal/users/42/habits
                 X-Internal-Key: <key>
                 → returns: [{"name":"Morning Exercise","type":"START","recurrency":"DAILY",...}...]

   [Coroutine C] GET http://selfmanager:1226/internal/users/42/routines
                 X-Internal-Key: <key>
                 → returns: [{"name":"Morning Routine","recurrency":"DAILY"}...]

   [Coroutine D] GET http://timer:1227/internal/users/42/stats
                 X-Internal-Key: <key>
                 → timer queries pomodoro_sessions for user 42, aggregates stats
                 → returns: {"totalSessions":32,"completedSessions":31,"totalWorkMinutes":310,...}

   awaitAll() — waits for all four to complete (or fail)
   Total latency ≈ max(A, B, C, D) not A+B+C+D

Step 5 — analytics: build inputData payload
   {
     "analysis_type": "PRODUCTIVITY_SUMMARY",
     "tasks": [{"name":"Write report","priority":"HIGH","completed":false,"timeSpentMinutes":45},...],
     "habits": [{"name":"Morning Exercise","type":"START","recurrency":"DAILY","timeSpentMinutes":0},...],
     "routines": [{"name":"Morning Routine","recurrency":"DAILY"},...],
     "pomodoro": {"totalSessions":32,"completedSessions":31,"totalWorkMinutes":310,...},
     "prompt": "Based on the user's tasks, habits, routines and focus sessions,
                provide a concise productivity summary. Highlight what went well
                and what needs attention."
   }

Step 6 — analytics: call billing for inference
   POST http://billing:1229/api/v1/internal/predict
   X-Internal-Key: <key>
   Content-Type: application/json
   {
     "selfmanager_user_id": 42,
     "model_id": "a1b2-...",
     "input_data": <payload from step 5>
   }

   (analytics uses a 120-second timeout for this call)

Step 7 — billing: validate and check credits
   a. Fetch model: SELECT * FROM ml_models WHERE id = 'a1b2-...' AND is_active = TRUE
      → found: provider=OLLAMA, model_name='llama3.2', cost_per_use=5
   b. ensure_welcome_credits(42):
      SELECT COUNT(*) FROM transactions WHERE selfmanager_user_id = 42
      → count = 3 (user has prior transactions), skip welcome deposit
   c. Get balance:
      SELECT balance_after FROM transactions
      WHERE selfmanager_user_id = 42 ORDER BY created_at DESC LIMIT 1
      → balance = 95 credits
   d. Check: 95 >= 5 (model cost) ✓
   e. Create PENDING prediction record in database

Step 8 — billing: LLM inference
   a. Format user data into readable text blocks:
      "USER TASKS:
       - Write report (priority: HIGH, completed: false, time: 45 min)
       ...
       USER HABITS:
       - Morning Exercise (type: START, daily, time: 0 min)
       ...
       POMODORO STATS:
       - Total sessions: 32, Completed: 31, Total work: 310 min
       ..."
   b. Combine with prompt:
      user_message = formatted_data + "\n\n" + prompt
   c. Call Ollama (via ngrok tunnel in production):
      POST https://abc123.ngrok.io/api/chat
      ngrok-skip-browser-warning: true
      {
        "model": "llama3.2",
        "messages": [
          {"role": "system", "content": "You are an expert productivity coach..."},
          {"role": "user", "content": <user_message>}
        ],
        "stream": false
      }
   d. Ollama responds (after ~8-25 seconds):
      {"message": {"content": "Your task completion rate shows a clear pattern..."},...}

Step 9 — billing: record transaction and update prediction
   a. INSERT INTO transactions (selfmanager_user_id, type, amount, balance_after, ...)
      VALUES (42, 'CHARGE', -5, 90, ...)
   b. UPDATE predictions SET status='SUCCESS', output_data=<response>, credits_charged=5
      WHERE id = <prediction_id>

Step 10 — billing responds to analytics:
   {
     "predictionId": "...",
     "result": "Your task completion rate shows a clear pattern...",
     "creditsCharged": 5,
     "modelUsed": "llama3.2"
   }

Step 11 — analytics: persist report and respond to client
   a. INSERT INTO analytics_reports
      (user_id, analysis_type, model_id, insight_text, credits_charged, created_at)
      VALUES (42, 'PRODUCTIVITY_SUMMARY', 'a1b2-...', 'Your task completion...', 5, NOW())
   b. Return to client:
      {
        "id": <report_id>,
        "type": "PRODUCTIVITY_SUMMARY",
        "insight": "Your task completion rate shows a clear pattern...",
        "creditsCharged": 5,
        "modelUsed": "llama3.2",
        "createdAt": "2026-05-18T14:23:07Z"
      }
```

The total end-to-end latency for this flow on the Render free tier with warm services is approximately 15–30 seconds, dominated by the Ollama inference step. With a cloud-hosted LLM provider (Anthropic or OpenAI), this would reduce to approximately 5–10 seconds.

#### Flow 3: Timer Time-Log (Cross-Service Write)

When a Pomodoro work interval is completed and was linked to a task, the timer automatically updates the task's accumulated time in selfmanager:

```
1. User completes a work interval in the timer
2. timer: UPDATE pomodoro_intervals SET status='COMPLETED', actual_minutes=25, ended_at=NOW()
3. timer: UPDATE pomodoro_sessions SET total_work_minutes = total_work_minutes + 25
4. If session.entity_type == 'TASK' and session.entity_id == 15:
   Fire-and-forget (not awaited):
   POST http://selfmanager:1226/internal/time-log
   X-Internal-Key: <key>
   {"entityType": "TASK", "entityId": 15, "minutes": 25}
5. selfmanager: UPDATE tasks SET time_spent_minutes = time_spent_minutes + 25 WHERE id = 15
6. timer returns the completed interval to the user
   (selfmanager's response is not awaited — timer does not fail if selfmanager is unavailable)
```

The fire-and-forget pattern means the timer's response to the user is not blocked by the time-log call to selfmanager. If selfmanager is temporarily unavailable, the time-log update is lost — the trade-off accepted for this proof-of-concept is that focus time attribution may be slightly underestimated during selfmanager downtime, rather than causing the timer's core functionality to fail.

---

### 3.13 Deployment: Docker, Colima, and Render in Detail

#### Local Development Environment

The local development environment runs all services on the developer's machine using Docker Compose managed by Colima. The complete local stack consists of eight containers: four service containers (selfmanager, timer, analytics, billing) and four database containers (one PostgreSQL instance per service).

**Colima** is started first:
```bash
colima start --cpu 4 --memory 8
```

This starts a Linux virtual machine with 4 CPU cores and 8 GB of RAM allocated — sufficient to run all eight containers simultaneously. Colima exposes a Docker socket at `/var/run/docker.sock` that the standard Docker CLI and Docker Compose tools connect to transparently.

**Building the Kotlin services.** The three Kotlin services must be compiled before Docker Compose can start them, because their Dockerfiles copy a pre-built JAR rather than building inside the container. The build command for each:

```bash
cd psocial_selfmanager
./gradlew :server:shadowJar
# Produces: server/build/libs/server-all.jar (~25 MB)
```

This approach keeps the Docker build fast (under 10 seconds, just a file copy) at the cost of requiring a local JDK and Gradle installation. An alternative approach (multi-stage Docker build) would compile inside the container but would require a full JDK in the build stage and take 3–5 minutes per build.

**Starting all services:**
```bash
docker compose --env-file .env up -d
```

Docker Compose reads the `docker-compose.yml`, starts the eight containers in dependency order (databases first, then services), and runs them in the background (`-d`). The `--env-file .env` flag loads environment variables (JWT secrets, database passwords, API keys) from a local `.env` file that is not committed to git.

**Health checks** ensure that service containers do not start before their databases are accepting connections:
```yaml
selfmanager_db:
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U ${DB_USER} -d selfmanager"]
    interval: 5s
    retries: 10
```

PostgreSQL takes 2–5 seconds to initialize. Without the health check, the service container might start, attempt to connect to the database, fail, and crash — requiring a manual restart. The `condition: service_healthy` dependency ensures the service starts only after the database health check passes.

**The `psocial_network` bridge network** allows containers to reach each other by service name:
- From `analytics`, `http://selfmanager:1226` resolves to the selfmanager container's internal IP
- From `timer`, `http://selfmanager:1226` resolves to the same address
- Port numbers are container-internal ports, not the host-mapped ports

This name-based addressing exactly mirrors the Render production environment, where each service is also reachable by its service name. The configuration is portable: the same service URLs work in both environments.

#### Production Deployment on Render

Each of the six application services (four backends, two dashboards) is a separate Render Web Service linked to its own GitHub repository. The deployment workflow:

1. Developer builds the fat JAR (for Kotlin services)
2. Developer commits the JAR to the repository (`git add server/build/libs/server-all.jar`)
3. Developer pushes to the main branch
4. Render receives a webhook notification from GitHub
5. Render clones the repository and builds the Docker image
6. Render deploys the new container, routing traffic to it once it is healthy

The Kotlin services' Dockerfiles are intentionally simple because the JAR is pre-built:
```dockerfile
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY server/build/libs/server-all.jar app.jar
EXPOSE 1226
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Render's build process for this Dockerfile takes approximately 45 seconds (pulling the base image, copying the JAR, pushing the resulting image to Render's registry). Deployment (replacing the running container with the new one) takes approximately 30 additional seconds, during which the old container continues serving traffic.

**Cold start behavior.** Render's free tier suspends a service after 15 minutes of inactivity. The next request triggers a cold start: the container is started from scratch, the JVM initializes, and the service connects to its database. For JVM-based services, this takes approximately 50 seconds. For the Python billing service, approximately 8 seconds.

The cold start problem is mitigated in the user dashboard with a visible banner: "Services may take up to 60 seconds to respond on first use (Render free tier cold start)." This sets expectations and prevents users from thinking the system is broken when the first request is slow.

---

### 3.14 Security Considerations

This section documents the security properties of the current implementation and the security improvements that would be necessary before a public production release.

#### Current Security Posture

**Authentication boundaries are enforced at the HTTP layer.** Every internal endpoint requires the X-Internal-Key header. Every user-facing endpoint (except `/api/v1/auth/identify` and `/api/v1/models`) requires a valid JWT. Unauthenticated requests receive 401 responses with no information about what the endpoint does.

**Secrets are stored in environment variables, not code.** The JWT secret, internal API key, database passwords, and LLM provider API keys are loaded from environment variables at startup. They are never committed to git, never returned in API responses, and never logged.

**The billing service has no public endpoints.** Even if the billing service's URL were somehow disclosed (e.g., via a browser network tab), none of its endpoints respond to requests without the X-Internal-Key header. This eliminates the possibility of direct credit manipulation by users.

**JWT expiration is enforced.** Access tokens expire after 24 hours. A stolen token has a bounded window of misuse.

#### Security Limitations in the Proof-of-Concept

**Email-only authentication is not production-grade.** The current authentication mechanism accepts any email address without verification. In production, email verification (or an OAuth2 provider like Google) would be required to prevent account impersonation.

**The shared JWT secret is a single point of trust.** If the JWT secret is compromised, all services are compromised simultaneously. In production, rotating the secret requires redeploying all three user-facing services simultaneously. A more robust approach would use asymmetric JWT (RS256) with a public/private key pair, allowing the secret signing key to be held by only the issuing service.

**No rate limiting on authentication.** The `/api/v1/auth/identify` endpoint has no rate limiting, making it susceptible to enumeration attacks (discovering which email addresses have accounts). Rate limiting and CAPTCHA would be required in production.

**No HTTPS enforcement in local development.** Local Docker Compose runs on HTTP. In production on Render, all services are served over HTTPS by Render's reverse proxy. However, the services themselves do not enforce HTTPS, so if deployed behind a non-HTTPS proxy, data would be transmitted in plaintext.

**The ngrok tunnel for Ollama is not secure for multi-user deployment.** The ngrok URL is shared in the billing service's environment variable. Any service with the correct X-Internal-Key can make inference calls through this tunnel, and all inference traffic passes through ngrok's infrastructure. For a production deployment, a dedicated cloud-hosted model serving endpoint (with proper authentication and rate limiting) would replace the ngrok arrangement.

---

### 3.15 Final Validation

The final validation of the ProductiveSocial proof-of-concept addresses the four specific concerns raised by the supervisor:

**1. Verifiable validation and baseline (addressed in Section 2.7)**

The experiment table in Section 2.7 documents 12 specific tests with explicit baselines, metrics, results, and conclusions. The most significant validation — the quality of LLM-generated analyses versus a rule-based baseline — shows a clear and substantial improvement: LLM scores of 3.9–4.2/5 versus baseline scores of 1.0–1.5/5 across three analysis types.

**2. Named competitors and comparative analysis (addressed in Section 1.3)**

Seven specific competing applications were analyzed across six dimensions, producing a comparison table that substantiates the claim of novelty. The unique combination of four productivity modules with a cross-domain AI analysis layer is not present in any of the analyzed competitors.

**3. Microservices architecture description (addressed in Sections 3.2, 3.3, 3.4)**

The architecture is described in detail: four services with explicit data model descriptions, inter-service communication protocols (JWT for client-to-service, X-Internal-Key for service-to-service), and the UserRegistry mechanism for cross-service identity. The service interaction diagram is:

```
psocial_user_dashboard ─── JWT ──▶ psocial_analytics
                       ─── JWT ──▶ psocial_selfmanager
                       ─── JWT ──▶ psocial_timer

psocial_admin_dashboard ── X-Internal-Key ──▶ psocial_billing
                        ── JWT ──────────────▶ psocial_analytics

psocial_analytics ── X-Internal-Key ──▶ psocial_selfmanager
                  ── X-Internal-Key ──▶ psocial_timer
                  ── X-Internal-Key ──▶ psocial_billing

psocial_timer ── X-Internal-Key ──▶ psocial_selfmanager (time-log, fire-and-forget)

psocial_timer ── JDBC ──▶ selfmanager_db (users table — UserRegistry)
```

Fault tolerance is achieved through: non-blocking upstream calls in the analytics service (selfmanager and timer failures return empty data rather than failing the request), fire-and-forget time-log calls from the timer (timer is not blocked by selfmanager availability), independent databases per service (no shared state that could cause cascading failures), and service-level independent deployment (each service can be restarted without affecting the others).

**4. ML/LLM end-to-end scenario (addressed in Sections 2.4, 2.5, 2.7)**

A complete end-to-end scenario is documented: input data → prompt construction → LLM invocation → output extraction → credit deduction → report persistence → user display. The actual prompt structure, the actual LLM output for PRODUCTIVITY_SUMMARY, and the comparison with the rule-based baseline are presented in Section 2.7.

---

## Conclusion

This report has documented the full lifecycle of ProductiveSocial's first proof-of-concept: from the identification of the problem domain and a structured analysis of the competitive landscape, through the architectural design and technical implementation of four independent microservices, to end-to-end validation of the system's core value proposition.

### Summary of Contributions

**Domain and problem definition.** The report identified and documented a structural gap in the personal productivity tooling market: no existing application provides all four core productivity modules (task management, habit tracking, Pomodoro timing, and AI analysis) in a single platform, and no existing application provides an AI layer with simultaneous access to cross-domain behavioral data. This gap was substantiated through a comparative analysis of seven leading products — Todoist, Habitica, Forest, Notion, Day One, RescueTime, and Sunsama — each analyzed across six capability dimensions.

**Architectural design.** The ProductiveSocial architecture was designed around five explicit principles — service independence, single source of truth per domain, authenticated access boundaries, offline-first data ownership, and the credit system as a behavioral incentive. These principles produced specific, defensible design decisions: the UserRegistry's cross-database JDBC connection for canonical user identity; the billing service's transaction-log-as-balance model for provably consistent credit accounting; the analytics service's non-blocking upstream data collection for graceful degradation; and the shared JWT secret for cross-service authentication without a centralized identity provider.

**Technical implementation.** The implementation spans four backend services (Kotlin/Ktor for selfmanager, timer, and analytics; Python/FastAPI for billing), two Streamlit web dashboards (admin and user), and a scaffolded Kotlin Multiplatform mobile application. The technology choices — Kotlin, Ktor, FastAPI, PostgreSQL, Docker, Colima, Streamlit, KMP, SQLDelight — were each made for specific technical reasons documented in Section 2.2, with the overarching goal of using the best tool for each job rather than forcing one language or framework on the entire system.

**ML integration.** The AI analysis pipeline implements a retrieval-augmented generation (RAG) pattern in which live user data is assembled at inference time rather than from a vector database. Six analysis types with distinct prompts were designed and tested. Validation against a rule-based baseline demonstrated substantial quality improvement: LLM scores of 3.9–4.2/5 versus baseline scores of 1.0–1.5/5 across three analysis types. The analyses identified cross-domain behavioral correlations and produced specific, actionable recommendations — capabilities that are definitively beyond any rule-based approach.

**Validation.** Twelve documented experiments verified functional correctness (JWT cross-service auth, credit arithmetic, sync idempotency, graceful degradation), LLM quality (three analysis type comparisons with example inputs and outputs), and operational behavior (cold start latency, warm latency, failed prediction credit exemption).

### Current Limitations

The proof-of-concept demonstrates the core value proposition in a fully functional system but has several limitations that distinguish it from a production-ready platform:

- **Mobile application completeness.** The KMP application is scaffolded with the authentication flow and SQLDelight schema in place, but the task, habit, routine, timer, and analytics screens are not yet implemented. The primary user interface for end users remains incomplete.

- **LLM hosting.** The production deployment uses a personal Ollama instance exposed via ngrok. This is not scalable (one developer's machine), not reliable (the machine must be running and connected), and not appropriate for a multi-user system. Production readiness requires switching to a cloud-hosted LLM provider (Anthropic Claude or OpenAI GPT-4o) or a cloud-hosted Ollama deployment.

- **Deployment tier.** Render's free tier introduces 50-second cold starts for JVM services after 15 minutes of inactivity. A paid deployment tier with persistent containers would eliminate this user experience problem.

- **Authentication model.** The email-only find-or-create authentication is suitable for a proof-of-concept but not for a system with real user data. Email verification, OAuth2 provider integration, or password-based authentication would be required.

### Roadmap: Next Development Phase

The next phase of development addresses the above limitations in priority order:

**Phase 2 (Immediate):**
1. Complete KMP mobile application screens for tasks, habits, routines, and the Pomodoro timer
2. Configure Anthropic Claude as the production LLM provider in the billing service
3. Upgrade Render deployment to eliminate cold starts
4. Implement email verification in the authentication flow

**Phase 3 (Medium-term):**
5. Implement the social module as a new microservice (public profiles, achievement sharing, community feed)
6. Add the journal module with AI-assisted day synthesis
7. Implement push notifications for habit reminders and routine start times
8. Add the notes module with entity cross-linking

**Phase 4 (Long-term):**
9. Accumulate real user behavioral data and begin training proprietary predictive models
10. Deploy trained sklearn models via the billing service's existing inference pathway
11. Build on-device model inference for fully offline AI features
12. Third-party integrations (Google Calendar, wearables)

### Architectural Readiness for Growth

The architecture is explicitly designed to accommodate the Phase 3 and Phase 4 additions without requiring structural changes to the existing services. New services (social, journal, notes) can be added as independent microservices that share the same user identity model (selfmanager integer user IDs) and the same authentication mechanism (shared JWT secret or X-Internal-Key). The billing service's model registry and sklearn inference pathway are already in place to receive trained models when they are ready. The analytics service's prompt system can be extended with new analysis types by adding a new case to the `buildPrompt()` function and a new analysis type constant — no other service changes are required.

ProductiveSocial is, at its core, a platform built for what it will become as much as for what it already is. The decisions made in this first phase — the shared user identity model, the internal API boundaries, the transaction-based credit ledger, the offline-first sync protocol — are not implementation choices that will need to be revisited as the platform grows. They are the architectural foundation on which the full vision of an intelligent productivity operating system can be built.

---

## References

---

## Appendix A: Glossary of Technical Terms

**API (Application Programming Interface):** A defined set of rules and endpoints through which one software system communicates with another. In the context of this project, each service exposes an HTTP API that clients and other services call to create, read, update, or delete data.

**Asynchronous programming:** A programming model in which operations that would otherwise block (waiting for a network response, a database result, a timer) are initiated and then the program continues running other work; when the operation completes, a callback or continuation resumes processing. Kotlin coroutines implement this model.

**Base64url encoding:** A variant of Base64 encoding that is safe for use in URLs and HTTP headers (replacing `+` with `-` and `/` with `_`). Used to encode the header and payload sections of a JWT.

**CORS (Cross-Origin Resource Sharing):** An HTTP mechanism that controls which web origins are permitted to make requests to a server. All Ktor services are configured to allow requests from the dashboard origins.

**CRUD:** Create, Read, Update, Delete — the four fundamental operations on persistent data, corresponding to HTTP POST, GET, PUT/PATCH, and DELETE respectively.

**DTO (Data Transfer Object):** A simple data class used to carry data between processes. In Kotlin services, DTOs are defined as data classes that map to JSON request/response bodies.

**HMAC (Hash-based Message Authentication Code):** A cryptographic construction that combines a hash function (SHA-256 in the case of HS256 JWTs) with a secret key to produce a message authentication code. Used as the JWT signature algorithm.

**HTTP header:** A key-value pair included in an HTTP request or response to convey metadata. The `Authorization` header carries the JWT bearer token; the `X-Internal-Key` header carries the service-to-service authentication key; `Content-Type` specifies the format of the request body.

**Idempotency:** A property of an operation whereby repeating it multiple times produces the same result as performing it once. The sync endpoint is idempotent: submitting the same sync request twice creates the same records once, not twice. This is critical for retry safety.

**JDBC (Java Database Connectivity):** A Java API for connecting to relational databases. Kotlin services use JDBC (with HikariCP for pooling) to connect to PostgreSQL. The UserRegistry uses a raw JDBC connection rather than an ORM, because it executes a single specific SQL statement that benefits from explicit control.

**JVM (Java Virtual Machine):** The runtime environment for Java and Kotlin programs. A compiled Kotlin class runs on the JVM, which provides garbage collection, JIT compilation, and platform independence. The JVM's startup time (~2–3 seconds) is the reason JVM-based services have longer cold starts than the Python billing service.

**JWT (JSON Web Token):** A compact, URL-safe format for representing claims as a JSON object, signed with HMAC or RSA. See Section 2.2.6 for a full description.

**KMP (Kotlin Multiplatform):** A technology for sharing Kotlin code across multiple platforms (JVM, iOS, JavaScript, native). See Section 2.2.8 for a full description.

**LLM (Large Language Model):** A neural network model trained on large amounts of text data that can generate coherent, contextually appropriate text responses to prompts. Examples: Anthropic Claude, OpenAI GPT-4o, Meta Llama. Used in ProductiveSocial via the billing service's inference engine.

**Microservice:** An independently deployable software service with a single, focused responsibility. See Section 2.2.1 for a full description.

**ORM (Object-Relational Mapper):** A library that translates between the object-oriented representations used in application code and the relational representations used in SQL databases. Exposed (used in Kotlin services) and SQLAlchemy (used in the billing service) are the ORMs in this project.

**Prompt engineering:** The practice of designing text inputs (prompts) to elicit specific types of responses from a language model. See Section 2.5 for a full description of the analysis-type-specific prompts used in this project.

**RAG (Retrieval-Augmented Generation):** A pattern for LLM-based applications in which relevant data is retrieved from a data source and included in the LLM prompt at inference time, providing the model with current, specific context that it could not have learned from pre-training alone. ProductiveSocial's analytics pipeline is an implementation of this pattern.

**REST (Representational State Transfer):** An architectural style for HTTP APIs characterized by stateless request/response pairs, resource-oriented URLs, and standard HTTP verbs (GET, POST, PUT, DELETE). All ProductiveSocial services expose REST APIs.

**Singleton:** A design pattern in which a class has at most one instance in the application, shared by all code that depends on it. Koin's `single { }` declaration creates singletons for all services and HTTP clients.

**Tombstone:** A record marking the deletion of another record, used in sync protocols to communicate that a record has been deleted on the server and should be removed from the client's local database.

**UUID (Universally Unique Identifier):** A 128-bit identifier generated in a way that makes collisions practically impossible. Used as client-assigned IDs for all synchronized entities, enabling offline record creation without server coordination.

---

## References

1. Allen, D. (2001). *Getting Things Done: The Art of Stress-Free Productivity*. Viking.

2. Cirillo, F. (2006). *The Pomodoro Technique*. FC Garage.

3. Newport, C. (2016). *Deep Work: Rules for Focused Success in a Distracted World*. Grand Central Publishing.

4. Clear, J. (2018). *Atomic Habits: An Easy and Proven Way to Build Good Habits and Break Bad Ones*. Avery.

5. Leroy, S. (2009). Why is it so hard to do my work? The challenge of attention residue when switching between work tasks. *Organizational Behavior and Human Decision Processes*, 109(2), 168–181.

6. Newman, S. (2021). *Building Microservices: Designing Fine-Grained Systems* (2nd ed.). O'Reilly Media.

7. Richardson, C. (2018). *Microservices Patterns: With Examples in Java*. Manning Publications.

8. Lewis, J., & Fowler, M. (2014). Microservices: a definition of this new architectural term. *martinfowler.com*. https://martinfowler.com/articles/microservices.html

9. Brown, T., Mann, B., Ryder, N., et al. (2020). Language Models are Few-Shot Learners. *Advances in Neural Information Processing Systems*, 33, 1877–1901.

10. Vaswani, A., Shazeer, N., Parmar, N., et al. (2017). Attention Is All You Need. *Advances in Neural Information Processing Systems*, 30.

11. Todoist Product Team. (2024). *Todoist: The to-do list to organize work and life*. https://todoist.com

12. Habitica Team. (2024). *Habitica: Gamify Your Life*. https://habitica.com

13. Notion Labs, Inc. (2024). *Notion: Connected workspace for your notes, docs, and projects*. https://notion.so

14. Day One Team. (2024). *Day One: Journal App*. https://dayoneapp.com

15. RescueTime Team. (2024). *RescueTime: Time Management Software*. https://rescuetime.com

16. Touchlight Ltd. (2024). *Sunsama: Daily Planner*. https://sunsama.com

17. Kotlin Foundation. (2024). *Kotlin Multiplatform*. https://kotlinlang.org/docs/multiplatform.html

18. JetBrains. (2024). *Ktor: A framework for building asynchronous server applications*. https://ktor.io

19. FastAPI. (2024). *FastAPI: Modern, fast web framework for building APIs with Python*. https://fastapi.tiangolo.com

20. Ollama. (2024). *Ollama: Get up and running with large language models*. https://ollama.ai

21. Anthropic. (2024). *Claude: AI assistant by Anthropic*. https://anthropic.com

22. SQLDelight. (2024). *SQLDelight: Typesafe SQL for Kotlin Multiplatform*. https://cashapp.github.io/sqldelight/

23. Render. (2024). *Render: The fastest way to host all your web apps*. https://render.com

24. Neon. (2024). *Neon: Serverless Postgres*. https://neon.tech

25. Fowler, M. (2002). *Patterns of Enterprise Application Architecture*. Addison-Wesley.

26. Deci, E. L., & Ryan, R. M. (1985). *Intrinsic Motivation and Self-Determination in Human Behavior*. Springer.

27. Sweller, J. (1988). Cognitive load during problem solving: Effects on learning. *Cognitive Science*, 12(2), 257–285.

28. Cirillo, F. (2018). *The Pomodoro Technique: The Acclaimed Time-Management System That Has Transformed How We Work*. Currency.

29. JetBrains. (2024). *Kotlin Multiplatform overview*. https://kotlinlang.org/docs/multiplatform.html

30. JetBrains. (2024). *Ktor documentation: Kotlin asynchronous framework*. https://ktor.io/docs/

31. FastAPI. (2024). *FastAPI documentation*. https://fastapi.tiangolo.com/

32. Docker Inc. (2024). *Docker documentation: Get started*. https://docs.docker.com/

33. Colima. (2024). *Colima: Container runtimes on macOS (and Linux) with minimal setup*. https://github.com/abiosoft/colima

34. SQLDelight. (2024). *SQLDelight: Typesafe SQL for Kotlin Multiplatform*. https://cashapp.github.io/sqldelight/

35. Tiangolo, S. (2024). *Pydantic documentation*. https://docs.pydantic.dev/

36. Jones, M., Bradley, J., & Sakimura, N. (2015). *RFC 7519: JSON Web Token (JWT)*. IETF. https://datatracker.ietf.org/doc/html/rfc7519

37. Atlassian. (2024). *Streamlit documentation*. https://docs.streamlit.io/

38. Meta AI. (2024). *Llama 3.2: Open foundation and fine-tuned chat models*. https://ai.meta.com/
