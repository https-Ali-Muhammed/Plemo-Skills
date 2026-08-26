# Odoo Regression & Runtime Validator

## Purpose

Use this skill to validate an Odoo change after implementation and determine, with evidence, whether the requested behavior works and whether relevant existing behavior still works.

The skill must validate only what the change and its impact evidence justify. It must distinguish static validation from runtime validation, feature verification from regression verification, fresh installation from module upgrade, repository correctness from database/runtime correctness, and confirmed results from unverified assumptions.

This skill is a **validation and evidence skill**.

It does not replace the agent's native planning, implementation, debugging, or fix workflow.

Its core question is:

```text
The code has changed.
What must now be proven before this change can be considered safe?
```

Core principles:

- Reuse existing investigation and impact evidence.
- Validate the actual diff, not an imagined change.
- Test the requested behavior.
- Test the regression surface supported by evidence.
- Use the smallest sufficient validation scope.
- Escalate validation depth when blast radius is larger.
- Separate static checks from runtime proof.
- Separate fresh install from upgrade behavior.
- Separate backend proof from browser/frontend proof.
- Separate security proof by user context.
- Separate pre-existing failures from failures caused by the change.
- Never claim a check passed if it was not run.
- Never treat "no traceback" as proof that behavior is correct.
- Return explicit PASS / FAIL / PARTIAL / BLOCKED evidence.

---

# 0. Native Agent Compatibility and Skill Scope

This skill extends Plemo's existing agent capabilities. It does not replace them.

The agent already performs native repository discovery, planning, implementation, targeted checks, final diff review, and ordinary debugging. This skill must not create a second generic planning system or a second implementation workflow.

Repository-specific instructions such as:

```text
plemo.md
configured addon paths
project conventions
customer restrictions
deployment conventions
available Plemo tools
```

take precedence over generic examples in this skill.

When reliable evidence already exists from:

```text
Odoo Codebase Investigator
Odoo Feature Impact Analyzer
Odoo Localization & Arabic QA
the current implementation task
the current Git diff
existing test output
```

reuse it.

Do not repeat a full investigation merely because this skill is activated.

Use the smallest applicable part of this skill for the actual change.

---

## 0.1 Relationship With the Previous Skills

The intended separation is:

```text
Odoo Codebase Investigator
    "What exists and how does it work?"
        ↓
Odoo Feature Impact Analyzer
    "What depends on this and what could break?"
        ↓
Agent native planning
        ↓
Agent native implementation
        ↓
Odoo Regression & Runtime Validator
    "What can we now prove works, and what remains unverified?"
```

This skill should consume the Feature Impact Analyzer's structured evidence when available.

Useful inputs include:

```text
Affected modules
Affected files/components
Reverse dependencies
Transitive chains
Runtime/dynamic dependencies
Upgrade/data risks
Security surface
Frontend/JS/RPC surface
Integration surface
Regression surface
Runtime verification requirements
Do-not-touch boundary
Remaining unknowns
```

Do not recreate these analyses unless implementation changed the impact surface or the existing evidence is stale/incomplete.

---

## 0.2 Task Mode and Fix Authorization

Determine the current task mode.

Common modes:

```text
Validation / Test / Review only
Fix + Validate
Implement + Validate
Full regression request
Specific runtime verification request
```

For **Validation / Test / Review only**:

- do not edit source files;
- do not silently fix failures;
- report failures with evidence;
- repository inspection remains read-only;
- prefer runtime validation in development, test, staging, cloned, or otherwise safe environments;
- when production is the only environment that can prove the behavior, allow only the narrowest safe read-only or clearly reversible verification permitted by repository/project rules;
- do not perform state-changing or business-impacting production validation without the authorization required by the project workflow.

For **Fix + Validate** or **Implement + Validate**:

- this skill owns the validation phase only;
- if validation fails, hand the failure evidence to the agent's native planning/debugging process;
- the original user request already authorizes the requested fix/implementation;
- do not ask for a second approval solely because validation found a defect;
- after the agent fixes the defect, rerun the failed validation and any tests whose assumptions changed.

Ask for new approval only when validation reveals:

- a destructive or irreversible operation;
- a material scope expansion;
- production-only data changes;
- external side effects not previously authorized;
- a security-sensitive decision;
- a migration strategy decision;
- another unresolved choice where proceeding would be unsafe.

---

## 0.3 Validation Materiality Gate

Do not force heavyweight regression work on trivial changes.

### Light validation is normally sufficient for isolated non-behavioral changes such as:

- comments;
- formatting-only changes;
- obvious text corrections;
- documentation changes;
- non-functional metadata where runtime behavior cannot change.

### Targeted validation is appropriate for contained behavioral changes such as:

- one form/view adjustment;
- one local model method change;
- one isolated controller behavior;
- one JS component change;
- one report/template change.

### Full material validation is required or strongly preferred for changes involving:

- model schema;
- stored computed fields;
- field type changes;
- required/default changes;
- constraints;
- create/write/unlink behavior;
- shared methods;
- methods with multiple overrides;
- accounting, stock, payment, payroll, or other high-impact business flows;
- inherited views with downstream child views;
- public/portal controllers;
- security/ACL/record-rule changes;
- multi-company behavior;
- multi-website behavior;
- JavaScript/OWL patches used broadly;
- asset bundle changes;
- Selection key changes;
- migrations or existing-data transformations;
- `noupdate` records;
- external integrations;
- shared custom modules;
- changes with uncertain blast radius.

Validation depth should follow risk, not line count.

---

# 1. Determine the Validation Target

Identify exactly what is being validated.

Record:

```text
Project:
Requested behavior:
Implemented change:
Target module(s):
Relevant workflow:
Acceptance criteria:
Known risk areas:
Requested validation depth:
```

If the user already defined acceptance criteria, preserve them.

Do not replace explicit business acceptance criteria with generic technical checks.

If acceptance criteria are not explicit, infer only what is strongly supported by the request and implementation evidence.

State inferred criteria clearly.

---

# 2. Determine the Actual Change Set

Before selecting tests, inspect the actual implementation.

Use:

```text
git status
git diff
git diff --stat
git diff --name-only
relevant commit diff
relevant branch diff
```

or equivalent safe repository tools.

Record:

```text
Changed files:
Added files:
Deleted files:
Manifest changes:
Migration files:
Security changes:
Asset changes:
Translation changes:
Unrelated/pre-existing dirty files:
```

Do not validate only the files the implementation plan expected to change.

Validate the files that actually changed.

---

## 2.1 Separate Task Changes From Pre-Existing Changes

A working tree may already contain changes unrelated to the current task.

Distinguish:

```text
Changed by this task
Pre-existing user/developer change
Unknown origin
Generated/runtime artifact
```

Do not revert, reformat, or "clean up" pre-existing work merely to simplify validation.

Do not attribute a pre-existing failure to the current implementation without evidence.

---

# 3. Reuse Impact Evidence

If an `IMPACT EVIDENCE` block exists, read it first.

Compare the final implementation against the predicted boundary.

Check:

```text
Did implementation stay inside the recommended safe boundary?
Did it touch any do-not-touch area?
Did it introduce a new dependency?
Did it add a new runtime consumer?
Did the final diff create a wider blast radius than originally analyzed?
Did implementation choices invalidate any previous test requirement?
```

If the implementation expanded materially beyond the prior impact analysis:

```text
Impact evidence status: STALE
```

Perform only the additional impact discovery needed for the new surface before continuing validation.

Do not blindly run the old regression plan against a materially different implementation.

---

# 4. Detect the Odoo Version

Determine the Odoo version from the strongest available evidence.

Valid evidence may include:

```text
target/custom module __manifest__.py version prefix
odoo/release.py
odoo/version.py
version_info
repository branch
Docker/build configuration
plemo.md
existing Odoo-version-specific APIs
```

A valid manifest version prefix is acceptable primary evidence when Odoo core source is not present.

Examples:

```text
17.0.1.0.0 -> Odoo 17
18.0.2.1.0 -> Odoo 18
19.0.1.0.0 -> Odoo 19
```

Record:

```text
Odoo version:
Version evidence:
Edition if known:
Runtime/build environment:
```

Do not run version-sensitive validation based on an assumed version.

---

# 5. Repository and Tool Precedence

Use this precedence:

1. `plemo.md` and repository-specific instructions;
2. agent-native repository and Odoo tools;
3. project-specific scripts/test helpers;
4. standard Odoo test/install/upgrade mechanisms;
5. generic shell commands only where appropriate.

Prefer available Plemo-native tools when they provide safer or more accurate evidence.

Examples may include:

```text
search_code
find_addon
read_file
odoo_local
read_odoo_log
get_build_log
upgrade_module
Plemo dispatch/build environment
equivalent project-native tools
```

Do not assume every repository exposes every example above.

Never invent a tool that is not actually available.

Do not dispatch to a build/runtime environment merely because a validation capability is missing locally. Use dispatch only when the repository/project workflow and available Plemo capabilities permit it and when the validation requirement materially justifies it.

---

# 6. Establish the Validation Environment

Before runtime testing, classify the environment.

Record:

```text
Environment:
Local / development / test / staging / production / unknown

Database:
Disposable / cloned / shared / production / unknown

Installed module state known:
Yes / No

Browser/runtime frontend access:
Available / unavailable

External integrations:
Sandbox / mocked / live / unknown
```

Validation conclusions depend on environment quality.

A test that passes on an empty development database does not automatically prove production-data upgrade safety.

---

# 7. Production Safety Gate

Do not perform destructive, state-changing, or business-impacting validation on production merely because runtime access exists.

Examples of operations that need special care:

- module installation;
- module upgrade;
- migrations;
- recomputation of large stored fields;
- creating financial documents;
- posting accounting moves;
- confirming live orders;
- generating real shipments;
- sending email/SMS/WhatsApp;
- calling live payment APIs;
- triggering vendor/customer integrations;
- deleting records;
- altering access rights;
- changing production configuration.

Prefer:

```text
disposable local DB
test DB
staging DB
cloned production-like DB
sandbox integration
```

If only production can reproduce the issue, use the narrowest safe read-only or reversible verification possible unless explicit authorization permits more.

---

# 8. Validation Order

Run validation in an order that catches cheap/high-confidence failures before expensive/runtime checks.

Recommended order:

```text
1. Diff and scope audit
2. Static syntax/structure validation
3. Odoo-specific static consistency checks
4. Module load/install/upgrade validation
5. Backend functional validation
6. Security/user-context validation
7. Frontend/browser validation
8. Integration validation
9. Performance validation when required
10. Final regression evidence review
```

Fail fast on blocking syntax/load errors.

Do not waste time browser-testing a feature when the module cannot load.

---

# 9. Build the Validation Matrix

For material changes, create a validation matrix.

Use:

```text
ID:
Scenario:
Layer:
Risk source:
Environment:
Expected:
Actual:
Evidence:
Status:
```

Statuses:

```text
PASS
FAIL
PARTIAL
BLOCKED
NOT APPLICABLE
```

The validation matrix should come from:

- explicit acceptance criteria;
- actual changed files;
- Impact Evidence;
- reverse dependencies;
- required runtime verification;
- relevant existing tests.

Do not create generic scenarios unrelated to the change.

---

# STATIC VALIDATION

# 10. Final Diff Review

Inspect the full final diff.

Check for:

- unrelated edits;
- accidental formatting;
- deleted logic;
- duplicated logic;
- unintentional renames;
- commented-out production code;
- debug prints/logging;
- stale temporary code;
- backup files;
- generated files committed accidentally;
- unexpected dependency changes;
- accidental `sudo()`;
- accidental security broadening;
- accidental API contract changes.

Record whether the diff stayed within the expected boundary.

---

# 11. Manifest Validation

For changed `__manifest__.py` files, verify:

- valid Python syntax;
- correct module version format;
- required `depends`;
- no missing cross-module dependency;
- correct `data` ordering;
- security files load before views that require them where relevant;
- actions/templates load before references that require them;
- asset bundle declarations are correct;
- no duplicate asset entry;
- `installable` remains correct;
- version bump follows repository/deployment convention.

If the Plemo/Odoo.sh workflow depends on a manifest version bump to trigger module upgrade, verify that the required version change exists.

Do not assume all Odoo.sh setups use identical automation; follow `plemo.md` and project conventions.

---

# 12. `__init__.py` Wiring Validation

For every new Python file/package, verify it is imported.

Check:

```text
module/__init__.py
models/__init__.py
controllers/__init__.py
wizard/__init__.py
report/__init__.py
other package __init__.py
```

A syntactically correct file that is never imported is not a working implementation.

---

# 13. Python Syntax Validation

For every changed Python file, run a real Python compile check where possible.

Prefer:

```text
python -m py_compile <file>
```

or equivalent.

Do not rely only on AST parsing.

Validate all changed Python files, not only the main model file.

---

# 14. Python Odoo-Semantic Validation

Static syntax passing is not sufficient.

Review changed Python for Odoo-specific correctness, including when relevant:

- `_name`;
- `_inherit`;
- `_inherits`;
- field definitions;
- reserved field/model attribute names;
- `@api.depends`;
- `@api.depends_context`;
- `@api.constrains`;
- compute/inverse/search method signatures;
- singleton assumptions;
- multi-record behavior;
- return contracts;
- `super()` calls;
- context handling;
- company handling;
- `sudo()`;
- create/write/unlink lifecycle;
- recursion risk;
- batch behavior.

Do not redefine an existing model `_name` accidentally when extension inheritance is intended.

---

# 15. Method Override Validation

For changed overrides verify:

```text
Original method:
Other overrides:
super() called:
Arguments preserved:
Return value preserved/modified intentionally:
Recordset cardinality assumptions:
Context assumptions:
Side effects:
```

Test both single-record and multi-record execution when the upstream method can receive batches.

An override that works from a form button can still fail from batch automation, POS close, cron, import, or server action.

---

# 16. Compute Field Validation

For changed computed fields verify:

- compute method exists;
- all required dependencies are declared;
- dependencies are load-safe across modules;
- `store=True` is intentional;
- inverse/search behavior is consistent;
- no side-effectful business actions occur from compute;
- multi-record compute is correct;
- values are assigned for every record;
- recomputation fan-out is understood.

For stored compute changes, static validation alone is insufficient.

Require upgrade/recompute evidence when existing records matter.

---

# 17. Constraint Validation

For changed constraints verify:

- multi-record `self` is handled safely;
- constraint covers intended fields;
- existing records are considered;
- upgrade recomputation cannot unexpectedly trigger invalid historical data;
- error message is user-safe;
- no context-dependent bypass creates inconsistent behavior unless intentional.

A constraint that works on newly created records may still block module upgrade on existing records.

---

# 18. Selection Validation

For any Selection change verify all relevant consumers:

```text
selection=
selection_add=
Python comparisons
domains
filters
search views
record rules
XML modifiers
JS comparisons
reports
exports
integrations
cron/server actions
migration code
existing stored values
```

Check whether removed or renamed keys exist in the database.

Do not mark a Selection change safe from code search alone when existing records have not been checked.

---

# 19. XML Well-Formedness Validation

For every changed XML file verify it parses as XML.

Check:

- opening/closing tags;
- escaped characters;
- invalid comments;
- duplicated attributes;
- malformed QWeb;
- malformed data records.

XML parse success is only the first layer.

Odoo view validation may still fail.

---

# 20. XML View Validation

For inherited views verify:

- referenced XML IDs exist;
- `inherit_id` owner is correct;
- cross-module XML IDs are prefixed correctly;
- XPath targets exist;
- XPath is scoped correctly;
- `position` is appropriate;
- downstream child views are not broken;
- priority assumptions are valid;
- labels/fields/modifiers remain structurally valid;
- version-specific view restrictions are respected.

When possible, validate by actually loading/upgrading the module in Odoo.

Do not claim an inherited view works merely because ElementTree parses it.

---

# 21. QWeb Validation

For changed QWeb/OWL XML verify, as applicable:

- correct template mechanism for the detected version;
- `t-inherit` / `t-inherit-mode`;
- XPath target;
- `t-foreach` and `t-key`;
- variable scope;
- escaped/raw content handling;
- template names;
- component linkage;
- no forbidden directives in server-side view architecture;
- downstream inherited templates still have their expected extension points.

Runtime render validation is required for meaningful frontend/template changes when browser or HTTP rendering is available.

---

# 22. JavaScript Static Validation

For every changed JS file verify:

- syntax;
- imports;
- module paths;
- exports;
- registry names;
- component names;
- patch targets;
- service names;
- selector names;
- RPC/model/route names;
- lifecycle method compatibility with the detected Odoo version;
- no obvious undefined variables;
- no stale debug code.

Static validation cannot prove an OWL patch is mounted or rendered.

---

# 23. Asset Static Validation

Inspect relevant asset declarations.

Confirm:

- file is included;
- bundle is appropriate;
- glob already includes the file if applicable;
- no duplicate inclusion;
- dependency/load order is plausible;
- patch loads after the implementation it patches where required;
- frontend code is not accidentally added to a global backend bundle;
- backend-only code is not pushed to public frontend unnecessarily.

If asset runtime is not available, mark actual loading as unverified.

---

# 24. Security Static Validation

When security is touched, inspect:

```text
ir.model.access.csv
ir.rule
groups
groups_id
field groups=
menu/action groups
sudo()
with_user()
with_company()
controller auth
csrf
record IDs accepted from users
```

Confirm new models have the intended ACL coverage.

Do not infer runtime access correctness solely from XML/CSV presence.

---

# 25. Migration Static Validation

For migration scripts verify:

- path/version naming follows repository convention;
- code imports;
- migration is idempotent where practical;
- target fields/models exist at the point the migration runs;
- data assumptions are explicit;
- failures do not silently lose data;
- large updates are considered;
- rename/backfill order is correct;
- external IDs remain stable or are migrated.

Do not execute migrations against production during ordinary validation.

---

# 26. Localization Validation Handoff

When the change includes Arabic localization, use the specialized Odoo Localization & Arabic QA skill for localization-specific checks.

This validator should still confirm high-level regression facts such as:

- modified PO/JS localization files are in the intended scope;
- no unrelated business logic changed;
- relevant assets still load;
- English and Arabic paths are included in runtime validation when required.

Do not duplicate the full localization skill unnecessarily.

---

# ODOO LOAD / INSTALL / UPGRADE VALIDATION

# 27. Determine Installed Module State

Before choosing install vs upgrade validation, determine whether the module is:

```text
Not installed
Installed
Upgrade pending
Unknown
```

Use available runtime/build evidence.

Do not assume a module is installed because its folder exists.

---

# 28. Fresh Install Validation

Fresh install validates a different failure surface from upgrade.

Run or request a fresh install when relevant, especially for:

- new modules;
- changed manifest load order;
- new security/data files;
- changed XML IDs;
- dependencies;
- installation hooks;
- views/actions that can be masked by long-lived DB state.

Validate:

```text
module installs
dependencies resolve
all data files load
security loads
views load
seed data loads
hooks succeed
registry starts
no traceback
```

A green incremental upgrade does not prove fresh installation works.

---

# 29. Module Upgrade Validation

Run or request module upgrade when the changed module is already installed and implementation affects:

- Python fields/models;
- XML views/data;
- security;
- stored computes;
- constraints;
- migrations;
- assets whose registration changed.

Validate:

```text
upgrade starts
migration runs if applicable
registry loads
stored recomputes complete
constraints do not abort
XML/data updates apply
security updates apply
logs remain clean
expected data exists afterward
```

Do not equate "build succeeded" with "module upgraded" unless the deployment workflow proves it.

---

# 30. Python Reload / Restart Awareness

Some Odoo changes require a server process restart before new Python code is active.

Do not validate a newly added Python field/method against a process that still has the old registry/code loaded.

Record:

```text
Python restart/reload required:
Performed:
Evidence:
```

---

# 31. Stored Compute Upgrade Validation

For stored compute changes, validate on existing records.

At minimum, where runtime access permits:

1. perform the relevant module upgrade/recompute;
2. monitor logs;
3. read representative existing records;
4. verify expected computed values;
5. verify constraints did not reject historical rows;
6. inspect performance/volume risk when material.

Do not claim stored-compute safety from new-record tests alone.

---

# 32. Existing Data Validation

When a change affects schema or semantics, inspect representative existing records.

Check as relevant:

- NULL values;
- old Selection keys;
- related records;
- company ownership;
- archived records;
- historical states;
- invalid legacy values;
- records created before the new field existed.

Existing data is part of the feature contract.

---

# 33. `noupdate` Runtime Validation

For changes to `noupdate="1"` data, verify the behavior on an existing database.

Determine whether the upgrade actually changes the record.

If it does not, do not report the code change as deployed merely because the XML file changed.

---

# 34. Migration Runtime Validation

When a migration exists, prefer a cloned/staging database containing realistic pre-change data.

Validate:

```text
pre-migration data state
migration execution
post-migration row counts
post-migration key values
constraints
references/XML IDs
downstream business behavior
rerun/idempotency behavior when relevant
```

Do not infer production migration safety from an empty database.

---

# BACKEND FUNCTIONAL VALIDATION

# 35. Use Odoo Runtime Evidence When Available

Use available Plemo/Odoo runtime tools where appropriate, such as:

```text
odoo_local
read_odoo_log
upgrade_module
build/test environment
equivalent project tools
```

The exact toolset depends on the environment.

Do not invent tool names.

Record runtime commands/actions and their outcomes.

---

# 36. Validate the Requested Happy Path

Every behavioral change needs at least one representative success scenario.

Record:

```text
Preconditions:
Action:
Expected result:
Actual result:
Database/result evidence:
Status:
```

The happy path should match the user's acceptance criteria, not merely confirm that the method returns without exception.

---

# 37. Validate Negative and Boundary Paths

Test meaningful invalid/edge scenarios supported by the change.

Examples:

- missing optional/required input;
- invalid state transition;
- empty recordset where supported;
- multiple-record batch;
- duplicate request;
- archived record;
- already-processed record;
- cross-company record;
- unauthorized user;
- missing integration response;
- invalid Selection key;
- absent optional relation.

Do not manufacture dozens of irrelevant edge cases.

Use risk evidence.

---

# 38. Validate Return Contracts

When a method/controller was changed, verify the return value expected by callers.

Examples:

```text
action dict
recordset
boolean
mapping
JSON payload
redirect
HTTP response
None
```

A workflow may perform the side effect correctly but still break its caller because the return contract changed.

---

# 39. Validate Batch Behavior

If an Odoo method can be called on multiple records, test a multi-record recordset.

Especially important for:

- `action_*`;
- `create` with multi-create behavior;
- `write`;
- `unlink`;
- constraints;
- computes;
- scheduled jobs;
- imports;
- mass actions.

Do not add `ensure_one()` unless the upstream contract truly guarantees singleton behavior.

---

# 40. Validate Context-Dependent Behavior

When the implementation reads or modifies context, test relevant contexts such as:

```text
active_id
active_ids
active_model
lang
tz
allowed_company_ids
website_id
default_*
skip_*
force_*
```

Only test context variants relevant to actual callers.

---

# 41. Validate CRUD Paths

When create/write/unlink logic changes, verify the operation through more than one path when risk justifies it.

Potential paths:

```text
backend form
import
RPC
server action
cron
batch write
copy/duplicate
website/portal
```

Onchange-only correctness does not prove create/write correctness.

---

# 42. Validate Automated Processes

If reverse dependencies include automation, test or inspect:

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

Confirm changed methods/data do not break automated execution.

---

# 43. Validate Reports and Exports

When changed data feeds reports or exports, verify representative output.

Check:

- report renders;
- totals remain correct;
- field is available;
- grouping/search works;
- CSV/XLSX/API export contract is unchanged or intentionally changed;
- historical records remain meaningful.

Do not validate only the form view if the field is used in reporting.

---

# SECURITY AND ACCESS RUNTIME VALIDATION

# 44. Security Validation Is User-Context Validation

Security cannot be proven solely as Administrator.

When relevant, validate as:

```text
Administrator
Internal user
Manager group
Restricted internal user
Portal user
Public user
Company A user
Company B user
```

Only include contexts supported by the feature.

---

# 45. ACL Runtime Validation

For new/changed models test:

- allowed operation succeeds;
- forbidden operation fails;
- read/create/write/unlink differ correctly where intended.

Record the actual user/group used.

Do not treat admin success as ACL proof.

---

# 46. Record Rule Runtime Validation

Test representative records both inside and outside the allowed domain.

For multi-company or ownership rules verify:

```text
visible expected record
hidden forbidden record
cross-company record
shared company_id=False record when applicable
```

Record-rule correctness must be demonstrated by data visibility, not just XML syntax.

---

# 47. `sudo()` Runtime Validation

If implementation uses `sudo()`:

- verify only the intended privileged operation is elevated;
- verify user-supplied record IDs cannot expose unrelated records;
- verify returned data is still filtered appropriately;
- verify company isolation;
- verify the public/portal path cannot read arbitrary records.

Treat unexpected broad access as a FAIL.

---

# 48. Public Controller Validation

For `auth="public"` routes, validate as an unauthenticated/public user when safe.

Check:

```text
HTTP method
CSRF behavior
input validation
record authorization
sudo scope
response data
error behavior
website isolation
```

Do not rely on an authenticated admin session.

---

# 49. Portal Controller Validation

For portal-facing routes, validate with a real portal-context user when possible.

Verify:

- login requirement;
- ownership/access domain;
- another customer's record is denied;
- response data contains no forbidden fields;
- token logic if used.

---

# 50. Multi-Company Runtime Validation

When relevant, test at least two company contexts.

Verify:

```text
Company A record behavior
Company B record behavior
multiple allowed companies active
cross-company access
company-dependent configuration
check_company behavior
```

A feature passing in one company is not proof of multi-company safety.

---

# 51. Multi-Website Runtime Validation

For website-facing changes, test more than one website when the project is multi-website or the impact evidence says website leakage is possible.

Verify:

```text
Website A
Website B
website-specific view resolution
website-specific settings
published content
company mapping where applicable
language behavior where relevant
```

---

# FRONTEND / JAVASCRIPT / OWL VALIDATION

# 52. Static Frontend Checks Are Not Runtime Proof

A manifest can list a JS file correctly while the browser still fails because of:

- asset caching;
- import resolution;
- patch timing;
- bundle order;
- component lifecycle;
- template mismatch;
- selector mismatch;
- runtime service availability;
- browser-only syntax/error.

Therefore distinguish:

```text
Frontend static validation
Frontend runtime validation
```

Never report the second when only the first was performed.

---

# 53. Browser Availability Gate

If a browser/runtime frontend tool is available, use it when the change requires browser proof.

If no browser is available:

- do not fake browser validation;
- use static checks and server logs;
- dispatch to an appropriate Plemo environment if available and allowed;
- otherwise mark the frontend runtime portion `PARTIAL` or `BLOCKED`.

The final decision must state exactly what remains unverified.

---

# 54. Asset Runtime Validation

For relevant frontend changes verify:

- asset bundle builds;
- browser actually requests/loads the changed asset;
- no asset compilation error;
- no stale asset version;
- no duplicate module definition;
- patch/component loads in the intended app/page.

When needed, include a hard refresh or asset rebuild according to repository/deployment conventions.

---

# 55. Browser Console Validation

For JS/OWL changes inspect browser console output.

Look for:

- import errors;
- registry errors;
- OWL lifecycle errors;
- template errors;
- undefined values;
- duplicate keys;
- patch errors;
- failed promises;
- RPC failures.

"No server traceback" does not prove the browser is clean.

---

# 56. Frontend Interaction Validation

Exercise the actual changed interaction.

Examples:

```text
click button
change field
open dialog
submit form
navigate page
switch tab
load list/kanban/form
trigger OWL rerender
perform checkout step
```

Verify both visual behavior and backend/RPC result.

---

# 57. JS-to-RPC Validation

For changed frontend requests verify:

```text
caller executes
payload matches backend expectation
auth/context is correct
controller/model receives request
response contract matches consumer
error path is handled
UI updates correctly
```

If an endpoint response changed, test every known repository consumer identified by Impact Evidence.

---

# 58. Selector and Template Contract Validation

When XML/QWeb markup changed, verify JS selectors still resolve.

Check:

```text
class
id
data-*
t-ref
template name
component mount point
DOM nesting assumptions
```

When JS changed, verify the corresponding markup exists in every relevant rendered variant.

---

# 59. OWL Patch Validation

For OWL patches verify:

- target exists in the detected version;
- patch loads after target;
- lifecycle method is valid;
- patch does not clobber another patch;
- state remains valid after rerender;
- `t-key` values remain unique;
- relevant services are available;
- component behaves in all affected actions/views.

Runtime browser proof is preferred.

---

# 60. Website Frontend Validation

For website features verify, as relevant:

- public page loads;
- authenticated page loads;
- form submission;
- multi-language;
- multi-website;
- responsive layout;
- RTL behavior;
- snippets/editor compatibility;
- selector behavior;
- no public console errors.

Only include mobile/RTL checks when the feature or localization impact makes them relevant.

---

# INTEGRATION VALIDATION

# 61. Identify Integration Validation Scope

If the implementation affects an external integration, use the impact evidence to identify:

```text
consumer/provider
endpoint
payload
authentication
identifiers
retry behavior
idempotency
error handling
sandbox availability
```

Do not call unrelated live external systems "just to test."

---

# 62. Prefer Sandbox or Mocked Integration Validation

Use, in order of preference:

```text
local mock
test fixture
provider sandbox
staging endpoint
controlled live verification only when explicitly safe/authorized
```

Avoid generating real charges, shipments, messages, or irreversible external records during routine validation.

---

# 63. API Contract Validation

Verify both directions when relevant.

### Outbound:

```text
method/URL
headers
payload
required identifiers
types
optional fields
timeouts
```

### Inbound:

```text
route type
auth/signature
payload parser
required keys
duplicate delivery
error response
status codes
```

Record expected vs actual contract.

---

# 64. Idempotency and Retry Validation

For webhooks, payment callbacks, queue jobs, or retried integrations, verify duplicate delivery does not create duplicate business effects.

Test where safe:

```text
same event delivered twice
timeout after remote commit
retry after partial local failure
already-processed identifier
```

Do not blindly retry an operation when evidence suggests the side effect may already have committed.

---

# PERFORMANCE VALIDATION

# 65. Separate Performance Smell From Measurement

Static review may identify likely performance risk.

Runtime measurement proves performance behavior.

Use language such as:

```text
Potential performance risk
Measured regression
No meaningful regression observed in tested dataset
Production-scale performance unverified
```

Do not claim a performance improvement or regression without measurement.

---

# 66. ORM Performance Validation

When performance risk is material, observe or measure:

- query count;
- repeated searches;
- N+1 behavior;
- compute frequency;
- record volume;
- SQL duration where tooling permits;
- batch behavior.

Prefer representative recordsets.

Do not benchmark using one record when production operates on thousands unless the purpose is only functional correctness.

---

# 67. Stored Recompute Performance Validation

For stored compute changes, record:

```text
record count tested
upgrade/recompute duration if measured
errors
locking/timeouts if observed
fan-out
```

If production record volume is unknown, state that limitation.

---

# 68. Frontend Performance Validation

When relevant measure or inspect:

- number of new RPC calls;
- payload size;
- repeated DOM scans;
- rerender frequency;
- asset size/load;
- expensive synchronous work.

Do not make frontend performance validation mandatory for a backend-only change.

---

# REGRESSION SELECTION

# 69. Validate the New Behavior and the Old Behavior

A change is not complete merely because the new path works.

For each material change identify:

```text
New/changed behavior to prove
Existing behavior that must remain unchanged
Alternate path affected by the same method/data
```

Examples:

```text
new validation works
existing valid records still save

new action branch works
standard action branch still works

new field computes
old reports still render

new public route behavior works
portal/internal routes remain correct
```

---

# 70. Use Reverse Dependencies to Select Regression Tests

Regression scenarios should come from actual dependencies.

For each changed component ask:

```text
Who calls it?
Who inherits it?
Who reads the field?
Who renders the template?
Who consumes the response?
Who runs it automatically?
Who exports/reports it?
```

Test the meaningful consumers.

Do not list generic Odoo applications without evidence.

---

# 71. Shared Module Regression Expansion

If a shared custom module changed, expand validation beyond the single visible feature.

Use reverse dependencies to identify:

- other customer features in the same project;
- shared views;
- shared models;
- integrations;
- scheduled jobs;
- reports.

Do not automatically search sibling customer projects unless repository architecture proves the shared module is consumed there.

---

# 72. High-Risk Workflow Regression

For high-impact business flows, validate the downstream chain supported by evidence.

Examples may include:

```text
Sales:
quotation -> confirmation -> delivery -> invoice

Purchase:
RFQ -> PO -> receipt -> vendor bill

Accounting:
draft -> post -> reconciliation/reversal

Website sale:
cart -> checkout -> payment -> order

Stock:
reservation -> picking -> validation

Payroll:
input -> computation -> validation/accounting
```

Use only the actual chain relevant to the change.

Do not run destructive live financial flows in production for validation.

---

# 73. Existing Test Discovery

Search relevant modules for existing automated coverage.

Examples:

```text
tests/
TransactionCase
SavepointCase
HttpCase
tour tests
QUnit
HOOT
pytest
project scripts
CI test jobs
```

Reuse existing tests before inventing redundant manual checks.

Record:

```text
Existing test:
Scope:
Result:
Relevance:
```

---

# 74. Targeted Tests vs Full Suite

Default to targeted tests selected from the actual impact surface.

Run a broader suite when:

- the module is shared;
- core business behavior was overridden;
- reverse dependencies are broad;
- a migration/schema change is high-risk;
- the user explicitly requests full regression;
- project CI convention requires it.

Do not automatically run the entire repository suite for every typo or isolated change.

---

# 75. Fresh Install and Upgrade Are Separate Regression Scenarios

When relevant, include both explicitly in the matrix.

Example:

```text
Scenario A:
Fresh install on clean DB

Scenario B:
Upgrade existing DB with historical records
```

Passing one does not imply the other passes.

---

# LOG AND FAILURE ANALYSIS

# 76. Establish a Log Baseline

When possible, inspect logs before the runtime test.

This helps distinguish:

```text
pre-existing warning/error
new error caused by validation
unrelated background error
test-specific traceback
```

Do not attribute every log error after a test to the change.

---

# 77. Read Logs After Material Runtime Actions

After:

- restart;
- module install;
- module upgrade;
- migration;
- functional workflow;
- scheduled job;
- controller request;

inspect relevant Odoo/build logs.

Capture the important traceback/error context.

Do not paste huge irrelevant logs into the final report.

---

# 78. Classify Failures

Classify each failure as:

```text
CHANGE FAILURE
PRE-EXISTING FAILURE
ENVIRONMENT FAILURE
TEST-DATA FAILURE
EXTERNAL DEPENDENCY FAILURE
UNVERIFIED / INCONCLUSIVE
```

Explain the evidence supporting the classification.

---

# 79. Do Not Hide Partial Success

A workflow can partially succeed.

Example:

```text
record created successfully
email failed
UI did not update
```

Report each part separately.

Do not mark the entire scenario PASS because the database side effect happened.

---

# 80. Failure Reproduction

For meaningful failures capture:

```text
Preconditions:
Exact action:
Expected:
Actual:
Error:
Relevant log:
Records affected:
Repeatable:
```

Avoid speculative root-cause claims when reproduction is not stable.

---

# 81. Validation-Only Failure Behavior

If the user asked only to validate/review/test:

- do not modify files;
- do not silently patch;
- report the failure;
- recommend the next investigation/fix boundary;
- preserve evidence.

The validator should not mutate into an implementation skill.

---

# 82. Fix-and-Validate Loop

If the original task authorizes fixing:

```text
Validation FAIL
        ↓
Produce failure evidence
        ↓
Return control to native planner/debugger
        ↓
Agent implements smallest safe fix
        ↓
Rerun failed scenario
        ↓
Rerun affected regression scenarios
        ↓
Update final validation status
```

Do not rerun every unrelated test after a tiny fix unless the fix changed the impact surface.

Do not ask for duplicate approval solely to enter the fix loop.

---

# EVIDENCE AND STATUS

# 83. Evidence Levels

Use explicit evidence labels:

```text
Static evidence
Repository evidence
Runtime/database evidence
Browser evidence
Security-context evidence
Integration evidence
Performance measurement
```

A conclusion may use more than one.

---

# 84. Confidence Labels

Use when useful:

```text
Confirmed
Strong evidence
Probable
Possible
Unverified
```

Do not turn a static inference into a runtime confirmation.

---

# 85. Required Result Statuses

Use only these primary validation statuses:

```text
PASS
FAIL
PARTIAL
BLOCKED
```

### PASS

All required validation for the agreed scope ran successfully, with no unresolved material verification gap.

Warnings may still exist if they are non-material and clearly documented.

### FAIL

At least one required scenario fails because of the implementation or a regression caused by it.

### PARTIAL

The checks that ran succeeded or produced no change failure, but one or more required validations could not be performed.

Examples:

- no browser available for OWL runtime validation;
- no production-like data for recompute scale;
- external sandbox unavailable.

### BLOCKED

Meaningful validation cannot proceed safely or reliably.

Examples:

- module does not load;
- required database unavailable;
- environment state unknown;
- migration would be destructive without a clone/backup;
- only production endpoint exists and test would cause real side effects.

Do not call a result PASS when it is actually PARTIAL.

---

# 86. Per-Check Status

Individual matrix rows may also use:

```text
PASS
FAIL
PARTIAL
BLOCKED
NOT APPLICABLE
```

`NOT APPLICABLE` is valid only with a reason.

Do not use it to skip inconvenient checks.

---

# 87. Static Pass vs Runtime Pass

When both matter, report separately:

```text
Static validation: PASS
Runtime validation: PARTIAL
Overall: PARTIAL
```

This prevents a common false conclusion:

```text
Code compiles -> feature works
```

---

# 88. Pre-Existing Failure Does Not Automatically Fail the Change

If a test fails for a clearly pre-existing unrelated reason:

- document it;
- determine whether it blocks the requested validation;
- if it does not block validation, continue;
- if it prevents meaningful proof, overall status may become PARTIAL or BLOCKED.

Do not "fix" unrelated pre-existing failures unless the user expands scope.

---

# CHANGE-TYPE VALIDATION PLAYBOOKS

# 89. New Field Playbook

For a new field, consider:

```text
manifest/dependency
Python definition
__init__ wiring
security/groups
view presence
default behavior
existing record value
create/write behavior
search/group/export behavior
fresh install
upgrade existing DB
multi-company if relevant
```

If computed/stored, also use the Stored Compute Playbook.

---

# 90. Stored Compute Playbook

For a stored compute change, require or justify:

```text
@api.depends review
multi-record compute review
upgrade/recompute
existing-record read-back
constraint impact
dependent stored fields
record volume/performance
downstream report/search consumers
```

A new-record-only test is insufficient.

---

# 91. Method Override Playbook

For an overridden business method validate:

```text
original contract
other overrides
super() chain
return value
single record
batch recordset when applicable
normal context
special caller context
direct workflow
at least one meaningful downstream workflow
logs
```

---

# 92. XML/QWeb Playbook

For inherited XML/QWeb validate:

```text
XML parse
external ID
inherit_id
XPath target
downstream inheritors
module load/upgrade
rendered result
JS selector contract
CSS contract
multi-website/language when relevant
```

---

# 93. JavaScript/OWL Playbook

For JS/OWL changes validate:

```text
syntax/imports
asset bundle
patch/component target
template connection
browser load
console
actual interaction
RPC request/response
rerender/state
other patches
```

Without browser runtime, overall frontend validation is normally PARTIAL unless the requested scope is explicitly static-only.

---

# 94. Controller/Route Playbook

For route changes validate:

```text
route declaration
auth
type
HTTP methods
CSRF
input validation
public/portal access
record authorization
payload
response contract
known callers
error path
multi-website if relevant
external consumers if any
```

---

# 95. Security Playbook

For ACL/rule/group changes validate:

```text
admin
allowed user
denied user
inside-domain record
outside-domain record
cross-company record
portal/public if relevant
sudo path
menu/action visibility
field groups
```

---

# 96. Selection Change Playbook

For Selection changes validate:

```text
source definition
all code comparisons
domains/filters
views/modifiers
record rules
JS consumers
reports/exports
existing DB values
migration/backfill
fresh install
upgrade
```

---

# 97. Migration Playbook

For migration/data transformation validate:

```text
realistic pre-change data
backup/clone safety
fresh install behavior separately
upgrade path
migration execution
row counts
key field values
references
constraints
idempotency when appropriate
downstream workflows
rollback implications
```

---

# 98. `noupdate` Playbook

For `noupdate` changes validate:

```text
fresh install result
existing DB upgrade result
whether record actually changes
XML ID stability
duplicate-record risk
manual migration need
```

---

# 99. Integration Playbook

For integration changes validate:

```text
sandbox/mock
authentication
outbound payload
inbound payload
identifiers
status/error handling
idempotency
retry
timeout
duplicate event
backward compatibility
```

---

# 100. Shared Module Playbook

For a shared module change:

```text
reverse-dependent modules
shared models/methods
shared views/templates
shared JS/assets
scheduled jobs
integrations
representative workflows
broader automated tests
```

Do not validate only the feature that motivated the change.

---

# STOP CONDITIONS

# 101. Stop Before Unsafe Runtime Actions

Stop and report `BLOCKED` or request the needed authorization/environment when validation would require:

- destructive production migration;
- irreversible data mutation;
- live payment charge;
- live customer/vendor notification;
- real stock/accounting movement;
- broad security change on production;
- external side effect with no sandbox;
- module upgrade on production without approval;
- operation that cannot be rolled back reasonably.

---

# 102. Stop When Environment Cannot Prove the Requirement

Examples:

```text
browser runtime required but unavailable
existing-data upgrade risk but only empty DB exists
public-user security required but no usable user/context exists
integration consumer contract required but consumer is unknown
multi-company risk but only one company exists in test data
```

Choose:

```text
PARTIAL
```

when meaningful validation was performed but a required portion remains unavailable.

Choose:

```text
BLOCKED
```

when the missing condition prevents meaningful validation of the change.

---

# 103. Do Not Turn Missing Runtime Access Into Guesswork

If runtime proof is required:

- state exactly what cannot be verified;
- state why static evidence is insufficient;
- provide the precise required check/environment;
- do not claim success.

This is a primary purpose of this skill.

---

# REQUIRED OUTPUT

# 104. Validation Evidence Summary

For every material validation, start the final output with:

```text
VALIDATION EVIDENCE

Project:
Change:
Odoo version:
Environment:

Impact Evidence reused:
Yes / No

Changed files:
- ...

Validation scope:
- ...

Static validation:
PASS / FAIL / PARTIAL / BLOCKED

Odoo load/install/upgrade:
PASS / FAIL / PARTIAL / BLOCKED / NOT APPLICABLE

Backend functional:
PASS / FAIL / PARTIAL / BLOCKED / NOT APPLICABLE

Security:
PASS / FAIL / PARTIAL / BLOCKED / NOT APPLICABLE

Frontend/browser:
PASS / FAIL / PARTIAL / BLOCKED / NOT APPLICABLE

Integration:
PASS / FAIL / PARTIAL / BLOCKED / NOT APPLICABLE

Performance:
PASS / FAIL / PARTIAL / BLOCKED / NOT APPLICABLE

Overall:
PASS / FAIL / PARTIAL / BLOCKED
```

---

# 105. Validation Matrix

Include the concrete scenarios that matter.

Example:

```text
ID: V-01
Scenario: Upgrade module with existing orders
Expected: Upgrade completes and stored total recomputes
Actual: Upgrade completed; 25 representative rows match expected totals
Evidence: upgrade log + database read-back
Status: PASS
```

Do not write vague rows such as:

```text
Test feature -> PASS
```

---

# 106. Required Detailed Report

For complex/material validation, use this structure.

## 1. Validation Target

```text
Project:
Requested behavior:
Acceptance criteria:
Target module(s):
```

## 2. Change Set

```text
Changed files:
Added files:
Deleted files:
Pre-existing dirty files:
Boundary deviation:
```

## 3. Environment

```text
Odoo version:
Database type:
Installed module state:
Browser available:
Integration environment:
```

## 4. Impact Evidence Reused

Summarize the relevant regression and runtime requirements from Skill 3.

## 5. Static Validation

List:

- diff audit;
- manifest;
- imports;
- Python;
- XML/QWeb;
- JS;
- assets;
- security;
- migration;
- localization handoff.

## 6. Install / Upgrade Validation

```text
Fresh install:
Module upgrade:
Python restart:
Stored recompute:
noupdate:
Migration:
```

## 7. Backend Functional Validation

List actual scenarios and evidence.

## 8. Security Validation

List users/groups/companies/routes tested.

## 9. Frontend / Browser Validation

List assets, console, interactions, RPC, websites/languages tested.

## 10. Integration Validation

List contract and sandbox/runtime results.

## 11. Performance Validation

Separate inferred risk from measured evidence.

## 12. Regression Matrix

List every material scenario with status.

## 13. Failures

For each failure:

```text
Classification:
Scenario:
Expected:
Actual:
Evidence:
Likely scope:
```

## 14. Pre-Existing / Unrelated Failures

Do not mix these with change failures.

## 15. Remaining Unknowns

Examples:

```text
browser unavailable
production-scale data unavailable
external consumer unknown
multi-company dataset unavailable
```

## 16. Final Decision

Use exactly one:

```text
PASS
FAIL
PARTIAL
BLOCKED
```

Explain the decision briefly.

## 17. Required Next Action

Only when not PASS:

```text
Fix required:
Runtime check required:
Environment required:
Scope clarification required:
```

Do not create a second generic implementation plan.

---

# 107. Concise Output for Small Changes

For a small contained change, do not force the full report.

Use:

```text
Validation: PASS / FAIL / PARTIAL / BLOCKED

Changed:
...

Checks:
- ...
- ...
- ...

Runtime:
...

Remaining:
...
```

Still distinguish unrun runtime checks from successful checks.

---

# 108. Files and Commands Investigated

When useful, list:

```text
Files inspected:
Commands/tests run:
Runtime operations:
Logs inspected:
Records/scenarios checked:
```

Do not claim tools/actions that were not actually used.

---

# 109. Final Diff Recheck

After all fixes and revalidation, inspect the final diff again.

Confirm:

- no validation/debug changes remain accidentally;
- no temporary instrumentation remains;
- no unrelated files changed during the fix loop;
- manifest/version/migration state still matches the final code;
- test additions are intentional;
- generated runtime files are not committed accidentally.

---

# 110. Hard Prohibitions

Never:

- replace Plemo's native planner with a validation plan;
- require duplicate implementation approval when the original request already authorizes fixes;
- silently fix code during validation-only mode;
- claim runtime validation from static inspection;
- claim browser validation without browser/runtime evidence;
- claim security correctness from admin-only testing;
- treat module folder presence as proof it is installed;
- treat build success as proof module upgrade happened;
- treat XML parse success as proof Odoo view load success;
- treat Python compile success as proof Odoo semantic correctness;
- treat "no traceback" as proof business correctness;
- treat a new-record test as proof stored-compute upgrade safety;
- ignore existing production data for schema/Selection/migration changes;
- skip fresh-install validation when load-order/installability is material;
- skip upgrade validation when existing installed databases are material;
- run destructive production tests without explicit authorization;
- call live payment/shipping/messaging integrations unnecessarily;
- ignore public/portal access for public-facing routes;
- ignore multi-company/multi-website behavior when Impact Evidence identifies it;
- hide pre-existing failures inside change failures;
- mark unavailable required checks PASS;
- call PARTIAL validation PASS;
- invent evidence;
- claim measured performance when only static smells were reviewed;
- modify unrelated code merely to make tests green;
- expand regression scope generically without dependency evidence;
- run the full repository suite for every trivial change without reason;
- commit or push unless explicitly requested by the user/project workflow.

---

# 111. Core Decision Tree

```text
START
  |
  v
What is the task mode?
  |
  +-- Validation only
  |      -> do not edit code
  |
  +-- Fix/Implement + Validate
         -> validator owns validation phase only
  |
  v
Read plemo.md / repository guidance.
  |
  v
Inspect actual final diff.
  |
  v
Is reliable Impact Evidence available?
  |
  +-- Yes -> reuse it
  |
  +-- No -> derive only the validation surface needed
  |
  v
Did implementation materially expand the analyzed boundary?
  |
  +-- Yes -> refresh impact evidence for the new surface
  |
  v
Determine Odoo version and validation environment.
  |
  v
Is the change trivial/non-behavioral?
  |
  +-- Yes -> light targeted validation
  |
  +-- No -> build material validation matrix
  |
  v
Run diff/static checks.
  |
  +-- Blocking failure?
  |      |
  |      +-- Yes -> FAIL
  |
  v
Does the change require install/upgrade proof?
  |
  +-- Yes -> fresh install / upgrade as appropriate
  |
  v
Does it require existing-data proof?
  |
  +-- Yes -> validate representative existing records
  |
  v
Run backend functional scenarios.
  |
  v
Does security context matter?
  |
  +-- Yes -> validate relevant users/groups/companies
  |
  v
Does frontend runtime matter?
  |
  +-- Yes
  |      |
  |      +-- Browser available -> validate runtime
  |      |
  |      +-- Browser unavailable -> PARTIAL/BLOCKED as appropriate
  |
  v
Does external integration matter?
  |
  +-- Yes -> sandbox/mock/controlled validation
  |
  v
Does performance require measurement?
  |
  +-- Yes -> measure in representative environment
  |
  v
Classify every scenario.
  |
  v
Any change-caused required failure?
  |
  +-- Yes
  |      |
  |      +-- Validation-only -> FAIL and report
  |      |
  |      +-- Fix authorized -> hand evidence to native planner,
  |                            fix, then revalidate
  |
  v
Any required validation unavailable?
  |
  +-- Yes -> PARTIAL or BLOCKED
  |
  +-- No -> PASS
  |
  v
Final diff recheck.
  |
  v
Produce Validation Evidence.
```

---

# 112. Primary Rules to Always Remember

```text
Validate the actual final change.

Reuse Codebase Investigator and Feature Impact Analyzer evidence.

Do not replace the agent's native planning or implementation.

Do not require duplicate approval for an already-authorized fix.

Validation-only tasks do not edit source code.

Use plemo.md and repository-native tools first.

Accept a valid manifest version prefix as Odoo-version evidence
when core source is unavailable.

Use risk-based validation scope.
Do not over-test trivial changes.

Separate:
static validation
runtime validation
browser validation
security-context validation
integration validation
performance measurement

Separate:
fresh install
module upgrade
existing production-data behavior

For stored computes:
upgrade/recompute existing records.

For method overrides:
validate super(), return contracts, single/batch behavior,
and meaningful downstream workflows.

For XML/QWeb:
validate parse, inheritance, module load, rendered behavior,
and downstream selectors/inheritors.

For JS/OWL:
validate assets, browser console, interaction, RPC, and rerender.
If browser proof is unavailable, say so.

For public/portal routes:
test the actual access context.

For security:
admin-only success is not sufficient.

For migrations:
use realistic pre-change data in a safe environment.

For integrations:
prefer mock/sandbox and protect idempotency.

For performance:
never call an inference a measurement.

Always distinguish:
PASS
FAIL
PARTIAL
BLOCKED

Never mark an unrun required check PASS.

Never claim complete validation while a material runtime requirement
remains unverified.

The final output must tell the developer exactly what was proven,
what failed, and what remains unknown.
