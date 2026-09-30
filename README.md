# Plemo Skills

A collection of reusable AI agent skills used for Odoo development, investigation, localization, impact analysis, QA, frontend/OWL engineering, integration reliability, reporting/document generation, operational data exchange, production diagnostics, cross-version source compatibility, accounting integrity, and related workflows at Plemo.

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

### 9. Odoo Automated Test Engineer

**File:** `odoo_automated_test_engineer_skill.md`

An Odoo automated-test engineering specialist for designing, implementing, reviewing, and maintaining durable regression coverage at the correct test layer. It reuses investigation, impact, security, migration, performance, localization, code-quality, and runtime-validation evidence; detects the actual Odoo version and repository test conventions before selecting APIs; inventories existing tests before adding new ones; and chooses the smallest reliable layer across ORM/business-flow, security/user-context, controllers/HTTP/RPC, reports, crons/jobs, integrations, migrations, frontend unit tests, and browser/tour tests. It emphasizes deterministic fixtures, actor/company/website-aware scenarios, positive and negative coverage, isolation from live external systems, version-verified test frameworks, discovery/CI verification, flakiness prevention, and semantic assertions over implementation-detail tests. It creates permanent automated protection without replacing Plemo's native planning/implementation workflow or the separate Regression & Runtime Validator, which remains responsible for environment-specific runtime proof.

### 10. Odoo Frontend & OWL Specialist

**File:** `odoo_frontend_owl_specialist_skill.md`

An Odoo frontend and OWL engineering specialist for investigating, designing, implementing, reviewing, debugging, and safely extending browser-side behavior across the backend web client, website, portal, and other relevant Odoo frontend surfaces. It detects the actual Odoo version and frontend architecture before selecting APIs; traces component, template, service, registry, patch, asset, RPC/controller, model, and CSS ownership; and prefers the narrowest stable framework extension boundary over global patches, copied upstream components, or fragile DOM manipulation. It adds deep guidance for OWL lifecycle and reactive state, async/race handling, service and registry contracts, patch composition/load order, QWeb/OWL template inheritance, asset bundles/import paths, frontend-to-server contracts, public/portal and multi-company/multi-website contexts, website/POS-specific architecture, browser debugging, and version upgrades. It keeps server-side security and business rules authoritative, reuses Impact/Security/Performance/Localization evidence instead of duplicating those skills, hands durable test design to the Automated Test Engineer, and leaves final browser/runtime proof to the Regression & Runtime Validator.

### 11. Odoo Integration & Webhook Reliability Specialist

**File:** `odoo_integration_webhook_reliability_specialist_skill.md`

An Odoo integration-reliability specialist for investigating, designing, implementing, reviewing, and debugging external API, webhook, polling, synchronization, queue/job, and provider workflows under real distributed-system failure conditions. It traces the Odoo business owner, transport boundary, external identifiers, synchronization state, credentials/configuration scope, provider contract/version, transaction boundaries, and recovery path; then applies explicit rules for bounded retry, timeout ambiguity, rate limits, durable idempotency, duplicate and out-of-order webhook delivery, pagination/cursors, source-of-truth mappings, schema drift, multi-company/multi-website ownership, reconciliation, operator recovery, and sanitized observability. It does not replace Security, Migration, Performance, Impact, Automated Testing, or Runtime Validation: it identifies and owns the integration protocol/reliability contract, reuses those skills' evidence, hands durable regression scenarios to the Automated Test Engineer, and leaves provider/sandbox/runtime proof to the Regression & Runtime Validator.

### 12. Odoo Reporting & Document Specialist

**File:** `odoo_reporting_document_specialist_skill.md`

An Odoo reporting and document-generation specialist for investigating, designing, implementing, reviewing, debugging, and safely extending QWeb/PDF/HTML reports, report actions, data providers, external layouts, paper formats, report attachments, mail-attached documents, portal/public downloads, barcodes/images, multi-company branding, multilingual/RTL output, and repository-supported custom export formats. It detects the actual Odoo version and rendering engine before selecting APIs or layout techniques; traces the report action → report-data → template/inheritance → external-layout → paper-format/renderer → attachment/download/mail chain; preserves authoritative business and legal values; keeps complex data preparation out of QWeb where appropriate; and explicitly separates HTML correctness from real PDF/document rendering proof. It reuses Localization for translation mechanics and Arabic wording, Security for deep authorization/access review, Migration for persistent report/XML-ID/`noupdate` changes, Performance for measured report bottlenecks, Automated Testing for durable semantic coverage, and Runtime Validation for actual renderer/PDF/portal/mail proof rather than duplicating those workflows.

### 13. Odoo Data Import, Export & Data Exchange Specialist

**File:** `odoo_data_import_export_exchange_specialist_skill.md`

A Plemo-directed specialist for the **operational movement of Odoo business records** through native import/export facilities and repository-supported CSV/XLSX/file pipelines. Based on Plemo's native capability assessment, it does not recreate native repository/version discovery or generic verification. Instead, it requires explicit data/field contracts, stable record identity and create/update/skip/reject rules, ambiguous/archived-record handling, repeatable imports, missing-versus-empty/null/zero/false semantics, relational/company mapping, validation previews, transaction atomicity, partial-failure recovery, restart checkpoints, row-level error reporting, data reconciliation, and stable raw-data export schemas and round trips. It reuses Migration for installed-database evolution, Integration for external transport and continuous synchronization, Reporting for printable/document-oriented exports, Security/Performance for deep safeguards, Automated Testing for durable scenarios, and Runtime Validation for actual data-level proof.

### 14. Odoo Production Diagnostics & Observability Specialist

**File:** `odoo_production_diagnostics_observability_specialist_skill.md`

A Plemo-directed **incident-triage and evidence-correlation specialist** grounded in Plemo's native-capability assessment and actual diagnostic-tool limits. It classifies active/historical/intermittent incidents, affected scope and business side-effect risk; reconstructs bounded timelines; distinguishes local developer logs, Odoo.sh build/startup logs, and production request-time evidence; traces only observed or clearly inferred layers; isolates the first trustworthy failure from wrapper/secondary errors; and produces precise evidence requests, confirmed-versus-hypothetical findings, confidence, and safe stop conditions. It cannot grant missing production log, PostgreSQL, worker, proxy, browser, or resource-metric access and does not replace native debugging or read-only authorization. It reuses Codebase Investigator for source ownership, Regression & Runtime Validator for post-fix proof, Performance for measurement, Integration for external reliability, Security for deep access review, and Automated Testing for durable incident-derived coverage.

### 16. Odoo Cross-Version Source Compatibility Analyzer

**File:** `odoo_cross_version_source_compatibility_analyzer_skill.md`

A Plemo-directed **cross-version evidence provider** created from the native-capability assessment for the proposed Version Compatibility skill. It does not duplicate Plemo's existing version detection or become a second upgrade planner. Given a source Odoo version, a target Odoo version, and a named symbol or framework area, it builds comparable source/target snapshots; classifies removed, renamed, moved, deprecated, signature-changed, behavior-changed, replaced, or now-native contracts; verifies semantic rather than name-only replacements; detects stale copied-upstream and compatibility-shim code; records module/dependency and existing-test-framework evolution; and emits reusable compatibility findings with evidence, status, confidence, affected custom code, migration implications, test requirements, and runtime-proof requirements. Installed-database/schema/data transformation remains with Upgrade & Migration Analyzer, target frontend architecture with Frontend & OWL Specialist, report implementation with Reporting & Document Specialist, external-provider versioning with Integration & Webhook Reliability Specialist, test design with Automated Test Engineer, and post-change proof with Regression & Runtime Validator.

### 17. Odoo Accounting Integrity & Financial Workflow Specialist

**File:** `odoo_accounting_integrity_financial_workflow_specialist_skill.md`

A Plemo-directed **financial-integrity and accounting-semantics specialist** created from the native-capability assessment for the broader Accounting & Financial Workflow proposal. It deliberately does not duplicate generic repository discovery, security authorization, migration mechanics, performance measurement, report rendering, provider reliability, automated-test architecture, cross-version API diffing, or runtime validation. Instead, it establishes the authoritative financial contract around `account.move`/`account.move.line`, draft-versus-posted history, balancing, correction paths, reversals and credit/refund flows, journals and numbering, lock dates, taxes and fiscal positions, country accounting-localization boundaries, currency and exchange differences, rounding categories, invoices/bills, payments, residual/payment-state truth, accounting reconciliation/unreconciliation, bank/cash flows, analytic accounting, multi-company financial configuration, financial imports/migrations, stock-valuation accounting boundaries, and financial idempotency. It supplies explicit before/after accounting evidence—balances, taxes, residuals, reconciliation, reversal linkage, company/journal/currency context—and hands authorization, transformation mechanics, measurement, tests, provider transport, document presentation, version evolution, incident correlation, and final runtime proof to their owning skills.

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
- Automated-test engineering must protect meaningful business and framework contracts with the smallest reliable test layer, reuse existing evidence, follow repository/version-specific test conventions, avoid live external dependencies and brittle implementation-detail assertions, and keep automated coverage separate from runtime/environment proof.
- A green automated test suite must not be treated as proof of browser asset behavior, existing-database upgrade safety, live integration behavior, production-scale concurrency/performance, or other runtime-only conditions that require the Regression & Runtime Validator.
- Frontend/OWL engineering must detect the actual Odoo version and frontend surface, trace component/template/service/registry/patch/asset/server ownership before implementation, prefer the narrowest supported extension point, keep business/security enforcement server-side, and separate static frontend confidence from actual browser/runtime proof.
- Integration/webhook reliability must treat external identifiers and synchronization state as compatibility data, classify timeout/retry behavior, make material side effects idempotent, handle duplicate/out-of-order delivery safely, preserve a reconciliation path for uncertain outcomes, and avoid live external side effects during ordinary automated testing.
- A transport success or HTTP 2xx must not automatically be treated as final business success, and a timeout must not automatically be treated as remote failure when the provider may already have committed the operation.
- Reporting/document work must detect the actual Odoo version and renderer, trace report action/data/template/layout/paper-format/delivery ownership, preserve authoritative business/legal values, keep security server-side, and distinguish HTML/static correctness from actual generated-document proof.
- Report localization must reuse the localization workflow for translation mechanics and Arabic terminology, while the reporting workflow owns document language context, layout/RTL behavior, renderer compatibility, attachment semantics, and final generated-output requirements.
- Operational data import/export must establish stable record matching and ambiguity behavior, explicit field-value and source-of-truth policies, preview/apply and transaction semantics, safe repeat/restart behavior, and reconciliation against actual business results; native discovery and generic version grounding are reused rather than duplicated.
- Machine-readable data exports must preserve deliberate schema/identity/encoding/order/null contracts and server-side actor/company/field authorization. Migration scripts, external API/webhook protocols, and document-oriented reporting remain with their owning skills.
- Production diagnostics must respect actual tool/environment reach: developer-local Odoo logs and Odoo.sh build/startup logs are not arbitrary production request-time evidence. When necessary sources are inaccessible, request a bounded redacted operator artifact; never claim unseen logs, PostgreSQL/worker state, metrics, or browser traces were inspected.
- Incident investigation must classify active/historical/intermittent state, affected record/user/company/database/service scope and side-effect risk; build a time-and-source-grounded timeline; separate first trustworthy failures from wrapper symptoms and confirmed causes from hypotheses; and prohibit unsafe live re-triggering during diagnosis-only work.
- Production Diagnostics owns incident intake, cross-layer evidence correlation, unavailable-evidence reporting, and narrow next-step requests—not post-change Runtime Validation, measured Performance methodology, static Codebase Investigation, or external Integration protocol engineering.
- Cross-version compatibility analysis must be scoped to a known source version, target version, and named symbol/area. It produces source→target delta evidence rather than re-performing generic version detection or creating a second implementation plan.
- Cross-version findings must compare semantic contracts, not only symbol names, and may classify APIs/extension points as compatible, deprecated, behavior/signature changed, renamed, moved, replaced, removed, now-native/redundant, or unverified. Unproven replacements must never be invented.
- Installed-database/schema/data compatibility remains with Upgrade & Migration Analyzer; external-provider API versioning remains with Integration; target frontend/report architecture remains with their domain specialists; automated test design and runtime proof remain with their owning skills.
- Accounting-integrity analysis owns financial correctness rather than generic authorization or CRUD mechanics: classify financial object/state, preserve authoritative posting/reversal/reconciliation workflows, protect debit/credit and company/journal/currency invariants, and keep derived residual/payment/tax states tied to their real accounting source.
- Posted financial history and lock dates are control boundaries, not inconveniences. Do not silently mutate posted history, bypass lock controls, force residual/payment state, or choose a journal/account/tax/write-off policy merely to make an implementation succeed.
- Financial correction must prefer the authorized accounting path—draft correction, reversal, credit/refund, controlled reconciliation adjustment, or another verified workflow—rather than arbitrary posted-record writes.
- Accounting reporting presents authoritative financial values but does not own their computation; external integrations prove provider-side outcomes but do not prove Odoo-side financial truth; imports/migrations own movement/transformation mechanics while Accounting Integrity defines the financial invariants that must survive.
- Retryable financial operations must be financially idempotent: a repeated invoice, posting, payment, reconciliation, credit/refund, bank-import application, or callback must not create a duplicate economic effect merely because technical retry occurred.
- Material accounting changes should define before/after evidence such as debit/credit totals, tax totals, document total, residual, payment/reconciliation state, reversal linkage, journal, company, currency, lock-date context, and the intended financial delta; absence of a traceback is not accounting proof.
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
├── odoo_code_quality_reviewer_skill.md
├── odoo_automated_test_engineer_skill.md
├── odoo_frontend_owl_specialist_skill.md
├── odoo_integration_webhook_reliability_specialist_skill.md
├── odoo_reporting_document_specialist_skill.md
├── odoo_data_import_export_exchange_specialist_skill.md
├── odoo_production_diagnostics_observability_specialist_skill.md
├── odoo_cross_version_source_compatibility_analyzer_skill.md
└── odoo_accounting_integrity_financial_workflow_specialist_skill.md
```
