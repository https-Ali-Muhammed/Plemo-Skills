# Odoo Frontend & OWL Specialist

## Purpose

Use this guidance to investigate, design, implement, review, debug, and safely extend Odoo frontend behavior across the web client, website, portal, and other Odoo frontend surfaces when relevant.

The goal is not to apply generic JavaScript advice to an Odoo repository.

The goal is to understand the actual Odoo frontend architecture in the detected version and repository, identify the component/template/service/registry/patch/asset/RPC ownership chain, and make the smallest safe frontend change without breaking framework contracts or downstream consumers.

This guidance adds **Odoo-specific frontend and OWL procedures, safeguards, and evidence**.

It does not replace Plemo's native repository discovery, planning, implementation, debugging, final review, or the existing impact, security, performance, code-quality, automated-test, and regression/runtime-validation workflows.

Its core question is:

```text
How does this frontend behavior actually work in this Odoo version,
where is the safest extension boundary,
and how can it be changed without creating fragile browser-side behavior?
```

Core principles:

- Detect the actual Odoo version and frontend architecture before choosing APIs.
- Follow `plemo.md`, repository conventions, and nearby working Odoo frontend code before generic examples.
- Trace frontend ownership across JavaScript, OWL components, templates, services, registries, patches, assets, RPC/controllers, models, and CSS when relevant.
- Reuse reliable codebase-investigation and feature-impact evidence instead of repeating full discovery.
- Prefer stable framework extension points over DOM hacks or large upstream copies.
- Prefer narrow component/service/registry/template extensions over global patches when the feature scope is local.
- Treat patches as shared runtime modifications whose blast radius must be understood.
- Treat asset bundle placement and load order as part of behavior, not packaging trivia.
- Keep frontend state ownership explicit.
- Keep async behavior race-safe and lifecycle-safe.
- Clean up listeners, subscriptions, timers, observers, and other resources when lifecycle requires it.
- Keep RPC/model/controller contracts intentional and version-verified.
- Never treat UI visibility as server-side security.
- Never use frontend checks as a substitute for ACLs, record rules, route authorization, or server-side validation.
- Separate static frontend correctness from actual browser/runtime proof.
- Separate browser bugs from backend/RPC failures instead of guessing from symptoms.
- Do not rewrite working frontend architecture solely to match a preferred JavaScript style.
- Do not copy large upstream components/templates when a smaller supported extension point exists.
- Do not invent import paths, registry keys, service names, lifecycle APIs, or test helpers from memory when repository/version evidence is available.
- Do not announce or expose internal capability selection during normal conversation.

---

# 0. Native Plemo Compatibility

This guidance extends Plemo's existing agent capabilities. It does not replace them.

Plemo already performs repository discovery, task-mode understanding, planning, implementation, ordinary debugging, targeted checks, and final diff review.

The existing skills already provide specialized evidence for codebase investigation, impact analysis, security, migration, performance, code quality, automated tests, localization, and post-change runtime validation.

This guidance must not create a second generic planning system or a second impact/regression workflow.

Repository-specific instructions such as:

```text
plemo.md
configured addon paths
customer restrictions
repository frontend conventions
asset conventions
build/deployment conventions
available Plemo tools
```

take precedence over generic examples here.

When reliable evidence already exists from the current task, reuse it.

Do not force a full frontend architecture audit for a trivial isolated change.

---

## 0.1 Relationship With Native Workflow and Existing Evidence

The intended separation is:

```text
existing codebase-investigation evidence
    "What owns this feature and how is it connected?"
        ↓
existing feature-impact evidence
    "What frontend/backend consumers could be affected?"
        ↓
Odoo Frontend & OWL Specialist
    "How should this frontend behavior be safely implemented or debugged
     in the actual Odoo architecture?"
        ↓
Plemo native planning / implementation
        ↓
Odoo Automated Test Engineer when durable frontend coverage is justified
        ↓
Odoo Regression & Runtime Validator
    "Does it actually work in the browser/runtime environment?"
```

Useful existing evidence includes:

```text
Feature owner
Relevant addon(s)
JavaScript files
OWL components
QWeb/OWL templates
Patches
Registries
Services
Asset bundles
Controller/RPC routes
Model methods
Reverse dependencies
Downstream template inheritance
Runtime consumers
Security surface
Performance-sensitive paths
Localization requirements
Regression scenarios
Do-not-touch boundaries
Remaining unknowns
```

Do not recreate those analyses unless the frontend implementation reveals a materially different surface or the evidence is incomplete/stale.

---

## 0.2 Task Mode and Authorization

Determine the current task mode.

Typical modes:

```text
Frontend investigation / review only
Diagnose frontend bug
Implement frontend feature
Fix frontend bug
Refactor frontend code
Odoo-version frontend migration
Frontend performance diagnosis
Frontend + backend integration change
```

For **investigation / review / diagnosis only**:

- remain read-only unless the user explicitly requested fixes;
- inspect repository and available runtime evidence;
- identify likely ownership, contracts, risks, and safest modification boundary;
- distinguish confirmed repository evidence from browser/runtime assumptions;
- do not silently modify source files.

For **implement / fix / refactor / migrate**:

- the original user request authorizes implementation within that scope;
- do not ask for duplicate approval solely because frontend analysis completed;
- continue through Plemo's normal planning and implementation workflow;
- preserve unrelated frontend behavior;
- do not broaden a local change into a global patch/refactor without evidence.

Ask for a new decision only when the frontend work reveals:

- a material behavior ambiguity;
- a security-sensitive product decision;
- destructive or irreversible data behavior;
- external live-system side effects;
- a major scope expansion;
- a browser compatibility requirement not inferable from project evidence;
- a migration/API compatibility decision with multiple materially different outcomes;
- another unresolved choice where guessing would be unsafe.

---

## 0.3 Frontend Materiality Gate

Do not force heavyweight frontend analysis on every change.

### Light frontend review is normally sufficient for:

- an isolated CSS adjustment with clear ownership;
- a small text/label adjustment where localization rules are already known;
- one obvious asset entry correction;
- a tiny template attribute change with no JS coupling;
- formatting/comment-only JavaScript changes.

### Targeted frontend analysis is appropriate for:

- one component change;
- one service call;
- one registry entry;
- one patch;
- one template inheritance;
- one frontend RPC interaction;
- one website/portal interaction;
- one dialog/action flow;
- one local asset-bundle change.

### Full material frontend analysis is strongly preferred for:

- global/shared patches;
- core web-client behavior;
- shared services;
- registry replacements;
- multiple frontend addons affecting the same target;
- deep template inheritance;
- JS-to-RPC contract changes;
- public website or portal workflows;
- payment/checkout flows;
- POS or offline-sensitive flows;
- multi-company or multi-website frontend behavior;
- frontend changes touching authentication/session context;
- broad asset bundle modifications;
- version upgrades that alter OWL/frontend APIs;
- browser-only regressions with unclear ownership;
- race conditions or lifecycle leaks;
- changes with uncertain browser/runtime blast radius.

Analysis depth should follow runtime and maintenance risk, not file count.

---

# 1. Determine the Frontend Target

Identify exactly what frontend behavior is being investigated or changed.

Record when useful:

```text
Project:
Requested behavior:
Frontend surface:
Target addon(s):
Feature owner:
Relevant action/page/route:
Relevant component(s):
Relevant template(s):
Relevant service(s):
Relevant registry/patch:
Relevant asset bundle(s):
Relevant RPC/controller/model:
Known browser symptom:
Acceptance criteria:
```

If the user already provided the target, do not ask them to repeat it.

Use repository discovery to resolve missing ownership before asking unnecessary questions.

---

# 2. Detect Odoo Version and Frontend Architecture

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
existing frontend import paths
nearby core/custom frontend code
```

A valid module manifest version prefix is acceptable primary evidence when Odoo core source is unavailable.

Record:

```text
Odoo version:
Version evidence:
Edition if known:
Frontend architecture evidence:
Relevant framework generation:
```

Do not assume that an API, import path, lifecycle hook, registry behavior, patch signature, RPC helper, browser-test framework, or asset convention is identical across Odoo versions.

Verify version-sensitive frontend APIs against actual source or working repository examples.

---

# 3. Repository Convention Precedence

Before recommending frontend structure, inspect how this repository already solves similar problems.

Prefer, in order:

1. `plemo.md` and repository/customer rules;
2. existing code in the same addon;
3. nearby project-specific frontend implementations;
4. verified Odoo core patterns from the detected version;
5. generic JavaScript/OWL guidance last.

Do not call an implementation wrong merely because it differs from another Odoo version or a generic OWL example.

Do not introduce TypeScript, a new bundler, a new state library, a new test library, or another frontend architecture unless the repository already uses it or the authorized task explicitly requires it.

---

# 4. Reuse Existing Evidence

Reuse reliable evidence already collected during the current task.

Possible inputs:

```text
CODEBASE INVESTIGATION evidence
IMPACT EVIDENCE
SECURITY EVIDENCE
PERFORMANCE EVIDENCE
MIGRATION EVIDENCE
TEST ENGINEERING EVIDENCE
runtime-validation findings
localization requirements
code-quality findings
actual Git diff
browser console/network evidence
```

Do not repeat a full inheritance/dependency investigation when existing evidence already identifies the owner and downstream consumers.

Refresh only the parts that become stale because the frontend implementation changes the boundary.

---

# 5. Inspect the Actual Frontend Change Set

For implemented or partially implemented work, inspect the real diff.

Record when relevant:

```text
Changed JS files:
Changed XML/QWeb/OWL templates:
Changed SCSS/CSS:
Changed manifest/assets:
Changed controllers/routes:
Changed Python model/RPC methods:
Changed localization files/helpers:
Added/removed registry entries:
Added/removed patches:
Added/removed services:
Unrelated/pre-existing frontend changes:
```

Do not reason only from the intended plan.

A small Python change can materially alter a frontend contract, and a small asset change can alter runtime load behavior broadly.

---

# 6. Classify the Frontend Surface

Determine which Odoo frontend surface owns the behavior.

Common surfaces include:

```text
backend web client
website
portal
point of sale
public website flow
customer self-service flow
report preview/editor surface
specialized application frontend
shared web/core frontend
```

Do not assume patterns from the backend web client apply unchanged to website, portal, POS, or another specialized application.

Record the surface because service availability, templates, assets, session state, routing, test mechanisms, and extension APIs can differ.

---

# 7. Establish Feature Ownership

Trace the behavior to the original and effective owner.

Identify when relevant:

```text
base component/class
owning addon
owning template
registry category/key
service owner
patch owner(s)
asset bundle owner
controller/route owner
model method owner
CSS selector owner
```

Search both upstream and downstream customizations.

Do not modify the first custom file that mentions the feature if another addon owns the actual extension boundary.

When a core feature is extended by multiple custom addons, build the effective chain before changing behavior.

---

# 8. Determine the Safe Extension Boundary

Prefer the smallest stable supported boundary that can implement the requirement.

Candidate boundaries may include:

```text
component extension
supported method override
component/template inheritance
registry addition/replacement when appropriate
service extension
narrow patch
new local component composed into existing flow
controller/model endpoint extension
asset bundle addition
scoped CSS/SCSS
```

Avoid by default:

```text
copying a full upstream component
copying a full upstream template
replacing a broad registry entry for a local behavior
patching a global prototype for one screen
manual DOM mutation when component state can own the behavior
backend changes solely to work around misunderstood frontend state
```

If a broad patch/replacement is genuinely the safest available option, record why narrower extension points are insufficient.

---

# 9. Build the Frontend Contract Map

For material frontend changes, map the end-to-end contract.

Example:

```text
user interaction
    ↓
component event handler
    ↓
component/service state
    ↓
ORM/RPC/service call
    ↓
controller or model method
    ↓
server validation/security/business logic
    ↓
response contract
    ↓
frontend state update
    ↓
template render / action / notification
```

Include only the layers that actually exist.

Record data shape, context, actor, company, website, and error behavior where relevant.

Do not diagnose a visible UI symptom without checking whether failure originates deeper in the chain.

---

# 10. Search for Existing Implementations and Duplicates

Before adding a new component/service/helper/patch, search for existing equivalents.

Search by:

```text
component name
registry key
service name
patch target
method name
route
model method
CSS class/data attribute
QWeb template name
user-visible behavior
RPC payload/response fields
```

Prefer extending existing owned behavior over creating parallel implementations.

Do not create a second frontend service/helper when the repository already has a stable one serving the same contract.

---

# OWL COMPONENT ARCHITECTURE

# 11. Component Ownership

A component should own a coherent UI responsibility.

Determine:

```text
what data it owns
what state it owns
what it receives as props/input
what services it depends on
what events/actions it emits
what template renders it
what child components it owns
```

Do not move unrelated business responsibilities into one component merely because it is already on the screen.

Do not split a simple local component into excessive abstraction without a real reuse or clarity benefit.

---

# 12. Prefer Framework Extension Points

When modifying existing OWL behavior, identify supported extension points before patching internals.

Possible extension points vary by version and feature and may include:

```text
component composition
subclass/registration pattern
registry contribution
service contribution
supported hooks
method patch
QWeb/OWL template inheritance
field/widget registration
view/controller/renderer extension
```

Verify which pattern the target Odoo version actually uses.

Do not assume a pattern from older legacy widgets or a newer OWL generation applies to the target version.

---

# 13. Component Lifecycle

Understand when frontend work executes relative to component lifecycle.

Inspect version-supported lifecycle mechanisms for:

```text
initial setup
preload/async preparation
mount
prop updates
render updates
unmount/cleanup
error handling
```

Do not perform async or DOM-dependent work in a lifecycle phase where the required state/DOM/service is not yet valid.

Do not leave subscriptions/listeners alive after component destruction.

Verify actual hook names and semantics from the detected version rather than relying on memory.

---

# 14. `setup()` and Initialization Boundaries

Where the detected OWL/Odoo architecture expects initialization through `setup()` or another framework-supported mechanism, follow that pattern.

Keep initialization explicit:

```text
services
state
refs
subscriptions
callbacks
initial async load
```

Do not override constructors or other low-level lifecycle behavior when the framework/version intentionally provides a safer extension point.

Do not generalize this rule across versions without checking the actual source.

---

# 15. Reactive State Ownership

Identify the single source of truth for each UI state value.

Examples:

```text
selected record
loading state
filter value
expanded/collapsed state
server result
validation error
modal state
pagination cursor
```

Avoid storing the same logical value in multiple independent state locations unless synchronization is intentional and necessary.

Prefer deriving cheap values from authoritative state rather than duplicating them.

Do not mutate reactive structures through unsupported patterns.

Verify the framework/version-supported state mechanism in nearby code.

---

# 16. Props and Input Contracts

Treat component props/input as contracts.

Inspect:

```text
required vs optional values
default behavior
value types/shapes
callbacks
record/model assumptions
company/website/language assumptions
mutability expectations
```

Do not mutate parent-owned data implicitly if the framework/component contract expects one-way ownership.

When a prop contract changes, trace all callers/components/templates that instantiate the component.

---

# 17. Derived State

Avoid duplicating state that can be reliably derived from props, services, or existing state.

Use cached/derived state only when there is a demonstrated correctness or performance reason.

If derived state can become stale because upstream values change, ensure the update mechanism is explicit.

Do not introduce manual synchronization merely to avoid a small expression.

---

# 18. Refs and DOM Access

Use framework-supported refs or component mechanisms when DOM access is truly required.

Avoid broad document queries such as global selectors for local component behavior when a scoped ref can identify the owned node.

Manual DOM access is especially risky when:

```text
nodes are conditionally rendered
components rerender
multiple instances exist
website snippets duplicate markup
selectors are shared by unrelated modules
```

Do not make the DOM a second hidden state store.

---

# 19. Event Handling

Trace where events originate and who owns the resulting state/business action.

Check:

```text
handler binding
payload shape
propagation
preventDefault/stopPropagation behavior
keyboard behavior
multiple component instances
double-click/repeated submission risk
```

Avoid attaching duplicate global listeners during rerender/setup.

Clean up manually registered listeners when required by lifecycle.

---

# 20. Async Operations and Stale State

Frontend Odoo code frequently performs async ORM/RPC/service calls.

For material async behavior, reason about:

```text
component destroyed before response
props/state changed before response
second request finishes before first
multiple clicks start duplicate requests
navigation occurs during request
company/website/session context changes
server returns partial/error response
```

Do not let an old response overwrite newer authoritative state without justification.

Use version-supported cancellation/guard patterns where available.

Do not invent a cancellation API not present in the target version.

---

# 21. Loading and Re-Entrancy State

When an action should not run concurrently, make the frontend and server contract clear.

Possible protections include:

```text
disabled/re-entrancy UI state
idempotent backend behavior
request identity
duplicate suppression
server-side state validation
```

Do not rely solely on disabling a button for correctness when duplicate RPC requests could still arrive.

Frontend prevention improves UX; backend enforcement protects the business invariant.

---

# 22. Cleanup and Resource Lifecycle

Inventory manually owned resources such as:

```text
event listeners
bus subscriptions
timers/intervals
observers
external library instances
pending callbacks
DOM integrations
```

Confirm cleanup at the correct lifecycle boundary.

A browser page that appears correct once can still leak handlers and execute duplicate actions after repeated navigation/mounts.

---

# 23. Frontend Error Handling

Classify errors by layer:

```text
input validation
business validation from server
access/security error
network/RPC transport error
unexpected server traceback
frontend programming error
asset/import error
external integration error surfaced through server
```

Present user-facing errors through project/version-supported mechanisms.

Do not swallow errors silently.

Do not expose sensitive server traceback/details to public users.

Do not convert access errors into generic success states.

---

# SERVICES, REGISTRIES, AND PATCHES

# 24. Service Dependency Analysis

When a component uses Odoo services, identify:

```text
service name
service owner
availability on this frontend surface
dependency/lifecycle assumptions
methods used
returned contract
```

Verify service names and APIs against the detected version/repository.

Do not assume a service available in backend web client is available unchanged on website, portal, POS, or another surface.

---

# 25. Service Ownership and Scope

Use a service when behavior is genuinely shared/lifecycle-wide and fits existing architecture.

Do not create a global service for one component's local state.

Avoid putting unrelated business calculations into frontend services when the server should own authoritative business logic.

When extending a service, trace all consumers before changing method signatures or returned data.

---

# 26. Registry Analysis

For registry-based behavior, identify:

```text
registry category
entry key
owning addon
existing registration
other custom registrations
selection/priority/sequence behavior if applicable
consumers of the key
```

Verify actual registry APIs and replacement/force/sequence semantics from target-version source.

Do not create key collisions accidentally.

Do not replace a global registry entry to modify one local use case when a narrower extension point exists.

---

# 27. Patch Analysis

Treat every patch as a potentially shared runtime modification.

Record:

```text
Patch target:
Owning addon:
Why patch is required:
Methods/properties changed:
Original behavior:
Other patches found:
Load order evidence:
Scope of affected consumers:
Version sensitivity:
```

Before implementing a patch, search for other patches against the same target/method.

Do not assume patches compose safely merely because files load without syntax errors.

---

# 28. Patch Contract Preservation

When a patch modifies an existing method/component/service:

- preserve expected arguments unless intentionally changing the contract;
- preserve return behavior unless change is authorized;
- preserve async/sync expectations;
- preserve `this`/instance semantics;
- preserve parent/original behavior where required;
- avoid swallowing upstream side effects;
- keep the patch narrow.

Verify the target version's supported way to invoke original/super behavior.

Do not use syntax copied from another Odoo version without evidence.

---

# 29. Patch Collision and Ordering

For multiple patches/extensions, analyze effective behavior rather than each file independently.

Check:

```text
asset ordering
module dependency ordering
patch registration order
method wrapping chain
same property/method modified multiple times
conditional loading
website/backend bundle separation
```

If correct behavior depends on fragile incidental load order, treat that as a maintainability risk.

Prefer explicit ownership/dependency where possible.

---

# 30. Legacy Widget / OWL Interoperability

Some repositories contain legacy widgets, older frontend APIs, OWL components, and migration layers together.

Do not rewrite legacy code solely because OWL exists elsewhere.

Determine:

```text
which architecture owns the current target
whether a bridge/adapter already exists
whether the target Odoo version still supports the legacy path
whether migration is part of the authorized scope
```

For contained fixes, prefer the safest extension within the existing supported architecture unless migration provides a clear required benefit.

---

# TEMPLATES AND QWEB/OWL CONTRACTS

# 31. Template Ownership

Identify the exact template that renders the behavior.

Trace:

```text
t-name / template identifier
owning addon
component using it
parent/inherited template
child/downstream inheritors
XPath modifications
conditional branches
slots/subtemplates where relevant
```

Do not modify a similarly named template without confirming runtime ownership.

---

# 32. Template Inheritance

Prefer narrow template inheritance over full template duplication.

Inspect:

```text
inherit target
XPath/selector
position
attributes/classes/data hooks
component assumptions
child inheritors
```

Avoid broad `replace` operations when an insertion or attribute update can safely implement the requirement.

If replacement is necessary, analyze downstream inheritors and JS selectors that may depend on the original node.

---

# 33. Template-to-JavaScript Contract

Templates often expose implicit contracts to JavaScript through:

```text
refs
CSS classes
data-* attributes
component props
event bindings
slot structure
field/widget identifiers
DOM hierarchy
```

Before changing/removing markup, search JavaScript and CSS consumers.

A visual-only-looking template change can break event handling or patch selectors.

---

# 34. Conditional Rendering

For conditional UI branches, check both visible and hidden paths.

Confirm the source of the condition:

```text
props
state
record data
service state
user group information
company/website context
server response
```

Do not use client-side conditions as authorization.

If an action must be forbidden, enforce it server-side as well.

---

# 35. List Rendering and Stable Identity

When rendering repeated items, use the framework/version-supported stable identity/key mechanism where required.

Avoid using unstable array position as identity when rows can reorder, insert, or delete and component state depends on identity.

Verify exact template syntax against the target version.

---

# 36. Accessibility and Interaction Semantics

For interactive UI changes, preserve meaningful browser interaction semantics when practical.

Consider:

```text
button vs non-interactive element
keyboard interaction
focus behavior
labels/accessible names
dialog focus/closure
loading/disabled state
```

Follow existing Odoo/repository patterns.

Do not create a custom accessibility system disconnected from the framework for one local change.

---

# ASSETS AND MODULE LOADING

# 37. Asset Bundle Ownership

Identify the bundle that should load the frontend code.

Possible bundle names and structures vary by Odoo version and surface.

Determine from actual source:

```text
bundle owner
bundle consumers
backend vs frontend/website scope
lazy/test bundle behavior
module dependency/load order
existing nearby entries
```

Do not guess bundle names from another Odoo version.

---

# 38. Narrowest Appropriate Bundle

Place frontend assets in the narrowest bundle that reliably reaches the owned surface.

Avoid loading customer-specific or page-specific behavior globally when the repository provides a narrower supported bundle.

Do not move existing assets across bundles solely for organization without analyzing consumers.

---

# 39. Asset Ordering and Dependencies

When runtime correctness depends on another JS/template/style module, verify how dependency/order is established.

Possible evidence includes:

```text
manifest dependencies
asset declarations
ES module imports
bundle inclusion
registry/service initialization
existing ordering directives supported by the version
```

Do not rely on alphabetical file order or incidental build output unless the framework explicitly guarantees it.

---

# 40. Duplicate Asset Inclusion

Search for the same asset/helper being included through multiple bundles or declarations when duplicate execution would matter.

Duplicate loading can cause:

```text
double registry registration
double event binding
duplicate patches
CSS conflicts
multiple service initialization
hard-to-reproduce debug/production differences
```

Do not remove duplicate-looking declarations without confirming whether bundles are mutually exclusive surfaces.

---

# 41. Debug vs Production Assets

A feature that works in debug-assets mode may still fail in built/minified/production asset mode, and vice versa.

When asset behavior is material, distinguish:

```text
source/import correctness
bundle inclusion
build output
cache invalidation
browser cache
attachment-generated assets if applicable
production compilation/minification behavior
```

Do not treat a source file existing on disk as proof that the browser loads it.

---

# 42. Import Path Verification

For every new or changed Odoo frontend import, verify the import path against the target version/repository.

Prefer existing working imports from the same version.

Do not invent paths such as service, hook, localization, patch, registry, or utility modules from memory.

If core source is available, confirm the exported symbol and its actual contract.

---

# RPC, ORM, AND SERVER CONTRACTS

# 43. Choose the Correct Frontend-to-Server Boundary

Determine how nearby code in the detected version communicates with the server.

Possible boundaries may include:

```text
ORM/model service
RPC service
HTTP/JSON route
specialized service
existing business service wrapper
```

Prefer the established Odoo/project abstraction for the surface.

Do not add a custom controller route when an existing model/service contract safely owns the behavior.

Do not force ORM-style calls where a route contract is intentionally required.

---

# 44. RPC Request Contract

Record material caller-controlled inputs:

```text
model
method/route
record IDs
domain
fields
context
company
website
pagination
filters
business values
```

Treat all browser-controlled values as untrusted at the server boundary.

Frontend validation is UX, not authorization or business-integrity enforcement.

---

# 45. RPC Response Contract

Treat returned field names, types, nullability, ordering, paging markers, and error behavior as frontend contracts when consumers depend on them.

Before changing a response shape, search all consumers.

Prefer returning only data the frontend needs.

Do not return raw unrestricted record data for convenience.

When response structure changes materially, provide impact evidence and automated/runtime test requirements.

---

# 46. Avoid Frontend RPC Chatter

Inspect whether rendering or interaction triggers repeated server calls.

Warning patterns include:

```text
RPC inside row/item loops
same lookup repeated per render
request on every keystroke without intended debounce/commit behavior
multiple components loading the same authoritative data independently
request triggered on every rerender
```

Do not prematurely optimize without evidence.

When scale/performance is material, hand measurement requirements to the Performance Analyzer.

---

# 47. Async Request Ordering

When multiple requests can overlap, define which response is authoritative.

Examples:

```text
search autocomplete
filters
pagination
record switching
tab navigation
website configuration changes
```

Protect against stale-response overwrite where the risk is real.

Do not add complex request orchestration for a path that cannot overlap in practice.

---

# 48. Server Validation Remains Authoritative

Any material business rule must remain enforceable outside the browser.

The same server entry point may be reached through:

```text
RPC
import
cron
server action
integration
another frontend
batch process
```

Do not place business correctness only in JavaScript.

Use frontend checks to improve feedback, not to replace authoritative server rules.

---

# SECURITY-AWARE FRONTEND DESIGN

# 49. UI Visibility Is Not Security

Hiding a button, menu, field, tab, component, or link does not prevent direct server calls.

For security-sensitive actions verify the server boundary through the relevant Security & Access evidence.

Frontend logic may hide/disable unavailable actions for usability, but server-side authorization must still enforce the rule.

---

# 50. Sensitive Data Exposure

Review what data is sent to the browser.

Risks include:

```text
hidden fields still included in payload
portal/public response exposes internal values
frontend service caches restricted data
multi-company data returned then filtered client-side
server error reveals sensitive details
```

Do not fetch broad privileged data and rely on JavaScript to hide it.

Filter/authorize data server-side.

---

# 51. Public / Portal Frontend Paths

For public or portal UI, trace the exact server authorization for every record identifier or token used by the browser.

Treat URL/query/body record IDs as untrusted.

Do not use a privileged server route merely because the frontend hides IDs.

When tokens are involved, defer deep token/security analysis to the Security & Access Reviewer and reuse its evidence.

---

# 52. Company and Website Context

Frontend behavior may depend on active company, allowed companies, current website, language, or session state.

Record where context originates and how it reaches the server.

Do not assume browser-selected company/website context automatically enforces data isolation.

Server-side rules and company/website filtering remain authoritative.

---

# WEBSITE AND PORTAL

# 53. Website Ownership

For website behavior, determine whether the target is:

```text
shared website asset
website-specific template
snippet/editor behavior
public controller flow
portal page
checkout/eCommerce flow
customer form
multi-website configuration
```

Do not apply backend web-client assumptions to website pages without evidence.

---

# 54. Multi-Website Resolution

When behavior can differ by website, inspect:

```text
website-specific records
fallback/global records
current website resolution
route domain/path behavior
template inheritance
asset loading
company relationship
language relationship
```

Repository evidence may not prove database-specific website configuration.

Separate repository logic from runtime website state.

---

# 55. Website Editor / Snippet Safety

When editing snippets or website-editor behavior, verify the actual version architecture before changing selectors, options, templates, or editor registries.

Avoid selectors that match all websites/components when behavior belongs to one snippet.

Do not break edit-mode behavior while fixing public render mode, or vice versa.

When editor-specific runtime behavior is material, require browser validation in edit mode.

---

# 56. Portal Navigation and Record Context

Portal flows often combine route authorization, breadcrumbs/navigation, record context, pager state, and templates.

When modifying one layer, trace the others if they consume the same contract.

Do not assume a template-only fix is safe when route-provided values or access tokens drive rendering.

---

# SPECIALIZED FRONTEND SURFACES

# 57. POS and Offline-Sensitive Frontends

Treat POS or another offline/cache-heavy frontend as a specialized architecture.

Before changes, identify target-version mechanisms for:

```text
data loading
local state/cache
offline behavior
sync
services/stores
screen/component registration
server synchronization
```

Do not apply ordinary backend-webclient RPC/state assumptions automatically.

If offline/reconnect behavior is material, require dedicated automated/runtime scenarios.

---

# 58. Specialized App Registries and Controllers

Some Odoo apps define their own registries, stores, services, controllers, or extension APIs.

Prefer those owned extension points over generic global patching.

Search same-app core code and custom extensions before choosing architecture.

---

# LOCALIZATION AND RTL BOUNDARY

# 59. User-Visible Frontend Text

When the change introduces or modifies user-visible text, apply the project's localization guidance.

Do not invent a translation approach inside this skill.

Reuse the existing Localization & Arabic QA evidence and Plemo-specific JavaScript localization policy when that policy applies to the project/task.

This skill owns frontend architecture; localization guidance owns translation mechanics and Arabic wording/RTL localization safeguards.

---

# 60. RTL-Sensitive Layout

When frontend structure changes could affect RTL behavior, identify the risk and hand localization-specific verification to the relevant localization guidance.

Avoid direction-specific DOM/CSS assumptions where the project supports Arabic/RTL.

Do not redesign unrelated styling solely because RTL is present.

---

# CSS / SCSS

# 61. CSS Ownership and Scope

Place styles with the feature/module that owns the frontend behavior.

Prefer scoped selectors tied to stable component/template structure.

Avoid broad selectors such as global tag overrides when the requirement is local.

Do not depend on unstable DOM depth when a stable class/component boundary exists.

---

# 62. `!important` and Specificity

Treat repeated/excessive `!important` as a signal to investigate ownership/specificity before adding more.

Do not remove necessary existing specificity without tracing downstream effects.

Prefer the smallest selector change that works across intended surfaces.

---

# 63. JavaScript-Dependent CSS Hooks

If JavaScript relies on CSS classes or data attributes as selectors/state hooks, treat them as cross-file contracts.

Before renaming/removing them, search all JS/template/CSS consumers.

Prefer separating semantic JS hooks from purely visual class names when repository conventions already support that distinction.

---

# PERFORMANCE-AWARE FRONTEND DESIGN

# 64. Render and State Update Cost

For components handling large data sets or frequent state updates, inspect:

```text
render frequency
list size
derived calculations
repeated mapping/filtering
child component fan-out
DOM size
RPC frequency
```

Do not claim a performance problem from code appearance alone.

When performance is material, provide suspected hot path and hand measurement to the Performance Analyzer.

---

# 65. Avoid Expensive Work in Render Paths

Keep expensive business calculations and repeated server access out of template/render loops when practical.

Do not move authoritative business calculations client-side solely for speed.

Use server aggregation/batching when it preserves correctness and evidence supports the need.

---

# 66. Memory and Listener Leaks

For long-lived web-client sessions, leaks can accumulate across navigation.

Inspect repeated mount/unmount paths for:

```text
listeners
subscriptions
timers
large retained state
external library objects
closures retaining record data
```

If leak suspicion is material, distinguish code evidence from measured browser-memory evidence.

Do not claim a leak is proven without runtime measurement.

---

# FRONTEND DEBUGGING

# 67. Diagnose by Layer

When debugging a frontend symptom, classify the failing layer before changing code.

Recommended order:

```text
1. Is the correct asset/template loaded?
2. Does the target component/service/patch instantiate/register?
3. Does the interaction/event fire?
4. Is local state/props/context correct?
5. Is the RPC/request sent?
6. Does the server accept/authorize/process it?
7. Is the response correct?
8. Does frontend code process the response correctly?
9. Does rendering reflect the new state?
10. Is CSS/layout hiding the correct state?
```

Do not fix layer 9 when layer 6 is actually failing.

---

# 68. Browser Console Evidence

When browser access exists, inspect relevant console errors/warnings.

Classify them as:

```text
module/import failure
asset/template failure
OWL lifecycle/render error
undefined service/registry dependency
RPC rejection
server traceback surfaced to client
unrelated/pre-existing warning
```

Do not treat every console warning as the root cause.

Correlate with the failing interaction and network/server evidence.

---

# 69. Network/RPC Evidence

For server-connected frontend bugs, inspect the actual request and response when browser tooling is available.

Record when useful:

```text
endpoint/model/method
request payload
context
status/error
response shape
timing/repetition
```

Redact secrets/tokens in user-facing reports.

Do not expose credentials merely to provide debugging evidence.

---

# 70. Asset Loading Diagnosis

For "code not running" bugs, verify:

```text
file is declared/imported
bundle is correct
module dependency is present
asset build succeeds
browser receives updated asset
cache is not serving stale code
module import executes
registry/patch registration occurs
```

Do not repeatedly edit component logic if the file is not loaded.

---

# 71. Template Loading Diagnosis

For template-not-found or stale-render issues, verify:

```text
template file is in correct asset/data mechanism for the version
identifier matches component expectation
inheritance target exists
XPath applies
bundle/template cache is updated
runtime database/module state includes the change
```

Separate repository correctness from installed database/asset runtime state.

---

# 72. Patch Debugging

When patched behavior does not execute, verify:

```text
correct target object/prototype/class
file loaded
patch syntax valid for version
method name still exists
another patch did not replace/wrap behavior unexpectedly
runtime consumer actually uses the patched target
```

Do not broaden the patch until target identity is proven.

---

# 73. Race Condition Diagnosis

Symptoms such as stale data, flicker, wrong record content, duplicate action, or state reverting may indicate async ordering issues.

Gather evidence of request/event ordering before adding arbitrary delays.

Do not "fix" a race with `setTimeout`/sleep unless timing itself is the documented contract and framework pattern.

Prefer deterministic ownership/order/cancellation/idempotency mechanisms.

---

# 74. Browser-Only Reproduction

Some issues cannot be proven statically.

If reproduction requires browser runtime, record:

```text
page/action
user
company/website/language
initial state
interaction sequence
expected result
actual result
console evidence
network evidence
server-log evidence
```

If the current environment cannot run a browser, mark the runtime requirement explicitly rather than claiming the bug is fixed from static review.

---

# VERSION COMPATIBILITY AND FRONTEND MIGRATION

# 75. Source-Grounded API Compatibility

For Odoo-version migration or when touching version-sensitive frontend code, verify each material API against target-version source.

Examples of version-sensitive areas:

```text
module import paths
OWL version/lifecycle behavior
patch API
registry APIs
service APIs
view/controller/renderer architecture
RPC/ORM helpers
website editor/snippet APIs
POS architecture
frontend test framework
asset bundle names/structure
```

Do not maintain or rely on a giant remembered compatibility table.

Use actual source evidence.

---

# 76. Copying Upstream Code Is Upgrade Debt

If an old customization copied a core component/template, compare it with the detected target-version upstream implementation before modifying it.

Prefer replacing the copy with a narrower supported extension when the authorized scope and risk justify it.

Do not perform a large migration/refactor merely because copied code is undesirable if the current task only needs a contained safe fix.

Record upgrade debt when it remains.

---

# 77. Removed or Renamed Frontend APIs

When an import/method/service/registry/template no longer exists:

1. confirm it existed in the source version if relevant;
2. find the target-version owner/replacement;
3. inspect nearby target-version usage;
4. adapt to the actual target architecture;
5. trace changed consumers/contracts;
6. define runtime verification.

Do not create compatibility aliases in custom code unless the project genuinely needs cross-version support and the boundary is maintainable.

---

# IMPLEMENTATION PROCEDURE

# 78. Frontend Implementation Sequence

For material frontend work, use this sequence:

```text
1. Determine task mode and acceptance criteria.
2. Reuse existing investigation/impact evidence.
3. Detect Odoo version and frontend surface.
4. Identify component/template/service/registry/patch/asset/server ownership.
5. Inspect nearby repository/core patterns for this version.
6. Map the frontend-to-server contract.
7. Search for existing implementations/patches/registrations.
8. Choose the smallest safe extension boundary.
9. Implement the scoped change.
10. Perform static import/template/asset consistency checks.
11. Add/update durable automated tests when justified.
12. Perform required browser/runtime validation.
13. Review final diff and do-not-touch boundary.
14. Report confirmed vs runtime-unverified behavior explicitly.
```

Do not start with a global patch simply because it is quick to write.

---

# 79. Static Frontend Checks

Before browser validation, inspect relevant static correctness.

Check when applicable:

```text
valid JS/module syntax
valid imports/exports
version-correct paths
component registration
registry keys
patch target/method names
template identifiers
XML well-formedness
XPath targets
asset declarations
manifest dependencies
CSS/SCSS syntax/imports
controller/model contract consistency
localization integration
```

Static checks reduce avoidable runtime failures but do not prove browser behavior.

---

# 80. Preserve Backend Contracts

A frontend implementation must not casually redefine server behavior.

If frontend work requires backend changes, make the server contract explicit and keep business/security rules authoritative there.

Trace all existing server consumers before changing shared model methods or routes.

If backend impact expands materially, refresh relevant impact/security/performance evidence rather than treating it as "just frontend".

---

# 81. Final Diff Review

Inspect the actual final diff.

Look for:

- unrelated JavaScript/XML/CSS changes;
- accidental global asset inclusion;
- duplicate registry keys;
- duplicate patches;
- copied upstream blocks larger than necessary;
- debug logging;
- commented-out code;
- temporary selectors;
- arbitrary delays;
- hardcoded IDs/URLs;
- accidental `sudo()` added server-side;
- response-contract changes;
- missing dependencies;
- stale asset/test files;
- disabled/skipped tests.

Do not hide a widened scope behind a small user-visible change.

---

# AUTOMATED TEST HANDOFF

# 82. Durable Frontend Test Coverage

When the frontend behavior represents a meaningful regression risk, provide the Automated Test Engineer with:

```text
protected behavior
frontend surface
component/service/registry/patch target
server contract
actor/company/website context
positive scenario
negative/boundary scenario
browser-only dependency
version-specific test framework evidence
```

Do not duplicate the full automated-test engineering workflow inside this skill.

---

# 83. Choose Unit vs Browser Coverage

As frontend architecture evidence, identify which layer is capable of proving the behavior:

```text
frontend unit test
Python/controller test
browser/tour test
combination
runtime-only scenario
```

The Automated Test Engineer owns final test design/implementation.

Do not create a browser test for pure server business logic solely because the feature is visible in the UI.

---

# RUNTIME VALIDATION HANDOFF

# 84. Separate Static Confidence From Browser Proof

The following are not equivalent:

```text
JS file parses
    != component runs in browser

asset declared
    != correct bundle loads in target page

patch target found in source
    != patch composes correctly with installed custom addons

RPC contract looks correct
    != real user/company/server state accepts it

frontend unit test passes
    != production asset build/browser integration passes
```

Use the existing Regression & Runtime Validator for environment-specific proof.

---

# 85. Browser Runtime Requirement

When browser proof is required, record an exact scenario.

Use:

```text
BROWSER RUNTIME REQUIREMENT

Surface/page/action:
Odoo version/build:
Actor:
Company:
Website:
Language/RTL if relevant:
Initial state:
Interaction:
Expected UI result:
Expected RPC/server result:
Console expectation:
Network expectation:
Regression behavior to re-check:
```

Do not write only "test in browser".

---

# 86. Asset Runtime Requirement

For asset-sensitive changes, runtime validation should confirm when relevant:

```text
module upgrade/restart requirements
asset rebuild
correct bundle/page
no import/template errors
patch/registry/service initialization
no duplicate execution
production-like asset mode when material
```

Do not claim runtime asset success from repository declarations alone.

---

# 87. Multi-Company / Multi-Website Runtime Requirement

When company or website state is runtime/database-dependent, specify explicit contexts to validate.

Examples:

```text
User with company A only
User with A+B, active A
User with A+B, active B
Website A public session
Website B public session
portal user owning one record but not another
```

Use only contexts justified by the feature's risk surface.

---

# OUTPUT CONTRACT

# 88. Frontend Evidence

For material frontend work, produce a structured evidence block.

Use:

```text
FRONTEND EVIDENCE

Task mode:
Odoo version:
Version evidence:
Frontend surface:
Target module(s):
Feature owner:

Component(s):
Template(s):
Service(s):
Registry entries:
Patch target(s):
Asset bundle(s):
Controller/RPC/model boundary:

Current frontend flow:
Existing extensions/patches:
Version-sensitive APIs verified:

Requested behavior:
Recommended frontend boundary:
Why this boundary is safe:

State/lifecycle risks:
Async/race risks:
Asset/load-order risks:
Template/DOM contract risks:
Security boundary:
Company/website context:
Performance considerations:
Localization/RTL considerations:

Automated test requirements:
Browser/runtime requirements:
Do-not-touch boundary:
Remaining unknowns:

Frontend status:
Confidence:
```

Useful frontend statuses:

```text
READY FOR IMPLEMENTATION
IMPLEMENTED / RUNTIME VALIDATION REQUIRED
VALIDATED FOR REVIEWED SCOPE
PARTIAL / NEEDS BROWSER EVIDENCE
PARTIAL / NEEDS SERVER CONTRACT EVIDENCE
BLOCKED
```

Do not report `VALIDATED FOR REVIEWED SCOPE` when required browser proof was unavailable.

---

# 89. Concise Output for Embedded Implementation Work

When this guidance runs inside an already-authorized implementation task, keep evidence concise unless the frontend surface is complex.

A concise handoff may be:

```text
Frontend surface:
Version/evidence:
Owner:
Component/template/asset:
Server contract:
Existing extensions/patches:
Safe frontend boundary:
Key risks:
Runtime test:
Do-not-touch:
```

Then continue through Plemo's native planning/implementation workflow.

Do not expose internal skill routing or produce a second generic implementation plan.

---

# 90. Full Frontend Review Output

For standalone frontend audits, complex bugs, architecture changes, or version migrations, provide relevant sections from:

```text
1. Frontend Target
2. Odoo Version / Evidence
3. Frontend Surface
4. Feature Ownership
5. Component Architecture
6. Template/QWeb/OWL Chain
7. Service Dependencies
8. Registry Entries
9. Patch Chain
10. Asset Bundle / Load Order
11. Frontend-to-Server Contract
12. State Ownership
13. Lifecycle Analysis
14. Async / Race Analysis
15. DOM / Event Contract
16. Website / Portal / POS Context
17. Company / Website Context
18. Security Boundary
19. Performance Considerations
20. Existing Implementations / Conflicts
21. Recommended Safe Frontend Boundary
22. Automated Test Requirements
23. Browser Runtime Requirements
24. Do-Not-Touch Areas
25. Remaining Unknowns
26. Frontend Evidence
```

Only include sections relevant to the actual feature.

Do not invent empty complexity.

---

# 91. Frontend Finding Severity

For frontend reviews, classify findings by impact.

Possible severities:

```text
CRITICAL
HIGH
MEDIUM
LOW
INFO
```

Examples:

### CRITICAL

- public frontend exposes or changes protected records through an unauthorized server path;
- frontend implementation can trigger destructive business effects repeatedly without server-side protection;
- a global patch breaks core application startup across major surfaces.

### HIGH

- shared patch/registry replacement changes behavior for many unrelated consumers;
- asset/load-order issue prevents a material application flow from loading;
- race condition can create duplicate payments/orders/other high-impact effects;
- multi-company/website frontend exposes wrong tenant data because server contract is unsafe.

### MEDIUM

- component lifecycle leaks repeated handlers;
- brittle template/DOM selector is likely to break on normal rerender/upgrade;
- local async state can display stale data;
- duplicated frontend implementation creates meaningful maintenance risk.

### LOW

- small naming/organization issue;
- contained CSS specificity concern;
- minor duplication with low change risk.

Keep severity separate from confidence.

Do not classify subjective JavaScript style preferences as high-severity defects.

---

# 92. Frontend Confidence

Express confidence based on evidence.

Useful levels:

```text
HIGH
MEDIUM
LOW
```

### HIGH

- target Odoo version confirmed;
- owning source/components/templates/assets verified;
- extension/patch chain traced;
- server contract inspected;
- relevant browser/runtime proof completed when required.

### MEDIUM

- repository architecture is clear but browser/runtime proof is unavailable;
- some installed-only patch/template/asset interaction remains unverified.

### LOW

- version/front-end API ownership uncertain;
- browser symptom cannot be reproduced;
- target component/asset/registry chain not confirmed;
- database-only website/company state materially affects behavior and is unavailable.

Never convert lack of runtime evidence into high confidence.

---

# 93. Do-Not-Touch Boundary

Record explicit boundaries when useful.

Examples:

```text
Do not replace the global registry entry for a local screen.
Do not copy the full upstream component.
Do not rewrite unrelated legacy widgets.
Do not move assets between global bundles without consumer evidence.
Do not add DOM polling or arbitrary delays for a lifecycle problem.
Do not move server business rules into JavaScript.
Do not broaden server sudo/access to make the UI work.
Do not rename stable template/registry/service contracts for aesthetics.
Do not introduce a new frontend framework/library for one fix.
Do not convert a contained fix into an Odoo-version frontend migration unless authorized.
```

This boundary protects scope and upgrade safety.

---

# 94. Stop Conditions

Stop and report rather than guessing when:

- the Odoo version cannot be determined and the required frontend API is version-sensitive;
- the actual component/template/service/registry/patch owner cannot be identified;
- the asset bundle or frontend surface cannot be established;
- a required import/API cannot be verified against source or working repository code;
- security behavior is ambiguous and proceeding could expose protected data/actions;
- a public/portal route requires an authorization decision not supported by evidence;
- a browser-only defect cannot be verified and repository evidence is insufficient to claim a fix;
- a required change would materially broaden scope beyond the user's authorization;
- the user requested review only and code modification would be required;
- runtime validation would require unsafe production actions.

A stop condition should identify:

```text
what is blocked
why it is blocked
what repository evidence is confirmed
what runtime/source input is required next
```

Do not replace missing evidence with remembered Odoo APIs.

---

# 95. Final Principles

The objective is not:

```text
maximum OWL usage
maximum patching
maximum component abstraction
maximum client-side logic
maximum browser automation
```

The objective is:

```text
version-correct Odoo frontend behavior
implemented at the narrowest stable extension boundary
with clear state/lifecycle/server contracts
and explicit browser/runtime proof requirements
```

Prefer:

```text
verified target-version APIs
    >
remembered import paths

framework extension point
    >
manual DOM hack

local extension
    >
global patch

single state owner
    >
duplicated synchronized state

server business/security enforcement
    >
client-only enforcement

explicit async ordering
    >
timing luck

narrow asset scope
    >
unnecessary global loading

semantic template contracts
    >
fragile DOM depth/selectors

runtime browser evidence
    >
static assumptions when browser proof is required

repository conventions
    >
generic frontend preferences
```

The final result should make clear:

```text
what owns the frontend behavior
which Odoo-version APIs and extension points were verified
where the change belongs
what contracts and runtime risks must be preserved
what browser/runtime evidence is still required
and how confident we are in the result
```
