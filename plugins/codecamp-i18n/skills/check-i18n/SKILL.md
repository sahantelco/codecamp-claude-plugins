---
name: check-i18n
description: Use this skill when the user asks to "check translations", "check i18n", "check localization", "audit UI text", "find hardcoded strings", "missing translation keys", or when you have just added/edited TypeScript or HTML files and want to verify all new UI text is properly translated. Also activate when reviewing a PR or diff that touches component files.
argument-hint: [file-or-directory-path]
allowed-tools: Read, Grep, Glob, Bash
---

# i18n / Translation Audit

This project uses **`@jsverse/transloco`** for localization. Translation keys are the text strings themselves (not dot-notation keys).

## How translation works in this codebase

**In HTML templates:**
```html
{{ 'Some UI text' | transloco }}
[attr.aria-label]="'Some label' | transloco"
[placeholder]="'Enter value' | transloco"
```

**In TypeScript:**
```ts
this.translate.translate('Some UI text')
```

**Registration** — every key must be registered in the site's `app-translations.ts`:
```
client/<site>/src/app/.../app-translations/app-translations.ts
```
Known locations:
- `client/teachers-site/src/app/shared-declarations/app-translations/app-translations.ts`
- `client/arena-site/src/app/_shared-declarations/app-translations/app-translations.ts`
- `client/enterprise-site/src/app/shared-declarations/app-translations/app-translations.ts`
- `client/onboarding/src/app/_shared/app-translations/app-translations.ts`
- `client/parents-site/src/app/_shared-declarations/app-translations/app-translations.ts`

---

## Audit Steps

### Step 1 — Identify files to audit

If `$ARGUMENTS` is provided, treat it as a file path or directory path to audit directly.

Otherwise run:
```
git diff --name-only HEAD
```
Or for staged changes:
```
git diff --name-only --cached
```
Focus on `.ts` and `.html` files under `client/`.

### Step 2 — Detect hardcoded UI text in HTML files

For each `.html` file, look for text that is NOT going through the transloco pipe.

**Patterns that are MISSING translation:**
```html
<!-- Raw text in element content -->
<span>Some text</span>
<p>Some message</p>
<button>Click me</button>
<h2>Some heading</h2>
<!-- Attribute values without transloco pipe -->
[placeholder]="'Enter value'"        <!-- missing | transloco -->
[title]="'Some title'"               <!-- missing | transloco -->
[aria-label]="'label'"               <!-- missing | transloco -->
[ariaLabel]="'label'"                <!-- missing | transloco -->
<!-- Hardcoded attribute strings -->
placeholder="Enter value"
title="Some tooltip"
aria-label="some label"
```

**Patterns that ARE correctly translated:**
```html
{{ 'Some UI text' | transloco }}
[placeholder]="'Enter value' | transloco"
'Text' | transloco
```

### Step 3 — Detect hardcoded UI text in TypeScript files

For each `.ts` file, look for string literals used in UI-facing contexts.

**Patterns that are MISSING translation** (string assigned to UI-facing properties):
```ts
title = 'Some title';
label = 'Click me';
placeholder = 'Enter value';
message = 'Something went wrong';
tooltip = 'Some tooltip';
buttonText = 'Submit';
heading = 'My Heading';
description = 'Some description';
errorMessage = 'Invalid input';
// Object/array entries
{ label: 'Option A' }
{ title: 'Section Name' }
{ text: 'Some text' }
```

**Patterns that ARE correctly translated:**
```ts
this.translate.translate('Some text')
this.translationService.translate('Some text')
```

**Patterns to IGNORE** (not UI text):
- Route paths, CSS class names, identifiers, API URLs
- Strings that are all-lowercase with no spaces and look like keys (e.g. `'someKey'`, `'ERROR_CODE'`)
- `console.log` strings
- Import paths
- `@Component` selector/templateUrl/styleUrls
- RegExp patterns
- Environment-specific strings like `'production'`, `'development'`
- Short single-word technical identifiers

### Step 4 — Check registration in app-translations.ts

For each problematic string found:
1. Determine which site the file belongs to (from its path, e.g. `client/teachers-site/...`)
2. Read the corresponding `app-translations.ts`
3. Check if `this.translate.translate('the text')` is already present

### Step 5 — Report findings

For each issue found, report in this format:

```
⚠️  MISSING TRANSLATION
File: client/teachers-site/src/app/some/component.html (line 42)
Text: "Click me"
Context: <button>Click me</button>

Fix in template:
  <button>{{ 'Click me' | transloco }}</button>

Add to app-translations.ts:
  this.translate.translate('Click me');
```

If a key is used in the template/TS but is NOT registered in `app-translations.ts`:

```
⚠️  MISSING REGISTRATION
File: client/teachers-site/src/app/some/component.html (line 15)
Text: "Some text"
Status: Uses transloco pipe ✓ — but NOT registered in app-translations.ts ✗

Add to client/teachers-site/src/app/shared-declarations/app-translations/app-translations.ts:
  this.translate.translate('Some text');
```

If everything is correct:
```
✅  All UI text in the audited files uses the translation system correctly.
```

### Step 6 — Offer to fix

After reporting, ask the user if they want you to:
1. Apply the transloco pipe/method wrapping to the flagged locations
2. Add the missing keys to `app-translations.ts`

Do not auto-apply fixes — confirm with the user first.

---

## Quick Reference: What counts as "UI text"

UI text is any human-readable string that:
- Will be visible in the browser to end users
- Is used as a label, title, placeholder, tooltip, heading, button text, error message, notification, or descriptive content

Non-UI strings (skip these):
- CSS class names, IDs, selectors
- Route paths (`/dashboard`, `/login`)
- Event names, action types
- Technical identifiers, enum values
- API endpoint paths
- Regex patterns
- Short all-lowercase single-word strings

## ARGUMENTS

$ARGUMENTS

If arguments are provided, treat them as the specific file path(s) or directory to audit. Otherwise audit `git diff --name-only HEAD` against the current branch.
