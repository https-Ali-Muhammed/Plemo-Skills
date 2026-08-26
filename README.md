# Plemo Skills

A collection of reusable AI agent skills used for Odoo development, investigation, localization, impact analysis, QA, and related workflows at Plemo.

These skills are designed to **extend the agent's native capabilities rather than replace them**. Repository-specific instructions such as `plemo.md`, configured addon paths, and the agent's normal planning and implementation workflow remain authoritative. Skills provide specialized Odoo evidence, procedures, safeguards, and validation rules.

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

## Skill Interaction Model

```text
Repository instructions / plemo.md
        ↓
Agent native discovery + task mode
        ↓
Codebase / localization evidence
        ↓
Feature impact evidence when material
        ↓
Agent native planning
        ↓
Agent native implementation
        ↓
Odoo Regression & Runtime Validator
        ↓
Evidence-backed PASS / FAIL / PARTIAL / BLOCKED
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

## Repository Structure

```text
Plemo-Skills/
├── README.md
├── odoo_localization_arabic_qa_skill.md
├── odoo_codebase_investigator_skill.md
├── odoo_feature_impact_analyzer_skill.md
└── odoo_regression_runtime_validator_skill.md
```
