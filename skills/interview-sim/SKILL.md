---
name: interview-sim
description: >
  Creates a realistic Node.js interview simulation — an existing codebase the candidate receives
  with intentional bugs to fix and/or features to add. Generates Express routes, an in-memory store,
  pre-written Jest + supertest tests, reference solutions, and a company-style README with no hints.
  Use when asked to "create an interview sim", "new interview challenge", or "generate a coding test".
argument-hint: [domain] [scope] [challenges] [duration]
allowed-tools: Read Glob Grep Write Edit Bash
---

# Interview Simulation Generator

You are generating a realistic Node.js coding challenge. The candidate receives an existing codebase with some failing tests and a set of TODO features. The goal is to simulate what a real company hands a candidate on interview day.

## Step 1 — Gather requirements

Ask the user the following questions before generating anything. Use `AskUserQuestion` with all questions in a single call.

Required inputs:
- **Domain** — what the app is about (e.g. e-commerce, task management, auth/users, blog/CMS, booking system, inventory)
- **Scope** — backend-only (Express REST API) or full-stack (Express API + vanilla JS frontend)
- **Challenges** — bugs only, features only, or both bugs and features
- **Duration** — target coding time in minutes (30, 45, 60, or 90)
- **Output path** — where to create the challenge (defaults to a new subfolder in the current directory)

If the user provides some of this in `$ARGUMENTS`, parse and use those values; only ask for what's missing.

---

## Step 2 — Design the simulation (do this before writing any files)

Based on the inputs, design the following. Think through each decision carefully.

### App design

Choose 2–4 resource types appropriate to the domain (e.g. for e-commerce: products, orders, inventory).

For each resource, design:
- The data shape (flat object, required fields, optional fields)
- 3–5 REST endpoints
- How resources relate to each other (e.g. placing an order checks inventory)

Keep the scope proportional to the duration:
- 30 min → 1–2 resources, 4–6 endpoints, 2–3 bugs, 1 feature
- 45 min → 2 resources, 6–8 endpoints, 3–4 bugs, 1–2 features
- 60 min → 2–3 resources, 8–10 endpoints, 4 bugs, 2 features
- 90 min → 3 resources, 10–14 endpoints, 5 bugs, 3 features

### Bug design

Design realistic, non-obvious bugs. Each bug must:
- Be a logic or semantic error, not a syntax error
- Pass a linter without warnings (avoid assignment-inside-filter `order.x = y` patterns — IDE linters auto-fix these; prefer other bug types)
- Cause 1–2 specific tests to fail
- Be fixable with a small, targeted change

**High-quality bug patterns to use:**
- Mutate-before-validate: push to array before checking uniqueness constraint
- Wrong arithmetic operator: `+` instead of `-`, `*` missing (total ignores quantity)
- Wrong field reference: filtering on `item.price` when it should be `item.unitPrice`
- Off-by-one or wrong comparison: `>` instead of `>=`, `<` instead of `<=`
- Logic inversion: checking `!required` when it should be `required`
- Missing condition branch: returning 200 with `null` instead of 404 when record not found
- State not restored: action modifies related data but doesn't undo on rollback/cancel
- Wrong default: a field initialized to `undefined` instead of a specific value

**Avoid:**
- Syntax errors or typos that any editor catches
- Assignment-in-condition patterns (linters auto-fix)
- Obvious wrong types (passing string where number expected)
- Bugs that break more than 3 tests at once (too disruptive)

### Feature design

Each feature should:
- Require adding 1 new endpoint or extending an existing one
- Touch at least 2 files (e.g. a route + the store, or two routes that interact)
- Have a clear, completable spec (stated in README as requirements)
- Have pre-written tests the candidate must make pass

One feature per challenge should have a structural pitfall (e.g. Express route ordering, async without await, shared state issue) to reward careful reading over mechanical typing.

### Test design

For each bug: write 1–2 tests that fail only because of that bug (other tests must still pass).
For each feature: write 3–5 tests that fully cover the expected behavior.

Starting score target:
- ~50–55% passing (the healthy-but-broken feel of a real codebase)

---

## Step 3 — Generate files

Create the following structure. Adjust for scope (omit `public/` if backend-only).

```
<challenge-name>/
  store.js                   in-memory data layer; exports { getStore, resetStore }
  app.js                     Express wiring; exports { app }; require.main guard
  routes/
    <resource1>.js            route handlers WITH bugs embedded (no BUG comments)
    <resource2>.js
    ...
  public/                    (full-stack scope only)
    index.html
    app.js
  tests/
    <resource1>.test.js
    <resource2>.test.js
    features.test.js          pre-written tests for all features; all fail initially
  solutions/
    routes/
      <resource1>.js          correct implementations (bugs fixed + features added)
      <resource2>.js
    app.js
  README.md                  company-style task document (see format below)
```

### `store.js` requirements

- Define seed data as `const SEED_*` constants at the top
- `resetStore()` deep-copies all seed arrays into mutable `let` variables
- `getStore()` returns `{ resource1, resource2, ..., allocate<Resource>Id() }`
- Call `resetStore()` at module load so first require is ready
- Export `{ getStore, resetStore }`

### Route file requirements

- Import `getStore` from `../store`
- Use `express.Router()`
- Access live data via `const { resource } = getStore()` inside each handler (not at module level)
- Mutate arrays in place to persist changes
- Embed bugs silently — no `// BUG` comments, no `// TODO: fix this`
- Include `// TODO (Feature X):` comments only for features, placed at the correct insertion point

### Test file requirements

- Use Jest + supertest
- `beforeEach` must call `jest.resetModules()` then `require('../app')` for a fresh state
- Test names must read like specifications: `'returns 404 when product does not exist'`
- Each `describe` block covers one endpoint or one feature
- Tests for bugs go in the resource test files (not features.test.js)
- All tests in `features.test.js` must fail on the initial codebase

### `app.js` requirements

```js
const express = require('express');
// ... router imports
const app = express();
app.use(express.json());
// ... mount routers
module.exports = { app };
if (require.main === module) { app.listen(PORT, ...); }
```

### README.md format (company-style — NO hints, NO guidance, NO reading order)

```markdown
# <App Name>

One paragraph describing what the app does.

---

## Getting started

\`\`\`bash
npm install
npm run test:<challenge>
npm run start:<challenge>
\`\`\`

---

## Project structure

\`\`\`
(file tree with one-line descriptions)
\`\`\`

---

## Domain

Short explanation of each resource and how they relate.

---

## API

Table of all endpoints with method, path, description.

---

## Your tasks

### Bug fixes

Brief statement that there are N failing tests pointing to bugs in the route files.

### New features

**Feature A — <name>**

Precise requirements as bullet points. No hints about implementation.

**Feature B — <name>**

...
```

### `solutions/` requirements

- Solutions must be complete and correct (all bugs fixed, all features implemented)
- Solutions reference `../../store` (two levels up from `solutions/routes/`)
- No `// SOLUTION` comments needed — solutions are just correct code

---

## Step 4 — Update `package.json`

Add two scripts to the nearest `package.json`:

```json
"test:<challenge-name>": "jest <challenge-name>/tests --verbose --runInBand",
"start:<challenge-name>": "node <challenge-name>/app.js"
```

---

## Step 5 — Verify

Run `npm run test:<challenge-name>` and confirm:
- Starting score is ~50–55% passing
- The right tests are failing (bugs in resource files, all of features.test.js)
- No unexpected failures in tests that should pass

Report the starting score to the user.

---

## Quality checklist before finishing

- [ ] No `// BUG` comments anywhere in routes/
- [ ] No `// SOLUTION` comments in solutions/ (just clean correct code)
- [ ] All TODO comments are feature TODOs only, placed at the right insertion point
- [ ] README reads like a company doc — task spec, not a tutorial
- [ ] Each bug is fixable with ≤ 3 lines changed
- [ ] Solutions directory has working implementations for everything (bugs fixed + features added)
- [ ] `npm run test:<challenge-name>` passes at ~50–55% on the initial codebase
