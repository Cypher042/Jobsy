# AutoJob Scope

## Product Summary

AutoJob is an authenticated job-search assistant that discovers relevant jobs, scores them against a candidate profile, enforces user-defined application policies, generates grounded application materials, and tracks the resulting workflow.

The product is designed for candidates who want application throughput without losing control over where and how their information is submitted.

## Problem

Job seekers spend substantial time searching, comparing listings, tailoring resumes, answering repeated questions, and tracking application status. Fully automatic submission can create additional risk when a job is a poor fit, violates a user constraint, or receives invented candidate claims.

AutoJob addresses both problems by combining automation with explicit policy checks, evidence-backed personalization, a hallucination guard, a kill switch, and an auditable timeline.

## In Scope

- Account creation and authentication through credentials or Google.
- Candidate profile, resume, resume variant, project, experience, skill, and proof-link management.
- Job ingestion from seeded, sandbox, dataset, or future external sources.
- Deduplication using a job hash.
- Match scoring across skill overlap, experience fit, and constraint fit.
- User policy enforcement for location, remote work, visa requirements, minimum match score, blocked companies, blocked roles, company cooldowns, and daily application limits.
- Discovery workflow execution through LangGraph and BullMQ workers.
- Queueing eligible jobs and recording skipped jobs with reasons.
- Personalized cover letters and application-question answers grounded in candidate artifacts.
- Validation of generated content before submission.
- Application submission through the configured job API or sandbox endpoint.
- Application, run, queue, and agent-log status tracking.
- Dashboard views for workflow control, job review, applications, skipped jobs, resumes, policies, and logs.

## Out of Scope for the Current Release

- Guaranteed access to or compatibility with every third-party job board.
- Applying to jobs without a configured submission endpoint.
- Fabricating experience, credentials, employment history, or answers.
- Fully autonomous changes to candidate policy or profile data.
- Interview scheduling, recruiter messaging, offer negotiation, or employment onboarding.
- Guaranteeing interviews, offers, ranking outcomes, or application delivery by an external site.
- Billing, team accounts, enterprise administration, or multi-user organization workflows.

## Primary Users

### Candidate

Owns the profile and application policy, uploads resumes, starts or stops discovery, reviews jobs, and monitors applications.

### Operator or Developer

Seeds or imports jobs, runs workers, inspects logs, configures integrations, and diagnoses failed workflows. This is an operational role rather than a separate end-user experience in the current release.

## Release Boundaries

### Current MVP

- Single-candidate workflow per authenticated user.
- Database-backed jobs and sandbox-oriented application flow.
- MongoDB persistence, Redis/BullMQ queues, LangGraph orchestration, and optional LLM/vector-store services.
- Manual workflow start and stop.
- Traceable outcomes for queued, skipped, submitted, retried, and failed applications.

### Later Releases

- Production job-source connectors with source-specific rate limits and adapters.
- Review-before-submit mode and per-job approval.
- Better resume parsing feedback loops and artifact versioning.
- Notifications, scheduled discovery, analytics exports, and richer application status synchronization.

## Success Measures

- Candidates can configure a policy and complete a discovery run without violating it.
- Every queued or skipped job has a persisted, explainable decision.
- Generated materials are grounded in stored candidate evidence before submission.
- A candidate can stop a running workflow and inspect its logs.
- Application outcomes and failures remain recoverable and auditable.

## Key Risks

- External job APIs can change schemas, authentication, or rate limits.
- LLM output can be inaccurate unless validation and evidence boundaries remain mandatory.
- Duplicate or stale listings can waste application capacity.
- Automated submissions can create reputational or compliance risk for the candidate.
- Worker, Redis, MongoDB, LLM, and vector-store failures can leave partial workflow state.
