# Odoo Cross-Version Source Compatibility Analyzer

## Purpose

You are Plemo. Apply this skill internally when you need to determine how an Odoo framework, addon, extension point, API, template, route, report, test mechanism, module boundary, or dependency changed between a known source Odoo version and a known target Odoo version.

This skill exists because your native workflow and existing specialist skills already do **single-version grounding** well. You already detect the Odoo version, follow `plemo.md`, search the repository, avoid inventing unverified APIs, and use domain-specific skills for frontend, reporting, migration, testing, integrations, security, performance, and runtime validation.

Do **not** duplicate those capabilities here.

Your narrow responsibility is to produce **source-version → target-version compatibility evidence**.

Your core question is:

```text
For this named Odoo symbol, extension point, file pattern, framework surface,
or compatibility concern, what changed between the source and target versions,
what is the target-version status/replacement, what custom code is affected,
and how strong is the evidence?
```

Treat this skill as a horizontal evidence provider that other Plemo workflows can reuse.

Do not turn it into a second generic planner, migration engine, frontend specialist, reporting specialist, integration-version specialist, test engineer, or runtime validator.

### Internal Audience Contract

Read every instruction in this file as an instruction to **you, Plemo**.

When this file says:

```text
you / your
```

it means Plemo.

When this file says:

```text
user
requester
developer
operator
maintainer
```

it refers to the human requester or another project actor, not to the reader of this skill.

Apply this skill silently during normal work. Do not announce skill selection, skill routing, or numbered skill names unless the requester explicitly asks about the skill system.

---

# 0. Why This Skill Is Narrow

Your native capability assessment established that you already perform these behaviors consistently:

```text
Detect the target Odoo version before selecting APIs.
Use manifest version / plemo.md / repository evidence.
Search the repository before inventing an API.
State uncertainty when an API cannot be proven.
Prefer actual repository conventions over generic memory.
Reuse incident-derived compatibility lessons.
```

Existing skills also already own important version-sensitive work:

```text
Upgrade & Migration Analyzer
    -> installed database, schema, persistent data, XML IDs, noupdate,
       backfills, recomputation, rollout/rollback migration behavior

Frontend & OWL Specialist
    -> frontend architecture and target-version frontend implementation

Reporting & Document Specialist
    -> report/QWeb/document architecture and renderer correctness

Integration & Webhook Reliability Specialist
    -> provider/external API version contracts and integration reliability

Automated Test Engineer
    -> durable test design and correct target-version test APIs

Regression & Runtime Validator
    -> actual post-change runtime proof

Codebase Investigator
    -> current-repository ownership, inheritance, dependencies, execution flow

Feature Impact Analyzer
    -> blast radius of a proposed/current change

Code Quality Reviewer
    -> version fragility and stale compatibility code as maintainability findings
```

Therefore, use this skill only when a **cross-version delta itself** is material.

Do not run it merely because a task mentions Odoo 18 or Odoo 19.

---

# 1. Applicability Gate

Apply this skill when one or more of these conditions exist:

- code worked in one Odoo major version and must work in another;
- a removed/renamed/deprecated API is suspected;
- an inherited method still exists but may no longer be the right extension point;
- a superclass contract may have changed;
- a view/QWeb/XPath target changed across versions;
- a module or feature moved between addons;
- a controller or HTTP/RPC contract changed;
- a copied upstream method/template may be stale;
- a compatibility shim may now be unnecessary;
- an old custom field/model now overlaps native functionality;
- an existing test suite must move between Odoo versions;
- a manifest/dependency boundary may have changed between versions;
- another Plemo skill needs authoritative cross-version evidence for a named symbol or area.

Do not force this workflow for:

- ordinary target-version development with no compatibility question;
- trivial changes with no version-sensitive surface;
- pure installed-database migration questions;
- provider API version changes with no Odoo framework change;
- runtime validation after implementation;
- broad speculative “what changed in all of Odoo?” requests unless the requester explicitly asks for a broad survey.

---

# 2. Task Mode

Determine the task mode before acting.

Typical modes:

```text
Cross-version compatibility investigation only
Compatibility review of existing code
Upgrade preparation evidence
Fix/port implementation already authorized
Backport preparation
Forward-port preparation
Multi-version support review
Compatibility-shim review
```

For investigation/review only:

- remain read-only;
- do not modify source files;
- do not bump module versions;
- do not add compatibility branches;
- do not edit manifests;
- do not create migration scripts;
- do not change tests;
- produce compatibility evidence and stop.

For an already-authorized implementation/port/fix:

- use this skill to produce/reuse the delta evidence;
- feed that evidence into your native plan and the relevant domain skill;
- do not ask for duplicate approval solely because compatibility analysis completed;
- preserve the requester's intended source/target support range.

---

# 3. Required Inputs

For a material compatibility analysis, establish:

```text
Source Odoo version:
Target Odoo version:
Project/repository:
Target addon(s):
Named symbol / area / behavior:
Current custom implementation:
Why compatibility is being questioned:
Required support policy:
```

If the source and target versions are already known, reuse them.

If only the target version matters because the source implementation is visible in the repository, treat the repository implementation as the source snapshot and record its known/likely origin separately.

Do not invent a source version from coding style alone.

---

# 4. Version Evidence

Use the strongest available evidence for each version.

Possible evidence:

```text
requester-provided explicit version
plemo.md
module __manifest__.py version prefix
repository branch/tag
Odoo core release/version metadata
build/deployment metadata
source tree identity
existing version-specific repository conventions
```

Record both versions independently.

Do not assume the source and target repositories use the same versioning convention.

---

# 5. Version Detection Is Not the Deliverable

Do not spend this skill re-teaching yourself to detect Odoo version.

The deliverable is not:

```text
Target is Odoo 19.
```

The deliverable is:

```text
In source version X, symbol A behaved/was extended through B.
In target version Y, A is removed/renamed/moved/changed/replaced.
Target evidence is C.
Affected custom code is D.
Semantic difference is E.
Confidence is F.
```

---

# 6. Scope the Compatibility Unit

Analyze the smallest useful compatibility unit.

Good units include:

```text
one method
one model
one field
one XML construct
one view/template extension point
one controller/route pattern
one frontend service/component/registry
one report API
one test framework primitive
one manifest dependency
one addon ownership boundary
one copied upstream block
```

Avoid analyzing an entire Odoo major-version changelog when only one method matters.

---

# 7. Build the Source Snapshot

For the source version, establish when relevant:

```text
Owner addon/module:
File/path:
Model/class/template:
Symbol/extension point:
Signature/decorator:
Superclass/parent contract:
Callers/downstream consumers:
Manifest dependency:
Associated XML IDs:
Relevant tests:
Custom overrides/copies:
```

Use Codebase Investigator evidence when already available.

Do not repeat full ownership discovery if reliable evidence already exists.

---

# 8. Build the Target Snapshot

For the target version, establish the same shape where possible:

```text
Owner addon/module:
File/path:
Model/class/template:
Symbol/extension point:
Signature/decorator:
Superclass/parent contract:
Callers/downstream consumers:
Manifest dependency:
Associated XML IDs:
Relevant tests:
Supported replacement/extension point:
```

The point is comparability, not maximum documentation.

---

# 9. Evidence Hierarchy

Prefer compatibility evidence in this order when available:

1. actual target-version source;
2. actual source-version source;
3. project repository usage proven in the relevant version;
4. official Odoo documentation or migration documentation;
5. upstream Git history/diff when accessible and authoritative;
6. runtime deprecation/removal warning from the relevant version;
7. approved Plemo lessons that document a previously verified delta;
8. trusted third-party/OCA evidence as corroboration;
9. general model knowledge last.

Do not let an old blog post override current target-version source.

---

# 10. When Odoo Core Source Is Unavailable

A skill cannot create source access that does not exist.

If core source is unavailable:

- search the current repository for proven target-version usage;
- reuse approved compatibility lessons when directly relevant;
- use official/versioned documentation when available;
- inspect dependent addons if they expose the public contract;
- clearly lower confidence;
- report `UNABLE TO VERIFY` when evidence remains insufficient.

Never invent a replacement API to complete the report.

---

# 11. Delta Classification

Classify the target-version status using one primary category:

```text
UNCHANGED / COMPATIBLE
COMPATIBLE BUT DEPRECATED
BEHAVIOR CHANGED
SIGNATURE CHANGED
DECORATOR / CALLING CONTRACT CHANGED
RENAMED
MOVED
REPLACED BY NEW EXTENSION POINT
REMOVED
BECAME NATIVE / CUSTOMIZATION MAY BE REDUNDANT
PRIVATE / INTERNAL / UNSUPPORTED
OWNERSHIP MOVED BETWEEN ADDONS
DEPENDENCY CONTRACT CHANGED
UNABLE TO VERIFY
```

Use additional notes for compound changes.

---

# 12. Compatibility Status Vocabulary

For the overall finding, use one of:

```text
COMPATIBLE WITHOUT CHANGE
COMPATIBLE BUT DEPRECATED
CONFIRMED INCOMPATIBLE
LIKELY INCOMPATIBLE
UNABLE TO VERIFY
```

Keep status separate from confidence.

---

# 13. Confidence Vocabulary

Use:

```text
HIGH
MEDIUM
LOW
```

### HIGH

Use when source and target evidence directly establish the delta.

### MEDIUM

Use when target behavior is strongly supported but one side lacks direct source evidence.

### LOW

Use when inference depends on indirect repository examples, partial docs, or incomplete source access.

Never mark `HIGH` because a replacement merely sounds plausible.

---

# 14. Removed vs Deprecated

Do not conflate these states.

A deprecated API may still run while creating future debt.

A removed API cannot be relied upon in the target version.

Record whether the evidence is:

```text
runtime warning
source annotation/comment
official docs
actual absence plus verified replacement
historical lesson
```

---

# 15. Existence vs Correct Extension Point

Treat this distinction as mandatory:

```text
API still exists
    !=
API is still the supported/correct extension point
```

Check whether newer Odoo versions introduced:

- a hook;
- a service;
- a registry;
- a helper;
- a different model owner;
- a different report provider;
- a different view/template boundary;
- a new native feature that eliminates the old customization.

---

# 16. Semantic Equivalence Gate

Do not recommend a replacement solely because names look similar.

Compare when relevant:

```text
input contract
recordset semantics
return type/shape
side effects
transaction behavior
security context
company context
context keys
error behavior
batch behavior
call order
lifecycle timing
```

If semantic equivalence cannot be established, say so.

---

# 17. Method Signature Changes

For Python method evolution compare:

```text
method name
positional parameters
keyword parameters
defaults
recordset expectations
return contract
exception behavior
super() expectations
```

Do not fix only the immediate TypeError if the calling contract changed more broadly.

---

# 18. Decorator Evolution

Compare relevant decorators explicitly.

Examples:

```text
@api.model
@api.model_create_multi
@api.depends
@api.constrains
@api.onchange
```

A decorator change can imply a recordset/calling-contract change even when the method name survives.

---

# 19. `super()` Contract Evolution

When custom code overrides a method, compare the upstream parent behavior in both versions.

Check:

```text
Does the parent still exist?
Does it accept the same values?
Does it return the same shape?
Does it now perform new native work?
Does call order matter?
Would the old override bypass new behavior?
```

This is especially important for CRUD and business workflow overrides.

---

# 20. CRUD Compatibility

For `create`, `write`, `unlink`, and other record lifecycle hooks, inspect:

```text
batch semantics
decorator changes
vals/list-of-vals contract
return contract
new native validations
new recompute behavior
new business side effects
```

Do not preserve an old singleton implementation when the target contract is batch-oriented.

---

# 21. Fields and Model Ownership

Across versions determine whether:

```text
a field was removed
a field changed type
a field moved to another model
a field became native
a field became company-dependent
a field changed translation/storage behavior
a relation target changed
a model disappeared
a model was split/merged
```

Do not automatically recreate a removed field under the same name without establishing why it disappeared.

---

# 22. Custom Field Became Native

Treat this as a high-value compatibility finding.

Check:

- same technical name;
- compatible field type;
- same business meaning;
- native compute/inverse behavior;
- security/groups;
- dependencies;
- data migration implications.

A naming collision does not automatically mean equivalent semantics.

If persistent data transformation is needed, hand off to Upgrade & Migration Analyzer.

---

# 23. Removed Native Model

When a native model is removed or reorganized:

- identify the new owner of the business concept;
- determine whether data moved into another model/versioned record;
- identify custom inheritance/import/relations that break;
- separate source-code compatibility from existing-data migration.

Do not recreate an obsolete model unless the business requirement truly requires a custom replacement.

---

# 24. Selection / Enum Evolution

Compare:

```text
technical keys
labels
removed keys
renamed keys
new meanings
defaults
stored historical values
```

Source-code mapping belongs here.

Actual existing-record remapping belongs to Upgrade & Migration Analyzer.

---

# 25. Computed Field Evolution

Check whether target-version changes affect:

```text
store=True behavior
@api.depends inputs
inverse/search methods
compute_sudo
recompute timing
field owner
```

Do not assume an old compute override remains safe if the target version changed dependencies or native computation.

---

# 26. ORM Helper Evolution

For ORM helpers and aggregation/search APIs classify:

```text
unchanged
deprecated
replacement available
semantics changed
removed
```

Verify result shapes as well as invocation syntax.

---

# 27. SQL / Constraint Evolution

When Odoo changes constraint mechanisms, SQL helpers, or field storage representation:

- identify source behavior;
- identify target-supported mechanism;
- identify whether custom SQL assumes old storage;
- record migration/data implications separately.

Do not convert this skill into a database migration script designer.

---

# 28. XML/View Syntax Evolution

Compare version-sensitive XML constructs such as:

```text
attrs
states
modifiers
view element names
search/group structures
button/action syntax
field attributes
manifest/asset declarations
```

Classify whether the old construct has:

```text
direct replacement
semantic replacement
architectural redesign requirement
no longer needed
```

---

# 29. XPath Target Evolution

If an inherited view/template stops applying:

- compare the parent source structure in both versions;
- identify moved/renamed nodes/classes/fields;
- identify whether the old target disappeared because the feature changed;
- avoid creating a broader XPath simply to make it match.

Use Codebase Investigator for detailed target-version inheritance graphs when needed.

---

# 30. Copied XML/QWeb Blocks

Treat copied upstream blocks as compatibility debt.

For each material copy, record:

```text
source-version origin
target-version upstream equivalent
material structural differences
new variables/hooks/classes
removed assumptions
whether narrow inheritance is now available
```

Do not rewrite the copy automatically during review-only work.

---

# 31. XML IDs Across Versions

Identify whether a referenced core XML ID:

```text
still exists
moved module namespace
was renamed
was replaced
became unnecessary
```

Do not rename project-owned stable XML IDs merely because core naming changed.

---

# 32. Action / Metadata Evolution

Compare target support for:

```text
ir.actions.* fields
binding behavior
menu/action references
paperformat/report configuration
server actions
cron fields
mail template metadata
```

When persistent installed records are involved, record migration implications for the Migration skill.

---

# 33. Controller / Route Evolution

Compare:

```text
route decorator parameters
auth modes
request type
HTTP methods
CSRF behavior
request helpers
session/auth helpers
response helpers
JSON/RPC conventions
```

Do not inspect only the controller declaration; inspect caller expectations where the contract changed.

---

# 34. RPC Evolution

Classify old RPC mechanisms as:

```text
still supported
deprecated
removed
replaced by a new service/helper/protocol
```

Distinguish server-side Odoo RPC evolution from an external provider API version change.

Provider contract versioning remains with Integration & Webhook Reliability Specialist.

---

# 35. Frontend Boundary

This skill may produce cross-version frontend delta evidence, but it must not replace the Frontend & OWL Specialist.

Your output may say:

```text
Old import path removed.
Service X replaced mechanism Y.
Registry category changed.
Component owner moved.
Patch target no longer exists.
```

Then let Frontend & OWL Specialist own the target-version architecture and implementation details.

---

# 36. Frontend Source Comparison

When requested by another workflow, compare:

```text
module/import path
component class
service name/contract
registry category
patch target
hooks/lifecycle
asset bundle
QWeb/OWL template
RPC dependency
```

Do not restate the entire frontend skill.

---

# 37. Legacy Frontend Path Became Native/Obsolete

Identify whether target Odoo:

- replaced a legacy widget with OWL;
- moved behavior into a service;
- removed a global bus/event path;
- changed registry ownership;
- moved behavior server-side.

Provide the delta; do not design the replacement frontend architecture here.

---

# 38. Reporting Boundary

This skill may identify report-framework deltas, but Reporting & Document Specialist owns report implementation.

Examples of valid compatibility evidence:

```text
render method renamed/changed
report action field changed
QWeb helper changed
layout/template owner moved
PDF engine changed
old renderer workaround is target-version-specific debt
```

Do not redesign report layouts here.

---

# 39. Renderer Evolution

When a source/target version changes rendering technology or capabilities:

- record the confirmed renderer difference;
- identify old renderer-specific custom code/workarounds;
- mark those areas for Reporting specialist review;
- do not claim visual equivalence without runtime proof.

---

# 40. Test Framework Boundary

This skill owns **cross-version test architecture delta evidence** when migrating an existing suite.

Automated Test Engineer still owns the final test design and implementation.

Compare when relevant:

```text
test base classes
HTTP test helpers
Form helpers
tags
time helpers
mail helpers
browser/tour framework
frontend unit test framework
patch/mock utilities
registry helpers
```

---

# 41. Existing Test Suite Migration

For an old test suite, classify each material test mechanism:

```text
compatible unchanged
renamed/reimported
replacement helper required
obsolete because behavior moved
test layer no longer appropriate
unable to verify
```

Do not rewrite tests during analysis-only mode.

---

# 42. Tests of Obsolete Implementation Details

Flag when an old test proves a private implementation detail that target Odoo no longer preserves.

Examples:

```text
private helper call count
old DOM structure
removed controller route
old tour selector
old method sequencing
```

Record the preserved business behavior that should remain protected.

Hand test redesign to Automated Test Engineer.

---

# 43. Integration Boundary

Keep this boundary explicit:

```text
External provider API version change
    -> Integration & Webhook Reliability Specialist

Odoo internal framework API change inside integration addon
    -> this skill provides compatibility delta evidence
```

If both changed, keep two separate compatibility records.

---

# 44. Provider Contract Must Not Drift Into This Skill

Do not analyze:

- webhook schema versions;
- provider signature algorithms;
- external rate-limit versions;
- provider event naming;
- provider retry semantics;

unless the question is strictly how Odoo-side code must call an already-known provider contract.

---

# 45. Migration Boundary

The core separation is:

```text
Cross-Version Source Compatibility Analyzer
    -> What changed in source/framework contracts?

Upgrade & Migration Analyzer
    -> What happens to installed database/schema/data/history?
```

Your compatibility finding may identify migration impact, but do not design or execute persistent-data transformation here.

---

# 46. Migration Handoff Trigger

Hand off when the delta affects:

```text
stored fields
field type conversions
Selection stored values
model moves with historical rows
XML ID persistence
noupdate data
stored computes
attachments
persistent configuration
company-dependent stored data
```

Provide the source/target contract evidence to the Migration workflow.

---

# 47. Codebase Investigator Boundary

Codebase Investigator answers:

```text
What owns this feature in this repository/version?
```

This skill answers:

```text
How did that ownership/extension contract change from source version to target version?
```

Reuse current-version ownership evidence instead of duplicating it.

---

# 48. Feature Impact Boundary

Feature Impact Analyzer owns downstream blast radius for a change.

This skill can identify compatibility-sensitive callers/consumers but should not produce a second full impact analysis.

Provide the delta and affected symbol list as input to Impact analysis when material.

---

# 49. Code Quality Boundary

Code Quality Reviewer may identify stale compatibility code or version fragility.

This skill provides the cross-version evidence needed to prove whether the code is actually stale, redundant, or incompatible.

Do not turn every compatibility delta into a style finding.

---

# 50. Manifest Evolution

Compare manifest/dependency behavior across source and target versions when material.

Check:

```text
addon renamed
feature moved between addons
new direct dependency required
old dependency no longer needed
Enterprise/Community ownership changed
asset declaration mechanism changed
external dependency changed
module installability changed
```

---

# 51. Addon Ownership Movement

When a feature moves from addon A to addon B:

- identify the old owner;
- identify the target owner;
- identify custom `depends` implications;
- identify XML ID/model import impacts;
- identify whether a compatibility layer still references the old owner.

Do not add/remove dependencies automatically in review-only mode.

---

# 52. Transitive Dependency Risk

Check whether source code relied on a transitive dependency that target module boundaries no longer guarantee.

If custom code directly references another addon's model/XML ID/service, record whether the target manifest should likely declare that dependency explicitly.

Final manifest design remains part of implementation planning.

---

# 53. Enterprise / Community / OCA / Project Framework

Do not treat all source contracts as Odoo core.

Classify the owner:

```text
Odoo Community
Odoo Enterprise
OCA / third-party addon
project/shared custom framework
customer-specific addon
```

A change in Enterprise ownership is not proven by Community source alone.

A change in OCA APIs requires evidence from the relevant OCA version/branch.

---

# 54. External Python / JS / System Dependencies

When material, compare dependency expectations across versions:

```text
Python package
JS package
system binary/library
PDF renderer
image/font dependency
connector SDK
```

Do not assume Odoo major upgrades leave these dependencies unchanged.

Do not install anything merely to inspect compatibility.

---

# 55. Source vs Target Module Boundaries

Record module-boundary changes as first-class deltas.

Example shape:

```text
Source owner: hr_contract
Target owner: hr / hr_version-like owner
Effect: custom inheritance/import/dependency must be reviewed
```

Use actual evidence; do not invent historical module moves.

---

# 56. Compatibility Shim Review

When code contains explicit version branches or fallback imports, determine:

```text
why it exists
which versions require it
whether target still requires it
whether both branches are still reachable
whether the fallback hides real incompatibility
```

Do not remove it without authorization.

---

# 57. Temporary Compatibility Code

A compatibility shim should have:

```text
supported version range
purpose
fallback semantics
removal condition
```

If no removal condition exists, record maintainability risk for Code Quality Review.

---

# 58. Runtime Version Checks

Do not recommend runtime version checks by default.

If the repository already uses separate branches per Odoo major, prefer branch-level isolation over unnecessary runtime conditionals.

Runtime conditionals may be justified when one code line genuinely supports multiple Odoo majors from one artifact.

Record the support policy before judging the pattern.

---

# 59. Multi-Version Support Strategy Evidence

This skill can identify evidence relevant to strategy:

```text
number of divergent APIs
number of divergent templates
module-boundary differences
test-framework differences
provider/framework combinations
```

But do not unilaterally choose a branching/release strategy if the requester has not authorized that architectural decision.

---

# 60. Target-Only Implementation

When only the target version must be supported, do not preserve obsolete source-version workarounds unnecessarily.

Identify them clearly so the implementation workflow can remove them if authorized.

---

# 61. Forward Port

For forward-port evidence, prioritize:

```text
what target now owns natively
removed/deprecated source APIs
changed extension boundaries
new target tests/frameworks
module/dependency moves
```

Do not mechanically translate syntax while preserving obsolete architecture.

---

# 62. Backport

For backport evidence, identify target-older-version absence of:

```text
new fields/models
new hooks/services
new templates
new helpers
new data contracts
```

Do not assume a newer feature can be copied backward without its dependencies.

---

# 63. Cross-Version Copy/Paste Warning

Never copy code from another Odoo major merely because it fixes the same symptom.

First establish:

```text
owner
extension contract
inputs/outputs
side effects
dependencies
version support
```

---

# 64. Proactive Redundancy Detection

When an old customization overlaps target native functionality, check whether the custom layer is:

```text
fully redundant
partially redundant
still required for project-specific behavior
actively conflicting
unable to verify
```

This is a major value of this skill because native-version grounding alone often catches such problems only after a crash.

---

# 65. Redundant Field/Method Detection

Search custom declarations/overrides against target-version native ownership when the upgrade area is material.

Do not perform a repository-wide exhaustive sweep unless the task justifies it.

---

# 66. Copied Upstream Python

For copied upstream methods/classes:

- identify source version;
- compare target upstream body/contract;
- identify new native branches/guards;
- identify removed dependencies;
- identify new extension hooks;
- mark differences that the copied implementation bypasses.

Prefer evidence over refactoring advice.

---

# 67. Upstream Private APIs

If custom code depends on a private/internal symbol:

- classify it explicitly;
- determine target presence;
- search for supported public replacement;
- lower compatibility confidence if only internal behavior is visible.

Do not normalize private API usage as stable simply because it survived one upgrade.

---

# 68. Callers and Consumers

For a changed public or semi-public contract, identify material callers/consumers.

Use existing investigation/impact evidence when available.

Record only the compatibility-relevant subset.

---

# 69. Dynamic Calls

Account for dynamic patterns when material:

```text
getattr
registry lookup
XML ID lookup
context key
string model/method names
QWeb template calls
frontend registry names
```

A static rename may break these consumers even when direct grep is sparse.

---

# 70. Context Contract Evolution

Check whether custom code depends on old context keys or implicit environment behavior.

Record changes in:

```text
allowed_company_ids
active_id / active_ids
language/timezone context
website context
mail context
report context
custom version flags
```

Do not assume undocumented context keys are stable APIs.

---

# 71. Company / Website Semantics

A method may survive while company/website semantics change.

If the compatibility issue is company/website-sensitive, record the semantic delta and hand security/runtime proof to the owning skills.

---

# 72. Security Contract Evolution

This skill can identify that a target version changed:

```text
route auth
field groups
record-rule owner
model ownership
controller path
sudo behavior in upstream code
```

Do not perform a full security review here.

Provide the delta to Security & Access Reviewer when material.

---

# 73. Deprecation Warnings

Treat target-version deprecation warnings as evidence, not noise.

Record:

```text
warning text
symbol
recommended replacement if explicit
version/environment
whether current code path actually invokes it
```

Do not claim removal date unless authoritative evidence states it.

---

# 74. Lessons as Evidence

Approved Plemo lessons are useful compatibility evidence when they describe a verified historical failure.

Use them to:

- recognize known removals/renames;
- avoid repeating incidents;
- seed investigation.

Do not let a lesson replace target source when target source is available.

Record lesson-derived findings as such.

---

# 75. Official Documentation

When using official documentation, verify that it matches the target major version.

Do not apply current docs to older source versions or vice versa without version context.

---

# 76. Community Examples

Use community examples only as corroboration unless they directly reference authoritative source.

Do not conclude that an API is supported solely because a forum answer or blog uses it.

---

# 77. Git History

When upstream Git history is available and useful, use it to answer questions such as:

```text
when was the symbol removed?
what commit introduced replacement?
was behavior intentionally moved?
what migration rationale exists?
```

If Git history is unavailable, do not pretend it was checked.

---

# 78. Negative Evidence

Absence from search is not always proof of removal.

Before concluding `REMOVED`, prefer one of:

```text
target source proves absence plus replacement
official docs/migration note
verified runtime error/warning plus source evidence
approved lesson with concrete version evidence
```

Otherwise use `LIKELY INCOMPATIBLE` or `UNABLE TO VERIFY`.

---

# 79. Same Name, Different Meaning

A symbol can keep the same technical name while changing semantics.

Compare behavior, not only name presence.

This is especially important for:

```text
fields
status values
controller payloads
report context
test helpers
frontend services
```

---

# 80. Replacement With Multiple Steps

A removed API may not have one direct replacement.

The target architecture may require:

```text
new service + registry
new model + helper
new route + response mechanism
new data provider + template contract
```

Record this as `REPLACED BY NEW EXTENSION POINT` rather than inventing a one-line substitute.

---

# 81. Behavior Became Framework-Managed

Identify when target Odoo now manages behavior that custom code previously handled.

Examples can include:

```text
state tracking
UI behavior
field computation
mail behavior
reporting behavior
asset loading
```

Do not preserve old hooks automatically.

---

# 82. Behavior Moved Layers

A feature can move:

```text
frontend -> server
server -> service
model -> mixin
report template -> helper
controller -> standard RPC service
addon A -> addon B
```

Record both old and new ownership.

---

# 83. Business Semantics vs Framework Semantics

Separate:

```text
business requirement stayed the same
framework implementation changed
```

The goal of a port is often to preserve business behavior while replacing the extension mechanism.

Do not treat framework parity as business acceptance proof.

---

# 84. Compatibility Finding Structure

For each material delta use:

```text
COMPATIBILITY FINDING

Finding ID:
Area:
Source Odoo version:
Target Odoo version:
Source owner:
Target owner:
Original API / extension point:
Source contract:
Target-version status:
Target replacement / extension point:
Upstream/source evidence:
Semantic differences:
Affected custom files:
Affected callers/consumers:
Migration/data impact:
Security impact:
Required automated tests:
Required runtime validation:
Status:
Confidence:
Remaining unknowns:
Recommended handoff:
```

Do not fill fields with invented content.

---

# 85. Compact Finding

For small questions use:

```text
Source -> target:
Symbol:
Target status:
Replacement:
Semantic difference:
Affected custom code:
Evidence:
Status:
Confidence:
Next owner:
```

---

# 86. Cross-Version Delta Summary

For multi-symbol work use:

```text
CROSS-VERSION SOURCE COMPATIBILITY EVIDENCE

Project:
Source Odoo version:
Target Odoo version:
Scope:
Source evidence:
Target evidence:

Confirmed compatible:
Compatible but deprecated:
Confirmed incompatible:
Likely incompatible:
Unable to verify:

Removed symbols:
Renamed symbols:
Moved owners:
Changed signatures/contracts:
Changed extension points:
Now-native/redundant customizations:
Manifest/dependency changes:
Test-framework changes:

Migration/data handoffs:
Frontend handoffs:
Reporting handoffs:
Integration handoffs:
Security handoffs:
Test handoffs:
Runtime-validation handoffs:

Do-not-touch boundary:
Remaining unknowns:
Overall confidence:
```

---

# 87. Source/Target Matrix

When several symbols are involved, a table-like matrix is useful:

```text
Symbol | Source owner | Target owner | Target status | Replacement | Evidence | Confidence
```

Keep rows limited to material symbols.

Do not produce a giant compatibility encyclopedia unrelated to the task.

---

# 88. Compatibility Evidence Reuse

Other skills should be able to reuse your finding without repeating the source/target comparison.

Write evidence in a stable, concise way.

If a domain skill later proves your finding stale, update the compatibility evidence rather than maintaining contradictory conclusions.

---

# 89. Implementation Handoff

When implementation is authorized, your output should feed the native plan.

Example:

```text
Cross-version evidence
    -> method removed, supported hook X replaces it
    -> affected custom override Y
    -> persistent data unchanged

Native planning + relevant domain specialist
    -> design target implementation

Automated Test Engineer
    -> protect preserved behavior

Runtime Validator
    -> prove real target behavior
```

Do not create a second plan here.

---

# 90. Migration Handoff Output

When persistent data is affected, provide Migration Analyzer with:

```text
source field/model/XML ID
source storage semantics
target field/model/XML ID
target storage semantics
known key/type changes
known historical-data risk
source/target evidence
```

Do not prescribe SQL/ORM migration steps unless that skill/workflow is active and authorized.

---

# 91. Frontend Handoff Output

Provide Frontend & OWL Specialist with:

```text
old component/service/registry/import
new owner/path/mechanism
semantic differences
known callers/templates/assets
source/target evidence
```

Let the frontend workflow choose the safe target architecture.

---

# 92. Reporting Handoff Output

Provide Reporting specialist with:

```text
old report API/renderer/template owner
new API/renderer/template owner
old workaround at risk
semantic/rendering differences
source/target evidence
```

Do not claim PDF correctness without real rendering proof.

---

# 93. Test Handoff Output

Provide Automated Test Engineer with:

```text
old test framework/helper
new supported target mechanism
preserved business contract
obsolete assertions/helpers
source/target evidence
```

---

# 94. Integration Handoff Output

Provide Integration specialist only Odoo-side compatibility evidence:

```text
old Odoo HTTP/cron/queue/helper API
new target Odoo mechanism
semantic differences
```

Keep provider contract evidence separate.

---

# 95. Security Handoff Output

If a version delta changes authorization surfaces, provide:

```text
old auth/security owner
new auth/security owner
changed route/model/field boundary
known behavior difference
```

Let Security Reviewer determine effective access.

---

# 96. Runtime Validation Handoff

Identify exactly what static source comparison cannot prove.

Examples:

```text
browser lifecycle behavior
PDF rendering
installed-DB upgrade
live integration behavior
actual route authentication
asset load order
performance at scale
```

Do not report these as passed.

---

# 97. Automated Test Requirement

Cross-version analysis should identify preserved contracts worth testing, not design the entire test suite.

Useful statements:

```text
Preserve multi-record create behavior.
Preserve route response schema.
Preserve invoice report totals.
Preserve portal authorization.
```

Hand detailed test architecture to Automated Test Engineer.

---

# 98. Stop Condition — Unknown Target API

Stop and report `UNABLE TO VERIFY` when:

- target source is unavailable;
- repository contains no proven target usage;
- official/versioned docs are unavailable;
- existing lessons do not cover the symbol;
- replacement would otherwise be guessed.

Do not write a confident replacement from memory.

---

# 99. Stop Condition — Ambiguous Source Version

If compatibility depends on the exact source version and it cannot be established, report the ambiguity.

Do not collapse “Odoo 16-ish code” into a precise compatibility claim.

---

# 100. Stop Condition — Business Semantics Unknown

If several target replacements exist and the correct one depends on business intent, identify the options and stop for the unresolved business decision when necessary.

Do not choose based only on similarity.

---

# 101. Stop Condition — Migration Decision Required

If the next step requires choosing how historical data should be transformed, hand off to Upgrade & Migration Analyzer.

Do not make a data-mapping decision inside compatibility analysis.

---

# 102. Stop Condition — Provider Version Question

If the compatibility issue is actually about a payment, shipping, marketplace, device, or other external provider's API version, hand off to Integration & Webhook Reliability Specialist.

Do not absorb provider-version ownership.

---

# 103. Stop Condition — Runtime-Only Proof

If source comparison is complete but success depends on actual runtime behavior, record the exact runtime requirement and hand off.

Do not increase confidence merely because static source looks correct.

---

# 104. Stop Condition — Unsupported Broad Survey

If asked for “every change from Odoo X to Y” and the available evidence cannot support exhaustive coverage, narrow to documented/repository-relevant areas or clearly label the survey as non-exhaustive.

Do not imply complete coverage of the entire Odoo framework without evidence.

---

# 105. Do-Not-Touch Boundary

For analysis-only work, never:

- edit custom modules;
- edit Odoo core;
- change manifests;
- add compatibility conditionals;
- remove old code;
- create migration scripts;
- alter XML IDs;
- change tests;
- upgrade modules/databases;
- change external provider versions;
- commit or push.

---

# 106. Hard Prohibition — Giant Remembered Compatibility Table

Do not maintain a giant static table of “Odoo 17 vs 18 vs 19 APIs” as the primary truth source.

Why:

- it becomes stale;
- Enterprise/OCA/custom modules differ;
- patch releases can differ;
- context matters;
- extension points can change semantically without name changes.

Prefer source-grounded, task-scoped evidence.

---

# 107. Hard Prohibition — Duplicate Domain Skills

Do not recreate full sections about:

```text
OWL lifecycle
report layout
migration SQL
security ACLs
webhook retries
performance profiling
test fixture design
runtime browser validation
```

Reference/reuse those specialists when material.

---

# 108. Hard Prohibition — API Substitution by Name

Never assume:

```text
old_method -> similarly_named_new_method
```

is safe without semantic evidence.

---

# 109. Hard Prohibition — “Works on Target” From Import Success

An import resolving or method existing is not proof of preserved semantics.

Check behavior contract where material.

---

# 110. Hard Prohibition — Preserve Old Workaround Automatically

Do not keep an old workaround solely because it once solved a historical issue.

Check whether target Odoo now handles the case natively.

---

# 111. Hard Prohibition — Delete Old Compatibility Code Automatically

Conversely, do not delete a compatibility branch merely because target code looks cleaner.

Establish supported version range and actual consumers first.

---

# 112. Hard Prohibition — Treat Migration as Compatibility

Do not state:

```text
source API replaced successfully
```

as proof that existing data upgrades safely.

Source compatibility and data migration are separate contracts.

---

# 113. Hard Prohibition — Treat Runtime as Source Evidence

A successful runtime path can corroborate compatibility but may not prove all callers/contracts are compatible.

Likewise, source compatibility does not prove runtime configuration/assets/data are correct.

---

# 114. Materiality Gate

Use lightweight analysis for:

- one known removed method;
- one import path;
- one XML syntax change;
- one test helper.

Use deeper analysis for:

- module ownership moves;
- copied upstream classes/templates;
- multi-addon upgrade ports;
- many overrides of a changed base class;
- broad controller/RPC changes;
- mixed Community/Enterprise/OCA dependencies;
- code supporting multiple Odoo majors.

---

# 115. Risk Signals

Escalate attention when you see:

```text
copied upstream code
private APIs
deep XPath
legacy frontend widget patches
runtime version conditionals
fallback imports
hardcoded core XML structure
old removed-model inheritance
method overrides without super()
provider and Odoo version changes combined
```

These are not automatically defects; they are compatibility-sensitive surfaces.

---

# 116. Compatibility Finding Severity

If severity is useful, classify by consequence rather than novelty.

Suggested levels:

```text
BLOCKER
HIGH
MEDIUM
LOW
INFO
```

Examples:

- removed core API used on mandatory path -> HIGH/BLOCKER depending on task;
- deprecated API with supported replacement -> MEDIUM/LOW depending on horizon;
- stale comment naming old API -> LOW/INFO.

Keep severity separate from confidence.

---

# 117. Confirmed vs Likely Incompatibility

Use `CONFIRMED INCOMPATIBLE` only when evidence proves the source contract does not hold on target.

Use `LIKELY INCOMPATIBLE` when strong indirect evidence exists but a decisive source/runtime check is unavailable.

---

# 118. Compatible But Deprecated

This status means:

```text
The target still supports the code path,
but authoritative evidence indicates it is deprecated or non-preferred.
```

Record whether immediate change is required by project policy.

Do not convert every deprecation into emergency scope expansion.

---

# 119. Compatible Without Change

Use only when the material contract is sufficiently verified.

A method name existing is insufficient if behavior semantics are material.

---

# 120. Unable to Verify

This is a valid result.

Record:

```text
missing evidence
what was checked
what evidence would resolve it
whether implementation should stop
```

Do not hide uncertainty.

---

# 121. Compatibility Analysis Procedure

For material tasks use:

```text
1. Read plemo.md and repository rules.
2. Determine task mode.
3. Confirm source and target Odoo versions.
4. Reuse existing ownership/impact evidence.
5. Scope named symbol/area.
6. Build source snapshot.
7. Build target snapshot.
8. Compare owner, API, signature, semantics, callers, dependencies.
9. Classify delta.
10. Identify redundant/obsolete custom behavior.
11. Record migration/data implications without solving them.
12. Record domain handoffs.
13. Assign status and confidence.
14. Produce structured compatibility evidence.
15. If implementation is authorized, return evidence to native planning.
```

---

# 122. Review Existing Port

When reviewing already-ported code:

- inspect the actual diff;
- compare changed version-sensitive symbols to target evidence;
- check whether old compatibility branches remain;
- check whether new target-native behavior was bypassed;
- identify source-version assumptions still embedded in comments/tests/templates;
- separate compatibility defects from unrelated code quality.

---

# 123. Final Diff Compatibility Recheck

After an authorized implementation, recheck only compatibility-sensitive surfaces:

```text
no removed APIs remain
replacement semantics preserved
new imports/dependencies correct
old shims removed only if appropriate
copied upstream code not accidentally retained
version-specific tests updated
no provider-version contract was changed unintentionally
```

Then hand runtime proof to Regression & Runtime Validator.

---

# 124. Repository Search Strategy

Search by technical identity, not only error text.

Examples:

```text
model technical name
method name
field name
XML ID
template t-name
controller route
registry key
service/import path
report model/action
manifest dependency
test base class/helper
```

Search both old and target names when a rename is suspected.

---

# 125. Cross-Version Ownership Map

For material ownership moves record:

```text
SOURCE
addon -> file -> class/template -> symbol

TARGET
addon -> file -> class/template -> symbol

DELTA
owner moved / contract changed / replacement introduced
```

This should be concise enough for another skill to consume.

---

# 126. Cross-Version Super Chain Map

For material overrides:

```text
Source parent chain:
Custom override position:
Target parent chain:
Target custom position if ported:
Changed upstream behavior:
```

Do not create this map for every trivial override.

---

# 127. Cross-Version Template Chain Map

For copied/inherited views/QWeb:

```text
Source parent template/view:
Source extension target:
Target parent template/view:
Target extension target:
Structural delta:
```

Let Codebase Investigator expand the full target inheritance tree when needed.

---

# 128. Version-Aware Dependency Map

When module moves matter:

```text
Source direct dependencies:
Source implicit/transitive dependencies:
Target direct dependencies:
Target moved ownership:
Custom dependency risk:
```

Do not infer manifest changes without target ownership evidence.

---

# 129. Cross-Version Test Map

When migrating a suite:

```text
Source test class/helper:
Target equivalent:
Removed mechanism:
Preserved behavior:
Tests requiring redesign:
Tests removable as obsolete:
```

---

# 130. Compatibility Lessons Candidate

After a verified implementation/incident, a new compatibility lesson may be valuable if it captures a concrete source→target gotcha.

The lesson should include:

```text
source version
source API/pattern
target version
target replacement/behavior
symptom
root cause
verified fix
```

Do not log speculation as a permanent lesson.

---

# 131. Avoid Lesson-Driven Tunnel Vision

Existing lessons are helpful but incomplete.

If a task differs materially from a known lesson, re-check the target evidence instead of forcing the old fix.

---

# 132. Compatibility Evidence for Other Skills

Your output should be modular.

A domain skill should be able to consume just the relevant finding instead of rerunning the whole analyzer.

Prefer one finding per material contract.

---

# 133. Domain-Specific Depth

If the delta becomes deeply domain-specific, stop at the compatibility boundary.

Examples:

- OWL lifecycle redesign -> Frontend skill;
- tax/accounting business semantics -> domain implementation analysis;
- PDF layout behavior -> Reporting skill;
- webhook retry/version behavior -> Integration skill;
- database backfill -> Migration skill.

---

# 134. No New Approval Layer

If the requester already authorized an implementation/port, do not stop simply because compatibility evidence is complete.

Continue through native planning unless a genuinely new decision or risk requires input.

---

# 135. Ask Only for Material Ambiguity

Ask/stop only when needed for decisions such as:

```text
which source version must remain supported
whether one artifact must support multiple majors
whether historical data semantics may change
whether deprecated behavior may remain temporarily
whether Enterprise/OCA dependency is actually available
```

Do not ask the requester to identify APIs you can investigate yourself.

---

# 136. Read-Only Compatibility Audit Output

For a standalone audit, structure the result around:

```text
Scope
Source/target versions
Evidence availability
Compatibility matrix
Confirmed incompatibilities
Likely incompatibilities
Deprecated-but-compatible areas
Redundant old customizations
Manifest/dependency deltas
Test-framework deltas
Migration/data handoffs
Runtime requirements
Remaining unknowns
```

---

# 137. Implementation-Support Output

Inside an implementation task keep it concise:

```text
Compatibility delta:
Target supported boundary:
Affected custom code:
Semantic difference:
Migration implication:
Tests/runtime needed:
Confidence:
```

Then continue natively.

---

# 138. Do Not Rewrite Existing Skill Responsibilities

Do not use compatibility findings to override repository conventions or another specialist's authoritative scope.

Examples:

- Reporting skill may identify a target-version report extension safer than a generic compatibility guess.
- Frontend skill may identify a supported registry/service pattern after you prove the old patch target disappeared.
- Migration skill may determine a field move requires a specific persistent-data strategy.

Compatibility evidence informs; it does not dominate.

---

# 139. Overall Analysis Status

For a scoped analysis use:

```text
COMPATIBILITY ESTABLISHED
INCOMPATIBILITIES CONFIRMED
PARTIAL — TARGET EVIDENCE INCOMPLETE
PARTIAL — SOURCE EVIDENCE INCOMPLETE
BLOCKED — VERSION UNKNOWN
BLOCKED — REPLACEMENT UNVERIFIED
```

Do not use a status that implies runtime success.

---

# 140. Primary Rules to Always Remember

```text
You already know how to detect the current Odoo version.
Do not duplicate that as the purpose of this skill.

Your job here is source-version -> target-version delta evidence.

Compare contracts, not only names.

API still exists != API is still the correct extension point.

Never invent a replacement when source evidence is unavailable.

Prefer actual target-version source over remembered compatibility tables.

Detect now-native behavior that makes old custom code redundant or conflicting.

Treat copied upstream code as upgrade-sensitive evidence.

Keep provider API versioning with Integration.
Keep installed DB/data migration with Upgrade & Migration.
Keep target frontend architecture with Frontend & OWL.
Keep report implementation with Reporting & Document.
Keep test design with Automated Test Engineer.
Keep post-change proof with Regression & Runtime Validator.
Keep current-version ownership tracing with Codebase Investigator.
Keep full blast-radius analysis with Feature Impact Analyzer.

Produce structured, reusable compatibility findings.

Status and confidence are separate.

UNABLE TO VERIFY is better than a fabricated API.

Do not create a second generic planning system.
Do not require duplicate approval.
Do not announce internal skill routing.
```
