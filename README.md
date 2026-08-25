# Plemo Skills

A collection of reusable AI agent skills used for development, code review, localization, QA, and other workflows at Plemo.

## Available Skills

### 1. Odoo Localization & Arabic QA

**File:** `odoo_localization_arabic_qa_skill.md`

A reusable skill for safely investigating, implementing, reviewing, and validating Arabic localization in Odoo modules across different Odoo versions.

### 2. Odoo Codebase Investigator

**File:** `odoo_codebase_investigator_skill.md`

A read-only investigation skill for understanding an Odoo codebase before changes are made. It traces feature ownership, XML/QWeb inheritance, Python model and method overrides, controllers, JavaScript, assets, dependencies, and existing implementations, then produces a structured investigation report with the safest recommended modification location.

### 3. Odoo Feature Impact Analyzer

**File:** `odoo_feature_impact_analyzer_skill.md`

A pre-implementation analysis skill for determining how a requested Odoo change may affect existing modules, dependencies, models, views, controllers, JavaScript, assets, security, data, integrations, performance, and regression risk, then identifying the smallest safe implementation scope.

## Repository Structure

```text
Plemo-Skills/
├── README.md
├── odoo_localization_arabic_qa_skill.md
├── odoo_codebase_investigator_skill.md
└── odoo_feature_impact_analyzer_skill.md
```
