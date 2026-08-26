# Plemo Skills

A collection of reusable AI agent skills used for Odoo development, investigation, localization, impact analysis, QA, and related workflows at Plemo.

These skills are designed to **extend the agent's native capabilities rather than replace them**. Repository-specific instructions such as `plemo.md`, configured addon paths, and the agent's normal planning and implementation workflow remain authoritative. Skills provide specialized Odoo evidence, procedures, safeguards, and validation rules.


### Silent Use During Normal Chat

Skill names and numbers in this README are repository documentation only.

During normal user conversations, Plemo should apply the relevant knowledge, procedures, and safeguards internally without requiring the user to invoke a named skill. Plemo should not announce which skill it selected, ask the user to choose a skill, expose internal skill routing, or refer to numbered skills unless the user explicitly asks about the skill library itself.

The operational skill files are therefore written as reusable guidance rather than as separate agents that route work to one another.

## Available Skills

### 1. Odoo Localization & Arabic QA

**File:** `odoo_localization_arabic_qa_skill.md`

A specialized Odoo localization skill for investigating, implementing, reviewing, and validating Arabic localization across different Odoo versions. It preserves the agent's native task mode, supports repository/module discovery before asking unnecessary questions, protects PO structure and existing translations, and applies Plemo's JavaScript `localize("English", "Arabic")` workflow only within the intended localization scope.

### 2. Odoo Codebase Investigator

**File:** `odoo_codebase_investigator_skill.md`

A read-only Odoo investigation skill for tracing feature ownership, repository structure, XML/QWeb inheritance, Python models and method overrides, controllers, JavaScript, assets, dependencies, and existing implementations. It acts as an evidence provider: standalone investigation requests remain read-only, while investigation performed inside an already-authorized implementation task hands its findings back to the agent's native planning workflow without requiring duplicate approval.

### 3. Odoo Feature Impact Analyzer

**File:** `odoo_feature_impact_analyzer_skill.md`

An Odoo-specific change-impact discovery skill for material changes. It analyzes direct, reverse, transitive, dynamic, and runtime dependencies; data and upgrade consequences; stored-compute and Selection-change risks; JS/RPC and asset impact; security, integrations, performance, regression surface, runtime-verification requirements, and explicit do-not-touch boundaries. It produces structured impact evidence for the agent's native planner instead of replacing the agent's own implementation planning.

### 4. Odoo Regression & Runtime Validator

**File:** `odoo_regression_runtime_validator_skill.md`

A post-implementation Odoo validation skill that proves what actually works after a change. It reuses existing investigation and impact evidence, validates the final diff, separates static checks from runtime/browser/security/integration proof, distinguishes fresh install from module upgrade and existing-data behavior, selects regression tests from the real impact surface, protects production from unsafe validation actions, and reports explicit `PASS`, `FAIL`, `PARTIAL`, or `BLOCKED` results without replacing Plemo's native planning, implementation, or debugging workflow.

### 5. Odoo Security & Access Reviewer

**File:** `odoo_security_access_reviewer_skill.md`

A deep Odoo security specialist for reviewing effective access across ACLs, record rules, groups and implied groups, field-level restrictions, `sudo()` and execution-user changes, controllers/RPC, portal/public routes, multi-company and multi-website isolation, tokens, attachments, reports/exports, automation users, secrets, and integration endpoints. It builds actor and access-path evidence, identifies privilege-escalation and data-leak paths, separates severity from confidence, recommends the smallest safe server-side enforcement boundary, and hands fixes back to Plemo's native planner rather than replacing it.

### 6. Odoo Upgrade & Migration Analyzer

**File:** `odoo_upgrade_migration_analyzer_skill.md`

An Odoo upgrade and migration specialist for determining how schema, stored-compute, Selection, XML ID, `noupdate`, dependency, configuration, and major-version changes affect installed databases and existing production data. It separates fresh installation from existing-database upgrade behavior, identifies required migration scripts and data mappings, analyzes recomputation, legacy-data, performance, downtime, and rollback risk, and produces structured migration evidence for Plemo's native planner without replacing the agent's deployment or implementation workflow.

### 7. Odoo Performance Analyzer

**File:** `odoo_performance_analyzer_skill.md`

An Odoo performance specialist for analyzing realistic execution paths, ORM/query behavior, N+1 patterns, search/write amplification, computed-field and recompute fan-out, cron/import/report workloads, controller/RPC latency, frontend request/render cost, caching, memory, locking/concurrency, indexes, and migration-time performance. It separates static performance risk from measured evidence, requires realistic data-scale and before/after proof when needed, recommends the smallest safe optimization boundary, and hands implementation back to Plemo's native planner instead of becoming a generic optimizer.

### 8. Odoo Code Quality Reviewer

**File:** `odoo_code_quality_reviewer_skill.md`

An Odoo code-quality and maintainability specialist for reviewing module ownership, addon boundaries, inheritance and `super()` contracts, multi-record safety, fields and ORM usage, computes, CRUD overrides, context usage, controllers, XML/XPath, QWeb, JavaScript/OWL, assets, duplication, abstraction, dead code, logging, testability, and version/upgrade fragility. It follows `plemo.md` and repository conventions before generic style preferences, separates correctness and maintainability risks from subjective style, recommends the smallest safe improvement boundary, and preserves Plemo's native planning, implementation, approval, and task-mode behavior.

## Skill Interaction Model

```text
Repository instructions / plemo.md
        ↓
Plemo native discovery + task mode
        ↓
Apply relevant specialized evidence, procedures, and safeguards internally
        ↓
Plemo native planning
        ↓
Plemo native implementation
        ↓
Relevant post-change validation
        ↓
Evidence-backed result
```

General rules:

- Reuse reliable evidence already collected by another skill during the same task.
- Do not repeat full investigation when existing evidence is sufficient.
- Do not force heavyweight workflows on trivial isolated changes.
- Review / audit / diagnosis tasks remain read-only unless fixes were explicitly requested.
- An existing add / fix / build / change / implement request already counts as implementation authorization; skills must not require a second approval solely because they performed investigation or impact analysis.
- Repository-specific guidance such as `plemo.md` takes precedence over generic repository-layout examples inside a skill.
- A valid Odoo module manifest version prefix may be used as version evidence when Odoo core source is unavailable.
- Validation must distinguish static evidence from actual runtime proof; an unavailable required runtime check must never be reported as passed.
- Production validation should prefer read-only/reversible observation and must not perform destructive or business-impacting actions without the authorization required by the project workflow.
- Security-sensitive changes should be reviewed by actor, access path, and effective server-side enforcement; UI visibility alone is not treated as security.
- `sudo()` is treated as a privileged bypass that requires explicit justification, narrow scope, and authorization of user-controlled records.
- Upgrade-sensitive changes must distinguish fresh installation from existing-database upgrade behavior and treat existing production data as part of the compatibility contract.
- Migration analysis must identify data mappings, recomputation, XML ID/`noupdate` behavior, rollback limitations, and runtime upgrade verification without replacing Plemo's native deployment planning.
- Performance-sensitive changes must distinguish static risk from measured runtime evidence; performance improvements or regressions should not be claimed without comparable measurement when measurement is required.
- Performance optimization must preserve business correctness and security, use realistic data volume/concurrency, and remain subordinate to Plemo's native planning and implementation workflow.
- Code-quality review must follow `plemo.md` and repository conventions before generic style preferences, distinguish objective maintainability/correctness risk from subjective style, and avoid unrelated refactoring.
- Skill names and numbers are documentation metadata only; operational guidance should be applied internally during normal chat without named routing, invocation requests, or capability announcements.

## Repository Structure

```text
Plemo-Skills/
├── README.md
├── odoo_localization_arabic_qa_skill.md
├── odoo_codebase_investigator_skill.md
├── odoo_feature_impact_analyzer_skill.md
├── odoo_regression_runtime_validator_skill.md
├── odoo_security_access_reviewer_skill.md
├── odoo_upgrade_migration_analyzer_skill.md
├── odoo_performance_analyzer_skill.md
└── odoo_code_quality_reviewer_skill.md
```
