# Vault Coach Pro — Slice Roadmap (Prompt Labels)

## How to use this
- Implement slices in order unless you intentionally skip.
- Prompts should reference the slice label (e.g., **S01_AUTH**).
- Each slice should end with: build passes + phone-friendly UI.

---

## S00_APP_SHELL — App skeleton + layout baseline
**Goal:** Establish app shell patterns used everywhere.  
**Deliverables:**
- Router base structure (`/login`, `/app` placeholder)
- Global layout (header spacing, container, typography)
- Basic theme colors (teal/black/gray) and reusable form styles

**Acceptance:**
- `npm run dev` loads a clean shell
- `npm run build` passes

**Prompt:** “Implement **S00_APP_SHELL** only. No Firebase yet.”

---

## S01_AUTH — Login / Signup / Reset Password + Auth state
**Goal:** Full Email/Password auth flow.  
**Deliverables:**
- Pages: `/login`, `/signup`, `/reset-password`
- `AuthProvider` + `ProtectedRoute`
- `/app` placeholder protected route
- Email verification sent after signup + banner in `/app` if unverified
- Firebase config via `.env.local` (documented)

**Acceptance:**
- Create account, verify email is sent
- Login works, logout works
- Reset password email sends
- Build passes

**Prompt:** “Implement **S01_AUTH** end-to-end with Firebase Auth modular SDK. Keep UI mobile-first.”

---

## S02_USER_PROFILE — Create/read user profile doc in Firestore
**Goal:** On signup, create a user profile doc; allow editing basics.  
**Deliverables:**
- Firestore: `users/{uid}` doc created on first login/signup
- Profile fields: `displayName`, `rolePreference` (athlete/coach), `createdAt`, `lastLoginAt`
- `/app/profile` page to edit `displayName`

**Acceptance:**
- Profile doc exists in Firestore
- Edits persist and reload correctly
- Build passes

**Prompt:** “Implement **S02_USER_PROFILE** with Firestore write/read under `users/{uid}`.”

---

## S03_PERSONAL_JUMPS — Personal jump log (private)
**Goal:** Athlete can log jumps privately.  
**Deliverables:**
- Firestore: `users/{uid}/jumps/{jumpId}`
- UI: `/app/jumps` list + add/edit form
- Fields: dateTime (manual entry allowed), barHeight, result (make/miss), notes, runwaySteps (optional)
- Sort/filter by date

**Acceptance:**
- Create/edit/delete jump
- Manual date/time entry works
- Mobile-friendly data entry
- Build passes

**Prompt:** “Implement **S03_PERSONAL_JUMPS** with CRUD, manual date/time input, and clean mobile UI.”

---

## S04_TEAMS_V1 — Team create + join request (no roster yet)
**Goal:** Basic coach team creation + athlete join flow.  
**Deliverables:**
- Firestore: `teams/{teamId}` (ownerUid, name, createdAt, inviteCode)
- Membership: `teams/{teamId}/members/{uid}` with status (pending/active), role
- UI: coach create team; athlete request to join by invite code
- Coach approve/deny pending members

**Acceptance:**
- Coach can create team and see invite code
- Athlete can request to join
- Coach can approve and membership updates
- Build passes

**Prompt:** “Implement **S04_TEAMS_V1** with invite code join + approve/deny.”

---

## S05_ROSTER_PLACEHOLDERS — Coach creates “placeholder athletes”
**Goal:** Coach can track athletes who don’t have accounts yet.  
**Deliverables:**
- Firestore: `teams/{teamId}/athletes/{athleteId}` docs (name, createdAt, linkedUid nullable)
- UI: coach roster list + add placeholder athlete

**Acceptance:**
- Coach can create placeholders and view roster
- Build passes

**Prompt:** “Implement **S05_ROSTER_PLACEHOLDERS**. No claim/merge yet.”

---

## S06_CLAIM_MERGE — Athlete-driven claim of placeholder athlete
**Goal:** Athlete claims their placeholder profile; merge data safely.  
**Deliverables:**
- Claim request flow (athlete initiates, coach approves) OR athlete claim code
- On claim: set `linkedUid`, move/merge placeholder jumps if any exist
- Audit fields: `claimedAt`, `claimedByUid`

**Acceptance:**
- Claim succeeds and links to athlete account
- No data loss
- Build passes

**Prompt:** “Implement **S06_CLAIM_MERGE** with athlete-driven claim + coach approval.”

---

## S07_TEAM_JUMPS — Team-managed jump logging + permissions
**Goal:** Coach logs jumps by default; athlete can edit with permission.  
**Deliverables:**
- Firestore: team jump logs (e.g., `teams/{teamId}/jumps/{jumpId}` linked to athleteId or uid)
- Permissions model: coach can edit; athlete can edit own if allowed
- UI: coach quick-entry jump logger (fast mobile form)

**Acceptance:**
- Coach can log for roster athletes
- Athlete can view their team jumps
- Edit permissions respected
- Build passes

**Prompt:** “Implement **S07_TEAM_JUMPS** with coach-first logging + editable rules.”

---

## S08_MEETS — Meet sessions + manual (paper) logging
**Goal:** Meet-based logging; allow later entry from paper.  
**Deliverables:**
- Meet entity: `teams/{teamId}/meets/{meetId}` (date, name, location)
- Jump entries attach to `meetId`
- UI: meet view + add jump later with manual timestamp

**Acceptance:**
- Create meet, add jumps with manual date/time
- Build passes

**Prompt:** “Implement **S08_MEETS** with meet sessions + manual timestamp entry UX.”

---

## S09_POLISH_V1 — UX polish + validation + performance
**Goal:** Make the app feel “real”.  
**Deliverables:**
- Better empty states, error states, loading skeletons
- Form validation everywhere
- Basic responsive nav (bottom nav on mobile)
- Clean consistent typography and spacing

**Acceptance:**
- No ugly edge cases
- Phone usability feels smooth
- Build passes

**Prompt:** “Implement **S09_POLISH_V1** focusing only on UX polish and validation.”

---

## S10_SECURITY_HARDEN — Firestore rules + App Check (optional)
**Goal:** Lock down team data and prevent abuse.  
**Deliverables:**
- Real Firestore rules for teams/members/jumps
- Optional: App Check config and docs

**Acceptance:**
- Unauthorized reads/writes fail
- Authorized flows still work
- Build passes

**Prompt:** “Implement **S10_SECURITY_HARDEN**. Update Firestore rules carefully; do not break existing flows.”

---

# Prompt Template (use this every time)
**“Codex: Implement [SLICE_LABEL].**
- Scope: only this slice.
- Must keep mobile-first UI.
- Must run `npm run build` successfully.
- Provide a short summary of changes + how to test.”
