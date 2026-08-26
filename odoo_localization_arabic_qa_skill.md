# Odoo Localization & Arabic QA

## Purpose

Use this guidance to investigate, implement, review, and validate Arabic localization for an Odoo module.

The analysis must work across different Odoo versions. Never assume the Odoo version, module path, frontend architecture, translation structure, or available JavaScript APIs before inspecting the actual project.

The required localization architecture is:

1. **JavaScript user-facing strings**
   - Use:
     ```javascript
     localize("English text", "Arabic text")
     ```
   - The helper must live at:
     ```text
     <module>/static/src/js/utils/localization.js
     ```

2. **Standard Odoo translatable content**
   - Use Odoo's normal translation system and:
     ```text
     <module>/i18n/ar.po
     ```
   - This includes Python, XML, QWeb, website templates, field labels, selection labels, server-side messages, reports, mail templates, and other strings handled through Odoo translation extraction.

3. **Do not use `_t()` for new JavaScript localization under this workflow.**
   - Existing `_t()` usage must be investigated before changing it.
   - Do not perform repository-wide `_t()` replacement unless explicitly requested.

The core operating principle is:

> Investigate first. Understand the existing implementation. Make the smallest safe localization changes. Validate before finishing.

# 0. Native Agent Compatibility, Task Mode, and Guidance Scope

this guidance adds Plemo-specific Odoo localization rules and safeguards. It does not replace the agent's native repository discovery, planning, implementation, or task-mode behavior.

Reuse reliable evidence already collected during the current task. Do not repeat previously collected investigation evidence unless the existing evidence is insufficient, stale, or does not cover the localization question.

Repository-specific instructions such as `plemo.md`, configured addon paths, and reliable native discovery tools take precedence over generic repository-layout examples in this guidance.

Determine the current task mode before making any repository change:

```text
Investigation / Review / Audit
Implementation / Fix
Validation
```

For **Investigation / Review / Audit**:

- remain read-only;
- do not create `i18n/`, `ar.po`, `localization.js`, or any other file;
- do not edit translations, JavaScript, XML, Python, manifests, assets, or styles;
- report missing localization infrastructure and localization issues only.

For **Implementation / Fix**:

- localization changes are allowed within the user's requested scope;
- perform the investigation phase first;
- the user's existing add/fix/build/localize request counts as implementation authorization;
- do not ask for a second approval solely because this guidance performed an investigation;
- ask only when the investigation reveals a material scope change, destructive operation, or unresolved decision that would make proceeding unsafe.

For **Validation**:

- validate the current implementation;
- remain read-only unless the user explicitly asked for fixes as part of validation.

Use the smallest applicable portion of this guidance for the task. Do not force a full localization audit/report for a trivial isolated correction unless the user explicitly requests one.

## 0.1 Plemo JavaScript Localization Policy Scope

The `localize("English", "Arabic")` helper architecture in this guidance is a Plemo/project localization policy.

Apply it only when the target module/workflow falls within this localization policy.

Do not treat the prohibition on new JavaScript `_t()` usage as a universal repository-wide Odoo rule for unrelated modules or projects unless their localization policy explicitly adopts this architecture.

Existing valid Odoo-native `_t()` usage outside the target workflow must not be rewritten merely to satisfy this guidance.

---

# 1. Mandatory Startup Procedure

Follow this order whenever this guidance is used:

1. Determine the target module.
2. Locate and verify the module directory.
3. Detect the Odoo version.
4. Inspect the module architecture.
5. Inspect the localization architecture.
6. Check for `i18n/`.
7. Check for `i18n/ar.po`.
8. Inspect JavaScript files.
9. Check for `static/src/js/utils/localization.js`.
10. Produce an investigation summary before substantial modifications.
11. Apply localization changes only after the investigation.
12. Validate all modified files.
13. Review the final diff.
14. Produce a final report.

---

# 2. Resolve the Target Module

If the target module is already provided or clearly established, use it and verify its path.

If the module is not explicitly named:

1. attempt to resolve it from repository evidence and current context;
2. use repository-specific guidance such as `plemo.md` when available;
3. use native addon/code discovery tools when available;
4. search by feature, model, field, XML ID, template, route, visible text, JavaScript selector, or other strong evidence;
5. verify the owning/custom module before editing anything;
6. ask the user only if multiple plausible module targets remain and choosing incorrectly would materially change the implementation.

Do not stop merely because the user described a feature or page instead of naming its module.

Do not modify files until the target module is resolved with sufficient confidence.

Record:

```text
Target module:
Module path:
Resolution evidence:
```

---

# 3. Locate and Verify the Module

Find the exact directory for the requested module.

Verify that it is an Odoo module by inspecting evidence such as:

```text
__manifest__.py
__init__.py
models/
controllers/
views/
static/
data/
report/
wizard/
i18n/
```

Not every directory has to exist.

Do not treat a directory as an Odoo module based only on its name.

If more than one directory matches the module name, investigate which one belongs to the active project before changing anything.

---

# 4. Detect the Odoo Version

Detect the actual Odoo version before using version-specific frontend APIs or changing localization infrastructure.

Inspect the strongest evidence available, such as:

```text
target/custom module __manifest__.py version prefix
odoo/release.py
odoo/__init__.py
odoo/version.py
version_info
Git branch
repository structure
Docker configuration
requirements files
project configuration
existing JavaScript imports
existing frontend APIs
```

A valid module manifest version prefix is acceptable primary evidence when the Odoo core source is not present in the workspace.

Examples:

```text
17.0.1.0.0 -> Odoo 17
18.0.2.1.0 -> Odoo 18
19.0.1.0.0 -> Odoo 19
```

When Odoo core source is available, use it as stronger confirmation where practical.

Do not reject valid manifest evidence merely because `odoo/release.py` is unavailable.

Record:

```text
Detected Odoo version:
Evidence:
Frontend architecture:
```

Example:

```text
Detected Odoo version: 18.0
Evidence: odoo/release.py
Frontend architecture: modern Odoo JavaScript modules
```

Do not rely only on:

- repository names;
- folder names;
- database names;
- comments;
- copied module code;
- assumptions from another project.

If the version cannot be established confidently, report the uncertainty before introducing version-specific code.

---

# 5. Investigate Before Modifying

When repository-native discovery tools are available, prefer them over broad shell scans. Examples may include `find_addon`, `search_code`, targeted `read_file`, or equivalent safe tools. Shell commands and paths shown in this guidance are examples, not mandatory tooling.

Before editing localization code or translation files, inspect the target module.

At minimum investigate relevant paths such as:

```text
<module>/__manifest__.py

<module>/i18n/
<module>/i18n/ar.po

<module>/static/src/js/
<module>/static/src/js/utils/localization.js
<module>/static/src/xml/

<module>/views/
<module>/templates/
<module>/models/
<module>/controllers/
<module>/wizard/
<module>/report/
<module>/data/
```

Also inspect any other files that contain user-visible content.

Determine:

- whether `i18n/` exists;
- whether `ar.po` exists;
- whether `localization.js` exists;
- how JavaScript currently detects language;
- whether JavaScript uses `localize()`;
- whether JavaScript uses `_t()`;
- whether hardcoded Arabic exists;
- whether hardcoded English user-facing text exists;
- whether Python strings use Odoo translation correctly;
- whether XML/QWeb strings are extractable;
- whether source strings already exist in `ar.po`;
- whether several translation approaches are mixed;
- whether existing Arabic translations are valid;
- whether another module actually owns a problematic string.

Do not make changes during the initial investigation stage.

---

# 6. Required Localization Architecture

Use two localization paths.

## 6.1 JavaScript

JavaScript user-facing text must use:

```javascript
localize("English text", "Arabic text")
```

through:

```text
<module>/static/src/js/utils/localization.js
```

Example:

```javascript
const messages = {
    invalidEmail: localize(
        "Please enter a valid email address.",
        "اكتب بريد إلكتروني صحيح."
    ),
    invalidPhone: localize(
        "Please enter a valid phone number.",
        "اكتب رقم تليفون صحيح."
    ),
    missingEmail: localize(
        "Please enter your email address.",
        "اكتب بريدك الإلكتروني."
    ),
    missingName: localize(
        "Please enter your name.",
        "اكتب اسمك."
    ),
};
```

Do not use `_t()` for new JavaScript localization under this workflow.

## 6.2 Standard Odoo Translation

Use:

```text
<module>/i18n/ar.po
```

for standard Odoo-translatable content, including:

- Python user-facing strings;
- XML views;
- QWeb templates;
- website templates;
- field labels;
- help text;
- selection labels;
- menu labels;
- buttons defined in XML;
- server-side validation messages;
- `UserError` / `ValidationError` text;
- controller messages returned to users;
- reports;
- mail templates;
- other Odoo-extracted strings.

---

# 7. Check the `i18n` Directory

The standard directory must be:

```text
<module>/i18n/
```

Use lowercase:

```text
i18n
```

Do not create alternatives such as:

```text
I18n
I18N
translations
locale
```

unless the project separately requires them for another purpose.

---

# 8. If `i18n/` Does Not Exist

If:

```text
<module>/i18n/
```

does not exist:

For **Investigation / Review / Audit / read-only Validation**:

1. do not create the directory;
2. report that localization infrastructure is missing;
3. check whether an Arabic PO export already exists elsewhere in the repository or was provided by the user.

For **Implementation / Fix**:

1. create:
   ```text
   <module>/i18n/
   ```
2. check whether an Arabic PO export has already been provided elsewhere in the repository or by the user;
3. do not invent a translation catalog manually.

Creating the directory does not authorize creating `ar.po` from scratch.

---

# 9. If `ar.po` Does Not Exist

This is a hard stop condition for PO translation.

If:

```text
<module>/i18n/ar.po
```

does not exist:

1. If the current task is read-only, do not create `i18n/`; only report that it is missing.
2. If the current task is an authorized Implementation / Fix, create `i18n/` if necessary.
3. Do **not** create `ar.po` from scratch.
4. Do **not** manually generate PO entries.
5. Do **not** guess source references.
6. Do **not** reconstruct a complete translation catalog from source files.
7. Do **not** copy another module's PO file.
8. Do **not** copy an old unrelated PO export and treat it as authoritative.
9. Ask the user to export the Arabic translation for the target module from Odoo.

Tell the user to place the exported file at:

```text
<module>/i18n/ar.po
```

Use a response similar to:

```text
The module does not currently contain i18n/ar.po.

I will not generate the Arabic PO catalog manually. The localization
workflow should start from the Arabic translation catalog exported by
Odoo for this module.

Please export the Arabic translation from Odoo and place the exported file at:

<module>/i18n/ar.po

Once the exported file is available, I can continue the localization work.
```

Do not continue PO translation until the exported file is available.

You may still report JavaScript localization issues found during investigation, but do not claim that localization is complete.

---

# 10. If `ar.po` Exists

Read and audit it before editing.

Inspect:

- PO header;
- language metadata;
- module references;
- active entries;
- translated entries;
- empty `msgstr`;
- fuzzy entries;
- obsolete entries;
- duplicate active entries;
- multiline entries;
- placeholders;
- plural forms;
- HTML fragments;
- XML fragments;
- Python references;
- JavaScript references if present;
- QWeb/XML references;
- entries that reference unexpected modules;
- malformed syntax.

Do not regenerate the entire PO file merely because translations are missing.

Do not replace the existing file with another export without comparing them first.

Protect valid existing Arabic translations.

---

# 11. Files and Content to Investigate

## 11.1 Python `.py`

Inspect user-visible strings such as:

- `ValidationError`;
- `UserError`;
- warnings;
- notifications;
- controller responses;
- wizard messages;
- report text;
- field labels;
- field help text;
- selection labels;
- status messages displayed to users.

Use the Odoo-native translation mechanism supported by the detected version and store Arabic translations in `i18n/ar.po`.

Do not translate technical strings unless they are displayed to users.

Normally do not translate:

- model names;
- database field names;
- route names;
- API keys;
- technical identifiers;
- SQL;
- internal-only log text.

## 11.2 XML / QWeb / Website Templates

Inspect:

- form views;
- list/tree views;
- kanban views;
- search views;
- website pages;
- QWeb templates;
- OWL XML templates;
- menu labels;
- action labels;
- button text;
- headings;
- descriptions;
- labels;
- placeholders;
- title attributes;
- alt text;
- other user-visible attributes.

Keep the source language in the template and store Arabic in `ar.po`.

Do not create Arabic/English template branches just to translate normal text.

Bad:

```xml
<t t-if="is_arabic">
    النص العربي
</t>
<t t-else="">
    English text
</t>
```

Use language-specific branches only when actual structure or behavior must differ.

## 11.3 JavaScript `.js`

Inspect for user-visible text including:

- validation messages;
- errors shown to users;
- warnings;
- notifications;
- toast messages;
- modal titles;
- modal descriptions;
- confirmation messages;
- buttons generated in JS;
- empty-state messages;
- loading text;
- dynamic labels;
- checkout/cart/booking messages;
- API error fallbacks shown to users;
- placeholders assigned by JS;
- `textContent`;
- `innerText`;
- user-visible `innerHTML`;
- user-visible template literals;
- DOM-generated labels.

Localize these using `localize("English", "Arabic")`.

Do not translate technical strings such as:

```text
GET
POST
PUT
DELETE
application/json
product_id
event_id
checkout-form
.ticket-card
#booking-modal
/api/cart
click
change
submit
```

Do not translate debug/log-only messages unless the same text reaches the user.

## 11.4 CSS / SCSS

Do not translate CSS or SCSS strings as localization content.

Inspect styles only when Arabic requires:

- RTL behavior;
- alignment changes;
- spacing corrections;
- icon-direction changes;
- Arabic typography/layout fixes.

Do not change styling unless required by the localization task.

## 11.5 JSON / Data Files

Inspect values only if they are displayed directly to users.

Do not translate:

- IDs;
- technical keys;
- API parameters;
- configuration identifiers.

## 11.6 Images

Do not edit images automatically.

If an image contains visible English text that should also exist in Arabic, report it separately.

Only modify the image when explicitly requested.

---

# 12. JavaScript Localization Helper Path

The required helper location is exactly:

```text
<module>/static/src/js/utils/localization.js
```

Inside the module, the path must therefore be:

```text
/static/src/js/utils/localization.js
```

Do not place the helper in arbitrary locations unless the user explicitly changes the architecture.

---

# 13. If `localization.js` Already Exists

Read the entire file before modifying it.

Understand:

- imports;
- Odoo APIs used;
- language sources;
- language normalization;
- fallbacks;
- public-page behavior;
- exported functions.

The preferred stable API is:

```javascript
getCurrentLang()
isArabicLang()
localize(enText, arText)
```

If the existing helper provides this behavior and is valid for the detected Odoo version, preserve it.

Do not replace working code only to standardize formatting.

Check whether the helper's Odoo-specific imports exist in the detected version.

---

# 14. Reference Helper Behavior

The helper should conceptually:

1. normalize language values;
2. collect reliable current-language candidates;
3. prefer Odoo/page language information over browser language;
4. safely detect Arabic;
5. expose a stable API for the rest of the module.

Conceptual structure:

```javascript
function normalizeLang(lang) {
    // Normalize language representation.
}

function getLanguageCandidates() {
    // Collect verified Odoo/page/browser language sources.
}

export function getCurrentLang() {
    // Return the best current language value.
}

export function isArabicLang() {
    // Return true when the active language is Arabic.
}

export function localize(enText, arText) {
    return isArabicLang() ? arText : enText;
}
```

The exact imports and internal language APIs may differ between Odoo versions.

---

# 15. How to Create `localization.js` When Missing

If:

```text
<module>/static/src/js/utils/localization.js
```

does not exist, do not immediately create it.

First determine whether the module contains JavaScript user-facing strings that require Arabic localization.

Examples:

- validation messages;
- modal messages;
- user-facing errors;
- notifications;
- warnings;
- buttons created in JS;
- checkout/cart/booking messages;
- labels inserted into the DOM;
- dynamic placeholders;
- loading/empty-state text.

If JavaScript localization is not required, do not create the helper unnecessarily.

If JavaScript localization is required, follow the procedure below.

---

# 16. Create the Required Directory Structure

Check whether these directories exist:

```text
<module>/static/
<module>/static/src/
<module>/static/src/js/
<module>/static/src/js/utils/
```

Create only the missing directories necessary to create:

```text
<module>/static/src/js/utils/localization.js
```

Do not move existing files or reorganize the module unnecessarily.

---

# 17. Detect the JavaScript Architecture Before Creating the Helper

Inspect the Odoo version and target module's existing JS files.

Determine whether the module uses:

- legacy Odoo JavaScript;
- `@odoo-module`;
- modern ES imports;
- OWL;
- public website JavaScript;
- backend web-client JavaScript;
- mixed architecture.

Follow the architecture already appropriate for that Odoo version and module.

Do not upgrade the entire frontend architecture merely to introduce the helper.

---

# 18. Determine Available Language APIs

Before writing `localization.js`, investigate how the detected Odoo version exposes the active language.

Search the actual repository/Odoo source for language-related APIs.

Potential sources include:

- Odoo localization service;
- Odoo session information;
- user context;
- frontend environment;
- HTML `lang`;
- reliable project-specific data attributes;
- browser language as a final fallback.

Use only APIs that actually exist.

Do not invent imports.

Do not copy an import from another version without verifying it.

---

# 19. Preferred Language Source Priority

When several sources are available, prefer approximately:

1. active Odoo localization/language service;
2. Odoo session or user-context language;
3. active page `<html lang="...">`;
4. reliable project-specific language attributes;
5. browser language as the final fallback.

The active Odoo/page language must take priority over browser preference.

Example:

```text
Browser language: en-US
Current Odoo website: Arabic
```

Expected:

```javascript
isArabicLang() === true
```

---

# 20. Required Stable Helper API

Regardless of Odoo version, the helper should expose:

```javascript
getCurrentLang()
isArabicLang()
localize(enText, arText)
```

The architecture should remain:

```text
Odoo-version-specific language detection
        ↓
localization.js
        ↓
stable helper API
        ↓
module JavaScript
```

Other module files should not need to understand Odoo's version-specific language APIs.

---

# 21. Required `localize()` Behavior

The helper must provide behavior equivalent to:

```javascript
export function localize(enText, arText) {
    return isArabicLang() ? arText : enText;
}
```

Do not store application-specific message translations inside the helper.

Bad:

```javascript
export function getCheckoutText() {
    return isArabicLang() ? "الدفع" : "Checkout";
}
```

Good:

```javascript
export function localize(enText, arText) {
    return isArabicLang() ? arText : enText;
}
```

Then consuming code uses:

```javascript
localize("Checkout", "الدفع")
```

---

# 22. Required Language Normalization

Create a normalization function that safely handles:

- `null`;
- `undefined`;
- empty strings;
- hyphens;
- underscores;
- uppercase/lowercase differences.

A suitable conceptual implementation is:

```javascript
function normalizeLang(lang) {
    return String(lang || "")
        .trim()
        .replace(/-/g, "_")
        .toLowerCase();
}
```

Example:

```text
ar-EG → ar_eg
```

Adjust only when the project requires another normalization convention.

---

# 23. Arabic Language Detection

Do not restrict detection to only one code unless the project explicitly requires that.

Arabic may appear as:

```text
ar
ar_001
ar-EG
ar_EG
```

After normalization, a reasonable detection approach is:

```javascript
normalizedLang === "ar" || normalizedLang.startsWith("ar_")
```

Use a narrower rule if the project has a verified reason.

Do not classify unrelated language codes as Arabic.

---

# 24. Collect Language Candidates Safely

Use a function conceptually similar to:

```javascript
function getLanguageCandidates() {
    const candidates = [];

    // Verified Odoo language source.
    // Verified session language source.
    // HTML/page language.
    // Verified project-specific attributes.
    // Browser language fallback.

    return candidates.filter(Boolean);
}
```

Optional/global language sources must be accessed safely.

A missing Odoo global must not crash a public website page.

---

# 25. Safe Global Access

Some public pages may not expose all backend globals.

Protect optional accesses appropriately.

Conceptual example:

```javascript
try {
    candidates.push(odoo?.session_info?.user_context?.lang);
} catch {
    // Continue with other language sources.
}
```

Use syntax compatible with the detected Odoo/browser target.

Do not let language detection break the page.

---

# 26. Modern Odoo Helper Template

If investigation confirms that the current Odoo version supports:

```javascript
import { localization } from "@web/core/l10n/localization";
```

and the target module uses compatible modern Odoo JS modules, the helper may use a structure like:

```javascript
/** @odoo-module **/

import { localization } from "@web/core/l10n/localization";

function normalizeLang(lang) {
    return String(lang || "")
        .trim()
        .replace(/-/g, "_")
        .toLowerCase();
}

function getLanguageCandidates() {
    const candidates = [];

    try {
        candidates.push(localization?.code);
    } catch {
        // Continue with fallback sources.
    }

    try {
        candidates.push(odoo?.session_info?.user_context?.lang);
    } catch {
        // Odoo globals may not exist on every public page.
    }

    candidates.push(document.documentElement?.getAttribute("lang"));
    candidates.push(document.querySelector("[data-lang]")?.dataset.lang);
    candidates.push(window.navigator?.language);

    return candidates.filter(Boolean);
}

export function getCurrentLang() {
    return getLanguageCandidates()[0] || "";
}

export function isArabicLang() {
    return getLanguageCandidates().some((lang) => {
        const normalizedLang = normalizeLang(lang);
        return normalizedLang === "ar" || normalizedLang.startsWith("ar_");
    });
}

export function localize(enText, arText) {
    return isArabicLang() ? arText : enText;
}
```

This is a template, not a mandatory implementation.

Before using:

```javascript
import { localization } from "@web/core/l10n/localization";
```

verify that:

- the module path exists;
- the exported symbol exists;
- the target Odoo version supports it;
- the relevant asset bundle can resolve it.

---

# 27. Project-Specific Language Sources

A project may expose reliable attributes such as:

```text
data-lang
data-ac-checkout-lang
data-website-lang
```

or another project-specific language attribute.

Only include such a source after verifying:

1. it exists in the current project;
2. it is populated reliably;
3. it represents the active Odoo/page language.

Do not copy project-specific selectors from another module blindly.

---

# 28. Older Odoo Versions

If a modern import such as:

```javascript
@web/core/l10n/localization
```

does not exist in the detected version:

Do not use it.

Investigate the actual language/session mechanisms available in that version.

Build the helper around verified APIs while preserving the stable public API:

```javascript
getCurrentLang()
isArabicLang()
localize(enText, arText)
```

Do not migrate the whole module to newer Odoo frontend architecture solely for localization.

---

# 29. Do Not Guess Imports

Before adding any Odoo import to `localization.js`:

1. search the actual Odoo codebase;
2. confirm the module exists;
3. confirm the exported symbol exists;
4. confirm it is suitable for the target frontend context;
5. confirm its asset bundle can resolve it.

If verification fails, do not use that import.

Use another verified language source.

---

# 30. Check Asset Configuration

After creating `localization.js`, inspect:

```text
<module>/__manifest__.py
```

and any relevant asset configuration.

Determine whether the helper is already included through a glob such as:

```text
static/src/js/**/*.js
```

If already covered, do not add a duplicate asset entry.

If it is not covered and explicit registration is required, add only the minimum necessary asset entry.

Do not modify unrelated bundles.

Do not load the helper globally when it is needed only by one frontend bundle.

---

# 31. Import the Helper Correctly

When another JS file needs `localize()`, use the import style compatible with the detected module architecture.

Example only:

```javascript
import { localize } from "./utils/localization";
```

Do not assume the same relative path is correct for every file.

Calculate the actual path based on the source file location.

Example:

```text
Source:
static/src/js/booking_form.js

Helper:
static/src/js/utils/localization.js
```

Possible relative import:

```javascript
./utils/localization
```

For a deeper source file, the path will differ.

Always verify it.

---

# 32. Do Not Duplicate Language Detection

After the helper exists, do not reproduce language checks throughout module files.

Bad:

```javascript
const isArabic = document.documentElement.lang === "ar";
```

Bad:

```javascript
const isArabic =
    odoo.session_info.user_context.lang === "ar_001";
```

Good:

```javascript
import { isArabicLang, localize } from "./utils/localization";
```

Use:

```javascript
isArabicLang()
```

for behavior that genuinely differs by language.

Use:

```javascript
localize("English", "Arabic")
```

for text localization.

---

# 33. Convert JavaScript User-Facing Strings

Before:

```javascript
const messages = {
    invalidEmail: "Please enter a valid email address.",
    invalidPhone: "Please enter a valid phone number.",
    missingEmail: "Please enter your email address.",
    missingName: "Please enter your name.",
};
```

After:

```javascript
const messages = {
    invalidEmail: localize(
        "Please enter a valid email address.",
        "اكتب بريد إلكتروني صحيح."
    ),
    invalidPhone: localize(
        "Please enter a valid phone number.",
        "اكتب رقم تليفون صحيح."
    ),
    missingEmail: localize(
        "Please enter your email address.",
        "اكتب بريدك الإلكتروني."
    ),
    missingName: localize(
        "Please enter your name.",
        "اكتب اسمك."
    ),
};
```

Keep English as the first argument and Arabic as the second.

---

# 34. Existing `_t()` in JavaScript

If existing JS contains `_t()`:

1. inspect where and why it is used;
2. determine whether it is part of a currently working architecture;
3. identify whether it belongs to the requested module;
4. report it;
5. only migrate it if the current localization task requires migration.

Do not blindly replace `_t()` repository-wide.

For new JS localization under this guidance, use `localize()`.

---

# 35. Avoid Manual Language Conditions for Text

Bad:

```javascript
if (lang === "ar") {
    message = "تم تأكيد الحجز";
} else {
    message = "Booking confirmed";
}
```

Bad:

```javascript
const message =
    document.documentElement.lang === "ar"
        ? "تم تأكيد الحجز"
        : "Booking confirmed";
```

Good:

```javascript
const message = localize(
    "Booking confirmed",
    "تم تأكيد الحجز"
);
```

Centralize language detection inside `localization.js`.

---

# 36. Avoid Double Translation

Trace the ownership of a string before localizing it.

Example:

```javascript
const message = response.message;
```

If `response.message` was already translated server-side by Python/Odoo, do not wrap it again with `localize()`.

Use:

```text
JS-owned text → localize()
Python/XML/Odoo-owned text → ar.po
```

Do not maintain two competing translations for the same source.

---

# 37. PO Translation Rule

When translating an existing PO entry:

Modify:

```text
msgstr
```

Do not modify:

```text
msgid
```

unless the user explicitly requests an English source change.

The `msgid` is the source translation key and must remain stable during translation-only work.

---

# 38. Critical Multiline PO Formatting Rule

When `msgid` is multiline, preserve equivalent multiline structure in `msgstr`.

Example source:

```po
msgid ""
"At Aqua City, the fun goes beyond water adventures. Families can also enjoy\n"
"                                            our\n"
"                                            unique Mini Zoo, where kids and adults can explore and interact with a\n"
"                                            variety\n"
"                                            of friendly animals in a safe and enjoyable environment."
```

Required translation style:

```po
msgstr ""
"في أكوا سيتي، المتعة مش بس في الألعاب المائية. العائلات كمان تقدر تستمتع\n"
"                                            بحديقة\n"
"                                            الحيوانات الصغيرة المميزة عندنا، ويتفاعل الأطفال والكبار مع\n"
"                                            أنواع مختلفة\n"
"                                            من الحيوانات الأليفة في مكان آمن وممتع."
```

Preserve:

- `msgstr ""`;
- continuation lines;
- `\n`;
- intentional leading spaces;
- indentation inside quoted strings;
- overall entry structure.

Do not collapse the translation into:

```po
msgstr "..."
```

when the source uses a multiline structure.

Do not reformat multiline entries for readability.

---

# 39. Strict PO Structural Preservation

When editing an entry, preserve:

- `msgid`;
- `msgctxt` if present;
- comments;
- source references;
- flags unless intentionally fixing them;
- multiline formatting;
- continuation lines;
- `\n`;
- escaped characters;
- intentional whitespace;
- indentation;
- placeholders;
- HTML;
- XML;
- format specifiers;
- variables.

Do not reformat unrelated entries.

Do not run a formatter that rewrites the whole PO file unnecessarily.

---

# 40. Placeholder Preservation

Never remove or alter placeholders.

Examples:

```text
%s
%d
%(name)s
{}
{0}
{name}
${name}
{{ value }}
```

Example:

```po
msgid "Hello %s"
msgstr "أهلًا %s"
```

Preserve the placeholder exactly.

Do not translate placeholder names.

Do not change the number of placeholders.

Do not introduce new placeholders.

---

# 41. HTML and XML Inside PO Entries

Translate visible text while preserving valid markup.

Example:

```html
<strong>Book now</strong>
```

may become:

```html
<strong>احجز دلوقتي</strong>
```

Do not accidentally modify:

- tag names;
- closing tags;
- IDs;
- classes;
- technical attributes;
- QWeb directives;
- variable expressions.

---

# 42. Existing Translation Protection

Never overwrite a valid Arabic translation with an empty translation.

Example:

Existing:

```po
msgstr "ترجمة صحيحة"
```

Incoming:

```po
msgstr ""
```

Keep the valid translation unless there is a confirmed reason to remove it.

When comparing PO files, classify entries such as:

```text
Both translated and identical
Both translated and different
Existing translated / incoming empty
Existing empty / incoming translated
Only in existing
Only in incoming
```

Do not blindly replace one file with another.

---

# 43. Fuzzy Entries

Inspect entries marked:

```po
#, fuzzy
```

Do not assume the translation is valid.

Review:

- current `msgid`;
- current `msgstr`;
- previous source if available;
- actual UI context.

If the translation is correct for the current source, update it and remove the fuzzy flag only when appropriate.

Do not bulk-remove fuzzy flags without reviewing the entries.

---

# 44. Obsolete Entries

Entries beginning with:

```po
#~
```

are obsolete.

Do not count them as active untranslated entries.

Do not reactivate them without evidence that the source string is active again.

---

# 45. Duplicate Entries

Detect genuine duplicate active entries carefully.

Consider:

- `msgctxt`;
- references;
- plural forms;
- module ownership.

Do not remove entries simply because the English text appears more than once.

---

# 46. Plural Entries

Preserve plural structures such as:

```po
msgid "Ticket"
msgid_plural "Tickets"
```

and all associated plural translation fields.

Do not collapse plural entries into a simple `msgstr`.

---

# 47. Arabic Translation Style

First inspect the module's existing valid Arabic translations.

Use them as the primary terminology and tone reference.

Maintain consistency.

When there is no established project style, use Arabic that is:

- clear;
- natural;
- easy to understand;
- professional;
- customer-friendly;
- not unnecessarily formal;
- not filled with difficult vocabulary;
- not literal when literal wording sounds unnatural.

When suitable for the project, prefer Egyptian-friendly Arabic for customer-facing website text.

Example:

```text
Please enter your name.
```

Preferred:

```text
اكتب اسمك.
```

Avoid unnecessarily formal phrasing when the project's existing style is simple and conversational.

---

# 48. Terminology Consistency

Before translating many strings, inspect recurring terms such as:

```text
Booking
Ticket
Checkout
Cart
Payment
Order
Event
Subscription
Station
Passenger
Confirmation
Cancel
Continue
Back
```

Reuse established translations when they are correct.

Do not translate the same business concept differently across screens without a reason.

---

# 49. Brand Names

Do not translate brand names blindly.

Examples:

```text
Aqua City
Odoo
Visa
Mastercard
Paymob
Fawry
```

Check whether the project already has an established Arabic rendering.

Preserve or transliterate consistently.

---

# 50. Hardcoded Arabic Audit

Search relevant source files for Arabic characters.

Classify each occurrence.

For JavaScript, user-facing Arabic should normally appear as the second argument of:

```javascript
localize("English", "Arabic")
```

Flag standalone Arabic such as:

```javascript
message = "النص العربي";
```

when it bypasses the helper.

For Python/XML, normal translations should usually use English source text plus `ar.po`.

Do not remove Arabic automatically before understanding its context.

---

# 51. Hardcoded English Audit

Search relevant:

```text
.py
.js
.xml
```

and template files for user-visible English.

Classify discovered strings as:

```text
Already localizable
Needs localization
Technical / do not translate
Already translated elsewhere
Uncertain
```

Do not assume every English string needs translation.

---

# 52. Do Not Modify Business Logic

Localization work must not change unrelated behavior.

Do not modify:

- pricing;
- payment logic;
- ticket rules;
- booking rules;
- database behavior;
- security;
- access rules;
- routes;
- record creation;
- API contracts;
- business workflows.

Only make minimal logic changes when genuinely required for localization or language detection.

---

# 53. Scope Control

Only modify the target module unless the user explicitly expands the scope.

You may inspect dependencies to understand a problem.

Do not edit dependency modules without permission.

If another module owns the actual source string or localization issue, report it instead of silently editing that module.

---

# 54. Do Not Change English Source Unnecessarily

Localization is not automatically an English copywriting task.

Do not modify English source text merely to make translation easier.

Example:

```text
Continue
```

must not become:

```text
Continue to the next step
```

unless the user explicitly requests English copy changes.

---

# 55. Do Not Commit or Push

Unless explicitly requested, do not run:

```text
git commit
git push
merge
rebase
force push
create PR
```

Modifying files and running validation are allowed within the requested scope.

Publishing Git changes requires explicit instruction.

---

# 56. Avoid Unnecessary Backup Files

Do not create unnecessary repository files such as:

```text
ar.po.bak
ar.po.old
ar-copy.po
temporary translation files
random backup directories
```

Use temporary system locations for validation output when possible.

---

# 57. PO Validation

After modifying `ar.po`, validate:

- syntax;
- quoting;
- escapes;
- malformed entries;
- duplicate active entries;
- placeholder mismatches;
- multiline formatting;
- HTML/XML integrity where applicable.

If `msgfmt` is available, use an appropriate command such as:

```bash
msgfmt --check <module>/i18n/ar.po -o /tmp/ar.mo
```

Do not leave generated `.mo` files in the repository unless explicitly required.

---

# 58. JavaScript Validation

After modifying JavaScript, check:

- syntax;
- imports;
- relative paths;
- unused imports;
- helper exports;
- `localize()` usage;
- asset inclusion;
- Odoo-version compatibility;
- public-page safety;
- language-detection behavior.

Run relevant lint/test commands when available and safe.

Do not run unrelated or destructive commands.

---

# 59. XML Validation

After modifying XML, validate:

- well-formed XML;
- opening and closing tags;
- escaped characters;
- attributes;
- QWeb directives;
- XPath expressions;
- template IDs;
- inherited views.

Do not restructure XML solely for translation.

---

# 60. Validate a Newly Created `localization.js`

After creating the helper, verify:

1. It exists at:
   ```text
   <module>/static/src/js/utils/localization.js
   ```
2. JavaScript syntax is valid.
3. Every Odoo import actually exists in the detected version.
4. It exports:
   ```text
   getCurrentLang
   isArabicLang
   localize
   ```
5. English mode returns English.
6. Arabic mode returns Arabic.
7. Missing optional Odoo globals do not crash public pages.
8. Browser language does not incorrectly override the active Odoo/page language.
9. Imports in consuming files resolve correctly.
10. The helper is included in the correct asset bundle.
11. There are no duplicate asset entries.
12. No unrelated source code was changed.

---

# 61. Runtime Validation When Available

If a runnable Odoo environment is available, verify representative behavior.

Example:

English page:

```javascript
localize("Continue", "متابعة")
```

Expected:

```text
Continue
```

Arabic page:

```javascript
localize("Continue", "متابعة")
```

Expected:

```text
متابعة
```

Also check:

```javascript
getCurrentLang()
isArabicLang()
```

Verify public website pages when the helper is used publicly because available Odoo globals may differ from backend pages.

Do not perform destructive or irreversible business actions just to test translation.

---

# 62. Failure Handling for Helper Creation

If a compatible current-language API cannot be identified confidently:

Do not invent one.

Report:

- detected Odoo version;
- detected JS architecture;
- APIs investigated;
- why compatibility remains uncertain.

Stop helper implementation until a verified language source is available.

A helper that can crash at runtime is worse than leaving existing JS untouched.

---

# 63. Post-Localization Audit

After modifications, audit the module again.

Report:

```text
Python user-facing strings
XML/QWeb user-facing strings
JavaScript user-facing strings
PO translated entries
PO empty entries
PO fuzzy entries
PO duplicates
Hardcoded Arabic findings
Hardcoded English findings
Potential untranslated strings
```

Do not claim 100% localization when unresolved issues remain.

---

# 64. Initial Investigation Report

Before substantial changes, provide a concise report:

```text
Localization Investigation

Module:
Module path:
Odoo version:
Version evidence:
Frontend architecture:
Arabic PO:
Localization helper:

Current architecture:
- JavaScript:
- Python:
- XML/QWeb:

Issues found:
1.
2.
3.

Planned changes:
1.
2.
3.
```

For very small tasks, this report may be shorter.

---

# 65. Missing `ar.po` Report

If `ar.po` is missing, clearly state:

```text
The module does not currently contain i18n/ar.po.

The i18n directory was confirmed or, for an authorized implementation task,
created when necessary. In a read-only review, no directory was created.
I will not generate the Arabic translation catalog manually.

Please export the Arabic translation for this module from Odoo and place
the exported file at:

<module>/i18n/ar.po

Once the exported PO is available, localization can continue safely.
```

This is a stop condition for PO translation, not merely a warning.

---

# 66. `localization.js` Creation Report

If the helper was created, include:

```text
JavaScript Localization Helper

Created:
<module>/static/src/js/utils/localization.js

Detected Odoo version:
...

Language sources used:
1.
2.
3.

Exports:
- getCurrentLang()
- isArabicLang()
- localize(enText, arText)

Asset configuration:
Already covered / updated

JS files using helper:
- ...

Validation:
- syntax:
- imports:
- English detection:
- Arabic detection:
- public-page fallback:
```

Do not claim runtime validation if only static validation was performed.

---

# 67. Final Report

After completing the localization task, provide:

```text
Localization Result

Module:
Odoo version:

Files changed:
- ...

JavaScript:
- X strings localized with localize()
- localization.js created / reused / updated
- existing _t() findings if relevant

PO:
- X translations added
- X translations updated
- X remaining empty entries
- X fuzzy entries
- X duplicate entries

Validation:
- PO syntax:
- placeholder validation:
- JavaScript validation:
- XML validation:
- runtime validation, if performed:

Remaining issues:
- ...

Scope:
- Confirm which module(s) were modified.
```

Do not claim that unrelated files were untouched unless this was actually verified.

---

# 68. Review the Final Diff

Before finishing, inspect the final changes.

Confirm:

- only intended files changed;
- no `msgid` values were accidentally changed;
- multiline PO structure was preserved;
- valid translations were not erased;
- placeholders still match;
- no unrelated business logic changed;
- `localization.js` imports are correct;
- Arabic text is valid;
- no unnecessary whole-file PO reformatting occurred;
- pre-existing user changes were not reverted.

Distinguish between changes made during this task and changes that existed before the task.

---

# 69. Hard Prohibitions

Never:

- assume the Odoo version;
- start without knowing the target module;
- generate `ar.po` from scratch when it is absent;
- invent PO entries as a substitute for an Odoo export;
- use `_t()` for new JavaScript localization under this workflow;
- convert valid `localize()` calls into `_t()`;
- scatter manual Arabic language checks throughout JavaScript;
- blindly copy `localization.js` from another Odoo version;
- invent Odoo imports;
- translate technical identifiers;
- change `msgid` during translation-only work;
- collapse multiline `msgstr` formatting;
- remove intentional whitespace from PO continuation lines;
- break placeholders;
- overwrite valid Arabic translations with empty values;
- blindly replace an existing PO file;
- modify unrelated modules;
- change unrelated business logic;
- commit or push without explicit permission;
- claim tests passed when they were not run;
- claim complete localization while known untranslated strings remain.

---

# 70. Core Decision Tree

```text
START
  |
  v
Is the module known?
  |
  +-- No --> Attempt repository/context discovery.
  |            |
  |            +-- One confident target --> Continue.
  |            |
  |            +-- Material ambiguity --> Ask the user.
  |
  +-- Yes
        |
        v
Locate and verify the module.
        |
        v
Detect Odoo version.
        |
        v
Inspect module localization architecture.
        |
        v
Check <module>/i18n/
        |
        +-- Missing
        |     |
        |     +-- Read-only task --> Report only; do not create.
        |     |
        |     +-- Implementation/Fix --> Create i18n/ if needed.
        |
        v
Check <module>/i18n/ar.po
        |
        +-- Missing
        |     |
        |     +--> STOP PO localization.
        |          Ask user to export Arabic PO from Odoo.
        |          Do not create ar.po manually.
        |
        +-- Exists
              |
              v
          Audit ar.po.
              |
              v
Inspect Python / XML / QWeb / Website / JS.
              |
              v
Check <module>/static/src/js/utils/localization.js
              |
              +-- Exists
              |     |
              |     +--> Read it.
              |          Verify version compatibility.
              |          Reuse it when valid.
              |
              +-- Missing
                    |
                    v
              Does JS need localization?
                    |
                    +-- No --> Do not create helper.
                    |
                    +-- Yes
                          |
                          v
                    Detect JS architecture.
                          |
                          v
                    Verify language APIs.
                          |
                          v
                    Create version-compatible helper.
                          |
                          v
                    Verify asset inclusion/imports.
              |
              v
Localize JS with localize("English", "Arabic").
              |
              v
Localize standard Odoo content through i18n/ar.po.
              |
              v
Preserve msgid, PO multiline structure, placeholders,
whitespace, HTML/XML, comments, and references.
              |
              v
Validate PO / JS / XML.
              |
              v
Perform post-localization audit.
              |
              v
Review final diff.
              |
              v
Provide final report.
```

---

# 71. Primary Rules to Always Remember

```text
Investigate first.

Resolve the module from context/repository evidence first.
Ask only when material ambiguity remains.

Preserve the agent's task mode: review/audit is read-only; authorized implementation may continue after investigation.

Detect the Odoo version from the strongest actual project evidence, including a valid manifest version prefix when core source is unavailable.

Never assume frontend APIs are the same across Odoo versions.

JavaScript user-facing strings:
localize("English", "Arabic")

Required helper path:
<module>/static/src/js/utils/localization.js

Standard Odoo translations:
<module>/i18n/ar.po

If i18n does not exist:
Create it.

If ar.po does not exist:
Stop PO work.
Ask the user to export Arabic PO from Odoo.
Do not create the catalog from scratch.

If localization.js does not exist:
Create it only when JS localization is needed.
First detect the Odoo version and JS architecture.
Verify language APIs before using them.
Keep the stable API:
getCurrentLang()
isArabicLang()
localize(enText, arText)

Preserve msgid exactly.

Preserve multiline msgstr structure.

Preserve \n values.

Preserve intentional indentation and whitespace.

Preserve placeholders.

Protect valid existing translations.

Do not modify unrelated modules.

Do not modify unrelated business logic.

Make the smallest safe change.

Validate before finishing.

Report exactly what was changed.
```