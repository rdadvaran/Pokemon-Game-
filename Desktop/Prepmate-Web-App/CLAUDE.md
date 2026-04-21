# CLAUDE.md — PrepMate

This file is the master index for the PrepMate project. It tells Claude Code how the project is organized, what tech stack to use, the conventions to follow, and where to look for the detailed specification of every page, section, and component.

**Do not put full page specs inside this file.** Everything that describes *what a page does or looks like* lives in `pages_prompts/`. Everything that describes *how to wire up a third-party component* lives in `prompts/`. This file only references them with the `@` syntax.

---

## 1. Project Overview

PrepMate is a digital mentorship platform connecting Grade 11 and 12 students applying to competitive university programs with current students already enrolled in those exact programs. Mentors are matched based on the student's target program, biography, and goals, giving each student access to someone who went through the exact same admissions process, for the exact same program, within the last one to two years.

The business, pricing, services, and differentiators are fully documented in:

- @pages_prompts/_business-context.md

Read that file first. Every page on the site is written to sell, support, or deliver the services described there. Do not invent services, prices, or policies that contradict it.

---

## 2. Tech Stack

- **Framework:** Next.js (App Router) + TypeScript. Chosen for SEO optimization.
- **Styling:** Tailwind CSS.
- **Component library:** shadcn/ui (components live in `/components/ui`).
- **Animation:** Framer Motion.
- **Icons:** lucide-react.
- **Backend / Auth / DB:** Supabase (recommended default — auth, Postgres, row-level security, storage).
- **Payments:** Stripe (checkout for packages, and for "buy more hours" on the user dashboard).
- **Scheduling / Booking:**
  - **Free consultation booking** (Consultation page): a Calendly-style calendar tool that is easy to expand later and can sync with Google Calendar. Cal.com (self-hosted or cloud) is the recommended default because it is open source, easy to expand, and integrates with Google Calendar and Zoom out of the box.
  - **Mentor ↔ Student session booking** (inside dashboards): a custom calendar backed by Supabase, where the mentor's availability page drives what the student can book. The same Cal.com / Google Calendar integration can be reused here to avoid building availability logic from scratch.
- **Video meetings:** Zoom API — auto-generate a Zoom link when a consultation or session is booked and email it to both parties.
- **Email:** Resend (for transactional emails: consultation confirmations, Zoom links, password setup, payment confirmations, payout notices).

> Note: Every backend technology above (Supabase, Stripe, Cal.com, Zoom, Resend) is a **recommendation**, not a lock-in. Replace any of them before writing production code if Sepanta/Radin prefer a different vendor.

---

## 3. Folder Structure (project root)

```
/
├── CLAUDE.md                    ← this file (master index)
├── .claude/                     ← plugins and skills (managed by Sepanta)
├── pages_prompts/               ← one file per page/section; full descriptive spec
│   ├── _business-context.md     ← company info, pricing, services, policies, voice
│   ├── _global.md               ← navbar, footer, SEO, brand voice, conventions
│   ├── home.md                  ← landing/home page
│   ├── about.md                 ← about us page
│   ├── consultation.md          ← consultation form + booking calendar
│   ├── login.md                 ← login + password setup + forgot password
│   ├── checkout.md              ← first-login package selection + payment
│   ├── user-dashboard.md        ← student dashboard (overview, booking, buy hours)
│   ├── mentor-dashboard.md      ← mentor dashboard (students, hours, availability)
│   └── admin-dashboard.md       ← admin dashboard (users, mentors, matching, payments)
├── prompts/                     ← verbatim pre-engineered component integration prompts
│   ├── packages-scroll-expansion.md
│   └── testimonials-animated.md
└── public/
    ├── logo.svg                 ← placeholder — replace with real logo
    ├── headshots/
    │   ├── sepanta.jpg          ← placeholder — replace with real headshot
    │   └── radin.jpg            ← placeholder — replace with real headshot
    └── testimonials/
        ├── README.md            ← documents the testimonial .txt file format
        ├── testimonial-01.txt   ← placeholder testimonial (one per file)
        ├── testimonial-02.txt
        └── testimonial-03.txt
```

---

## 4. Global Conventions

Before building any page, read:

- @pages_prompts/_global.md

This covers the navbar, footer, brand voice, typography, SEO metadata, mobile behavior, and the conventions that apply to every page.

---

## 5. Page-Level Specs

Each of these files is the **single source of truth** for its page. If something is unclear, ask before changing it; do not silently restructure.

- **Landing / Home page:** @pages_prompts/home.md
- **About Us page:** @pages_prompts/about.md
- **Consultation page (form + booking calendar):** @pages_prompts/consultation.md
- **Login page:** @pages_prompts/login.md
- **Checkout page (first-login package selection + payment):** @pages_prompts/checkout.md
- **Student dashboard:** @pages_prompts/user-dashboard.md
- **Mentor dashboard:** @pages_prompts/mentor-dashboard.md
- **Admin dashboard:** @pages_prompts/admin-dashboard.md

---

## 6. Pre-Engineered Component Prompts

These prompts are the **verbatim** integration instructions for specific third-party components from 21st.dev. They are already engineered; do not rewrite them. Use them exactly when building the section they belong to.

- **Packages section (Home page):** @prompts/packages-scroll-expansion.md
  - Used on the home page to present the three packages (Accelerated, Momentum, Path to Success) as scroll-expanding feature blocks, where each package expands into its own immersive media experience.
- **Testimonials section (Home page):** @prompts/testimonials-animated.md
  - Used on the home page to render the testimonial carousel. Testimonial content is sourced from `.txt` files in `/public/testimonials/` — one testimonial per file — following the format documented in `/public/testimonials/README.md`.

The "Create the need" section on the home page is **not** built from a pre-engineered component prompt. It is custom-built in Tailwind per the spec in @pages_prompts/home.md (the two-column "What most applicants do" vs. "What PrepMate applicants know" layout, followed by the "What makes us different" list).

---

## 7. User Roles and Account Lifecycle

Three roles exist, and every page you build must enforce role-based routing:

1. **Student** — default role after the admin approves them post-consultation.
2. **Mentor** — added manually by the admin.
3. **Admin** — Sepanta, Radin, and anyone else granted admin access.

**Lifecycle for a new student:**

1. Prospect lands on the site → clicks "Book a free consultation" → fills the consultation form → books a 30-min slot → receives Zoom link.
2. Consultation happens. If it goes well, the admin adds the prospect's email to the approved sign-in list via the admin dashboard.
3. The prospect visits the login page, enters the approved email, is prompted to create a secure password, and is logged in for the first time.
4. On first login, the student sees the **checkout page** showing the three package options. They pick one → pay via Stripe → land on the student dashboard.
5. From the dashboard, they can book sessions with their assigned mentor, see remaining hours, and buy 1–9 additional hours at a time.

**Lifecycle for a mentor:** admin creates the account, mentor sets password on first login, lands on mentor dashboard.

**Anyone not on the approved sign-in list** who tries to log in sees a polite message and a "Book a free consultation" button that redirects to the consultation page.

---

## 8. Payment Rules (hard-coded; do not alter without approval)

Pricing is defined in @pages_prompts/_business-context.md. Never let these drift. Summary:

- Accelerated Program: $90/hr × 10 hrs = **$900**
- Momentum Program: $85/hr × 20 hrs = **$1,700**
- Path to Success: $75/hr × 40 hrs = **$3,000**
- Additional hours purchasable from the dashboard: **1 to 9 hours** self-serve. More than 9 requires contacting PrepMate directly.
- **Refund policy:** Full refund if the student requests one within the first 60 minutes of their first session. If the student doesn't like the first session (≤60 min), the mentor is not compensated.
- **Mentor switching:** allowed anytime.
- Mentor payouts: bi-weekly, $30/hr.

---

## 9. Design & Voice Conventions

Full brand voice lives in @pages_prompts/_global.md. Short version:

- Write like a student talking to another student who is a year or two younger. Warm, direct, confident, slightly personal — not corporate.
- Never use corporate filler ("leverage our platform to unlock your potential"). Never over-promise.
- Prefer concrete language ("your mentor got into Queen's Commerce last year") over vague language ("we match you with a qualified advisor").
- Every page ends with either a consultation CTA or a dashboard action. Don't leave dead ends.

---

## 10. Workflow for Claude Code

When asked to build a page:

1. Read @pages_prompts/_business-context.md and @pages_prompts/_global.md first.
2. Read the specific page spec from `pages_prompts/`.
3. If the page references a pre-engineered component prompt, read that file from `prompts/` and follow it verbatim.
4. Build the page. Do not restructure the spec. If something is ambiguous or seems wrong, **stop and ask** before changing it.
5. Keep components small and composable. Put reusable pieces under `/components/` and shadcn primitives under `/components/ui/`.

When asked to change copy, change it only in the relevant `pages_prompts/` file, then update the page to match. The `pages_prompts/` file is the source of truth.

---

## 11. Build Status (as of April 2026)

**All 29 routes are built and the Next.js build passes with zero TypeScript errors.** The frontend is complete. Supabase auth and database are fully wired and live. Resend is live with API key configured. Stripe and Zoom are UI-only — credentials not yet configured.

### Environment

- **Node version:** Must be Node 20. Run `nvm use 20` before `npm run dev` or `npm run build`. The `.nvmrc` file pins this.
- **Dev server:** `npm run dev` from the project root.
- **Supabase project:** `qnrvkqzaozilinnalozs.supabase.co` — live and connected via `.env.local`.

### Completed pages and routes

| Route | Page | Status |
|---|---|---|
| `/` | Home (Hero, CreateTheNeed, WhatMakesUsDifferent, Packages, Testimonials, FinalCTA) | ✅ Built |
| `/about` | About Us with founders section | ✅ Built |
| `/consultation` | Multi-step questionnaire + Cal.com calendar slot | ✅ Built |
| `/login` | Allowlist-gated sign-in | ✅ Built |
| `/login/forgot-password` | Password reset request | ✅ Built |
| `/login/reset-password` | Password reset form | ✅ Built |
| `/checkout` | First-login package selection + Stripe payment | ✅ Built |
| `/checkout/success` | Post-payment confirmation | ✅ Built |
| `/dashboard` | Student dashboard overview | ✅ Built |
| `/dashboard/booking` | Session booking calendar | ✅ Built |
| `/dashboard/hours` | Buy additional hours | ✅ Built |
| `/dashboard/account` | Student account settings | ✅ Built |
| `/mentor` | Mentor dashboard — student list | ✅ Built |
| `/mentor/schedule` | Mentor schedule view | ✅ Built |
| `/mentor/availability` | Mentor availability settings | ✅ Built |
| `/mentor/compensation` | Hours & pay summary | ✅ Built |
| `/mentor/account` | Mentor account settings | ✅ Built |
| `/mentor/students/[id]` | Per-student detail for mentors | ✅ Built |
| `/admin` | Admin overview with metrics | ✅ Built |
| `/admin/students` | Student list and filters | ✅ Built |
| `/admin/students/[id]` | Student detail page | ✅ Built |
| `/admin/mentors` | Mentor list and filters | ✅ Built |
| `/admin/mentors/[id]` | Mentor detail page | ✅ Built |
| `/admin/matching` | Student ↔ mentor matching panel | ✅ Built |
| `/admin/consultations` | Consultation funnel management | ✅ Built |
| `/admin/approved-signups` | Allowlist management | ✅ Built |
| `/admin/payments` | Revenue, payouts, refunds | ✅ Built |
| `/admin/account` | Admin account settings | ✅ Built |

### Key component notes

**PackagesSection (`components/home/PackagesSection.tsx`):**
The 21st.dev `ScrollExpandMedia` component was replaced entirely with a custom Framer Motion implementation. The original hijacked global scroll events (wheel + touchmove `preventDefault` + `scrollTo(0,0)`) which broke all three stacked instances and prevented `IntersectionObserver` from firing. The custom implementation uses:
- Tall outer containers with `sticky top-0 h-[100dvh]` inner panels
- `useScroll({ offset: ["start start", "end end"] })` for per-package 0→1 progress
- `clipPath: inset(...)` zoom via `useMotionTemplate` — expands from a bordered card to full viewport
- One-way animation via `peakProgress` ref — once zoomed in, the card never un-zooms when scrolling back up
- Phase rows revealed sequentially via `useState<boolean[]>` that only ever flips `true` — phase reveals are also permanent
- Auto-scroll to the next package triggers at progress ≥ 0.88, landing 14% into the next package's scroll container so the zoom-in plays during the smooth scroll
- z-index stacking (10, 20, 30) so later packages render on top of earlier ones

**TestimonialsSection (`components/home/TestimonialsSection.tsx`):**
Uses the 21st.dev `AnimatedTestimonials` component. Testimonial content is sourced from `/public/testimonials/*.txt` at build time.

### What still needs real content before launch

- `/public/headshots/sepanta.jpg` and `/public/headshots/radin.jpg` — replace placeholders with real photos
- `/public/hero.mp4` and `/public/hero-poster.jpg` — replace placeholder with real campus footage
- `/public/testimonials/testimonial-01.txt` through `testimonial-03.txt` — replace with real student testimonials
- Radin's program and year in `pages_prompts/about.md` and `pages_prompts/_business-context.md`
- Real email address — currently `hello@prepmate.ca` throughout; confirm before launch
- Stripe publishable key + secret key + webhook secret in `.env.local`
- Cal.com embed URL configured with Sepanta/Radin's actual availability
- Zoom API credentials in `.env.local`
- Resend domain verification — verify `prepmate.ca` in Resend dashboard, then update `FROM` in `lib/email.ts` from `onboarding@resend.dev` to `hello@prepmate.ca` (see Section 16 for full steps)
- In Supabase dashboard → Authentication → URL Configuration: set Site URL to `http://localhost:3000` (prod: real domain) and add `http://localhost:3000/login/reset-password` to Redirect URLs

---

## 12. Backend Workplan

The frontend (all 29 routes) is complete. Branches 1 (`backend/auth-db`), 4 (`backend/email`), and 5 (`backend/admin`) are fully complete and merged into `main`. Branches 2 (`backend/payments`) and 3 (`backend/scheduling`) are the remaining work and can be developed in parallel.

---

### Branch 1 — `backend/auth-db` ✅ COMPLETE

All Supabase infrastructure is live. See Section 14 for the full details of what was built.

---

### Branch 2 — `backend/payments`

Depends on: `backend/auth-db` (needs `profiles` table and Supabase client).

**Stripe setup**
- Add `STRIPE_PUBLISHABLE_KEY`, `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET` to `.env.local`
- Create the three package products and one-off price objects in Stripe; store their price IDs in `lib/packages.ts` alongside the existing package config

**API routes**

| Route | What to do |
|---|---|
| `POST /api/checkout` | Create Stripe Checkout Session for package purchase; `PackageSelectionClient.tsx` calls this with a `slug` |
| `POST /api/hours/checkout` | Create Checkout Session for 1–9 add-on hours at the student's package hourly rate; `dashboard/hours/page.tsx` calls this |
| `POST /api/stripe/webhook` | Verify Stripe signature; on `checkout.session.completed`: set `package_purchased_at`, `hours_included`, `hours_remaining` on `profiles`; trigger receipt email via Resend |

**`/checkout/success` page**
- Replace the current static confirmation with a server component that queries Supabase to confirm fulfillment has happened; show a "matching you with a mentor now" pending state if the webhook hasn't fired yet

---

### Branch 3 — `backend/scheduling`

Depends on: `backend/auth-db`.

**Zoom** — create `lib/zoom.ts`
- Implement `createMeeting(topic, startTime, durationMinutes)` using the Zoom Server-to-Server OAuth flow
- Returns `{ join_url, meeting_id }`
- Add `ZOOM_ACCOUNT_ID`, `ZOOM_CLIENT_ID`, `ZOOM_CLIENT_SECRET` to `.env.local`
- Called from: consultation booking webhook, session booking route

**Cal.com integration**
- Configure Cal.com with Sepanta/Radin's real availability and the 30-min slot + 30-min buffer settings
- Create a Cal.com webhook that POSTs to `POST /api/consultation` on booking confirmation
- `POST /api/consultation` — save all form fields + booked_slot + zoom_link to `consultations` table; status = `consultation_booked`

**Session booking**

| Route | What to do |
|---|---|
| `POST /api/sessions` | Create row in `sessions`, deduct hours from `profiles.hours_remaining`, call `lib/zoom.ts`, call Resend to email both student and mentor |
| `POST /api/sessions/[id]/cancel` | Set status = cancelled, refund hours, email both parties |

**Mentor availability**

| Route | What to do |
|---|---|
| `GET /api/mentor/availability` | Return recurring slots from `mentor_availability` so the availability page loads saved state on mount |
| `POST /api/mentor/availability` | Save recurring slots; called by the availability page on save |

**Booking calendar (`/dashboard/booking`)**
- Wire up the calendar to fetch the assigned mentor's available slots from Supabase (combining `mentor_availability` + existing `sessions` to exclude already-booked times)
- Connect the "Book session" confirmation modal to `POST /api/sessions`

---

### Branch 4 — `backend/email` ✅ COMPLETE

All 11 email functions are built in `lib/email.ts`. Resend SDK installed. 4 functions wired into existing routes. See Section 16 for full details.

**Still pending (depend on other branches):**
- `sendConsultationConfirmation` + `sendAdminConsultationNotification` — wire into `POST /api/consultation` when built (branch 3)
- `sendPackageReceipt` — wire into Stripe webhook when built (branch 2)
- `sendAddOnHoursReceipt` — wire into `POST /api/hours/checkout` success when built (branch 2)
- `sendSessionConfirmation` + `sendSessionCancelled` — wire into `POST /api/sessions` and `POST /api/sessions/[id]/cancel` when built (branch 3)
- `sendMentorPayoutNotice` — wire into `POST /api/admin/payouts/[id]/mark-paid` when built (branch 2)

---

### Branch 5 — `backend/admin` ✅ COMPLETE

All admin pages fetch real Supabase data. All admin API routes are live. See Section 15 for full details.

**Still pending (depend on other branches):**
- `POST /api/admin/refund` — needs Stripe (branch 2) to call the refund API
- `POST /api/admin/payouts/[id]/mark-paid` email notification — needs Resend (branch 4) to call `sendMentorPayoutNotice`

---

### Required `.env.local` keys (summary)

```
NEXT_PUBLIC_SUPABASE_URL=        ✅ set
NEXT_PUBLIC_SUPABASE_ANON_KEY=   ✅ set
SUPABASE_SERVICE_ROLE_KEY=       ✅ set

STRIPE_PUBLISHABLE_KEY=          ⬜ pending (branch 2)
STRIPE_SECRET_KEY=               ⬜ pending (branch 2)
STRIPE_WEBHOOK_SECRET=           ⬜ pending (branch 2)

ZOOM_ACCOUNT_ID=                 ⬜ pending (branch 3)
ZOOM_CLIENT_ID=                  ⬜ pending (branch 3)
ZOOM_CLIENT_SECRET=              ⬜ pending (branch 3)

RESEND_API_KEY=                  ✅ set — verify prepmate.ca domain in Resend before launch
```

---

## 13. Git Branch Strategy

All five backend branches have been created. Branches 1, 4, and 5 are complete and merged into `main`. Branches 2 and 3 are the remaining work.

### Branch overview

| Branch | Scope | Status |
|---|---|---|
| `backend/auth-db` | Supabase setup, all DB tables, auth API routes, middleware | ✅ Merged into `main` |
| `backend/payments` | Stripe checkout, add-on hours, webhook | ⬜ Not started |
| `backend/scheduling` | Sessions API, availability, Cal.com webhook, Zoom | ⬜ Not started |
| `backend/email` | Resend — all email templates and send calls | ✅ Merged into `main` |
| `backend/admin` | Admin API routes, replace all mock data | ✅ Merged into `main` |

### How to work day-to-day

Create and switch to the branch for your current task, branching off `main`:

```bash
git checkout main && git pull
git checkout -b backend/payments   # Branch 2
git checkout -b backend/scheduling # Branch 3
```

Commit normally on your branch. When a branch is ready for review, open a PR into `main`.

### Merge order for remaining branches

```bash
# Rebase onto main before merging to pick up auth-db, email, and admin
git checkout backend/payments
git rebase main
git checkout main
git merge backend/payments

# Repeat for scheduling
```

### Wiring email into branches 2 and 3

`lib/email.ts` is already on `main` with all 11 functions exported and typed. Branches 2 and 3 just need to import and call the relevant functions — no new email code needed. See Section 16 for the full function signatures.

---

## 14. Completed Backend Work (branch: `backend/auth-db`)

Everything below is built, live, and working as of April 2026.

### Supabase project

- Project URL: `https://qnrvkqzaozilinnalozs.supabase.co`
- Credentials in `.env.local` (`NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`)
- Migration files in `supabase/migrations/`

### Supabase client files

| File | Purpose |
|---|---|
| `lib/supabase.ts` | Browser client (`createBrowserClient` from `@supabase/ssr`) |
| `lib/supabase-server.ts` | Server/RSC client (cookie-based, for server components and route handlers) |
| `lib/supabase-admin.ts` | Service role client — bypasses RLS, server-only, used for admin operations |

### Database tables (all live with RLS policies)

All 9 tables created via `supabase/migrations/001_initial_schema.sql` and `002_add_name_phone_fields.sql`:

| Table | Notes |
|---|---|
| `approved_signups` | Email allowlist. Service-role only write. Seeded with founder emails. |
| `profiles` | One row per user. Columns: `id`, `email`, `role`, `first_name`, `last_name`, `full_name`, `phone`, `package_name`, `hours_included`, `hours_remaining`, `package_purchased_at`, `mentor_id` |
| `mentor_profiles` | Mentor-specific data: `school`, `program`, `year`, `bio`, `headshot_url`, `payout_email` |
| `consultations` | Consultation form submissions + booking records |
| `sessions` | Booked mentor↔student sessions |
| `mentor_availability` | Recurring weekly slots (public read) |
| `mentor_availability_exceptions` | One-off blackouts/openings (public read) |
| `payouts` | Mentor pay records |
| `admin_actions` | Audit log for all admin actions |

RLS policies use `auth.jwt() -> 'app_metadata' ->> 'role'` (not profiles table) to avoid circular lookups.

### Auth API routes (all live)

| Route | Implementation |
|---|---|
| `POST /api/auth/check-email` | Checks `auth.users` via admin client + queries `approved_signups` |
| `POST /api/auth/sign-in` | `supabase.auth.signInWithPassword`; role from `app_metadata`; redirects student to `/checkout` if no package |
| `POST /api/auth/create-account` | Creates user via admin client with `app_metadata: { role }` + inserts `profiles` row with first/last name + phone; creates `mentor_profiles` row if mentor |
| `POST /api/auth/sign-out` | `supabase.auth.signOut` |
| `POST /api/auth/change-password` | `supabase.auth.updateUser({ password })` — used by all three account pages |
| `POST /api/auth/reset-password` | `supabaseAdmin.auth.admin.generateLink` (type: `recovery`) — generates reset URL silently without Supabase sending its own email; then calls `sendPasswordReset` from `lib/email.ts` to send the branded Resend email |
| `POST /api/auth/update-password` | `exchangeCodeForSession(code)` then `updateUser({ password })` — PKCE flow |

### Middleware (route protection)

`middleware.ts` at project root protects all dashboard routes server-side:
- `/dashboard/*` → student only
- `/mentor/*` → mentor only
- `/admin/*` → admin only
- `/checkout` → student with no package only
- `/login` → redirects to dashboard if already authenticated

### Account pages (all load/save real Supabase data)

- `app/dashboard/account/page.tsx` — loads + saves `first_name`, `last_name`, `phone` from `profiles`
- `app/mentor/account/page.tsx` — loads + saves `profiles` + `mentor_profiles` in parallel
- `app/admin/account/page.tsx` — loads + saves `first_name`, `last_name` from `profiles`

### Dashboard header

`components/dashboard/DashboardLayout.tsx` — fetches the logged-in user's name and initials from Supabase on mount. No hardcoded names anywhere. Falls back to email prefix if name is not set.

### Account creation flow

`components/auth/LoginForm.tsx` — when a first-time allowlist user signs up, the form collects: first name, last name, phone, new password, confirm password. All fields are saved to the `profiles` row on creation.

### Seed data

`supabase/seed.sql` pre-inserts admin accounts into `approved_signups`:
- `rdnradin@yahoo.com` — admin
- `sepantayalameha2006@gmail.com` — admin

Test accounts also added:
- `mentor@prepmate.ca` — mentor
- `student@prepmate.ca` — student

### Known fixes applied (April 2026)

- **`@supabase/ssr` not installed locally:** `package.json` lists it but a fresh clone may need `npm install` to pull it in. Run `npm install` if you see `Module not found: Can't resolve '@supabase/ssr'` on startup.
- **LoginForm create-account placeholders:** first name and last name input placeholders were accidentally set to Radin's name. Fixed to `Jane` / `Smith`.
- **Dashboard account page phone field:** `select("first_name, last_name, phone")` fails silently if migration 002 hasn't been applied yet, causing phone to show empty. Fixed to `select("*")` so the query always succeeds regardless of migration state. Also falls back to `full_name` for the name display if `first_name`/`last_name` are not yet in the schema.

---

### Pending Supabase dashboard config (one-time, manual)

In Supabase → Authentication → URL Configuration:
- **Site URL:** `http://localhost:3000` (change to production domain before launch)
- **Redirect URLs:** add `http://localhost:3000/login/reset-password`

---

## 15. Completed Backend Work (branch: `backend/admin`)

Everything below is built, live, and working as of April 2026. Build passes with zero TypeScript errors.

### Database migration applied

- `ALTER TABLE profiles ADD COLUMN IF NOT EXISTS mentor_notes text;` — run manually in Supabase SQL Editor (April 2026).

### Admin API routes (all live)

| Route | Implementation |
|---|---|
| `POST /api/admin/approve-consultation` | Marks consultation `approved`, upserts email into `approved_signups` with role `student`, logs to `admin_actions` |
| `POST /api/admin/decline-consultation` | Marks consultation `declined`, logs to `admin_actions` |
| `POST /api/admin/match-mentor` | Writes `mentor_id` into student's `profiles` row, logs to `admin_actions` |
| `POST /api/admin/approved-signups` | Adds email + role to `approved_signups`; returns 409 on duplicate |
| `DELETE /api/admin/approved-signups/[id]` | Removes row from `approved_signups`, logs to `admin_actions` |

### Profile + notes API routes (all live)

| Route | Implementation |
|---|---|
| `PUT /api/student/profile` | Updates `first_name`, `last_name`, `full_name`, `phone` on `profiles` for the logged-in student |
| `PUT /api/mentor/profile` | Updates `school`, `program`, `year`, `bio` on `mentor_profiles` for the logged-in mentor |
| `POST /api/mentor/students/[id]/notes` | Saves `mentor_notes` to the student's `profiles` row; caller must be the assigned mentor or an admin |

### Admin pages (all now fetch real Supabase data)

Every admin page was converted from a `"use client"` component with hardcoded mock arrays to a server component + interactive client child:

| Page | Server component | Client component |
|---|---|---|
| `/admin` | `app/admin/page.tsx` | — (no interactivity needed) |
| `/admin/students` | `app/admin/students/page.tsx` | `components/admin/StudentsClient.tsx` |
| `/admin/students/[id]` | `app/admin/students/[id]/page.tsx` | `components/admin/StudentDetailActions.tsx` |
| `/admin/mentors` | `app/admin/mentors/page.tsx` | `components/admin/MentorsClient.tsx` |
| `/admin/mentors/[id]` | `app/admin/mentors/[id]/page.tsx` | — |
| `/admin/matching` | `app/admin/matching/page.tsx` | `components/admin/MatchingClient.tsx` |
| `/admin/consultations` | `app/admin/consultations/page.tsx` | `components/admin/ConsultationsClient.tsx` |
| `/admin/approved-signups` | `app/admin/approved-signups/page.tsx` | `components/admin/ApprovedSignupsClient.tsx` |

### Shared helpers

- `lib/admin-helpers.ts` — exports `logAdminAction(adminId, actionType, targetId, details?)`. Uses try/catch so a failed audit log never blocks the admin action.
- `types/admin.ts` — shared TypeScript interfaces: `StudentRow`, `MentorRow`, `ConsultationRow`, `ApprovedSignupRow`, `AdminActionRow`, `SessionRow`. All field names match the actual DB schema.

### Dashboard hours page fix

`app/dashboard/hours/page.tsx` is now a server component that derives the student's real hourly rate via `getPackageBySlug()` from `lib/packages.ts` (falls back to `$85/hr` if package not found). Interactive UI moved to `components/dashboard/BuyHoursClient.tsx`.

### Remaining admin tasks — complete after other branches

These tasks were intentionally left out of this branch because they depend on Stripe and Resend. Come back to these once those branches are merged into `main`:

**After Branch 2 (`backend/payments`) is merged:**
- Build `POST /api/admin/refund` — call Stripe refund API, mark student inactive in `profiles`, log to `admin_actions`
- Wire `/admin/payments` page to real data — revenue tab from Stripe transactions, payouts tab from `payouts` table, refunds tab from Stripe refunds

**After Branch 4 (`backend/email`) is merged:**
- ✅ `sendAccountApproved` wired into `POST /api/admin/approve-consultation`
- ✅ `sendMatchIntro` wired into `POST /api/admin/match-mentor` (fetches student + mentor profiles, sends two separate emails with full briefing)
- Build `POST /api/admin/payouts/[id]/mark-paid` — set `payouts.status = paid`, call `sendMentorPayoutNotice`, log to `admin_actions`

---

## 16. Completed Backend Work (branch: `backend/email`)

Everything below is built, live, and working as of April 2026. Build passes with zero TypeScript errors.

### Resend setup

- `resend` npm package installed
- `RESEND_API_KEY` set in `.env.local` ✅
- All emails currently send from `onboarding@resend.dev` (Resend test sender) — switch to `hello@prepmate.ca` once domain is verified (see "Required before launch" block below)

### lib/email.ts — all 11 email functions

| Function | Signature | Wired into route |
|---|---|---|
| `sendConsultationConfirmation` | `(prospect, slot, zoomUrl)` | ⬜ Wire into `POST /api/consultation` (branch 3) |
| `sendAdminConsultationNotification` | `(prospect, slot)` | ⬜ Wire into `POST /api/consultation` (branch 3) |
| `sendAccountApproved` | `(email)` | ✅ `POST /api/admin/approve-consultation` |
| `sendMentorInvite` | `(email)` | ✅ `POST /api/admin/approved-signups` (mentor role only) |
| `sendMatchIntro` | `(student, mentor)` | ✅ `POST /api/admin/match-mentor` — sends two separate emails: student gets congrats + mentor details; mentor gets student details + package phases |
| `sendPackageReceipt` | `(student, packageName, amountCents)` | ⬜ Wire into Stripe webhook (branch 2) |
| `sendAddOnHoursReceipt` | `(student, hours, hourlyRate, amountCents)` | ⬜ Wire into `POST /api/hours/checkout` success (branch 2) |
| `sendSessionConfirmation` | `(student, mentor, session)` | ⬜ Wire into `POST /api/sessions` (branch 3) |
| `sendSessionCancelled` | `(cancelledBy, notifyParty, otherParty, session)` | ⬜ Wire into `POST /api/sessions/[id]/cancel` (branch 3) |
| `sendMentorPayoutNotice` | `(mentor, amount, period)` | ⬜ Wire into `POST /api/admin/payouts/[id]/mark-paid` (branch 2) |
| `sendPasswordReset` | `(email, resetUrl)` | ✅ `POST /api/auth/reset-password` — uses `supabaseAdmin.auth.admin.generateLink` to get the reset URL silently, then sends branded Resend email |

### lib/packages.ts additions

- Added `PackagePhase` interface and `phases` array to each package (all phases from the product spec)
- Added `getPackageByName(name)` helper — used by `sendMatchIntro` to look up phases from the student's `package_name` profile field

### Password reset route change

`app/api/auth/reset-password/route.ts` now uses `supabaseAdmin.auth.admin.generateLink` instead of `supabase.auth.resetPasswordForEmail`. This means Supabase does **not** send its own email — only the branded Resend email goes out. The security behaviour (always return success, never reveal if an email is registered) is preserved.

### ⬜ Required before launch: verify prepmate.ca in Resend and switch FROM address

**Current state:** `lib/email.ts` sends from `onboarding@resend.dev` (Resend's test sender). This only delivers to the Resend account owner's email — it cannot reach students, mentors, or any other recipient.

**Steps to fix (one-time, ~30 minutes):**

1. **In Resend dashboard → Domains → Add Domain**
   - Enter `prepmate.ca`
   - Resend generates 3 DNS records: two DKIM TXT records and one SPF TXT record

2. **In your DNS provider** (wherever prepmate.ca nameservers are managed — Cloudflare, GoDaddy, Namecheap, etc.)
   - Add all 3 records exactly as Resend shows them
   - The DKIM records go on subdomains like `resend._domainkey.prepmate.ca`
   - The SPF record goes on the root `prepmate.ca` (or merges with an existing SPF record if one exists)

3. **Back in Resend → Domains → click Verify**
   - DNS propagation takes 5–30 minutes
   - Status will flip to "Verified" with green SPF and DKIM badges

4. **In `lib/email.ts`, update the FROM constant (one line):**
   ```typescript
   // Change this:
   const FROM = 'PrepMate <onboarding@resend.dev>'
   // To this:
   const FROM = 'PrepMate <hello@prepmate.ca>'
   ```
   Also remove the TODO comment on the line above it.

5. **Restart the dev server** and re-run the test endpoint to confirm emails arrive from `hello@prepmate.ca`:
   ```
   http://localhost:3000/api/dev/test-emails?to=your@email.com
   ```
   Once the domain is verified, `to` can be any email address — not just the Resend account email.

6. **In Resend dashboard → Emails**, confirm all 13 test emails show status "Delivered".

---

## 17. Troubleshooting

### Git object store corruption (`signal 10` / `unpack-objects failed`)

**Symptoms:** Any git operation (`fetch`, `pull`, `gc`) fails with:
```
error: rev-list died of signal 10
error: unpack-objects died of signal 10
fatal: unpack-objects failed
```

**Cause:** macOS corrupted the git pack files — usually triggered by the disk sleeping mid-write, Time Machine backing up `.git/objects` while git is writing to them, or memory pressure during a large fetch.

**Fix (takes ~30 seconds, no code is lost as long as you push regularly):**

```bash
# From the project root
rm -rf .git
git init
git remote add origin https://github.com/rdadvaran/Prepmate-Web-App.git
git fetch --depth=1 origin main
git reset --hard origin/main
```

This wipes only the `.git` directory — all source files stay untouched. The working tree is then reset to match the latest remote commit.

**Prevention:**
- System Settings → Battery → disable "Put hard disks to sleep when possible"
- System Settings → General → Time Machine → Options → add the project folder to the exclusion list (Time Machine and git pack writes conflict)
- Push frequently so the remote is always the source of truth
