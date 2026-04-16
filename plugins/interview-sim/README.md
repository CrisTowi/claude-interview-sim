# claude-interview-sim

A Claude Code skill that generates realistic coding interview simulations — existing codebases with intentional bugs to fix and features to add. Supports Node.js, TypeScript, Python/FastAPI, and Python/Flask.

## What it generates

- A REST API (+ optional vanilla JS frontend) on a domain you choose
- Intentional but non-obvious bugs embedded in the route files (no hint comments)
- Pre-written tests — some failing (bugs), some waiting for features
- Reference solutions in a `solutions/` folder
- A company-style README with no hints — just the task spec

## Install

```bash
/plugin install interview-sim@CrisTowi/claude-interview-sim
```

## Usage

```
/interview-sim
```

Claude will ask for:
- **Domain** — e-commerce, task management, auth, blog, booking, etc.
- **Scope** — backend-only or backend + frontend
- **Stack** — `node`, `typescript`, `fastapi`, or `flask`
- **Challenges** — bugs only, features only, or both
- **Duration** — 30 / 45 / 60 / 90 min
- **Output path** — where to create the challenge folder

You can also pass arguments directly:

```
/interview-sim e-commerce backend node bugs+features 60min
/interview-sim task-management backend fastapi bugs 45min
/interview-sim booking backend+frontend typescript both 90min
```

## Supported stacks

| Stack | Language | Framework | Test runner |
|---|---|---|---|
| `node` | JavaScript | Express | Jest + supertest |
| `typescript` | TypeScript | Express | ts-jest + supertest |
| `fastapi` | Python | FastAPI | pytest + httpx |
| `flask` | Python | Flask | pytest + Flask test client |

## Example output (Node.js)

```
booking-challenge/
  store.js
  app.js
  routes/
    rooms.js
    reservations.js
  tests/
    rooms.test.js
    reservations.test.js
    features.test.js
  solutions/
    routes/
      rooms.js
      reservations.js
  README.md
```

## Example output (FastAPI)

```
booking-challenge/
  store.py
  main.py
  routers/
    rooms.py
    reservations.py
  tests/
    conftest.py
    test_rooms.py
    test_reservations.py
    test_features.py
  solutions/
    routers/
      rooms.py
      reservations.py
  requirements.txt
  README.md
```

Starting score: ~50–55% passing. Target: 100%.

## License

MIT
