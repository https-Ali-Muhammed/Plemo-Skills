# Odoo Upgrade/migration analysis

## Purpose

Use this guidance to analyze how an Odoo code or data-model change affects an already installed database and whether migration, backfill, recomputation, compatibility handling, rollout safeguards, or rollback preparation is required.

This is Odoo-specific **upgrade and migration evidence guidance**. It does not replace Plemo's native repository discovery, planning, implementation, deployment, debugging, or general validation workflow.

Core question:

```text
What happens when this change meets an existing database and real historical data?
```

Core principles:

- Treat existing production data as part of the compatibility contract.
- Separate fresh installation from upgrade of an installed database.
- Separate code compatibility from schema/data compatibility.
- Reuse existing Plemo investigation and impact evidence.
- Use the smallest migration surface justified by evidence.
- Do not create migration work for changes that do not require it.
- Do not claim fresh-install success proves upgrade safety.
- Do not claim upgrade success proves migrated data is correct.
- Do not claim rollback is possible unless the data transformation is reversible.
- Preserve Plemo's native task mode and authorization behavior.

## 0. Native Agent Compatibility and Scope

This guidance extends Plemo; it does not override it.

Repository-specific instructions such as `plemo.md`, customer rules, migration conventions, deployment conventions, module-versioning rules, and available Plemo tools take precedence over generic examples in this guidance.

Reuse reliable evidence already collected from:

- existing codebase-investigation evidence
- existing feature-impact evidence
- security and access analysis
- broader regression and runtime validation
- current Git diff
- existing migration scripts
- runtime/build logs
- database inspection

Do not repeat a full investigation when existing evidence is sufficient.

### 0.1 Relationship With Native Workflow and Existing Evidence

```text
Existing codebase investigation
    "What exists?"
        ↓
feature-impact analysis
    "What could this change affect?"
        ↓
Upgrade/migration analysis
    "What happens to installed databases and existing data?"
        ↓
Relevant security/access analysis
    "Is the security boundary safe?" when relevant
        ↓
Plemo native planning
        ↓
Plemo native implementation
        ↓
Broader regression/runtime validation
    "What can we prove works after implementation?"
```

This guidance provides migration evidence to Plemo's native planner and exact upgrade scenarios to broader regression and runtime validation after implementation.

Do not create a second generic implementation or deployment plan.

### 0.2 Task Mode and Authorization

Determine whether the current task is:

- upgrade/migration analysis only
- migration review
- fix migration issue
- implement schema/data change
- validate upgrade behavior

For analysis/review only:

- remain read-only;
- do not create migration scripts;
- do not alter schema/data;
- do not run destructive operations.

For an already-authorized fix or implementation:

- the original request is sufficient authorization for the requested scope;
- do not ask for a second approval merely because migration analysis finished;
- continue into Plemo's native planning process using the migration evidence;
- request new resolution only for destructive/irreversible data operations, material scope expansion, production-only transformations, migration strategy decisions, or rollback limitations that require a user/business choice.

### 0.3 Migration Materiality Gate

Usually no migration is required for:

- comments/documentation;
- translation-only changes;
- non-persistent frontend-only behavior;
- CSS-only changes;
- non-stored logic where no persisted semantics must be corrected.

Migration/upgrade analysis is required or strongly preferred for:

- new stored fields;
- stored compute changes;
- field/model rename;
- field type changes;
- required/default changes;
- Selection key rename/removal;
- relational target changes;
- `company_id` or company semantics changes;
- XML ID changes;
- `noupdate` data changes;
- security/configuration data changes;
- data split/merge/backfill;
- fields/models moved between modules;
- dependency/load-order changes;
- removed fields/models/data;
- large recomputations;
- existing legacy data;
- major Odoo version upgrades.

Validation depth follows data risk, not line count.

## 1. Determine the Upgrade Target

Record:

```text
Project:
Requested change:
Current Odoo version:
Target Odoo version:
Current module version:
Target module version:
Installed module state:
Production-like data available:
```

Do not treat a same-major module upgrade as a major-version migration unless evidence supports it.

## 2. Detect Odoo and Module Versions

Use the strongest available evidence:

- module `__manifest__.py` version prefix;
- `odoo/release.py`;
- `odoo/version.py`;
- repository branch;
- `plemo.md`;
- build/deployment configuration.

A manifest prefix such as `17.0.x`, `18.0.x`, or `19.0.x` is acceptable evidence when core source is unavailable.

Record whether a version bump is required by the actual project workflow. Do not assume all Odoo.sh or Plemo deployments behave identically.

## 3. Repository and Tool Precedence

Use:

1. `plemo.md` and project rules;
2. Plemo-native repository/Odoo tools;
3. project-specific migration helpers;
4. migration code already in the repository;
5. generic read-only shell/database tools where appropriate.

Potential tools may include `find_addon`, `search_code`, `read_file`, `odoo_local`, `read_odoo_log`, `get_build_log`, `upgrade_module`, or equivalents.

Never invent unavailable tools.

Do not dispatch merely because local analysis is incomplete. Dispatch only when the project workflow permits it and the migration requirement materially justifies it.

## 4. Reuse Impact Evidence

If `IMPACT EVIDENCE` already exists, reuse:

- affected modules/models/fields;
- reverse dependencies;
- stored-compute risks;
- Selection risks;
- upgrade/data risks;
- security surface;
- runtime requirements;
- do-not-touch boundary.

Refresh only what is stale or materially changed.

## 5. Inspect the Actual Change Set

Inspect the actual diff and relevant repository state.

Record:

```text
Schema changes:
Stored semantic changes:
XML/data changes:
Dependency changes:
Model ownership changes:
Migration files:
Potential destructive operations:
```

Analyze what actually changed, not only what the plan expected.

## 6. Build the Migration Surface

Classify each material change as one or more of:

```text
Code-only
Schema
Data
Configuration
Security data
XML ID / metadata
Stored compute
Selection
Relational
Company semantics
Website semantics
Integration contract
Major-version compatibility
```

## 7. New Stored Field Analysis

For each new stored field record:

```text
Model:
Field:
Type:
required:
default:
compute:
related:
company-dependent:
index:
existing-record behavior:
```

Check:

- what existing records receive;
- whether `required=True` breaks historical rows;
- whether the default is business-correct;
- whether backfill is needed;
- whether stored recomputation is required;
- whether reports/constraints depend on it.

## 8. Field Rename Analysis

A Python rename is not automatically a safe database rename.

Search the old and new field names across:

- Python;
- XML/views;
- domains;
- record rules;
- reports/exports;
- integrations;
- server actions;
- database column/data.

Determine whether the safe path is:

```text
column rename
data copy
compatibility alias
temporary dual-read
migration script
later cleanup
```

Do not silently abandon the old populated column.

## 9. Field Type Change Analysis

For type changes, determine:

- PostgreSQL conversion safety;
- invalid historical values;
- relation conversion needs;
- index/constraint effects;
- downstream caller/report/integration expectations.

Examples include Char→Selection, Integer→Float, Many2one→Many2many, Date→Datetime, and precision changes.

Require explicit conversion when automatic ORM/schema behavior is unsafe.

## 10. Required and Default Changes

For `required=False -> True`, inspect existing null/empty records before upgrade.

Do not invent a fake business value solely to satisfy the schema.

For default changes, explicitly decide whether the requirement is:

```text
future records only
historical backfill
conditional backfill
no historical change
```

A new code default does not automatically migrate existing rows.

## 11. Relational Changes

For Many2one/One2many/Many2many changes inspect:

- comodel/inverse changes;
- relation tables;
- foreign keys;
- orphan risk;
- `ondelete`;
- `check_company`;
- company consistency;
- downstream reports/integrations.

## 12. Model Rename, Move, or Removal

For model rename/move inspect:

- `_name` / `_inherit`;
- `ir.model`;
- `ir.model.fields`;
- ACLs/rules;
- XML IDs;
- mail references;
- server actions/cron;
- properties;
- attachments;
- reports/integrations.

For removals, search all consumers and classify:

```text
safe cleanup
still referenced
requires archive
requires migration
requires compatibility period
```

Do not drop historical data merely because current code no longer reads it.

## 13. Stored Compute Migration

For `store=True` compute changes record:

```text
Old logic:
New logic:
@api.depends:
@api.depends_context:
inverse:
downstream stored fields:
record volume:
```

Analyze the chain:

```text
module upgrade
→ recompute
→ dependent recomputes
→ inverse/write effects
→ constraints
→ automation/tracking
→ locks/performance
```

A recompute may be technically successful but business-wrong if historical values should remain frozen.

## 14. Historical Snapshot Protection

Explicitly check whether values such as historical tax, price, currency, salary, approval, or accounting snapshots should remain unchanged.

Do not rewrite history just because a new compute formula exists.

## 15. Large Recompute Risk

Record:

```text
Estimated rows:
Dependent rows:
Batchability:
Lock risk:
Timeout risk:
Memory risk:
Upgrade duration risk:
```

If volume is unknown, report it as unknown.

Do not invent performance numbers.

## 16. Selection Migration

For Selection additions inspect domains, reports, state machines, rules, integrations, and defaults.

For renamed/removed keys search:

- database values;
- Python comparisons;
- XML modifiers/domains;
- filters;
- record rules;
- JS;
- reports/exports;
- server actions/cron;
- integrations.

Create a deterministic mapping:

```text
old_key -> new_key
```

If multiple old states collapse into one new state, document the loss of semantic detail.

## 17. Legacy Selection Values

Determine what happens to records containing obsolete values.

Possible handling:

```text
map
archive
manual review
temporary compatibility
reject
```

Do not let upgrade failure be the first time legacy values are discovered.

## 18. XML ID Stability

Inspect `record id=`, external references, and `ir.model.data`.

Changing an XML ID may create a duplicate record instead of updating the original.

Classify:

```text
identity preserved
intentional new identity
duplicate risk
broken-reference risk
```

## 19. `noupdate="1"` Analysis

For changed `noupdate` records compare:

```text
fresh install result
existing database upgrade result
```

Determine whether the installed record actually changes and whether explicit migration is required.

Never assume editing XML under `noupdate` updates an installed database.

## 20. Security and Configuration Data Upgrade

For ACLs, rules, groups, sequences, menus, actions, templates, and configuration data inspect:

- XML ID stability;
- `noupdate`;
- customer customization;
- group membership;
- company-specific records.

A fresh install and upgrade can produce different effective configuration.

## 21. Sequence and Numbering Changes

Determine whether existing numbering must continue.

Inspect company-specific sequences, prefixes/suffixes, next numbers, and historical references.

Do not reset production numbering accidentally.

## 22. Manifest Dependency and Load-Order Changes

For added dependencies ask:

- is it installed everywhere;
- does load order change;
- are cross-module fields/XML IDs available at upgrade time;
- is there a circular dependency;
- does deployment scope expand?

For removed dependencies, search all remaining model, field, XML ID, view, group, route, asset, and report references.

## 23. Cross-Module Upgrade Order

For multi-module changes build the dependency order explicitly.

Example:

```text
Module A requires migrated field from Module B
Module B requires data mapping before Module C view loads
```

Do not assume manifests alone express every safe migration sequence.

## 24. Moving Features Between Modules

When fields/views/data move between custom modules inspect:

- ownership;
- XML IDs;
- DB columns;
- `ir.model.data` ownership;
- security;
- dependencies;
- differences between installed customer databases.

Avoid duplicate definitions during transition.

## 25. Detect the Repository Migration Convention

Follow the repository's actual convention, which may include:

```text
migrations/<version>/
pre-migration.py
post-migration.py
end-migration.py
upgrade scripts
OpenUpgrade helpers
custom Plemo scripts
```

Do not invent a folder or naming convention when one already exists.

## 26. Pre-Migration vs Post-Migration

Use pre-migration only when work must happen before the new registry/schema loads, such as:

- column/table rename;
- preserving old data;
- preparing incompatible values;
- adjusting constraints.

Use post-migration when the new registry/schema is needed, such as:

- ORM backfill;
- relinking records;
- semantic data conversion;
- cleanup after schema creation.

Choose timing based on dependency requirements, not preference.

## 27. Migration Idempotency

Where practical, check for:

- record existence;
- column/table existence;
- already-mapped values;
- duplicate inserts;
- unique constraints.

Avoid blind repeated inserts/updates.

## 28. Partial Failure and Atomicity

Analyze what happens if migration fails halfway.

Consider:

- transaction rollback;
- DDL;
- manual commits;
- external side effects;
- file operations.

Avoid irreversible external API effects inside DB migration unless explicitly controlled.

## 29. Data Reconciliation

Define before/after evidence such as:

```text
source row count
target row count
null count
invalid value count
orphan count
critical aggregate
mapping completeness
```

A script finishing without traceback is not sufficient evidence of migration correctness.

## 30. Referential Integrity

Check:

- Many2one targets;
- Many2many relation rows;
- foreign keys;
- XML references;
- attachments;
- properties;
- followers/activities.

Do not leave orphaned references.

## 31. Duplicate Prevention

For split/merge/move migrations identify stable uniqueness criteria:

- XML IDs;
- external IDs;
- business identifiers;
- relation uniqueness;
- database constraints.

Handle partial/repeated migration safely.

## 32. Production Data Profiling

When safely available, profile representative real data:

- row counts;
- nulls;
- legacy states;
- archived rows;
- company distribution;
- website distribution;
- orphaned relations;
- customized configuration.

Production inspection should be read-only unless project workflow explicitly authorizes changes.

## 33. Legacy Data Exceptions

Expect old databases to contain states current code would never create.

Examples:

- missing now-required values;
- obsolete states;
- historical manual edits;
- old XML IDs;
- cross-company inconsistencies;
- legacy configuration.

Design migration around real data, not an idealized clean database.

## 34. Archived and Inactive Records

Include `active=False`, archived users/products/partners, old orders, and historical business records in upgrade risk.

They can still fail constraints or recomputation.

## 35. Multi-Company Migration

For company-aware data inspect:

- `company_id`;
- shared `company_id=False` records;
- company-dependent fields/properties;
- company-specific config;
- `check_company`.

Do not assign the wrong company during backfill.

## 36. Multi-Website Migration

For website-aware data inspect:

- `website_id`;
- publication state;
- website-specific settings;
- company mapping;
- shared records.

Do not accidentally publish or transfer private records across websites.

## 37. Environment-Specific Data

Identify data that differs between local/dev/staging/production:

- payment configs;
- credentials;
- company/warehouse/website IDs;
- journals;
- external endpoints;
- sequences.

Do not hardcode environment-specific integer DB IDs into migrations.

Prefer stable XML IDs or business keys when appropriate.

## 38. Configuration Preservation

Determine whether customer customization must survive migration.

Potentially sensitive records include:

- mail templates;
- settings;
- sequences;
- security groups/rules;
- website pages;
- payment providers;
- journals.

Do not overwrite intentional production customization without evidence.

## 39. Major Odoo Version Upgrade Gate

For major-version upgrades expand analysis to:

- Python ORM APIs;
- method signatures;
- renamed/removed models and fields;
- view/XML architecture;
- QWeb;
- JS/OWL imports/services/patches;
- assets;
- routes/controllers;
- security behavior;
- mail/report APIs;
- manifests/tests.

Do not treat a major-version upgrade as only a database schema migration.

## 40. Target-Version Verification

For suspected deprecated/version-sensitive behavior, verify using:

- target Odoo source;
- repository examples;
- Plemo instructions/lessons;
- authoritative migration notes when available.

Do not rely on a giant hardcoded compatibility table.

## 41. View Compatibility

Inspect target-version behavior for:

- inherited view XPath;
- modifiers;
- settings view structure;
- renamed XML IDs/views;
- QWeb directives;
- version-specific architecture changes.

## 42. JavaScript/OWL Compatibility

Review:

- imports;
- registries/services;
- patch API;
- lifecycle;
- templates;
- RPC/ORM service use;
- assets.

Material frontend version upgrades require browser/runtime proof after implementation.

## 43. Controller/Route Compatibility

Review route decorators, auth/type, request APIs, JSON behavior, HTTP responses, CSRF/CORS, and controller inheritance against the target Odoo version.

## 44. Migration Mapping

For each transformation record:

```text
Source:
Target:
Mapping rule:
Fallback:
Invalid-source handling:
Unmapped rows:
Reversibility:
```

Prefer deterministic mappings.

## 45. Preserve Business Meaning

A migration is not correct merely because every row fits the new schema.

Verify that old business meaning maps correctly to new meaning.

Document lossy mappings explicitly.

## 46. Backfill Strategy

Classify each backfill as:

```text
constant default
derived from existing fields
derived from related records
derived from business rules
manual review
leave null
```

Use the least misleading strategy.

## 47. Manual Review Bucket

If data cannot be inferred safely, define records requiring manual review rather than inventing values.

Record:

```text
criteria
reason
safe temporary state
blocking/non-blocking
```

## 48. Migration Volume and Performance

Estimate:

```text
records scanned
records updated
records recomputed
indexes/constraints touched
relation rows affected
```

Classify volume as small/moderate/large/unknown.

Do not claim performance without measurement.

## 49. Locking and Downtime Risk

Consider:

- long transactions;
- table locks;
- index creation;
- large recompute;
- concurrent writes;
- cron/queue interference;
- worker restart.

State whether maintenance/downtime may be required, but do not invent a duration.

## 50. Batch Strategy

For large migrations assess whether project conventions favor ORM batches, SQL, checkpointing, or another controlled method.

Do not recommend manual commits casually; analyze transaction and rollback consequences first.

## 51. Index and Constraint Changes

Before adding/changing constraints inspect historical rows for duplicates, nulls, and invalid relationships.

Constraints can fail during upgrade even when new records are valid.

## 52. Cron and Automation During Upgrade

Determine whether scheduled jobs, automated actions, mail tracking, queue jobs, or webhooks can react to partial migration state.

Controlled upgrade environments are preferred.

## 53. Rollback Classification

Classify:

```text
Fully reversible
Code-reversible, data not automatically reversible
Requires backup restore
Irreversible/lossy
Unknown
```

Do not call a migration rollback-safe merely because Git can revert code.

## 54. Destructive Change Detection

Flag:

- column/table drops;
- data deletion;
- record merges;
- selection collapse;
- lossy type conversion;
- XML ID replacement;
- historical data rewrite.

Require backup/rollback consideration.

## 55. Backup Dependency

For high-risk/irreversible migrations record whether safe recovery requires:

- database backup;
- snapshot;
- clone;
- export;
- external reconciliation.

Do not execute destructive production migration without the project's required authorization and backup workflow.

## 56. Forward-Fix vs Rollback

Some migrations are safer to forward-fix than revert.

Record:

```text
rollback safe:
forward-fix preferred:
reason:
```

## 57. Fresh Install Analysis

Fresh install proves:

- manifest/dependencies;
- data load order;
- security/data XML;
- initial hooks/data;
- installability.

It does not prove historical migration.

## 58. Existing Database Upgrade Analysis

Existing DB upgrade must prove:

- schema transition;
- legacy-value handling;
- stored recompute;
- `noupdate`;
- migration scripts;
- constraints;
- company/website data;
- customer configuration preservation.

Fresh install PASS does not imply upgrade PASS.

## 59. Installed-State Variants

Consider:

```text
module not installed
module installed old version
dependency absent
partial prior migration
customer-specific optional module installed
```

Different customer databases may need different handling.

## 60. Upgrade Validation Matrix

For material migration define:

```text
ID:
Scenario:
Database state:
Expected:
Evidence required:
Status:
```

Potential scenarios:

- fresh install;
- upgrade from prior module version;
- upgrade with legacy values;
- archived records;
- multi-company data;
- `noupdate` behavior;
- stored recompute;
- rollback/restore rehearsal when required.

## 61. Pre-Upgrade Assertions

Capture relevant baseline evidence:

- row counts;
- state distributions;
- null/invalid counts;
- orphan counts;
- XML IDs;
- critical totals;
- configuration values.

## 62. Post-Upgrade Assertions

Verify:

- mapped values;
- row counts;
- null/invalid counts;
- orphan counts;
- references;
- critical totals;
- security/configuration;
- business workflows.

## 63. Business Reconciliation

For high-impact domains compare meaningful aggregates, such as:

- invoice/order totals;
- stock quantities;
- payroll totals;
- counts by business state.

Row count alone may be insufficient.

## 64. Migration Log Review

Inspect upgrade logs for:

- tracebacks;
- missing XML IDs;
- constraint failures;
- recompute errors;
- lock/timeout issues;
- deprecated API failures.

Separate pre-existing noise from migration-caused failures.

## 65. Security During Migration

If migration changes ACLs, rules, groups, ownership, company data, portal access, tokens, or secrets, apply the relevant security and access checks where material.

A correct data transformation can still create a security regression.

## 66. Migration Risk Severity

Use:

```text
Critical
High
Medium
Low
Informational
```

Consider data loss, business-history corruption, upgrade failure, security exposure, financial/stock/payroll impact, irreversibility, and downtime.

## 67. Evidence Confidence

Use:

```text
Confirmed
Strong evidence
Probable
Possible
Needs runtime/database verification
```

Keep confidence separate from severity.

## 68. Finding Structure

For each material finding record:

```text
Finding ID:
Title:
Severity:
Confidence:
Change:
Existing-data risk:
Upgrade failure mode:
Affected modules/models/fields:
Affected records:
Evidence:
Migration requirement:
Runtime validation required:
Rollback implication:
```

## 69. Migration Boundary Recommendation

State:

```text
Module(s):
Migration location:
Pre/post stage:
Fields/data affected:
Dependencies:
Do-not-touch areas:
Reason:
```

This is evidence for Plemo's planner, not a second generic implementation plan.

## 70. Do-Not-Touch Boundary

For material migrations record:

```text
Odoo core:
Third-party/OCA:
Shared custom modules:
Sibling customer projects:
Production data outside scope:
Security-sensitive data:
External systems:
Other protected areas:
```

## 71. Required Migration Evidence Output

For material analysis output:

```text
MIGRATION EVIDENCE

Project:
Change:
Current Odoo version:
Target Odoo version:
Current module version:
Target module version:

Fresh install impact:
Existing DB upgrade impact:
Schema impact:
Stored compute impact:
Selection impact:
XML ID/noupdate impact:
Dependency/load-order impact:
Existing-data impact:
Multi-company impact:
Multi-website impact:
Security migration impact:
Migration script required:
Runtime upgrade validation required:
Rollback classification:
Do-not-touch boundary:
Remaining unknowns:
Overall migration risk:
```

## 72. Migration Requirement Status

Use:

```text
NO MIGRATION REQUIRED
MIGRATION REQUIRED
MIGRATION RECOMMENDED
PARTIAL / NEEDS DATABASE VERIFICATION
BLOCKED
```

`NO MIGRATION REQUIRED` means no persistent compatibility work is required for the reviewed change.

`MIGRATION REQUIRED` means an installed database cannot safely adopt the change without explicit migration handling.

`MIGRATION RECOMMENDED` means automatic upgrade may technically work but explicit backfill/reconciliation is safer or required for business correctness.

`PARTIAL / NEEDS DATABASE VERIFICATION` means repository evidence is insufficient to prove existing-data behavior.

`BLOCKED` means required database state, version information, or migration convention is unavailable.

## 73. Detailed Report Structure

For complex work use:

1. Upgrade Target
2. Version Evidence
3. Actual Change Set
4. Migration Surface
5. Schema Changes
6. Stored Compute/Recompute
7. Selection Changes
8. XML IDs and `noupdate`
9. Manifest/Dependency/Load Order
10. Existing Production Data
11. Multi-Company / Multi-Website
12. Migration Scripts
13. Major-Version Compatibility
14. Performance / Downtime Risk
15. Rollback / Reversibility
16. Validation Matrix
17. Findings
18. Recommended Migration Boundary
19. Runtime Validation Requirements
20. Remaining Unknowns
21. Final Migration Requirement Status

## 74. Concise Output for Small Changes

For a contained change use:

```text
Migration Status:
NO MIGRATION REQUIRED / MIGRATION REQUIRED / MIGRATION RECOMMENDED / PARTIAL / BLOCKED

Change:
Existing DB impact:
Required migration:
Upgrade validation:
Rollback:
```

Do not force a full migration report on trivial changes.

## 75. Integration With Native Planning

For an authorized implementation provide concise `MIGRATION EVIDENCE`, then return control to the native planner.

Do not require duplicate approval and do not generate another generic implementation plan.

## 76. Post-Implementation Validation Requirements

After implementation provide exact runtime scenarios, such as:

```text
fresh install
upgrade prior version
verify legacy selection values mapped
verify stored compute recomputed
verify no orphaned relations
verify customer configuration preserved
verify critical totals reconciled
```

The broader post-change validation phase owns the final PASS/FAIL/PARTIAL/BLOCKED result.

## 77. Final Recheck

After implementation/fix recheck:

- final Git diff;
- manifest version;
- migration folder/version;
- schema definitions;
- XML IDs;
- `noupdate`;
- dependencies/load order;
- migration scripts;
- temporary compatibility code.

## 78. Hard Prohibitions

Never:

- replace Plemo's native planner or deployment workflow;
- require duplicate approval for an already-authorized fix;
- create migration scripts during review-only mode;
- assume fresh install proves upgrade safety;
- assume upgrade success proves data correctness;
- assume defaults update historical rows;
- rename fields without protecting old data;
- remove/rename Selection keys without checking stored values;
- change XML IDs casually;
- assume `noupdate` edits affect installed databases;
- overwrite customer configuration without evidence;
- hardcode environment-specific database IDs;
- drop data without rollback/backup analysis;
- rewrite historical snapshots blindly;
- ignore archived/legacy records;
- ignore multi-company/multi-website semantics when relevant;
- assume migration scripts are idempotent;
- ignore partial-failure behavior;
- claim rollback because Git can revert code;
- invent migration performance or downtime estimates;
- run destructive production migration without authorization and backup workflow;
- modify core/third-party code against project rules;
- expand into sibling customer projects without evidence;
- claim migration complete when required DB/runtime verification is unavailable;
- commit or push unless explicitly requested.

## 79. Core Decision Tree

```text
START
  ↓
Read plemo.md / project rules
  ↓
Determine task mode
  ↓
Reuse impact evidence
  ↓
Inspect actual change
  ↓
Persistent schema/data/config change?
  ├─ No → likely NO MIGRATION REQUIRED
  └─ Yes
       ↓
Classify migration surface
       ↓
Analyze fields / stored compute / Selection
       ↓
Analyze XML IDs / noupdate / config
       ↓
Analyze dependencies / module order
       ↓
Major Odoo version change?
       ├─ Yes → expand compatibility analysis
       ↓
Profile existing data when safely available
       ↓
Define migration mapping and reconciliation
       ↓
Assess volume / locks / rollback
       ↓
Produce MIGRATION EVIDENCE
       ↓
Analysis-only?
       ├─ Yes → stop
       └─ No → continue into Plemo native planning using the evidence
       ↓
After implementation → carry upgrade scenarios into broader post-change runtime validation
```

## 80. Primary Rules to Always Remember

```text
Upgrade safety is about existing databases, not only code.

Reuse existing Plemo evidence.
Do not duplicate investigation.

Preserve Plemo native planning, implementation, and deployment.
Do not require duplicate approval.

Use plemo.md and project migration conventions first.

Separate:
fresh install
installed database upgrade
production-data behavior

For new fields:
check historical rows, defaults, and required values.

For renames:
preserve existing data and identity.

For type changes:
prove data conversion safety.

For stored computes:
analyze recomputation, side effects, history, and volume.

For Selection changes:
inspect real stored values.

For XML IDs:
preserve identity unless intentionally migrated.

For noupdate:
do not assume XML edits update installed records.

For dependencies:
analyze load and upgrade order.

For major-version upgrades:
verify target-version backend, view, frontend, and route APIs.

For migrations:
use deterministic mappings and reconciliation evidence.

For production data:
expect legacy exceptions.

For rollback:
separate code rollback from data rollback.

For destructive changes:
backup restore may be the only real rollback.

For performance:
measure or state uncertainty.

Output migration evidence, not a second generic implementation plan.

After implementation:
perform broader post-change regression/runtime validation to prove the upgrade path.
```