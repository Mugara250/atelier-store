# Atelier Store

Next.js (App Router) + TypeScript + Tailwind CSS, with Better Auth, Drizzle ORM and Postgres on Neon.

## Setup

```bash
npm install
cp .env.example .env.local   # fill in DATABASE_URL and BETTER_AUTH_SECRET
npm run auth:generate        # generate Better Auth tables into src/db/schema.ts
npm run db:push              # or: npm run db:generate && npm run db:migrate
npm run dev
```

## Structure

| Path | Purpose |
| --- | --- |
| `src/db/index.ts` | Drizzle client (Neon HTTP driver) |
| `src/db/schema.ts` | Drizzle table definitions |
| `drizzle.config.ts` | drizzle-kit config (migrations output to `drizzle/`) |
| `src/lib/auth.ts` | Better Auth server instance (Drizzle adapter) |
| `src/lib/auth-client.ts` | Better Auth React client |
| `src/app/api/auth/[...all]/route.ts` | Better Auth route handler |

## Scripts

- `dev`, `build`, `start`, `lint`, `typecheck`
- `db:generate`, `db:migrate`, `db:push`, `db:studio` — drizzle-kit
- `auth:generate` — Better Auth CLI schema generation
