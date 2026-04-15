# claude-interview-sim

A Claude Code skill that generates realistic Node.js interview simulations — existing codebases with intentional bugs to fix and features to add.

## What it generates

- An Express REST API (+ optional vanilla JS frontend) on a domain you choose
- Intentional but non-obvious bugs embedded in the route files (no hint comments)
- Pre-written Jest + supertest tests — some failing (bugs), some waiting for features
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
- **Challenges** — bugs only, features only, or both
- **Duration** — 30 / 45 / 60 / 90 min
- **Output path** — where to create the challenge folder

You can also pass arguments directly:

```
/interview-sim e-commerce backend+frontend bugs+features 60min
```

## Example output

```
booking-challenge/
  store.js
  app.js
  routes/
    rooms.js
    reservations.js
  public/
    index.html
    app.js
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

Starting score: ~17/30. Target: 30/30.

## License

MIT
