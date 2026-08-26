# Odoo Performance analysis

## Purpose

Use this guidance to analyze Odoo performance risk in ORM code, computed fields, CRUD overrides, cron jobs, imports, reports, controllers, RPC, integrations, frontend behavior, caching, memory usage, locking, concurrency, and upgrade-time workloads.

This is specialized **Odoo performance evidence guidance**.

It does not replace Plemo's native repository discovery, planning, implementation, debugging, deployment, or general validation workflow.

Its core question is:

```text
Could this Odoo implementation become slow, expensive, or unstable
at realistic data volume, concurrency, or runtime conditions?
```

Core principles:

- Separate performance suspicion from measured evidence.
- Follow the real execution path, not isolated code snippets only.
- Reuse prior Plemo investigation, impact, migration, security, and regression evidence.
- Do not optimize blindly.
- Do not trade correctness or security for speed.
- Focus first on ORM/query count, recompute fan-out, batching, algorithmic cost, locking, memory, RPC/network cost, and data scale.
- Do not claim a regression or improvement without measurement when measurement is required.
- Preserve Plemo's native task mode and authorization behavior.

## 0. Native Agent Compatibility and Scope

This guidance extends Plemo; it does not override it.

Repository-specific instructions such as `plemo.md`, customer constraints, project conventions, deployment limits, database/runtime rules, and available Plemo tools take precedence over generic examples in this guidance.

Reuse reliable evidence from:

- existing codebase-investigation evidence
- existing feature-impact evidence
- upgrade and migration analysis
- security and access analysis
- broader regression and runtime validation
- current Git diff
- runtime logs
- query/profiling output
- database statistics
- browser/network evidence

Do not repeat a full investigation when existing evidence is sufficient.

### 0.1 Relationship With Native Workflow and Existing Evidence

```text
Existing codebase investigation
    "What exists and how does it work?"
        ↓
feature-impact analysis
    "What could this change affect?"
        ↓
Existing upgrade/migration analysis
    "What happens to installed databases and existing data?"
        ↓
Relevant security/access analysis
    "Is the access model safe?" when relevant
        ↓
Performance analysis
    "Could this become slow or unstable at realistic scale?"
        ↓
Plemo native planning
        ↓
Plemo native implementation
        ↓
Broader regression/runtime validation
    "What can we prove works after implementation?"
```

This guidance provides performance evidence and required measurements to Plemo's native planner. It does not create a second generic optimization or implementation plan.

### 0.2 Task Mode and Authorization

Determine whether the current task is:

- performance review only
- diagnose slowness
- fix performance issue
- implement performance-sensitive feature
- validate performance after a fix

For review/diagnosis only:

- remain read-only;
- do not silently refactor code;
- do not add/drop indexes;
- do not alter worker/database configuration;
- do not disable business or security controls to make measurements look better.

For an already-authorized fix/implementation:

- the original request is sufficient authorization for the requested scope;
- do not ask for a second approval merely because performance analysis finished;
- continue into Plemo's native planning process using the performance evidence;
- request new resolution only when optimization requires a material behavior change, schema/index decision, migration, deployment configuration change, external architecture change, or another scope expansion not already authorized.

### 0.3 Performance Materiality Gate

Usually light review only:

- comments/documentation;
- translations;
- labels;
- CSS-only changes;
- tiny UI changes with no data/RPC effect.

Targeted analysis:

- one model method;
- one compute;
- one controller;
- one report;
- one cron;
- one new RPC call;
- one domain/search change.

Full analysis strongly preferred for:

- `search()`/`write()` in loops;
- large recordsets;
- stored compute changes;
- broad `@api.depends` chains;
- shared methods with many callers;
- high-volume accounting/stock/payroll flows;
- imports/exports;
- large reports;
- cron over many records;
- website/catalog endpoints;
- many RPC calls;
- large frontend lists;
- direct SQL;
- large migrations/backfills;
- high concurrency;
- known production slowness;
- unknown but potentially large data volume.

Performance depth follows realistic cost, not code length.

## 1. Determine the Performance Target

Record:

```text
Project:
Feature/workflow:
Requested change:
Observed problem:
Target module(s):
Primary model(s):
Primary method(s):
Primary route(s):
Primary frontend component(s):
Expected data volume:
Expected concurrency:
Known production symptoms:
```

Preserve explicit user symptoms and budgets exactly.

## 2. Detect Odoo Version and Runtime Context

Record:

```text
Odoo version:
Version evidence:
Edition if known:
Environment:
Database context:
Worker/runtime configuration if known:
```

A valid module manifest version prefix is acceptable version evidence when core source is unavailable.

Do not make version-sensitive ORM or frontend assumptions without evidence.

## 3. Repository and Tool Precedence

Use:

1. `plemo.md` and project instructions;
2. Plemo-native repository/Odoo tools;
3. project-specific profiler/test helpers;
4. standard Odoo/Python/PostgreSQL/browser profiling mechanisms;
5. generic shell tools where appropriate.

Potential available capabilities may include `find_addon`, `search_code`, `read_file`, `odoo_local`, `read_odoo_log`, `get_build_log`, `upgrade_module`, or equivalents.

Never invent unavailable tools.

Do not dispatch merely because profiling is unavailable locally. Dispatch only when the project workflow permits it and the performance requirement materially justifies it.

## 4. Reuse Existing Evidence

Reuse, when available:

- feature owner;
- reverse dependencies;
- runtime callers;
- stored-compute risk;
- migration volume;
- security context;
- integration callers;
- regression scenarios;
- production-like record counts.

Refresh only what is stale or insufficient for performance analysis.

## 5. Inspect the Actual Change Set

Inspect the actual diff and record:

```text
Potential new queries:
Potential removed queries:
New loops:
Changed compute logic:
Changed domains:
Changed write patterns:
New RPC calls:
Changed reports:
Changed cron/batch logic:
Index/schema changes:
```

Analyze what actually changed, not only what the implementation plan expected.

## 6. Build the Real Execution Path

Trace:

```text
entry point
→ controller / button / cron / import / RPC
→ model method(s)
→ search/read/create/write/unlink
→ compute/recompute
→ automation/chatter/integration
→ report/frontend response
```

A fast helper can still be inside a slow outer loop.

## 7. Search-in-Loop Analysis

Search for repeated ORM calls inside loops, especially:

```text
for rec in records:
    env['x'].search(...)
```

Also inspect repeated `search_count`, `browse`, relation lookup, or identical searches.

Record:

```text
Outer records:
Queries per iteration:
Estimated query growth:
Batch alternative possible:
```

Prefer batching only when semantics remain correct.

## 8. Write Amplification

Inspect repeated:

```text
write
create
unlink
```

inside loops or nested workflows.

Before recommending batching, account for:

- constraints;
- computed fields;
- tracking/chatter;
- automated actions;
- mail;
- business side effects;
- record-specific values.

One batch write is not always semantically equivalent to many isolated writes.

## 9. Create Batching

For high-volume creates inspect whether multi-create semantics can be used safely in the detected Odoo version.

Check per-record sequence/default/side-effect requirements before recommending batching.

## 10. Prefetch Awareness

Odoo prefetch can make recordset access efficient.

Potential anti-patterns include:

- repeatedly browsing isolated IDs;
- rebuilding recordsets unnecessarily;
- repeated relational searches instead of recordset traversal;
- forcing singleton logic on batch-capable methods.

Do not assume every field access triggers a query.

## 11. Python Filtering vs SQL Domains

Compare loading a huge recordset then filtering in Python with filtering through a domain when semantics permit.

Do not replace readable recordset operations blindly; use actual scale and query evidence.

## 12. Search Domain Quality

Inspect heavy domains for:

- unindexed fields;
- broad `OR` chains;
- `ilike` over large tables;
- deep relational paths;
- many2many joins;
- non-stored computed fields;
- company/website filters;
- broad active/inactive scans.

Record whether a large full scan is plausible.

## 13. Duplicate Queries

Look for repeated identical searches, duplicate `search_count + search`, repeated reads, or recalculation of the same data within one request/workflow.

Reuse results safely when semantics and cache lifetime allow.

## 14. Aggregation Strategy

For sums/counts/grouping over large sets, assess whether Odoo grouping APIs are more appropriate than loading all records and aggregating in Python.

Verify version-specific `_read_group` / `read_group` behavior before recommending a change.

## 15. UI Hot-Path Methods

Review frequently invoked display/search methods when they become slow, including version-relevant display-name/search hooks.

Avoid expensive unrelated searches in methods called for many rows or autocomplete requests.

## 16. Record Rule Query Cost

When a query is slow only for restricted users, inspect effective record-rule domains.

Potential risks:

- many joins;
- deep relational conditions;
- large `OR` expressions;
- company/user-specific domains.

Never remove security restrictions for speed. Apply the relevant security and access checks before accepting any performance-related change to security behavior.

## 17. Direct SQL

When relevant, review:

```text
query shape
parameters
indexes
row count
locking
ORM flush/cache invalidation
security implications
```

Direct SQL may be faster but bypasses ORM behavior and security. Require strong evidence before recommending it.

## 18. Index Analysis

Consider an index only when repeated selective query patterns justify it.

Record:

```text
Query/domain:
Existing index:
Selectivity:
Expected benefit:
Write overhead:
Migration/build cost:
```

Do not add indexes blindly.

## 19. Composite Index Analysis

For multi-column filters, evaluate whether a composite index matches actual predicate patterns and column selectivity.

Do not create oversized indexes from every field in a domain.

## 20. Query Plan Evidence

When safe and materially useful, inspect PostgreSQL query plans using project-approved mechanisms.

Record:

```text
Scan type:
Estimated/actual rows:
Join type:
Sort/hash operations:
Timing if measured:
```

Do not run expensive `EXPLAIN ANALYZE` blindly on production.

## 21. Computed Field Cost

For compute methods inspect:

- record count;
- dependencies;
- nested loops;
- searches per record;
- writes/side effects;
- relational traversal;
- aggregation strategy.

A compute that is cheap for one record can be expensive for thousands.

## 22. Stored Compute Fan-Out

Map:

```text
source field change
→ invalidated records
→ compute batch
→ dependent stored fields
→ downstream recompute
```

Estimate fan-out and reuse existing upgrade/migration evidence for existing-data recompute risk.

## 23. Dependency Breadth

Review `@api.depends` and `@api.depends_context` for unnecessarily broad invalidation while preserving correctness.

Do not remove required dependencies merely for speed.

## 24. Compute Side Effects

Flag compute methods that perform expensive searches, business writes, messages, or external calls.

These costs can multiply during recomputation.

## 25. Non-Stored Compute Hot Paths

Determine how often a non-stored field is recalculated in:

- list views;
- forms;
- reports;
- exports;
- RPC;
- website rendering.

Do not automatically make it stored; storing introduces invalidation and migration cost.

## 26. CRUD Override Performance

For `create`, `write`, and `unlink` overrides inspect:

- super-call strategy;
- per-record vs batch behavior;
- repeated searches;
- recomputes;
- constraints;
- tracking;
- automation.

A tiny override can affect every write to a major model.

## 27. Constraint Performance

Inspect searches and cross-model checks inside constraints, especially on batch writes.

Do not remove correctness constraints for performance.

## 28. Onchange Performance

For slow forms inspect:

- server round trips;
- heavy searches;
- broad value loads;
- chained onchange behavior;
- unnecessary trigger fields.

Measure realistic form state when material.

## 29. Chatter and Tracking Cost

For heavily updated `mail.thread` models inspect tracking, message creation, followers, and activities.

Do not disable audit/history behavior without business authorization.

## 30. Automated Action Multiplication

Trace whether writes trigger:

- `base.automation`;
- server actions;
- mail;
- webhooks;
- queues;
- scheduled follow-up.

Performance can be dominated by indirect side effects.

## 31. Cron Performance

Record:

```text
Frequency:
Records scanned:
Records processed:
Batch size:
Search domain:
Overlap possible:
Lock risk:
Retry behavior:
```

Check whether the next run can begin while the previous one is still active.

## 32. Batch and Resume Design

For long-running jobs consider bounded batches, idempotency, checkpointing/resume, failure recovery, and duplicate processing.

Do not introduce manual commits without understanding transaction semantics.

## 33. Import Performance

For large imports inspect:

- per-row searches;
- create/write batching;
- constraints;
- tracking;
- computed fields;
- external validation.

Use realistic row counts.

## 34. Report and Export Performance

Large reports/exports may stress:

- ORM reads;
- related/computed fields;
- aggregation;
- images/binaries;
- QWeb rendering;
- PDF/XLSX generation;
- memory.

Profile the actual report path when material.

## 35. Controller and API Query Cost

For routes inspect:

```text
searches
records fetched
fields fetched
pagination
serialization
sudo impact
```

Public/catalog routes can become high-frequency hotspots.

## 36. Pagination

Flag unbounded endpoints such as large `search([])` or oversized JSON/table responses.

Use bounded/paginated results when product requirements permit.

## 37. Serialization Cost

Inspect unnecessary fields, binaries, deeply nested payloads, and duplicate related data.

Do not remove fields that consumers actually require.

## 38. RPC Chatter

Count per interaction:

```text
RPC count
sequential dependencies
duplicate calls
polling frequency
payload size
```

Reduce unnecessary chatter when semantics allow.

## 39. External Integration Latency

Inspect synchronous external calls for timeout, retry, batching, provider latency, and idempotency.

Do not keep user-facing transactions blocked on slow external systems unless required by business semantics.

## 40. N+1 External Calls

The same N+1 problem can happen with external APIs.

Assess provider batching support and business correctness before recommending aggregation.

## 41. Frontend Performance Scope

For JS/OWL inspect:

- RPC count;
- rerender frequency;
- list size;
- DOM scans;
- event handlers;
- asset weight;
- polling;
- synchronous loops.

Static suspicion is not measured frontend performance.

## 42. OWL Rerender Cost

Review unstable keys, broad state updates, rebuilding large arrays, repeated service calls, expensive getters, and global patches affecting many screens.

Prefer runtime browser evidence when material.

## 43. DOM/Event Cost

Inspect repeated `querySelectorAll`, per-row listeners, mutation observers, scroll/resize handlers, and large DOM traversal.

Use bounded operations/event delegation where appropriate.

## 44. Asset Weight

When assets change, compare JS/CSS/media size and whether code is loaded globally or only where needed.

Do not make asset analysis mandatory for backend-only changes.

## 45. Browser Network Evidence

When browser tools are available, inspect request count, slow RPCs, duplicate requests, payload sizes, and blocking assets.

Do not claim frontend performance without browser/runtime evidence when runtime proof is required.

## 46. Cache Suitability

Consider caching only when data is expensive, read frequently, changes less often, and has clear invalidation.

Do not add caching merely to hide an inefficient query.

## 47. Odoo Cache Awareness

Review ORM/framework cache behavior before adding custom caching.

Custom caches can create stale data or cross-user/company leakage.

Security-sensitive cache design must include the relevant security and access checks.

## 48. Cache Keys and Invalidation

When framework caching is used, verify key dimensions such as user, company, language, context, and arguments when they affect correctness.

Verify invalidation strategy.

## 49. Large Recordset Memory

Flag workflows that load very large recordsets/lists into Python memory unnecessarily.

Prefer bounded processing when semantics permit.

## 50. Binary and Attachment Memory

Avoid loading large binaries/base64 values unnecessarily during reports, integrations, or list workflows.

Use appropriate attachment/streaming mechanisms where supported.

## 51. Report Memory Pressure

Large QWeb/PDF/XLSX reports can exhaust workers even if query count is acceptable.

Measure representative record and media volume.

## 52. Transaction Duration

Long transactions can block other workers.

Identify workflows that:

- update many rows;
- wait on external APIs before commit;
- run large recomputes;
- hold locks during long processing.

Shorten critical sections only when correctness permits.

## 53. Contention Hotspots

Look for shared records frequently written under concurrency, such as sequences, counters, configuration, stock/accounting aggregates, or singleton state.

Single-user tests do not prove concurrency safety.

## 54. Explicit Locking

If code uses row/advisory locks, review lock scope, ordering, contention, timeout behavior, and failure semantics.

Do not introduce locking solely from speculation.

## 55. Duplicate Work Under Concurrency

Check whether multiple workers can process the same record/job, causing duplicate external calls, emails, recompute, or wasted CPU.

Coordinate with correctness/integration specialists when idempotency is involved.

## 56. Record Volume Profiling

When safely available, record representative counts for main and related models, archived records, company distribution, and historical growth.

Classify scale:

```text
small
moderate
large
very large
unknown
```

Do not claim scale safety while scale is unknown when scale is material.

## 57. Growth Risk

Describe cost qualitatively as constant, roughly linear, super-linear, or unknown where useful.

Avoid unsupported exact complexity claims for database-heavy behavior.

## 58. Measurement Before Optimization

When a real slowdown exists, establish a baseline when practical.

Record:

```text
Scenario:
Dataset size:
Environment:
User/company context:
Duration:
Query count:
RPC count:
Memory/CPU if available:
```

Without a comparable baseline, avoid quantified improvement claims.

## 59. Representative Benchmark Scenario

Benchmark the real workflow rather than an unrelated micro-function.

Examples:

```text
post 500 invoices
run cron over 50,000 records
render report with 1,000 lines
open form with 300 lines
load portal list over a large dataset
```

## 60. Warm vs Cold Effects

Be aware of ORM cache, PostgreSQL buffers, browser/asset cache, compiled templates, and connection warmup.

Record whether before/after runs are comparable.

## 61. Repeatability

Use repeated measurements when noise is material.

Prefer ranges/medians over one lucky run when practical.

Do not fabricate statistical precision.

## 62. Before/After Comparison

Compare the same scenario, dataset, environment, user/company context, and measurement method.

If conditions differ, state that the comparison is approximate.

## 63. Query Count Evidence

Record query count before/after when it is a useful metric.

A locally faster run with dramatically more queries may still scale poorly.

## 64. Runtime Duration Evidence

Use wall-clock duration only when the environment is stable enough for the conclusion.

Do not project local laptop timings directly to production.

## 65. CPU and Memory Evidence

When available, distinguish DB-bound, I/O-bound, CPU-bound, memory-bound, and external-latency bottlenecks.

Do not recommend more workers when the actual problem is N+1 queries.

## 66. Performance Budgets

If the user/project defines explicit budgets, preserve them exactly.

Do not invent arbitrary latency or throughput targets.

## 67. Migration-Time Performance

Reuse existing upgrade/migration evidence for large updates, recomputes, indexes, constraints, and downtime risk.

Performance analysis deepens runtime-cost evidence; it does not duplicate migration semantics.

## 68. Stored Recompute Benchmark

For material stored compute changes record records recomputed, duration, failures, locks/timeouts, and dependent recomputes where measurable.

If only a small test DB exists, state that production-scale performance remains unverified.

## 69. Never Trade Security for Speed

Do not optimize by:

- removing record rules;
- broadening ACLs;
- using unjustified `sudo()`;
- creating unsafe cross-user/company caches;
- exposing broad public endpoints.

Security trade-offs must include the relevant security and access checks.

## 70. Security Context Performance

If performance differs by user, compare effective record-rule domains and company context before blaming the business method alone.

## 71. Performance Finding Severity

Use:

```text
Critical
High
Medium
Low
Informational
```

Consider user latency, worker starvation, DB load, batch failure, migration timeout, locking, frequency, business criticality, and production scale.

## 72. Evidence Confidence

Use:

```text
Measured
Strong static evidence
Probable
Possible
Needs runtime measurement
```

Keep severity separate from confidence.

## 73. Finding Structure

For each material finding record:

```text
Finding ID:
Title:
Severity:
Confidence:
Execution path:
Data volume:
Current pattern:
Performance risk:
Evidence:
Likely bottleneck:
Affected workflows:
Recommended optimization boundary:
Measurement required:
Regression risk:
```

## 74. Bottleneck Classification

Use when helpful:

```text
ORM/query count
SQL/query plan
compute/recompute
Python algorithm
write amplification
automation side effects
network/RPC
external integration
frontend render
asset weight
memory
locking/concurrency
migration-time
unknown
```

## 75. Performance Boundary Recommendation

State:

```text
Module:
Method/component:
Primary bottleneck:
Recommended strategy:
Do-not-touch areas:
Required before/after measurement:
```

Potential strategies include batching, grouped aggregation, pagination, targeted indexing, narrower compute dependencies, bounded cron batches, fewer RPCs, or asynchronous integration where justified.

Do not prescribe an optimization unsupported by evidence.

## 76. Do-Not-Touch Boundary

For material performance work record:

```text
Odoo core:
Third-party/OCA:
Shared custom modules:
Security rules:
Business correctness:
Historical data:
Sibling customer projects:
Deployment configuration:
Other protected areas:
```

## 77. Required Performance Evidence Output

For material analysis output:

```text
PERFORMANCE EVIDENCE

Project:
Feature:
Odoo version:
Environment:
Primary execution path:
Expected data volume:
Expected concurrency:

ORM/query risk:
Compute/recompute risk:
Write amplification:
Cron/batch risk:
Controller/RPC risk:
Frontend risk:
Integration latency risk:
Memory risk:
Locking/concurrency risk:
Migration-time risk:

Measurements available:
Measured baseline:
Measured result:
Improvement/regression proven:

Primary bottleneck:
Recommended optimization boundary:
Do-not-touch boundary:
Runtime measurement required:
Remaining unknowns:
Overall performance risk:
```

## 78. Performance Assessment Status

Use:

```text
ACCEPTABLE FOR REVIEWED SCALE
REQUIRES OPTIMIZATION
PARTIAL / NEEDS MEASUREMENT
BLOCKED
```

`ACCEPTABLE FOR REVIEWED SCALE` means no material issue was identified for the tested/reviewed scenario and scale. It does not prove larger untested scale.

`REQUIRES OPTIMIZATION` means a material bottleneck or highly supported performance problem exists.

`PARTIAL / NEEDS MEASUREMENT` means useful static evidence exists but realistic runtime measurement is still required.

`BLOCKED` means the required environment, data scale, runtime path, logs, or profiler evidence is unavailable.

## 79. Detailed Report Structure

For complex analysis use:

1. Performance Target
2. Odoo Version / Environment
3. Actual Change Set
4. Execution Path
5. Data Volume / Concurrency
6. ORM / Query Analysis
7. Compute / Recompute
8. CRUD / Automation
9. Cron / Batch
10. Controllers / RPC / Integrations
11. Frontend / Assets
12. Caching
13. Memory
14. Locking / Concurrency
15. Migration-Time Performance
16. Measurements / Baseline
17. Findings
18. Recommended Optimization Boundary
19. Required Runtime Measurements
20. Remaining Unknowns
21. Final Performance Assessment

## 80. Concise Output for Small Reviews

For contained work use:

```text
Performance Assessment:
ACCEPTABLE / REQUIRES OPTIMIZATION / PARTIAL / BLOCKED

Execution path:
Data scale:
Observed risk:
Evidence:
Recommended boundary:
Measurement required:
```

Do not force a full report for a small isolated method.

## 81. Integration With Native Planning

For an authorized fix/implementation, output concise `PERFORMANCE EVIDENCE`, then return control to Plemo's native planner.

Do not create a duplicate optimization plan and do not require duplicate approval.

## 82. Post-Implementation Validation Requirements

After implementation provide exact scenarios such as:

```text
same workload before/after
same dataset
same user/company context
query count comparison
cron duration
report duration
RPC count
browser/network comparison
locking/concurrency scenario
```

The broader post-change validation phase owns the final runtime validation status.

## 83. Performance Fix Recheck

After Plemo implements a fix:

1. inspect the final diff;
2. confirm optimization stayed inside the intended boundary;
3. rerun the original slow scenario;
4. collect comparable measurements;
5. rerun affected regression scenarios;
6. update performance evidence.

Do not claim improvement from code appearance alone.

## 84. Pre-Existing Performance Problems

Separate:

```text
introduced by current change
pre-existing
exposed by current change
environment-specific
unknown
```

Do not silently expand scope to unrelated performance debt.

## 85. Production Safety

Prefer safe test/staging/cloned environments for intrusive profiling.

On production:

- prefer read-only observation;
- avoid heavy diagnostic queries;
- avoid dangerous `EXPLAIN ANALYZE`;
- do not enable expensive profiling/logging broadly without authorization;
- do not create/drop indexes casually;
- do not change worker/database settings merely to test a theory.

## 86. Hard Prohibitions

Never:

- replace Plemo's native planner or implementation workflow;
- require duplicate approval for an already-authorized fix;
- silently optimize during review-only mode;
- claim improvement without evidence;
- claim production-scale safety from tiny test data;
- optimize only from microbenchmarks when the real path differs;
- assume every field access causes a query;
- assume `mapped()` or `filtered()` is inherently slow;
- use `sudo()` as a performance optimization;
- remove security restrictions for speed;
- use direct SQL only because ORM code looks slower;
- add indexes blindly;
- add caches without invalidation analysis;
- create cross-user/company cache leakage;
- disable tracking/automation without business authorization;
- batch record-specific writes without checking semantics;
- introduce manual commits without transaction analysis;
- ignore concurrency/locking on hot workflows;
- ignore stored recompute fan-out;
- ignore migration-time performance when upgrade and migration analysis identifies it;
- claim frontend performance without browser evidence when runtime proof is required;
- invent performance budgets or downtime;
- run dangerous production profiling without authorization;
- modify core/third-party code against project rules;
- expand into sibling customer projects without evidence;
- commit or push unless explicitly requested.

## 87. Core Decision Tree

```text
START
  ↓
Read plemo.md / project rules
  ↓
Determine task mode
  ↓
Reuse investigation / impact / migration evidence
  ↓
Identify the actual execution path
  ↓
Estimate data volume / concurrency
  ↓
Material performance risk?
  ├─ No → light review
  └─ Yes
       ↓
Inspect ORM/query behavior
       ↓
Inspect compute/recompute
       ↓
Inspect writes/automation/cron
       ↓
Inspect controller/RPC/integration
       ↓
Inspect frontend/assets if relevant
       ↓
Inspect memory/locking/concurrency
       ↓
Migration-time risk?
       ├─ Yes → reuse upgrade and migration analysis evidence
       ↓
Runtime measurement available?
       ├─ Yes → establish baseline / measure
       └─ No → mark NEEDS MEASUREMENT
       ↓
Classify bottleneck and confidence
       ↓
Produce PERFORMANCE EVIDENCE
       ↓
Analysis-only?
       ├─ Yes → stop
       └─ No → continue into Plemo native planning using the evidence
       ↓
After fix → carry before/after scenarios into broader post-change runtime validation
```

## 88. Primary Rules to Always Remember

```text
Performance analysis must follow the real execution path.

Reuse existing Plemo evidence.
Do not duplicate investigation.

Preserve Plemo native planning, implementation, and deployment.
Do not require duplicate approval.

Use plemo.md and project rules first.

Separate:
static performance risk
measured performance evidence

Do not call an inference a measurement.

For ORM:
watch query count, search-in-loop, duplicate searches,
write amplification, and broad domains.

For computed fields:
analyze dependency breadth, fan-out, and recompute cost.

For CRUD overrides:
consider every caller, not only the visible form.

For cron/import/report:
test realistic record volume.

For controllers/RPC:
measure request count, payload, pagination, and serialization.

For frontend:
measure RPC chatter, rerenders, DOM cost, and assets when relevant.

For caching:
prove invalidation and user/company safety.

For memory:
avoid loading huge recordsets or binaries unnecessarily.

For concurrency:
consider locks, long transactions, and duplicate work.

For indexes:
use actual query evidence and consider write cost.

For migration-time performance:
reuse existing upgrade/migration evidence.

Never trade security or correctness for speed.

Output performance evidence, not a second optimization plan.

After implementation:
perform broader post-change regression and runtime validation.
```