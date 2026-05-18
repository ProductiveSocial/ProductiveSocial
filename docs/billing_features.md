# psocial_billing — How It Works

## Overview

Billing is a purely internal service. It has no public-facing endpoints, no user authentication, and no user table. Every endpoint requires the `X-Internal-Key` header — clients never call billing directly. Users are identified solely by their selfmanager integer `user_id`, passed in as a string on every request. Billing does not create or manage user records; it only knows users by their ID.

---

## Credits and Transactions

Every user's credit balance is derived from their transaction history — specifically from the `balance_after` field on their most recent transaction. There is no separate balance column to update; instead, every credit movement (deposit, charge, or refund) is recorded as a new transaction row with an `amount`, `type`, `description`, an optional link to a related prediction, and a `balance_after` snapshot. Querying the balance means finding the single most recent transaction for the user and reading its `balance_after`. If no transactions exist yet, the balance is zero.

The first time billing is called for a user — specifically when `POST /api/v1/internal/predict` is called — the service checks whether that user has any existing transactions. If they don't, it auto-creates a welcome deposit of `DEFAULT_CREDITS_ON_REGISTER` credits (default: 100) before proceeding. This means every new user starts with credits without any explicit onboarding step.

Every successful prediction deducts `cost_per_prediction` credits from the user's balance and records a CHARGE transaction. Failed predictions are not charged — if the inference fails, no transaction is written and the user keeps their credits.

---

## Prediction Lifecycle

When a prediction request arrives at `POST /api/v1/internal/predict`, the service walks through a defined sequence:

1. **Welcome credits** — if this is the user's first time, a deposit transaction is created before anything else
2. **Validation** — the requested model must exist and be active, and the user must have enough credits to cover its cost
3. **Pending record** — a prediction row is created with status `PENDING`, capturing the user, model, and input data
4. **Inference** — the service calls the appropriate LLM or sklearn backend
5. **Success path** — the prediction status is set to `SUCCESS`, the output text is stored, `completed_at` is recorded, and a CHARGE transaction is written against the user's balance
6. **Failure path** — the prediction status is set to `FAILED`, the error message is stored, and no credit transaction is created

---

## LLM Inference

Billing supports three LLM providers, and each registered model specifies which one to use:

**Ollama** is the default for local development. It exposes an OpenAI-compatible API, and the billing client sends a `ngrok-skip-browser-warning` header and follows redirects to support ngrok tunnel setups (used when running Ollama locally while billing runs on Render).

**Anthropic** calls Claude models using the Anthropic SDK. Requires `ANTHROPIC_API_KEY`.

**OpenAI** calls GPT models using the OpenAI SDK. Requires `OPENAI_API_KEY`.

The prompt is constructed from whatever structured fields are present in the request (tasks, habits, routines, pomodoro stats, context, query, custom prompt), combined with the model's own `system_prompt` stored in its registry record. The caller — analytics — is responsible for building and formatting the input data; billing receives it and forwards it to the LLM.

---

## sklearn Inference

For sklearn models, billing loads the `.pkl` file using joblib with an in-memory cache so subsequent predictions don't re-read from disk. Input can be provided as `{ "features": [...] }` or `{ "data": [[...]] }`. The response is `{ "prediction": [...] }`, with an additional `{ "probabilities": [...] }` field if the model implements `predict_proba`.

---

## Model Registry

Models are registered via `POST /api/v1/models` and stored in the `ml_models` table. Each model record carries its name, version, description, type (`LLM` or `SKLEARN`), LLM provider and model ID (for LLM types), a system prompt (for LLM types), cost per prediction in credits, and an active flag.

On startup, the service auto-seeds a default Ollama model (UUID `fb96de86-3c73-489d-8f02-aaf8343d6101`, 5 credits) and keeps it in sync with the `OLLAMA_MODEL` env var. Deactivating a model via `DELETE /api/v1/models/{id}` is a soft delete — it sets the active flag to false rather than removing the record, preserving the link from historical predictions.

`GET /api/v1/models` is the one endpoint that doesn't require the internal key — it's publicly accessible so user-facing dashboards and clients can list available models for display without needing backend credentials.

---

## Internal Endpoints

These are the endpoints analytics calls during normal operation, all protected by `X-Internal-Key`:

- `POST /api/v1/internal/predict` — run inference for a user; auto-deposits welcome credits on first call
- `GET /api/v1/internal/users/{selfmanager_user_id}/balance` — get the user's current credit balance
- `GET /api/v1/internal/users/{selfmanager_user_id}/transactions` — paginated transaction history with a `credits_balance` field in the response
- `POST /api/v1/internal/users/{selfmanager_user_id}/deposit` — add credits to a user's balance

---

## Admin Endpoints

Admin operations go through a separate set of endpoints, also protected by `X-Internal-Key`. These are called by the admin dashboard:

- `POST /api/v1/admin/users/{selfmanager_user_id}/deposit` — top up any user's credits manually
- `GET /api/v1/admin/predictions` — all predictions across all users, with optional status filtering
- `GET /api/v1/admin/analytics` — system-wide totals: total predictions, total credits charged, total credits deposited, active model count
