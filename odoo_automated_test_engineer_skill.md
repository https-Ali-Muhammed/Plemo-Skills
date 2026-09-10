# Odoo Automated Test Engineer

## Purpose

Use this guidance to design, implement, review, and maintain automated tests for Odoo changes.

The goal is not to create tests merely to increase test count.

The goal is to build durable executable coverage that proves important Odoo behavior, protects meaningful regression surfaces, and remains understandable and maintainable as the module evolves.

This guidance adds **Odoo-specific automated-test engineering procedures and evidence**.

It does not replace Plemo's native repository discovery, planning, implementation, debugging, final review, or the existing regression/runtime-validation workflow.

Its core question is:

```text
Which behaviors are important enough to protect permanently,
and what is the smallest reliable Odoo automated test suite that proves them?
```

Core principles:

- Reuse reliable investigation, impact, security, migration, performance, code-quality, and validation evidence already collected for the task.
- Test behavior and contracts, not implementation trivia.
- Prefer the smallest test layer that can prove the requirement reliably.
- Add higher-level tests only when lower-level tests cannot prove the important behavior.
- Preserve repository-specific test conventions before applying generic examples.
- Detect the Odoo version before choosing version-sensitive test APIs.
- Do not assume a test class, helper, browser framework, frontend framework, or tour API exists in every supported Odoo version.
- Keep tests deterministic.
- Keep tests isolated from external live systems.
- Make user, company, website, language, timezone, and context assumptions explicit when they affect behavior.
- Test both expected success and meaningful failure paths.
- Protect security boundaries with actor-specific tests when security is material.
- Protect existing-database behavior with upgrade/migration tests when persistent data is material.
- Protect frontend behavior with the version-appropriate frontend/browser test mechanism when browser behavior is material.
- Do not use broad end-to-end tests when a focused model/controller test proves the same contract more reliably.
- Do not over-mock Odoo ORM behavior that should be exercised for real.
- Do not make tests depend on execution order.
- Do not silently weaken production code to make tests easier.
- Do not create brittle tests tied to incidental implementation details.
- Do not treat manually executed validation as permanent automated coverage.
- Do not claim a scenario is covered unless an executable test actually proves it.
- Do not announce or expose internal capability selection during normal conversation.

---

# 0. Native Plemo Compatibility

This guidance extends Plemo's existing agent capabilities. It does not replace them.

Plemo already performs native repository discovery, task-mode understanding, planning, implementation, ordinary debugging, targeted checks, and final diff review.

The existing regression/runtime-validation capability already answers:

```text
The implementation is complete.
What can now be proven to work in the available environments?
```

This guidance answers a different question:

```text
What automated tests should exist in the repository
so important behavior remains protected after this task is finished?
```

Do not create a second generic planning system.

Do not recreate a full regression analysis when reliable impact/regression evidence already exists.

Do not require a second approval when the user's original request already authorizes implementation.

Repository-specific instructions such as:

```text
plemo.md
configured addon paths
customer restrictions
repository test conventions
CI conventions
deployment conventions
available Plemo tools
```

take precedence over generic examples in this guidance.

Use the smallest applicable part of this guidance for the actual change.

---

## 0.1 Relationship With Native Workflow and Existing Skills

The intended separation is:

```text
existing codebase-investigation evidence
    "What exists and how does it work?"
        ↓
existing feature-impact evidence
    "What depends on this and what could break?"
        ↓
specialized evidence when relevant
    security / migration / performance / localization / quality
        ↓
Plemo native planning
        ↓
Plemo native implementation
        ↓
automated-test engineering
    "Which important behaviors deserve permanent executable protection,
     and how should those tests be implemented?"
        ↓
regression/runtime validation
    "What can now be proven in the actual available runtime environments?"
```

This guidance may run before implementation when test design materially affects architecture.

It may also run during or after implementation to add or repair test coverage.

When reliable evidence already exists, reuse it.

Useful inputs include:

```text
Feature owner
Affected modules
Affected files
Changed models/methods/fields
Reverse dependencies
Transitive consumers
Security surface
Upgrade/data risks
Frontend/JS/RPC surface
Integration contracts
Performance-sensitive paths
Regression scenarios
Acceptance criteria
Do-not-touch boundaries
Runtime validation requirements
Existing test failures
Actual final diff
```

Do not repeat full codebase investigation, impact analysis, security review, migration analysis, or runtime validation merely because this guidance is active.

---

## 0.2 Task Mode and Authorization

Determine the current task mode.

Typical modes:

```text
Test design / review only
Add automated tests
Fix broken tests
Implement feature + automated tests
Refactor tests
Increase coverage for a known regression
Convert manual regression steps into automated coverage
```

For **Test design / review only**:

- remain read-only;
- inspect existing tests and relevant production code;
- identify missing or weak coverage;
- recommend the smallest useful automated-test boundary;
- do not modify source or test files unless the user requested implementation.

For **Add automated tests**, **Fix broken tests**, **Implement feature + automated tests**, or **Refactor tests**:

- the user's original request authorizes test changes within that requested scope;
- do not ask for duplicate approval solely because test analysis completed;
- continue through Plemo's native planning and implementation workflow;
- keep production-code changes separate from test-only changes unless production behavior genuinely requires a fix;
- do not change production behavior merely to force a test to pass.

Ask for a new decision only when the test work reveals:

- a material behavior ambiguity;
- a destructive or irreversible operation;
- production-only data mutation;
- an external live-system side effect;
- a security-sensitive product decision;
- a migration/data-mapping decision;
- a major scope expansion;
- another unresolved choice where guessing would be unsafe.

---

## 0.3 Automated-Test Materiality Gate

Do not force large automated-test work on every change.

### New automated tests are usually unnecessary for:

- comments;
- formatting-only changes;
- documentation;
- repository metadata with no executable behavior;
- generated-file refreshes with no logic change;
- trivial wording changes already covered by a more appropriate localization/content process.

### Targeted automated tests are usually appropriate for:

- one model method;
- one compute;
- one constraint;
- one CRUD override;
- one wizard;
- one controller route;
- one report data-preparation method;
- one scheduled action;
- one integration adapter;
- one isolated frontend behavior when a supported frontend test mechanism exists.

### Strong automated regression coverage is preferred for:

- business invariants;
- bug fixes with reproducible regressions;
- accounting, stock, payment, payroll, subscription, or other high-impact workflows;
- stored computed fields;
- constraints;
- create/write/unlink behavior;
- methods with multiple callers or overrides;
- public/portal routes;
- ACL/record-rule-sensitive behavior;
- privileged `sudo()` paths;
- multi-company behavior;
- multi-website behavior;
- external API/webhook contracts;
- idempotency/retry behavior;
- Selection-key behavior;
- migration or existing-data transformations;
- `noupdate`-sensitive records;
- shared custom modules;
- frontend behavior with meaningful browser-side logic;
- recurring jobs whose duplicate execution could cause business damage;
- defects that previously escaped review.

Test depth should follow business and regression risk, not line count.

---

# 1. Determine the Test Target

Identify exactly what behavior needs automated protection.

Record when useful:

```text
Project:
Requested behavior:
Target module(s):
Feature owner:
Relevant model(s):
Relevant method(s):
Relevant field(s):
Relevant route(s):
Relevant view/template/component:
Known bug or regression:
Acceptance criteria:
Risk surface:
Expected test layer:
```

If the user supplied acceptance criteria, preserve them.

Do not replace explicit business acceptance criteria with generic technical assertions.

When acceptance criteria are implicit, infer only what is strongly supported by the task and existing evidence.

State material inferred assumptions when they affect the test contract.

# 2. Reuse Existing Evidence

Before creating a new test strategy, reuse reliable evidence from the current task.

Look for:

```text
CODEBASE INVESTIGATION evidence
IMPACT EVIDENCE
SECURITY EVIDENCE
MIGRATION EVIDENCE
PERFORMANCE EVIDENCE
code-quality findings
localization requirements
runtime-validation matrix
existing automated-test output
current implementation diff
```

Convert relevant evidence into permanent automated coverage where appropriate.

Do not mechanically turn every analysis finding into a test.

Automate scenarios whose future regression cost justifies permanent coverage.

# 3. Detect the Odoo Version

Determine the Odoo version from the strongest available repository evidence.

Valid evidence may include:

```text
target/custom module __manifest__.py version prefix
odoo/release.py
odoo/version.py
version_info
repository branch
Docker/build configuration
plemo.md
existing version-specific test imports
existing project test infrastructure
```

A valid manifest version prefix is acceptable primary evidence when Odoo core source is not present.

Examples:

```text
17.0.1.0.0 -> Odoo 17
18.0.2.0.0 -> Odoo 18
19.0.1.0.0 -> Odoo 19
```

Record:

```text
Odoo version:
Version evidence:
Edition if known:
Existing test framework evidence:
```

Do not assume version-sensitive APIs.

Verify actual repository/version support before selecting test base classes, Form helpers, browser/tour APIs, frontend unit-test frameworks, HTTP helpers, tag behavior, mail helpers, time-freezing helpers, registry helpers, or patching utilities.

Do not copy a test pattern from another Odoo version without verification.

# 4. Repository and Test Convention Precedence

Before creating test files, inspect the repository's existing test structure.

Use this precedence:

1. `plemo.md`;
2. project/customer test conventions;
3. nearby tests in the same addon;
4. shared test helpers already used by the repository;
5. Odoo test conventions appropriate to the detected version;
6. generic examples last.

Inspect where relevant:

```text
tests/
tests/__init__.py
test_*.py
common.py
test_common.py
fixtures
demo/test data
frontend test directories
tour directories
CI scripts
test tags
Docker/test commands
project-specific test runners
```

Prefer the repository's established naming, setup, helper, tagging, and fixture conventions.

Do not introduce a second test framework without a strong reason.

# 5. Inventory Existing Automated Coverage

Before adding tests, identify what already exists.

Search by model name, method name, field name, route, XML ID, business state, error message, bug symptom, controller path, RPC method, JS component/service, integration endpoint, Selection key, cron XML ID, and report XML ID.

Record:

```text
Existing direct tests:
Existing indirect tests:
Existing helper/base classes:
Existing fixtures:
Existing frontend/browser tests:
Existing security tests:
Existing upgrade/migration tests:
Known test gaps:
Potential duplicate tests:
```

Do not create a near-identical test under a new filename when the existing suite can be extended cleanly.

Do not delete apparently redundant tests without understanding whether they protect different contracts.

# 6. Determine Test Ownership and File Boundary

Place tests with the addon that owns the behavior being protected.

Prefer:

```text
<owner_addon>/tests/test_<feature>.py
```

or the repository's established equivalent.

Do not place customer-specific regression tests in a shared addon unless the shared addon owns the behavior.

Do not place tests in a downstream addon merely because the bug was observed there when the actual contract belongs upstream.

When a test intentionally verifies integration between multiple addons, place it where repository architecture and dependency direction make that ownership clear.

Avoid creating a new test addon solely for theoretical purity unless the repository already uses dedicated integration-test addons or the dependency graph requires one.

---

# TEST STRATEGY

# 7. Select the Smallest Sufficient Test Layer

Choose the lowest reliable layer that proves the contract.

Possible layers include:

```text
Python model/service test
ORM/business-flow test
security/user-context test
controller/HTTP test
report-generation test
cron/job test
integration-adapter test
upgrade/migration test
frontend unit test
browser/tour test
full cross-layer scenario
```

Prefer a model/service test when the behavior is server-side business logic.

Prefer an HTTP/controller test when route/auth/request/response behavior is part of the contract.

Prefer a browser/frontend test when the regression depends on OWL/component lifecycle, browser events, DOM state, asset loading, frontend service behavior, client-side navigation, JS -> RPC interaction, or rendered interaction.

Do not use a browser test merely because a feature has a button.

If the business behavior can be proven through the underlying server contract and the UI contains no meaningful client-specific logic, keep the test lower-level and more stable.

# 8. Build the Automated-Test Matrix

For material changes, build a test matrix before implementation.

Use:

```text
ID:
Behavior:
Risk source:
Test layer:
Actor:
Company/website:
Setup:
Action:
Expected result:
Negative/boundary case:
Existing test coverage:
New test required:
```

The matrix should come from acceptance criteria, actual feature ownership, impact/reverse-dependency evidence, security evidence, migration/data evidence, important runtime-validation scenarios, and known historical bugs.

Do not create a generic exhaustive matrix unrelated to the change.

# 9. Prioritize Contract Tests Over Implementation Tests

Prefer assertions about business state, computed results, access decisions, stable response contracts, idempotency, company isolation, upgrade-preserved values, and semantic report content.

Avoid brittle assertions about private-helper call counts, local variables, incidental ORM call order, internal method order, or other implementation details unless those details are themselves the contract.

Mock interactions only when the interaction itself is the contract or the external boundary must be isolated.

Tests should survive safe internal refactoring.

# 10. Positive, Negative, and Boundary Coverage

For behavior with meaningful failure modes, consider:

```text
expected valid input
missing required business condition
invalid state transition
empty recordset
single record
multiple records
minimum/maximum meaningful value
duplicate request
already-processed record
archived record
inactive configuration
missing optional configuration
unauthorized user
wrong company
wrong website
legacy data shape
external timeout/error
```

Do not add every possible permutation.

Select cases supported by real business or regression risk.

---

# TEST DATA AND ISOLATION

# 11. Deterministic Test Data

Create only the records necessary to prove the behavior.

Prefer explicit deterministic values for names, dates, amounts, currencies, companies, users, states, quantities, Selection keys, and external identifiers.

Avoid depending on current wall-clock time, random ordering, database sequence values, existing demo records, records created by another test, internet availability, live external services, or developer-local configuration.

When wall-clock behavior is part of the contract, use the version/project-supported deterministic time-control mechanism when available.

Do not introduce an unsupported third-party test library merely because it is familiar.

# 12. Test Isolation

Each test must be safe to run independently.

Do not depend on another test running first, class execution order, leftover database records, global mutable state created by another test, filesystem artifacts from another test, browser state from another scenario, or external sandbox state that is not resettable.

Use the repository/version-appropriate Odoo transactional test mechanism.

Do not assume a historical Odoo test base class still exists in the detected version.

Verify actual available base classes before implementation.

# 13. Shared Setup Quality

Use shared setup when it reduces repeated expensive or noisy fixture creation without hiding the scenario.

Good shared setup may include company, currency, user/groups, core product/customer, stable configuration, common model records, or a mock integration adapter.

Keep scenario-specific state inside the individual test when shared mutation would make tests hard to reason about.

Avoid giant base classes containing unrelated setup for many features.

# 14. Existing Reference Data

Use existing XML-ID reference data only when it is part of the stable contract.

Avoid making tests depend on optional demo data unless the test specifically concerns demo behavior.

When referencing groups, companies, sequences, stages, journals, templates, or other reference records:

- verify the XML ID exists in the detected Odoo version and installed dependencies;
- prefer records owned by declared dependencies;
- do not depend on unrelated optional addons.

---

# PYTHON / ORM TESTING

# 15. Model Method Tests

For important model methods, test the public or stable business entry point whenever possible.

Check when relevant:

```text
single-record behavior
multi-record behavior
record state before/after
return contract
created/updated records
side effects
messages/activities
context-sensitive behavior
company-sensitive behavior
errors for invalid conditions
```

Do not directly test a private helper when the public behavior proves the same contract, unless the helper itself is an intentionally reusable stable boundary.

# 16. CRUD Override Tests

For `create`, `write`, and `unlink` changes, include scenarios that represent actual Odoo entry paths.

Consider single create, batch create when supported/relevant, single write, multi-record write, allowed and denied unlink paths, defaults, computed/related effects, constraints, and tracking/automation side effects when material.

Protect framework contracts such as expected return values and recordset behavior.

Do not test CRUD only through a form if imports, RPC, cron, or backend automation can reach the same path.

# 17. Computed Field Tests

For computed fields, test the business result and the dependencies that should cause it to change.

For stored computes, consider initial computation, dependency update, multi-record recomputation, existing-record upgrade/recompute behavior, company-dependent inputs, and related dependent records.

Do not assert internal recompute implementation details unless required to prove a bug.

When existing-database recomputation is material, coordinate with migration/upgrade evidence rather than pretending a fresh test database proves upgrade safety.

# 18. Constraint and Validation Tests

For constraints and business validation, test valid cases, invalid cases, error type/message when contractually relevant, multi-record operations, write transitions, create paths, and historical/existing-data implications when material.

Avoid asserting the full exact error text unless wording is intentionally part of the user-facing contract.

Prefer asserting the meaningful semantic portion where repository conventions permit it.

# 19. Context-Dependent Behavior

When behavior changes under context, explicitly create the relevant context.

Examples include `active_id`, `active_ids`, `lang`, `tz`, `allowed_company_ids`, version-equivalent company controls, mail/tracking flags, custom feature flags, and website context.

Do not rely on the ambient test runner context accidentally matching the production path.

If a custom context flag controls important business behavior, include at least one test with and one without the flag when both paths matter.

# 20. Batch and Recordset Safety

If a method can receive multiple records, include a multi-record test when the change could introduce singleton assumptions.

Test representative batches rather than only one record repeatedly.

Do not add `ensure_one()` to production merely because the initial test was written for a singleton.

The test should reflect the real upstream contract.

---

# SECURITY TESTING

# 21. Actor-Specific Security Tests

When security is material, test with real user contexts rather than only administrator/superuser.

Representative actors may include:

```text
administrator
ordinary internal user
restricted internal user
manager
portal user
public user
integration/service user
company A user
company B user
```

Use only actors relevant to the feature.

Test both intended allow and intended deny behavior.

Admin success is not sufficient proof of access-control correctness.

# 22. ACL and Record Rule Coverage

For new or changed security-sensitive behavior, test create, read, write, and unlink as relevant, including records inside and outside the allowed scope.

Remember that ACLs and record rules are distinct mechanisms.

Do not use `sudo()` in the test merely to make the scenario pass unless elevation is part of the exact production contract being tested.

# 23. `sudo()` and Privileged Flow Tests

When production code uses `sudo()` or another elevated context, prove that caller authorization remains enforced.

A useful pattern is:

```text
authorized actor -> permitted business result
unauthorized actor -> denied before privileged effect
privileged operation -> limited to intended record/data
```

Do not test only the elevated internal implementation.

Test the caller-visible security boundary.

# 24. Multi-Company Tests

When company behavior is material, create explicit company-scoped scenarios.

Consider company A records, company B records, users allowed only one company or multiple companies, active-company switching, shared/global records, company-specific configuration, and cross-company relations.

Verify both business behavior and data isolation.

Do not assume setting `company_id` alone reproduces real user/company context.

Use the version-appropriate company/context mechanism.

# 25. Multi-Website Tests

When website behavior is material, test explicit website ownership/resolution.

Consider website-specific configuration, shared fallbacks, public/portal access, and website-owned records.

Do not claim multi-website safety from a single default website scenario.

---

# CONTROLLER / HTTP / RPC TESTING

# 26. Controller Contract Tests

When a controller is part of the feature contract, verify route, HTTP method, auth context, request payload, response status, response body/shape, redirects, validation failures, unauthorized access, record ownership, and company/website behavior as relevant.

Use the repository/version-appropriate HTTP testing mechanism.

Do not bypass route-level behavior by calling the controller method directly when routing/auth/request handling is exactly what needs proof.

# 27. Public and Portal Route Tests

For public/portal paths, include hostile or unauthorized record identifiers when IDOR-style access is a material risk.

Test owned records, unowned records, invalid/missing tokens, valid tokens, expired/revoked tokens when applicable, missing records, and malformed input as relevant.

Route `auth` configuration alone is not sufficient coverage.

Prove record-level authorization behavior.

# 28. JSON / RPC Contract Tests

For JSON/RPC behavior, protect stable contract elements such as field names, value types, error shape, pagination, state transitions, idempotency-key behavior, and caller-controlled IDs.

Avoid asserting incidental dictionary ordering unless the external contract requires ordering.

If frontend code consumes the response, use impact evidence to determine whether an additional frontend/browser test is justified.

---

# FRONTEND / OWL / BROWSER TESTING

# 29. Frontend Test Framework Gate

Before writing frontend tests, detect the actual frontend test framework used by the Odoo version and repository.

Possible mechanisms vary by version and repository.

Verify actual support before choosing frontend unit tests, QUnit-style tests, HOOT-style tests, web tours, `HttpCase` browser helpers, or project-specific browser runners.

Do not introduce a test API based on memory alone.

Prefer nearby working tests from the same Odoo version as the primary implementation reference.

# 30. Frontend Unit Tests

Use frontend unit tests for deterministic client-side logic such as component state transitions, service behavior, registry behavior, formatting logic, event handling, patch behavior, RPC-wrapper contracts, and template/component interaction.

Mock only true boundaries.

Do not mock the component under test so heavily that the test can pass while the real integration is broken.

# 31. Browser / Tour Tests

Use browser/tour-level testing when behavior depends materially on real UI interaction.

Examples include interaction sequences, modal/dialog lifecycle, navigation, rendered state, asset-loaded patch behavior, client-side validation, JS -> RPC -> rendered results, website flows, portal flows, and other multi-step browser workflows.

Keep tours focused.

Avoid giant tours covering an entire business application when smaller independent scenarios would isolate failures better.

Do not use arbitrary sleeps when the framework provides event/state-driven waiting.

# 32. Asset and Patch Regression Tests

For asset or patch changes, determine whether the important risk is bundle inclusion, module import, registry registration, patch application, patch ordering, selector/template availability, service dependency, or browser runtime behavior.

Static manifest inspection may be enough for a simple bundle declaration.

Actual patch behavior generally requires the version-appropriate frontend/browser test or runtime validation.

Do not claim browser behavior is protected by a Python import test.

---

# REPORT TESTING

# 33. Report Data Tests

Prefer testing report data preparation separately when business calculations are substantial.

Verify records included/excluded, totals, grouping, currency/company behavior, language-sensitive source values when material, and date/time boundaries.

Keep business calculations out of brittle PDF-byte assertions when a stable server-side data contract can be tested directly.

# 34. Rendered Report Tests

When rendered structure is itself important, use the repository/version-appropriate report-rendering mechanism.

Assert stable semantic output rather than full binary equality.

Possible stable checks include required text, record identity, expected section presence, expected generated attachment, or expected report action/output type.

Do not compare full PDF binary output unless the repository has a proven deterministic mechanism and binary identity is actually the contract.

---

# CRON / AUTOMATION / JOB TESTING

# 35. Scheduled Action Tests

For cron/scheduled behavior, test the callable business boundary directly where possible.

Consider eligible records, ineligible records, batch limits, already-processed records, failure isolation, idempotent reruns, company scope, and resulting state.

Do not make tests wait for real wall-clock cron execution.

Use deterministic invocation of the owned business method.

# 36. Idempotency Tests

For jobs, webhooks, imports, payments, or retryable workflows, test repeated execution when duplicates are plausible.

Useful contract:

```text
first execution -> intended business effect
second equivalent execution -> no duplicate destructive effect
```

When exact idempotency is not intended, test the documented retry behavior instead.

---

# INTEGRATION TESTING

# 37. External-Service Isolation

Automated tests must not call live external systems by default.

Avoid live payment gateways, live shipping providers, live SMS/email vendors, live ERP/accounting APIs, real webhooks, or internet-dependent HTTP endpoints.

Use the repository's established mock/fake/sandbox boundary.

If no safe boundary exists, identify the testability gap rather than silently creating a live dependency.

# 38. Mock at the Integration Boundary

Mock the narrow external boundary, not the Odoo business logic.

Prefer:

```text
mock HTTP transport/provider client
real Odoo model workflow
real mapping logic
real state transitions
real ORM persistence
```

over mocking the entire model method and asserting the mock returned its configured value.

Tests should still exercise the Odoo-owned contract.

# 39. Integration Failure Tests

When relevant, test timeout, connection error, non-2xx response, malformed response, partial response, duplicate webhook, out-of-order callback, retry, rate-limit response, and authentication failure.

Protect the required Odoo-side result:

```text
no duplicate records
stable retry state
clear failure status
no accidental invalid partial state
safe user-visible error
reconciliation possible
```

Do not simulate every provider error if only a small subset affects the business contract.

---

# MAIL / NOTIFICATION TESTING

# 40. Mail and Notification Tests

When mail/message/activity behavior is part of the requirement, use Odoo/project test helpers where available.

Verify semantic outcomes such as message creation, recipients, selected templates, created activities, notification suppression/enabling, and once-only triggering.

Avoid delivering real external email/SMS/WhatsApp from automated tests.

Do not assert unstable generated message IDs.

---

# UPGRADE AND MIGRATION TESTING

# 41. Fresh Database vs Existing Database

Do not confuse a test created on a fresh transactional database with existing-database upgrade proof.

For persistent-data changes classify:

```text
fresh-install behavior
existing-installed-database behavior
historical/legacy record behavior
```

If a change requires migration or recomputation, coordinate with migration evidence.

A normal model unit test cannot prove that an upgrade script transforms an existing database correctly.

# 42. Migration Test Coverage

When the repository supports migration tests or deterministic migration fixtures, test material transformations such as field renames, model moves, Selection mappings, required-field backfills, XML-ID mappings, `noupdate`-sensitive data, stored-compute historical behavior, relationship conversion, and legacy invalid values.

Verify row counts/reconciliation, mapping completeness, important business values, referential integrity, and expected/idempotent rerun behavior where relevant.

Do not invent a migration-test framework if the repository does not have one.

When automated upgrade proof is not feasible locally, record the required runtime upgrade scenario for the Runtime Validator.

# 43. Upgrade-Sensitive Tests

For changes that affect installed databases, add permanent tests for the post-upgrade business contract where useful even when the actual migration execution must be validated separately.

Keep migration execution proof and post-migration business-contract proof conceptually separate.

---

# SELECTION FIELD TESTING

# 44. Selection-Key Coverage

For changed Selection values or business logic based on Selection keys, search for material consumers.

Create tests for new valid keys, legacy keys when supported, mapped keys, invalid/removed-key behavior, domains/filters when business-critical, state transitions, reports, and integrations that branch on the key.

Do not test only the field declaration.

Selection values often behave as persisted API contracts.

---

# TESTING ERROR AND TRANSACTION BEHAVIOR

# 45. Exception Tests

Test expected exceptions at the business boundary.

Assert the relevant exception type and meaningful condition.

Avoid catching broad exceptions in the test merely to mark it successful.

When transaction rollback behavior matters, verify that prohibited partial state is not persisted.

# 46. Savepoint / Transaction Semantics

Understand the transaction behavior of the selected Odoo test base in the detected version.

Do not assume historical `SavepointCase` or `TransactionCase` semantics without verifying the actual framework.

When a scenario depends on commit/rollback boundaries, jobs, cursors, post-commit hooks, or separate HTTP transactions, choose a test layer that reproduces those semantics.

A convenient transactional unit test may be insufficient for a commit-sensitive bug.

# 47. Concurrency and Locking Tests

Do not create fake concurrency tests with sequential calls and label them concurrent.

When true concurrency/locking behavior is material:

- determine whether the repository has a supported test pattern;
- use separate cursors/transactions/processes only when the framework and environment safely support it;
- keep the scenario deterministic;
- avoid flaky timing races.

If reliable automated concurrency proof is not feasible, record the runtime validation requirement.

---

# TEST QUALITY AND MAINTAINABILITY

# 48. Naming Tests

Test names should communicate business behavior.

Prefer:

```text
test_portal_user_cannot_read_other_partner_order
test_duplicate_webhook_does_not_create_second_payment
test_confirm_recomputes_delivery_total
test_company_user_sees_only_company_configuration
```

Avoid vague names such as:

```text
test_method
test_case_1
test_fix
test_new_feature
```

Repository naming conventions take precedence.

# 49. One Failure Meaning

A test should ideally fail for one understandable business reason.

Do not place many unrelated workflows into one test merely to reduce setup code.

It is acceptable for one scenario to contain several assertions when they all prove one coherent contract.

# 50. Avoid Brittle Assertions

Avoid asserting incidental details such as database IDs, unordered record order, private-helper sequence, exact generated timestamps, full HTML/PDF byte equality, full translated error messages, implementation-specific call counts, or temporary DOM structure unless those details are intentionally part of the contract.

Prefer stable semantic assertions.

# 51. Avoid Over-Mocking

Excessive mocks can make Odoo tests meaningless.

Do not mock ORM create/write/search, computed fields, record rules, company behavior, constraints, or business state transitions when those are exactly what the test needs to prove.

Mock external boundaries and genuinely expensive/unavailable dependencies.

Exercise Odoo-owned behavior for real.

# 52. Avoid Tests That Duplicate Framework Internals

Do not write custom-module tests whose only purpose is proving standard Odoo behavior untouched by the module.

Test the module's customization of that behavior, not Odoo itself.

# 53. Test Helpers

Extract test helpers when they improve clarity or reduce meaningful repeated setup.

Good helpers have clear business intent such as:

```text
_create_portal_customer()
_create_confirmed_order()
_post_signed_webhook()
_assert_access_denied()
```

Avoid generic helper layers that hide every ORM action behind wrappers.

The test should still show the important business setup and action.

# 54. Shared Base Test Classes

Create or extend a shared test base only when multiple test modules genuinely share stable setup/behavior.

Avoid inheritance hierarchies that make individual tests impossible to understand without reading several files.

Prefer composition/simple helpers when inheritance adds little value.

# 55. Tags and Test Selection

Follow repository/Odoo conventions for test tags.

Use tags intentionally for standard install-time tests, post-install tests, slow/integration tests, frontend/browser tests, or project-specific CI groups.

Do not add custom tags unless the repository's test runner recognizes and uses them.

Do not mark important regressions out of the normal CI path merely to shorten test time without evidence.

# 56. Performance of the Test Suite

Test performance matters because slow suites are often skipped.

Avoid creating thousands of records when a small representative set proves the contract, repeatedly installing setup-heavy data per test, browser tests for server-only logic, real network waits, arbitrary sleeps, or huge report rendering for simple calculations.

Keep tests representative without making every test a load test.

Performance benchmarking itself belongs to the performance-analysis/measurement workflow, not ordinary correctness tests.

# 57. Flakiness Review

Treat flaky tests as defects.

Common Odoo flakiness sources include wall-clock time, record ordering without explicit order, shared mutable setup, external network, mail queues, cron timing, asynchronous browser state, arbitrary sleeps, company/user context leakage, timezone/language assumptions, filesystem leftovers, and non-deterministic generated values.

Fix the nondeterminism rather than increasing retries.

Do not add automatic retry as the default solution to a flaky correctness test.

# 58. Production Code Changes for Testability

Sometimes production code needs a small refactor to expose a stable testable boundary.

This can be appropriate when it also improves architecture, for example separating external transport behind a narrow provider method, request parsing from business action, report data preparation from rendering, or cron selection from per-record processing.

Do not add test-only branches, hidden context flags, public APIs, or production hooks solely to satisfy tests unless the repository explicitly supports such patterns.

Preserve the smallest safe production boundary.

---

# REVIEWING EXISTING TESTS

# 59. Review Test Intent

For an existing test, determine:

```text
What production contract is it protecting?
What regression would make it fail?
Is that contract still relevant?
Does the assertion actually prove it?
```

A passing test with no meaningful contract may create false confidence.

# 60. Detect False-Positive Tests

Watch for tests that never execute the changed path, assert only that a record exists, mock the method under test, catch and ignore all exceptions, assert a constant, use superuser and therefore bypass intended security, skip the branch that contains the regression, or fail to assert the business side effect.

Strengthen the test rather than merely preserving a green suite.

# 61. Detect False-Negative / Fragile Tests

Watch for failures caused by incidental record ordering, translation wording, generated IDs, timezone differences, unrelated demo data, non-contractual HTML, random values, network instability, test-order dependence, or version-specific helper changes.

Repair the assertion boundary without weakening the real business contract.

# 62. Removing or Replacing Tests

Do not delete a test simply because new implementation makes it inconvenient.

Before removing or replacing it, determine whether the protected behavior was intentionally removed, the behavior moved to another layer, another test now proves the same contract more reliably, or the old test was invalid/duplicated.

Preserve regression intent when refactoring test structure.

---

# IMPLEMENTATION PROCEDURE

# 63. Automated-Test Implementation Sequence

For material test work, use this sequence:

```text
1. Identify acceptance criteria and regression risk.
2. Reuse existing investigation/impact/specialist evidence.
3. Detect Odoo version.
4. Inspect repository test conventions.
5. Inventory existing related tests.
6. Choose the smallest sufficient test layer.
7. Build the test matrix.
8. Define deterministic fixtures and actors.
9. Implement the focused tests.
10. Run the narrowest relevant test target.
11. Fix test defects or production defects according to task authorization.
12. Run the broader relevant addon/test group.
13. Review the test diff for brittleness and duplication.
14. Hand runtime-only requirements to the existing validator.
```

Do not start by writing a large generic test file before understanding the protected contracts.

# 64. Test-First vs Test-After

This guidance does not force one universal development methodology.

A regression bug often benefits from:

```text
reproduce with failing automated test
    ↓
implement fix
    ↓
prove test passes
```

A new feature may use:

```text
define acceptance/test matrix
    ↓
implement feature and tests together
```

A risky legacy system may require investigation before any useful test can be written.

Choose the approach that produces reliable evidence with the least wasted work.

# 65. Reproducing a Bug

For a known bug, attempt to create a test that fails for the reported reason before fixing it when practical.

Record:

```text
Reproduction status:
Reproducing test:
Observed failure:
Expected behavior:
```

Do not force a pre-fix failing test when reproduction requires unavailable production-only state, the bug is already fixed in the working tree, creating the failing test would require destructive actions, or runtime-only behavior cannot be represented reliably.

In those cases, still create a permanent regression test for the final expected behavior when feasible.

# 66. Narrow Run Before Broad Run

Run the smallest relevant test target first, using verified repository/Odoo tooling.

Then expand to the relevant module/addon/test group when the narrow test passes.

Do not run an enormous unrelated suite first when a syntax/import error can be caught in one targeted test.

Do not claim broader regression safety from the narrow run alone.

# 67. Failure Classification

When a test fails, classify the failure before editing code.

Possible categories:

```text
production defect
incorrect test expectation
test setup defect
version/API mismatch
missing dependency
security-context mismatch
company/website-context mismatch
migration/data assumption
frontend timing/framework issue
external mock/fixture defect
pre-existing unrelated failure
environment/runtime limitation
```

Do not automatically change production code because a newly written test is red.

The test itself may be wrong.

# 68. Existing Suite Failures

If broader tests reveal existing failures:

- determine whether the failure existed before the task when evidence is available;
- do not silently fix unrelated failures;
- do not attribute unrelated failures to the current change;
- report them separately;
- continue focused validation when the unrelated failure does not block trustworthy conclusions.

# 69. Static Test File Validation

Before or alongside running Odoo tests, check:

```text
Python syntax
imports
tests/__init__.py wiring
manifest/dependency assumptions
frontend-test registration where relevant
test tags
unused imports
obvious dead fixtures
```

A test file that is not discovered by the runner provides zero protection.

# 70. Test Discovery Verification

Verify that the new test is actually collected/executed.

Evidence can include:

```text
test name in runner output
expected test count increase
intentional temporary failure during development when safe
runner discovery output
specific test selector execution
```

Do not assume a file under `tests/` automatically runs if package imports/tags/configuration exclude it.

# 71. Test Diff Review

Before finalizing, inspect the actual test diff.

Check for unrelated test rewrites, broad fixture changes, disabled/skipped tests, weakened assertions, accidental superuser use, hidden network calls, arbitrary sleeps, hardcoded developer paths, version-incompatible APIs, duplicate coverage, test-only production branches, and accidental generated artifacts.

Keep the test change inside the intended boundary.

---

# RUNTIME VALIDATION HANDOFF

# 72. Separate Automated Coverage From Runtime Proof

Automated tests are not automatically equivalent to runtime validation.

Examples:

```text
model test passes
    != actual module upgrade on existing database proved

frontend unit test passes
    != browser asset bundle and patch order proved

controller test passes
    != external live integration proved

migration fixture passes
    != production-sized migration runtime proved

security unit test passes
    != every installed-only record-rule interaction proved
```

Use automated tests as durable protection.

Use the existing Regression & Runtime Validator for environment-specific proof.

# 73. Runtime Requirements Handoff

When an important scenario cannot be fully automated in the repository, record it explicitly.

Use:

```text
RUNTIME TEST REQUIREMENT

Scenario:
Why automation is insufficient:
Required environment:
Required actor/company/website:
Required setup:
Expected result:
Risk if not executed:
```

Do not hide missing runtime proof behind a green unit-test result.

# 74. Browser Runtime Handoff

If frontend behavior depends on real bundle composition, browser state, installed-only templates, or environment-specific assets and the automated frontend framework cannot prove it completely, require browser validation.

Specify page/action, actor, company/website/language, interaction, expected visible result, and console/network expectations when material.

Do not simply write `Test manually in browser`.

# 75. Upgrade Runtime Handoff

If existing-database behavior is material and cannot be proven by repository migration tests, require a real module-upgrade scenario on a safe existing-data database.

Specify:

```text
pre-upgrade state
module version/state
representative historical records
upgrade action
post-upgrade assertions
log/error checks
reconciliation checks
```

Do not substitute fresh-install tests.

---

# OUTPUT CONTRACT

# 76. Test Engineering Evidence

For material automated-test work, produce a structured evidence block.

Use:

```text
TEST ENGINEERING EVIDENCE

Task mode:
Odoo version:
Version evidence:
Target module(s):
Feature owner:

Protected behavior:
Acceptance criteria:

Existing automated coverage:
Coverage gaps:

Selected test layer(s):
Test file(s):
Test class/base:
Version support verified:

Actors:
Companies/websites:
Fixtures:
External mocks/fakes:

Test scenarios:
- ...
- ...

Negative/boundary scenarios:
- ...
- ...

Security coverage:
Upgrade/migration coverage:
Frontend/browser coverage:
Integration coverage:

Tests added/changed:
Tests executed:
Narrow result:
Broader result:

Runtime-only requirements:
Do-not-touch boundary:
Remaining unknowns:

Coverage status:
Confidence:
```

Allowed coverage statuses:

```text
ADEQUATE FOR REVIEWED SCOPE
PARTIAL / RUNTIME COVERAGE REQUIRED
PARTIAL / ADDITIONAL AUTOMATION RECOMMENDED
BLOCKED
NOT APPLICABLE
```

Do not use `ADEQUATE FOR REVIEWED SCOPE` if material scenarios remain untested without explanation.

# 77. Concise Output for Embedded Implementation Work

When this guidance runs inside an already-authorized implementation task, keep internal evidence concise unless the test surface is complex.

A concise handoff may be:

```text
Automated test target:
Existing coverage:
New regression cases:
Selected layer:
Test file:
Actors/fixtures:
Runtime-only gaps:
Do-not-touch:
```

Then continue through Plemo's native implementation workflow.

Do not flood the user with internal skill routing or duplicate planning.

# 78. Full Test Review Output

For standalone test audits, complex feature suites, or high-risk changes, provide:

```text
1. Test Target
2. Odoo Version / Evidence
3. Existing Test Architecture
4. Existing Relevant Coverage
5. Coverage Gaps
6. Risk-to-Test Matrix
7. Test Layer Decisions
8. Fixture/Data Strategy
9. Actor/Security Matrix
10. Company/Website Matrix
11. Backend Test Scenarios
12. Controller/RPC Scenarios
13. Frontend/Browser Scenarios
14. Integration Scenarios
15. Upgrade/Migration Scenarios
16. Flakiness Risks
17. Testability Concerns
18. Recommended Test File Boundary
19. Runtime-Only Validation Requirements
20. Do-Not-Touch Areas
21. Remaining Unknowns
22. Test Engineering Evidence
```

Only include sections relevant to the reviewed feature.

Do not invent empty complexity.

# 79. Finding Severity for Test Reviews

When reviewing an existing test suite, classify findings by impact.

Possible severities:

```text
CRITICAL
HIGH
MEDIUM
LOW
INFO
```

Examples:

- **CRITICAL**: tests perform destructive live external side effects; security regression coverage systematically bypasses authorization; migration tests corrupt shared persistent data.
- **HIGH**: a known high-impact regression has no stable automated protection; public/portal tests bypass routing/authorization; integration tests call live providers; new tests are not discovered by CI.
- **MEDIUM**: important negative paths are missing; tests are flaky; fixtures obscure business intent; tests mock the behavior they claim to validate.
- **LOW**: unclear naming, small fixture duplication, or minor organization inconsistency.

Keep severity separate from confidence.

Do not label style preferences as high-severity defects.

# 80. Test Coverage Confidence

Express confidence based on evidence.

Useful levels:

```text
HIGH
MEDIUM
LOW
```

### HIGH

Relevant existing tests were inspected, new tests were discovered and executed, important positive/negative contexts are covered, broader related tests passed, and runtime-only limitations are explicitly separated.

### MEDIUM

Targeted tests pass but broader environment/runtime proof is unavailable, or some important dynamic consumer is known but not automatable locally.

### LOW

Test framework/version support is uncertain, tests cannot be executed, repository discovery is unknown, or important runtime-only behavior remains unresolved.

Never convert lack of evidence into confidence.

# 81. Do-Not-Touch Boundary

Record explicit boundaries when useful.

Examples:

```text
Do not rewrite unrelated test bases.
Do not rename stable production APIs for test convenience.
Do not weaken ACLs or use broad sudo to make tests pass.
Do not add live external network dependencies.
Do not replace the repository test framework.
Do not rewrite all legacy tests while adding one regression case.
Do not regenerate demo/reference data unnecessarily.
Do not convert every runtime validation scenario into an end-to-end browser test.
```

This boundary protects scope and maintainability.

# 82. Stop Conditions

Stop and report rather than guessing when:

- the Odoo version cannot be determined and the required test API is version-sensitive;
- the repository's test framework cannot be identified;
- required runtime/database access is unavailable for a scenario that cannot be represented safely;
- test execution would require destructive production changes;
- a live external service would be called without an approved sandbox/mock;
- security behavior is ambiguous and guessing could grant access;
- migration mapping is ambiguous;
- the user asked only for review and implementation would be required;
- existing failures prevent trustworthy interpretation of the requested test result.

A stop condition should identify:

```text
what is blocked
why it is blocked
what evidence is already established
what exact input/environment is required next
```

Do not convert a blocked scenario into a false PASS.

# 83. Final Principles

The objective is not:

```text
maximum test count
maximum mocking
maximum browser automation
maximum code coverage percentage
```

The objective is:

```text
small, durable, deterministic Odoo tests
that protect important business and framework contracts
at the correct layer
with explicit runtime gaps
```

A strong Odoo automated test suite should make future unsafe changes fail for understandable reasons.

Prefer:

```text
business contract
    >
implementation detail

determinism
    >
timing luck

real Odoo behavior
    >
over-mocked behavior

focused test layer
    >
unnecessary end-to-end complexity

actor-specific security proof
    >
admin-only success

existing-database awareness
    >
fresh-database assumptions

version-verified APIs
    >
remembered APIs

repository conventions
    >
generic style

durable regression protection
    >
temporary manual confidence
```

The final result should make clear:

```text
what behavior is permanently protected
which automated tests prove it
which important scenarios remain runtime-only
and how confident we are in the coverage
```
