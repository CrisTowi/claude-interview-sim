---
name: interview-sim
description: >
  Creates a realistic coding interview simulation — an existing codebase the candidate receives
  with intentional bugs to fix and/or features to add. Supports Node.js/Express, TypeScript/Express,
  Python/FastAPI, and Python/Flask. Generates routes/handlers, an in-memory store, pre-written tests,
  reference solutions, and a company-style README with no hints.
  Use when asked to "create an interview sim", "new interview challenge", or "generate a coding test".
argument-hint: [domain] [scope] [stack] [challenges] [duration]
allowed-tools: Read Glob Grep Write Edit Bash
---

# Interview Simulation Generator

You are generating a realistic coding challenge. The candidate receives an existing codebase with some failing tests and a set of TODO features. The goal is to simulate what a real company hands a candidate on interview day.

## Step 1 — Gather requirements

Ask the user the following questions before generating anything. Use `AskUserQuestion` with all questions in a single call.

Required inputs:
- **Domain** — what the app is about (e.g. e-commerce, task management, auth/users, blog/CMS, booking system, inventory)
- **Scope** — backend-only (REST API) or full-stack (API + vanilla JS frontend)
- **Stack** — which tech stack to use:
  - `node` — Node.js + Express + Jest + supertest (default)
  - `typescript` — TypeScript + Express + ts-jest + supertest
  - `fastapi` — Python + FastAPI + pytest + httpx TestClient
  - `flask` — Python + Flask + pytest + Flask test client
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
- Pass a linter without warnings (avoid assignment-inside-filter patterns — IDEs auto-fix these)
- Cause 1–2 specific tests to fail
- Be fixable with a small, targeted change (≤ 3 lines)

**High-quality bug patterns to use:**
- Mutate-before-validate: push to array before checking uniqueness constraint
- Wrong arithmetic operator: `+` instead of `-`, `*` missing (total ignores quantity)
- Wrong field reference: filtering on `item.price` when it should be `item.unitPrice`
- Off-by-one or wrong comparison: `>` instead of `>=`, `<` instead of `<=`
- Logic inversion: checking `!required` when it should be `required`
- Missing condition branch: returning 200 with `null` instead of 404 when record not found
- State not restored: action modifies related data but doesn't undo on rollback/cancel
- Wrong default: a field initialized to `undefined` (JS/TS) or `None` (Python) instead of a specific value

**Avoid:**
- Syntax errors or typos that any editor catches
- Assignment-in-condition patterns
- Obvious wrong types
- Bugs that break more than 3 tests at once

### Feature design

Each feature should:
- Require adding 1 new endpoint or extending an existing one
- Touch at least 2 files (e.g. a route + the store)
- Have a clear, completable spec (stated in README as requirements)
- Have pre-written tests the candidate must make pass
- One feature per challenge should include a structural pitfall (e.g. route ordering, missing `await`, shared state) to reward careful reading

### Test design

For each bug: write 1–2 tests that fail only because of that bug.
For each feature: write 3–5 tests that fully cover the expected behavior.

Starting score target: ~50–55% passing (healthy-but-broken feel of a real codebase).

---

## Step 3 — Generate files

Choose the structure and file contents based on the selected **stack**. All stacks share the same layout philosophy: store module, handlers/routes, tests, solutions, and a company-style README.

---

### Stack: `node` (Node.js + Express + Jest + supertest)

```
<challenge-name>/
  store.js
  app.js
  routes/
    <resource1>.js
    <resource2>.js
  public/                    (full-stack scope only)
    index.html
    app.js
  tests/
    <resource1>.test.js
    <resource2>.test.js
    features.test.js
  solutions/
    routes/
      <resource1>.js
      <resource2>.js
    app.js
  README.md
```

**`store.js`**
- Define seed data as `const SEED_*` constants at the top
- `resetStore()` deep-copies all seed arrays into mutable `let` variables
- `getStore()` returns `{ resource1, resource2, ..., allocate<Resource>Id() }`
- Call `resetStore()` at module load; export `{ getStore, resetStore }`

**Route files**
- Import `getStore` from `../store`; use `express.Router()`
- Access live data via `const { resource } = getStore()` inside each handler
- Embed bugs silently — no `// BUG` comments
- Include `// TODO (Feature X):` comments only for features at the correct insertion point

**`app.js`**
```js
const express = require('express');
const app = express();
app.use(express.json());
// mount routers
module.exports = { app };
if (require.main === module) { app.listen(PORT, () => {}); }
```

**Test files**
- Use Jest + supertest
- `beforeEach` calls `jest.resetModules()` then `require('../app')` for fresh state
- Test names read like specs: `'returns 404 when product does not exist'`
- Bug tests go in resource test files; all feature tests go in `features.test.js`

---

### Stack: `typescript` (TypeScript + Express + ts-jest + supertest)

Same directory structure as `node` but with `.ts` extensions. Add:

```
<challenge-name>/
  tsconfig.json
  package.json              (local, not shared)
  store.ts
  app.ts
  routes/
    <resource1>.ts
    <resource2>.ts
  tests/
    <resource1>.test.ts
    <resource2>.test.ts
    features.test.ts
  solutions/
    routes/
      <resource1>.ts
      <resource2>.ts
    app.ts
  README.md
```

**`tsconfig.json`**
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "strict": true,
    "esModuleInterop": true,
    "outDir": "dist"
  },
  "include": ["*.ts", "routes/**/*.ts", "tests/**/*.ts"]
}
```

**`package.json`** (local to the challenge)
```json
{
  "scripts": {
    "test": "jest --verbose --runInBand",
    "start": "ts-node app.ts"
  },
  "dependencies": { "express": "^4.18.0" },
  "devDependencies": {
    "@types/express": "^4.17.0",
    "@types/jest": "^29.0.0",
    "@types/supertest": "^6.0.0",
    "jest": "^29.0.0",
    "supertest": "^6.3.0",
    "ts-jest": "^29.0.0",
    "ts-node": "^10.9.0",
    "typescript": "^5.0.0"
  },
  "jest": {
    "preset": "ts-jest",
    "testEnvironment": "node"
  }
}
```

**Store and route files** — same logic as `node` stack but with TypeScript interfaces, explicit return types, and typed `Request`/`Response` from express.

**`store.ts`** — define an interface per resource (e.g. `interface Product { id: number; name: string; ... }`); `getStore()` returns a typed object.

**Test files** — same patterns as `node` but with TypeScript imports and typed supertest responses.

---

### Stack: `fastapi` (Python + FastAPI + pytest + httpx)

```
<challenge-name>/
  store.py
  main.py
  routers/
    <resource1>.py
    <resource2>.py
  tests/
    conftest.py
    test_<resource1>.py
    test_<resource2>.py
    test_features.py
  solutions/
    routers/
      <resource1>.py
      <resource2>.py
    main.py
  requirements.txt
  README.md
```

**`requirements.txt`**
```
fastapi>=0.110.0
uvicorn>=0.29.0
httpx>=0.27.0
pytest>=8.0.0
```

**`store.py`**
```python
from copy import deepcopy

SEED_PRODUCTS = [{"id": 1, "name": "Widget", ...}]
# ... other seed data

_store: dict = {}

def reset_store() -> None:
    global _store
    _store = {
        "products": deepcopy(SEED_PRODUCTS),
        # ...
        "_next_product_id": 2,
    }

def get_store() -> dict:
    return _store

reset_store()
```

**`main.py`**
```python
from fastapi import FastAPI
from routers import resource1, resource2

app = FastAPI()
app.include_router(resource1.router, prefix="/resource1", tags=["resource1"])
app.include_router(resource2.router, prefix="/resource2", tags=["resource2"])
```

**Router files**
```python
from fastapi import APIRouter, HTTPException
from store import get_store

router = APIRouter()

@router.get("/{item_id}")
def get_item(item_id: int):
    store = get_store()
    item = next((p for p in store["items"] if p["id"] == item_id), None)
    if item is None:
        raise HTTPException(status_code=404, detail="Not found")
    return item
```

Embed bugs silently; use `# TODO (Feature X):` comments only for features.

**`tests/conftest.py`**
```python
import pytest
from fastapi.testclient import TestClient
from store import reset_store
import main

@pytest.fixture(autouse=True)
def fresh_store():
    reset_store()
    yield

@pytest.fixture
def client():
    return TestClient(main.app)
```

**Test files**
```python
def test_returns_404_when_item_not_found(client):
    response = client.get("/items/9999")
    assert response.status_code == 404
```

**Solutions** — reference `from store import get_store` (store is one level up from solutions/routers/ but import paths use the project root since tests run from `<challenge-name>/`).

---

### Stack: `flask` (Python + Flask + pytest + Flask test client)

```
<challenge-name>/
  store.py
  app.py
  routes/
    <resource1>.py
    <resource2>.py
  tests/
    conftest.py
    test_<resource1>.py
    test_<resource2>.py
    test_features.py
  solutions/
    routes/
      <resource1>.py
      <resource2>.py
    app.py
  requirements.txt
  README.md
```

**`requirements.txt`**
```
flask>=3.0.0
pytest>=8.0.0
```

**`store.py`** — same pattern as FastAPI stack.

**`app.py`**
```python
from flask import Flask
from routes.resource1 import resource1_bp
from routes.resource2 import resource2_bp

def create_app():
    app = Flask(__name__)
    app.register_blueprint(resource1_bp)
    app.register_blueprint(resource2_bp)
    return app

if __name__ == "__main__":
    create_app().run(debug=True, port=3000)
```

**Route files**
```python
from flask import Blueprint, jsonify, request, abort
from store import get_store

resource1_bp = Blueprint("resource1", __name__, url_prefix="/resource1")

@resource1_bp.get("/<int:item_id>")
def get_item(item_id):
    store = get_store()
    item = next((p for p in store["items"] if p["id"] == item_id), None)
    if item is None:
        abort(404)
    return jsonify(item)
```

Embed bugs silently; use `# TODO (Feature X):` comments only for features.

**`tests/conftest.py`**
```python
import pytest
from store import reset_store
from app import create_app

@pytest.fixture(autouse=True)
def fresh_store():
    reset_store()
    yield

@pytest.fixture
def client():
    app = create_app()
    app.config["TESTING"] = True
    with app.test_client() as c:
        yield c
```

**Test files**
```python
def test_returns_404_when_item_not_found(client):
    response = client.get("/items/9999")
    assert response.status_code == 404
```

---

### README.md format (all stacks — company-style, NO hints, NO guidance)

```markdown
# <App Name>

One paragraph describing what the app does.

---

## Getting started

\`\`\`bash
# Node/TypeScript
npm install
npm test

# Python
pip install -r requirements.txt
pytest tests/ -v
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

Brief statement that there are N failing tests pointing to bugs in the route/router files.

### New features

**Feature A — <name>**

Precise requirements as bullet points. No hints about implementation.

**Feature B — <name>**

...
```

---

## Step 4 — Install dependencies and configure test script

### Node.js / TypeScript

If the challenge has its own `package.json` (TypeScript stack), run `npm install` inside the challenge folder.

Otherwise (Node.js stack), add two scripts to the nearest root `package.json`:
```json
"test:<challenge-name>": "jest <challenge-name>/tests --verbose --runInBand",
"start:<challenge-name>": "node <challenge-name>/app.js"
```

### Python (FastAPI / Flask)

Run `pip install -r requirements.txt` inside the challenge folder.

The test command is: `pytest <challenge-name>/tests -v` (or just `pytest tests/ -v` from inside the folder).

---

## Step 5 — Verify

Run the test command for the chosen stack and confirm:
- Starting score is ~50–55% passing
- The right tests are failing (bug tests + all of `features.test.js` / `test_features.py`)
- No unexpected failures in tests that should pass

Report the starting score to the user.

---

## Quality checklist before finishing

- [ ] No `# BUG` / `// BUG` comments anywhere in routes/routers/
- [ ] No `# SOLUTION` / `// SOLUTION` comments in solutions/ (just clean correct code)
- [ ] All TODO comments are feature TODOs only, placed at the right insertion point
- [ ] README reads like a company doc — task spec, not a tutorial
- [ ] Each bug is fixable with ≤ 3 lines changed
- [ ] Solutions directory has working implementations for everything (bugs fixed + features added)
- [ ] Tests pass at ~50–55% on the initial codebase
- [ ] Python challenges: `conftest.py` uses `autouse=True` fixture to reset store before each test
- [ ] TypeScript challenges: `tsconfig.json` and local `package.json` are present and correct
