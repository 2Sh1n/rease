# Rease — Product Overview & Technical Specification

## 1. Product Overview

### 1.1 Problem Statement

Landlords in the Philippines currently manage tenant billing and property maintenance through fragmented, informal tools — Messenger threads, manual spreadsheets, and physical cash tracking. This creates missed payments, lost maintenance requests, and heavy administrative overhead for what should be routine operations.

### 1.2 Target Users

**Landlord (Property Manager)** — Manages one or more units or buildings. Portfolio size varies widely: a single house with one tenant, one building with several tenants, or multiple buildings with many tenants. Needs clear visibility into revenue, pending payments, and open maintenance issues at whatever scale their portfolio sits at, and values automation that prevents disputes.

**Tenant (End User)** — Rents a single unit. Wants a frictionless way to pay rent, track payment history for personal budgeting, and report issues without having to chase the landlord down.

**Dual-Role User** — A person signs up as either a Landlord or a Tenant, the same as any other user. But Landlord and Tenant aren't locked account types — they're role profiles attached to one identity. From Profile & Settings, a user can add the other role profile at any time, and if they hold both, they can switch between the two views with a single tap. Their login (Google/Apple/Facebook identity + app PIN) is shared; their financial data, properties, and leases are kept fully separate per role.

### 1.3 Value Proposition

A single unified mobile application that centralizes property operations for landlords and gives tenants a frictionless financial and communication portal, with fast, passwordless entry modeled on the PIN + social-login pattern Filipino users already trust from GCash and Maya. It offers role-based views for each side, with the option for one person to hold both roles under one identity.

### 1.4 Product Vision

Build a large, verified database of rental units by becoming the operational tool landlords and tenants rely on daily — laying the groundwork for a future SaaS-enabled rental marketplace.

### 1.5 Business Model

- **SaaS subscription** — Freemium for landlords (e.g., free up to 3 units, paid tier for 4+).
- **Convenience fees** — Small platform fee added to the tenant's bill when paying in-app.
- **Gateway commissions** — Percentage of transaction volume processed through PayMongo.

### 1.6 Market & Competitive Analysis

|  |  |
|---|---|
| **Competitors** | Manual ledgers, Google Sheets, Facebook Messenger, Western SaaS (Buildium, AppFolio) |
| **Manual methods** | Free but unscalable |
| **Western SaaS** | Robust, but lacks Philippine localization (GCash, QR Ph, manual Meralco tracking) and typically still relies on email/password login rather than the OAuth + PIN pattern PH users already know |
| **Our differentiation** | Hyper-localized for the Philippine market: built-in e-wallet integrations, submeter photo verification, optional dual-role switching, and a login experience (social sign-in + MPIN) that matches GCash/Maya conventions instead of a Western-style password form |

### 1.7 Lean Canvas Summary

**Problem:** Fragmented tracking of rent and maintenance. **Solution:** Automated billing and ticketing. **Unfair advantage:** Hyper-localization + familiar PIN-based UX. **Key metrics:** Active handshakes, Gross Transaction Value.

---

## 2. Product Requirements & Scope

### 2.1 Product Requirements Document (PRD)

- **What are we building?** A 3-tier, multi-tenant mobile app, web dashboard, and internal admin panel.
- **Who is it for?** Super Admins/internal staff, Landlords, and Tenants.
- **What problem does it solve?** Automates rent collection, tracks utility expenses, and formalizes maintenance requests.
- **What does success look like?** 85% of invited tenants download the app; 30% reduction in late payments for participating landlords.

### 2.2 MVP Scope

Social sign-in with PIN-based re-entry, property setup, tenant handshakes, PayMongo e-wallet payments, photo-verified submetering, double-verification ticketing, optional dual-role switching, and an internal admin panel scoped to §3.2.1 (MVP Admin Panel).

### 2.3 Out of Scope (Post-MVP)

- Public marketplace for landlords to list vacant units.
- Tenant-facing search engine for browsing new listings.
- Full CMS/marketing-ops admin panel tooling (§3.2.2).
- Landlord property-ownership document verification badge (§3.1) — collected optionally at MVP, formally reviewed by admin staff post-MVP.

### 2.4 Sample Feature Specification Format

- **Feature:** The "Handshake" (Tenant Onboarding)
- **User:** Landlord & Tenant
- **Flow:** Landlord creates a unit → System generates a code → Tenant enters the code → System links tenant to unit.
- **Acceptance criteria:** Tenant cannot join a unit without a valid, unused, unexpired code (see §4.1 for the full security spec).

---

## 3. Feature Specifications

### 3.1 Core System & Onboarding (All Roles)

- **Social sign-in only, no username/password.** Users continue with Google, Apple, or Facebook. There is no email/password registration form anywhere in the app. Apple's App Store review guidelines require Sign in with Apple to be offered whenever other third-party social logins are present, so all three are launched together, not staggered.
- **Phone number capture, once, after first sign-in.** Immediately after the very first social login, the app asks for a mobile number and verifies it via SMS OTP. This number is used for MPIN recovery, SMS rent reminders, and payout matching for e-wallet methods — it is collected once and shared across both role profiles if the user later adds a second one.
- **MPIN setup, once, after phone verification.** The user sets a 6-digit PIN. From then on, opening the app prompts for the PIN (not another Google/Apple/Facebook login) — the same pattern as GCash's MPIN and Maya's lock-screen code. Biometric login (FaceID/fingerprint) can be enabled as a faster alternative to typing the PIN, and always falls back to the PIN if biometric verification fails.
- **PIN recovery.** "Forgot PIN" re-authenticates the user through their original social provider (or a fresh SMS OTP to their verified number) and then lets them set a new PIN — mirroring GCash's "reset via OTP or face verification" flow.
- **Role selection, right after onboarding.** Once the phone and PIN are set, the app asks "I manage a property" (Landlord) or "I'm renting a place" (Tenant). Choosing Landlord leads into KYC (§3.3); choosing Tenant requires a valid invite code before the user can proceed any further into the app (§4.1) — a tenant profile with no linked unit isn't allowed to exist past this screen.
- **Optional second role, added later.** From Profile & Settings, a user can add the role they didn't start with. Once both exist, the same menu shows "Switch to Landlord" / "Switch to Tenant."
- **Multi-theme support** — Dark Mode and Light Mode toggles.
- **Note:** The internal Admin Panel (§3.2) uses a fully separate, traditional email/password login for internal staff — see §6.2 for why it's intentionally not part of this consumer-facing flow.

### 3.2 Super Admin / Internal Admin Panel

This is company/developer-only tooling — landlords and tenants never see it. Structurally, this is a CMS-style admin dashboard in the same family as multi-vendor marketplace admin panels like eDemand: a central console where internal staff manage every side of the platform — accounts, money, disputes, content — from data-rich tables and charts rather than raw database access. Split into what you need at launch versus what can wait.

#### 3.2.1 MVP Admin Panel (build alongside the core app)

| Feature | Description |
|---|---|
| Global Dashboard | Platform-wide metrics with charts: total active landlords, total tenants, total Gross Transaction Value, revenue trend over time. |
| User & Account Management | Search accounts, view each user's role profile(s), view KYC status and uploaded documents, suspend an account, manually unlock a locked-out user. |
| Property & Unit Oversight | Read-only visibility into all properties/units across all landlords, for support purposes. |
| Subscription & Revenue Control | Manage SaaS plans, landlord billing, commission rates, convenience fees. |
| Dispute Oversight | Visibility into disputed submeter invoices (§4.2) and stalled tickets, for escalations a landlord can't resolve alone. |
| Payment Health Monitor | Surface webhook failures or reconciliation mismatches from §4.4, so a missed payment doesn't go unnoticed. |
| Platform Broadcasts | Push global announcements (e.g., "Server maintenance tonight") to all users. |
| System Users & Permissions | Add internal team members with scoped access levels (§6.2). |

#### 3.2.2 Post-MVP Admin Panel (defer until you have real usage/pain points)

| Feature | Description |
|---|---|
| Landlord Verification Review Queue | Manually review optional property-ownership documents and grant a "Verified Landlord" badge. |
| CMS / Landing Page Editor | Edit marketing website copy, banners, homepage sections without a code deploy. |
| Blog & FAQ Manager | Publish blog posts and FAQ entries for SEO. |
| Notification Template Editor | Edit push/email/SMS copy without redeploying. |
| Bulk CSV Import/Export | Bulk-manage properties/units for landlords with many units. |
| Media/Gallery Manager | Central library for uploaded images across the platform. |
| Database Backup & Maintenance Mode UI | Manual backup trigger, scheduled maintenance banner. |
| App Update Enforcement | Force-update gating for outdated mobile app versions. |
| Multi-language Content | Only relevant once you expand beyond English/Filipino. |

### 3.3 Landlord

| Feature | Description |
|---|---|
| KYC Onboarding | Collected right after choosing "I manage a property": legal name (from the social profile, editable), government ID upload (front + back — Philippine passport, driver's license, UMID, or PhilSys National ID), a selfie for liveness/face-match against the ID, and payout details (bank account or GCash/Maya account for receiving rent). A property-ownership document (land title or tax declaration) is offered as optional at this stage — skipping it doesn't block onboarding, but submitting it later earns a "Verified Landlord" badge once admin staff review it (§3.2.2). |
| Property Setup | Add buildings, create units, assign tenant details, generate invite codes. |
| Billing Engine | Set monthly rent and due dates; generate automated digital invoices. |
| Promo Codes & Discounts | Issue unique discount codes (e.g., "Early Bird Rent") tenants can apply to their bill. |
| Submeter Management | Input electric kWh usage per tenant, with a mandatory photo of the physical meter attached to the bill. |
| Adaptive Financial Dashboard | Monthly overview of Total Expected Revenue, Total Collected, and Outstanding Balances, laid out differently depending on portfolio size: a landlord with one building and one or two units sees their units directly on the dashboard; a landlord with one building and many units sees a summary card above the unit list; a landlord with multiple buildings sees portfolio-wide totals with a building switcher. All three views read from the same `properties → units → leases` data — this is a display-logic decision, not a different data model. |
| Targeted Announcements | Push/SMS/email broadcasts, filterable by building (e.g., send only to Building A). |
| Ticket Management | Dashboard for maintenance requests — approve/deny with a written reason, plus a dedicated chat thread. |
| Community Chat | Group chat room scoped to a specific building or property. |
| Add/Switch Tenant Profile | From Profile & Settings: add a Tenant profile if one doesn't exist yet, or switch to it if it does. |

### 3.4 Tenant

| Feature | Description |
|---|---|
| Handshake Gate | A brand-new Tenant profile can't be used until a valid invite code is entered — there's no "browse the app first, join a unit later" state. |
| Frictionless Payments | In-app payment via local e-wallets (GCash/Maya) and bank transfer via QR Ph, all through PayMongo. |
| Automated Reminders | Push notifications at 7 days before due date, 1 day before, and on the due date. |
| Budget & Expense Tracker | Visual dashboard of historical rent and utility payments. |
| Ticketing & Support | Submit repair requests with photo attachments; reply to the landlord within the ticket's chat thread. |
| Add/Switch Landlord Profile | From Profile & Settings: add a Landlord profile if one doesn't exist yet (routes into the same KYC flow as §3.3), or switch to it if it does. |

---

## 4. Core Business Logic & Workflows

### 4.1 The "Handshake" (Tenant Onboarding) — with Security Spec

1. Landlord creates "Unit 101" in the app.
2. System generates a unique 6-character alphanumeric code (e.g., `A7X-92P`), excluding visually ambiguous characters (`0`/`O`, `1`/`I`/`l`).
3. The person redeeming the code signs in with Google/Apple/Facebook (or unlocks their existing session via PIN), sets up a Tenant profile if choosing that role for the first time, and enters the code — the code entry screen is mandatory and blocking for a new Tenant profile.
4. System links the tenant profile to Unit 101 and the landlord's database partition.

**Security rules:**

- **Expiry:** An unclaimed code expires 7 days after generation. The landlord can regenerate a fresh code at any time, which invalidates the old one.
- **Single use:** A code becomes invalid the moment it's claimed. The unit moves to "Pending Confirmation" until the tenant completes Lease Confirmation (Screen 3.1).
- **Rate limiting:** Max 5 code-entry attempts per device per 15 minutes. After that, exponential backoff (e.g., 1 hr, then 24 hr lockout).
- **One tenant per unit:** The system rejects a second handshake attempt against an already-occupied unit unless the landlord explicitly reassigns it.

### 4.2 Submeter Utility Engine — with Dispute Handling

1. Landlord inputs the new submeter reading (kWh) and uploads a required photo of the physical meter.
2. System pulls the previous month's reading automatically.
3. System calculates: `(Current Reading − Previous Reading) × Rate = Total Utility Cost`.
4. System appends the utility cost and meter photo to the tenant's upcoming invoice.

**Dispute & edge-case handling:**

- **Tenant dispute window:** Tenant can flag a submeter reading as disputed within 48 hours of invoice issuance, optionally attaching their own photo/comment. Disputed invoices move to `under_review` and are excluded from due-date reminder cron jobs until resolved.
- **Resolution:** Landlord either revises the reading and reissues the invoice, or rejects the dispute with a written reason — both actions are logged (§8.4). Unresolved disputes are also visible to internal staff via the Admin Panel's Dispute Oversight tool (§3.2.1).
- **Meter rollover / negative usage:** If `current_kwh < previous_kwh`, the system does not auto-calculate a negative cost — it flags the entry for manual landlord review instead.

### 4.3 Ticketing & Auto-Close Rule

1. Tenant submits a maintenance ticket (e.g., "Broken AC") with a photo.
2. Landlord and tenant communicate within the ticket thread.
3. Once resolved, the landlord manually sets status to "Resolved."
4. **System rule:** If a ticket sits inactive (no messages from either party) for 7 days, it's automatically flagged "Closed due to inactivity."

### 4.4 Payment Processing & Webhook Handling (PayMongo)

- **Idempotency:** Every incoming PayMongo webhook carries a unique event ID. The system stores processed event IDs and discards duplicate deliveries.
- **Partial payments:** If `amount_paid < total_amount`, the invoice status becomes `partially_paid`, and the remaining balance rolls onto the next invoice.
- **Failed / reversed payments:** A `payment_failed` or chargeback webhook reverts the invoice to `issued` or `overdue` and notifies the landlord immediately.
- **Reconciliation job:** A nightly job cross-checks PayMongo's settlement report against the local `payments` table.

### 4.5 Lease Termination & Move-Out Reconciliation

1. Landlord or tenant initiates move-out; lease status changes to `ending` with a confirmed `end_date`.
2. The system auto-generates a final invoice, prorating base rent for the partial final month.
3. Any outstanding submeter usage up to the move-out date is billed on this final invoice.
4. Once the final invoice is marked `paid`, lease status changes to `terminated` and the unit reverts to `vacant`.

### 4.6 Adding or Switching Role Profiles

1. From Profile & Settings, a user taps "Add [Other Role]" or "Switch to [Other Role]."
2. **Adding:** the user is walked through the other role's first-run flow (KYC + Property Setup for a new Landlord profile, Code Entry for a new Tenant profile).
3. **Switching:** the app re-scopes the current session to the selected profile's ID and re-renders the corresponding dashboard — no re-authentication required, since the underlying identity and its PIN session are shared.
4. Financial data, properties, leases, and tickets are fully scoped per profile and never mixed.

### 4.7 Session & PIN Authentication

1. First-ever app open: user taps Continue with Google / Apple / Facebook. A successful OAuth callback creates (or matches) a `users` row.
2. The app then collects and OTP-verifies a phone number, and has the user set a 6-digit PIN.
3. On every subsequent app open, the app checks for a valid device-bound refresh token; if present, it prompts for the PIN (or biometric, falling back to PIN) rather than repeating the OAuth flow.
4. A wrong PIN attempt is rate-limited the same way invite codes are (§4.1) to prevent brute-forcing.
5. "Forgot PIN" invalidates the local session and requires a fresh OAuth login or SMS OTP before a new PIN can be set.

---

## 5. UX/UI Documentation

### 5.1 Navigation Model & Global Patterns

Three patterns repeat throughout the app and should be built once as reusable components rather than one-off screens:

- **Adaptive Properties navigation.** The entry point into the Properties section depends on portfolio size (§3.3). Single-building landlords skip straight to the Units List. Multi-building landlords land on a Buildings List first. Both paths converge on the same Unit Detail and Tenant Profile screens beneath them — only the entry point changes, not the destination screens.
- **Bottom sheet previews.** Every tappable dashboard summary metric (Overdue, Collected, and — for multi-building landlords — the portfolio Revenue Breakdown) opens as a bottom sheet modal over the current screen, not a new full page. This keeps the dashboard as "home base": the user glances at a filtered list without losing their place, and dismisses it by swiping down or tapping outside it.
- **Deep-linking from previews into Properties.** Tapping a tenant row inside any bottom sheet navigates directly to that tenant's Tenant Profile screen, skipping the Buildings List / Units List steps entirely. The bottom tab bar switches to "Properties" as the active tab so the back button behaves correctly — back returns to that tenant's Units List, not to the dashboard.

### 5.2 Flow at a Glance

Splash Screen
↓
Welcome Carousel (first launch only, 3 slides)
↓
Continue with Google / Apple / Facebook
↓
(First-time users only)
Phone Number Entry + OTP Verification
↓
Set Up 6-Digit PIN (+ optional biometric)
↓
Role Selection: "I manage a property" (Landlord) | "I'm renting a place" (Tenant)
↓
Landlord: KYC → Property Setup | Tenant: Enter 6-Digit Code (blocking)
↓
Landlord: Generate Code | Tenant: Confirm Lease
↓
Main Dashboard (bottom-sheet drill-downs into Overdue/Collected/Revenue)
↓
Returning visits: Splash → PIN / Biometric Unlock → Main Dashboard
↓
Profile & Settings, anytime → Add [Other Role] / Switch to [Other Role] (optional)


*(The Admin Panel is a separate authenticated web app with its own email/password login — see §6.2, screens in §5.6.)*

### 5.3 Global Entry Flow (Universal)

**Screen 1.0 — Splash Screen** Initial app load and session check. If a device-bound session exists, routes to PIN/Biometric Unlock; otherwise routes to the Welcome Carousel or straight to social sign-in for a returning uninstalled-reinstalled user.

**Screen 1.1 — Welcome Carousel (first time only)** Three swipeable graphics with "Next" and "Skip" actions.

**Screen 1.2 — Continue With** Three buttons: Continue with Google, Continue with Apple, Continue with Facebook. No email/password fields anywhere on this screen.

**Screen 1.3 — Phone Verification (first-time users only)** Mobile number entry, SMS OTP confirmation.

**Screen 1.4 — Set Up PIN (first-time users only)** 6-digit PIN entry and confirmation, with an "Enable FaceID/Fingerprint" toggle immediately after.

**Screen 1.5 — Role Selection** Two cards: "I manage a property" and "I'm renting a place." Creates the user's first role profile.

**Screen 1.6 — Permissions Opt-In** Prompts for Push Notification access, with copy explaining why.

**Screen 1.7 — PIN / Biometric Unlock (returning users)** Shown on every app open after the first; biometric attempts first if enabled, falling back to the PIN pad on failure.

### 5.4 The Landlord Journey

**Screen 2.0 — KYC** Legal name (pre-filled from social profile), government ID front + back upload, selfie for liveness check, payout bank/e-wallet details, and an optional property-ownership document upload.

**Screen 2.1 — Empty State & Property Setup** Landlord selects "Multi-Unit Building" or "Single Rental House," then enters property name, address, base rent, and rent due date.

**Screen 2.2 — Unit Generation & Handshake** Displays the generated invite code, with "Copy Code" and "Share via SMS/Messenger."

**Screen 2.3 — Landlord Main Hub (Home)** Three tappable summary cards — Total Expected Revenue, Total Collected, Overdue — laid out per the Adaptive Dashboard rule (§3.3). Tapping any card opens a bottom sheet:

- **Screen 2.3a — Overdue Preview (bottom sheet):** Tenants with an overdue invoice, sorted by days overdue descending — tenant name, unit/building label, amount overdue, days overdue. Tapping a row deep-links to that tenant's Tenant Profile (Screen 2.7). Empty state: "Nothing overdue — nice work." A "View All" link appears if the list exceeds ~8 rows, opening a full-screen filterable version.
- **Screen 2.3b — Collected Preview (bottom sheet):** Tenants who paid in the current billing period — tenant name, unit/building, amount paid, date paid. Tapping a row deep-links to Tenant Profile.
- **Screen 2.3c — Revenue Breakdown (bottom sheet, multi-building landlords only):** Expected/Collected/Overdue broken down per building. Tapping a building row switches the dashboard's building filter rather than navigating away.

**Screen 2.4 — Buildings List (multi-building landlords only)** Entry point into Properties for landlords with more than one building. A card per building: name, address, unit count, occupied/vacant summary, this month's collection percentage. Tapping a building opens its Units List (Screen 2.5).

**Screen 2.5 — Units List** For single-building landlords, this is the direct destination when tapping the Properties tab. For multi-building landlords, it's reached from Screen 2.4. A card per unit: unit number, tenant name or "Vacant," lease status badge (Active / Pending Confirmation / Vacant), and this month's payment status (Paid / Partial / Overdue / Not Yet Due). Tapping a unit opens Unit Detail (Screen 2.6).

**Screen 2.6 — Unit Detail** Unit number, base rent, current lease status. If occupied: a tenant summary card (name, photo, phone) — tapping it opens Tenant Profile (Screen 2.7). If vacant: a "Generate Invite Code" button instead. Quick actions below: Add Submeter Reading, View Invoice History, Edit Unit Details, and — if occupied — "Reassign / End Lease" (opens the move-out flow, §4.5).

**Screen 2.7 — Tenant Profile** Reached either by tapping a unit's tenant card (Screen 2.6) or via deep-link from a dashboard bottom sheet (2.3a/2.3b). Shows: tenant contact info, a link back to their unit/building (tappable), lease terms (start date, rent, due day), an invoice ledger (tap any row → Invoice Detail, Screen 2.8), submeter reading history with photos, and this tenant's ticket history (tap a ticket → Ticket Detail, Screen 2.10). Actions: Message Tenant, Initiate Move-Out.

**Screen 2.8 — Invoice Detail** Base rent + utility breakdown, status badge (draft/issued/paid/partially_paid/overdue/under_review), meter photo thumbnail if a submeter charge is attached, itemized payment history for partial payments, and — if disputed — the dispute thread with Resolve/Reject actions. This screen is shared with the tenant app (§5.5), rendered read-only there.

**Screen 2.9 — Ticketing View (Kanban)** Open / Resolved by Me / Closed Confirmed columns, filterable by building for multi-building landlords. Tapping a ticket card opens Ticket Detail (Screen 2.10).

**Screen 2.10 — Ticket Detail** Subject, description, photo attachment, chat thread, status control (approve/deny/mark resolved with a written reason), and a link back to the tenant's Tenant Profile.

**Screen 2.11 — Profile & Settings** Manage SaaS subscription tier, toggle Dark Mode, edit bank payout details, view/submit the optional verification document, change PIN, broadcast global announcements, and Add/Switch Tenant Profile.

### 5.5 The Tenant Journey

**Screen 3.0 — The Handshake (Code Entry)** 6-character input field — mandatory and blocking for a new Tenant profile.

**Screen 3.1 — Lease Confirmation** Displays binding details with "Confirm & Bind."

**Screen 3.2 — Tenant Main Hub (Home)** "Next Payment Due" card — tapping it opens Payment Gateway Checkout (Screen 3.3) if unpaid, or Invoice Detail (Screen 3.7) if already settled — plus a breakdown of base rent + utility charges and a "Pay Now" button. A "Recent Activity" list below shows the last 3 payments/tickets, each tappable to its own detail screen.

**Screen 3.3 — Payment Gateway Checkout** Native PayMongo checkout UI: GCash, Maya, QR Ph, or bank transfer.

**Screen 3.4 — Expense Tracker (Budget)** Visual chart of historical rent and utility payments. Tapping a payment row opens Invoice Detail (Screen 3.7, read-only).

**Screen 3.5 — Support & Ticketing** List of active/past tickets, submission with text + photo. Tapping a ticket opens Ticket Detail (Screen 3.6).

**Screen 3.6 — Ticket Detail & Chat (Tenant view)** Chat interface with the landlord, plus "Confirm Resolution" with a 1–5 star rating once the landlord marks it resolved.

**Screen 3.7 — Invoice Detail (read-only)** Same underlying screen as Screen 2.8, without landlord-only edit actions. If the invoice includes a submeter charge and the 48-hour dispute window is still open, a "Dispute this reading" action appears here (§4.2) — this is the only place the dispute action lives, not a separate screen.

**Screen 3.8 — Profile & Settings** View active lease terms, toggle Dark Mode, update contact info, change PIN, download past payment receipts, and Add/Switch Landlord Profile.

### 5.6 The Admin Panel

Separate authenticated web app with its own email/password login (§6.2).

**Screen 4.0 — Admin Login** Email/password fields, no OAuth options.

**Screen 4.1 — Global Dashboard** Metric cards — Total Active Landlords, Total Tenants, Total Gross Transaction Value — plus a Revenue Trend chart. Tapping "Total Active Landlords" or "Total Tenants" opens User & Account Management (Screen 4.2) pre-filtered to that role.

**Screen 4.2 — User & Account Management** Searchable, filterable table of accounts — identity, role profile badge(s), KYC status. Tapping a row opens User Detail (Screen 4.3).

**Screen 4.3 — User Detail** Full profile: role profile(s) held, linked properties/leases, KYC documents (viewable, not editable), Suspend/Unlock action, and an audit trail of past admin actions on this account.

**Screen 4.4 — Property & Unit Oversight** Read-only list of all properties across all landlords. Tapping a row opens Property Detail (Screen 4.5).

**Screen 4.5 — Property Detail (read-only)** Units, tenants, and a link back to the owning landlord's User Detail (Screen 4.3).

**Screen 4.6 — Subscription & Revenue Control** SaaS plan list, commission rate settings, per-landlord billing overrides.

**Screen 4.7 — Dispute Oversight** List of open submeter/ticket disputes across all landlords. Tapping one opens a read-only thread view with an internal "Escalate/Flag" note field — resolution authority stays with the landlord per §4.2; admin visibility exists for cases where a landlord is unresponsive.

**Screen 4.8 — Payment Health Monitor** List of webhook failures and reconciliation mismatches. Tapping a row shows the raw event (event ID, timestamp, error, linked invoice).

**Screen 4.9 — Landlord Verification Review Queue** List of landlords who submitted an optional ownership document. Tapping one opens a document viewer with Approve/Reject actions — firing the `kyc_status_changed` notification (§9) on either action.

**Screen 4.10 — Platform Broadcasts** Compose screen: title, body, target audience (all / landlords only / tenants only), send button, with a history of past broadcasts below.

**Screen 4.11 — System Users & Permissions** List of internal staff accounts and permission levels; "Invite" opens a form (email, permission level).

### 5.7 Information Architecture

Application
│
├── Authentication
│ ├── Continue with Google/Apple/Facebook
│ ├── Phone Verification (first-time only)
│ ├── PIN Setup + Biometric (first-time only)
│ ├── PIN / Biometric Unlock (returning)
│ └── Role Selection (first-time only)
│
├── Tenant
│ ├── Home (Next Payment Due, Recent Activity)
│ ├── Budget (Expense Tracker → Invoice Detail)
│ ├── Support (Ticketing → Ticket Detail & Chat)
│ └── Profile (+ Add/Switch Landlord, Change PIN)
│
└── Landlord
├── Home (Adaptive Dashboard + Overdue/Collected/Revenue bottom sheets)
├── Properties
│ ├── Buildings List (multi-building only)
│ ├── Units List
│ ├── Unit Detail
│ │ └── Tenant Profile
│ │ ├── Invoice Detail
│ │ └── Ticket Detail
├── Tickets (Kanban → Ticket Detail)
└── Profile (+ Add/Switch Tenant, Change PIN, Verification)

Admin Panel (separate authenticated web app, email/password login — §6.2)
├── Global Dashboard
├── User & Account Management → User Detail
├── Property & Unit Oversight → Property Detail
├── Subscription & Revenue Control
├── Dispute Oversight
├── Payment Health Monitor
├── Landlord Verification Review Queue
├── Platform Broadcasts
└── System Users & Permissions


### 5.8 Empty & Error States

- **Units List, no units yet:** "You haven't added a unit yet" + "Add a Unit" CTA.
- **Buildings List, no properties yet:** "You haven't added a property yet" + "Add Your First Property" CTA (routes to Screen 2.1).
- **Overdue bottom sheet, nothing overdue:** "Nothing overdue — nice work" (positive framing, not a blank void).
- **Collected bottom sheet, nothing collected yet this period:** "No payments collected yet this month."
- **Support/Tickets, no tickets:** "No tickets yet."
- **Admin Dispute Oversight, no open disputes:** "No open disputes."
- **Global network/loading failure:** a standard retry banner with a "Try Again" action — one shared component used across every screen rather than a custom message per screen.

### 5.9 Wireframes & Design System

Low-fidelity wireframes to be built in Figma, covering every screen in §5.3–§5.6 above, including the three bottom-sheet preview states as their own modal components. Design system requires multi-theme support (Dark Mode / Light Mode), a numeric PIN-pad component, and a reusable bottom-sheet component used for Overdue/Collected/Revenue previews. FIGMA LINK

---

## 6. Technical Architecture

### 6.1 Third-Party Integrations

| Category | Provider | Purpose |
|---|---|---|
| Social Identity | Google, Apple, Facebook (via Supabase Auth's OAuth providers) | Sole account creation/login method — no password grant is enabled |
| Payment Gateway | PayMongo | Automated collections via e-wallets and QR Ph |
| Cloud Object Storage | Supabase Storage | Government ID photos, selfies, submeter photos, ticket attachments. Chosen over Cloudinary since it shares auth and access-policy configuration with Supabase Postgres/Auth — one platform, one RLS model, less integration surface for a 5-person team. |
| Push Notifications | Expo Push Notifications | Rent reminders, announcements, chat messages |
| SMS & Email | **Semaphore** (primary, PH-focused SMS for OTP and reminders), Resend (transactional email) | OTP delivery, digital invoices, invite codes. Twilio is kept in reserve only if the product later expands outside the Philippines — Semaphore is cheaper and purpose-built for PH numbers. |
| Biometrics | expo-local-authentication | FaceID/fingerprint as a fast layer over the PIN |

**Why PayMongo over Xendit:** PayMongo is Philippines-first — GCash, Maya, QR Ph, cards, and BNPL are all natively supported with public, transparent pricing and no monthly platform fee at MVP volume. Xendit's differentiators are cross-border Southeast Asia support and over-the-counter retail payment (7-Eleven, Cebuana Lhuillier), neither of which Rease needs as a Philippines-only, e-wallet-first product.

### 6.2 Security & Access Control

- **No stored passwords.** Identity is entirely delegated to Google/Apple/Facebook; Rease never stores a password hash for a landlord or tenant account.
- **PIN storage:** the 6-digit PIN is hashed (bcrypt/argon2) and verified server-side on every unlock — never compared client-side, and never stored in plaintext even briefly.
- **Multi-tenancy Row-Level Security (RLS):** every query is partitioned by `landlord_profile_id` / `property_id` on the landlord side, and by `tenant_profile_id` on the tenant side.
- **Role-switch isolation:** switching between a user's Landlord and Tenant profiles changes which data the session can query, but never re-authenticates or leaks one profile's data into the other.
- **Admin Panel isolation:** the internal Admin Panel is a separate authenticated web application with its own traditional email/password login — intentionally not part of the consumer OAuth+PIN flow, so a compromised social account or leaked PIN can never reach admin functionality.
- **Granular admin permissions:** internal staff access the panel through `system_users` with scoped permission levels (full / support / finance-only).
- **Webhook signature verification:** every PayMongo webhook is verified against PayMongo's signing secret before processing.
- **Encryption at rest for sensitive files:** government ID photos, selfies, and bank account details are encrypted at rest in Supabase Storage, required for Data Privacy Act compliance (§13.3).
- **Rate limiting:** applies to both invite-code entry (§4.1) and PIN entry (§4.7), same pattern, same backoff curve.

### 6.3 Backend Stack

| Component | Technology | Purpose |
|---|---|---|
| Server Runtime | Node.js (TypeScript) | Server-side business logic and APIs |
| API Framework | NestJS | REST routing, validation, webhooks |
| Database | PostgreSQL (Supabase Postgres) | Relational data — properties, leases, ledgers |
| Authentication | Supabase Auth, OAuth providers only (Google, Apple, Facebook) — email/password grant disabled | Identity; custom PIN layer built on top at the application level |
| File Storage | Supabase Storage | KYC documents, submeter photos, attachments |
| Hosting | Vercel | Serverless API deployment via GitHub CI/CD |

Consolidating on Supabase for Postgres, Auth, and Storage (rather than splitting storage to Cloudinary and DB to Neon) means one dashboard, one billing relationship, and one RLS policy language covering both your data and your files — a meaningful simplification for a 5-person team building this for the first time.

### 6.4 Frontend & Mobile Stack

| Component | Technology | Purpose |
|---|---|---|
| Mobile Framework | React Native (Expo) | Cross-platform iOS/Android app |
| Web Dashboard | Next.js (React) | Browser app for Landlords, plus the internal Admin Panel (§3.2) |
| UI Styling | Tailwind CSS / NativeWind | Shared utility-first styling |
| Component Library (mobile) | React Native Paper | Pre-built accessible mobile components |
| Component Library (web) | shadcn/ui | Pre-built accessible web components |
| State Management | Zustand & React Query | Auth/PIN session state, active-profile state, server cache sync |
| Data Visualization | Victory Native (mobile) / Recharts (web) | Expense trends, utility history, revenue |

---

## 7. API Contract

Starting skeleton — expand with full request/response bodies before implementation.

### 7.1 Auth, Identity & PIN

| Method | Path | Description | Auth |
|---|---|---|---|
| POST | `/v1/auth/oauth/callback` | Complete Google/Apple/Facebook OAuth, create or match the `users` row | Public |
| POST | `/v1/auth/phone/send-otp` | Send SMS OTP to a phone number for first-time verification | Session required (post-OAuth) |
| POST | `/v1/auth/phone/verify-otp` | Confirm the OTP and mark the phone verified | Session required |
| POST | `/v1/auth/pin/setup` | Set the account's PIN for the first time | Session required |
| POST | `/v1/auth/pin/verify` | Verify the PIN to unlock a returning session | Device-bound refresh token required |
| POST | `/v1/auth/pin/reset` | Start PIN recovery via fresh OAuth or SMS OTP | Public (rate-limited) |
| GET | `/v1/me/profiles` | List which role profile(s) exist for the current identity | Session required |
| POST | `/v1/me/profiles` | Add a new role profile (Landlord or Tenant) to the current identity | Session required |
| POST | `/v1/me/active-profile` | Switch the session's active role profile | Session required |

### 7.2 Properties & Units

| Method | Path | Description | Auth |
|---|---|---|---|
| POST | `/v1/properties` | Create a property | Landlord |
| GET | `/v1/properties` | List landlord's properties | Landlord |
| POST | `/v1/properties/:id/units` | Create a unit under a property | Landlord |
| POST | `/v1/units/:id/invite-code` | Generate/regenerate a handshake code | Landlord |
| POST | `/v1/units/handshake` | Redeem an invite code (body: `code`) | Tenant |

### 7.3 KYC

| Method | Path | Description | Auth |
|---|---|---|---|
| POST | `/v1/landlord/kyc` | Submit government ID, selfie, payout details, and optional ownership document | Landlord |
| GET | `/v1/landlord/kyc/status` | Check current KYC/verification status | Landlord |

### 7.4 Leases & Invoices

| Method | Path | Description | Auth |
|---|---|---|---|
| POST | `/v1/leases/:id/terminate` | Initiate move-out (§4.5) | Landlord or Tenant |
| GET | `/v1/invoices?lease_id=` | List invoices for a lease | Landlord or Tenant (own data) |
| POST | `/v1/invoices/:id/submeter` | Attach submeter reading + photo | Landlord |
| POST | `/v1/invoices/:id/dispute` | Flag a submeter reading as disputed (§4.2) | Tenant |
| POST | `/v1/invoices/:id/dispute/resolve` | Resolve or reject a dispute | Landlord |

### 7.5 Payments

| Method | Path | Description | Auth |
|---|---|---|---|
| POST | `/v1/payments/checkout` | Create a PayMongo checkout session for an invoice | Tenant |
| POST | `/v1/webhooks/paymongo` | PayMongo webhook receiver (signature-verified, idempotent) | Gateway only |

### 7.6 Tickets

| Method | Path | Description | Auth |
|---|---|---|---|
| POST | `/v1/tickets` | Submit a maintenance ticket | Tenant |
| PATCH | `/v1/tickets/:id/status` | Update ticket status | Landlord |
| POST | `/v1/tickets/:id/messages` | Post to the ticket's chat thread | Landlord or Tenant |
| POST | `/v1/tickets/:id/close-confirm` | Tenant confirms resolution + rating | Tenant |

### 7.7 Admin Panel

| Method | Path | Description | Auth |
|---|---|---|---|
| GET | `/v1/admin/users` | Search/list landlord & tenant accounts | System User |
| PATCH | `/v1/admin/users/:id/suspend` | Suspend an account | System User (full access) |
| GET | `/v1/admin/disputes` | List open submeter/ticket disputes across all landlords | System User |
| GET | `/v1/admin/payments/health` | Webhook failures & reconciliation mismatches (§4.4) | System User (finance) |
| POST | `/v1/admin/system-users` | Invite an internal staff member with a permission level | System User (full access) |
| PATCH | `/v1/admin/landlords/:id/verify` | Approve/reject a landlord's optional ownership document | System User |

---

## 8. Database Schema (ERD)

### 8.1 Core User & Property Tables

**users** *(one row per person — no password field; identity is delegated to the OAuth provider)*
- `id` (UUID, PK), `full_name`, `email` (unique), `oauth_provider` (enum: google, apple, facebook), `oauth_subject_id` (string, unique per provider), `phone_number` (unique), `phone_verified_at` (timestamp, nullable), `pin_hash`, `pin_set_at`, `created_at`

**landlord_profiles**
- `id` (UUID, PK), `user_id` (FK → users.id, unique), `kyc_status` (enum: pending, verified, rejected), `government_id_url`, `selfie_url`, `payout_bank_details` (encrypted), `ownership_doc_url` (nullable), `ownership_verified` (boolean, default false), `subscription_tier` (enum: free, pro), `created_at`

**tenant_profiles**
- `id` (UUID, PK), `user_id` (FK → users.id, unique), `created_at`

**system_users** *(supports the Admin Panel's own separate login and permission model)*
- `id` (UUID, PK), `user_id` (FK → users.id — a separate identity space from the consumer `users` table's OAuth accounts), `permission_level` (enum: full, support, finance_only), `invited_by`, `created_at`

**properties**
- `id` (UUID, PK), `landlord_profile_id` (FK), `name`, `address`, `property_type` (enum: Building, Single_House), `has_submeter` (boolean)

**units**
- `id` (UUID, PK), `property_id` (FK), `unit_number`, `base_rent` (decimal), `status` (enum: vacant, occupied, maintenance), `invite_code` (unique), `invite_code_expires_at`, `invite_code_claimed_at` (nullable)

### 8.2 Leasing & Financial Tables

**leases**
- `id` (UUID, PK), `unit_id` (FK), `tenant_profile_id` (FK), `start_date`, `end_date`, `rent_due_day`, `status` (enum: active, ending, terminated)

**invoices**
- `id` (UUID, PK), `lease_id` (FK), `billing_period`, `base_rent_amount`, `previous_kwh` (nullable), `current_kwh` (nullable), `kwh_rate` (nullable), `utility_amount`, `meter_photo_url` (nullable), `promo_code_id` (nullable), `total_amount`, `due_date`, `status` (enum: draft, issued, paid, partially_paid, overdue, under_review), `is_final_invoice` (boolean, default false)

**payments**
- `id` (UUID, PK), `invoice_id` (FK), `amount_paid`, `payment_method` (enum: gcash, maya, bank_transfer, cash), `reference_number`, `gateway_event_id` (unique, nullable), `paid_at`

**tenant_external_expenses**
- `id` (UUID, PK), `tenant_profile_id` (FK), `expense_type` (enum), `amount`, `receipt_photo_url` (optional), `paid_date`

**promo_codes**
- `id` (UUID, PK), `landlord_profile_id` (FK), `code` (unique), `discount_type` (enum: percentage, fixed_amount), `discount_value`, `valid_from`, `valid_until`, `max_uses` (nullable), `times_used` (default 0), `status` (enum: active, expired, disabled)

### 8.3 Maintenance & Ticketing Tables

**tickets**
- `id` (UUID, PK), `lease_id` (FK), `subject`, `description`, `attachment_url`, `status` (enum: open, resolved_by_landlord, closed_confirmed, reopened), `tenant_rating` (1–5, nullable), `tenant_feedback` (nullable), `updated_at`

**ticket_messages**
- `id` (UUID, PK), `ticket_id` (FK), `sender_user_id` (FK → users.id), `message`, `created_at`

### 8.4 Audit & Compliance Table

**audit_log**
- `id` (UUID, PK), `actor_user_id` (FK → users.id), `action`, `entity_type`, `entity_id`, `old_value` (jsonb, nullable), `new_value` (jsonb, nullable), `created_at`

---

## 9. Notifications & Automation Matrix

| Event Type | Trigger | Primary Channel | Recipients |
|---|---|---|---|
| `otp_requested` | Phone verification or PIN reset | SMS | Requesting user |
| `rent_due_7d` | 7 days before due date (cron) | Push + Email | Assigned Tenant |
| `rent_due_1d` | 1 day before due date (cron) | Push + SMS | Assigned Tenant |
| `rent_due_today` | 00:00 on due date (cron) | Push | Assigned Tenant |
| `payment_received` | Webhook confirmation from gateway (idempotent, §4.4) | Push + Email | Landlord & Paying Tenant |
| `payment_failed` | Failed/reversed payment webhook (§4.4) | Push + Email | Landlord |
| `submeter_bill_issued` | Landlord uploads reading + photo | Push | Assigned Unit Tenant |
| `submeter_disputed` | Tenant flags a reading (§4.2) | Push + Web Alert | Landlord |
| `ticket_created` | Tenant submits a ticket | Push + Web Alert | Assigned Landlord |
| `ticket_status_changed` | Landlord approves/denies/resolves | Push | Ticket Submitter (Tenant) |
| `ticket_comment_added` | New comment in ticket thread | Push | Landlord/Tenant (excluding author) |
| `announcement_posted` | Landlord broadcasts announcement | Push + In-App Banner | Filtered target audience |
| `lease_ending` | Move-out initiated (§4.5) | Push + Email | Landlord & Tenant |
| `kyc_status_changed` | Admin approves/rejects a landlord's ownership document | Push | Landlord |

---

## 10. Marketing Website Structure

| Nav Item | Sections / Components | Purpose |
|---|---|---|
| Home | Hero, feature highlights, ROI calculator, social proof, CTAs | Explain the product, capture inbound landlord signups |
| Features | Automated invoicing, photo-verified submetering, tenant expense dashboard, ticketing, one-tap PIN login | Deep dive on operational features for property managers |
| Pricing | Free tier (up to 3 units), Pro Landlord (tiered subscription), Enterprise | Transparent SaaS monetization tiers |
| FAQ & Contact | Common landlord questions, security explanations, consultation booking | Build trust, handle pre-sales inquiries |

---

## 11. QA & Test Accounts

Since login is entirely social OAuth, real Google/Apple/Facebook accounts can't be scripted for automated or manual QA runs. **Staging only** exposes a dev auth bypass — a `POST /v1/auth/dev-login` endpoint, disabled in production by an environment flag, that logs directly into a seeded account without a real OAuth round-trip.

| Test Account | Role | Test Scope |
|---|---|---|
| superadmin@rease.test | Superadmin (full system_users access) | Global metrics, user suspension, plan management, dispute oversight |
| landlord.active (seeded, dev-login) | Landlord only, KYC pre-verified | Building creation, submeter logging, ticket updates, promo codes |
| tenant.active (seeded, dev-login) | Tenant only | Rent payments, expense graphs, ticket creation |
| tenant.unassigned (seeded, dev-login) | Tenant (New) | Invite code redemption, blocking-gate behavior |
| dual.role (seeded, dev-login) | Landlord + Tenant | Adding a second role profile, switching, data isolation |
| pin.reset.test (seeded, dev-login) | Tenant | PIN forgot/reset flow |

---

## 12. Team & Engineering Process

### 12.1 Roles & Ownership

| Area | Owner | Scope |
|---|---|---|
| Backend & API | [Name] | NestJS services, DB schema, payment webhook handling |
| Mobile (React Native) | [Name] | Landlord + Tenant app screens (§5.3, §5.4) |
| Web Dashboard & Admin Panel (Next.js) | [Name] | Landlord/Admin web views, including §3.2's internal panel |
| Product & QA | [Name] | Acceptance criteria, test accounts (§11), release sign-off |
| Design / UX | [Name] | Figma wireframes, design system (§5.9) |

### 12.2 Git Workflow

- **Branches:** `main` (production) ← `dev` (staging) ← `feature/*`, `fix/*` branches per task.
- **Pull requests:** Minimum 1 reviewer approval before merge into `dev`; no direct commits to `main` or `dev`.
- **Commit convention:** Conventional Commits style (`feat:`, `fix:`, `chore:`).
- **Merge strategy:** Squash merge to keep history linear.

### 12.3 Coding Standards

- TypeScript strict mode across backend, mobile, and web.
- ESLint + Prettier enforced via pre-commit hook.
- No secrets or `.env` files committed; use `.env.example` as the template.
- No `console.log` in production builds — use a structured logger.

### 12.4 Definition of Done

1. Code reviewed and approved by at least one other team member.
2. Meets the acceptance criteria defined for that feature (§2.4 format).
3. Manually verified against the relevant QA test account (§11).
4. Deployed to staging and confirmed working before merging toward `main`.

### 12.5 Environments

`local` → `staging` → `production`. Staging and production use separate database instances, separate PayMongo sandbox/live keys, and separate OAuth app credentials for each provider (Google/Apple/Facebook require distinct client IDs per environment's redirect URI).

---

## 13. Legal & Compliance

Start these in parallel with development — each has its own lead time.

### 13.1 IP Assignment

Every person contributing code, design, or product work should sign an IP assignment agreement transferring ownership to the company entity.

### 13.2 Business Registration for Payment Collection

PayMongo requires SEC (corporation) or DTI (sole proprietorship) registration plus BIR registration before activating live payment collection. Start now if not already underway.

### 13.3 Data Privacy Act (RA 10173) Compliance

This product collects government ID photos, selfies, and financial data, so it falls under the Data Privacy Act regardless of company size:

- Designate a Data Protection Officer (DPO).
- Write a data retention & deletion policy — this now explicitly needs to cover selfie/liveness images, not just ID photos.
- Encrypt sensitive data at rest (§6.2).
- Draft a breach response plan.
- Mandatory NPC registration applies once processing sensitive personal information of 1,000+ individuals — build compliant habits from day one regardless.

### 13.4 Privacy Policy & Terms of Service

Required before collecting any real user data, including in a closed beta. Must disclose what's collected (including OAuth profile data, phone number, and biometric selfie use) and data-subject rights under the DPA.

### 13.5 NDA

For any contractor, advisor, or early pilot landlord who sees the product before public launch.