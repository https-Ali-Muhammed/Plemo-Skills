# Odoo Code Quality Reviewer

## Purpose

Use this guidance to review Odoo code for maintainability, clarity, correctness of extension patterns, consistency with the repository, and long-term safety of future changes.

The goal is not to enforce personal style preferences.

The goal is to answer:

```text
Is this Odoo implementation clear, maintainable, idiomatic for this repository,
safe to extend, and unlikely to create unnecessary technical debt?
```

This guidance adds Odoo-specific code-quality procedures and evidence. It does not replace Plemo's native repository discovery, task-mode handling, planning, implementation, debugging, or approval behavior.

Core principles:

- Follow `plemo.md` and repository-specific conventions first.
- Prefer existing project patterns over generic style preferences.
- Distinguish correctness defects from maintainability concerns.
- Distinguish objective repository inconsistency from subjective style.
- Do not refactor working code merely to make it look different.
- Do not create abstraction without demonstrated reuse or complexity reduction.
- Do not split code into extra modules/classes/files merely for theoretical purity.
- Preserve public behavior and extension contracts unless change is explicitly authorized.
- Prefer the smallest maintainable boundary.
- Reuse reliable evidence already collected during the current task.
- Do not repeat investigation that is already sufficient.
- Do not force heavyweight review on trivial changes.
- Do not announce or expose internal capability selection during normal conversation.

---

# 0. Native Plemo Compatibility

This guidance must fit Plemo's normal development cycle.

Repository-specific instructions such as `plemo.md`, customer restrictions, repository structure, addon conventions, naming conventions, version constraints, deployment rules, and available tools take precedence over generic examples here.

Plemo already handles repository discovery, task-mode understanding, planning, implementation, ordinary debugging, targeted validation, and final diff review.

This guidance only adds code-quality evidence and Odoo-specific maintainability checks.

Do not create a second planning system.

Do not require a second approval when the user's original request already authorizes implementation.

Do not make a review-only request modify source code.

## 0.1 Task Mode

Determine the current mode from the user's request.

Typical modes:

```text
Review / audit only
Diagnose code-quality problem
Fix / refactor
Implement feature with quality safeguards
Validate a refactor
```

For review / audit only:

- remain read-only;
- do not silently refactor;
- report findings with evidence;
- identify the smallest safe improvement boundary.

For fix / refactor / implement:

- use the user's original request as implementation authorization for that scope;
- do not ask for duplicate approval solely because quality analysis finished;
- continue through Plemo's normal planning and implementation behavior;
- ask only when a material ambiguity, behavior change, destructive change, or scope expansion requires a real decision.

For validation after refactor:

- verify that behavior and extension contracts remain intact;
- perform broader runtime checks when the change requires them.

## 0.2 Review Materiality Gate

Do not force a full code-quality review for every change.

Light review is normally sufficient for comments, documentation, labels, formatting, and tiny configuration edits.

Targeted review is appropriate for one method, compute, controller, inherited view, JS component, report, cron, or local refactor.

Full review is strongly preferred for shared custom modules, large methods/classes, repeated business logic, multiple inheritance layers, widely called overrides, public/portal controllers, broad `sudo()` usage, schema/model redesign, duplicated integrations, fragile inherited views, global frontend patches, major refactors, or major Odoo-version upgrades.

Review depth should follow maintenance risk, not file count.

---

# 1. Determine the Review Target

Record when useful:

```text
Project:
Requested change:
Target module(s):
Target file(s):
Primary models:
Primary methods:
Primary views/templates:
Primary controllers/routes:
Primary JS components:
Known maintenance concern:
```

If the target is already clear, do not ask the user to repeat it. Use repository discovery before asking unnecessary questions.

# 2. Detect Odoo Version

Determine the Odoo version from reliable repository evidence such as the target module `__manifest__.py` version prefix, Odoo release/version files, repository branch, `plemo.md`, build configuration, or version-specific APIs already used.

A valid manifest version prefix is acceptable evidence when core source is unavailable.

Do not recommend version-specific APIs without confirming the version.

# 3. Repository Convention Precedence

Before judging code quality, inspect how the repository already solves similar problems.

Prefer, in order:

1. `plemo.md`;
2. existing customer/project modules;
3. existing shared project conventions;
4. nearby comparable implementation;
5. Odoo conventions appropriate to the detected version;
6. generic style guidance last.

Do not call code poor quality merely because it differs from a generic preference while matching an intentional repository convention.

# 4. Reuse Existing Evidence

Reuse reliable evidence already available from the current task, such as feature ownership, inheritance chains, callers, reverse dependencies, runtime paths, upgrade/data risks, security-sensitive paths, performance-sensitive paths, existing tests, and the final diff.

Do not rebuild the same evidence unless it is incomplete or stale.

# 5. Review the Actual Diff

For implemented work, inspect the actual change rather than only the intended plan.

Look for:

- unrelated edits;
- accidental reformatting;
- deleted logic;
- copied logic;
- debug code;
- stale temporary code;
- commented-out production code;
- unnecessary file movement;
- unexpected dependency additions;
- large refactors mixed with a small functional change.

Prefer keeping functional changes and unrelated cleanup separate when practical.

---

# MODULE AND ADDON DESIGN

# 6. Correct Modification Boundary

Determine whether the implementation is placed in the right addon/module.

Check feature ownership, existing extension modules, dependency direction, customer restrictions, and shared-versus-project-specific behavior.

Do not create a new addon solely for theoretical separation if the project expects extension inside an existing addon.

Do not put customer-specific behavior into a shared addon without evidence that it belongs there.

# 7. Avoid Unnecessary New Modules

A new addon should solve a real ownership, deployment, dependency, or reuse problem.

Question patterns such as one tiny field, one label change, or one local button creating a new addon.

Do not prohibit a new addon when the repository architecture clearly requires it.

# 8. Dependency Quality

Inspect manifest dependencies for missing, unnecessary, indirect-only, or circular dependencies.

Keep dependency direction intentional and consistent with ownership.

# 9. File and Package Organization

Check whether files follow the repository's established module structure.

Do not split a small coherent implementation across many files without benefit.

Do not keep unrelated responsibilities in one giant file when separation would materially improve maintainability.

---

# PYTHON MODEL QUALITY

# 10. Correct `_name` / `_inherit` Usage

Verify model extension intent.

Do not accidentally create a new model identity when extension inheritance is intended.

Review `_name`, `_inherit`, `_inherits`, `_rec_name`, and `_order` when relevant.

# 11. Preserve Upstream Contracts

For overridden methods inspect signature, recordset expectations, return value, context assumptions, exceptions, side effects, and `super()` behavior.

An override should preserve the upstream contract unless the authorized change intentionally modifies it.

# 12. `super()` Quality

Check whether `super()` is called when required, on the correct recordset, and at the right point in lifecycle behavior.

Avoid copy-pasting large upstream methods merely to change one small piece when a stable extension point exists.

If copying upstream logic is unavoidable, document the compatibility risk.

# 13. Multi-Record Safety

Review methods for hidden singleton assumptions.

Determine whether upstream callers can pass multiple records.

Do not add `ensure_one()` merely to silence an error when batch behavior should be supported.

# 14. Method Size and Responsibility

Large methods are a concern when they mix unrelated responsibilities such as validation, lookup, calculation, writes, notifications, external calls, and response formatting.

Extract helpers only when the separation produces a clearer contract.

Do not create one-line helpers merely to satisfy a style rule.

# 15. Helper Method Quality

Good helpers should have clear names, inputs, outputs, side effects, and recordset expectations.

Avoid helpers that only rename one expression or make simple logic harder to follow.

# 16. Naming Quality

Names should reveal business intent.

Avoid vague names such as `data`, `res`, `temp`, `do_stuff`, or `process_data` when the surrounding logic is complex.

Do not rename stable public methods/fields solely for aesthetics.

# 17. Magic Values

Review repeated state strings, context keys, XML IDs, selection keys, timeouts, and limits.

Extract constants only when doing so reduces duplication or error risk.

Do not create a constant for every literal automatically.

# 18. Context Usage

Inspect `with_context()`, `env.context`, `active_id`, `active_ids`, `allowed_company_ids`, language/timezone keys, defaults, and custom flags.

Context keys should have clear ownership and narrow scope.

Avoid hidden control flow driven by undocumented context flags.

# 19. Side Effects

A method should make significant side effects visible from its name and placement when practical.

Watch for hidden writes, messages, emails, external calls, record creation, or state changes inside apparently read-only helpers.

---

# FIELDS AND ORM DESIGN

# 20. Field Definition Quality

Review field type, required/readonly/copy/index/tracking options, company behavior, compute/inverse/search/store, and groups where relevant.

Do not add options without a demonstrated requirement.

# 21. Computed Field Clarity

For computed fields inspect compute method naming, dependencies, storage, inverse/search behavior, assignment for all records, and multi-record handling.

Keep compute logic focused on computing the field.

Avoid unrelated business side effects inside compute methods.

# 22. `@api.depends` Quality

Dependencies should be complete but not unnecessarily broad.

Missing dependencies create stale values; overly broad dependencies create unnecessary recomputation.

Use the actual data relationship.

# 23. Related Field Quality

Use related fields when they represent a stable relationship.

Review path, storage, readonly behavior, company behavior, and exposure of restricted data.

Do not persist a related field merely to avoid a simple traversal when persistence/search is unnecessary.

# 24. Onchange vs Server-Side Logic

Do not rely on `onchange` for business correctness.

If a rule must also hold for imports, RPC, cron, server actions, batch writes, or integrations, enforce it in an appropriate server-side layer.

# 25. Constraint Quality

Constraints should express real invariants, handle multi-record `self`, avoid unrelated side effects, provide useful errors, and avoid repeated expensive work.

Do not use constraints for behavior better represented as workflow validation.

# 26. CRUD Override Quality

For `create`, `write`, and `unlink`:

- preserve framework semantics;
- support batch behavior where appropriate;
- call `super()` correctly;
- avoid duplicate logic;
- avoid hidden recursive writes;
- avoid changing unrelated fields;
- preserve return contracts.

# 27. Search and Domain Readability

Complex domains should remain understandable.

For repeated non-trivial domain fragments, consider a helper only when reuse or clarity justifies it.

Do not build domains through opaque incremental mutation when a direct expression is clearer.

---

# SECURITY-AWARE QUALITY

# 28. `sudo()` Maintainability

Treat `sudo()` as a privileged boundary, not a convenience.

Inspect why elevation is required, how broad its scope is, whether caller authorization is visible, and whether returned data remains restricted.

Do not recommend `sudo()` merely to avoid access errors.

When access control is material, apply the relevant security checks without exposing internal capability names to the user.

# 29. Company-Aware Quality

For company-aware models inspect clarity around `company_id`, `allowed_company_ids`, `with_company()`, `check_company`, and shared records.

Avoid hidden assumptions that code always runs in one company.

---

# CONTROLLERS AND API QUALITY

# 30. Controller Responsibility

Controllers should primarily handle request parsing, authentication context, authorization entry, calling business logic, and response formatting.

Avoid embedding large reusable business workflows directly inside controller methods.

Move reusable business logic to an appropriate server-side boundary when this improves reuse and testability.

# 31. Route Clarity

Review route path, auth, type, methods, CSRF, and website behavior.

The route declaration should match the method's real intent.

Do not override routes casually without understanding the upstream decorator contract.

# 32. Input Mapping

Avoid passing raw request dictionaries directly into broad model `create()`/`write()` calls.

Prefer explicit mapping of fields the endpoint owns.

# 33. Response Contracts

For JSON/API routes keep response shape intentional and stable.

Avoid returning raw ORM records or unnecessarily large payloads.

Treat externally consumed field names and types as compatibility contracts.

---

# XML, QWEB, AND REPORT QUALITY

# 34. View Inheritance Quality

Prefer the smallest stable inheritance point.

Review `inherit_id`, XPath, `position`, priority, and downstream inheritors.

Avoid replacing large sections of upstream views when a smaller insertion or attribute change can safely achieve the requirement.

# 35. XPath Robustness

Fragile XPath includes deep absolute paths, position-based indexes, translated labels, or volatile generated structures.

Prefer stable fields, attributes, IDs, and structural anchors.

Do not make XPath overly broad.

# 36. `position="replace"` Caution

Replacing upstream nodes increases maintenance risk because downstream inheritors may depend on the original node.

Use replacement only when a smaller extension cannot meet the requirement.

# 37. XML ID Quality

Use stable, descriptive XML IDs.

Do not rename existing XML IDs merely for cosmetics because they may be referenced externally or in installed databases.

# 38. QWeb Template Responsibility

Avoid placing complex business calculations directly in templates.

Prepare reusable business data before rendering when this improves clarity and testability.

# 39. Template Inheritance

Prefer narrow template inheritance over full template duplication.

If copying is unavoidable, document why inheritance cannot safely express the change and recognize the upgrade debt.

# 40. Report Data Preparation

Complex reports should have a clear data preparation boundary.

Avoid repeated searches and duplicated business rules inside QWeb expressions.

---

# JAVASCRIPT / OWL / ASSETS

# 41. Frontend Ownership

Place frontend behavior in the module that owns the feature.

Avoid global patches for behavior belonging to one local screen when a narrower extension point exists.

# 42. Patch Quality

For patches inspect target, reason, scope, overridden behavior, super behavior, version sensitivity, and other patches.

A global patch should have strong justification.

# 43. Component Responsibility

Avoid components that mix data loading, business calculation, RPC, DOM manipulation, navigation, and notifications without a clear structure.

Do not split components aggressively if the result becomes harder to follow.

# 44. State Quality

Keep component state minimal and intentional.

Avoid duplicate sources of truth or storing values that are cheaply derived unless caching has a clear reason.

# 45. RPC Ownership

Keep frontend RPC/model/route calls understandable.

Avoid duplicating the same wrapper across multiple components.

Do not create a generic abstraction without demonstrated reuse.

# 46. DOM Manipulation

Prefer framework/component mechanisms over manual DOM manipulation where practical.

Manual DOM code should have clear lifecycle and cleanup behavior.

Avoid selectors tied to fragile markup.

# 47. Asset Scope

Place files in the narrowest appropriate bundle.

Avoid adding local behavior to a global bundle unnecessarily.

# 48. CSS/SCSS Quality

Prefer scoped selectors.

Avoid broad tag overrides, excessive `!important`, and selectors dependent on unstable DOM depth.

Do not introduce a new design system for one small fix.

---

# DUPLICATION, ABSTRACTION, AND DEAD CODE

# 49. Duplication Review

Not all duplication needs abstraction.

Classify duplicated code as coincidental, one business rule, one integration contract, one formatting concern, or one technical helper.

Abstract only when duplicated code represents one concept that should change together.

# 50. Premature Abstraction

Warning signs include generic helpers used once, base classes with one subclass, configuration layers for one constant, factories for one implementation, or service wrappers around one trivial call.

Prefer simple code until real use cases justify abstraction.

# 51. Over-Generalized Helpers

Helpers with many flags can hide multiple responsibilities.

Prefer clearer domain-specific methods when behavior becomes hard to reason about.

Do not split solely to reduce argument count if business semantics remain coherent.

# 52. Copy-Paste From Upstream

Copying large upstream methods/templates creates upgrade risk.

Prefer `super()`, a small hook, targeted override, or narrow inheritance when available.

If copying is unavoidable, record source version, reason, and compatibility risk.

# 53. Dead Code

Look for unused methods, fields, imports, stale commented code, unreferenced templates, old feature flags, and obsolete compatibility branches.

Do not delete code solely because local search finds no caller if dynamic/runtime use remains plausible.

# 54. Temporary Compatibility Code

Temporary migration/version compatibility code should have a clear purpose and removal condition.

Avoid permanent temporary branches with no documented lifecycle.

---

# ERROR HANDLING AND OBSERVABILITY

# 55. Exception Quality

Use framework/business exceptions appropriately.

Avoid bare `except`, silent `except Exception`, or swallowing errors that should stop the workflow.

# 56. User-Facing Errors

Error messages should explain the business problem without exposing internal details.

Keep technical diagnostics in logs where appropriate.

# 57. Logging Quality

Logging should be actionable, appropriately leveled, non-duplicative, and safe for sensitive data.

Avoid verbose logs in hot loops and remove debug prints.

# 58. Assertions

Do not rely on Python `assert` for user/business validation.

Use explicit validation appropriate to Odoo behavior.

---

# TESTABILITY AND COMPATIBILITY

# 59. Testable Boundaries

Business logic is easier to maintain when it has clear inputs/outputs and limited hidden context dependencies.

Do not refactor solely for testability if production code becomes less clear.

# 60. Existing Tests

Review relevant existing tests before proposing new structure.

Prefer extending established coverage patterns.

Do not duplicate tests that prove the same behavior.

# 61. Regression-Sensitive Refactors

Even behavior-preserving refactors require regression awareness.

Especially review method extraction, batching, domain rewrites, controller/model separation, template inheritance changes, and frontend restructuring.

# 62. Version-Specific APIs

Do not introduce APIs from another Odoo version.

Confirm Python APIs, view syntax, controller APIs, frontend imports, patch syntax, and service APIs against the detected version.

# 63. Upgrade Fragility

Identify code tightly coupled to unstable upstream details, such as deep XPath into internal layouts, copied upstream methods, patches of private frontend internals, or hardcoded core XML structure.

Prefer stable extension points.

# 64. Backward Compatibility

When modifying a method, field, route, or response used elsewhere, identify compatibility needs for other modules, frontend callers, integrations, reports, automations, and stored data.

Do not rename public contracts solely for naming cleanliness.

---

# FINDINGS AND OUTPUT

# 65. Finding Categories

Use categories such as:

```text
Correctness risk
Maintainability risk
Upgrade fragility
Readability issue
Duplication
Over-abstraction
Repository inconsistency
Dead/stale code
Testability concern
Version compatibility concern
```

# 66. Severity

Use:

```text
Blocker
Major
Moderate
Minor
Informational
```

Severity should reflect future maintenance/correctness impact, not personal dislike.

# 67. Confidence

Use:

```text
Confirmed
Strong evidence
Probable
Possible
Needs runtime verification
```

Keep confidence separate from severity.

# 68. Objective vs Preference

For each finding ask:

```text
Does this violate repository convention?
Does this violate an Odoo contract?
Does it duplicate one business rule?
Does it create upgrade/runtime risk?
Or is it merely a different style?
```

Do not report personal preference as a defect.

# 69. Finding Structure

For each material finding record:

```text
Finding ID:
Title:
Category:
Severity:
Confidence:
Location:
Observed pattern:
Why it matters:
Repository/Odoo evidence:
Affected extension point:
Recommended improvement boundary:
Behavior that must remain unchanged:
Validation required:
```

Do not turn this into a generic implementation plan.

# 70. Improvement Boundary

Recommend the smallest maintainable improvement.

Record:

```text
Module:
File/component:
Responsibility to improve:
Preferred boundary:
Do-not-touch areas:
Reason:
```

Avoid unrelated cleanup.

# 71. Do-Not-Touch Boundary

For material refactors identify:

```text
Odoo core:
Third-party/OCA:
Shared custom modules:
Sibling customer projects:
Public APIs/routes:
Stored schema/data:
Security behavior:
Business workflow:
Other protected areas:
```

Do not refactor across these boundaries without evidence and authorization.

# 72. Code Quality Evidence Summary

For material reviews use:

```text
CODE QUALITY EVIDENCE

Project:
Review target:
Odoo version:

Repository convention fit:
Module boundary:
Inheritance quality:
Method/API contract quality:
ORM/model quality:
View/QWeb quality:
Controller/API quality:
Frontend quality:
Duplication:
Abstraction level:
Error/logging quality:
Testability:
Version/upgrade fragility:

Blocker findings:
Major findings:
Moderate findings:
Minor findings:

Recommended improvement boundary:
Behavior that must remain unchanged:
Runtime/regression verification required:
Remaining unknowns:
Overall assessment:
```

# 73. Overall Assessment

Use:

```text
ACCEPTABLE FOR REVIEWED SCOPE
IMPROVEMENT RECOMMENDED
REFACTOR REQUIRED
PARTIAL
BLOCKED
```

`ACCEPTABLE FOR REVIEWED SCOPE` means no material code-quality problem was identified for the reviewed area.

`IMPROVEMENT RECOMMENDED` means real improvements exist, but the implementation is not fundamentally unsafe.

`REFACTOR REQUIRED` means a material structural/maintainability problem should be corrected before the change is considered safe to maintain.

`PARTIAL` means useful evidence exists but important context or runtime behavior is unavailable.

`BLOCKED` means critical repository/version/ownership context is missing.

# 74. Detailed Report Structure

For complex reviews use:

1. Review Target
2. Odoo Version and Repository Conventions
3. Actual Diff
4. Module/Dependency Design
5. Python Model Design
6. Fields/ORM
7. Inheritance / `super()`
8. Context / Side Effects
9. Controllers / API
10. XML / QWeb / Reports
11. JavaScript / OWL / Assets
12. Duplication / Abstraction
13. Error Handling / Logging
14. Testability / Existing Tests
15. Version / Upgrade Fragility
16. Findings
17. Recommended Improvement Boundary
18. Behavior to Preserve
19. Required Validation
20. Remaining Unknowns
21. Overall Assessment

Do not force the full report for a small local review.

# 75. Concise Output for Small Reviews

Use:

```text
Code Quality:
ACCEPTABLE / IMPROVEMENT RECOMMENDED / REFACTOR REQUIRED / PARTIAL / BLOCKED

Main concern:
Evidence:
Recommended boundary:
Behavior to preserve:
Validation:
```

# 76. Fix / Refactor Flow

When the user's request already authorizes changes:

```text
Review evidence
    ↓
Identify smallest maintainable boundary
    ↓
Plemo performs its normal planning
    ↓
Implement the authorized change
    ↓
Recheck final diff
    ↓
Verify behavior and extension contracts
```

Do not add a second approval step solely because review occurred.

# 77. Final Diff Recheck

After refactoring/implementation check:

- no unrelated cleanup slipped in;
- public method signatures remain intentional;
- XML IDs remain stable unless intentionally changed;
- imports/package wiring are correct;
- no debug code remains;
- no temporary compatibility code was accidentally left;
- no behavior changed outside authorized scope.

# 78. Pre-Existing Quality Debt

Separate:

```text
introduced by current change
pre-existing
exposed by current change
unknown origin
```

Do not blame the current task for historical debt without evidence.

Do not silently expand scope to clean unrelated debt.

# 79. Hard Prohibitions

Never:

- replace Plemo's native planner or implementation workflow;
- require duplicate approval for already-authorized work;
- modify source during review-only mode;
- announce or expose internal capability selection during normal chat;
- force generic style preferences over `plemo.md` or repository conventions;
- create a new addon merely for theoretical purity;
- split every method into tiny helpers;
- introduce generic abstractions with no real reuse;
- rename stable public APIs/XML IDs solely for aesthetics;
- copy large upstream methods/templates when a stable extension point exists;
- call `ensure_one()` automatically to hide multi-record bugs;
- rely on `onchange` for business correctness;
- use `sudo()` as a generic AccessError fix;
- remove security behavior to simplify code;
- move customer-specific behavior into shared modules without evidence;
- refactor across sibling customer projects without repository evidence;
- delete apparently unused code without considering dynamic/runtime callers;
- rewrite complex but correct code without demonstrated maintainability benefit;
- mix unrelated cleanup into a small functional fix unnecessarily;
- introduce version-incompatible APIs;
- claim quality improvement when behavior or maintainability became less clear;
- commit or push unless explicitly requested.

# 80. Core Decision Tree

```text
START
  ↓
Read plemo.md / project rules
  ↓
Determine task mode
  ↓
Reuse existing investigation evidence
  ↓
Detect Odoo version
  ↓
Inspect repository conventions
  ↓
Inspect actual diff / target code
  ↓
Is review material?
  ├─ No → light review
  └─ Yes
       ↓
Check module ownership / dependencies
       ↓
Check model / inheritance / super contracts
       ↓
Check fields / compute / CRUD / context
       ↓
Check controllers / routes if relevant
       ↓
Check XML / QWeb / reports if relevant
       ↓
Check JS / OWL / assets if relevant
       ↓
Check duplication / abstraction / dead code
       ↓
Check errors / logging / tests / version fragility
       ↓
Classify findings objectively
       ↓
Recommend smallest improvement boundary
       ↓
Review-only?
       ├─ Yes → report and stop
       └─ No → continue through Plemo's normal plan/implementation
       ↓
Recheck final diff and required behavior
```

# 81. Primary Rules to Always Remember

```text
Code quality is maintainability plus correctness of extension boundaries.

Use plemo.md and repository conventions first.

Do not confuse personal style with a defect.

Preserve Plemo native planning, implementation, and task mode.
Do not require duplicate approval.

Reuse existing evidence.
Do not duplicate investigation.

Prefer existing addon/module ownership.

Do not create abstractions without real benefit.

Do not create new addons solely for theoretical separation.

Preserve upstream contracts and super() behavior.

Support multi-record semantics when the upstream contract requires it.

Keep compute methods focused and dependencies accurate.

Do not rely on onchange for server-side correctness.

Keep controllers focused on transport and authorization boundaries.

Prefer narrow, stable XML/QWeb inheritance.

Avoid global frontend patches when a narrower extension exists.

Avoid copied upstream implementations when stable extension points exist.

Keep context flags understandable and narrowly scoped.

Separate:
correctness defect
maintainability risk
repository inconsistency
style preference

Recommend the smallest improvement boundary.

Preserve behavior unless the user authorized behavior change.

Do not announce internal capability selection.

After refactoring, verify the final diff and relevant behavior.
```
