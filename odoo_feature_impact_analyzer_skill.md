# Odoo Feature Impact Analyzer

## Purpose

Use this skill to analyze the impact of a requested Odoo feature, enhancement, bug fix, behavioral change, refactor, or technical modification before implementation.

The skill must determine what existing implementation the request touches, which modules and layers are affected, what direct and indirect dependencies exist, what runtime behavior may change, what regressions are possible, what upgrade or data-migration consequences may exist, and where the smallest safe implementation boundary should be.

This skill is impact-analysis and planning only by default.

Do not modify files, create implementation files, change manifests, install or upgrade modules, update the database, run migrations, refactor code, implement the feature, commit, push, merge, or create a pull request during the impact-analysis phase.

Core principles:

- Understand the requested change precisely.
- Reuse existing investigation evidence when available.
- Trace the current implementation before predicting impact.
- Analyze direct and transitive effects.
- Separate confirmed impact from possible impact.
- Minimize blast radius.
- Prefer extension over replacement.
- Protect Odoo upgradeability.
- Identify regression tests before implementation.
- Produce the impact report before modifying anything.

---

## 1. Determine the Requested Change

Determine exactly what the user wants changed.

The request may involve:

- a new feature;
- a modification to an existing feature;
- a bug fix;
- a UI change;
- a workflow change;
- a business-rule change;
- a field;
- a model;
- a method;
- a controller;
- an HTTP/JSON route;
- an XML/QWeb view;
- an OWL component;
- JavaScript behavior;
- a website or portal page;
- an eCommerce flow;
- a payment flow;
- a report;
- an email;
- an automation;
- security;
- access rights;
- localization;
- performance;
- data import/export;
- integration behavior;
- an Odoo upgrade adaptation;
- a refactor.

If the request is already clear, do not ask unnecessary questions.

Normalize the request into:

```text
Requested change:
Current behavior:
Desired behavior:
Affected project:
Known module, if any:
Known page/workflow, if any:
Business importance:
Explicit constraints:
```

Do not begin impact analysis from a vague phrase when the surrounding code or request provides enough information to resolve it.

---

## 2. Relationship With the Odoo Codebase Investigator

This skill is not a replacement for the Odoo Codebase Investigator.

The Codebase Investigator answers questions such as:

```text
Where is the feature implemented?
Which module owns it?
What inherits or overrides it?
How does the feature execute end-to-end?
```

The Feature Impact Analyzer answers:

```text
If we change this feature, what else can be affected?
What can break?
What data, security, UI, integration, upgrade, or regression risks exist?
What is the safest implementation boundary?
What must be tested after implementation?
```

If a recent reliable Codebase Investigation Report already exists, reuse it.

If no reliable investigation exists, perform the minimum read-only investigation required to establish ownership, inheritance, dependencies, and execution flow before analyzing impact.

Never predict impact from filenames or a single matching source file alone.

---

## 3. Mandatory Startup Procedure

Follow this sequence:

1. Determine the requested change.
2. Determine the project and repository.
3. Detect the Odoo version.
4. Locate the relevant customer addons.
5. Reuse or perform codebase investigation.
6. Identify the original feature owner.
7. Identify customer/custom extensions.
8. Trace current execution flow.
9. Build the dependency graph.
10. Identify all affected technical layers.
11. Analyze direct impact.
12. Analyze indirect and transitive impact.
13. Analyze data and upgrade impact.
14. Analyze security and access impact.
15. Analyze frontend and asset impact.
16. Analyze integrations and contracts.
17. Analyze performance impact.
18. Analyze regression surface.
19. Compare implementation options.
20. Recommend the smallest safe implementation scope.
21. Define required validation and tests.
22. Produce the Feature Impact Analysis Report.
23. Wait for an explicit implementation request before modifying code.

---

## 4. Detect the Odoo Version

Before making architectural or compatibility assumptions, determine the actual Odoo version.

Inspect strong evidence such as:

```text
odoo/release.py
odoo/version.py
odoo/__init__.py
version_info
Git branch
Docker configuration
requirements
project configuration
existing JavaScript imports
existing frontend APIs
```

Record:

```text
Odoo version:
Evidence:
Edition if known:
Frontend architecture:
```

Do not assume compatibility between Odoo versions.

A safe implementation in Odoo 17 may be incorrect in Odoo 18 or later.

---

## 5. Find the Repository and Project

Identify:

```text
Repository root:
Odoo source path:
Standard addon paths:
Enterprise addon path:
Customer project path:
Customer addon path:
Other configured addon paths:
```

Customer projects in this environment normally use:

```text
odoo/Customers/<project_name>/addons/
```

When a project name is known, inspect this path first for project-specific changes.

Do not assume the project folder itself is an Odoo module.

Verify modules using `__manifest__.py`.

---

## 6. Establish the Current Baseline

Before evaluating a requested change, establish how the feature works now.

Record:

```text
Original feature owner:
Customer extension:
Primary model:
Primary views/templates:
Primary methods:
Primary controllers/routes:
Primary JavaScript:
Primary assets:
Primary data records:
Current execution flow:
```

The baseline must be based on repository evidence and, when available, runtime/database evidence.

Do not analyze impact against an imagined baseline.

---

## 7. Identify the Change Entry Point

Determine the most likely implementation entry point.

Examples:

```text
XML inherited view
QWeb template
Python model extension
method override
computed field
controller override
route extension
OWL component patch
JavaScript module
SCSS/CSS
security record
scheduled action
server action
report template
mail template
integration adapter
```

Record:

```text
Primary candidate entry point:
Owning module:
Owning file:
Why it is central:
```

The entry point is not automatically the final recommended modification location.

---

## 8. Build the Change Surface Map

Create a map of every technical layer potentially affected.

Use categories such as:

```text
Manifest/dependencies
Python models
Fields
Computed fields
Constraints
Onchange logic
Business methods
Controllers
Routes
XML views
QWeb templates
OWL templates
JavaScript
Registries/services/patches
Assets
CSS/SCSS
Security
Record rules
Access rights
Groups
Data XML/CSV
Scheduled actions
Server actions
Reports
Mail templates
Website
Portal
eCommerce
Payment
Accounting
Inventory
Sales
Purchases
Manufacturing
HR
Integrations
External APIs
Imports/exports
Database/data migration
Localization
Testing
Upgrade behavior
Performance
```

Classify each layer:

```text
Directly affected
Indirectly affected
Possible impact
No relevant impact found
Needs runtime verification
```

---

## 9. Identify Direct Impact

Direct impact means the requested change intentionally modifies behavior in a specific component.

Examples:

```text
Adding a field changes a model schema.
Changing an XPath changes a rendered view.
Changing a route response changes the frontend contract.
Changing a compute method changes a stored or displayed value.
Changing a validation rule changes accepted transactions.
```

For every direct impact record:

```text
Component:
Module:
File:
Current behavior:
Requested behavior:
Direct consequence:
```

---

## 10. Identify Indirect and Transitive Impact

A feature may affect components that are not edited directly.

Search for:

- callers;
- overrides;
- inherited views;
- dependent computed fields;
- automated actions;
- reports;
- exports;
- API consumers;
- frontend consumers;
- integration consumers;
- downstream business workflows.

Example:

```text
Field A
    ↓ used by
Compute B
    ↓ used by
Report C
    ↓ consumed by
External export D
```

Report these chains explicitly.

Do not stop after identifying direct dependencies.

---

# MODULE AND DEPENDENCY IMPACT

## 11. Analyze Manifest Impact

Inspect relevant `__manifest__.py` files.

Determine whether the requested change may require:

- a new dependency;
- removal of a dependency;
- new data files;
- new assets;
- sequencing changes;
- installability changes;
- version bump;
- post-init or migration logic.

Do not modify the manifest during analysis.

Record:

```text
Module:
Current depends:
Potential dependency change:
Reason:
Risk:
```

---

## 12. Analyze Direct Dependencies

Identify modules explicitly required by relevant modules.

Example:

```text
customer_feature
    ↓ depends on
website_sale
    ↓ depends on
sale
```

Determine which dependency behavior the planned change relies on.

---

## 13. Analyze Reverse Dependencies

Search for modules that depend on the module being changed.

A change inside a shared custom module may affect several customer features.

Record:

```text
Changed module:
Reverse-dependent modules:
Shared usage:
Potential blast radius:
```

Do not treat a module as isolated simply because the request names only one feature.

---

## 14. Detect Shared Modules

Classify the target module as:

```text
Customer-specific
Shared custom
Theme
Integration
Utility
Odoo standard
Enterprise
```

Changes to shared custom modules require broader regression analysis than changes isolated to one customer-specific module.

---

## 15. Detect Missing or New Dependency Requirements

If the proposed implementation would reference:

- another module's external ID;
- model;
- field;
- view;
- template;
- controller;
- JS module;
- service;
- asset;

verify whether the required module dependency already exists.

Report:

```text
Required dependency:
Currently declared:
Why needed:
Install/upgrade risk:
```

Do not silently recommend code that would introduce an undeclared dependency.

---

# PYTHON MODEL IMPACT

## 16. Analyze Model Ownership

For every affected model determine:

```text
Model:
Original module:
Custom extensions:
Files:
Fields involved:
Methods involved:
```

Trace `_inherit`, `_name`, and `_inherits`.

---

## 17. Analyze Field Schema Impact

For every added, changed, or removed field evaluate:

- field type;
- required state;
- default;
- readonly behavior;
- index;
- relation;
- inverse field;
- domain;
- context;
- tracking;
- company dependency;
- translation behavior;
- storage;
- copy behavior;
- compute;
- inverse;
- search method.

Record:

```text
Field:
Current definition:
Proposed impact:
Existing data consequence:
Upgrade consequence:
```

---

## 18. Analyze Stored Computed Fields

Stored computed fields are high-impact.

Inspect:

```text
compute=
store=True
@api.depends
@api.depends_context
inverse=
search=
```

Determine:

- what triggers recomputation;
- whether dependency changes alter recomputation;
- whether existing records need recomputation;
- whether module upgrade is sufficient;
- whether a migration or explicit recompute may be required;
- possible performance impact.

Do not treat a compute-method edit as a normal method change when `store=True`.

---

## 19. Analyze Related Fields

For related fields inspect:

```text
related=
store=
readonly=
company_dependent=
```

Determine whether changes to the source field propagate safely.

Check whether reports, domains, search, grouping, or integrations depend on the related field.

---

## 20. Analyze Constraints

Inspect:

```text
@api.constrains
_sql_constraints
required fields
database constraints
business validation
```

Determine whether the requested change may make existing records invalid.

Record:

```text
Constraint:
Affected records:
Install/upgrade risk:
User workflow impact:
Migration need:
```

---

## 21. Analyze Onchange and Client-Side Form Behavior

Inspect:

```text
@api.onchange
view modifiers
domains
context defaults
JS form logic
```

Determine whether server-side and client-side behavior remain consistent.

A change may work during record creation but fail during imports, RPC calls, automated actions, or batch updates if logic exists only in `onchange`.

Report this risk.

---

## 22. Analyze Method Overrides

For every affected method:

1. locate the original implementation;
2. find all relevant overrides;
3. inspect `super()`;
4. identify callers;
5. determine return contracts;
6. identify context assumptions.

Record:

```text
Method:
Original owner:
Overrides:
Callers:
Return contract:
Context assumptions:
Potential impact:
```

---

## 23. Analyze `super()` Chain Impact

Changing one method in an inheritance chain can alter several modules.

Example:

```text
Base implementation
    ↓
Theme/custom override
    ↓
Customer override
```

Determine:

- execution order;
- whether every override calls `super()`;
- whether arguments are preserved;
- whether return values are modified;
- whether context is changed.

Flag overrides that bypass parent behavior.

---

## 24. Analyze CRUD Lifecycle Impact

When relevant inspect overrides of:

```text
create()
write()
unlink()
copy()
name_get()/display_name behavior when version-appropriate
search()
search_read()
read_group()
default_get()
```

Determine whether the proposed change affects:

- imports;
- mass editing;
- automated jobs;
- integrations;
- record duplication;
- deletion;
- reporting;
- search performance.

---

## 25. Analyze Context-Dependent Behavior

Search for relevant use of:

```text
self.env.context
with_context()
allowed_company_ids
active_id
active_ids
active_model
lang
tz
website_id
pricelist
force_company
```

A feature that behaves correctly in one UI context may behave differently in scheduled jobs, RPC, portal, website, or multi-company flows.

---

# SECURITY IMPACT

## 26. Analyze Access Rights

Inspect:

```text
ir.model.access.csv
access records
groups
sudo()
```

Determine whether the requested feature introduces or exposes:

- a new model;
- a new field;
- a new route;
- a new action;
- a new workflow;
- privileged behavior.

Record:

```text
Model/action:
Current access:
Required access:
Potential security impact:
```

---

## 27. Analyze Record Rules

Inspect relevant `ir.rule` records.

Determine whether the feature behaves differently by:

- company;
- salesperson;
- portal user;
- website user;
- employee;
- manager;
- public user;
- custom security group.

A new search or route may unexpectedly return fewer or more records because of record rules.

---

## 28. Analyze `sudo()` Usage

If current or proposed logic uses `sudo()`:

- identify why;
- identify what permissions are bypassed;
- identify whether user-provided record IDs are involved;
- identify whether data leakage is possible;
- identify whether a safer scoped elevation exists.

Do not recommend `sudo()` merely to make access errors disappear.

---

## 29. Analyze Public and Portal Security

For website/portal routes inspect:

```text
auth="public"
auth="user"
website=True
csrf
HTTP method
route parameters
record access
tokens
sudo()
```

Determine whether the change affects authentication, authorization, or exposure of private data.

---

## 30. Analyze Multi-Company Impact

If affected models are company-aware, inspect:

```text
company_id
company_ids
check_company
company_dependent
allowed_company_ids
record rules
property fields
```

Determine whether the change behaves correctly when:

- multiple companies are active;
- records belong to another company;
- configuration differs by company.

Report multi-company behavior explicitly when relevant.

---

## 31. Analyze Multi-Website Impact

For website features inspect:

```text
website_id
website-specific views
website-specific settings
domains
request.website
website context
```

Determine whether the change is:

```text
Global
Website-specific
Potentially leaking across websites
Needs runtime verification
```

---

# XML / QWEB / UI IMPACT

## 32. Analyze XML Inheritance Impact

Whenever XML/QWeb is involved, trace:

```text
Original view
    ↓
Inherited view
    ↓
Further inherited views
```

Analyze both upward and downward inheritance.

Determine whether the proposed XPath target is stable.

---

## 33. Analyze XPath Blast Radius

For the element being changed, search all relevant inherited views that target:

- the same node;
- the same attribute;
- neighboring structure;
- descendants that depend on the node.

Record:

```text
XPath:
Current target:
Other views touching target:
Potential conflict:
Risk level:
```

---

## 34. Analyze `replace` Impact

Treat:

```text
position="replace"
```

as higher risk than small additive inheritance.

Determine whether replacement removes:

- expected classes;
- data attributes;
- QWeb variables;
- JS hooks;
- child extension points;
- accessibility attributes;
- downstream XPath targets.

Prefer preserving extension points where possible.

---

## 35. Analyze Attribute Changes

For:

```text
position="attributes"
```

check whether changes to:

- class;
- name;
- id;
- invisible;
- readonly;
- required;
- groups;
- domain;
- context;
- data attributes;

affect other XML, JS, CSS, security, or behavior.

---

## 36. Analyze View Priority

When multiple inherited views touch the same structure, inspect priority where relevant.

Report whether the planned change relies on execution order.

Do not assume file order controls final rendering.

---

## 37. Analyze Database-Only View Risk

Repository XML may not represent the final runtime UI.

Consider:

```text
Website Editor
Odoo Studio
database-created ir.ui.view
inactive/active view state
website-specific views
```

If the planned change depends on final rendered structure and repository evidence is incomplete, mark runtime/database verification as required.

---

# JAVASCRIPT / OWL IMPACT

## 38. Analyze Frontend Ownership

Identify:

```text
JS entry file:
Component/widget:
Registry:
Service:
Patch:
Template:
Selectors:
Data attributes:
RPC/HTTP calls:
Asset bundle:
```

Do not recommend a JS change before confirming the responsible component.

---

## 39. Analyze Component and Patch Impact

Inspect version-appropriate mechanisms such as:

```text
patch()
registry.category(...)
service overrides
OWL components
publicWidget
legacy include()/extend()
```

Determine whether the planned change affects:

- all component instances;
- only one registry entry;
- backend;
- website;
- portal;
- POS;
- other bundles.

---

## 40. Analyze JavaScript Selectors

If behavior depends on:

```text
querySelector
querySelectorAll
closest
dataset
CSS classes
DOM IDs
t-ref
```

check whether XML changes alter those selectors.

A harmless-looking template change may break JS silently.

---

## 41. Analyze Frontend State and Events

Inspect:

- event listeners;
- component state;
- lifecycle hooks;
- services;
- bus events;
- DOM events;
- debounce/throttle;
- async operations.

Determine whether the requested change introduces race conditions or stale state.

---

## 42. Analyze RPC and HTTP Contracts

For every affected frontend request record:

```text
Caller:
Route/model method:
Payload:
Authentication:
Response:
Error contract:
Consumers:
```

If the response shape changes, find every consumer.

Do not change a JSON contract without identifying all consumers.

---

# CONTROLLER AND ROUTE IMPACT

## 43. Analyze Route Ownership

For affected routes record:

```text
Route:
Original module:
Controller:
Inherited controllers:
auth:
type:
methods:
csrf:
website:
```

---

## 44. Analyze Route Override Impact

Determine whether the requested change:

- overrides an existing route;
- redeclares a route;
- changes parameters;
- changes authentication;
- changes return type;
- changes redirects;
- changes JSON structure.

Search for frontend and external consumers.

---

## 45. Analyze External Contract Compatibility

If a route/API is used by external systems, classify the change as:

```text
Backward compatible
Potentially breaking
Breaking
Unknown until consumer verification
```

Prefer additive response changes over destructive contract changes where practical.

---

# ASSET IMPACT

## 46. Analyze Asset Bundles

Inspect:

```text
__manifest__.py
asset declarations
legacy asset XML when applicable
glob patterns
bundle directives
```

Determine:

- which bundles are affected;
- whether a new file is automatically included;
- whether duplicate inclusion may occur;
- whether bundle order matters.

---

## 47. Analyze Asset Load Order

When patching or overriding frontend behavior, determine whether the custom asset loads after the original implementation.

Check version-appropriate ordering mechanisms.

Report load-order assumptions explicitly.

---

## 48. Analyze CSS / SCSS Impact

When changing classes, DOM structure, or component markup, search CSS/SCSS consumers.

Determine whether styling depends on:

- exact nesting;
- class names;
- IDs;
- pseudo-selectors;
- RTL selectors;
- theme variables.

A structural UI change can break styling even when the XPath succeeds.

---

# DATA AND UPGRADE IMPACT

## 49. Analyze Existing Data Impact

For each schema or business-rule change determine whether existing records:

```text
Remain valid
Need recomputation
Need backfill
Need migration
Need normalization
May become invalid
Need manual review
```

Do not analyze only new-record behavior.

---

## 50. Analyze Module Install Impact

Determine whether the feature affects fresh installation.

Check:

- manifest dependencies;
- data file order;
- defaults;
- required fields;
- XML references;
- demo data if relevant;
- external IDs.

---

## 51. Analyze Module Upgrade Impact

Determine what happens during:

```text
-u <module>
```

Consider:

- new fields;
- changed stored compute logic;
- changed XML data;
- noupdate records;
- deleted XML IDs;
- renamed fields;
- renamed models;
- renamed external IDs;
- changed constraints;
- changed security;
- data files.

Do not assume a module upgrade will automatically migrate every semantic change.

---

## 52. Analyze `noupdate` Records

If the change touches records loaded with `noupdate="1"`, determine whether module upgrade will update them.

Report when manual migration or another safe strategy may be required.

---

## 53. Analyze Migration Need

Classify migration impact:

```text
None
Recompute only
Data backfill
Data transformation
External ID migration
Schema migration
Manual operational step
Needs further investigation
```

Do not write migration scripts during impact analysis.

---

## 54. Analyze Data Volume

For recomputes, updates, or migrations estimate impact based on available evidence:

```text
Affected model:
Approximate record volume if known:
Operation:
Potential lock/time risk:
Batching concern:
```

Do not claim exact timing without measurement.

---

# BUSINESS WORKFLOW IMPACT

## 55. Trace Upstream Inputs

Identify what feeds the changed behavior.

Examples:

```text
Configuration
User input
Imported data
API data
Computed values
Company settings
Website settings
Pricelists
Fiscal positions
Products
Partners
Stock
Accounting
```

---

## 56. Trace Downstream Outputs

Identify what consumes the changed result.

Examples:

```text
Orders
Invoices
Stock moves
Payments
Reports
Emails
Portal pages
Exports
Dashboards
Integrations
Accounting entries
Analytics
```

---

## 57. Analyze Cross-Application Impact

Odoo applications are connected.

A Sales change may affect:

```text
Inventory
Accounting
Website
Purchase
Manufacturing
Subscriptions
Helpdesk
Projects
```

Only report cross-application impact supported by actual dependencies or execution flow.

Do not list generic Odoo apps without evidence.

---

## 58. Analyze Automated Processes

Search for:

```text
ir.cron
server actions
automated actions
queue jobs
scheduled jobs
webhooks
mail activities
background processing
```

Determine whether these processes call affected methods or depend on changed data.

---

## 59. Analyze Reporting Impact

Search:

- QWeb reports;
- spreadsheet exports;
- XLSX/CSV exports;
- dashboards;
- pivot/group-by use;
- custom report models;
- SQL views.

Determine whether changed fields or logic alter reports.

---

## 60. Analyze Mail and Notification Impact

Search:

```text
mail.template
message_post
activities
notifications
email templates
SMS/WhatsApp integrations if present
```

Determine whether business-state or field changes alter communications.

---

# INTEGRATION IMPACT

## 61. Identify External Integrations

Search for relevant:

- REST APIs;
- SOAP;
- GraphQL;
- payment gateways;
- shipping providers;
- accounting integrations;
- webhooks;
- message queues;
- external database access;
- SFTP/files;
- vendor/customer APIs.

---

## 62. Analyze Integration Contracts

For affected integrations identify:

```text
Input contract:
Output contract:
Authentication:
Identifiers:
Required fields:
Optional fields:
Error behavior:
Retry behavior:
Idempotency:
```

Determine whether the requested change is backward compatible.

---

## 63. Analyze Identifier Stability

Do not casually change:

```text
external IDs
route names
field technical names
model names
selection keys
API keys in payloads
XML IDs
JS registry keys
template names
```

These may be consumed outside the visible feature.

---

# PERFORMANCE IMPACT

## 64. Analyze ORM Query Impact

Inspect proposed logic for risks such as:

```text
search() inside loops
browse/search duplication
N+1 queries
unbounded searches
large recordsets
repeated computed calls
missing indexes
expensive read_group
```

Report likely performance concerns before implementation.

---

## 65. Analyze Compute Performance

For compute methods determine:

- dependency frequency;
- record volume;
- `store=True` consequences;
- recomputation fan-out;
- cross-model dependencies.

A small dependency change can trigger large recomputations.

---

## 66. Analyze Frontend Performance

Consider:

- additional RPC calls;
- repeated DOM scans;
- large payloads;
- duplicated assets;
- heavy OWL rerenders;
- expensive listeners;
- unnecessary global bundle inclusion.

---

## 67. Analyze Caching Impact

When relevant inspect:

- browser cache;
- asset cache;
- Odoo cache;
- computed values;
- integration cache;
- website page caching behavior.

Report whether deployment requires asset rebuild/reload or cache invalidation.

---

# COMPATIBILITY AND UPGRADEABILITY

## 68. Prefer Upgrade-Safe Extension

Prefer:

- custom module extension over Odoo core modification;
- inherited XML over copied full views;
- `super()`-preserving model extension;
- supported patches/registries over monkey patching;
- stable Odoo APIs over private internals;
- additive APIs over destructive contract changes.

---

## 69. Detect Core Modification Risk

If the easiest implementation appears to require editing:

```text
odoo/addons/
odoo core
enterprise core
```

search for a custom-extension approach first.

Classify direct core modification as high upgrade risk unless there is a verified exceptional reason.

---

## 70. Analyze Odoo-Version Compatibility

For any proposed API, import, hook, registry, or view behavior:

- confirm it exists in the detected version;
- search current repository usage;
- avoid copying patterns from another Odoo version blindly.

Record version-sensitive assumptions.

---

## 71. Analyze Community vs Enterprise Dependency

If the feature depends on Enterprise code, state it.

Do not recommend Enterprise-only inheritance when the deployment may be Community-only.

---

# REGRESSION ANALYSIS

## 72. Build the Regression Surface

List everything that must continue to work after the change.

Example:

```text
Primary feature
Existing alternate path
Portal path
Website path
Backend path
Multi-company behavior
Security restrictions
Imports
Exports
Reports
API consumers
Scheduled jobs
Install
Upgrade
Assets
Localization
```

---

## 73. Identify Regression Scenarios

Create concrete scenarios, not generic statements.

Example:

```text
Scenario:
Public user submits form without optional phone field.

Current expected behavior:
Form succeeds.

Regression risk:
New validation may reject public submissions.

Required test:
Submit with and without phone in public website context.
```

---

## 74. Prioritize Regression Risk

Use a practical level such as:

```text
Critical
High
Medium
Low
```

Consider:

- financial impact;
- data corruption;
- security exposure;
- workflow blockage;
- number of users;
- shared module blast radius;
- upgrade/install failure;
- silent incorrect behavior.

Explain why a risk has its level.

---

## 75. Separate Severity From Confidence

Risk severity and evidence confidence are different.

Use:

```text
Risk severity:
Confidence:
```

Confidence may be:

```text
Confirmed
Strong evidence
Probable
Possible
Needs runtime verification
```

Do not downgrade a severe risk merely because runtime verification is still needed.

---

# IMPLEMENTATION OPTION ANALYSIS

## 76. Identify Candidate Implementation Strategies

When more than one safe approach exists, compare them.

Example:

```text
Option A: Extend existing customer module.
Option B: Create a small dedicated addon.
Option C: Modify shared custom module.
Option D: Patch frontend component.
```

---

## 77. Compare Options

For each option evaluate:

```text
Scope:
Files/modules touched:
Upgradeability:
Regression risk:
Dependency impact:
Data impact:
Complexity:
Test burden:
Reusability:
Reason to choose/reject:
```

Do not choose solely by lowest line count.

---

## 78. Prefer the Smallest Safe Change

The preferred solution should minimize:

- modules changed;
- duplicated logic;
- overridden code;
- replaced XML;
- global side effects;
- migration complexity;
- new dependencies;
- regression surface.

Smallest does not mean unsafe shortcut.

---

## 79. Recommend the Implementation Boundary

State clearly:

```text
Recommended module:
Recommended files:
Recommended extension point:
Expected files not to touch:
Reason:
```

If a new module is preferable, explain why.

Do not create it during impact analysis.

---

## 80. Identify Files and Modules That Should Not Be Modified

Protect:

- Odoo core;
- Enterprise source;
- unrelated customer modules;
- shared modules when a customer-specific extension is sufficient;
- generated assets;
- unrelated translation files;
- unrelated data.

Explicitly list important no-touch areas.

---

# TEST AND VALIDATION PLANNING

## 81. Discover Existing Tests

Search relevant modules for:

```text
tests/
SavepointCase
TransactionCase
HttpCase
browser tests
tour tests
QUnit
HOOT
pytest
custom scripts
```

Record what existing coverage already protects the feature.

---

## 82. Define Required Automated Tests

Recommend tests for:

- business logic;
- constraints;
- computed fields;
- access rights;
- routes;
- JSON contracts;
- UI behavior;
- installation/upgrade where practical.

Do not write tests during impact analysis unless explicitly requested separately.

---

## 83. Define Required Manual Tests

When runtime behavior matters, define representative manual checks.

Examples:

```text
Backend administrator
Normal internal user
Portal user
Public user
Arabic website
English website
Company A
Company B
Website A
Website B
```

Only include contexts relevant to the feature.

---

## 84. Define Install/Upgrade Validation

When module data/schema changes, include:

```text
Fresh install test
Upgrade existing database
Existing record validation
Stored compute recompute verification
Security load verification
Asset bundle verification
```

---

## 85. Define Browser and Asset Validation

For frontend changes include:

```text
Asset compilation
Browser console errors
Network requests
Template rendering
Selector behavior
Mobile/responsive behavior when relevant
RTL behavior when relevant
```

---

## 86. Define Data Validation

If existing records are affected, specify how to confirm:

```text
No records lost
No invalid states introduced
No unexpected defaults
Correct recomputation
Correct historical values
Correct reports
```

---

# OPERATIONAL AND DEPLOYMENT IMPACT

## 87. Analyze Deployment Requirements

Determine whether implementation may require:

```text
Module upgrade
Server restart
Asset rebuild
Browser hard refresh
Data migration
Scheduled maintenance
Configuration change
External integration coordination
```

Do not perform these actions during analysis.

---

## 88. Analyze Rollback Complexity

For each recommended strategy determine whether rollback is:

```text
Code-only
Code + module upgrade
Code + data rollback
Difficult because of irreversible data transformation
```

Flag irreversible changes clearly.

---

## 89. Analyze Configuration Dependency

Identify settings required for the feature:

```text
System parameters
Company configuration
Website settings
Access groups
Payment provider configuration
Warehouse configuration
Localization configuration
```

Distinguish code impact from deployment/configuration impact.

---

# EVIDENCE AND CONFIDENCE

## 90. Evidence Levels

For important conclusions use:

```text
Confirmed by repository
Confirmed by runtime/database
Strong evidence
Probable
Possible
Needs runtime verification
```

Do not present a theoretical impact as confirmed.

---

## 91. Separate Observed Behavior From Predicted Impact

Use language such as:

```text
Observed:
The route returns keys A, B, and C.

Predicted impact:
Removing B would likely break booking_form.js because it reads response.B.
```

This separation is mandatory for important conclusions.

---

## 92. Record Unknowns

Examples:

```text
Installed module state unknown
Database-only inherited views unknown
Production record volume unknown
External API consumers unknown
Website Editor changes unknown
Company-specific configuration unknown
```

Do not invent certainty.

---

# SAFETY AND SCOPE

## 93. Read-Only by Default

Do not:

- edit files;
- create implementation files;
- delete files;
- change manifests;
- modify XML;
- modify Python;
- modify JavaScript;
- modify SCSS;
- modify security;
- update data files;
- update translations;
- install modules;
- upgrade modules;
- update the database;
- run migrations;
- commit;
- push;
- merge;
- rebase;
- create pull requests.

Implementation starts only after an explicit user instruction.

---

## 94. Safe Read-Only Commands

Examples:

```text
find
grep
rg
ls
tree
cat
head
tail
sed for reading
git status
git diff
git log
git show
```

Use non-destructive inspection commands.

---

## 95. Do Not Implement Even If the Solution Is Obvious

If the requested change is straightforward, still report:

```text
Current implementation:
Impact:
Risks:
Recommended implementation location:
Required tests:
```

Do not apply the change during impact analysis.

---

## 96. Do Not Expand Scope Silently

If the safest solution requires changing another module, report that dependency.

Do not edit it automatically.

If a shared module would broaden blast radius, prefer a customer-specific extension when technically sound.

---

# REQUIRED REPORT

## 97. Odoo Feature Impact Analysis Report

Always produce a report before implementation.

Use this structure.

### 1. Requested Change

```text
Project:
Feature:
Current behavior:
Desired behavior:
Known module:
Constraints:
```

### 2. Environment

```text
Repository root:
Odoo version:
Version evidence:
Edition:
Customer addon path:
Relevant addon paths:
```

### 3. Current Implementation Baseline

```text
Original feature owner:
Customer extension:
Primary model:
Primary view/template:
Primary methods:
Primary controller/route:
Primary JS:
Primary assets:
```

Current flow:

```text
UI
    ↓
JavaScript
    ↓
Route
    ↓
Controller
    ↓
Model
    ↓
Data
```

Adapt the flow to the actual feature.

### 4. Relevant Modules

For each module:

```text
Module:
Type:
Path:
Purpose:
Depends:
Relationship to change:
```

### 5. Dependency Graph

Example:

```text
sale
    ↓
website_sale
    ↓
theme_module
    ↓
customer_feature
```

Also include reverse dependencies when relevant.

### 6. Feature Ownership

```text
Original owner:
UI owner:
XML/QWeb owner:
Model owner:
Controller owner:
JavaScript owner:
Asset owner:
Security owner:
```

### 7. Change Surface Map

Use a table or structured list:

```text
Layer:
Component:
Impact classification:
Reason:
Confidence:
```

### 8. Direct Impact

```text
Component:
Current behavior:
Requested change:
Direct consequence:
```

### 9. Indirect / Transitive Impact

Show chains such as:

```text
Changed field
    ↓
Compute method
    ↓
Report
    ↓
External export
```

### 10. Model and Field Impact

```text
Model:
Field:
Schema impact:
Compute impact:
Constraint impact:
Existing-data impact:
```

### 11. Method / Business Logic Impact

```text
Method:
Original owner:
Overrides:
Callers:
super() chain:
Return contract:
Predicted impact:
```

### 12. XML / QWeb Impact

```text
Original view:
Inherited views:
XPath target:
Priority:
Other views touching target:
Conflict risk:
```

### 13. JavaScript / OWL Impact

```text
Entry file:
Component/widget:
Selectors:
Templates:
Patches:
RPC calls:
State/events:
Asset bundle:
```

### 14. Controller / Route Impact

```text
Route:
Controller:
Overrides:
auth/type:
Payload:
Response contract:
Consumers:
Compatibility risk:
```

### 15. Asset Impact

```text
Bundle:
Files:
Load order:
Duplicate inclusion risk:
Compilation impact:
```

### 16. Security Impact

```text
ACL:
Record rules:
Groups:
sudo():
Public/portal exposure:
Multi-company:
```

### 17. Data and Migration Impact

```text
Existing records:
Stored computes:
Constraints:
Backfill:
Migration:
noupdate:
Upgrade behavior:
```

### 18. Integration Impact

```text
Integration:
Contract:
Consumer:
Backward compatibility:
Coordination needed:
```

### 19. Performance Impact

```text
ORM:
Compute:
Frontend:
Payload:
Asset:
Expected concern:
```

Do not claim measured performance without measurement.

### 20. Cross-Workflow Impact

List actual connected workflows supported by evidence.

### 21. Regression Surface

List everything that must remain functional.

### 22. Risk Register

For every material risk:

```text
Risk:
Severity:
Confidence:
Evidence:
Consequence:
Mitigation:
Required test:
```

### 23. Implementation Options

```text
Option A:
Scope:
Pros:
Cons:
Risk:
Upgradeability:

Option B:
...
```

### 24. Recommended Implementation Boundary

```text
Recommended module:
Recommended files:
Recommended extension point:
Why:
Modules/files not to modify:
```

### 25. Validation Plan

```text
Automated tests:
Manual tests:
Install test:
Upgrade test:
Browser test:
Security test:
Data validation:
Integration validation:
```

### 26. Deployment Impact

```text
Module upgrade:
Restart:
Assets:
Migration:
Configuration:
External coordination:
```

### 27. Rollback Considerations

```text
Rollback type:
Data risk:
Irreversible actions:
```

### 28. Existing Tests Found

List relevant current tests.

### 29. Files Investigated

List important files actually examined.

### 30. Remaining Unknowns

List anything requiring:

```text
runtime access
database access
production configuration
integration consumer verification
record-volume measurement
browser verification
```

### 31. Decision

End with one of:

```text
Safe to implement with low expected blast radius.
Safe to implement with listed precautions.
Implementation should wait for runtime/database verification.
Implementation should wait for dependency/integration clarification.
High-risk change; prefer alternative implementation strategy.
```

Explain the decision briefly.

---

## 98. Risk Register Is Mandatory

Every impact analysis must include a risk register.

At minimum evaluate:

```text
Regression risk
Data risk
Security risk
Upgrade risk
Integration risk
Performance risk
Deployment risk
```

Use `No relevant risk found` when evidence supports it.

Do not omit a category silently.

---

## 99. Change Surface Map Is Mandatory

Always summarize impact across relevant layers.

Example:

```text
Python model: Directly affected
XML view: Directly affected
JavaScript: No relevant impact found
Controller: No relevant impact found
Security: Possible impact
Existing data: Needs upgrade verification
```

This prevents analysis from focusing only on the file that will be edited.

---

## 100. Regression Plan Is Mandatory

Before implementation, define what must be tested.

Do not finish with only:

```text
Run tests.
```

Specify concrete scenarios based on the actual feature.

---

## 101. Data / Upgrade Analysis Is Mandatory for Schema or Stored Logic Changes

If the requested change involves:

- fields;
- field types;
- defaults;
- required flags;
- constraints;
- stored computes;
- related stored fields;
- XML data;
- security records;

explicitly analyze existing databases and module-upgrade behavior.

---

## 102. Security Analysis Is Mandatory for Routes or Data Exposure

If the feature changes:

- controller routes;
- public website behavior;
- portal behavior;
- record retrieval;
- file access;
- `sudo()`;
- user-provided record IDs;
- access groups;

include explicit authentication and authorization impact analysis.

---

## 103. External Contract Analysis Is Mandatory for Integrations

If external consumers exist or may exist, do not approve a breaking contract change without identifying them.

When consumer visibility is incomplete, mark:

```text
Needs integration-consumer verification
```

---

## 104. Core Questions to Answer

Before finishing, answer as many as relevant:

1. What exactly is changing?
2. What is the current behavior?
3. What Odoo version is used?
4. Which module originally owns the feature?
5. Which customer/custom module currently extends it?
6. What is the current end-to-end execution flow?
7. What is the safest implementation entry point?
8. Which modules depend on the target module?
9. Which modules are reverse-dependent on it?
10. Is the module shared across features or customers?
11. Which models are affected?
12. Which fields are affected?
13. Are any fields stored computed fields?
14. Will existing records require recomputation?
15. Will existing records become invalid?
16. Are constraints affected?
17. Are create/write/unlink flows affected?
18. Are imports or integrations affected?
19. Which methods are overridden?
20. What is the `super()` chain?
21. Which callers depend on return values?
22. Which XML/QWeb views are affected?
23. Which other views target the same nodes?
24. Are XPath operations fragile?
25. Are JS selectors tied to changed markup?
26. Are OWL components or patches affected?
27. Which asset bundles are affected?
28. Does asset load order matter?
29. Which routes are affected?
30. Will request or response contracts change?
31. Are public or portal users affected?
32. Are ACLs or record rules affected?
33. Is `sudo()` involved?
34. Is multi-company behavior affected?
35. Is multi-website behavior affected?
36. Does the change touch external APIs?
37. Is the external contract backward compatible?
38. Is a data migration required?
39. Will module upgrade apply the change safely?
40. Are `noupdate` records involved?
41. Is performance likely to change?
42. What is the regression surface?
43. What existing tests protect the feature?
44. What new tests are required?
45. What deployment steps are required?
46. Is rollback simple or data-sensitive?
47. Which implementation options exist?
48. Which option has the smallest safe blast radius?
49. Which files/modules should not be modified?
50. What still requires runtime/database verification?

---

## 105. Hard Prohibitions

Never:

- analyze impact without understanding the current implementation;
- assume ownership from the first matching file;
- assume the Odoo version;
- modify Odoo core when a safe extension exists;
- recommend copying full views when inheritance is sufficient;
- recommend replacing templates without checking downstream inheritance;
- ignore reverse dependencies;
- ignore stored-compute recomputation;
- ignore existing-data impact;
- assume module upgrade performs every required migration;
- ignore `noupdate` behavior;
- ignore ACLs and record rules;
- recommend `sudo()` as a generic access fix;
- change a route contract without tracing consumers;
- change technical identifiers casually;
- ignore multi-company or multi-website behavior when relevant;
- ignore asset bundle/load-order behavior;
- ignore frontend selectors tied to XML;
- ignore external integrations;
- claim performance impact is measured when it was only inferred;
- claim a risk is impossible without evidence;
- skip regression planning;
- implement code during impact analysis;
- install or upgrade modules during impact analysis;
- update the database during impact analysis;
- commit or push without explicit instruction;
- claim runtime validation when only static analysis was performed.

---

## 106. Core Decision Tree

```text
START
  |
  v
Is the requested change clear?
  |
  +-- No --> Resolve from context or ask only if necessary.
  |
  +-- Yes
        |
        v
Determine project/repository.
        |
        v
Detect Odoo version.
        |
        v
Is there a reliable recent Codebase Investigation?
        |
        +-- Yes --> Reuse it and verify relevant facts.
        |
        +-- No --> Perform sufficient read-only investigation.
        |
        v
Establish current implementation baseline.
        |
        v
Identify original owner and customer extensions.
        |
        v
Build dependency and reverse-dependency graph.
        |
        v
Build change surface map.
        |
        v
Analyze direct impact.
        |
        v
Analyze indirect/transitive impact.
        |
        v
Does change affect models/fields?
        |
        +-- Yes --> Analyze schema, compute, constraints,
        |          existing data, install, upgrade, migration.
        |
        v
Does change affect XML/QWeb?
        |
        +-- Yes --> Trace inheritance, XPath targets,
        |          priority, downstream views, JS/CSS hooks.
        |
        v
Does change affect JS/OWL?
        |
        +-- Yes --> Analyze components, selectors, state,
        |          RPC, assets, load order.
        |
        v
Does change affect routes/data exposure?
        |
        +-- Yes --> Analyze auth, ACL, record rules,
        |          sudo, contract consumers.
        |
        v
Does change affect integrations?
        |
        +-- Yes --> Analyze backward compatibility.
        |
        v
Analyze performance.
        |
        v
Build regression surface and risk register.
        |
        v
Compare implementation options.
        |
        v
Recommend smallest safe implementation boundary.
        |
        v
Define tests, deployment, rollback.
        |
        v
Produce Feature Impact Analysis Report.
        |
        v
STOP.
Wait for explicit implementation instruction.
```

---

## 107. Primary Rules to Always Remember

```text
Analyze before implementing.

Understand the current feature before predicting impact.

Reuse the Codebase Investigator when reliable evidence already exists.

For customer projects, begin project-specific discovery from:

odoo/Customers/<project_name>/addons/

but always trace the feature to its original owner.

Detect the actual Odoo version.

Build both dependency and reverse-dependency relationships.

Analyze the full change surface, not only the edited file.

For Python:
analyze model inheritance, field schema, stored computes,
constraints, CRUD behavior, method overrides, callers, and super() chains.

For XML/QWeb:
trace inheritance upward and downward,
analyze XPath targets, priority, downstream child views,
and JS/CSS hooks.

For JavaScript/OWL:
trace components, patches, selectors, templates,
RPC calls, state, events, assets, and load order.

For controllers:
trace route ownership, overrides, auth, payloads,
responses, consumers, and external compatibility.

For security:
analyze ACLs, record rules, groups, sudo(),
public/portal exposure, multi-company, and multi-website behavior.

For data:
analyze existing records, recomputation, migration,
noupdate behavior, fresh install, and module upgrade.

For integrations:
protect external contracts and technical identifiers.

For performance:
inspect ORM, stored computes, frontend calls, payloads, and assets.

Always build:
- a Change Surface Map;
- a Risk Register;
- a Regression Plan;
- a Recommended Implementation Boundary.

Prefer the smallest safe extension.

Do not modify anything during impact analysis.

Always produce the impact report first.

Implementation requires an explicit user instruction.
```
