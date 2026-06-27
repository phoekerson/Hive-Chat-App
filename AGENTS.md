# AGENTS.md

## Cursor Cloud specific instructions

**Product:** "Hive" — a single Next.js 15 (App Router, React 19) web app for real-time chat + video calling. The Convex backend functions live in `convex/`. Auth is Clerk, chat/video is Stream (GetStream), data is Convex. Package manager is npm (`package-lock.json`).

**Standard commands** (see `package.json`):
- Install: `npm install`
- Lint: `npm run lint` (ESLint; clean except pre-existing warnings)
- Build: `npm run build` (Turbopack)
- Dev server: `npm run dev` (Turbopack, port 3000)

### Required external services / secrets
The app depends on three external SaaS providers and **hard-throws on startup** when their env vars are missing (see `components/ConvexClientProvider.tsx`, `lib/stream.ts`, `lib/streamServer.ts`). Put them in `.env.local` (gitignored via `.env*`):
- Convex: `NEXT_PUBLIC_CONVEX_URL` (from `npx convex dev`, which needs a Convex login/deployment).
- Clerk: `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`, `CLERK_SECRET_KEY`, plus `NEXT_PUBLIC_CLERK_FRONTEND_API_URL` (Clerk issuer). A Clerk **JWT template named `convex`** must exist and `CLERK_JWT_ISSUER_DOMAIN` must be set on the Convex dashboard for Convex↔Clerk auth (`convex/auth.config.ts`).
- Stream: `NEXT_PUBLIC_STREAM_API_KEY`, `STREAM_API_SECRET_KEY` (server token signing in `actions/createToken.ts`).

### Non-obvious gotchas
- **`npm run dev` uses Turbopack, which does NOT support Clerk keyless dev mode** — it hard-errors with `Missing publishableKey` if no real Clerk keys are set. Running the webpack dev server instead with `npx next dev` enables Clerk **keyless mode**, which auto-provisions temporary keys so the landing page and the sign-in/sign-up modal work without real Clerk credentials. This is only enough for the public landing page + auth modal.
- The authenticated `/dashboard` flow (user sync, chat, video) still requires a real Convex deployment URL and real Stream keys; a placeholder `NEXT_PUBLIC_CONVEX_URL` stops the provider from throwing but the dashboard's Convex mutations and Stream `connectUser` will fail.
- To run data/chat end-to-end, run `npx convex dev` alongside the Next dev server so Convex functions in `convex/` are pushed.
