# AutoJob Product Requirements Document

## 1. Product Vision

Give job seekers a controllable automation layer for repetitive application work: find better-fit jobs, prepare truthful materials, submit only within policy, and make every decision visible afterward.

## 2. Goals

1. Reduce time spent finding and triaging relevant jobs.
2. Increase application relevance through profile-aware matching and personalization.
3. Prevent applications that violate explicit candidate constraints.
4. Keep generated content grounded in resume and profile evidence.
5. Provide reliable status, error, and audit information for every run.

## 3. Non-Goals

- Replacing candidate judgment for all applications.
- Acting as a universal job-board crawler.
- Creating claims that are not supported by candidate artifacts.
- Optimizing only for application volume.

## 4. User Stories and Acceptance Criteria

### Profile and Resume

As a candidate, I can create an account and maintain my profile, resumes, variants, skills, experience, projects, and proof links.

Acceptance criteria:

- Unauthenticated users cannot access candidate-owned data.
- Resume artifacts can be associated with a user and selected for an application.
- Missing or invalid profile data produces an actionable error before a workflow begins.

### Policy Configuration

As a candidate, I can configure location, remote, visa, match-score, blocked-company, blocked-role, cooldown, daily-limit, and kill-switch settings.

Acceptance criteria:

- Policy values are persisted to the candidate record.
- Discovery checks policy before a job enters the application queue.
- The kill switch prevents new work from progressing to submission.

### Discovery and Ranking

As a candidate, I can start a discovery run that finds and ranks jobs against my profile.

Acceptance criteria:

- A run is created with a visible lifecycle status.
- Duplicate jobs are removed using their job hash and prior application state.
- Match results expose a score and a reason based on skill, experience, and constraints.
- The run records counts for found, queued, skipped, and applied work.

### Queue and Skip Decisions

As a candidate, I can understand why a job was queued or skipped.

Acceptance criteria:

- Eligible jobs create queue entries tied to the user, run, and job.
- Skipped jobs store a human-readable reason and policy context.
- Daily limits, blocked entities, cooldowns, and minimum score are enforced consistently.

### Personalized Application

As a candidate, I receive a tailored resume selection, cover letter, and answers to job questions.

Acceptance criteria:

- Generated content references available candidate artifacts.
- Requirement-to-evidence mapping and confidence information are retained.
- Content that fails grounding validation is blocked from submission and marked in the workflow.

### Submission and Tracking

As a candidate, I can see whether an application was submitted, retried, or failed and inspect its receipt or error.

Acceptance criteria:

- Submission attempts are persisted with status and timestamps.
- Successful submissions store an external receipt when provided.
- Failed work records an error and can follow the retry policy without duplicating a successful application.

### Workflow Control and Observability

As a candidate or operator, I can start, stop, and inspect a workflow.

Acceptance criteria:

- A second start is rejected while a run is already active.
- Stop requests are persisted and honored at workflow checkpoints.
- Agent logs include stage, level, message, and relevant run context.

## 5. Product Metrics

- Percentage of discovered jobs with explainable ranking and policy decisions.
- Percentage of queued jobs that complete without an unrecoverable worker error.
- Duplicate-application rate, target: zero.
- Policy-violation submission rate, target: zero.
- Grounding-block rate and manual review rate.
- Median time from discovery start to persisted queue decisions.
- Candidate-reported usefulness of matches and generated materials.

## 6. UX Requirements

- Dashboard shows current run status, counts, recent jobs, and workflow controls.
- Jobs, applied jobs, skipped jobs, resumes, and policy settings have dedicated navigable views.
- Destructive actions such as stopping a workflow communicate the resulting state.
- Empty, loading, failure, and unauthorized states are explicit.
- Logs and application timelines use plain language and stable status labels.

## 7. Dependencies and Constraints

- Next.js App Router and React provide the web application.
- NextAuth provides authentication.
- MongoDB/Mongoose persist users, jobs, runs, queues, applications, resumes, and logs.
- Redis and BullMQ provide asynchronous discovery and application workers.
- LangGraph orchestrates workflow nodes.
- LLM and vector-store services are optional runtime dependencies for personalization and grounding.

## 8. Open Product Decisions

- Which production job sources are supported first, and what terms permit automated submission?
- Should users approve each application before submission, or only configure an account-level automatic mode?
- What retention and deletion controls are required for resumes, generated content, and logs?
- Which application statuses should be synchronized after external submission?
- What notification channels should report failures, receipts, or policy changes?
