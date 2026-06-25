# StuScout (MVP)

Skill-first hiring for B.Tech students: composite score, tag-based roles, top-10 matches, shortlist and basic pipeline.

## Prerequisites

- Node.js 20+
- A [Clerk](https://clerk.com/) app for authentication
- A [Supabase](https://supabase.com/) project for the database

## Environment

Copy `.env.example` to `.env.local` and set:

| Variable | Description |
|----------|-------------|
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | Clerk publishable key |
| `CLERK_SECRET_KEY` | Clerk secret key |
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | Server-side Supabase key |

## Run locally

```bash
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000). Sign up with Clerk, complete onboarding, then use the student or recruiter flows.

## Deploy

1. Add your Clerk and Supabase environment variables to the hosting platform.
2. Run the SQL in `supabase/migrations/001_initial_schema.sql` against your Supabase project.
3. Deploy the repo; run `npm run build` locally first to verify.

## Scripts

- `npm run dev` — development server
- `npm run build` — production build
- `npm run start` — start production server
- `npm run lint` — ESLint
