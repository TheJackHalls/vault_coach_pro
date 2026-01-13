# Vault Coach Pro — Master Plan (Web-Only, Mobile-Optimized)

## Purpose of This Document
This is the **single source of truth** for how **Vault Coach Pro** is built and evolved as a **web-only**, **mobile-optimized** pole vault coaching app.

It exists to:
- Keep the product aligned with real coaching needs
- Prevent feature creep
- Give clear guidance to AI tools/collaborators
- Keep database, UI, and permission rules consistent

If anything conflicts with this plan, **this plan wins**.

---

## What Vault Coach Pro Is
Vault Coach Pro is a **cloud-backed web app** optimized for **phones** (iPhone/Android) in any modern browser.

Key constraints:
- **Web only** (no native builds)
- **No PWA service worker** (to avoid stale app versions)
- Updates should apply immediately after deploy
- Data is stored in the cloud (Firestore)

---

## Locked Technical Decisions

### App Type
- **Standard web app** (no PWA service worker)
- **Mobile-first responsive UI**

### Frontend
- **React + Vite** (single-page app)

### Backend / Cloud
- **Firebase**
  - **Firebase Auth** (email + password)
  - **Firestore** (source of truth)
  - **Firebase Hosting** (deployment + hosting)
  - **Cloud Functions** (used for secure claim-code linking + sensitive workflows)

### Offline Strategy
- **Online-first but resilient**
  - Optimistic UI where appropriate
  - Automatic retries on transient failures
  - Clear “failed to sync” messaging if a write fails
  - **Firestore IndexedDB persistence enabled** when supported (for resilience), but the app is still conceptually cloud-first

### Updates / Caching
- No service worker.
- Hosting configured to avoid “stuck” JS bundles (standard Firebase Hosting headers + hashed assets from Vite).

---

## Accounts & Identity (Locked)

### Accounts are required for everyone
- All users authenticate via **email + password**.
- **Email verification required before:**
  - creating a team
  - joining a team
  - claiming a placeholder athlete

### Roles
A user may act as:
- **Athlete**
- **Coach**
…and can be both across different teams.

---

## Product Modes

### Athlete Solo Mode (No Team Required)
- An athlete can use Vault Coach Pro alone to log jumps.
- **Solo/personal jumps are private** to that athlete.
- Joining a team does **not** automatically share personal history.

### Team Mode (Ideal Use)
- A coach creates one or more teams.
- Athletes join teams and log within the team context.

---

## Core Data Rules (No Ambiguity)

### Manual Date/Time Entry (Meet Paper Reality)
Every jump has:
- `performedAt` (timestamp) — **required**
- Default: “now”
- Always editable (supports back-entering meet jumps later)

---

## Data Model (Firestore)

### Top-Level Collections
- `users/{uid}`
- `teams/{teamId}`

### User Documents
`users/{uid}` includes:
- `displayName`
- `email`
- `preferredUnits` (`imperial` | `metric`)
- `lastActiveTeamId` (nullable)
- `createdAt`, `updatedAt`

#### Personal (Solo) Data
- `users/{uid}/personalJumps/{jumpId}`

Personal jumps are visible only to the owning user.

---

### Teams
`teams/{teamId}` includes:
- `name`
- `createdByUid`
- `joinCode` (short, human-typable)
- `createdAt`, `updatedAt`
- `deletedAt` (soft delete)

#### Team Membership
- `teams/{teamId}/members/{uid}`
  - `role` (`coach` | `athlete`)
  - `status` (`pending` | `active`)
  - `joinedAt`

**Coach approval is required** for athletes to become `active`.

#### Athletes (Roster Entries)
- `teams/{teamId}/athletes/{athleteId}`

An athlete roster entry can be:
- **Linked athlete**: `userUid = <uid>`
- **Placeholder athlete**: `userUid = null`

Fields:
- `displayName`
- `userUid` (nullable)
- `createdByUid`
- `createdAt`, `updatedAt`
- `deletedAt` (soft delete)

#### Athlete Claim Codes (Athlete-driven claim)
- Implemented via **Cloud Function**:
  - Coach generates a one-time claim code for a placeholder athlete.
  - Athlete signs in → enters code → function links `athletes/{athleteId}.userUid = athleteUid`.
  - One-time use + expiration enforced server-side.
  - After claim, all historical jumps under that athleteId are now associated with the account.

#### Practices
- `teams/{teamId}/practices/{practiceId}`
  - `type` (enum-like string)
  - `title` (optional)
  - `date` (date or timestamp)
  - `createdByUid`
  - `createdAt`, `updatedAt`
  - `deletedAt` (soft delete)

Practice types:
- `vault`
- `weights`
- `sprints`
- `meet` (meet is a practice type)

Meet extras (minimal, not “meet management”):
- `meetName` (optional)
- `venue` (optional)

#### Attendance
- `teams/{teamId}/attendance/{practiceId}_{athleteId}`
  - `practiceId`
  - `athleteId`
  - `status` (`present` | `absent` | `late` | `excused` | `injured`)
  - `updatedByUid`
  - `updatedAt`

#### Jumps (Team Context)
- `teams/{teamId}/jumps/{jumpId}`

Fields (core):
- `athleteId`
- `performedAt` (**required**, editable)
- `practiceId` (nullable; jumps can exist without a practice)
- `createdByUid`
- `createdByRole` (`coach` | `athlete`)
- `barHeightUm` (integer micrometers; see Units below)
- `barHeightInputUnit` (`imperial` | `metric`)
- `result` (`make` | `miss` | `pass`)
- `notes` (optional)
- `lastEditedAt`
- `lastEditedByUid`
- `deletedAt` (soft delete)

Optional structured fields (allowed but not required v1):
- `poleId` or `poleLabel`
- `steps`
- `approachMark` (store as integer millimeters or a well-defined decimal; if used, define once and stick to it)
- `wind` / `runDirection` (later)

---

## Units & Precision (Locked: No Drift Ever)

### Bar Height Storage
**Canonical:** `barHeightUm` (integer micrometers)

Conversions:
- inches → `um = inches * 25400`
- cm → `um = cm * 10000`

Display:
- Convert from `barHeightUm` to the user’s preferred units for UI only.
- Store `barHeightInputUnit` only to respect how it was entered (UX), not as a second “truth.”

---

## Joining & Onboarding (Locked)

### Athlete joins a team
- Athlete enters `joinCode` (or scans QR that encodes it)
- Creates `members/{uid}` with `status = pending`
- Coach approves → `status = active`

### Placeholder athlete
- Coach can add an athlete without an account (roster entry with `userUid = null`)
- Coach logs jumps under that athleteId
- Later: coach generates one-time claim code → athlete claims after signing in

---

## Permissions (Locked)

### Default logging/editing rules
- **Coach-created jumps**
  - Coach can edit
  - Athlete can edit
- **Athlete-created jumps**
  - Athlete can edit
  - Coach can edit **only if athlete has granted permission**

### Athlete permission setting (simple)
Each athlete user has a global setting:
- `allowCoachEditsForAthleteLoggedJumps` (boolean; default **false**)

---

## Audit Trail (Locked: Option B)
For now, jumps store only:
- `lastEditedAt`
- `lastEditedByUid`

No full version history in v1.

---

## Deletion Policy (Locked)
- **Soft delete only** in v1:
  - `deletedAt`, `deletedByUid`
- Deleted items hidden by default; recoverable.

---

## Mobile UX Requirements (Non-Negotiable)

- Thumb-first UI (bottom nav + primary actions reachable one-handed)
- Tap targets ≥ 44px
- Minimal typing (chips/pickers/steppers)
- “Log Jump” flow should be fast:
  - defaults to last-used values where sensible
  - supports quick entry during practice
- Bright-sun readability (contrast + font sizes)
- Team switcher always accessible

---

## Navigation (Default Structure)
- **Bottom nav** with context-aware tabs (team vs solo)
  - Home
  - Log Jump
  - History
  - Team (when in team context)
  - Settings

---

## What’s Explicitly Out of Scope (Right Now)
- Video upload/analysis
- Meet management (flights, attempt order, officials, etc.)
- Social feed / recruiting / messaging platform

(Meets are allowed only as a lightweight practice type for logging.)

---

## Definition of Success
Vault Coach Pro is successful when:
- A coach can run practice faster than with paper
- Athletes trust their data
- Bar heights remain exact forever
- Attendance and habits are easy to track
- Logging works even when signal is flaky
- It feels calm and reliable on a phone at the track

If it becomes slow, distracting, or fragile, it has failed.
