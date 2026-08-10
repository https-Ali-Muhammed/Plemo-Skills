# Odoo Codebase Investigator

## Purpose

Use this skill to investigate an Odoo codebase before any implementation, modification, or architectural decision.

The skill must determine how the repository is structured, which module owns a feature, which modules extend or override it, how XML/QWeb inheritance works, how Python inheritance and controllers connect to the feature, how JavaScript and assets participate, whether an implementation already exists, and where a future change should safely be made.

This skill is investigation-only by default.

Do not modify files, create files, fix issues, implement features, refactor code, update the database, commit, or push during the investigation.

Core principles:

- Investigate first.
- Understand ownership.
- Trace inheritance.
- Trace dependencies.
- Trace execution flow.
- Search for existing implementations.
- Report before modifying.

---

## 1. Determine the Investigation Target

Determine what the user wants investigated.

The target may be a project, module, page, website section, XML view, QWeb template, form, button, field, model, Python method, controller, route, JavaScript behavior, OWL component, public widget, asset, error, business feature, workflow, performance problem, or existing customization.

If the target is already clear, do not ask unnecessary questions. If unclear, ask what project, module, page, feature, or problem should be investigated.

---

## 2. Detect the Odoo Version

Before making architectural assumptions, determine the Odoo version.

Inspect strong evidence such as:

```text
odoo/release.py
odoo/version.py
odoo/__init__.py
version_info
repository branch
Docker configuration
requirements
existing JavaScript imports
existing frontend APIs
project configuration
```

Prefer direct evidence from the Odoo source.

Record:

```text
Odoo version:
Evidence:
Frontend architecture:
```

Do not assume the version from folder names, repository names, database names, comments, or copied code.

---

## 3. Find the Repository Root

Identify the actual repository root.

Inspect for:

```text
.git/
odoo/
addons/
enterprise/
requirements files
Docker files
configuration files
customer directories
custom addon directories
```

Record:

```text
Repository root:
Odoo source path:
Standard addon paths:
Customer addon paths:
Other configured addon paths:
```

---

## 4. Custom Project Structure

Customer-specific Odoo projects in this environment normally use:

```text
odoo/
└── Customers/
    └── <project_name>/
        └── addons/
            ├── custom_module_1/
            ├── custom_module_2/
            └── ...
```

The primary custom-module location is:

```text
odoo/Customers/<project_name>/addons/
```

Example:

```text
odoo/Customers/Taverna/addons/
```

When a project/customer name is provided, investigate this directory first for project-specific modules.

Do not treat:

```text
odoo/Customers/<project_name>/
```

as an Odoo module. The modules are normally inside its `addons/` directory.

---

## 5. Customer Project Discovery

For a known project:

1. Locate `odoo/Customers/<project_name>/`.
2. Locate `odoo/Customers/<project_name>/addons/`.
3. Enumerate candidate modules.
4. Verify modules by checking for `__manifest__.py`.
5. Read manifests for relevant modules.
6. Search customer modules first for customizations.
7. Trace those customizations back into their original Odoo or dependency modules.

Do not assume that a feature visible in a customer website was originally defined by a customer module.

---

## 6. Understand the Repository Structure

Classify relevant code areas as:

- Odoo core;
- Odoo standard addons;
- Enterprise addons;
- customer-specific addons;
- shared custom addons;
- theme modules;
- integration modules;
- utility modules;
- unrelated customer projects.

A typical structure may be:

```text
odoo18/
├── addons/
├── filestore/
├── odoo/
│   ├── addons/
│   └── Customers/
│       ├── Customer_A/
│       │   └── addons/
│       ├── Customer_B/
│       │   └── addons/
│       └── Taverna/
│           └── addons/
└── ...
```

Do not search unrelated customer projects unless there is evidence that shared code or dependencies are involved.

---

## 7. Feature Ownership Search Order

For a known customer project, use this practical search order:

1. `odoo/Customers/<project_name>/addons/`
2. other configured custom addon paths;
3. `odoo/addons/`;
4. other repository addon directories such as `addons/`;
5. Enterprise addons if available.

This order is for efficient discovery, not for assuming ownership.

Always trace a feature back to its original owner.

---

## 8. Identify Relevant Modules

Search by:

- feature name;
- template ID;
- XML external ID;
- model name;
- field name;
- controller route;
- button text;
- visible text;
- CSS class;
- DOM ID;
- data attribute;
- JavaScript function;
- component;
- RPC route;
- service;
- asset bundle;
- Python method.

Classify modules as:

```text
Original owner
Primary customer customization
Direct extension
Indirect extension
Dependency
Possible related module
Unrelated
```

---

## 9. Read Relevant Module Manifests

For each relevant module inspect `__manifest__.py`.

Record important fields such as:

```text
name
version
depends
data
assets
demo
auto_install
application
installable
```

Pay special attention to `depends`, `data`, and `assets`.

---

## 10. Build the Module Dependency Graph

Build a dependency relationship for relevant modules.

Example:

```text
website_event
    ↓
theme_aquacity
    ↓
pl_aqua_event_services
```

Explain important dependency relationships when relevant.

---

## 11. Detect Missing or Suspicious Dependencies

If one module references another module's external ID, view, model, controller, template, asset, or field, verify that the dependency is appropriately declared in `__manifest__.py`.

Example:

If a custom module contains:

```xml
<field name="inherit_id" ref="website_event.some_template"/>
```

verify that the required dependency on `website_event` is declared where appropriate.

Report suspicious or missing dependencies. Do not modify the manifest during investigation.

---

## 12. Find the Original Feature Owner

Find the original:

- XML view;
- QWeb template;
- model;
- field;
- method;
- controller;
- route;
- JavaScript component;
- asset.

Distinguish:

```text
Original owner
Extension
Override
Theme customization
Customer customization
Runtime/database customization
```

Do not stop at the first custom file that matches.

---

## 13. Trace the Feature End-to-End

Typical frontend-to-backend flow:

```text
XML / QWeb
    ↓
JavaScript
    ↓
RPC / HTTP request
    ↓
Controller
    ↓
Model
    ↓
Database / business logic
```

Backend-to-frontend flow may be:

```text
Model
    ↓
Controller
    ↓
QWeb context
    ↓
Template
    ↓
JavaScript
    ↓
Rendered UI
```

Trace every relevant layer. Do not stop at the first matching file.

---

# XML AND QWEB INHERITANCE

## 14. XML Inheritance Is a Primary Investigation Target

Whenever XML or QWeb is involved, identify:

- original view;
- external ID;
- module owner;
- source XML file;
- model;
- view type;
- `inherit_id`;
- parent view;
- child inherited views;
- XPath expressions;
- `position`;
- priority;
- groups;
- website restrictions;
- relevant context;
- purpose of each inheritance layer.

Do not recommend an XML modification until the inheritance chain is understood.

---

## 15. Resolve the Original View

When finding:

```xml
<field name="inherit_id" ref="website_sale.product"/>
```

resolve:

```text
Module: website_sale
External ID: website_sale.product
Source file: actual XML file defining the view
```

Then inspect the original view.

Do not stop at the inherited record.

---

## 16. Trace XML Inheritance Upward

For every inherited view, trace upward until the original view is found.

Example:

```text
customer_module.custom_product_view
    ↓ inherits
theme_module.product_view
    ↓ inherits
website_sale.product
```

Report the chain in original-to-custom order:

```text
website_sale.product
    ↓
theme_module.product_view
    ↓
customer_module.custom_product_view
```

---

## 17. Trace XML Inheritance Downward

Do not only follow `inherit_id` upward.

For every central view, search for all views that inherit it. Then search for views inheriting those inherited views.

Build the descendant tree when relevant.

---

## 18. Always Search XML Inheritance in Both Directions

Upward:

```text
Current custom view
    ↓
Parent view
    ↓
Parent view
    ↓
Original Odoo view
```

Downward:

```text
Original Odoo view
    ↓
Direct inherited views
    ↓
Views inheriting those views
    ↓
Further customizations
```

An XML investigation is incomplete if only one direction is checked.

---

## 19. Build an XML Inheritance Graph

For complex cases, create a graph such as:

```text
website_event.event_description_full
│
├── theme_aquacity.event_description
│   │
│   └── pl_aqua_event_services.event_description_services
│
└── another_module.event_description_custom
```

For each node include:

```text
Module:
Module type:
External ID:
File:
inherit_id:
Purpose:
```

---

## 20. Classify View Ownership

Classify each relevant view as:

```text
Odoo standard
Enterprise
Customer custom
Theme
Shared custom
Other extension
```

---

## 21. Analyze Every Relevant XPath

For every inherited view record:

```text
XPath expression:
Position:
Target:
Effect:
```

Common positions:

```text
inside
before
after
replace
attributes
```

Explain what the XPath actually changes.

---

## 22. Trace XPath Targets

For each relevant XPath:

1. find its target in the parent view;
2. confirm what it selects;
3. determine whether an earlier inherited view changed that element;
4. determine whether a later view changes it again.

This is especially important for `replace`, `attributes`, and `inside`.

---

## 23. Detect XML Inheritance Conflicts

Search for multiple views targeting:

- the same XPath;
- the same element;
- the same attribute;
- nearby structure.

Report risks such as:

- two modules replacing the same node;
- one module removing a node another expects;
- multiple modules changing the same attribute;
- a fragile XPath depending on structure changed earlier.

---

## 24. Investigate View Priority

When several inherited views affect the same parent, inspect priority where relevant.

Do not assume source-file order determines the final result.

Record:

```text
View:
Priority:
Possible effect:
```

---

## 25. Resolve External IDs Carefully

Distinguish:

```text
XML record ID
external ID
template ID
DOM ID
CSS ID
```

Always resolve important records to their full external ID, such as:

```text
custom_event.event_checkout_template
```

---

## 26. QWeb Inheritance

Investigate version-appropriate QWeb inheritance mechanisms such as:

```text
t-inherit
t-inherit-mode
xpath
```

and legacy mechanisms when relevant.

Build:

```text
Original template
    ↓
Inherited template
    ↓
Further inherited templates
```

Explain what each layer changes.

---

## 27. Website and Theme Inheritance

For website features investigate:

- base website templates;
- theme templates;
- customer templates;
- snippets;
- portal templates;
- event templates;
- checkout templates;
- website-specific inherited views.

Determine whether visible content comes from standard Odoo, a theme, a customer module, or a database-created view.

---

## 28. Database-Only View Customizations

The final UI may differ from repository XML because of:

```text
Website Editor
Odoo Studio
manually created ir.ui.view records
database-only inherited views
database modifications
```

If repository code cannot explain the visible result and database access is available, investigate relevant `ir.ui.view` records.

Clearly separate repository-defined inheritance from database-only customization.

---

## 29. Separate Source Evidence From Runtime State

Runtime behavior may depend on:

- installed modules;
- view priority;
- database views;
- active/inactive views;
- website;
- company;
- groups;
- language;
- context;
- asset bundles;
- module versions.

State whether a conclusion is:

```text
Confirmed by repository
Strong evidence
Probable
Needs runtime/database verification
```

---

# PYTHON MODEL INVESTIGATION

## 30. Investigate Model Inheritance

Search for:

```text
_name
_inherit
_inherits
```

Distinguish extension inheritance, new-model inheritance, and delegation inheritance.

Build relevant model inheritance chains.

---

## 31. Build a Model Inheritance Graph

Example:

```text
event.event
    ↓
theme_aquacity/models/event.py
    ↓
pl_aqua_event_services/models/event.py
```

For each extension record:

```text
Module:
File:
Model:
Fields added:
Methods overridden:
Purpose:
```

---

## 32. Investigate Field Ownership

For important fields determine:

- original module;
- original model;
- defining file;
- field type;
- extensions;
- overridden attributes;
- XML views displaying it.

Do not assume the XML module owns the field.

---

## 33. Investigate Method Overrides

For important methods:

1. find the original definition;
2. search all relevant modules for overrides;
3. inspect `super()` calls;
4. determine the likely execution chain.

Example:

```text
sale.order._cart_update
    ↓
website_sale implementation
    ↓
custom_shop override
    ↓
customer_module override
```

Report overrides that omit `super()` when that may affect behavior.

---

## 34. Trace `super()` Chains

Record:

```text
Original method:
Override 1:
Override 2:
super() behavior:
Likely execution order:
```

Do not assume the last source file found is the only implementation that executes.

---

# CONTROLLER INVESTIGATION

## 35. Investigate Controllers

For relevant controllers identify:

- module;
- file;
- controller class;
- base controller;
- route;
- request type;
- HTTP methods;
- auth;
- website flag;
- CSRF behavior when relevant;
- model methods called;
- templates returned;
- JSON response structure when relevant.

---

## 36. Controller Inheritance

Search for subclasses and overrides.

Record:

```text
Original controller:
Inherited controllers:
Methods overridden:
Routes redeclared:
Routes extended:
```

---

## 37. Route Ownership

Record:

```text
Route:
Original module:
Controller:
Overrides:
Called models:
Called from:
Returned result:
```

---

## 38. Trace Route Execution

Example:

```text
booking_form.js
    ↓
/event/check_availability
    ↓
EventController.check_availability()
    ↓
event.event.check_availability()
    ↓
database
    ↓
JSON response
    ↓
booking_form.js
    ↓
UI
```

---

# JAVASCRIPT INVESTIGATION

## 39. Investigate JavaScript

Search for:

- DOM selectors;
- event listeners;
- widgets;
- components;
- OWL components;
- services;
- registries;
- patches;
- RPC calls;
- HTTP calls;
- imports;
- public widgets;
- template references;
- data attributes;
- state management.

Identify the actual file controlling the behavior.

---

## 40. JavaScript Overrides and Patches

Search for extension mechanisms appropriate to the Odoo version, including when applicable:

```text
patch()
registry entries
service overrides
component inheritance
publicWidget.extend
include()
legacy widget extension
```

Record:

```text
Original JS component:
Patch/extension:
Module:
File:
Behavior changed:
```

---

## 41. Trace XML to JavaScript

Search XML markup for classes, IDs, data attributes, template names, component names, `t-ref`, and event attributes consumed by JS.

Example:

```html
class="aqua-booking-form"
```

Search JS for:

```javascript
.aqua-booking-form
```

---

## 42. Trace JavaScript to XML

For selectors, `dataset`, `querySelector`, template names, and similar references in JS, find the corresponding XML/QWeb markup.

Example:

```javascript
element.dataset.eventId
```

Search XML for:

```text
data-event-id
```

---

## 43. Trace JavaScript to Controller

For frontend requests record:

```text
JS file:
Function:
Route:
Payload:
Controller:
Model method:
Response:
UI handler:
```

---

# ASSET INVESTIGATION

## 44. Investigate Assets

Inspect:

```text
__manifest__.py
asset declarations
legacy asset XML when applicable
JavaScript
SCSS
CSS
XML templates
```

Determine:

- which bundle contains a file;
- whether the file is actually loaded;
- whether a glob includes it;
- whether it is explicitly listed;
- whether another module changes the same bundle.

---

## 45. Asset Load Order

When relevant investigate:

- dependency ordering;
- bundle ordering;
- patches;
- before/after directives where supported;
- duplicate inclusions;
- missing inclusions.

A valid implementation may still fail if its asset is not loaded.

---

# EXISTING IMPLEMENTATION SEARCH

## 46. Search Before Recommending New Code

Search for existing logic using:

- feature names;
- methods;
- routes;
- field names;
- external IDs;
- template IDs;
- labels;
- selectors;
- JS functions;
- error messages;
- business terms;
- RPC endpoints.

Do not recommend duplicate code before checking whether a reusable implementation exists.

---

## 47. Search Beyond the Named Module

When needed, search across:

- customer addons;
- shared custom addons;
- Odoo standard addons;
- Enterprise addons;
- dependencies.

Do not search unrelated customer projects without a reason.

---

## 48. Detect Duplicate Implementations

Look for:

- repeated JS validation;
- copied model calculations;
- duplicate controller routes;
- duplicate XPath customizations;
- duplicate helpers;
- duplicate fields.

Report meaningful duplication.

---

## 49. Summarize Feature Ownership

At the end classify ownership clearly.

Example:

```text
Original feature owner:
website_event

Customer UI customization:
theme_aquacity

Business logic extension:
pl_aqua_event_services

JavaScript owner:
theme_aquacity

Controller owner:
pl_aqua_event_services

Base model:
event.event

Model extension:
pl_aqua_event_services
```

---

# INVESTIGATION ENTRY POINTS

## 50. Starting From a UI Element

1. Identify text/classes/data attributes.
2. Locate the XML/QWeb template.
3. Resolve the external ID.
4. Find the owning module.
5. Trace parent inheritance.
6. Trace child inheritance.
7. Analyze XPath changes.
8. Find connected JS.
9. Find requests/routes.
10. Find controller.
11. Find model methods.
12. Find model inheritance.
13. Verify assets.

Do not stop at the first template.

---

## 51. Starting From XML

1. Resolve the external ID.
2. Identify the source module.
3. Identify the XML file.
4. Check `inherit_id`.
5. Trace upward to the original view.
6. Search downward for relevant child views.
7. Analyze XPath modifications.
8. Analyze priority.
9. Check groups/context/website restrictions.
10. Find connected JS.
11. Verify assets.
12. Identify models and fields referenced.

---

## 52. Starting From a Python Method

1. Locate the model.
2. Find the original method.
3. Search overrides.
4. Trace `super()` calls.
5. Find callers.
6. Find controllers.
7. Find frontend requests.
8. Identify module dependencies.

---

## 53. Starting From a Controller

1. Locate the route definition.
2. Identify the controller.
3. Find inherited controllers.
4. Find route overrides.
5. Identify models called.
6. Identify templates returned.
7. Search JS calls.
8. Verify assets loading that JS.

---

## 54. Starting From JavaScript

1. Identify module and file.
2. Find imports.
3. Identify component/widget/service.
4. Find selectors.
5. Find corresponding XML.
6. Find RPC/routes.
7. Find controller.
8. Find model calls.
9. Identify patches/overrides.
10. Verify asset inclusion.

---

# SAFETY AND SCOPE

## 55. Confidence Levels

Use these when useful:

```text
Confirmed
Strong evidence
Probable
Needs runtime verification
```

Do not present an unverified runtime assumption as confirmed.

---

## 56. No Modifications During Investigation

This skill is read-only by default.

Do not:

- edit files;
- create files;
- delete files;
- format files;
- modify manifests;
- modify XML;
- modify Python;
- modify JavaScript;
- modify SCSS;
- update translations;
- update the database;
- install modules;
- upgrade modules;
- commit;
- push;
- merge;
- rebase;
- create PRs.

Implementation should happen only after the report and an explicit user request.

---

## 57. Safe Read-Only Commands

Examples:

```text
find
grep
rg
git status
git diff
git log
git show
cat
sed for reading
head
tail
tree
ls
```

Do not use commands that alter files or repository state.

---

## 58. Do Not Modify Even If the Fix Is Obvious

If the investigation reveals an obvious fix, report:

```text
Issue:
Cause:
Recommended file:
Recommended change:
```

Do not apply it during investigation.

---

# REQUIRED REPORT

## 59. Odoo Codebase Investigation Report

Always produce a report before implementation.

Use this structure:

### 1. Investigation Target

```text
Project:
Feature:
Requested module, if any:
Repository:
Odoo version:
```

### 2. Executive Summary

Explain:

- original feature owner;
- customer/custom modules extending it;
- important code locations;
- major inheritance relationships;
- safest future modification location.

### 3. Repository Structure

```text
Repository root:
Odoo source:
Customer project:
Customer addons:
Other relevant addon paths:
```

### 4. Relevant Modules

For each module:

```text
Module:
Type:
Path:
Purpose:
Depends:
```

### 5. Module Dependency Graph

Example:

```text
website_event
    ↓
theme_aquacity
    ↓
pl_aqua_event_services
```

### 6. Feature Ownership

```text
Original feature owner:
UI owner:
XML/QWeb owner:
JavaScript owner:
Controller owner:
Model owner:
Asset owner:
```

### 7. XML / QWeb Inheritance

```text
Original view external ID:
Module:
Module type:
File:
```

Inheritance chain:

```text
original.view
    ↓
module_a.inherited_view
    ↓
module_b.inherited_view
```

For each relevant inherited view:

```text
External ID:
Module:
File:
inherit_id:
Priority:
XPath:
Position:
Effect:
```

### 8. XML Inheritance Graph

Example:

```text
website_event.original_view
│
├── theme_aquacity.custom_view
│   │
│   └── pl_aqua_event_services.extended_view
│
└── another_module.other_view
```

### 9. XPath Analysis

```text
XPath:
Position:
Target:
Effect:
Potential conflict:
```

### 10. Model Inheritance

```text
Model:
Original module:
Extensions:
Fields involved:
Methods involved:
```

### 11. Method Overrides

```text
Method:
Original implementation:
Override 1:
Override 2:
super() chain:
Likely execution flow:
```

### 12. Controller Flow

```text
Route:
Original controller:
Overrides:
Auth/type:
Models called:
Template/response:
```

### 13. JavaScript Flow

```text
Entry file:
Component/widget:
Selectors:
Template connection:
RPC/HTTP calls:
Patches/overrides:
```

### 14. Asset Flow

```text
Bundle:
Relevant files:
How included:
Ordering concerns:
Missing/duplicate assets:
```

### 15. End-to-End Feature Flow

Example:

```text
XML button
    ↓
booking_form.js
    ↓
/event/book
    ↓
EventController
    ↓
event.event.action_book()
    ↓
database
    ↓
response
    ↓
JavaScript
    ↓
UI
```

### 16. Existing Implementations Found

List code already implementing all or part of the requested behavior.

### 17. Overrides and Conflicts

```text
XML conflicts:
XPath conflicts:
Model overrides:
Controller overrides:
JS patches:
Asset conflicts:
Dependency issues:
```

### 18. Risks

Examples:

```text
Fragile XPath
Multiple modules targeting the same element
Missing dependency
Duplicate implementation
Override without super()
Database-only customization
Asset load-order issue
Customer code unexpectedly overriding standard behavior
```

### 19. Recommended Modification Location

Do not implement.

State:

```text
Recommended module:
Recommended file:
Recommended view/model/controller:
Reason:
```

### 20. Files Investigated

List the important files actually examined.

### 21. Remaining Unknowns

List anything requiring:

```text
database access
runtime inspection
browser testing
installed-module verification
additional repository access
```

---

## 60. XML Inheritance Report Is Mandatory When XML Is Involved

If XML/QWeb participates in the feature, explicitly show:

```text
Original View
    ↓
Inherited View
    ↓
Inherited View
    ↓
Final Customization
```

For every relevant view provide:

```text
External ID
Module
Module type
File
inherit_id
Priority when relevant
XPath
Position
Effect
```

Do not omit this section.

---

## 61. Find All Relevant Child Views

When a view is central to the investigation, search for child views inheriting both the original view and important intermediate views.

A separate custom module may modify the final rendered structure.

---

## 62. Recommend the Safest Future Change Location

Prefer:

- customer/custom modules over Odoo core;
- XML inheritance over copying full templates;
- model extension over copying business logic;
- existing controller/model flows over duplicate routes;
- reusable JS components/helpers over duplicate implementations.

This is a recommendation only. Do not implement it during investigation.

---

## 63. Core Questions to Answer

Before finishing, answer as many as relevant:

1. What Odoo version is used?
2. Where is the customer project?
3. Which custom modules are relevant?
4. Which module originally owns the feature?
5. Which customer module changes it?
6. Which XML/QWeb view originally defines it?
7. What is its full external ID?
8. What is the complete XML inheritance chain?
9. Which other views inherit the original view?
10. Which views inherit intermediate custom views?
11. Which XPath expressions modify the relevant area?
12. What does each XPath change?
13. Are XPath operations conflicting?
14. Which model owns the data?
15. Which modules inherit the model?
16. Which module defines the field?
17. Which methods are overridden?
18. What is the `super()` chain?
19. Which controller handles the request?
20. Is the controller or route overridden?
21. Which JS controls the frontend behavior?
22. How does JS connect to XML?
23. Which route does JS call?
24. Which asset bundle loads the JS?
25. Are required dependencies declared?
26. Is there an existing implementation to reuse?
27. Are duplicate implementations present?
28. Where is the safest future modification location?
29. Which files/modules should not be modified?
30. What still requires runtime/database verification?

---

## 64. Primary Rules

Investigate before modifying.

Understand repository structure before analyzing one file.

For customer projects, start with:

```text
odoo/Customers/<project_name>/addons/
```

but always trace features back to their original owners.

Read manifests and build dependency relationships.

For XML:

- trace upward to the original view;
- trace downward to relevant child views;
- resolve full external IDs;
- analyze XPath operations;
- inspect priority where relevant;
- build an inheritance graph.

For Python:

- trace model inheritance;
- trace field ownership;
- trace method overrides;
- trace `super()` chains.

For controllers:

- trace original routes;
- trace inherited controllers;
- trace route overrides.

For JavaScript:

- trace selectors;
- trace XML connections;
- trace RPC calls;
- trace patches and overrides.

For assets:

- verify files are actually loaded;
- verify the correct bundle;
- investigate load order when relevant.

Search the relevant codebase for an existing implementation before recommending new code.

Do not modify anything during investigation.

Always produce the investigation report first.
