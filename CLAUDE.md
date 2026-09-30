# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENTS.md

Next.js 16 docs for the installed version are in `node_modules/next/dist/docs/` (`01-app/` covers the App Router). Read them before using Next APIs, since conventions differ from older versions.

## Commands

```bash
npm run dev          # dev server (http://localhost:3000)
npm run build        # production build — requires DATABASE_URL to be set (see below)
npm run lint         # ESLint (flat config, eslint-config-next)
npm run typecheck    # tsc --noEmit

npm run auth:generate   # Better Auth CLI: writes auth tables into src/db/schema.ts
npm run db:generate     # drizzle-kit: create SQL migration in drizzle/
npm run db:migrate      # apply migrations
npm run db:push         # push schema directly (no migration files)
npm run db:studio       # Drizzle Studio
```

There is no test framework yet.

Env setup: `cp .env.example .env.local` and fill in `DATABASE_URL` (Neon pooled connection string), `BETTER_AUTH_SECRET`, and `BETTER_AUTH_URL`.

## Architecture

Stack: Next.js App Router with a `src/` directory, React 19, TypeScript (strict), Tailwind v4 (configured via `@tailwindcss/postcss` and `@theme` in `src/app/globals.css`, with no `tailwind.config`), Better Auth, Drizzle ORM, and Neon Postgres. The import alias is `@/*` → `src/*`.

Request/data flow:

- `src/lib/auth-client.ts` is the browser-side Better Auth client (`better-auth/react`). It has no `baseURL`, so it calls the same origin at `/api/auth/*`.
- `src/app/api/auth/[...all]/route.ts` is a catch-all handler that passes every auth request to the server instance via `toNextJsHandler(auth)`.
- `src/lib/auth.ts` holds the single `auth` instance. It uses the Drizzle adapter (`provider: "pg"`) with the shared `db` and `schema`. Server code should import this instance for sessions (for example `auth.api.getSession`). The `nextCookies()` plugin must stay **last** in `plugins`. Better Auth reads `BETTER_AUTH_SECRET` and `BETTER_AUTH_URL` from the environment itself.
- `src/db/index.ts` holds the single `db` instance, built with Drizzle and the Neon **HTTP** driver (`drizzle-orm/neon-http`). Each query is a stateless HTTP request, so interactive transactions are not available with this driver. It throws at module load if `DATABASE_URL` is missing. The auth route imports it, so `next build` fails without that variable.
- `src/db/schema.ts` is the one schema file, shared by the runtime (`db` and the auth adapter) and by drizzle-kit (`drizzle.config.ts`, which loads `.env.local` and `.env` through dotenv because it runs outside Next). It is currently empty. Better Auth needs its tables, generated with `npm run auth:generate`, before auth requests work.

Current state: this is scaffolding only. No auth methods are enabled, there are no app tables, and `src/app/page.tsx` and the layout metadata are still the create-next-app defaults.
