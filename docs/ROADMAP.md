# Rease — Development Roadmap & Task Backlog

**Categories:** Backend · Database · Mobile · Web/Admin · DevOps · UI/Design · Integrations · QA · Ops (Legal/Business)

Each milestone is a Notion **Milestone**. Each task below is one Notion **Task**, moving through your pipeline: Product Backlog → Sprint Backlog → To Do → In Progress → Code Review (PR) → Merged to Dev → Done. 

## Table of Contents

- Milestone 0: Project Foundation  
- Milestone 1: Identity & Onboarding  
- Milestone 2: Billing Core  
- Milestone 3: Payments  
- Milestone 4: Maintenance & Communication  
- Milestone 5: Admin Panel MVP  
- Milestone 6: Marketing Website  
- Milestone 7: Polish, QA & Compliance  
- Milestone 8: Launch

---

# Milestone 0: Project Foundation

### Initialize Repositories — *DevOps*

Set up version-controlled codebases for every part of the project.

- Create backend, mobile (Expo), and web/admin (Next.js) repos, or one monorepo.  
- Add a README to each with local setup steps.  
- Set up branch protection: `main` and `develop` require PR review, no direct commits.

### Write Contributor & Onboarding Guide — *DevOps* 

Give any new team member a single doc that gets them running locally. 

- Write a `CONTRIBUTING.md` covering how to clone, install, run, and submit a PR.  
- Document the branch naming convention (`feature/*`, `fix/*`) and commit style with real examples.  
- Add PR and issue templates so descriptions stay consistent across the team.

### Set Up Docker for Local Development — *DevOps*

Package the backend so every dev's local environment matches.

- Write a `Dockerfile` for the backend service, matching the Node version required by `package.json`.  
- Write a `docker-compose.yml` including the local Postgres instance.  
- Document the one-command startup in the README.

### Configure Environment Variables & Secrets — *DevOps*

Separate local, staging, and production configuration safely.

- Create `.env.example` files for backend, mobile, and web.  
- Set up staging and production secrets in the hosting platform (Vercel/Railway), never in Git.  
- Confirm `.env` is in `.gitignore` across every repo.

### Register OAuth Applications — Devops

Get social login credentials set up before any auth code is written. 

- Register a Google Cloud OAuth client (web, iOS, and Android configs).  
- Register an Apple Sign in with Apple service ID and key.  
- Register a Facebook Login app in Meta for Developers.  
- Configure separate redirect URIs/client IDs for local, staging, and production.

### Configure Environment Variables & Secrets — Devops

Separate local, staging, and production configuration safely. 

- Create `.env.example` files for backend, mobile, web, and admin, including placeholders for the three OAuth client IDs/secrets and the SMS OTP provider key.  
- Set up staging and production secrets in Vercel, never in Git.  
- Confirm `.env` is in `.gitignore` across every workspace.

### Set Up CI/CD Pipeline — *DevOps*

Automate checks and deployments.

- Add a GitHub Actions workflow that runs lint \+ tests on every PR.  
- Auto-deploy the `dev` branch to a staging environment (Vercel monorepo deployment).  
- Block merges to `dev`/`main` if checks fail.

### Set Up Testing Framework — *Backend* 

Give the team a way to actually write the tests the CI pipeline runs. 

- Install a test runner (Jest or Vitest) in the backend.  
- Write one unit test and one integration test to establish the pattern before Milestone 1 work starts.  
- Document how to run tests locally in the README.

### Set Up API Documentation — *Backend* 

Keep the API contract in sync with what's actually built. 

- Install Swagger/OpenAPI tooling for NestJS.  
- Auto-generate docs from route decorators as endpoints are built, rather than hand-maintaining a separate doc.

### Set Up Error Tracking & Logging — *DevOps*

Get visibility into failures as they happen.

- Install Sentry (or similar) in the backend, mobile app, and web app.  
- Replace `console.log` with a structured logger in the backend.  
- Confirm a test error shows up in the dashboard.

### Set Up API Versioning Convention — *Backend*

Protect future changes from breaking existing clients.

- Prefix all API routes with `/v1/`.  
- Document the convention in the backend README.

### Provision PostgreSQL Database — *Database*

Stand up the database for local and staging use.

- Create a Supabase Postgres project — the same platform will also provide Auth and Storage.  
- Create separate projects/databases for local, staging, and production.  
- Confirm the backend can connect to each.

### Set Up Migrations Tool — *Database*

Make schema changes trackable and reversible.

- Install Prisma or Drizzle in the backend.  
- Configure it to run migrations against local and staging.  
- Commit the first empty migration to confirm the setup works.

### Write Database Seed Script — *Database* 

Give the team and QA a fast way to get realistic test data locally. 

- Write a seed script that creates the QA test accounts, including the dual-role and PIN-reset accounts.  
- Include sample properties, units, and leases so a fresh local environment isn't empty.  
- Document the seed command in the README.

### Build Design System — *UI/Design*

Establish the shared visual language before screens get built.

- Define color palette, typography scale, and spacing units.  
- Define dark mode and light mode token sets.  
- Design a numeric PIN-pad component, used on both the PIN setup and unlock screens.  
- Design a reusable bottom-sheet component, used for the Overdue, Collected, and Revenue Breakdown previews.  
- Document components (buttons, inputs, cards) in Figma

### Low-Fidelity Wireframes — *UI/Design*

Rough out every screen before high-fidelity design work starts.

- Wireframe all Global Entry screens (splash, carousel, social sign-in, phone verification, PIN setup, PIN/biometric unlock, role selection).  
- Wireframe the full Landlord Journey (Product Overview §5.4): Home with the three bottom-sheet drill-downs, Buildings List, Units List, Unit Detail, Tenant Profile, Invoice Detail, Ticketing Kanban, Ticket Detail, Profile & Settings.  
- Wireframe the full Tenant Journey (§5.5): Home, Payment Checkout, Expense Tracker, Invoice Detail, Support/Ticketing, Ticket Detail & Chat, Profile & Settings.  
- Wireframe the full Admin Panel (§5.6): Login, Global Dashboard, User Management \+ User Detail, Property Oversight \+ Property Detail, Subscription & Revenue Control, Dispute Oversight, Payment Health Monitor, Landlord Verification Queue, Platform Broadcasts, System Users.  
- Wireframe the Overdue, Collected, and Revenue Breakdown bottom sheets as their own reusable modal components, not full screens.

---

# Milestone 1: Identity & Onboarding

*Goal: a user can sign in with a social account, set a PIN, choose a role, and either create a property or join one via a code.* 

### Core Schema Migration — *Database*

Create the foundational tables everything else depends on.

- Create `users` (shared identity — OAuth provider, phone, PIN hash, no password field), `landlord_profiles`, `tenant_profiles`, `properties`, `units`, `leases` tables.  
- Add Row-Level Security policies partitioned by `landlord_profile_id` on landlord data and `tenant_profile_id` on tenant data.  
- Run the migration against local and staging.

### OAuth Sign-In Endpoints — *Backend* 

Let users sign in with Google, Apple, or Facebook — no password grant. 

- Configure Supabase Auth with Google, Apple, and Facebook providers, and confirm the email/password grant is disabled.  
- Build `POST /v1/auth/oauth/callback` to create or match a `users` row from the provider's profile.  
- Confirm a returning user's second sign-in matches the existing row instead of creating a duplicate.

### Phone Verification Endpoints — *Backend* 

Verify a phone number once, right after first sign-in. 

- Integrate Semaphore for SMS OTP delivery.  
- Build `POST /v1/auth/phone/send-otp` and `POST /v1/auth/phone/verify-otp`.  
- Rate-limit OTP requests per phone number to prevent SMS-bombing abuse.

### PIN Endpoints — *Backend* 

Let a user set and verify a 6-digit PIN for fast, passwordless re-entry. 

- Build `POST /v1/auth/pin/setup`, storing the PIN as a salted hash.  
- Build `POST /v1/auth/pin/verify`, gated behind a valid device-bound refresh token.  
- Build `POST /v1/auth/pin/reset`, requiring a fresh OAuth callback or OTP verification first.  
- Apply the same rate-limiting/backoff pattern used for invite codes.

### Dev Auth Bypass (Staging Only) — *Backend* 

Let QA and CI log into seeded accounts without a real OAuth round-trip. 

- Build `POST /v1/auth/dev-login`, gated behind an environment flag that's always false in production.  
- Confirm it logs directly into a seeded test account.

### Role Profile Endpoints — *Backend* 

Let one identity hold and switch between role profiles. 

- Build `GET /v1/me/profiles` to list existing role profile(s) for the current session.  
- Build `POST /v1/me/profiles` to add the other role profile to the current identity.  
- Build `POST /v1/me/active-profile` to switch which profile is active for the session.  
- Confirm a session can only query data scoped to its currently active profile.

### Session Token Expiry & Refresh — *Backend* 

Prevent sessions from staying valid forever. 

- Set access tokens to expire after a set window (e.g., 1 hour).  
- Implement a refresh token flow bound to the device, unlocked by the PIN rather than a fresh login every time.  
- Test that an expired access token is correctly rejected and refreshed.

### CORS Configuration — *Backend* 

Restrict which apps can call the API. 

- Configure allowed origins for staging and production separately.  
- Test that an unapproved origin is blocked.  
- 

### Property & Unit Endpoints — *Backend* 

Let landlords manage their properties. 

- Build `POST /v1/properties`, `GET /v1/properties`.  
- Build `POST /v1/properties/:id/units`.  
- Enforce that a landlord profile can only see its own properties.

### Landlord KYC Endpoints — *Backend* 

Collect and store the landlord verification details needed before payouts can happen. 

- Build `POST /v1/landlord/kyc` to accept government ID, selfie, payout details, and an optional ownership document upload to Supabase Storage.  
- Build `GET /v1/landlord/kyc/status`.  
- Confirm uploaded files are encrypted at rest and not publicly accessible without a signed URL.

### Handshake (Invite Code) Logic — *Backend* 

Build the tenant-onboarding code system. 

- Generate a 6-character code excluding ambiguous characters (0/O, 1/I/l).  
- Set the code to expire 7 days after generation if unclaimed.  
- Mark a code as single-use once redeemed.  
- Build `POST /v1/units/handshake` for redemption — reject any attempt to leave a new Tenant profile without a linked unit.

### Invite Code Rate Limiting — *Backend* 

Stop the code field from being brute-forced. 

- Limit code-entry attempts to 5 per device per 15 minutes.  
- Apply exponential backoff after the limit is hit.

### Onboarding Screens — *Mobile* 

Build the shared entry flow every user goes through. 

- Splash screen with session check.  
- Welcome carousel (first-time users only).  
- Social sign-in screen (Google/Apple/Facebook buttons only, no email/password fields).  
- Phone verification screen.  
- PIN setup screen with a biometric enable toggle.  
- Role selection screen (Landlord / Tenant).

### PIN / Biometric Unlock Screen — *Mobile* 

Build the screen every returning user sees. 

- Biometric prompt first if enabled, falling back to a PIN pad on failure or after 3 failed biometric attempts.  
- Wire "Forgot PIN" to the reset flow (fresh OAuth or OTP).

### Landlord KYC & Property Setup Flow — *Mobile* 

Build the landlord's first-run experience. 

- KYC screen: government ID capture (front and back), selfie capture, payout details form, optional ownership document upload.  
- Property creation screen (name, address, base rent, due date).  
- Unit \+ invite code generation screen with copy/share actions.

### Tenant Handshake Flow — *Mobile* 

Build the tenant's first-run experience. 

- Code entry screen (blocking — no skip option).  
- Lease confirmation screen showing unit \+ rent details.  
- Confirm & bind action that calls the handshake endpoint.

### Add/Switch Role UI — *Mobile* 

Build the optional dual-role control described in the Product Overview. 

- Add "Add \[Other Role\]" / "Switch to \[Other Role\]" control to Profile & Settings on both sides.  
- Wire "Add" into the same KYC/Property Setup or Handshake flow the other role uses normally.  
- Wire "Switch" to `POST /v1/me/active-profile` and confirm it re-scopes the UI without a full re-login.

### Milestone 1 Manual Test Pass — *QA* 

Confirm the core loop works end-to-end. 

- New user: complete Google sign-in → phone verification → PIN setup → role selection → property/handshake flow, end-to-end.  
- Close and reopen the app; confirm PIN/biometric unlock works without re-triggering OAuth.  
- Test "Forgot PIN" end-to-end using the seeded PIN-reset test account.  
- Confirm rate limiting and expiry trigger correctly for both invite codes and PIN attempts.  
- Using the seeded dual-role account: add the second role profile, switch between them, and confirm each shows only its own data.

---

# Milestone 2: Billing Core

*Goal: landlord can bill, tenant can see what they owe. No real payments yet.*

### Cloud Storage Integration — *Integrations*

Wire up file uploads for the app.

- Set up a Supabase Storage bucket.  
- Build a reusable upload endpoint/helper.  
- Retrofit the Milestone 1 KYC screen to actually upload.

### Financial Schema Migration — *Database*

Add the billing tables.

- Create `invoices` table, including `partially_paid` and `under_review` statuses.  
- Create `promo_codes` table, keyed to `landlord_profile_id`.  
- Run and verify the migration on staging.

### Database Indexing Pass — *Database*

Keep queries fast as data grows.

- Add an index on `invoices.lease_id`.  
- Add an index on `units.property_id` and `properties.landlord_profile_id`.  
- Confirm dashboard queries use the indexes (check the query plan).

### Invoice Generation Engine — *Backend*

Auto-calculate what a tenant owes.

- Implement `base_rent + (current_kwh − previous_kwh) × rate`.  
- Flag for manual review instead of calculating negative cost when `current_kwh < previous_kwh`.  
- Build `POST /v1/invoices/:id/submeter` to attach a reading and photo.

### Optimistic Locking on Submeter Edits — *Backend*

Prevent two simultaneous edits from silently overwriting each other.

- Add an `updated_at` check on invoice update requests.  
- Reject the save with a conflict error if the record changed since it was loaded.

### Submeter Dispute Endpoints — *Backend*

Let tenants flag a questionable reading.

- Build `POST /v1/invoices/:id/dispute` (48-hour window from invoice issuance).  
- Build `POST /v1/invoices/:id/dispute/resolve` for the landlord.  
- Log every resolution to `audit_log`.

### Landlord Adaptive Dashboard & Submeter Entry — *Mobile* 

Build the landlord's billing screens so they scale with portfolio size. 

- Submeter reading entry with required photo capture.  
- Financial dashboard: Total Expected, Collected, Overdue.  
- Implement the three layout states: single-building/few-units shows units directly; single-building/many-units shows a summary card above a unit list; multi-building shows portfolio totals with a building switcher.  
- Implement the Overdue, Collected, and (multi-building only) Revenue Breakdown bottom sheets, each deep-linking into Tenant Profile.  
- Confirm the same API responses drive all three layout states — this is front-end layout logic, not separate endpoints.

### Properties Navigation & Detail Screens — *Mobile* 

Build the drill-down screens landlords use to manage their portfolio. 

- Buildings List screen (multi-building landlords) and Units List screen (single-building landlords land here directly from the Properties tab).  
- Unit Detail screen with a tappable tenant summary card, or a "Generate Invite Code" button if vacant.  
- Tenant Profile screen: contact info, lease terms, invoice ledger, submeter history, ticket history.  
- Invoice Detail screen, built as a shared component reused by both the landlord and tenant apps (read-only for tenants).  
- Confirm dashboard bottom-sheet taps (from the task above) correctly deep-link into Tenant Profile, bypassing Buildings List/Units List.

### Tenant Invoice View — *Mobile* 

Build the tenant's billing screen. 

- "Next Payment Due" card with rent \+ utility breakdown.  
- Tapping the card opens the shared Invoice Detail screen; show a "Dispute this reading" action if a submeter charge is within its 48-hour dispute window.

### Milestone 2 Manual Test Pass — *QA*

Confirm billing works correctly.

- Landlord issues an invoice with a submeter charge.  
- Tenant sees the correct total.  
- Trigger a dispute and confirm it moves to `under_review`.  
- Test the adaptive dashboard against at least two portfolio shapes.  
- Confirm a dashboard bottom-sheet tap correctly deep-links into the right Tenant Profile.

---

# Milestone 3: Payments

*Goal: tenants can actually pay.*

### Payment Gateway Sandbox Setup — *Integrations*

Get test payments working before writing backend logic.

- Create a PayMongo account and get sandbox keys.  
- Manually test a GCash/Maya sandbox transaction in their dashboard.

### Payment Checkout Endpoint — *Backend*

Create a payable session for an invoice.

- Build `POST /v1/payments/checkout`.  
- Return the gateway's checkout URL/session to the client.

### Webhook Receiver — *Backend*

Safely process payment confirmations.

- Build `POST /v1/webhooks/paymongo`.  
- Verify every incoming request against PayMongo's signing secret.  
- Store each `gateway_event_id` and ignore duplicate deliveries.  
- Handle `partially_paid`, `paid`, and `payment_failed` states correctly.

### SQL Injection Review Pass — *Backend*

Confirm queries are safe by construction.

- Review every raw query (if any) for string concatenation.  
- Confirm the ORM's parameterized queries are used everywhere, especially on payment and search endpoints.

### Nightly Reconciliation Job — *Backend*

Catch any webhook that never arrived.

- Build a scheduled job comparing the gateway's settlement report to the local `payments` table.  
- Alert (via Sentry or email) if a mismatch is found.

### Payment Checkout Screen — *Mobile*

Build the tenant's "Pay Now" flow.

- Wire the checkout button to the gateway's native UI.  
- Handle success, failure, and cancellation states.

### Expense Tracker Screen — *Mobile*

Build the tenant's budgeting view.

- Chart of historical rent and utility payments.  
- "+" button to manually log an external payment with a receipt photo.  
- Tapping a past payment row opens the shared Invoice Detail screen (read-only). 

### Milestone 3 Manual Test Pass — *QA*

Confirm money moves correctly.

- Pay a sandbox invoice via GCash test mode; confirm it's marked "Paid."  
- Manually resend the same webhook event and confirm it's NOT double-credited.  
- Test a partial payment and confirm the invoice shows `partially_paid`.

---

# Milestone 4: Maintenance & Communication

### Ticketing Schema Migration — *Database*

Add the tables for maintenance requests.

- Create `tickets` table.  
- Create `ticket_messages` table, keyed to `sender_user_id` so either role profile can post to the thread.

### Ticketing Endpoints — *Backend*

Build the maintenance request system.

- Build `POST /v1/tickets`, `PATCH /v1/tickets/:id/status`.  
- Build `POST /v1/tickets/:id/messages` for the chat thread.  
- Build `POST /v1/tickets/:id/close-confirm` with a 1–5 rating.

### Auto-Close Inactive Tickets — *Backend*

Prevent stale tickets from cluttering dashboards.

- Build a cron job that flags tickets inactive for 7+ days as "Closed due to inactivity."

### XSS Sanitization on Messages — *Backend*

Keep chat/ticket text safe to display.

- Sanitize all user-submitted text before storing or rendering it.  
- Test with a script-tag input to confirm it's neutralized.

### Landlord Ticketing View — *Mobile*

Build the landlord's maintenance dashboard.

- Kanban board: Open / Resolved by Me / Closed Confirmed.  
- Ticket detail screen with the chat thread.  
- Ticket Detail includes a link back to the tenant's Tenant Profile. 

### Tenant Support & Ticketing Screens — *Mobile*

Build the tenant's maintenance flow.

- Ticket list (active \+ past).  
- Submission screen (text \+ photo).  
- Chat thread \+ "Confirm Resolution" with star rating.

### Push Notification Integration — *Mobile*

Wire up real-time alerts.

- Integrate Expo push notifications.  
- Wire every event from the notification matrix (rent reminders, payment confirmations, ticket updates, disputes, announcements, OTP requests, KYC status changes), routed to the correct role profile.

### Milestone 4 Manual Test Pass — *QA*

Confirm the ticketing loop works.

- Submit a ticket, reply as landlord, confirm resolution, and rate it.  
- Leave a test ticket inactive and confirm the auto-close cron fires.

---

# Milestone 5: Admin Panel MVP

### Compliance Schema Migration — *Database*

Add the internal tooling tables.

- Create `audit_log` table, keyed to `actor_user_id`.  
- Create `system_users` table with a `permission_level` field.

### Admin Panel Auth — *Backend*

Keep admin access fully separate from customer logins.

- Build a standalone email/password login for the admin panel, deliberately not tied to the consumer OAuth+PIN flow.  
- Enforce permission levels (full / support / finance-only) on every admin route.

### CSRF Protection on Admin Forms — *Backend*

Protect the admin panel's form submissions.

- Add CSRF tokens to all state-changing admin panel requests.

### Admin API Endpoints — *Backend*

Build the internal tools.

- `GET /v1/admin/users`, `PATCH /v1/admin/users/:id/suspend`.  
- `GET /v1/admin/disputes`, `GET /v1/admin/payments/health`.  
- `POST /v1/admin/system-users`.  
- `PATCH /v1/admin/landlords/:id/verify` for optional ownership document review.

### Lease Termination Endpoint — *Backend*

Handle move-outs correctly.

- Build `POST /v1/leases/:id/terminate`.  
- Auto-generate a prorated final invoice based on the move-out date.

### Admin Web UI Build — *Web/Admin*

Build the internal dashboard.

- Global Dashboard with clickable metric cards routing into User & Account Management.  
- User & Account Management screen and User Detail screen (role profiles, KYC documents, suspend/unlock, audit trail).  
- Property & Unit Oversight screen and Property Detail screen.  
- Subscription & Revenue Control screen.  
- Dispute Oversight screen with a read-only thread view and escalate/flag note field.  
- Payment Health Monitor screen.  
- Landlord Verification Review Queue with document viewer and Approve/Reject actions.  
- Platform Broadcasts compose screen with send history.  
- System Users & Permissions screen with an invite form.

### Milestone 5 Manual Test Pass — *QA*

Confirm internal tools work and stay isolated.

- Log into the admin panel with a `system_users` account.  
- Suspend a test account and confirm they're locked out of every role profile they hold.  
- Approve a test landlord's ownership document and confirm the badge appears.  
- Confirm a landlord/tenant login cannot reach any admin route.

---

# Milestone 6: Marketing Website

### Homepage Build — *Web/Admin*

Build the primary landing page.

- Hero section with value prop and app mockups.  
- Feature highlights section.  
- Social proof \+ CTA sections.

### Features Page Build — *Web/Admin*

Detail the product for prospective landlords.

- Sections for invoicing, submetering, expense tracking, ticketing.

### Pricing Page Build — *Web/Admin*

Show the subscription tiers.

- Free, Pro, and Enterprise tier cards with feature comparisons.

### FAQ & Contact Page Build — *Web/Admin*

Handle pre-sales questions.

- FAQ accordion.  
- Contact/consultation booking form.

### Marketing Site Visual Design — *UI/Design*

Apply the design system to the public site.

- High-fidelity comps for homepage, features, pricing, FAQ.

---

# Milestone 7: Polish, QA & Compliance

### Full Regression Test Pass — *QA*

Walk every flow before launch.

- Test every screen against all 4 QA test accounts.  
- Log any bugs found into the Bug Tracker.

### Staging Smoke Test Checklist — *QA*

Final pre-deploy sanity check.

- Write a checklist covering auth, handshake, billing, payments, tickets, admin panel.  
- Run it against staging before every release candidate.

### Enable Automated Database Backups — *DevOps*

Protect against data loss.

- Turn on daily automated backups in Supabase/Neon.  
- Confirm a backup can actually be restored.

### Set Up Custom Domain — *DevOps*

Move off default hosting URLs.

- Point your domain at the marketing site.  
- Point an API subdomain at the backend.

### App Store & OAuth Review Prep — *Mobile* 

Confirm the social login flow is fully review-ready before submission. 

- Confirm Sign in with Apple is fully functional, required alongside Google/Facebook per Apple's guidelines.  
- Verify Google and Facebook OAuth consent screens show accurate app name, branding, and privacy policy links.

### Final High-Fidelity Design Polish Pass — *UI/Design*

Replace any placeholder UI.

- Review every screen against the design system.  
- Fix inconsistent spacing, colors, and typography.

### Business Registration for Live Payments — *Ops*

Unlock real money collection.

- Complete SEC/DTI registration.  
- Complete BIR registration.  
- Submit both to PayMongo/Xendit for live account activation.

### Data Protection Officer Designation — *Ops*

Meet Data Privacy Act obligations.

- Assign a DPO within the team.  
- Document the role internally.

### Privacy Policy & Terms of Service Publication — *Ops*

Required before collecting real user data.

- Draft (or have drafted) both documents.  
- Publish them on the marketing site and link from the app.

### IP Assignment Agreements Signed — *Ops*

Protect company ownership of the codebase.

- Have all 5 team members sign IP assignment agreements.

---

# Milestone 8: Launch

### Switch to Live Payment Gateway Keys — *DevOps*

Move from sandbox to real transactions.

- Replace PayMongo sandbox keys with live keys in production environment variables.  
- Run one real, small test transaction to confirm.

### Switch OAuth Apps to Production Mode — *DevOps* 

Move all three social providers out of testing/development status. 

- Move Google, Apple, and Facebook OAuth apps to full production/live status — each has its own review step, so start ahead of your target launch date. 

### Cost & Usage Monitoring Setup — *DevOps* 

Avoid billing surprises. 

- Check current usage against Vercel/Supabase free-tier limits.  
- Set up a usage alert if available.

### App Store Submission — *Ops* 

Get the iOS app live. 

- Prepare screenshots, description, and privacy details.  
- Submit for App Store review.

### Play Store Submission — *Ops* 

Get the Android app live. 

- Prepare store listing assets.  
- Submit for Play Store review.

### Marketing Site Deployment — *Ops* 

Go live publicly. 

- Deploy the finished site to production.  
- Confirm the custom domain resolves correctly.

### Pilot Landlord Onboarding — *Ops* 

Soft-launch before a wide release. 

- Identify 2–3 pilot landlords.  
- Walk them through onboarding personally and gather feedback before opening more broadly.

