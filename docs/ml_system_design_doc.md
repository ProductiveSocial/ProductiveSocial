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
  [//Блок-схема решения](https://drive.google.com/file/d/1d9CVSUS3K3sv02dq-EpMQfZmihcB6Cln/view?usp=sharing)
  
## The Stages of Solving the Problem

### 1. Data Preparation Stages

#### 1.1 Data Preparation
For ProductiveSocial the necessary data for model pre-training are publicly available datasets about human productivity patterns.
Most productivity datasets are biased toward office workers, so to mitigate this a diverse synthetic data representing students, freelancers and night-shift workers can be included.

The primary categories of external data for pre-training:
- Behavioral Logs: Datasets like the 90-Day Habit Tracker or Kaggle Time Management Insights. These provide correlations between sleep, exercise, and self-reported productivity. 
- Proof of Work (Wearables): Fitbit Consumer/PMData datasets. These are used to train the model to recognize "active" vs. "sedentary" periods as proxies for focus. 
- Textual Summaries: Publicly available journaling prompts and "Day-in-the-life" blogs (scraped from Reddit/Productivity forums) to teach the LLM the desired narrative style.

The main source of data for the model should be user generated data.
#### 1.2 Data Sourcing and Generation
- External - Sourced from Kaggle and open-source lifelogging repositories (e.g., PMData)
- Internal - On-device generated data from the app modules.
- Synthetic - Usage of a LLM to generated synthetic productivity reports to boost the training set for the small on-device model.

#### 1.3. Confidentiality & Privacy
A local Named Entity Recognition (NER) layer should be used to strip phone numbers and specific locations before syncing user data.

### 2. Model development and Personalization
#### 2.1 Pre-training + Fine-tuning Strategy
1. Global Pre-training: Train a Small Language Model (SLM) on external and synthetic data to understand the structure of a journal entry.
2. On-Device Personalization (The Refinement): As the user edits their "Smart Journal," the model performs Low-Rank Adaptation (LoRA) locally. This adjusts the model's weights to match the user's specific vocabulary and goal-priority patterns.

#### 2.2 ML Metrics
- Loss Function - Weighted Cross-Entropy (prioritizing the correct identification of "Goal-related" tasks over "General" tasks).
- Primary Metric - Edit Distance (Levenshtein). We measure the difference between the ML draft and the user's final version.Target: 
  - Average Edit Distance should decrease by $15\%$ every month.

#### 2.3 ML Validation Scheme
We use a "User-Fold" Cross-Validation:
- Train on 100 "Persona" types (simulated users). 
- Test on 20 "Held-out" personas to ensure the model generalizes to different lifestyles before it even meets a real user.\

#### 2.4 Baseline vs. Advanced Solution
- Baseline - A rule-based system using string templates (e.g., "You finished 3 tasks today"). 
- Advanced - The fine-tuned SLM that can infer sentiment (e.g., "Despite the long meetings, you maintained focus on the Python project").

### 3. Inference and Model Optimization
Since the goal is an "Offline-First" experience, we cannot rely on cloud GPUs. We must squeeze the model into the mobile device's RAM and CPU/NPU.

#### 3.1 Model Quantization and Compression
- To fit a Small Language Model (SLM) like Phi-3 or Gemma-2b on a phone, we must reduce its size. 
- Technique: 4-bit Quantization (using GGML or bitsandbytes). This reduces a 5GB model to roughly 1.2GB - 1.8GB, making it viable for modern smartphones with 6GB+ RAM. 
- Format: Convert models to TFLite (Android) or CoreML (iOS) to leverage hardware acceleration (NPU/GPU). 
- Memory Footprint Calculation:
    Memory ~= (Parameters * Bits)/8
  - For a 2 billion parameters model ~= 1 GB of VRAM.

#### 3.2 ML Execution Trigger Logic
Running a generative model is resource-intensive so we define two triggers
- Scheduled (Once every 24h) - At a user defined time or during night time when the user is presumably sleeping and the phone is charging.
- On-demand - When the user manually prompts for a report.

### 4. Business Rules
The ML output must be filtered through "Business Rules" to ensure it remains helpful, safe, and aligned with the "Productive Social" philosophy.

#### Rule 1
if the model's confidence score for a generated task prediction is below 0.4, the system will not auto-fil the gap, instead it will prompt the user:
"We noticed a gap in your log - did you work on (goal x) on (given day)?"
#### Rule 2
The ML must never suggest "deleting" a goal or habit. It can suggest on how to improve or adjust a goal. The ML system should be hard-coded to be a supportive coach, not a critical judge
#### Rule 3
Before the model generates a social "Success Blueprint," a rule-based layer ensures:
Removal of all numerical identifiers (phone numbers, addresses).
Replacement of specific project names with generic "Sector" tags (e.g., "Secret Project X" becomes "Software Development").

### 5. Final Report and Business Metrics
| Metric                | Target | Method of Measurement                                                         |
|-----------------------|--------|-------------------------------------------------------------------------------|
| User Effort Reduction | >50%   | Comparison of time spent writing a manual journal vs. editing an AI draft.    |
| Inference Latency     | <15s   | P95 of time from "Trigger" to "Draft Displayed."                              |
| Habit Retention       | +15%   | Success rate of users maintaining a 14-day streak vs. a non-ML control group. |

### 6. Implementation and Production

#### 8.1 Solution Architecture
- Local Layer: Kotlin/Swift app utilizing TFLite/CoreML runtimes for inference.
- Cloud Layer: GoLang microservices for social feed management and anonymized "Success Blueprint" storage.

#### 8.2 Infrastructure & Scalability
- Stack: AWS Lambda for lightweight metadata sync; PostgreSQL for social data.
  - Why: Serverless architecture scales automatically and fits the "low-cost" requirement by only charging for active sync time.

#### 8.3 Security & Data Privacy
- GDPR Compliance: Users have a "Right to Erasure" (one-click wipe of local and remote data). No PII (Personally Identifiable Information) is sent to the cloud for ML processing.
- NER Layer: Anonymizes text before social sharing to prevent accidental disclosure of project names or contact info.

#### 8.4 Risks and Mitigation
| Risk          | Impact | Mitigation Strategy                                                     |
|---------------|--------|-------------------------------------------------------------------------|
| Model Drift   | Medium | Periodic on-device re-tuning (LoRA) based on user edits.                |
| Battery Drain | High   | Restrict heavy inference/training to when the device is charging.       |
| Hallucination | High   | Use "Rule 1" (Confidence threshold) to avoid generating false activity. |
