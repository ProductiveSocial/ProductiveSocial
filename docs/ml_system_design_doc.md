_Общие назначение проекта можно найти в папке 'definition'_

# ML System Design Doc - Productive Social

## 1. Зачем идем в разработку продукта?
Бизнес-цель Productive Social strives to become the definitive "Operating System for the Self" Our 
mission is to move beyond simple tracking and create an intelligent ecosystem capable of deep
behavioral analysis. By synthesizing data from various productivity tools, the system identifies 
individual habit patterns, both constructive and not, to provide users with hyper-personalized
reports and actionable recommendations.

 - Competitive advantages:
   - Integration - it unifies goal and habit ecosystem, task management, note-taking and focus tools into a single workflow, ensuring no productivity data is lost.
   - Individualized ML - it utilizes a dedicated Machine Learning engine that learns from unique user data
   - Social accountability - it introduces a specialized social layer designed to drive retention through "proof of work" and shared milestones, allowing for studying of habit stacks and progress of high-performers.
 
## 2. Business Requirements
#### 2.1 High-Levee Product Requirements
 - _Unified Productivity Suite_ - the platform must provide four core standalone modules - _Task Management, Note-taking, Pomodoro Timer and Journaling_ that function both independently and as a fully integrated ecosystem.
 - _Goal-Oriented_ - All user actions must map to long-term objectives.
 - _Offline-First Resilience_ All core productivity features must remain fully functional without an internet connection. Synchronization with cloud and social features should occur seamlessly once connectivity is restored.
 - _Data-Driven Feedback Loop_ - THe system must collect anonymized behavioral data from the three primary modules to generate "smart" journal entries and productivity reports.

| Feature                      | Requirement Description                                                                                                                                                                                                   |
|:-----------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Goal & Habit Ecosystem**   | A hierarchical system where **Tasks** are mapped to **Long-term Goals**. Includes native **Habit Tracking** (recurring triggers, streaks, and frequency logic) to ensure daily actions align with overarching objectives. |
| **Note-Taking**              | Rich-text support (Markdown), image attachments, and linking notes to specific tasks, habits, or goals.                                                                                                                   |
| **Pomodoro Timer**           | Customizable focus/break intervals with the ability to "tag" focus sessions to specific habits or goal-related tasks.                                                                                                     |
| **Rich Journaling**          | Manual daily entries combined with **Contextual Metadata** (automatic injection of habits completed, tasks checked off, and focus time).                                                                                  |
| **Social Productivity Feed** | Public profiles featuring **"Success Blueprints"** and a milestone-based feed for progress benchmarking and habit-stacking inspiration.                                                                                   |

## Ограничения
   - Infrastructure Costs: The project must minimize reliance on expensive paid cloud platforms (e.g., AWS, GCP). Architecture should prioritize low-cost or open-source alternatives where possible
   - On-Device Processing: To maintain the "Offline-First" requirement and reduce server costs, ML inference for journaling should ideally happen on the user's device.
   - Distribution: Deployment is restricted to the Google Play Store (Android) and Apple App Store (iOS) for the initial pilot phase.

#### Pilot Business Process (integration of ML)
The ML model is not a separate feature but the connective tissue of the user experience.
 - _Data Aggregation_ - The application continuously logs interactions within the Goal, Task, Pomodoro and Note modules.
 - _Daily Synthesis_ - Every 24 hours the ML model processes these logs to generate a "Smart" journal entry"
 - _Absence recovery_ - if the user is inactive for 2+ days, the system performs "Predictive Backfilling" it analyzes previous patters and pending tasks to suggest what the user likely accomplished, allowing the user to maintain their streak and historical record with minimal manual effort.

#### Success Criteria
While the app working as intended is the ultimate goal, we will measure success through these specific metrics:
 1. ML Accuracy & Utility - The acceptance rate of ML-generated journal drafts should exceed 60%.
    - the Prediction Error (difference between predicted tasks and actual user corrections) should decrease over a 30-day period as the model learns individual behavior.
 2. User Retention - A measurable increase in Daily active users specifically driven by the Social Feed and desire to maintain Public productive blueprints.
 3. Data Integrity - A seamless synchronization of data across all different app modules.
 
## 3. Project Scope
 - In-Scope:
   - The full four modules integration;
   - Predictive ML;
   - User productivity benchmarking;
   - Offline-first sync - Local database storage with seamless cloud synchronization.
   
 - Out-of_Scope:
   - Direct Messaging (DMs): To prevent distraction and maintain focus. 
   - Generic Social Content: No status updates or media unrelated to productivity/goals. 
   - Third-Party Integrations: External tool sync (Notion/Google) is deferred to post-MVP.

## 4. Solution Assumptions

| Entity      | Description                                          | Relationship                                          |
|:------------|:-----------------------------------------------------|:------------------------------------------------------|
| **Goal**    | The high-level objective (e.g., "Learn Python").     | Has many Habits and Tasks.                            |
| **Habit**   | Recurring behavioral patterns (e.g., "Code 1 hour"). | Linked to a Goal; generates "Logs".                   |
| **Task**    | One-time discrete actions (e.g., "Install VS Code"). | Linked to a Goal or a specific Habit.                 |
| **Session** | Metadata from the **Pomodoro Timer**.                | References a Task or Habit; provides "Proof of Work". |
| **Journal** | The synthesis of all data for a specific date.       | Aggregates all Sessions, Habits, and Tasks.           |

### 4.1 Data Hierarchy
The system is built on a hierarchical data model to ensure the ML engine understands the intent behind every action.
- Goals: The North Star (e.g., "Become a Senior Developer").
- Habits: Recurring actions that support the goal (e.g., "Code for 2 hours daily").
- Tasks: Discrete items supporting the goal (e.g., "Finish Chapter 4 of React docs").
- Events: Pomodoro sessions or Journal logs that verify the work done.
- 
### 4.2 Data Scientist tasks
1. Natural Language Summarization - Turning raw logs into a coherent daily journal narrative.
2. Gap-Filler Logic - Predicting activity for days when the user didn't log in
3. Anonymized Benchmarking Engine - Providing comparative analytics for the social feed.

#### Horizon and Nature of Processing
- _Analysis Horizon_ - Since the core function is the generation of a daily journal and productivity reports, the model operates in Near Real-Time. Generation triggers either at the end of a "Focus Session" or at a user-defined "Review Time" (e.g., 9:00 PM).
- _Processing Granularity_ - The model works at the Individual User Level. Data from different users is strictly siloed to ensure privacy and personalized habit modeling.
- _Justification_ - Productivity is highly subjective; a "good day" for a developer looks different from one for a student. Global models would fail to capture these nuances.

#### Data Quality Assumptions
_Textual Data_ - Notes and Journal entries are assumed to be "Structured-Unstructured" (Markdown/Text). We assume minimal OCR issues as data is entered digitally.
_Goal Consistency_ - We assume that the user’s Goal & Habit Catalog is relatively stable. If a user changes goals daily, the ML’s predictive accuracy will decrease.

#### Model Performance 
_Inference Performance_ - Journal generation should take $\le$ 30 seconds to maintain a smooth UI experience.
_User Role_ - The user is the final editor. The ML provides a Draft; the user verifies, edits, or deletes
_Justification_ - Full automation of a personal journal is philosophically impossible—the user's subjective reflection is the most important part of the process.

### 5. Постановка задачи
Build an ML/LLM pipeline to automatically synthesize a user's daily activity into a reflective "Smart Journal" through the following sequence:
1. _Entity & Event Extraction_ 0 Identify completed tasks, total focus time, and sentiment from notes.
2. _Behavioral Correlation_ - Map these events to established Goals and Habits to calculate progress.
3. _Generative Synthesis_ - Formulate a structured narrative that highlights wins, identifies bottlenecks, and suggests improvements for the next day.
4. _Gap Recovery_ - Predictively backfill missing days using historical probability patterns.
_Task Type_ - Pipeline extraction -> Normalization -> Narrative Generation.

#### Data Preparation
To the train the predictive models, we'll utilize all the app features:

| Feature Type    | Attribute Name      | Source        | Processing / Engineering                                                      |
|:----------------|:--------------------|:--------------|:------------------------------------------------------------------------------|
| **Target**      | `user_satisfaction` | Journal Edits | Binary: 1 if user saves draft with <20% changes, 0 if heavily edited/deleted. |
| **Textual**     | `note_content`      | Note Module   | Clean -> NLP Sentiment Analysis -> Embedding.                                 |
| **Numerical**   | `focus_efficiency`  | Pomodoro Logs | Ratio of (Planned Pomodoros / Completed Pomodoros).                           |
| **Categorical** | `goal_sector`       | Goal Module   | One-Hot Encoding (e.g., Career, Health, Hobby).                               |
| **Temporal**    | `activity_pattern`  | Event Logs    | Calculating "Peak Productivity Hours" (time of day vs. task completion).      |

## 6. Блок-схема решения
TODO()