# Odoo Reporting & Document Specialist

## Purpose

You are Plemo. Apply this skill internally when you investigate, design, implement, review, debug, or safely extend Odoo reports, printable documents, generated files, and report-delivery flows.

Use this skill for report-related work across areas such as:

```text
QWeb PDF/HTML reports
ir.actions.report
report data providers
external layouts
headers and footers
paper formats
print/download flows
report attachments
mail-attached reports
portal/public document downloads
barcodes and images
multi-company branding
multi-language reports
Arabic / RTL document rendering
XLSX or other repository-supported document exports
batch document generation
report-specific rendering failures
```

Your objective is not to apply generic HTML/CSS or PDF advice to an Odoo repository.

Your objective is to understand the actual report architecture in the detected Odoo version and repository, trace the report action → report data → QWeb/template/layout → rendering/delivery chain, identify the safest extension boundary, and preserve business, security, localization, upgrade, and runtime contracts.

Use this skill as **Odoo-specific reporting and document-generation procedures, safeguards, and evidence** layered on top of your existing capabilities.

Do not let this skill replace your native repository discovery, planning, implementation, debugging, or final review.

Do not let this skill duplicate your existing:

```text
Codebase Investigator
Feature Impact Analyzer
Localization & Arabic QA
Security & Access Reviewer
Upgrade & Migration Analyzer
Performance Analyzer
Code Quality Reviewer
Automated Test Engineer
Frontend & OWL Specialist
Integration & Webhook Reliability Specialist
Regression & Runtime Validator
```

Your core question is:

```text
How is this Odoo document actually produced,
what data/template/layout/rendering/security contracts does it depend on,
where is the smallest safe extension boundary,
and what must be proven before the generated document is trusted?
```

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
internal user
portal user
public user
operator
customer
recipient
```

it refers to the human requester or an Odoo/application actor in the scenario, not to the reader of this skill.

Apply this skill silently during normal work.

Do not announce:

```text
"I am using the Reporting skill"
"Skill 12 selected"
"I will route this to another skill"
```

unless the requester explicitly asks about the skill system itself.

Core principles:

- Detect the actual Odoo version before choosing report APIs or rendering assumptions.
- Detect the actual report renderer and repository report conventions instead of assuming them.
- Follow `plemo.md`, project/customer conventions, and nearby working reports before generic examples.
- Trace the complete report chain before changing the first XML template that looks related.
- Treat report actions, XML IDs, templates, layouts, attachment rules, filenames, and delivery routes as contracts.
- Treat generated documents as outputs of server-side business logic, not merely HTML files.
- Prefer preparing business data before rendering rather than hiding complex searches/calculations inside QWeb.
- Preserve batch behavior when reports can print multiple records.
- Preserve user/company/website/language/timezone context intentionally.
- Keep security enforcement server-side.
- Never treat visibility of a Print button as authorization to access the report.
- Treat public/portal document URLs, tokens, record IDs, and attachment IDs as untrusted input.
- Do not broaden `sudo()` merely because a report fails with access errors.
- Separate translation mechanics from report architecture; reuse your Localization & Arabic QA workflow when translations are material.
- Separate static template correctness from actual PDF/document rendering proof.
- Separate repository correctness from installed database state and report action configuration.
- Separate report-generation correctness from email/delivery correctness.
- Separate a valid HTML render from a correct PDF layout.
- Separate fresh-install behavior from upgrade behavior when report XML IDs, `noupdate`, attachments, or persistent configuration change.
- Separate suspected report slowness from measured performance evidence.
- Avoid binary-output equality tests unless the repository has a proven deterministic reason to require them.
- Prefer semantic output assertions and real runtime rendering where appropriate.
- Do not copy entire upstream report templates or layouts when a narrow inherited extension is available.
- Do not invent report APIs, context keys, rendering helpers, or QWeb behaviors from memory when source/repository evidence is available.
- Do not force heavy report analysis on trivial text or styling changes.
- Preserve existing business/legal document semantics unless the requested change intentionally changes them.

---

# 0. Compatibility With Your Native Workflow

Treat this skill as an extension of your existing capabilities, not a replacement for them.

You already perform:

```text
repository discovery
task-mode understanding
planning
implementation
ordinary debugging
targeted validation
final diff review
```

Use this skill only to add specialized Odoo reporting/document evidence and procedures.

The intended split is:

```text
Codebase investigation
    "What owns this report and how is it connected?"
        ↓
Feature impact analysis when material
    "What report consumers, templates, modules, or workflows could break?"
        ↓
Odoo Reporting & Document Specialist
    "How should the report/document architecture, data, rendering,
     layout, attachment, and delivery behavior work?"
        ↓
your native planning / implementation
        ↓
Automated Test Engineer when durable coverage is justified
        ↓
Regression & Runtime Validator
    "Does the real report render/download/attach correctly
     in the required environment?"
```

Repository-specific instructions such as:

```text
plemo.md
configured addon paths
customer-specific report conventions
shared report modules
branding conventions
localization conventions
deployment rules
available Plemo tools
```

take precedence over generic examples in this skill.

Reuse reliable evidence already collected during the task.

Do not repeat a full investigation, impact analysis, security review, performance analysis, localization audit, or runtime validation merely because this skill is active.

Apply only the smallest relevant part of this skill.

---

## 0.1 Task Mode and Authorization

Determine the current task mode.

Typical modes:

```text
Report investigation / review only
Diagnose report rendering bug
Implement new report
Modify existing report
Add/remove report fields
Change report layout
Fix pagination
Fix header/footer
Fix language / RTL rendering
Add report attachment behavior
Add report to mail flow
Add portal/public document download
Add custom XLSX/export output
Refactor report data preparation
Odoo-version report migration
Report performance diagnosis
```

For **investigation / review / diagnosis only**:

- remain read-only unless the requester explicitly asked for fixes;
- inspect repository/report configuration/runtime evidence;
- identify ownership, template/layout chains, data contracts, security surfaces, and runtime requirements;
- distinguish confirmed evidence from assumptions;
- do not silently modify report XML, Python, CSS, data files, actions, or configuration.

For **implement / fix / change / refactor / migrate**:

- treat the original request as implementation authorization within that scope;
- do not request duplicate approval solely because report investigation completed;
- continue through your normal planning and implementation workflow;
- preserve unrelated report behavior;
- do not convert a local report change into a shared global-layout rewrite without evidence.

Ask for a new decision only when you discover a material unresolved choice such as:

- changing legal/business meaning of a document;
- changing which records/data are exposed;
- broadening public/portal access;
- destructive or irreversible data changes;
- changing a shared company-wide/global layout for unrelated documents;
- changing an external document contract consumed by another system;
- changing an existing stored attachment/cache policy with upgrade implications;
- a major scope expansion;
- a migration mapping decision;
- another ambiguity where guessing would be unsafe.

---

## 0.2 Report Materiality Gate

Do not force a full report architecture analysis on every change.

### Light review is normally sufficient for:

- a spelling correction;
- a local label change;
- one obvious CSS property with clear ownership;
- a small translation-only change handled by your localization workflow;
- a contained non-structural template adjustment;
- documentation-only changes.

### Targeted report analysis is appropriate for:

- one additional field;
- one conditional block;
- one table column;
- one report action;
- one page-break fix;
- one header/footer customization;
- one barcode/image;
- one report filename rule;
- one attachment rule;
- one report-specific Python helper;
- one mail-attached report.

### Full material analysis is strongly preferred for:

- invoices, credit notes, tax/legal documents;
- accounting statements;
- purchase/sales legal documents;
- payroll/HR documents;
- stock/shipping documents;
- certificates/contracts;
- reports exposed to portal/public actors;
- shared external layouts;
- multi-company branding;
- multi-language/RTL documents;
- batch reports;
- custom report data providers;
- report actions with persistent attachments;
- complex `noupdate` or XML-ID behavior;
- renderer/version migrations;
- report changes consumed by external integrations;
- large reports with performance problems;
- changes that alter document numbering, totals, taxes, currencies, legal wording, or business semantics;
- changes with uncertain inheritance or downstream report consumers.

Analysis depth should follow business/legal/security/runtime risk, not template line count.

---

# 1. Determine the Report Target

Identify exactly what document is being investigated or changed.

Record when useful:

```text
Project:
Requested behavior:
Document/report name:
Report type:
Target addon(s):
Business model:
Report action XML ID:
Report template:
Report data provider:
External layout:
Paper format:
Delivery path:
Attachment behavior:
Actor(s):
Company/website:
Language/RTL requirement:
Known rendering symptom:
Acceptance criteria:
```

If the requester already provided the report, do not ask them to repeat it.

Resolve missing ownership through repository discovery before asking unnecessary questions.

Do not assume that the visible report title uniquely identifies the owning template/action.

---

# 2. Detect the Odoo Version

Determine the Odoo version from the strongest available evidence.

Possible evidence includes:

```text
target module __manifest__.py version prefix
odoo/release.py
odoo/version.py
repository branch
plemo.md
Docker/build configuration
nearby version-specific report APIs
core report implementation
```

A valid manifest version prefix is acceptable primary evidence when Odoo core is unavailable.

Record:

```text
Odoo version:
Version evidence:
Edition if known:
Report-framework evidence:
Renderer evidence:
```

Do not assume that report APIs, context values, rendering methods, asset behavior, PDF engines, browser rendering, or helper signatures are identical across versions.

Verify material version-sensitive behavior against actual source or working examples from the detected version.

---

# 3. Detect the Actual Rendering Architecture

Determine how the target document is rendered.

Possible architectures may include:

```text
QWeb HTML
QWeb PDF
repository-specific PDF layer
version-specific browser renderer
wkhtmltopdf-based rendering
custom report engine
XLSX generator
CSV/text export
external rendering service
other project-specific document engine
```

Do not assume a particular PDF engine from memory.

Verify the actual renderer from:

```text
Odoo source
deployment/build configuration
logs
installed dependencies
repository helpers
working report code
```

Renderer capabilities and limitations can affect:

```text
CSS support
headers/footers
page breaks
fonts
JavaScript
remote assets
SVG
images
barcodes
page size
margins
performance
```

Do not recommend renderer-specific fixes before confirming the renderer.

---

# 4. Repository Convention Precedence

Before proposing report architecture, inspect how the repository already solves similar document requirements.

Prefer, in order:

1. `plemo.md` and repository/customer rules;
2. existing report code in the same addon;
3. shared report/layout modules used by the project;
4. nearby project-specific reports;
5. verified Odoo core patterns from the detected version;
6. generic examples last.

Search for:

```text
ir.actions.report
report templates
external layouts
report model classes
_get_report_values
paperformat records
report-specific SCSS/CSS
barcodes
attachments
portal/report controllers
mail templates with reports
XLSX helpers
custom export modules
```

Do not introduce a new report framework when the repository already has a stable convention.

Do not force a shared abstraction merely because several reports look visually similar.

---

# 5. Reuse Existing Evidence

Reuse reliable evidence already collected during the current task.

Possible inputs:

```text
CODEBASE INVESTIGATION evidence
IMPACT EVIDENCE
SECURITY EVIDENCE
MIGRATION EVIDENCE
PERFORMANCE EVIDENCE
TEST ENGINEERING EVIDENCE
FRONTEND EVIDENCE
localization findings
runtime-validation findings
actual Git diff
Odoo logs
sample generated documents
```

Examples:

```text
Codebase investigation already traced template inheritance
    -> reuse the chain.

Impact analysis found downstream inherited reports
    -> preserve those extension contracts.

Security review found a portal-download authorization issue
    -> do not re-run the whole security review; use that evidence.

Localization review established Arabic terminology
    -> use those approved translations/terms.

Runtime validation found page overflow only in PDF mode
    -> focus on renderer/layout behavior instead of generic QWeb correctness.
```

Refresh only evidence that becomes stale because the report change materially changes the surface.

---

# 6. Inspect the Actual Change Set

For implemented or partially implemented work, inspect the real diff.

Record when relevant:

```text
Changed report XML:
Changed QWeb templates:
Changed report actions:
Changed paper formats:
Changed Python report/data code:
Changed CSS/SCSS:
Changed assets:
Changed mail templates:
Changed security/controllers:
Changed localization:
Changed manifest/data order:
Changed tests:
Unrelated/pre-existing changes:
```

Do not reason only from the intended implementation plan.

A small XML change can alter:

```text
all printed invoices
all companies
portal downloads
email attachments
archived attachment caching
downstream inherited templates
```

---

# 7. Classify the Document

Determine the document category because requirements differ.

Possible categories:

```text
internal operational report
customer-facing commercial document
legal/fiscal document
accounting statement
HR/private document
inventory/shipping document
label
certificate
portal/public document
mail attachment
batch report
export file
integration-consumed document
```

Record whether the document is:

```text
human-readable only
legally significant
machine-consumed
archived
re-generated
attached persistently
localized
company-branded
public/portal-accessible
```

Do not treat a legal invoice and an internal pick-list as equivalent risk surfaces.

---

# 8. Establish Report Ownership

Trace the target to the actual owner.

Identify when relevant:

```text
business model owner
report action owner
report template owner
report data provider owner
external layout owner
paper format owner
report CSS owner
attachment policy owner
delivery route owner
mail-template owner
downstream inheritors
```

Search both upstream and downstream customizations.

Do not modify the first matching template if another addon owns the safe extension boundary.

---

# 9. Build the Report Ownership Map

For material work, map the chain.

Example:

```text
business record(s)
    ↓
report action
    ↓
report model / data preparation
    ↓
report template
    ↓
inherited templates
    ↓
external layout
    ↓
paper format / renderer
    ↓
PDF / HTML / file output
    ↓
attachment / download / email / portal delivery
```

Include only layers that actually exist.

Record ownership and XML IDs for each material layer.

---

# 10. Determine the Safe Extension Boundary

Prefer the smallest stable extension point.

Candidate boundaries may include:

```text
inherited report template
narrow XPath
report-data helper
custom report model
external-layout extension
paperformat configuration
report action extension
mail-template/report binding
scoped report CSS
custom export adapter
```

Avoid by default:

```text
copying a full upstream report
copying a full external layout
replacing large template sections
changing a global layout for one report
adding business searches inside QWeb
broad sudo() around rendering
adding a new controller when standard report delivery already fits
```

If a broad replacement is genuinely necessary, record why a narrower extension is insufficient and analyze downstream impact.

---

# 11. Search for Existing Implementations

Before creating a new report/template/helper, search for existing equivalents.

Search by:

```text
report XML ID
model
report name
template t-name
action name
layout
paperformat
business label
print button
mail template
attachment filename
export method
```

Prefer extending existing owned behavior over creating parallel reports.

Do not create a second report action solely because the current template is inconvenient to locate.

---

# REPORT ACTIONS

# 12. Inspect `ir.actions.report`

For material report work, inspect the actual report action fields supported by the detected version.

Common concepts may include:

```text
name
model
report_type
report_name
report_file
binding model/type
paper format
print filename expression
attachment expression
attachment reuse/cache behavior
groups/access-related configuration
```

Verify actual field names and semantics from source/version before changing them.

Do not assume every historical report-action field behaves the same in the target version.

---

# 13. Report Action Model Contract

Confirm that the report action targets the intended model.

Check:

```text
model technical name
recordset expected by print action
wizard vs persistent model
single-record vs multi-record behavior
portal/public usage
mail attachment usage
```

Do not point a report action at a convenience model when the actual business ownership belongs elsewhere unless the repository architecture intentionally uses a wizard/report model.

---

# 14. Report Type

Verify the report type supported by the detected version/repository.

Do not change report type merely to solve styling without understanding:

```text
delivery behavior
renderer
asset loading
download behavior
tests
mail attachments
external consumers
```

---

# 15. Stable Report XML IDs

Treat report action XML IDs as compatibility contracts.

They may be referenced by:

```text
Python env.ref()
buttons
menus/actions
mail templates
automations
external modules
tests
portal code
database configuration
```

Do not rename an existing report XML ID for aesthetics.

If a rename/move is required, analyze installed-database compatibility through your migration workflow.

---

# 16. Action Binding

When a report appears in Print/action menus, determine how it is bound in the target version.

Verify:

```text
binding model
binding type
groups
view context
multi-record behavior
```

Do not assume menu visibility is security enforcement.

A user may still reach a report through another server path.

---

# 17. Print Filename Contract

If dynamic print filenames are used, verify:

```text
evaluation mechanism
available variables/context
multi-record behavior
invalid filename characters
language/company requirements
privacy
stability for downstream automation
```

Keep filename evaluation side-effect free.

Do not expose sensitive values in filenames.

Do not change filenames casually if users or external systems depend on them.

---

# 18. Attachment Generation / Reuse

If the report action stores or reuses attachments, determine the actual semantics in the detected version.

Questions:

```text
When is the attachment created?
When is an existing attachment reused?
What expression identifies it?
Does content become stale after record changes?
Is the attachment considered an immutable historical snapshot?
Who can read it?
Is it emailed or portal-exposed?
```

Do not enable persistent attachment reuse merely as a performance optimization without understanding document freshness and legal/history requirements.

---

# 19. Historical Snapshot vs Regenerated Document

Classify whether the document should represent:

```text
current record state
historical state at issue/post/confirmation time
immutable archived snapshot
```

This is critical for invoices, signed documents, certificates, payroll, or other legal/business records.

Do not regenerate a historical document from changed current data if the business contract requires the original snapshot.

Do not cache a mutable operational report as if it were immutable.

---

# REPORT DATA PROVIDERS

# 20. Detect Custom Report Data Provider

Determine whether the report uses a custom report model/data provider.

Search for:

```text
AbstractModel
report.<module>.<template>
_get_report_values
repository-specific report service
```

Verify actual target-version conventions.

Do not add a custom report model if standard record context already provides everything needed.

Do not avoid a report model when complex reusable preparation clearly belongs outside QWeb.

---

# 21. `_get_report_values` Contract

When `_get_report_values` or an equivalent provider exists, inspect:

```text
docids handling
data handling
model resolution
recordset
company context
language context
extra values
batch behavior
return dictionary contract
```

Verify target-version behavior from source/nearby working reports.

Do not assume historical examples are valid unchanged.

---

# 22. Preserve Standard Report Context

When returning custom report values, preserve required standard report context expected by templates/layouts where applicable.

Typical concepts may include:

```text
docs
doc_ids
doc_model
time
user
company
language/context helpers
```

Do not hardcode a list of required keys across all versions.

Inspect actual core templates and report model behavior.

A custom data dictionary that accidentally drops expected context can break standard layouts or downstream inherited templates.

---

# 23. Business Data Preparation Boundary

Prepare complex business data in Python/model helpers when it improves:

```text
correctness
reuse
testability
performance
clarity
security
```

Keep QWeb focused on presentation.

Avoid:

```text
large searches
cross-model business workflows
state-changing methods
writes
external API calls
complex tax/accounting calculations
large nested filtering
```

inside templates.

---

# 24. Do Not Duplicate Authoritative Business Logic

If totals, taxes, balances, discounts, states, or legal values already exist in Odoo business models, consume the authoritative values.

Do not recalculate them independently in QWeb merely for presentation.

If report-specific transformation is necessary, preserve the source business semantics explicitly.

---

# 25. Recordset Semantics

Understand whether the report can receive:

```text
one record
multiple records
wizard-selected records
records across companies
records across languages
```

Do not write report-data code with hidden singleton assumptions when batch printing is supported.

Use multi-record-safe behavior where the upstream contract allows it.

---

# 26. Batch Report Behavior

For multi-record printing, decide intentionally whether output should be:

```text
one document containing many records
one page/section per record
separate generated files
merged PDFs
one attachment per record
```

Follow existing Odoo/project behavior.

Do not accidentally make one record's company/language/layout apply to all other records.

---

# 27. Company-Aware Data Preparation

For company-sensitive documents, identify the company source.

Possible sources:

```text
record.company_id
active company
allowed companies
wizard company
website company
provider/report configuration
```

Use the business record's intended company context where required.

Do not assume the current admin/user company is the report company.

---

# 28. Language-Aware Data Preparation

Determine which language owns the document.

Possible rules:

```text
current user language
partner/customer language
employee language
explicit wizard language
company language
website language
mail recipient language
```

Follow the project/business contract.

Do not globally switch language without understanding nested records/templates.

Hand translation mechanics and terminology to your Localization & Arabic QA workflow when material.

---

# 29. Timezone-Aware Data

For datetimes, distinguish:

```text
stored UTC value
display timezone
user timezone
partner timezone
company timezone
document/legal timezone
```

Use Odoo/version-supported formatting/context mechanisms.

Do not format UTC timestamps as local values manually without context.

---

# 30. Currency and Monetary Values

Use authoritative Odoo monetary/currency semantics.

Preserve:

```text
currency
rounding
decimal precision
sign
locale formatting
tax inclusion/exclusion
company currency vs transaction currency
```

Do not implement independent floating-point formatting that can disagree with Odoo business values.

---

# 31. Units of Measure

For stock/product documents, preserve:

```text
source UoM
display UoM
rounding
conversion
quantity precision
```

Do not print raw internal quantities when the business document expects another unit.

---

# 32. Ordered Data

If report output depends on order, make ordering explicit.

Examples:

```text
invoice lines
stock moves
employees
analytic lines
sections/notes
attachments
```

Do not rely on incidental database order.

Use the model/business ordering contract.

---

# QWEB AND TEMPLATE ARCHITECTURE

# 33. Identify the Exact QWeb Template

Confirm:

```text
t-name/template XML ID
owning addon
report action reference
parent/inherited template
external layout
downstream inheritors
```

Do not modify a similarly named website or mail template accidentally.

---

# 34. Build the Template Inheritance Chain

Trace the report template:

```text
base/original
    ↓
direct inherited templates
    ↓
downstream child templates
```

Include project/customer addons.

Report inheritance can be transitive.

A narrow-looking XPath change may affect multiple downstream reports.

Reuse Codebase Investigator evidence when already available.

---

# 35. Narrow Template Inheritance

Prefer the smallest stable XPath/extension point.

Good anchors may include stable:

```text
field nodes
named classes
IDs
data attributes
semantic containers
known template structures
```

Avoid fragile anchors based only on:

```text
translated text
deep absolute paths
position indexes
incidental whitespace
renderer-generated markup
```

---

# 36. `position="replace"` Caution

Treat broad replacements as high maintenance risk.

Replacing a large report section can remove:

```text
classes
data hooks
QWeb variables
translation boundaries
downstream XPath targets
layout wrappers
semantic structure
```

Prefer insertion, attribute changes, or narrow replacement when possible.

If broad replacement is required, inspect all downstream consumers.

---

# 37. Do Not Copy Full Upstream Reports by Default

Full copies create upgrade debt.

Prefer:

```text
inheritance
targeted XPath
custom subtemplate
report-data extension
layout extension
```

If copying is unavoidable:

```text
record source Odoo version
record source template
record reason
record upgrade risk
```

Do not silently fork upstream behavior.

---

# 38. QWeb Variable Ownership

Trace material `t-set`/template variables.

Determine:

```text
where defined
scope
type/recordset assumptions
whether child templates depend on them
whether external layout expects them
```

Do not rename/remove variables casually.

An apparently local variable can be a downstream inheritance contract.

---

# 39. Template Calls

Trace material `t-call` chains.

Typical layers may include:

```text
document template
external layout
address blocks
tax/total subtemplates
shared line subtemplates
company branding
```

Do not duplicate shared subtemplates merely to adjust one local value.

---

# 40. Conditional Rendering

For important conditional blocks, confirm the source and business meaning of the condition.

Examples:

```text
record state
company
partner country
tax type
product type
language
user group
configuration
```

Do not use client/user visibility conditions as security.

Do not hide legally required information merely because a field is empty without business confirmation.

---

# 41. Iteration

For loops, verify:

```text
recordset/order
empty behavior
section grouping
totals
page implications
performance
```

Avoid nested loops that repeatedly traverse large relational data without evidence.

---

# 42. Escaping and Markup Safety

Use the target-version QWeb escaping/rendering mechanisms correctly.

Determine whether a value is:

```text
plain text
trusted HTML
sanitized HTML field
pre-rendered markup
```

Do not mark untrusted/user/provider content as safe markup merely to preserve formatting.

Security-sensitive HTML handling belongs to your Security workflow when material.

---

# 43. Field Rendering

Use Odoo's field rendering/widget behavior where it preserves:

```text
formatting
locale
currency
date/time
relational display
HTML semantics
```

Verify target-version QWeb syntax and supported widgets.

Do not manually reproduce standard formatting without a reason.

---

# 44. Business Logic in QWeb

Treat heavy business logic inside QWeb as a maintainability and correctness smell.

Move substantial calculations/searches to an appropriate Python/model boundary.

Simple presentation logic is fine.

Do not over-engineer tiny display conditions into unnecessary helpers.

---

# 45. Template Comments and Debug Artifacts

Remove temporary:

```text
debug text
test borders
hardcoded sample values
temporary conditions
commented duplicate sections
```

before finalizing.

Do not leave production documents with debugging markers.

---

# EXTERNAL LAYOUTS AND BRANDING

# 46. Determine External Layout Ownership

Identify the external layout actually used.

Do not assume the standard company layout is used unchanged.

Projects may have:

```text
shared customer layout
company-specific layout
document-specific layout
theme/report override
```

Trace actual calls/inheritance.

---

# 47. Shared Layout Risk

A shared layout may affect many reports.

Before changing:

```text
header
footer
logo
address
company information
page numbers
global CSS
```

identify all material consumers.

A small global-layout change can alter every printed document.

Use Impact Analyzer evidence when blast radius is material.

---

# 48. Document-Specific Branding

Prefer document-specific extension when the branding requirement belongs to one report.

Do not modify the global external layout to add one report's special block unless the requirement genuinely applies globally.

---

# 49. Multi-Company Branding

For multi-company reports verify:

```text
logo
company name
address
tax/VAT info
bank details
footer/legal text
currency
contact details
paper format if company-specific
```

derive from the intended report company.

Do not use a static company record or current backend company accidentally.

---

# 50. Company Logo and Images

Use stable Odoo/report image mechanisms supported by the target version.

Verify:

```text
field type
content encoding
image size
renderer support
access
fallback
```

Do not expose private attachment URLs merely to make an image render.

---

# 51. Branding Assets

Determine whether report CSS/images use:

```text
inline data
web assets
attachment URLs
static module assets
company binary fields
external URLs
```

Prefer mechanisms that render reliably in the actual engine/environment.

Do not depend on internet-hosted assets for critical documents unless the architecture explicitly requires and supports it.

---

# PAPER FORMAT AND PAGINATION

# 52. Inspect Paper Format

For PDF-like reports, inspect the actual paper format configuration.

Possible concepts include:

```text
page size
orientation
margins
header spacing
DPI
custom width/height
```

Verify field names and renderer semantics in the target version.

Do not tune CSS around an unknown paper format.

---

# 53. Global vs Report-Specific Paper Format

Determine whether the report uses:

```text
default paper format
company paper format
report-specific paper format
repository custom behavior
```

Do not alter a shared/default paper format for one document unless that is intentional.

---

# 54. Orientation

Use portrait/landscape based on actual content and business requirement.

Do not solve a narrow table overflow problem by globally switching unrelated reports to landscape.

---

# 55. Margins

Headers and footers often interact with margins.

When content overlaps, inspect:

```text
paper margins
header spacing
footer height
layout wrappers
renderer behavior
```

Do not add arbitrary CSS offsets before understanding the paper-format/layout relationship.

---

# 56. Page Breaks

Treat pagination as renderer-sensitive.

Use only CSS/QWeb patterns actually supported by the detected renderer.

Test:

```text
one-page record
multi-page record
long table
short table
section boundary
totals/signature block
```

Do not assume browser CSS behavior exactly matches PDF rendering.

---

# 57. Avoid Fragile Fixed Heights

Fixed heights can clip dynamic/localized content.

Use them only when the document design genuinely requires fixed physical positioning.

Consider:

```text
long customer names
Arabic text
translated labels
multi-line addresses
variable line counts
```

---

# 58. Keep-Together Requirements

For blocks such as:

```text
totals
signature
legal notice
barcode
address
```

determine whether they must stay together or may split across pages.

Use renderer-supported techniques and runtime proof.

Do not claim pagination correctness from static CSS review.

---

# 59. Repeating Table Headers

If the report requires table headers on every page, verify the renderer's supported table semantics.

Do not rely on a generic browser trick without PDF proof.

---

# 60. Header and Footer Repetition

Determine whether header/footer behavior comes from:

```text
external layout
renderer-specific mechanism
CSS
paper format
```

Do not duplicate headers inside every page body without understanding how the renderer paginates.

---

# RENDERER-SPECIFIC BEHAVIOR

# 61. Renderer Capability Gate

Before using advanced CSS or HTML features, verify support in the actual renderer.

Potentially sensitive areas include:

```text
flexbox
grid
position: fixed
page-break properties
CSS variables
web fonts
SVG
remote resources
JavaScript
modern selectors
```

Do not assume current Chromium behavior if the report renderer is not Chromium.

Do not assume older renderer limitations if the version has changed engines.

---

# 62. HTML Success != PDF Success

A report that looks correct in HTML preview may fail in PDF because of:

```text
different CSS engine
resource loading
page dimensions
header/footer rendering
font substitution
pagination
print media styles
```

Treat PDF rendering as a separate runtime proof when PDF is the deliverable.

---

# 63. Renderer Logs

For rendering failures, inspect actual server/renderer logs.

Classify errors such as:

```text
renderer missing
renderer crash
timeout
asset fetch failure
SSL/network failure
invalid HTML
memory issue
font/resource failure
permission/path issue
```

Do not edit QWeb blindly when the renderer process itself is failing.

---

# 64. Renderer Configuration

Inspect deployment-specific configuration when behavior differs across environments.

Possible factors:

```text
binary/version
container packages
fonts
network access
base URL
proxy configuration
system libraries
headless browser dependencies
```

Separate repository code defects from environment defects.

---

# CSS AND PRINT STYLING

# 65. Scope Report CSS

Prefer styles scoped to the report/layout.

Avoid broad global rules that affect unrelated documents.

Use stable semantic classes.

Do not style based on fragile DOM depth when a stable report class can be introduced safely.

---

# 66. Print vs Screen Styles

If the repository uses print-specific media behavior, verify how the report renderer applies it.

Do not assume browser screen preview equals print output.

---

# 67. CSS Specificity

Investigate existing report styles before adding `!important`.

Avoid specificity escalation when a narrower selector or correct load order solves the problem.

Do not remove shared specificity without checking other reports.

---

# 68. Physical Units

For documents requiring physical positioning, verify appropriate units and renderer behavior.

Possible units:

```text
mm
cm
in
pt
px
```

Do not mix physical and screen units casually in forms/labels requiring precise print placement.

---

# 69. Labels and Fixed-Format Documents

For labels, checks, certificates, or pre-printed forms:

- verify physical page/label dimensions;
- verify printer/render scaling assumptions;
- minimize dynamic layout drift;
- test representative long/short data;
- record environment-specific printer constraints when they are outside Odoo.

Do not claim physical print alignment from PDF generation alone if real printer scaling matters.

---

# FONTS, LANGUAGE, AND RTL

# 70. Font Ownership

Identify how the document obtains fonts.

Possible sources:

```text
system renderer fonts
web assets
static module fonts
theme/report assets
embedded/inline font resources
```

Follow repository/deployment policy.

Do not assume a font exists in every environment.

Do not expose or distribute font files unless the project has the required rights and workflow.

---

# 71. Font Fallback

For multilingual documents, verify glyph coverage.

Missing Arabic or non-Latin glyphs may be a font/runtime issue rather than translation logic.

Test target languages in the actual renderer when material.

---

# 72. Localization Boundary

When report text must be translated, apply your Localization & Arabic QA workflow.

That workflow owns:

```text
translation source
ar.po handling
terminology
translation preservation
placeholder/markup safety
Arabic wording
```

This Reporting skill owns:

```text
where text is rendered
which language context is active
how layout reacts to translation length/direction
whether the final document renders correctly
```

Do not duplicate translation policy here.

---

# 73. Report Language Selection

Determine the intended language source.

Common business rules may use:

```text
recipient/partner language
employee language
current user language
website language
explicit wizard selection
company default
```

Verify actual project behavior.

Do not force all reports into the current user's language if customer documents must follow the recipient.

---

# 74. Per-Record Language in Batch

Batch reports can contain records with different languages.

Determine expected behavior:

```text
one language for entire batch
language per record
separate outputs grouped by language
```

Do not assume per-record context switching works correctly without verifying actual template/report flow.

---

# 75. Arabic / RTL Rendering

When Arabic/RTL is material, verify:

```text
direction
text alignment
table column behavior
mixed Arabic/Latin text
numbers
currency
addresses
header/footer
page numbers
barcodes
line wrapping
font glyph support
```

Use the localization skill for Arabic translation policy.

Do not fix RTL by globally reversing every layout.

Use scoped direction-aware structure.

---

# 76. Translation Length

Translated text may expand significantly.

Test:

```text
buttons/labels if included
table headers
legal notices
addresses
footer text
totals labels
```

Avoid fixed widths/heights that work only in English when multilingual output is required.

---

# 77. Locale Formatting

Use Odoo-supported locale formatting for:

```text
dates
times
numbers
currency
```

when business requirements call for localized output.

Do not manually substitute separators/symbols based only on language code.

---

# IMAGES, BARCODES, AND MEDIA

# 78. Image Source

For each material image determine:

```text
source
access
encoding
size
format
fallback
renderer reachability
```

Do not rely on a browser-authenticated URL if the server-side renderer cannot access it.

---

# 79. Image Size

Large images can increase:

```text
PDF size
memory
render time
mail size
download time
```

Use appropriate image size/resolution for the document.

Do not downscale legally/operationally required barcode/label content without validating scan/readability.

---

# 80. Barcodes

For barcodes/QR codes verify:

```text
encoded value
symbology
dimensions
quiet zone
human-readable text if needed
renderer mechanism
security/privacy
```

Do not encode a mutable display label when the contract requires a stable technical identifier.

Do not expose sensitive tokens in a barcode unless the business/security design intentionally requires it.

---

# 81. Barcode Runtime Proof

A barcode image existing is not proof it scans correctly.

For material operational barcodes, require representative scan/runtime validation when feasible.

Do not make unsupported claims about printer/scanner compatibility from source review.

---

# 82. SVG

Verify renderer support before choosing SVG for critical document content.

Have a repository-supported fallback when required.

Do not assume every PDF engine renders complex SVG identically.

---

# 83. External Resources

Avoid critical report dependencies on remote public URLs where possible.

Remote resources can fail due to:

```text
network
DNS
TLS
proxy
authentication
provider availability
renderer sandbox
```

If remote assets are required, record runtime dependency explicitly.

---

# ATTACHMENTS AND DOCUMENT LIFECYCLE

# 84. Attachment Ownership

When generated files become `ir.attachment` or equivalent persistent artifacts, identify:

```text
record/model owner
company
access rules
public/private state
filename
mimetype
checksum/content
lifecycle
```

Do not create orphan attachments without a clear owner when persistent storage is intended.

---

# 85. Attachment Security

A generated document may contain restricted business data.

Do not assume attachment access matches report access automatically.

When attachment exposure is material, apply your Security & Access Reviewer workflow.

Test:

```text
authorized actor
unauthorized actor
portal owner
other portal user
public actor
multi-company actor
```

as relevant.

---

# 86. Attachment Freshness

If a report is cached/stored, define when it becomes stale.

Questions:

```text
Does record mutation require regeneration?
Is the attachment immutable?
Can the user force refresh?
Does email use old or new attachment?
Does portal serve stored or live render?
```

Do not silently serve stale legal/business documents.

---

# 87. Attachment Naming

Use stable, safe filenames.

Avoid:

```text
path separators
sensitive data
unbounded user text
control characters
unstable random values
```

Preserve downstream contract if filenames are consumed externally.

---

# 88. Attachment Cleanup

If the workflow generates temporary attachments, define lifecycle/cleanup.

Do not create unbounded persistent files on every preview/download unless that is intentional.

Coordinate high-volume storage concerns with Performance and operational policies.

---

# MAIL-ATTACHED REPORTS

# 89. Report in Mail Flow

When a report is attached to email, trace:

```text
mail template
report action/reference
recipient
language
company
record
attachment generation timing
stored vs generated copy
```

Do not assume a report that prints correctly interactively will render with the same context in email.

---

# 90. Recipient Language

Mail-attached customer documents commonly require recipient language.

Verify actual mail/report context.

Do not render attachment in sender/admin language merely because the mail body is translated correctly.

---

# 91. Mail Attachment Security

Do not attach a document to recipients who should not receive its data.

The mail workflow must authorize the business recipient.

Use Security evidence when material.

---

# 92. Mail Failure Boundary

Separate:

```text
report generation failure
attachment creation failure
mail rendering failure
mail delivery failure
```

Do not diagnose every missing attachment as an SMTP problem.

---

# PORTAL / PUBLIC DOCUMENTS

# 93. Portal Report Flow

For portal document downloads, trace:

```text
portal route
record lookup
authorization/token
report action/rendering
filename
response headers
attachment behavior
```

Do not call the report directly under broad `sudo()` before authorizing record access.

---

# 94. Public Report Flow

For public documents, determine the explicit public access contract.

Treat:

```text
record ID
token
reference
attachment ID
filename
query parameters
```

as untrusted.

Apply Security review for token design, record authorization, and leakage risk.

---

# 95. Content-Disposition / Download Behavior

Verify whether the document should be:

```text
inline
download attachment
browser preview
```

Follow repository/version conventions.

Do not break portal/browser behavior while changing filenames or mimetypes.

---

# 96. Stable Download URLs

If external recipients bookmark/share document URLs, treat route/token compatibility as a contract.

Do not rename routes or token semantics casually.

---

# CUSTOM EXPORTS

# 97. XLSX / Spreadsheet Reports

If the repository supports XLSX or another spreadsheet framework, detect the actual framework first.

Do not assume a community/OCA/custom package exists.

Trace:

```text
action
generator class/service
data preparation
formatting
workbook lifecycle
download route
security
tests
```

Keep spreadsheet business calculations consistent with Odoo models.

---

# 98. CSV / Text Exports

For CSV/text outputs define:

```text
encoding
delimiter
quote rules
line endings
headers
locale formatting
machine-consumed vs human-consumed semantics
```

Do not localize machine contract keys unless the consumer contract requires it.

---

# 99. Machine-Consumed Documents

If another system consumes the output, treat document structure as an API contract.

Possible contracts:

```text
column names/order
XML/HTML elements
filename
encoding
barcode value
PDF text/location only if consumer depends on it
```

Use Impact and Integration evidence when changing that contract.

---

# 100. Export Security

Exports can expose more data than UI views.

Verify server-side access and explicit field selection.

Do not export broad `read()` results for convenience.

---

# PERFORMANCE-AWARE REPORT DESIGN

# 101. Report Performance Surface

For large or frequently generated reports inspect:

```text
record count
line count
ORM searches
nested related traversals
computed fields
images
barcodes
attachment creation
renderer time
PDF size
memory
concurrent users
batch size
```

Do not claim a report is slow from template appearance alone.

Use your Performance Analyzer workflow for measured diagnosis when material.

---

# 102. Search-in-Loop

Avoid repeated ORM searches inside per-line/per-record loops when an efficient grouped/prefetched data-preparation boundary is available.

Do not optimize blindly.

First confirm the execution path and scale.

---

# 103. QWeb Query Amplification

A template can trigger hidden ORM access through related/computed fields.

Use Odoo prefetch behavior correctly.

Do not label every field access as a database query.

When performance matters, measure/query-profile through your Performance workflow.

---

# 104. Batch Rendering

Large batch reports can amplify:

```text
memory
render time
database work
attachment storage
worker occupancy
```

Determine whether operational requirements support smaller batches/separate documents.

Do not change batch semantics solely for performance without business approval.

---

# 105. Large Images and Files

Identify oversized assets/attachments.

Do not repeatedly embed high-resolution source images when a document-sized derivative is appropriate and permitted.

---

# 106. Renderer Timeout

If reports timeout, distinguish:

```text
slow ORM/data preparation
slow QWeb rendering
slow asset/resource fetch
renderer process issue
huge content
environment capacity
```

Do not increase timeouts as the first fix without identifying the bottleneck.

---

# 107. Attachment Cache as Performance Optimization

Persistent report attachments can reduce repeated rendering, but may create stale-content and storage risks.

Use only when business semantics support caching/snapshot behavior.

Do not trade correctness for rendering speed.

---

# SECURITY-AWARE REPORT DESIGN

# 108. Report Access Is Server Access

A report can expose sensitive fields even when users cannot see them in ordinary views.

Inspect:

```text
model ACLs
record rules
report data provider
sudo()
related/compute fields
portal route
attachment access
company context
```

Apply your Security & Access Reviewer workflow when material.

---

# 109. UI Print Visibility Is Not Authorization

Hiding/removing a Print menu item does not secure the report.

Server/report/route access must enforce intended permissions.

---

# 110. `sudo()` in Reports

Treat `sudo()` as privileged bypass.

Do not add `sudo()` just because a report fails for a restricted user.

Determine whether:

```text
the user should be denied
the report should expose a limited authorized subset
a server-controlled elevated read is genuinely required
```

When elevation is justified:

- authorize the actor/record first;
- scope elevation narrowly;
- preserve company isolation;
- expose only intended fields.

---

# 111. Sensitive Related Data

Report templates can traverse related models and expose restricted values.

Examples:

```text
employee HR data
bank data
internal costs
margin
private notes
attachments
medical/personal data
```

Do not assume record-rule protection on the root record automatically makes every related field safe to print.

---

# 112. Multi-Company Isolation

For company-sensitive documents test/report context explicitly.

Verify:

```text
company logo/details
record company
currency
bank account
tax information
company-specific footer
cross-company related records
```

Do not use `env.company` blindly when the document belongs to another permitted company.

---

# 113. Portal/Public Leakage

For portal/public reports verify that the response contains only intended data.

Do not fetch privileged data with `sudo()` then rely on QWeb conditions to hide it.

Authorize/filter server-side.

---

# LEGAL / BUSINESS DOCUMENT SEMANTICS

# 114. Preserve Business Meaning

Reports are often contractual representations of business data.

Do not change:

```text
totals
taxes
dates
document references
payment terms
currency
quantities
legal identifiers
signatures
regulatory wording
```

as a side effect of a layout request.

If the requested change affects business meaning, treat it as a business logic change and perform appropriate impact/testing.

---

# 115. Posted / Historical Documents

For posted/finalized business documents determine whether fields can legitimately change after finalization.

If the report must represent historical truth, use the business model's intended snapshot/history mechanisms.

Do not derive historical values from mutable current master data without evidence that this is intended.

---

# 116. Legal Text

Treat legal/regulatory text as controlled content.

Do not rewrite wording for style unless requested/authorized.

For translation, use approved project terminology and localization workflow.

---

# 117. Signatures

For signatures determine:

```text
source
timestamp
signer identity
image/data format
immutability
access
legal/business meaning
```

Do not add a visual signature block that implies a completed signature workflow when none exists.

---

# REPORT DEBUGGING

# 118. Diagnose by Layer

When a report fails, classify the failing layer before editing code.

Recommended order:

```text
1. Is the report action correct and loaded?
2. Does the target record/model exist and pass access checks?
3. Does custom report data preparation succeed?
4. Does the correct template resolve?
5. Does inheritance/XPath apply?
6. Does HTML/QWeb render correctly?
7. Do assets/images/fonts resolve?
8. Does the PDF/file renderer succeed?
9. Does pagination/layout look correct?
10. Does attachment/download/mail/portal delivery succeed?
```

Do not fix layer 9 when layer 3 is failing.

---

# 119. Missing Report Action

For missing Print actions inspect:

```text
XML record loaded
manifest data order
binding configuration
model
groups
module upgrade
installed database state
```

Do not create a duplicate action before verifying why the existing one is absent.

---

# 120. Template Not Found

Verify:

```text
template XML ID
module installed/upgraded
file loaded in manifest
correct report_name
correct namespace
inheritance target
version-specific naming
```

Separate source correctness from database/module state.

---

# 121. XPath Not Applying

Check:

```text
actual parent template
namespace/XML ID
priority/order
target node exists
other inherited templates
database-stored view/template state
```

Do not keep broadening XPath until it matches something arbitrary.

---

# 122. Report Data Error

If QWeb raises missing/invalid values, trace:

```text
report provider
context
recordset
field availability
permissions
language/company
optional values
batch behavior
```

Do not fix by adding `or ""` everywhere if a required business value is genuinely missing.

---

# 123. Access Error

Determine whether the access error is:

```text
expected denial
incorrect report design
related-record access
company mismatch
portal ownership
attachment access
sudo regression
```

Do not automatically bypass it.

---

# 124. Empty / Wrong Data

Trace the authoritative business source.

Check:

```text
record state
filters
context
company
language
date range
wizard values
provider method
cached attachment
```

Do not assume the template is wrong if the report is intentionally rendering an old cached snapshot.

---

# 125. Stale Attachment

When output is stale, determine whether attachment reuse is intentional.

If not, fix the correct cache/invalidation/generation boundary.

Do not delete all report attachments indiscriminately on production.

---

# 126. Wrong Language

Trace:

```text
report request context
partner/recipient lang
t-lang or equivalent target-version mechanism
mail template context
batch behavior
translation availability
```

Use Localization evidence for missing/incorrect translations.

---

# 127. Wrong Company Branding

Trace company context through:

```text
record
report provider
external layout
company variables
mail/portal context
```

Do not hardcode a company to fix one environment.

---

# 128. Missing Image / Logo

Check:

```text
binary field content
access
source URL
renderer reachability
encoding
format
size
company context
```

Do not assume CSS sizing is the root cause.

---

# 129. PDF Layout Mismatch

Compare:

```text
HTML output
paper format
renderer engine/version
print CSS
fonts
page size
margins
header/footer
resource loading
```

Do not use arbitrary negative margins before understanding the mismatch.

---

# 130. Environment-Only Failure

If report works locally but not staging/production, compare:

```text
Odoo version/build
renderer/version
fonts/packages
base URL
proxy
assets
database configuration
company/report action records
module upgrade state
```

Do not change source merely to accommodate an unidentified environment difference.

---

# VERSION COMPATIBILITY

# 131. Source-Grounded Report APIs

For Odoo version changes, verify material APIs against actual target-version source.

Version-sensitive areas may include:

```text
report action fields
render methods
QWeb helpers
context variables
report model conventions
PDF engine
asset loading
paperformat behavior
controller routes
test helpers
frontend download behavior
```

Do not maintain a giant remembered compatibility table.

Use source evidence.

---

# 132. Upstream Template Changes

During upgrades, compare custom inherited/copied templates against target-version upstream templates.

Check whether:

```text
XPath targets moved
classes changed
variables changed
subtemplates changed
layout changed
business fields changed
```

Do not blindly reapply an old XPath to a new report architecture.

---

# 133. Copied Template Upgrade Debt

If a customization copied a full core report, diff it against the target upstream version.

Prefer converting to narrow inheritance when safe and within scope.

If not, record the remaining upgrade debt.

---

# 134. Renderer Migration

If Odoo/project changes PDF engines, re-evaluate:

```text
CSS support
fonts
page breaks
headers/footers
assets
SVG
JavaScript
remote URLs
performance
```

Do not assume old renderer workarounds remain necessary.

---

# UPGRADE / MIGRATION HANDOFF

# 135. Persistent Report Configuration

Changes to persistent report records may affect installed databases.

Examples:

```text
report action XML ID
paperformat
noupdate report config
attachment rule
mail template binding
company-specific report configuration
```

Use your Upgrade & Migration Analyzer workflow when existing database behavior is material.

---

# 136. `noupdate` Report Records

If report/paperformat/mail records are `noupdate`, distinguish:

```text
fresh install behavior
existing installed database behavior
manual database customization
```

Do not assume editing XML updates existing production records.

---

# 137. Renaming Report XML IDs

Treat as migration-sensitive.

Search all references.

Preserve compatibility or provide deterministic migration when required.

---

# 138. Attachment Policy Changes

Changing from live render to stored attachments, or vice versa, can alter:

```text
storage
historical semantics
portal behavior
mail behavior
freshness
access
```

Analyze existing records and migration implications.

---

# AUTOMATED TEST HANDOFF

# 139. Durable Report Test Coverage

When report behavior creates meaningful regression risk, provide your Automated Test Engineer workflow with:

```text
protected business/document behavior
report action
report template
data provider
actor
company
language
batch/single behavior
attachment behavior
security boundary
renderer/runtime dependency
```

Do not duplicate the full automated-test engineering workflow here.

---

# 140. Prefer Semantic Tests

Good automated assertions may cover:

```text
report action resolves
data provider returns correct business values
authorized actor can generate
unauthorized actor cannot
expected template text/semantic section exists
correct filename
attachment created/reused when contract requires
mail flow attaches intended report
```

Avoid asserting entire PDF bytes.

---

# 141. Report Data Unit Tests

Test report-specific calculations/data preparation at the Python boundary where possible.

Do not require a full PDF render to test arithmetic/grouping logic.

---

# 142. Template Rendering Tests

Where the repository/version supports deterministic rendering, test stable semantic HTML/QWeb output.

Do not make assertions depend on incidental whitespace or complete markup when not contractual.

---

# 143. Multi-Language Tests

When language is important, test:

```text
expected language context
translated semantic text
record-language selection
fallback behavior
```

Use localization fixtures/policy.

Do not duplicate all translation QA inside report tests.

---

# 144. Security Tests

For sensitive reports, include actor-specific allow/deny tests through the correct server path.

Do not run every report test as superuser.

---

# 145. Attachment Tests

When attachment semantics matter, test:

```text
created vs reused
owner record
filename
content type
authorized access
freshness rule where deterministic
```

---

# 146. Batch Tests

When report supports multi-record output, test representative multiple records.

Include different companies/languages only if the business contract permits that combination.

---

# 147. Renderer Test Boundary

A unit/QWeb test is not automatically proof that the PDF engine renders correctly.

Record renderer-dependent scenarios for Runtime Validation.

---

# RUNTIME VALIDATION HANDOFF

# 148. Separate Static Confidence From Document Proof

The following are not equivalent:

```text
XML parses
    != report action works in installed database

QWeb renders HTML
    != PDF layout is correct

PDF generates
    != barcode scans

template contains Arabic
    != Arabic/RTL renders correctly

report works as admin
    != portal/restricted actor can access safely

unit tests pass
    != production renderer/fonts/assets match
```

Use your Regression & Runtime Validator workflow for environment-specific proof.

---

# 149. Report Runtime Requirement

When runtime proof is required, record an exact scenario.

Use:

```text
REPORT RUNTIME REQUIREMENT

Report:
Odoo version/build:
Environment:
Renderer:
Actor:
Record(s):
Company:
Website if relevant:
Language:
Initial state:
Generation path:
Expected filename:
Expected semantic content:
Expected layout/pagination:
Expected attachment behavior:
Expected access behavior:
Logs/errors to check:
```

Do not write only:

```text
test the report
```

---

# 150. PDF Runtime Validation

For material PDF changes verify actual PDF rendering.

Check when relevant:

```text
page count
page size/orientation
header/footer
page breaks
tables
totals
images/logo
fonts
Arabic/RTL
barcodes
signatures
long content
short content
```

Use representative business data.

---

# 151. Multiple Data Shapes

Layout bugs often appear only with certain data.

Use representative cases such as:

```text
short name vs long name
few lines vs many lines
empty optional block
large totals
long legal text
foreign language
Arabic
multi-line address
discount/tax variants
```

Do not validate only a minimal demo record when production data is materially more complex.

---

# 152. Multi-Company Runtime Validation

For shared reports/layouts, validate material company variants.

Check:

```text
branding
bank/tax information
footer
currency
paperformat
access
```

Use only scenarios justified by impact risk.

---

# 153. Multi-Language / RTL Runtime Validation

When multilingual output is required, render the actual document in representative languages.

For Arabic, include RTL-specific visual review.

Do not claim RTL correctness from English PDF proof.

---

# 154. Portal/Public Runtime Validation

When report is portal/public:

```text
authorized user succeeds
unauthorized user denied
token behavior correct
record ownership enforced
filename/download correct
no sensitive data leaked
```

Use safe test/staging data when possible.

---

# 155. Email Attachment Runtime Validation

When report is sent by email, verify:

```text
correct recipient
correct language
correct company
correct report
correct filename
attachment opens
content is current/historical as intended
```

Do not need to send real production email when a safe mail capture/test environment can prove it.

---

# 156. Physical Printer / Scanner Boundary

If success depends on a real printer/scanner:

```text
label size
scaling
barcode scan
pre-printed form alignment
```

record that as external runtime proof.

Do not claim printer-level validation from generated PDF inspection alone.

---

# IMPLEMENTATION PROCEDURE

# 157. Reporting Implementation Sequence

For material report work, use this sequence:

```text
1. Determine task mode and acceptance criteria.
2. Reuse existing investigation/impact/specialist evidence.
3. Detect Odoo version.
4. Detect actual report renderer.
5. Identify business/report action/template/provider/layout ownership.
6. Trace template inheritance and downstream consumers.
7. Inspect existing repository report conventions.
8. Classify document business/legal/security significance.
9. Map data, language, company, attachment, and delivery contracts.
10. Choose the smallest safe extension boundary.
11. Implement report-data changes outside QWeb where appropriate.
12. Implement narrow template/layout/action changes.
13. Apply localization workflow when user-visible translated text is affected.
14. Perform static XML/Python/action consistency checks.
15. Add/update durable automated tests when justified.
16. Perform actual report/PDF/runtime validation when required.
17. Review final diff and do-not-touch boundary.
18. Report confirmed vs runtime-unverified behavior.
```

Do not begin by copying the whole upstream template.

---

# 158. Static Report Checks

Before runtime rendering, inspect when applicable:

```text
XML well-formedness
template IDs
inherit_id
XPath targets
report action references
model names
paperformat references
manifest data inclusion/order
Python imports
report model naming
_get_report_values contract
CSS syntax
asset/static paths
mail-template report references
translation integration
security/controller references
test discovery
```

Static checks catch cheap failures.

They do not prove final rendering.

---

# 159. Validate XML IDs

For every new/changed cross-reference, verify the full XML ID and owner module.

Do not assume the current module prefix.

---

# 160. Manifest/Data Order

Ensure report records/templates/layouts/paperformats load in a valid order according to actual references.

Do not add unnecessary manifest dependencies.

Do not rely on an indirect dependency when the module directly references another addon's XML ID/model.

---

# 161. Python Syntax and Wiring

For report Python changes:

```text
compile syntax
verify __init__.py imports
verify report model registration
verify dependency
verify multi-record behavior
```

A correct class that is never imported does not work.

---

# 162. Template Consistency

Review:

```text
unclosed tags
wrong template names
stale XPath
undefined variables
invalid record field access
duplicated IDs/classes that matter
unreachable branches
```

Do not rely solely on generic XML parsing for Odoo QWeb semantics.

---

# 163. Final Diff Review

Inspect the actual final diff.

Look for:

- unrelated report/template changes;
- broad external-layout edits;
- copied upstream blocks larger than necessary;
- debug text/styles;
- hardcoded company/customer values;
- hardcoded URLs;
- accidental `sudo()`;
- sensitive fields added to documents;
- stale temporary sample data;
- duplicate report actions;
- renamed XML IDs;
- missing manifest dependencies;
- unexpected attachment/caching changes;
- disabled/skipped tests;
- renderer-specific hacks without evidence.

Do not hide a broad document change behind a small visual request.

---

# REPORT REVIEW PROCEDURE

# 164. Review Report Intent

For an existing report, determine:

```text
Who receives it?
What business event creates it?
What values are authoritative?
Is it historical or live?
Is it legal/financial?
Is it archived?
Which languages/companies apply?
How is access enforced?
```

Do not review styling in isolation from the document's business purpose.

---

# 165. Find Incorrect Business Data

A visually perfect report can still be wrong.

Compare printed values to authoritative Odoo model values.

Prioritize correctness over cosmetic alignment.

---

# 166. Find Hidden Data Exposure

Inspect all fields/related values included in output, including hidden/conditional blocks.

A value may still be present in HTML/PDF even if visually obscured.

Do not rely on CSS to protect sensitive data.

---

# 167. Find Upgrade Fragility

Warning patterns:

```text
full copied upstream template
deep positional XPath
hardcoded core DOM structure
renderer-specific hacks
private/internal report APIs
large duplicated layout
```

Record upgrade risk even if current rendering works.

---

# 168. Find Localization Fragility

Warning patterns:

```text
fixed widths
hardcoded English strings
manual date/number formatting
left/right-specific layout assumptions
text embedded in images
```

Apply localization guidance where relevant.

---

# 169. Find Performance Fragility

Warning patterns:

```text
search inside QWeb loop
nested repeated related traversals
huge binary images
rendering thousands of records
persistent attachment generated repeatedly
```

Use Performance Analyzer for measurement before claiming severity where runtime scale matters.

---

# 170. Find Security Fragility

Warning patterns:

```text
sudo().browse(request_id)
public route + raw ID
portal route without ownership
attachment ID exposed
restricted related data
cross-company rendering
```

Apply Security evidence.

---

# OUTPUT CONTRACT

# 171. Reporting Evidence

For material reporting work, produce a structured evidence block internally or in a user-facing report when appropriate.

Use:

```text
REPORTING EVIDENCE

Task mode:
Odoo version:
Version evidence:
Renderer:
Renderer evidence:

Document/report:
Document category:
Target module(s):
Business model:
Feature/report owner:

Report action:
Report action XML ID:
Report type:
Report template:
Template owner:
Inherited templates:
External layout:
Paper format:
Report data provider:

Delivery paths:
Attachment behavior:
Historical vs live behavior:

Actor(s):
Security boundary:
Company context:
Website context:
Language rule:
RTL requirement:

Current report flow:
Requested behavior:
Recommended modification boundary:
Why this boundary is safe:

Business-data risks:
Template/inheritance risks:
Layout/pagination risks:
Renderer/environment risks:
Attachment/cache risks:
Security risks:
Localization/RTL risks:
Performance considerations:
Upgrade/migration considerations:

Automated-test requirements:
Runtime-render requirements:
Do-not-touch boundary:
Remaining unknowns:

Report status:
Confidence:
```

Useful report statuses:

```text
READY FOR IMPLEMENTATION
IMPLEMENTED / RUNTIME RENDER REQUIRED
VALIDATED FOR REVIEWED SCOPE
PARTIAL / NEEDS PDF EVIDENCE
PARTIAL / NEEDS SECURITY EVIDENCE
PARTIAL / NEEDS LOCALIZATION EVIDENCE
PARTIAL / NEEDS UPGRADE EVIDENCE
BLOCKED
```

Do not report `VALIDATED FOR REVIEWED SCOPE` when required renderer/runtime proof was unavailable.

---

# 172. Concise Embedded Output

When this skill runs inside an already-authorized implementation task, keep evidence concise unless the report is complex.

Example:

```text
Report:
Version/renderer:
Owner:
Action/template/layout:
Data provider:
Safe boundary:
Company/language:
Attachment behavior:
Key risks:
Automated tests:
Runtime render:
Do-not-touch:
```

Then continue through your native planning/implementation workflow.

Do not expose internal skill routing or produce a second generic implementation plan.

---

# 173. Full Report Review Output

For standalone report audits, high-risk legal documents, complex rendering bugs, shared layouts, or version migrations, provide relevant sections from:

```text
1. Report Target
2. Odoo Version / Evidence
3. Renderer / Evidence
4. Document Classification
5. Business Ownership
6. Report Action
7. Report Data Provider
8. Template Ownership
9. Template Inheritance Chain
10. External Layout
11. Paper Format
12. Data Contract
13. Company Context
14. Language / RTL Context
15. Business / Legal Semantics
16. Attachment / Historical Snapshot Behavior
17. Delivery Paths
18. Portal / Public Security Boundary
19. QWeb / Template Review
20. CSS / Pagination Review
21. Fonts / Images / Barcode Review
22. Renderer / Environment Review
23. Performance Considerations
24. Upgrade / Migration Considerations
25. Existing Implementations / Conflicts
26. Recommended Safe Modification Boundary
27. Automated Test Requirements
28. Runtime Render Requirements
29. Do-Not-Touch Areas
30. Remaining Unknowns
31. Reporting Evidence
```

Only include sections relevant to the actual report.

Do not invent empty complexity.

---

# 174. Reporting Finding Severity

Classify report findings by impact.

Possible severities:

```text
CRITICAL
HIGH
MEDIUM
LOW
INFO
```

### CRITICAL

Examples:

- public/portal report leaks protected financial/HR/customer data;
- printed legal/financial totals materially disagree with authoritative records;
- cross-company report exposes another company's sensitive data;
- generated document falsely represents a completed legal/signature/payment state with material consequences.

### HIGH

Examples:

- shared layout change breaks many business documents;
- persistent attachment behavior serves stale legally significant documents;
- report security relies on a hidden Print button while direct access remains possible;
- batch report mixes companies/languages and produces materially wrong customer/legal documents;
- renderer/pagination defect omits material content such as totals or legal lines.

### MEDIUM

Examples:

- fragile XPath is likely to break on ordinary upgrade;
- report does repeated expensive work at moderate scale;
- RTL layout is materially unreadable;
- attachment lifecycle creates unnecessary storage growth;
- inconsistent filename/language causes operational problems.

### LOW

Examples:

- local styling inconsistency;
- small maintainability issue;
- minor duplication;
- non-critical spacing issue.

Keep severity separate from confidence.

Do not classify subjective aesthetic preferences as high severity.

---

# 175. Reporting Confidence

Express confidence based on evidence.

Useful levels:

```text
HIGH
MEDIUM
LOW
```

### HIGH

Use HIGH only when material evidence is complete, for example:

- Odoo version confirmed;
- renderer confirmed;
- report action/template/provider/layout chain verified;
- downstream inheritance checked;
- business/security/company/language context understood;
- required automated tests pass;
- required runtime PDF/document proof completed.

### MEDIUM

Use MEDIUM when:

- repository architecture is clear;
- static/data behavior is well supported;
- but renderer/browser/printer/production-specific proof remains unavailable.

### LOW

Use LOW when:

- report owner is unclear;
- renderer/version is uncertain;
- runtime-only issue cannot be reproduced;
- security/company/language behavior is unresolved;
- database-only report configuration materially affects behavior and is unavailable.

Never convert lack of evidence into high confidence.

---

# 176. Recommended Modification Boundary

Recommend the smallest boundary that satisfies the requirement.

Examples:

```text
inherit one report template
extend one report-data helper
add one report-specific paperformat
add one scoped CSS block
extend one shared subtemplate
fix one report action attachment expression
add one portal authorization check
```

Do not redesign all reporting infrastructure for one local problem.

---

# 177. Do-Not-Touch Boundary

Record explicit boundaries when useful.

Examples:

```text
Do not rename the report XML ID.
Do not replace the shared external layout.
Do not change invoice totals/business calculations.
Do not broaden sudo().
Do not alter report attachment caching.
Do not change the global paperformat.
Do not rewrite unrelated reports.
Do not replace the renderer.
Do not create a second report action.
Do not hardcode company branding.
Do not regenerate all historical attachments.
Do not change legal wording.
```

This boundary protects scope and compatibility.

---

# 178. Stop Conditions

Stop and report rather than guessing when:

- the Odoo version cannot be determined and the required report API is version-sensitive;
- the actual report action/template/provider owner cannot be identified;
- the renderer cannot be established and the fix depends on renderer capabilities;
- a legal/financial value is ambiguous and changing it could misrepresent business data;
- public/portal access rules are ambiguous and proceeding could expose protected data;
- company ownership is ambiguous;
- attachment snapshot/freshness semantics are unknown for a legal/historical document;
- a report XML-ID change requires unresolved migration behavior;
- a required translation/Arabic policy decision is unresolved;
- a browser/PDF/printer-only defect cannot be verified and repository evidence is insufficient to claim a fix;
- the requester asked only for review and implementation would be required;
- validation would require unsafe production actions.

A stop condition should identify:

```text
what is blocked
why it matters
what evidence is confirmed
what exact source/runtime/business decision is required next
```

Do not replace missing evidence with remembered Odoo behavior.

---

# 179. Final Principles

Your objective is not:

```text
maximum QWeb
maximum CSS
maximum template inheritance
maximum PDF customization
maximum visual similarity at the cost of business correctness
```

Your objective is:

```text
a correct Odoo document
built at the narrowest stable report boundary
with authoritative business data,
intentional security/company/language behavior,
renderer-aware layout,
and explicit runtime proof requirements
```

Prefer:

```text
authoritative Odoo business values
    >
recalculating values in QWeb

verified report ownership
    >
editing the first matching template

narrow inherited template
    >
full upstream report copy

report-data preparation
    >
complex ORM work inside QWeb

server authorization
    >
hidden Print button

record/company context
    >
implicit current-company assumptions

recipient/document language
    >
admin/user language by accident

renderer evidence
    >
generic browser assumptions

semantic automated tests
    >
binary PDF equality

actual PDF runtime proof
    >
HTML-only confidence when PDF is the deliverable

historical snapshot semantics
    >
stale or silently regenerated legal documents

repository conventions
    >
generic reporting preferences
```

The final result should make clear:

```text
what owns the document
how the report action/data/template/layout/rendering chain works
which business and security contracts must be preserved
where the safest modification boundary is
what automated tests protect the behavior
what real rendering/runtime proof is still required
and how confident you are in the result
```
