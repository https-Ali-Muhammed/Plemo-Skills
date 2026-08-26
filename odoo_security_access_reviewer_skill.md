# Odoo Security & Access Reviewer

## Purpose

Use this skill to perform a deep Odoo-specific security and access review of a model, feature, workflow, controller, route, portal/website flow, integration surface, or implemented change.

The skill must determine:

- what data or operation is being protected;
- which users or systems can reach it;
- which Odoo security layers enforce access;
- whether ACLs, record rules, groups, company boundaries, website boundaries, controller authentication, tokens, `sudo()`, RPC, attachments, or other bypass paths can expose data or privileged behavior;
- whether security rules are correct in both repository design and relevant runtime contexts;
- what must be changed or validated before the security model can be considered safe.

This is a **specialized Odoo security evidence and review skill**.

It does not replace Plemo's native repository discovery, planning, implementation, debugging, or general regression workflow.

Its core question is:

```text
Who can do what, to which records, through which path,
and can any path bypass the intended Odoo security boundary?
```

Core principles:

- Review the real Odoo security model, not only UI visibility.
- Treat ACLs, record rules, groups, controllers, RPC, `sudo()`, company context, website context, and tokens as one connected system.
- Separate authentication from authorization.
- Separate UI restrictions from server-side enforcement.
- Separate Administrator behavior from normal-user behavior.
- Separate repository evidence from runtime proof.
- Separate security severity from evidence confidence.
- Prefer least privilege.
- Never recommend `sudo()` merely to make an access error disappear.
- Never claim a feature is secure because it works as Administrator.
- Never claim a route is safe because the page hides the button.
- Report exact bypass paths and required safeguards.
- Preserve Plemo's native task mode and implementation authorization behavior.

---

# 0. Native Agent Compatibility and Skill Scope

This skill extends Plemo's existing Odoo capabilities. It does not replace them.

Plemo already performs:

```text
repository discovery
plemo.md reading
module discovery
normal implementation planning
implementation
ordinary debugging
targeted validation
final diff review
```

This skill adds security-specific procedures, evidence, and guardrails.

Repository-specific instructions such as:

```text
plemo.md
customer restrictions
configured addon paths
project security conventions
deployment conventions
available Plemo tools
```

take precedence over generic examples in this skill.

Reuse reliable evidence already collected from:

```text
Odoo Codebase Investigator
Odoo Feature Impact Analyzer
Odoo Regression & Runtime Validator
current task investigation
current Git diff
existing tests
runtime logs
```

Do not repeat a full investigation when existing evidence is sufficient.

Use the smallest security-review depth justified by the actual security surface.

---

## 0.1 Relationship With the Other Plemo Skills

The intended division of responsibility is:

```text
Odoo Codebase Investigator
    "Where is the feature and how does it work?"
        ↓
Odoo Feature Impact Analyzer
    "What could this change affect?"
        ↓
Odoo Security & Access Reviewer
    "Is the security model safe and correctly enforced?"
        ↓
Plemo native planning
        ↓
Plemo native implementation
        ↓
Odoo Regression & Runtime Validator
    "What can we prove works after implementation?"
```

The Security Reviewer may also be used as a standalone audit after implementation.

When Skill 4 already has a security runtime matrix, reuse it.

When this skill identifies security-specific runtime requirements, hand them to Skill 4 for broader post-change validation when appropriate.

Do not duplicate the entire Regression & Runtime Validator.

---

## 0.2 Task Mode and Authorization

Determine the current task mode.

Common modes:

```text
Security review / audit only
Security diagnosis
Fix security issue
Implement security-sensitive feature
Validate a security fix
```

For **Security review / audit / diagnosis only**:

- remain read-only in the repository;
- do not silently change ACLs, record rules, controllers, groups, fields, tokens, or `sudo()` usage;
- safe read-only runtime inspection may be used when available;
- report findings, evidence, and recommended security boundaries.

For **Fix security issue** or **Implement security-sensitive feature**:

- this skill owns the security evidence phase;
- the original user request already counts as authorization for the requested fix/implementation;
- do not ask for a second approval merely because the security review finished;
- hand the security evidence to Plemo's native planner;
- continue unless the review reveals a material scope expansion, destructive operation, migration decision, production-only security change, or another unresolved decision that makes proceeding unsafe.

For **Validate a security fix**:

- perform security-specific validation;
- reuse Skill 4 when broader regression/runtime coverage is needed.

---

## 0.3 Security Review Activation Gate

Use this skill when the task involves or may materially affect:

- new models;
- ACLs;
- record rules;
- security groups;
- group inheritance / implied groups;
- field-level `groups=`;
- menus/actions whose visibility is security-sensitive;
- public or portal routes;
- authenticated controllers;
- RPC-exposed model methods;
- `sudo()`;
- `with_user()`;
- `with_company()`;
- user-provided record IDs;
- multi-company data;
- multi-website data;
- website forms;
- portal documents;
- access tokens;
- attachments or document downloads;
- external APIs/webhooks;
- secrets/credentials;
- server actions or scheduled actions with privileged users;
- sensitive HR/payroll/accounting/customer data;
- mass-assignment risk;
- direct SQL touching authorization-sensitive data;
- privilege escalation paths;
- data exposure through reports, exports, computed/related fields, or integrations.

Do not force a full security review for unrelated cosmetic or non-security changes.

---

# 1. Determine the Security Review Target

Record:

```text
Project:
Feature/workflow:
Requested change, if any:
Target module(s):
Primary model(s):
Primary route(s):
User types involved:
Sensitive data/operation:
Known security concern:
```

If the feature is already clear, do not ask unnecessary questions.

Resolve missing ownership using repository evidence before asking the user.

---

# 2. Detect the Odoo Version

Determine the Odoo version from the strongest available evidence.

Valid evidence may include:

```text
target/custom module __manifest__.py version prefix
odoo/release.py
odoo/version.py
version_info
plemo.md
repository branch
build configuration
existing Odoo APIs
```

A valid manifest version prefix is acceptable primary evidence when Odoo core source is unavailable.

Examples:

```text
17.0.1.0.0 -> Odoo 17
18.0.2.1.0 -> Odoo 18
19.0.1.0.0 -> Odoo 19
```

Record:

```text
Odoo version:
Evidence:
Edition if known:
```

Security behavior that is version-sensitive must be verified against the detected version.

---

# 3. Repository and Tool Precedence

Use:

1. `plemo.md` and project instructions;
2. Plemo-native repository/Odoo discovery tools;
3. project-specific helpers;
4. Odoo source when available;
5. generic read-only shell tools.

Examples of useful Plemo capabilities may include:

```text
find_addon
search_code
read_file
odoo_local
read_odoo_log
get_build_log
upgrade_module
equivalent project-native tools
```

Do not assume a tool exists merely because it appears in this skill.

Do not dispatch to another environment solely because a security test is unavailable locally. Dispatch only when project workflow permits it and the security requirement materially justifies it.

---

# 4. Reuse Existing Investigation and Impact Evidence

If reliable evidence already exists, reuse:

```text
feature owner
model ownership
route ownership
method overrides
XML/QWeb inheritance
JS/RPC callers
reverse dependencies
multi-company impact
multi-website impact
runtime requirements
do-not-touch boundaries
```

Refresh only the security-relevant parts that are stale or incomplete.

Do not rebuild the entire feature map from scratch.

---

# 5. Establish the Protected Asset

Security analysis starts with what must be protected.

Classify the protected asset:

```text
record visibility
record modification
record deletion
workflow/action execution
financial posting
salary/HR information
customer/partner information
attachment/document
report/export
portal document
website/private content
configuration
credential/secret
integration endpoint
system parameter
administrative operation
```

Record:

```text
Protected asset:
Sensitivity:
Business impact if exposed:
Business impact if modified:
```

Do not review ACL syntax without understanding what the ACL protects.

---

# 6. Build the Actor Matrix

Identify relevant actors.

Examples:

```text
Superuser / Administrator
Internal user
Restricted internal user
Manager
Employee
Salesperson
Accountant
HR user
Portal user
Public user
Company A user
Company B user
Website A visitor
Website B visitor
External API client
Scheduled-action user
Integration/service user
```

For each actor record:

```text
Actor:
Expected access:
Expected denied access:
Groups:
Company context:
Website context:
Authentication mode:
```

Only include actors relevant to the feature.

---

# 7. Build the Access-Path Map

List every way the protected asset can be reached.

Possible paths:

```text
backend form/list/kanban
menu/action
model RPC
button method
controller route
website form
portal page
report download
attachment download
export
import
cron
server action
automated action
integration/webhook
direct model method called by other code
JS -> RPC
```

A secure backend view does not prove the controller path is secure.

Record:

```text
Path:
Entry point:
User/context:
Server-side enforcement:
Potential bypass:
```

---

# ODOO SECURITY MODEL

# 8. Review ACLs

Inspect:

```text
security/ir.model.access.csv
XML access records
groups
perm_read
perm_write
perm_create
perm_unlink
```

For each relevant model record:

```text
Model:
ACL source:
Group:
Read:
Write:
Create:
Unlink:
Reason:
```

Check for missing ACLs on new models and overly broad ACLs on sensitive models.

---

# 9. ACLs Are Additive

Treat ACL grants as additive across applicable groups.

A restrictive ACL does not cancel a broader ACL granted by another group.

When a user has several groups, determine the effective permission set.

Search all ACL entries for the model, not only the target module.

Record:

```text
Model:
All matching ACLs:
Effective grant:
Unexpected broad grant:
```

Verify version/source if the security behavior is uncertain.

---

# 10. UI Group Restrictions Are Not ACLs

Security controls such as:

```text
groups on menus
groups_id on actions
groups on views
field groups
button groups
```

may hide UI elements but do not automatically replace model authorization.

If a user can still call the model method or route directly, UI hiding is not sufficient.

Report:

```text
UI restriction:
Server-side enforcement:
Bypass possible:
```

---

# 11. Review Record Rules

Inspect all relevant `ir.rule` records across modules.

For each rule record:

```text
Rule:
Model:
Global:
Groups:
Domain:
Read:
Write:
Create:
Unlink:
Module/file:
```

Determine the effective rule set for each relevant actor.

Do not review only the rule added by the current module.

---

# 12. Record Rule Composition

Verify record-rule composition against the detected Odoo version/source.

Common Odoo behavior to validate includes:

- global rules restrict all applicable users and intersect with other global rules;
- group-scoped rules can combine differently from global rules;
- a broad group rule may unexpectedly expand access relative to another group-scoped rule;
- ACL permission must exist before record rules matter.

Do not make intuitive assumptions about AND/OR behavior.

Build the effective domain for the actual user's groups when security is material.

---

# 13. Default-Allow Record Rule Risk

When ACL permission exists but no applicable record rule restricts a model, access may be broader than intended.

Check sensitive models for:

```text
broad ACL + no restrictive rule
broad internal-user ACL
manager-only data with no record isolation
company-aware model with no company rule
```

Do not assume "no rule" means "no access."

---

# 14. CRUD-Specific Rule Behavior

Record rules can apply differently to:

```text
read
write
create
unlink
```

Check the relevant permissions separately.

A record may be readable but not writable.

A user may have create access but be unable to read the created record afterward.

Test the intended lifecycle.

---

# 15. Create-Time Security

For create operations, inspect:

- ACL create permission;
- field values accepted from the caller;
- company assignment;
- owner/partner/user assignment;
- defaults/context;
- post-create record-rule visibility;
- `sudo()` around create;
- mass-assignment of privileged fields.

Do not trust user-provided ownership/company/group fields blindly.

---

# 16. Write-Time Security

Inspect sensitive writes for:

```text
state
company_id
user_id
partner_id
groups_id
active
amount
price/cost
approval fields
access token fields
security flags
configuration fields
```

Determine whether a user can write a value that gives access they should not have.

---

# 17. Unlink Security

Deletion may require stricter security than read/write.

Check:

- ACL unlink;
- record-rule unlink;
- state-based business protection;
- attachment cascade;
- accounting/stock consequences;
- `sudo().unlink()` paths.

Do not infer unlink safety from normal UI availability.

---

# 18. Field-Level Security

Inspect sensitive fields using:

```text
groups=
compute_sudo
related fields
inverse methods
readonly
```

Determine whether restricted data can leak through:

- another unrestricted related field;
- a compute;
- report;
- export;
- controller serialization;
- search/grouping;
- chatter;
- API response.

Field hiding in the form is not sufficient if the value remains retrievable elsewhere.

---

# 19. Related and Computed Data Exposure

For a restricted source field, search:

```text
related=
compute=
read_group/_read_group
reports
exports
controllers
JSON responses
website templates
mail templates
```

A derived field may expose sensitive information even when the original field is group-restricted.

Record the data lineage.

---

# GROUPS AND PRIVILEGE

# 20. Review Security Groups

Inspect:

```text
res.groups
category
implied_ids
users
menu/action groups
field groups
ACL groups
record-rule groups
```

Build the relevant group graph.

---

# 21. Review Implied Groups

A group may silently grant another group's permissions through `implied_ids`.

Trace:

```text
Assigned group
    ↓
Implied group
    ↓
ACL/rule/menu/field permissions
```

Do not evaluate a group's security from its own XML record alone.

---

# 22. Detect Privilege Escalation Through Group Assignment

Search for code that changes:

```text
groups_id
user groups
role/group configuration
security categories
```

Check whether non-administrative users can cause themselves or another user to gain privileged groups.

Treat self-service group escalation as high risk.

---

# 23. Group Checks in Python

Inspect:

```text
has_group()
user_has_groups()
env.user
target_user.has_group()
```

Verify the check is performed against the intended user.

A security decision during authentication or user provisioning may accidentally check the current execution user instead of the target user.

Record:

```text
Check:
User actually evaluated:
Expected user:
Risk:
```

---

# SUDO AND ELEVATION

# 24. Inventory `sudo()` Usage

Search relevant code for:

```text
sudo()
sudo(False)
with_user()
with_company()
SUPERUSER_ID
```

For every security-sensitive usage record:

```text
File/method:
Reason:
Scope:
Input source:
Records elevated:
Data returned:
Can elevation be narrowed:
```

---

# 25. `sudo()` Is a Security Boundary Bypass

Treat `sudo()` as privileged execution.

Do not recommend it merely to fix an AccessError.

For each use ask:

```text
Why is elevation required?
Could normal ACL/rules be corrected instead?
Can only the minimal search/read/write be sudoed?
Are user-provided IDs used under sudo?
Is sudoed data returned to an unprivileged caller?
Does company isolation disappear?
```

---

# 26. IDOR Through `sudo()` and User-Provided IDs

A common dangerous pattern is:

```text
public/portal user supplies record_id
        ↓
controller/model does sudo().browse(record_id)
        ↓
record is returned/modified
```

Check every route or RPC method that accepts:

```text
id
record_id
partner_id
order_id
invoice_id
attachment_id
document_id
token target
```

The record must still be authorized for the caller.

Treat "record exists" as different from "caller may access record."

---

# 27. Scoped Elevation

When elevation is necessary, prefer the smallest scope.

Review whether code can:

```text
validate caller first
resolve allowed record in normal user context
sudo only the final narrow operation
return only explicitly allowed fields
```

Do not broaden the whole method or recordset without need.

---

# 28. `with_user()` and Execution Identity

Inspect code that changes execution user.

Determine:

```text
source user
target user
why changed
effective groups
company context
record rules
side effects
```

Do not assume changing user is equivalent to `sudo()`.

Verify behavior against the detected Odoo version when important.

---

# MULTI-COMPANY SECURITY

# 29. Identify Company-Aware Models

Inspect:

```text
company_id
company_ids
company_dependent
check_company
_check_company_auto
allowed_company_ids
with_company()
record rules
property fields
```

Classify:

```text
Company-specific
Shared across companies
Company-dependent configuration
Not company-aware
```

---

# 30. Multi-Company Access Matrix

For relevant models build:

```text
Actor:
Active companies:
Record company:
Expected access:
Actual rule/domain:
Potential leak:
```

At minimum consider:

```text
Company A user -> Company A record
Company A user -> Company B record
User with A+B active -> both
Shared company_id=False record
```

---

# 31. Global Company Rule Review

For company-scoped sensitive models, verify whether a global company isolation rule exists where required by the design.

A manager "see all" group rule must not unintentionally bypass cross-company isolation.

Inspect how global and group-scoped rules combine.

Do not use a broad manager rule as a substitute for company isolation.

---

# 32. `sudo()` and Company Leakage

When `sudo()` is used on company-aware data, verify explicit company scoping.

Check:

```text
allowed_company_ids
company_id domain
with_company()
company-specific configuration
cross-company relations
```

A privileged search without a company domain may return records from companies the caller should not see.

---

# 33. `check_company` and Cross-Company Relations

Inspect relational fields using:

```text
check_company=True
```

and relevant company consistency checks.

Verify that:

- the field's company information is available where required;
- programmatic writes do not bypass intended company consistency;
- sudoed operations do not silently connect incompatible companies.

---

# WEBSITE AND PORTAL SECURITY

# 34. Identify Website and Portal Entry Points

Search:

```text
@http.route
CustomerPortal
portal mixins
website=True
auth="public"
auth="user"
website forms
QWeb templates
JS RPC
download routes
```

Record every security-relevant entry point.

---

# 35. Controller Authentication Review

For every route record:

```text
Route:
Controller:
auth:
type:
methods:
csrf:
cors if present:
website:
```

Authentication answers:

```text
Who are you?
```

Authorization answers:

```text
May you access this record/action?
```

A correct `auth=` value does not prove record authorization.

---

# 36. Public User Review

For `auth="public"`:

- assume unauthenticated visitors can reach the route;
- inspect the shared Public user's effective access;
- inspect `sudo()`;
- inspect every user-provided identifier;
- inspect data returned;
- inspect rate/abuse implications when material;
- inspect website/company scoping.

Do not test only while logged in as Administrator.

---

# 37. Portal User Review

For portal flows verify:

- ownership relationship;
- partner/customer scoping;
- document access;
- token handling;
- another portal user's record;
- archived/private records;
- attachments;
- reports/downloads.

Portal login alone is not sufficient authorization.

---

# 38. Route Override Security

When controllers inherit/override routes:

- locate the original route;
- inspect the overriding method;
- inspect the redeclared `@http.route`;
- verify auth/type/method/csrf semantics did not accidentally change;
- identify other controller overrides.

A route override can change security even when business logic is unchanged.

---

# 39. HTTP Method and CSRF Review

Inspect actual route configuration for the detected version.

For state-changing browser requests, verify CSRF behavior is appropriate.

For external webhooks, verify the design uses an appropriate authentication/signature mechanism when CSRF cannot apply.

Do not disable CSRF merely to make a browser form work.

---

# 40. CORS Review

If a route enables cross-origin access, inspect:

```text
cors
allowed origin behavior
credentials/cookies
authorization
response data
```

Do not expose authenticated sensitive endpoints broadly across origins without a verified requirement.

---

# 41. Input Allowlisting

For public, portal, or integration requests, do not pass raw request dictionaries directly into privileged model create/write operations.

Review patterns such as:

```text
request.params
kwargs
post
json payload
vals
```

Use an explicit allowlist of writable fields.

Flag mass assignment of sensitive fields.

---

# 42. Server-Side Validation

Client-side JS validation is not security enforcement.

For untrusted input verify server-side checks for:

- type;
- length/format;
- allowed state;
- record ownership;
- company;
- price/amount;
- product/service;
- file type/size when relevant;
- identifiers;
- authorization.

---

# RPC AND MODEL METHOD SECURITY

# 43. Identify RPC-Callable Methods

Search JS, controllers, external clients, and model APIs for calls to model methods.

Review public model methods that can be reached through RPC according to the detected Odoo version.

Do not rely on the UI button's groups alone.

---

# 44. Private/Internal Method Boundary

Methods intended only for internal use should not be exposed casually as public RPC entry points.

When relying on underscore/private naming or framework RPC restrictions, verify the behavior in the detected Odoo version.

Do not create a public wrapper that simply `sudo()`s a sensitive internal method without its own authorization.

---

# 45. RPC Parameter Authorization

For RPC methods receiving:

```text
record ids
domain
fields
company
user
partner
state
amount
```

verify the caller cannot broaden access by controlling those parameters.

Especially review caller-provided domains and fields in generic search/export helpers.

---

# TOKENS, ATTACHMENTS, AND DOCUMENTS

# 46. Access Token Review

For token-protected records inspect:

```text
token generation
token entropy
storage
comparison
expiration if applicable
revocation if applicable
record binding
user binding if applicable
URL exposure
logging
```

A token is an authorization mechanism and must be treated as sensitive.

---

# 47. Portal Document Token Review

For portal documents verify that the token or portal access helper authorizes the exact intended record.

Test:

```text
valid token + correct record
invalid token
token for another record
logged-in wrong portal user
expired/revoked behavior if supported
```

Do not authorize solely because a token parameter exists.

---

# 48. Attachment Access Review

Search for:

```text
ir.attachment
/web/content
/content routes
binary fields
download controllers
sudo attachment reads
```

Verify:

- attachment ownership/model binding;
- record authorization;
- public/private status;
- tokens if used;
- cross-company access;
- portal/public exposure.

Treat attachment IDs as untrusted identifiers.

---

# 49. Report and Export Download Security

Check reports/exports that accept record IDs or domains.

Verify the requester is authorized for every included record.

Do not let a restricted user export hidden records through a custom report or `sudo()` route.

---

# DATA EXPOSURE THROUGH UI AND OUTPUTS

# 50. View Visibility vs Data Visibility

An invisible/hidden field or button is not a complete security mechanism.

Check whether the data remains reachable through:

```text
RPC
export
report
search_read
controller
related field
compute
chatter
mail
attachment
integration
```

---

# 51. Search, Group, and Export Exposure

Sensitive data may leak through:

```text
search_read
read_group/_read_group
group_by
export
spreadsheet/report
dashboard
API serialization
```

Verify field/group/model security applies to these paths.

---

# 52. Chatter and Mail Exposure

For `mail.thread` / portal-visible models inspect:

- messages;
- followers;
- attachments;
- message bodies;
- email templates;
- activities;
- portal chatter.

Ensure restricted information is not copied into a channel visible to broader users.

---

# 53. Error Message Leakage

Public/portal/API errors should not expose:

```text
stack traces
SQL
filesystem paths
secrets
internal model names when sensitive
private record data
provider credentials
```

Preserve enough information for debugging in logs while returning safe user-facing errors.

---

# XSS, MARKUP, AND CONTENT SECURITY

# 54. QWeb Escaping Review

Inspect risky rendering such as:

```text
t-raw
Markup
HTML fields
safe content flags
manual innerHTML
```

Determine whether untrusted content can reach an unescaped rendering path.

Prefer normal escaping for user-supplied text.

---

# 55. JavaScript DOM Injection Review

Search changed/relevant JS for:

```text
innerHTML
insertAdjacentHTML
dangerous template construction
untrusted URL insertion
DOM from API response
```

If untrusted content is rendered as HTML, verify sanitization/escaping.

---

# 56. Markup and Message Safety

When using `Markup` or HTML messages, verify which portions are trusted markup and which are escaped user-controlled values.

Do not wrap an entire user-controlled string in a trusted-markup type.

---

# DATABASE AND QUERY SECURITY

# 57. Direct SQL Review

Search relevant code for:

```text
env.cr.execute
cursor.execute
SQL helpers
```

Verify:

- parameters are bound safely;
- user input is not string-concatenated into SQL;
- row-level Odoo security bypass is intentional;
- company/domain restrictions are reimplemented safely if ORM rules are bypassed;
- returned fields are authorized.

Direct SQL bypasses normal ORM record-rule enforcement.

---

# 58. Raw Domain and Dynamic Evaluation Review

Inspect:

```text
safe_eval
eval-like expressions
server action code
caller-provided domains
dynamic model names
dynamic field names
getattr
```

Determine whether untrusted input can control execution or broaden record access.

---

# SECRETS AND CONFIGURATION

# 59. Secret Storage Review

Identify:

```text
API keys
tokens
passwords
client secrets
webhook secrets
private URLs
signing keys
```

Check whether secrets are:

- hardcoded in repository;
- stored in configuration/system parameters;
- exposed to normal users;
- logged;
- returned to frontend;
- included in reports/errors.

Do not move secrets into source code for convenience.

---

# 60. `ir.config_parameter` Access

Sensitive parameters should not be casually exposed to frontend or broad internal users.

Review reads/writes of system parameters and any `sudo()` used around them.

Check whether a settings field unintentionally makes a secret readable to more users.

---

# 61. Logging Sensitive Data

Inspect logs for:

```text
tokens
passwords
authorization headers
payment data
personal data
full webhook payloads
secret configuration
```

Redact or avoid logging secrets.

Do not treat logs as a safe secret store.

---

# AUTOMATION AND SERVICE USERS

# 62. Scheduled Action Security

Inspect relevant `ir.cron` / scheduled actions.

Record:

```text
Scheduled action:
Execution user:
Companies:
Method:
sudo usage:
Records processed:
```

A cron may run with broader permissions than the interactive user.

---

# 63. Server and Automated Actions

Review:

```text
ir.actions.server
base.automation
safe_eval code
service-user context
```

Determine whether the automation bypasses normal user authorization or can be triggered by an untrusted actor.

---

# 64. Service and Integration Users

For service users inspect:

```text
groups
companies
API access
record rules
sudo wrappers
allowed models
token lifecycle
```

Prefer least privilege rather than Administrator-equivalent service users.

---

# MULTI-WEBSITE SECURITY

# 65. Website Isolation

For website-aware models inspect:

```text
website_id
request.website
website-specific views
published records
domains
company mapping
```

Multi-website presentation does not automatically guarantee data isolation.

Test whether Website A can retrieve Website B's private records.

---

# 66. Public Website Search Domains

For public list/search routes, ensure the domain explicitly limits:

- publication state;
- website;
- company where relevant;
- ownership/category as required;
- private/archived records.

Do not use `sudo().search([])` and rely on templates to hide records.

---

# INTEGRATION AND WEBHOOK SECURITY

# 67. External Endpoint Authentication

For external callbacks/webhooks inspect:

```text
signature
secret
token
IP restriction if intentionally used
timestamp/nonce
provider verification
```

Do not trust a webhook merely because its URL is obscure.

---

# 68. Replay and Idempotency Security

For payment/webhook/integration callbacks, determine whether an attacker or provider retry can replay the same request.

Review:

```text
provider event ID
transaction ID
idempotency key
processed flag
unique constraint
timestamp/signature
```

Repeated callbacks must not duplicate privileged business effects.

---

# 69. Outbound Integration Data Exposure

Review data sent to third parties.

Confirm only required fields are sent.

Do not leak:

```text
internal IDs unnecessarily
secret fields
salary/cost data
unrelated partner data
attachments
tokens
```

---

# BUSINESS-WORKFLOW AUTHORIZATION

# 70. Sensitive Action Review

For privileged workflows such as:

```text
approve
post
cancel
reverse
refund
pay
validate
confirm
unlock
archive
delete
reset to draft
```

verify authorization exists server-side.

Button `groups=` alone is not enough.

---

# 71. State-Transition Authorization

Check whether RPC, automation, or direct method calls can invoke a transition outside the intended UI.

Verify:

```text
current state
actor group
record ownership/company
approval threshold
business constraints
```

---

# 72. Monetary and Sensitive Value Protection

For financial/HR/sensitive values inspect who can write:

```text
amount
price
discount
cost
salary
bank account
tax/fiscal fields
approval limits
```

Verify server-side access and business authorization.

---

# STATIC SECURITY REVIEW PROCEDURE

# 73. Security Search Checklist

For the target surface, search as relevant for:

```text
ir.model.access.csv
ir.rule
res.groups
implied_ids
groups=
groups_id
sudo(
with_user(
with_company(
has_group(
@http.route
auth=
csrf=
cors=
request.params
kwargs
search(
search_read(
browse(
ir.attachment
access_token
env.cr.execute
safe_eval
ir.config_parameter
ir.cron
ir.actions.server
base.automation
Markup
t-raw
innerHTML
```

Use targeted search, not blind repository-wide noise.

---

# 74. Security Diff Review

When reviewing an implemented change, inspect the actual Git diff.

Check for:

- newly added `sudo()`;
- removed group restriction;
- broader ACL;
- broader record-rule domain;
- removed company domain;
- public route auth change;
- CSRF/CORS change;
- new raw request fields;
- new report/download route;
- new token;
- new secret;
- direct SQL;
- route response expansion;
- sensitive field made unrestricted.

Do not validate only the files expected by the plan.

---

# RUNTIME SECURITY VALIDATION

# 75. Runtime Security Is Actor-Specific

When runtime access is available, test the actual actor context.

Examples:

```text
Administrator
allowed internal user
denied internal user
manager
portal user
public user
Company A user
Company B user
```

Do not mark security as verified from Administrator-only tests.

---

# 76. Positive and Negative Security Tests

For each security boundary include both:

```text
Allowed actor -> operation succeeds
Denied actor -> operation fails
```

For record isolation also test:

```text
Allowed record
Forbidden record
```

A positive test alone cannot prove authorization.

---

# 77. ACL Runtime Matrix

For relevant operations record:

```text
Actor:
Read:
Create:
Write:
Unlink:
Expected:
Actual:
Evidence:
```

Test only the operations material to the feature.

---

# 78. Record Rule Runtime Matrix

Use representative data.

Example:

```text
Actor: Salesperson A
Record: Own customer/order
Expected: Allowed

Actor: Salesperson A
Record: Salesperson B customer/order
Expected: Denied
```

Record the exact context and result.

---

# 79. Multi-Company Runtime Matrix

When relevant:

```text
Actor:
allowed_company_ids:
Record company:
Expected:
Actual:
```

Test at least the expected company and one forbidden company.

---

# 80. Public Route Runtime Test

For public routes, use an unauthenticated context when safe.

Verify:

- route reachable as expected;
- hidden/private record IDs denied;
- user-controlled IDs cannot cross authorization boundary;
- returned payload contains only intended fields;
- unsafe method/state changes are blocked.

---

# 81. Portal Runtime Test

Use a real portal-context user where possible.

Verify:

```text
own record allowed
other portal user's record denied
token behavior
attachment behavior
download/report behavior
```

---

# 82. RPC Runtime Test

When RPC exposure is material, invoke the relevant method using the actor that could realistically call it.

Verify UI-hidden operations cannot be called successfully by unauthorized users.

---

# 83. `sudo()` Abuse Test

When a user-provided identifier reaches sudoed code, test a record the caller should not access.

Expected result:

```text
Denied / filtered / not found for that caller
```

If arbitrary records become accessible, classify as a material security failure.

---

# 84. Attachment Runtime Test

When attachments are material, test:

```text
authorized download
unauthorized record attachment
guessed/changed attachment ID
portal/public context
token path if present
```

---

# 85. Security Test Data Safety

Use safe test/staging/cloned environments for state-changing security checks when practical.

On production, prefer the narrowest read-only/reversible verification permitted by project rules.

Do not:

- alter real user groups casually;
- expose real private records;
- change live ACLs/rules for testing;
- create real financial/security side effects;
- send real external notifications.

---

# FINDING CLASSIFICATION

# 86. Security Severity

Use:

```text
Critical
High
Medium
Low
Informational
```

Consider:

- unauthorized data disclosure;
- unauthorized modification;
- privilege escalation;
- cross-company leakage;
- public/portal exposure;
- financial impact;
- sensitive HR/customer data;
- exploit reachability;
- required attacker privileges;
- reversibility.

Explain the severity.

---

# 87. Evidence Confidence

Separate severity from confidence.

Use:

```text
Confirmed
Strong evidence
Probable
Possible
Needs runtime verification
```

A potentially Critical issue can still have `Needs runtime verification` confidence.

Do not lower severity solely because evidence is incomplete.

---

# 88. Finding Structure

For every material finding record:

```text
Finding ID:
Title:
Severity:
Confidence:

Protected asset:
Actor:
Entry path:
Security control expected:
Observed behavior:
Bypass path:

Repository evidence:
Runtime evidence:

Impact:
Exploit prerequisites:
Affected records/users/companies/websites:

Recommended safeguard:
Required validation:
```

Keep remediation focused on the security boundary, not a generic implementation plan.

---

# 89. Distinguish Design Gap From Runtime Failure

Use categories such as:

```text
DESIGN GAP
IMPLEMENTATION DEFECT
CONFIGURATION RISK
RUNTIME-ONLY RISK
EXTERNAL/INTEGRATION RISK
UNKNOWN / NEEDS VERIFICATION
```

This helps Plemo decide whether the fix belongs in code, data/configuration, deployment, or runtime validation.

---

# SECURITY BOUNDARY RECOMMENDATION

# 90. Recommend the Smallest Safe Security Boundary

When remediation is needed, state:

```text
Recommended module:
Recommended enforcement layer:
Expected files/components:
Do-not-touch areas:
Reason:
```

Examples of enforcement layers:

```text
ACL
record rule
server-side method group check
controller authorization
company domain
token validation
field groups
safe scoped sudo
input allowlist
integration signature
```

Do not create a second generic implementation plan.

---

# 91. Prefer Server-Side Enforcement

When a security requirement is currently implemented only in:

```text
button groups
view invisibility
JavaScript
frontend route hiding
template conditions
```

recommend an appropriate server-side authorization layer.

UI restrictions may remain for UX, but must not be the only security control for sensitive operations.

---

# 92. Avoid Over-Restricting Legitimate Workflows

Least privilege does not mean breaking required business access.

Before recommending stricter ACL/rules, identify:

```text
legitimate actor
legitimate workflow
batch/cron/integration caller
manager override
company scope
portal requirement
```

Security fixes must preserve intended business behavior.

---

# RELATIONSHIP WITH IMPLEMENTATION AND VALIDATION

# 93. Handoff to Plemo Native Planner

For an authorized fix/implementation task, output concise security evidence:

```text
SECURITY EVIDENCE

Protected asset:
Affected actors:
Entry paths:
Current controls:
Security gaps:
Required safeguards:
Recommended enforcement boundary:
Do-not-touch areas:
Runtime security tests required:
Remaining unknowns:
```

Then return control to Plemo's native planner.

Do not write a duplicate full execution plan.

---

# 94. Handoff to Regression & Runtime Validator

After implementation, provide Skill 4 with security-specific scenarios such as:

```text
allowed user succeeds
denied user fails
cross-company record denied
public route cannot access arbitrary ID
portal user cannot access another customer's record
sudo path remains scoped
attachment download respects record access
```

Skill 4 owns the broader post-change regression result.

---

# 95. Fix-and-Recheck Loop

If a security issue is found during an authorized fix task:

```text
Security finding
        ↓
Security evidence
        ↓
Plemo native planner/debugger
        ↓
Smallest safe fix
        ↓
Security-specific recheck
        ↓
Skill 4 broader regression/runtime validation when needed
```

Do not ask for duplicate approval solely to enter this loop.

---

# REQUIRED OUTPUT

# 96. Security Evidence Summary

For every material security review start with:

```text
SECURITY EVIDENCE

Project:
Feature:
Odoo version:
Review mode:

Protected asset:
Relevant actors:
Relevant entry paths:

ACL status:
Record-rule status:
Group/privilege status:
sudo/elevation status:
Controller/RPC status:
Portal/public status:
Multi-company status:
Multi-website status:
Token/attachment status:
Secret/integration status:

Critical findings:
High findings:
Medium findings:
Low findings:

Runtime verification required:
Do-not-touch boundary:
Remaining unknowns:
```

---

# 97. Access Matrix

For relevant actors include:

```text
Actor:
Path:
Operation:
Record scope:
Expected:
Observed/evidence:
Status:
```

Possible statuses:

```text
ALLOWED AS EXPECTED
DENIED AS EXPECTED
UNEXPECTEDLY ALLOWED
UNEXPECTEDLY DENIED
PARTIAL
NOT VERIFIED
```

---

# 98. Full Security Review Report

For complex reviews use:

## 1. Review Target

```text
Project:
Feature:
Requested change:
Modules:
Models:
Routes:
```

## 2. Environment and Version

```text
Odoo version:
Version evidence:
Runtime environment:
```

## 3. Protected Assets

List the sensitive data/operations.

## 4. Actor Matrix

List actors and expected permissions.

## 5. Access Paths

List UI, RPC, controller, portal, report, attachment, automation, integration paths.

## 6. ACL Analysis

List all relevant grants and effective access.

## 7. Record Rule Analysis

List rules, domains, composition, and effective record scope.

## 8. Groups and Privilege Analysis

Include implied groups and escalation paths.

## 9. Field-Level Security

Include related/computed/report/export leakage.

## 10. `sudo()` / Elevation Analysis

Include every relevant bypass path.

## 11. Controller / RPC Analysis

Include auth, authorization, input, response, CSRF/CORS when relevant.

## 12. Public / Portal Analysis

Include ownership, tokens, attachments, downloads.

## 13. Multi-Company Analysis

Include effective company isolation.

## 14. Multi-Website Analysis

Include website isolation and publication domains.

## 15. Token / Attachment / Document Analysis

Include authorization and IDOR risk.

## 16. Secrets / Configuration Analysis

Include storage and logging.

## 17. Automation / Service User Analysis

Include cron/server action execution identity.

## 18. Integration Security

Include webhook authentication, replay/idempotency, data exposure.

## 19. Runtime Security Validation

List actor-context tests actually performed.

## 20. Findings

Use the mandatory finding structure.

## 21. Security Boundary Recommendation

State the smallest safe enforcement boundary.

## 22. Runtime Validation Requirements

List exact tests Skill 4 should run.

## 23. Files Investigated

List important files actually examined.

## 24. Remaining Unknowns

List anything requiring runtime/database/external verification.

## 25. Security Assessment

Use one:

```text
ACCEPTABLE FOR REVIEWED SCOPE
REQUIRES REMEDIATION
PARTIAL
BLOCKED
```

Explain briefly.

---

# 99. Concise Output for Small Security Reviews

For a contained review use:

```text
Security Assessment:
ACCEPTABLE / REQUIRES REMEDIATION / PARTIAL / BLOCKED

Protected asset:
Actors:
Entry path:

Checks:
• ACL
• record rules
• group enforcement
• sudo/elevation
• runtime access if available

Findings:
...

Required safeguard:
...

Runtime verification:
...
```

Do not force the full report when the surface is small.

---

# 100. Security Assessment Definitions

Use:

```text
ACCEPTABLE FOR REVIEWED SCOPE
REQUIRES REMEDIATION
PARTIAL
BLOCKED
```

### ACCEPTABLE FOR REVIEWED SCOPE

No material security issue was identified in the reviewed scope, and required evidence for that scope is sufficient.

Do not interpret this as a guarantee that the entire Odoo deployment is secure.

### REQUIRES REMEDIATION

At least one material security issue or incorrect authorization boundary was identified.

### PARTIAL

Useful review evidence exists, but a required runtime/configuration/security context could not be verified.

### BLOCKED

The review cannot meaningfully establish the security boundary because required code, configuration, runtime context, or access is unavailable.

---

# 101. Production Safety

Do not change live security configuration merely to test it.

Do not:

- remove ACLs/rules;
- temporarily grant broad groups;
- expose a private route;
- alter real user roles;
- disable CSRF/authentication;
- publish private records;
- leak real data.

Prefer staging/cloned environments for state-changing security tests.

---

# 102. Pre-Existing Security Issues

When reviewing a change, separate:

```text
introduced by current change
pre-existing
possibly exposed by current change
unknown origin
```

Do not blame the current task for an old security weakness without evidence.

Do not silently expand scope to fix unrelated historical issues unless requested.

---

# 103. Hard Prohibitions

Never:

- replace Plemo's native planner or implementation workflow;
- require duplicate approval for an already-authorized security fix;
- silently edit code during audit-only mode;
- assume Odoo version;
- treat menu/view/button visibility as complete security;
- treat Administrator success as security proof;
- inspect only one ACL when other modules may grant access;
- assume a restrictive ACL cancels a broader ACL;
- ignore record-rule composition;
- ignore no-rule/default-allow risk when ACL grants access;
- recommend `sudo()` as a generic AccessError fix;
- use `sudo().browse(user_id)`-style access without authorization checks on user-provided IDs;
- return sudoed sensitive data directly to public/portal callers without filtering;
- ignore company isolation when using `sudo()`;
- assume `auth="user"` authorizes access to every record;
- assume `auth="public"` is safe because the route is hard to guess;
- disable CSRF merely to make a browser request work;
- trust user-supplied create/write dictionaries without field allowlisting;
- rely on JavaScript validation for authorization;
- expose secrets in source, frontend payloads, or logs;
- trust attachment IDs without record authorization;
- trust portal tokens without binding them to the intended record;
- concatenate untrusted input into SQL;
- use direct SQL without considering lost ORM security enforcement;
- expose dynamic domains/model/field names without reviewing abuse potential;
- ignore implied groups;
- ignore service-user/cron execution privileges;
- ignore exports/reports/related fields as data-leak paths;
- claim runtime security validation when only repository analysis was performed;
- claim a route is secure without testing the relevant user context when runtime proof is required;
- call the whole deployment secure based on one feature review;
- modify unrelated code merely to remove a warning;
- commit or push unless explicitly requested by the user/project workflow.

---

# 104. Core Decision Tree

```text
START
  |
  v
Is there a material security surface?
  |
  +-- No -> Do not force this skill.
  |
  +-- Yes
        |
        v
Read plemo.md / project rules.
        |
        v
Determine task mode.
        |
        v
Reuse Investigator / Impact evidence.
        |
        v
Detect Odoo version.
        |
        v
Identify protected asset.
        |
        v
Build actor matrix.
        |
        v
Build access-path map.
        |
        v
Review ACLs.
        |
        v
Review record rules and composition.
        |
        v
Review groups / implied groups / privilege.
        |
        v
Review field-level and derived-data exposure.
        |
        v
Is sudo/elevation involved?
        |
        +-- Yes -> trace exact bypass scope and user inputs.
        |
        v
Are controllers/RPC involved?
        |
        +-- Yes -> review auth, authorization, input, response,
        |          CSRF/CORS, IDOR, mass assignment.
        |
        v
Are portal/public paths involved?
        |
        +-- Yes -> ownership, tokens, attachments, downloads.
        |
        v
Is model company-aware?
        |
        +-- Yes -> multi-company isolation matrix.
        |
        v
Is website-aware?
        |
        +-- Yes -> multi-website isolation.
        |
        v
Are secrets/integrations involved?
        |
        +-- Yes -> storage, signatures, replay, data exposure.
        |
        v
Are automation/service users involved?
        |
        +-- Yes -> execution identity and privilege.
        |
        v
Runtime security proof required?
        |
        +-- Yes
        |      |
        |      +-- Environment available -> test relevant actors.
        |      |
        |      +-- Unavailable -> PARTIAL/BLOCKED.
        |
        v
Classify findings by severity and confidence.
        |
        v
Recommend smallest safe security boundary.
        |
        v
Is task audit-only?
        |
        +-- Yes -> report and stop.
        |
        +-- No, fix already authorized
              |
              v
        Hand evidence to Plemo native planner.
              |
              v
        Implement smallest safe fix.
              |
              v
        Recheck security-specific scenarios.
              |
              v
        Hand runtime requirements to Skill 4 when needed.
```

---

# 105. Primary Rules to Always Remember

```text
Security is about effective access, not only visible UI.

Reuse existing Plemo evidence.
Do not duplicate investigation.

Preserve Plemo native planning and task mode.
Do not require duplicate approval.

Use plemo.md and project rules first.

Identify:
protected asset
actor
entry path
server-side enforcement
bypass path

ACLs grant permissions.
Review all ACLs that affect the model.

Record rules define record scope.
Review their effective composition.

UI groups are not a substitute for server authorization.

Never use sudo() as a generic fix.
Every sudo path needs a reason, narrow scope, and caller authorization.

User-provided IDs + sudo() require explicit record authorization.

For public/portal:
authentication is not authorization.

For multi-company:
test company isolation explicitly.

For multi-website:
do not assume website isolation automatically protects data.

For field security:
check related/computed/report/export leakage.

For attachments/tokens:
bind access to the intended record and actor.

For controllers/RPC:
review auth, method, CSRF/CORS, input allowlists,
record authorization, response data, and known callers.

For secrets:
do not expose them in source, frontend, or logs.

For automation:
review the execution user and company context.

For runtime security:
test both allowed and denied actors.

Separate severity from confidence.

Output security evidence, not a second generic implementation plan.

After a fix:
recheck the security boundary,
then use Odoo Regression & Runtime Validator for broader validation when needed.
