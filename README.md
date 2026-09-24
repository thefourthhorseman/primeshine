# PrimeShine

Multi-tenant cleaning service platform. Two apps live in this workspace:

| Folder | What | Local port |
|---|---|---|
| `primeshine-back/` | Node/Express REST API, PostgreSQL via Sequelize | 3001 |
| `primeshine-front/` | React 18 (Create React App) admin dashboard + public site | 3000 |
| `primeshine-tech/` | Expo/React Native technician app | Expo |
| `odoo/` | Optional Odoo back-office (Docker) | 8069 |

## Prerequisites

- Node.js 22 (`node -v`)
- PostgreSQL running locally on `127.0.0.1:5432` with the `primeshine` role
  (password `primeshine123`) and the `prime_shine` database.
  Check with `pg_isready -h 127.0.0.1`.
- `primeshine-back/.env` and `primeshine-front/.env` populated
  (see `primeshine-back/.env.example`; the front needs `REACT_APP_API_URL=http://localhost:3001`).

## Start everything (two terminals)

Terminal 1, backend:

```bash
cd primeshine-back
npm install          # first time only
npm run db:migrate   # apply any new migrations
npm run dev          # nodemon on http://localhost:3001
```

Terminal 2, frontend:

```bash
cd primeshine-front
npm install          # first time only (note: postinstall runs a full build, ~2 min)
npm run dev          # CRA dev server on http://localhost:3000
```

Open http://localhost:3000. The admin dashboard is at `/dashboard` after logging in
with an ADMIN user. Swagger API docs are served by the backend at `/api-docs`.

Do not use `npm start` for local work: on the backend it runs plain `node` without
reload, and on the frontend it serves the static `build/` folder.

## Tests

Backend (integration tests use a separate `prime_shine_test` database):

```bash
cd primeshine-back
npm run test:db:reset      # rebuild prime_shine_test from the dev schema; rerun after new migrations
npm run test:integration   # or: npm test, npm run test:unit
```

Frontend:

```bash
cd primeshine-front
npm test                   # or: npm run test:coverage
```

## Useful backend scripts

| Command | Purpose |
|---|---|
| `npm run db:migrate` / `db:migrate:undo` | Apply / roll back Sequelize migrations |
| `npm run db:seed` | Run seeders |
| `npm run restore:from-prod` | Restore an anonymized production snapshot locally |
| `npm run backup:local` | Back up the local database |

## Conventions

See `CLAUDE.md` for architecture notes and the "technician" vocabulary rule
(legacy code says "maid"; all new code and UI copy says "technician").
