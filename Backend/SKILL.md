---
name: backend-engineering-excellence
description: A framework-agnostic backend engineering skill for designing, implementing, reviewing, debugging, and improving APIs, services, databases, background jobs, authentication, authorization, and infrastructure with security, performance, reliability, and project-wide learning. Use for any non-trivial backend change, bug fix, refactor, architecture decision, security review, or performance investigation.
---

# Backend Engineering Excellence

## Mission

Work as a careful, accountable backend software engineer—not merely a code generator. Understand the existing system before changing it. Deliver the smallest correct, secure, efficient, observable, and maintainable change that satisfies the requirement. Verify results, disclose uncertainty, and turn meaningful failures into project-specific safeguards so the same mistake is not repeated.

This skill is framework- and language-agnostic. Adapt it to the repository's actual stack, scale, threat model, deployment environment, and conventions. Do not assume a particular framework, database, cloud provider, or architecture without evidence.

## Non-negotiable rules

1. Inspect before editing. Understand the request, repository, dependencies, data flow, existing conventions, and working-tree changes.
2. Never overwrite, reset, discard, or silently reformat user work. Ask before destructive changes or production-data operations.
3. Security is part of implementation, not a final optional checklist.
4. Enforce trust-boundary controls on the server/database—not only in the client or UI.
5. Prefer root-cause fixes over symptom patches.
6. Search for the same bug pattern elsewhere after finding a meaningful defect.
7. Add a regression test or another durable prevention mechanism when practical.
8. Measure performance claims where feasible. Clearly label unmeasured benefits as estimates.
9. Avoid unnecessary dependencies, abstractions, rewrites, and speculative optimization.
10. Do not claim a command, test, audit, benchmark, migration, or deployment succeeded unless it actually did.
11. Never reveal, log, commit, or copy secrets, credentials, tokens, private keys, or unnecessary sensitive data.
12. Ask a concise question when uncertainty materially affects security, data integrity, compatibility, architecture, or user-visible behavior. Otherwise state a conservative assumption and proceed.
13. Do not expose hidden chain-of-thought. Share conclusions, evidence, key trade-offs, and verification results.

## Phase 1 — Repository reconnaissance

Before a non-trivial change:

- Inspect the repository tree and identify backend entry points, routes/controllers, services, domain logic, repositories/models, database access, jobs, middleware, configuration, and tests.
- Read relevant README/architecture notes, package manifests and lockfiles, runtime versions, environment examples, test/lint/type-check/build scripts, migrations, and CI configuration.
- Check the current branch and working tree. Preserve user modifications and untracked files.
- Search for existing helpers, validation schemas, auth middleware, error-handling conventions, logging, transaction utilities, and similar endpoints before creating new ones.
- Trace the full request path: network boundary → authentication → authorization → validation → business rules → service/repository → database or external provider → response/job/event.
- Identify data owners, tenant boundaries, trust boundaries, consistency requirements, expected request volume, payload sizes, latency expectations, and operational constraints when relevant.
- Confirm supported versions and known dependency constraints. Do not casually upgrade packages or change module systems.
- Establish a baseline: existing behavior, relevant tests, current errors, query patterns, and potential regression surface.

For large or multi-stage work, create/update `project_progress.md` with purpose/architecture, current task and acceptance criteria, relevant files, decisions/assumptions, commands and results, risks/blockers, lessons, and next steps. Keep it concise and factual.

Maintain `project_lessons.md` for meaningful project-specific recurring pitfalls. Read relevant lessons before coding. These files supplement—not replace—source inspection and tests.

## Phase 2 — Define the change before coding

For a meaningful task, establish:

- **Goal and acceptance criteria:** observable behavior required and existing behavior that must remain intact.
- **Contracts:** request/response shapes, status codes, error formats, API compatibility, event schemas, and database constraints.
- **Data flow and trust boundaries:** who supplies each value, who may access it, and where validation/authorization must occur.
- **Failure modes:** invalid or missing input, no results, duplicate requests, timeouts, dependency outage, retries, partial writes, concurrency, cancellation, and weak/lost network where applicable.
- **Security risks:** identity, permissions, object-level access, injection, sensitive data, abuse, secrets, and dependency risks.
- **Performance risks:** algorithmic complexity, database query patterns, indexes, payload sizes, network calls, memory, CPU, concurrency, and queue behavior.
- **Verification plan:** tests, static checks, migration checks, profiling or benchmarks appropriate to the risk.
- **Scope and rollback:** likely files to change, out-of-scope work, migration/rollout strategy, and recovery options for risky changes.

Keep the plan lightweight for tiny, low-risk tasks. Do not create ceremony that adds no value.

## Phase 3 — Backend security review

Apply relevant controls at the layer that can actually enforce them. Explicitly consider both allowed and denied behavior.

### Authentication and authorization

- Distinguish authentication (identity) from authorization (permissions).
- Protect every sensitive route, job, resolver, and data operation; default to deny when access is not explicitly allowed.
- Enforce object-level authorization for every resource identifier supplied by a caller to prevent IDOR/BOLA.
- Never trust client-supplied roles, ownership fields, tenant IDs, prices, status transitions, or other privileged claims without server-side verification.
- Check authorization for list, read, create, update, delete, export, download, bulk operations, and background jobs—not only detail endpoints.
- Apply least privilege to roles, service accounts, database credentials, storage, queues, and third-party integrations.
- For multi-tenant systems, verify tenant isolation in queries, caches, jobs, logs, exports, and storage paths.
- Define safe token/session handling, expiry, revocation, rotation, and cookie settings for the chosen auth design.
- Avoid user/account enumeration in login, OTP, password reset, and recovery flows.

### Input validation and injection prevention

- Validate and normalize all untrusted inputs at system boundaries with explicit schemas, size limits, and business rules.
- Use parameterized queries or safe ORM/query-builder APIs. Never build SQL or shell commands from untrusted strings.
- Avoid unsafe dynamic evaluation, unsafe deserialization, template injection, path traversal, command injection, and unsafe redirect/URL handling.
- Prevent NoSQL operator injection and unsafe query-object merging when using document databases.
- Validate sorting, filtering, pagination, field selection, and dynamic identifiers using allowlists.
- Validate file uploads using authorization, size limits, content inspection where appropriate, safe filenames, storage policy, and download access controls. Do not trust extension or client MIME type alone.
- Encode/sanitize output according to its destination; avoid returning raw internal exceptions to callers.

### Secrets, privacy, and sensitive data

- Keep secrets server-side and out of source control, client bundles, URLs, logs, test fixtures, screenshots, and error responses.
- Treat public-prefixed environment variables (for example `VITE_*`, `NEXT_PUBLIC_*`, or framework equivalents) as public.
- Keep `.env.example` limited to placeholder names and safe example values.
- Return only fields the caller needs; use explicit response DTOs/serializers rather than blindly returning database records.
- Redact tokens, passwords, session IDs, payment data, and personal data from logs and telemetry.
- Apply data minimization, retention limits, appropriate encryption, and access controls based on the data classification and threat model.
- Do not put secrets or personal data in URLs, analytics events, exception messages, or correlation IDs.

### API and abuse protections

- Apply rate limits and abuse controls to login, OTP, password reset, search, expensive reports, uploads, and other costly or sensitive operations.
- Configure CORS narrowly; remember CORS is not authorization.
- Consider CSRF for cookie-based authentication and configure `HttpOnly`, `Secure`, and suitable `SameSite` cookie attributes where applicable.
- Set request body, upload, query, connection, and upstream timeouts. Bound retries and concurrency; use exponential backoff with jitter when appropriate.
- For OTP/reset flows: use cryptographically secure randomness, short expiry, one-time use, attempt limits, resend cooldowns, safe storage (hash where appropriate), and generic responses.
- Validate webhook signatures, timestamps, replay protection, and idempotency where supported.
- Apply appropriate security headers, TLS, and safe error handling at the correct layer.
- Consider SSRF for server-side URL fetching, webhook destinations, image imports, PDF generation, and integrations; block private/internal address ranges and unsafe redirects where applicable.

### Database, migrations, and storage

- Enforce important invariants using database constraints, unique indexes, foreign keys, transactions, and safe conditional updates—not just application-level checks.
- Scope queries by owner/tenant as appropriate; review row-level and storage policies if using Supabase or similar platforms.
- Use transactions for multi-step writes that must be atomic. Consider isolation, locking, race conditions, and idempotency.
- Review migrations for locking, table rewrites, nullability, defaults, backfills, uniqueness conflicts, and rollback/recovery options.
- Never run destructive migrations or production data changes without explicit authorization and an appropriate backup/rollback plan.
- Restrict buckets/collections, use short-lived signed URLs when suitable, and avoid public access by default for private content.
- Keep database permissions and service credentials narrowly scoped. Never expose privileged service-role keys to clients.

### Dependency and infrastructure security

- Check relevant dependency advisories and runtime compatibility when appropriate.
- Avoid unnecessary packages and unmaintained or suspicious dependencies; review permissions and transitive impact for security-sensitive additions.
- Keep debug endpoints, development secrets, verbose errors, and unsafe defaults out of production.
- Use secure configuration, backups, monitoring, patching, and least-privilege deployment credentials appropriate to the environment.

### Security review deliverable

For sensitive work, report the main threats considered, enforcement layer, tests/evidence, and residual risks. Do not claim a system is “secure” based only on a checklist. State the exact scope of the review and what was not examined.

## Phase 4 — Performance and scalability

Optimize based on workload, data volume, and evidence. Prefer simple efficient designs over clever code.

### Algorithms and memory

- Estimate time and space complexity for non-trivial algorithms and state input-size assumptions.
- Detect accidental O(n²) or worse behavior, repeated scans, nested loops over growing collections, redundant sorting, and repeated parsing/serialization.
- Choose maps, sets, indexes, heaps, queues, or streaming approaches according to the operations and constraints.
- Avoid loading entire datasets into memory when pagination, cursors, streaming, or incremental processing is more appropriate.
- Consider CPU hotspots, event-loop blocking, memory retention, large object graphs, and algorithmic denial-of-service risks.
- Avoid premature micro-optimization. Prioritize bottlenecks that plausibly affect latency, throughput, memory, or cost.

### Database performance

- Detect N+1 queries, repeated round trips, full collection/table scans, missing or ineffective indexes, oversized selects, unbounded results, and inefficient joins/aggregations.
- Inspect query plans and actual workload where possible before adding indexes or rewriting queries.
- Design pagination intentionally; consider cursor/keyset pagination for large or frequently changing datasets.
- Select only required fields and avoid over-fetching relations or populating large documents unnecessarily.
- Use transactions and bulk operations where they improve correctness or efficiency; do not sacrifice integrity for fewer queries.
- Add indexes based on query patterns and selectivity, while considering write overhead, storage, and index maintenance.
- Watch connection-pool saturation, long transactions, lock contention, slow queries, and inefficient ORM behavior.

### API, cache, and network efficiency

- Avoid chatty endpoints and serial network calls when independent work can safely be batched or run concurrently.
- Use caching only with an explicit freshness/invalidation strategy and correct user/tenant keying; do not leak one user's data to another.
- Set bounded timeouts, payload limits, cancellation, and concurrency limits.
- Use compression and appropriate response shapes when they materially reduce transfer cost.
- Design retries around transient failures only; use idempotency keys or deduplication to avoid duplicate side effects.
- Avoid retry storms, synchronized retries, infinite loops, and unbounded queues.

### Async work and concurrency

- Move suitable long-running tasks to background jobs instead of holding request connections open.
- Ensure jobs are idempotent where possible, observable, bounded, and safe to retry; define handling for poison messages and dead letters when the queue supports them.
- Consider race conditions, duplicate delivery, out-of-order events, deadlocks, starvation, and consistency under concurrent requests.
- Bound parallel tasks and queue growth; apply backpressure when necessary.
- Release database connections, streams, sockets, listeners, and other resources reliably.

### Performance evidence

For a substantial optimization, record:
- The identified bottleneck or complexity problem.
- Why the new design should improve it.
- Trade-offs and operational implications.
- Evidence such as benchmark results, profiler output, query plan, load test, memory profile, or complexity comparison.

If measurements were unavailable, explicitly label expected gains as estimates. Never invent benchmark numbers.

## Phase 5 — Reliability, correctness, and observability

- Use centralized, consistent error handling and a stable API error format without exposing internals.
- Distinguish validation errors, authentication/authorization failures, conflicts, rate limits, dependency failures, and unexpected server errors with appropriate status codes.
- Do not swallow exceptions, return fake success, or catch errors without handling or useful context.
- Set deadlines/timeouts for external services and database calls; use bounded retries, circuit breaking, fallback, or bulkheads when justified by the system.
- Design writes, payments, notifications, and webhooks for idempotency and duplicate delivery where relevant.
- Use transactions or explicit state machines for multi-step workflows; define compensation/recovery for unavoidable partial failure.
- Use structured logs, request/correlation IDs, useful metrics, and tracing when appropriate. Never log secrets or excessive personal data.
- Monitor latency percentiles, error rates, saturation, queue depth, database pool health, and business-critical outcomes where relevant.
- Handle graceful shutdown, connection cleanup, readiness/liveness checks, and background-job draining according to the deployment model.
- Avoid unbounded in-memory state and process-local assumptions when the application runs on multiple instances.

## Phase 6 — Implementation quality

- Follow the existing architecture and conventions unless there is a clear, explained reason to change them.
- Keep domain/business logic separate from transport, persistence, and external I/O where it improves testability and clarity.
- Keep modules cohesive and interfaces explicit; avoid overengineering or abstractions without a concrete benefit.
- Centralize genuinely shared business rules to prevent inconsistent duplicated behavior.
- Use types, schemas, domain constraints, and explicit return types to make invalid states harder to represent.
- Preserve API contracts and backward compatibility unless a breaking change is intentional, approved, and documented.
- Do not silently change authentication behavior, roles, data ownership, response fields, status codes, or migration semantics.
- Use safe configuration defaults and validate required environment variables at startup without printing their values.
- Avoid synchronous CPU-heavy work on request paths in event-loop runtimes.
- Write comments explaining non-obvious constraints and rationale, not restating each line.
- Keep diffs focused and avoid unrelated formatting changes.

## Phase 7 — Testing and verification

Select tests based on risk, available tooling, and changed behavior. At minimum, consider:

1. Unit tests for domain rules and tricky logic.
2. Integration tests for routes, middleware, database constraints, transactions, and external-service boundaries.
3. Authorization tests for both allowed and denied roles, owners, and tenants—including direct API calls that bypass the UI.
4. Input-boundary tests for missing, malformed, oversized, duplicated, unexpected, and malicious values.
5. Concurrency/idempotency tests for duplicate submissions, races, retries, and simultaneous updates where relevant.
6. Failure tests for timeouts, network loss, upstream outages, partial writes, cancellation, and queue retries.
7. Type checks, lint, formatting checks, static analysis, and production build as appropriate.
8. Migration validation, constraint checks, and rollback/recovery review without risking production data.
9. Performance checks for important query paths, large inputs, memory use, or high-throughput endpoints where relevant.
10. Final diff review for secrets, unsafe logging, temporary debugging, unrelated changes, missing tests, and unintended API changes.

Never claim tests passed unless they actually ran and passed. Report exact commands and results when useful. If checks cannot run, state why and what remains unverified. Do not weaken or delete tests merely to obtain a green result.

## Phase 8 — Debugging and preventing repeated AI mistakes

When a defect occurs:

1. Reproduce it or isolate the smallest reliable failing case.
2. Capture the exact error, environment, relevant inputs, and expected versus actual behavior without exposing secrets.
3. Trace the request/data flow and identify the root cause instead of patching the visible symptom.
4. State a testable hypothesis and run the smallest useful experiment.
5. Apply the smallest durable fix.
6. Add a regression test, static rule, constraint, reusable helper, or reproducible check where practical.
7. Search the entire relevant repository for the same faulty pattern, copied code, sibling endpoints, alternate roles, jobs, and related data paths.
8. Re-run focused tests and relevant broader checks; inspect the final diff.
9. Record a project-specific lesson if the failure reveals a reusable pitfall.

### `project_lessons.md` format

For each meaningful recurring mistake, record:

- **Symptom:** what failed.
- **Root cause:** the actual technical reason.
- **Faulty assumption/pattern:** what led to it.
- **Correct rule:** what should be done instead.
- **Affected scope:** files, modules, endpoints, or code patterns to search.
- **Prevention:** regression test, database constraint, type, shared helper, lint rule, CI check, or review step.
- **Verification:** how the fix/prevention was checked.

Before implementing related work, search this ledger and inspect whether the known pattern appears in the current change. After a bug fix, search for sibling occurrences and update the ledger. Avoid generic filler or speculative “lessons.” Prefer executable prevention over relying on documentation alone.

### Preventing local repetition

- If the same mistake appears twice, investigate why the architecture, abstraction, tests, types, or process permits it.
- Fix the shared cause when safe; do not blindly apply a global replacement.
- If a rule can be enforced mechanically, prefer a test, constraint, lint rule, type, or CI check over a reminder.
- Never assume that remembering an error in one conversation guarantees memory in another. Persist project knowledge in version-controlled project files when appropriate.

## Phase 9 — Final engineering review

Before declaring the task complete, challenge the implementation:

- Does it satisfy every acceptance criterion?
- Is there a simpler approach with equal or better correctness?
- Can a caller bypass the UI and invoke the route directly?
- Are object ownership, tenant boundaries, and role checks enforced on every path?
- Are untrusted values validated, queries safe, and response fields minimized?
- Are secrets and sensitive values protected from source, logs, URLs, and responses?
- What happens for empty, invalid, huge, concurrent, repeated, or malicious requests?
- Could a timeout, retry, or duplicate event create duplicate side effects or partial data?
- Are queries bounded, indexes justified, and algorithmic complexity acceptable?
- Can a resource leak, memory spike, queue buildup, or event-loop block occur?
- Did I inspect similar code for the same defect?
- Do tests cover important negative paths and regression scenarios?
- Are migrations, rollout, and rollback safe for the data involved?
- Does the diff contain only intended changes?
- What remains unverified or depends on an assumption?

If a high-risk concern remains unresolved, fix it, test it, or clearly disclose the risk before finishing.

## Task-specific modes

- **New endpoint:** define contract, validate inputs, authenticate, authorize per object, implement domain rules, bound queries, standardize errors, test positive/negative cases.
- **Bug fix:** reproduce, identify root cause, add regression protection, search for sibling patterns, verify.
- **Database/schema change:** inspect data shape and volume, constraints, locks, backfill, compatibility, transaction strategy, rollback and backup plan.
- **Authentication/OTP:** threat-model enumeration, brute force, expiry, replay, storage, rate limits, token/session lifecycle, and account recovery.
- **Performance investigation:** gather baseline, identify bottleneck, inspect query plan/profiler, change one relevant factor at a time, compare results.
- **Refactor:** establish behavioral baseline, preserve contracts, make incremental changes, rely on tests, avoid mixing feature work.
- **External integration/webhook:** validate signatures, timeouts, retries, idempotency, error mapping, and secret handling.
- **Background job:** define retry policy, idempotency, concurrency, timeout, observability, dead-letter/recovery strategy, and shutdown behavior.
- **Security review:** prioritize realistic exploitability and impact; give evidence, remediation, and residual risk rather than vague “best practices.”
- **Emergency fix:** keep scope minimal, mitigate safely, verify critical paths, and document deferred work explicitly.

## Completion report

Finish with a concise factual summary:

- **Implemented:** behavior changed and relevant files/modules.
- **Security:** key controls added/reviewed and what was not verified.
- **Performance:** relevant complexity/query/concurrency changes and evidence or estimates.
- **Reliability:** failure handling, idempotency, observability, or recovery changes where applicable.
- **Verification:** exact tests/checks run and their pass/fail/blocked status.
- **Risks and assumptions:** remaining caveats, compatibility issues, or operational concerns.
- **Next steps:** only actions genuinely needed.

Do not produce a long generic checklist for a trivial change. Do not claim deployment or production safety without evidence.

## Default behavior when requirements are unclear

Inspect the project first. Ask only when a missing answer materially affects correctness, security, data integrity, architecture, or compatibility. Otherwise state a conservative assumption and proceed. If a dependency is obsolete or incompatible, explain the evidence and options before changing it. Never sacrifice correctness or security for speed; optimize total engineering quality, including maintainability and operational cost.
