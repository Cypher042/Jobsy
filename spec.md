# AutoJob Technical Specification

## 1. System Context

AutoJob is a Next.js application backed by MongoDB and Redis. Synchronous API routes handle authenticated reads and commands. Long-running work is submitted to BullMQ and processed by workers that execute LangGraph workflows.

```text
Browser
  -> Next.js pages and API routes
  -> MongoDB for durable state
  -> Redis/BullMQ for discovery and application jobs
  -> LangGraph workflow nodes
  -> LLM/vector-store services for personalization and grounding
  -> configured sandbox or external job API for submission
```

## 2. Core State Model

### User

Stores identity, authentication provider, candidate profile, resume references and artifacts, parsing feedback, activity state, and `applyPolicy`.

### Job

Stores source identity, company, title, location, remote flag, description, requirements, skills, questions, apply URL, and unique `jobHash`.

### AgentRun

Represents one discovery/application execution for a user. It tracks lifecycle status, counters, timestamps, kill-switch state, and workflow progress.

### JobMatch

Stores ranking output including overall match score, skill overlap, experience fit, constraint fit, evidence coverage, and ranking reason.

### ApplyQueue

Links a run, user, and job with queue status, policy decision, skip reason, attempts, and processing metadata.

### Application

Stores the submitted job relationship, resume variant, answers, validation state, status, attempts, receipt, error, and timeline.

### AgentLog

Stores workflow observability data including run context, stage, level, message, checkpoint, and metadata.

## 3. Workflow Contracts

### Discovery Workflow

1. `load-run` loads the run, user profile, and policy and verifies the run can proceed.
2. `artifact` prepares resume artifacts, evidence, skills, and proof links.
3. `job-discovery` fetches jobs, deduplicates them, calculates match dimensions, and evaluates policy.
4. `queue-decision` persists queued or skipped outcomes and enqueues eligible application work.
5. `log` records stage outcomes and errors.

Discovery state includes fetched IDs, counts, ranked IDs, match data, allowed/skipped IDs, skip reasons, policy checks, queue IDs, and cooldown IDs.

### Apply Workflow

1. `load-run` loads the job-specific application context.
2. `artifact` prepares the selected resume and evidence bank.
3. `personalize` generates the cover letter, answers, and requirement evidence map.
4. `hallucination-guard` validates grounding and calculates confidence and risks.
5. `apply` submits only validated content to the configured endpoint and stores the result.
6. `finalize`, `tracker`, and `log` update queue/application state and observability.

The apply workflow must stop before submission when the kill switch is active, the run is stopped, required artifacts are absent, or validation fails.

## 4. API Surface

The current application exposes route groups for:

- Authentication: `/api/auth/[...nextauth]`
- Workflow: `/api/workflow/start`, `/api/workflow/stop`, `/api/workflow/discovery`
- Runs and logs: `/api/agent-run`, `/api/logs`, `/api/logs/stats`
- Jobs: `/api/jobs`, `/api/jobs/[id]`, `/api/jobs/skipped`
- Applications: `/api/applications`
- Policy: `/api/apply-policy`
- Resumes: `/api/resume/list`, `/api/resume/[id]`, `/api/resume/save`, `/api/resume/upload`, `/api/resume/feedback`
- Artifacts: `/api/artifacts`, `/api/artifacts/generate`

Commands should validate identifiers and ownership before reading or mutating user-owned data. API responses should return stable `success`/`error` fields consistent with the existing route style.

## 5. Policy Evaluation

For each candidate-job pair, evaluate:

1. Remote-only and location constraints.
2. Visa requirement when applicable.
3. Minimum match score.
4. Blocked company and role lists using normalized matching.
5. Company cooldown since the most recent application.
6. Maximum applications per day.
7. Kill-switch and active-run state.

The evaluator should return a structured decision, not only a boolean, so the queue and UI can show the failed checks and a stable skip reason. Policy evaluation must be deterministic for the same inputs and current database snapshot.

## 6. Grounding and Safety

- Candidate claims must come from profile, resume, project, experience, or proof-link artifacts.
- Generated content must retain its validation result and risk list.
- The hallucination guard is an enforcement boundary, not merely an informational score.
- External API responses must be treated as untrusted input and persisted with bounded size.
- Secrets and provider credentials must remain in environment configuration and never be stored in planning documents or client code.
- Logs should avoid passwords, tokens, full resume contents, and unnecessary personal data.

## 7. Reliability

- Queue jobs must be idempotent by run, user, and job identity where possible.
- Worker failures should update queue/application status and preserve the error.
- Retries must be bounded and must not create duplicate successful applications.
- Workflow checkpoints should make stop requests observable and effective.
- Redis, MongoDB, LLM, and vector-store outages should fail with actionable logs and leave durable state consistent.

## 8. Non-Functional Requirements

- Authorization: every user-owned read and write is scoped to the authenticated user.
- Privacy: resume and application data are treated as sensitive personal data.
- Performance: dashboard reads remain responsive while workers process long-running jobs asynchronously.
- Observability: each workflow stage emits structured logs tied to an agent run.
- Maintainability: workflow nodes remain small, testable units with explicit state inputs and outputs.
- Compatibility: provider adapters should isolate source-specific job schemas and submission behavior.

## 9. Testing Strategy

- Unit-test match scoring, normalized policy checks, cooldowns, daily limits, deduplication, and grounding decisions.
- Route-test authentication, ownership checks, invalid IDs, duplicate starts, and stop behavior.
- Worker-test retry, idempotency, partial failure, and status transitions.
- Integration-test MongoDB and Redis-backed discovery and apply flows with sandbox jobs.
- End-to-end test the candidate path: sign in, configure policy, upload resume, start run, inspect queue decision, and inspect application timeline.

## 10. Implementation Gaps to Track

- Production source adapters and external submission contracts are not defined by the current data model.
- Review-before-submit behavior is not yet represented as a first-class queue state.
- Scheduled runs and notification delivery need their own persistence and retry contracts.
- Retention, deletion, and export requirements need product decisions before production launch.
