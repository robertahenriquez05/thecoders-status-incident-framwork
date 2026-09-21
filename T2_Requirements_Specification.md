# T2 — Requirements and Stakeholder Specification

**Team:** TheCoders — Service Status and Incident Framework
**Course:** Software Engineering (EECE-4081-002)
**Members:** Roberta Henriquez (Team Lead), Sam Morris, Rommel Mensah, Justin Brown
**Date:** September 21, 2026
**Traces to:** Team Charter and Project Brief, v0.1 (September 9, 2026)

---

## 1. Stakeholder Analysis

Interest and influence are scored Low / Medium / High. Interest = how much
the outcome matters to them. Influence = how much power they have to change
the system's direction or requirements.

| Stakeholder | Interest | Influence | Why influence differs from interest | What they need from the system |
|---|---|---|---|---|
| **Subscribers / end users** of monitored services | High — they are directly affected when a service they depend on breaks | Low — they cannot change scope, priorities, or design; they only receive output | They care a great deal what happens but have no lever to make it happen differently | Fast, accurate status updates with no false "all clear" or missed incident; ability to subscribe only to what's relevant to them (by component/severity) |
| **On-call engineers / internal team** using the tool day-to-day | High — this tool is part of their incident-response workflow | Medium — they can request changes informally and their friction is a real signal, but they don't control the backlog directly | Their interest and influence are closer than the other rows, but priority-setting still sits with the backlog owner, not with them directly | Low-friction incident logging; accurate automatic detection so they aren't manually policing uptime; a summary tool that saves them time instead of adding a writing task mid-incident |
| **Course instructor (Michael Bartz)** | High — this is a graded, program-reported artifact | Very High — sets the rubric, the grade, and effectively the requirements this document must satisfy | Interest and influence are both maximal here — the one stakeholder where they match | Evidence of real process discipline (Playbook followed, tickets closed, CI passing) and a working, defensible technical design — not a demo that only looks good on stage |
| **Team members** (Roberta, Sam, Rommel, Justin) | High — grade and portfolio artifact | High — they make every implementation decision within scope | Both high, but constrained by the instructor's rubric and the charter's scope, so their influence is real but bounded | Fair distribution of work matching the roles table; a shippable artifact worth including in a portfolio; a grade reflecting effort |
| **Backlog/priority owner** (rotates per charter's decision-making section) | Medium — cares about the project succeeding but is not personally building every part | High — decides what gets built each sprint within the charter's scope | Lower day-to-day interest than the builders, but real control over what gets prioritized | Scope staying aligned to what's gradable and demoable each sprint, so effort doesn't drift into features that don't matter for the deadline |

**Note on distinguishing interest from influence:** the instructor and the
backlog owner are the two rows where influence exceeds interest relative to
personal stake — both can redirect the project without being the ones
"affected" by it day to day the way subscribers or on-call engineers are.
Subscribers are the mirror case: maximal interest, minimal influence. This
gap is exactly why the AI-summary human-approval gate and the notification
requirements below exist — the stakeholders with the least influence
(subscribers) are the ones the system exists to serve, so their needs are
encoded as requirements rather than left to be advocated for informally.

---

## 2. Functional Requirements

Organized by the four in-scope feature areas from the charter's Scope
Statement, plus the two required views.

### 2.1 Component Registry
- **FR-1:** The system SHALL allow an authenticated admin to add a component
  with a name, URL, and description.
- **FR-2:** The system SHALL allow an authenticated admin to edit or delete
  an existing component.
- **FR-3:** The system SHALL display each component's current status
  (operational, degraded, down) on both the admin view and the public status
  page.

### 2.2 Scheduled Health Checks
- **FR-4:** The system SHALL perform an automated HTTP health check against
  each registered component's URL every 60 seconds by default.
- **FR-5:** A health check SHALL be considered successful only if the
  component responds with a 2xx status code within the configured timeout.
- **FR-6:** A component's status SHALL transition to "down" only after 3
  consecutive failed health checks, not after a single failure.
- **FR-7:** The system SHALL log the result (pass/fail, timestamp, response
  time) of every health check performed.

### 2.3 Incident Tracking
- **FR-8:** The system SHALL automatically open a new incident when a
  component's status transitions to "down."
- **FR-9:** An authenticated team member SHALL be able to manually open an
  incident for a component, independent of automatic detection.
- **FR-10:** Every incident SHALL move through the fixed state sequence:
  investigating → identified → monitoring → resolved.
- **FR-11:** The system SHALL record a timestamped log entry every time an
  incident's state changes, including who or what triggered the change.
- **FR-12:** An authenticated team member SHALL be able to manually advance
  or update an incident's state.

### 2.4 Subscriber Notifications
- **FR-13:** A subscriber SHALL be able to register to receive updates
  filtered by component, by severity, or both.
- **FR-14:** A subscriber SHALL be able to choose email, webhook, or both as
  their notification channel.
- **FR-15:** The system SHALL send a notification to all matching
  subscribers whenever a subscribed incident changes state.
- **FR-16:** A subscriber SHALL be able to unsubscribe without requiring an
  authenticated account (e.g., via a link/token issued at signup).

### 2.5 AI-Generated Incident Summaries
- **FR-17:** The system SHALL automatically generate a draft public-facing
  summary of an incident whenever the incident is created or its state
  changes.
- **FR-18:** An AI-generated summary SHALL NOT be published to the public
  status page until an authenticated team member explicitly approves it.
- **FR-19:** A team member SHALL be able to edit an AI-generated summary
  before approving it; the edited version, not the original draft, is what
  publishes.

### 2.6 Views
- **FR-20:** The public status page SHALL be viewable without
  authentication and SHALL display current component statuses and a basic
  incident history.
- **FR-21:** The admin view SHALL require authentication and SHALL allow
  managing components, incidents, and subscribers.
- **FR-22:** The system SHALL support exactly two roles — admin and
  subscriber — per the charter's out-of-scope statement ruling out
  fine-grained permissions.

---

## 3. Non-Functional Requirements

Each is written to be falsifiable — a number, threshold, or named condition.

- **NFR-1 (Health-check timeliness):** A health check request SHALL
  complete (send + response received) within 5 seconds or be recorded as a
  failed check. Verified by timing 20 consecutive checks against a test
  component: 0 of 20 may exceed 5 seconds without being logged as a
  failure.
- **NFR-2 (Notification latency):** Subscriber notifications (email and
  webhook) SHALL be dispatched within 60 seconds of the triggering incident
  state change being saved to the database. Verified by triggering 10 state
  changes and comparing the database write timestamp to the notification
  dispatch log timestamp: 0 of 10 may exceed 60 seconds.
- **NFR-3 (Admin access control):** Every admin-view endpoint (component
  create/edit/delete, incident management, subscriber list — 5 endpoints
  minimum) SHALL reject an unauthenticated request with an HTTP 302
  redirect to the login page or an HTTP 401/403 response. Verified by
  requesting each endpoint directly while logged out: 0 of the tested
  endpoints may return admin data.
- **NFR-4 (Public page exposure):** The public status page's rendered HTML
  SHALL contain zero admin-only controls (edit, delete, approve buttons or
  their underlying form actions) when accessed by an unauthenticated user.
  Verified by inspecting the rendered page source while logged out.
- **NFR-5 (False-alarm avoidance):** A component SHALL NOT be marked "down"
  after fewer than 3 consecutive failed health checks. Verified by
  injecting exactly 2 consecutive failures followed by 1 success: the
  component's status must remain "operational" or "degraded" and must
  never read "down" during that sequence.
- **NFR-6 (AI-summary human gate):** Zero AI-generated incident summaries
  SHALL reach the public status page without a recorded human-approval
  action (an `approved_by` user and timestamp). Verified by attempting to
  publish a summary with no approval record via direct API request: the
  request must be rejected at the application layer, not merely hidden by
  the UI.

---

## 4. Epics and User Stories

Each story follows: **As a `<role>`, I want to `<goal>`, so that
`<benefit>`.** Acceptance criteria are Given/When/Then, each independently
checkable.

### Epic A — Component Registry (Owner: Sam Morris)

**US-A1: As an admin, I want to add a new component with a name and URL,
so that the system can start monitoring it.**
- Given I am on the "Add Component" form, when I submit a valid name and
  URL, then a new component is created with status "operational" by
  default and I am redirected to its detail view.
- Given I submit the form with the URL field empty, when I click Save,
  then the form re-renders with an inline error and no component is
  created.

**US-A2: As an admin, I want to edit or remove a component, so that the
registry stays accurate as our services change.**
- Given a component exists, when I update its URL and save, then future
  health checks target the new URL, not the old one.
- Given I delete a component, when I return to the registry list, then it
  no longer appears and no further health checks run against it.

### Epic B — Scheduled Health Checks (Owner: Sam Morris)

**US-B1: As the system, I want to check every registered component's URL
every 60 seconds, so that status reflects reality without manual polling.**
- Given a component is registered, when 60 seconds pass, then a health
  check request is sent to its URL and the result is logged.
- Given a component responds with a 2xx status within the timeout, when the
  check completes, then it is logged as a pass.

**US-B2: As an admin, I want to see a component's recent health-check
history, so that I can tell whether a failure was a one-off or a pattern.**
- Given a component has at least 5 logged checks, when I view its detail
  page, then the 5 most recent results (pass/fail, timestamp) are listed
  in reverse chronological order.

### Epic C — Incident Tracking (Owner: Rommel Mensah)

**US-C1: As the system, I want to automatically open an incident when a
component goes down, so that failures are tracked without someone noticing
manually.**
- Given a component has 3 consecutive failed health checks, when the 3rd
  failure is recorded, then a new incident is created in the
  "investigating" state, linked to that component.
- Given a component has only 2 consecutive failures, when the health check
  runs again, then no incident is created yet.

**US-C2: As a team member, I want to manually open an incident, so that I
can log a problem the automated checks didn't catch.**
- Given I am on the incident dashboard, when I submit a new incident with a
  component and description, then it is created in the "investigating"
  state.

**US-C3: As a team member, I want to move an incident through
investigating → identified → monitoring → resolved, so that its status
reflects our actual progress.**
- Given an incident is in "investigating," when I change its state to
  "identified," then the change is saved with a timestamp and my user ID.
- Given an incident is "resolved," when I view its detail page, then its
  full state-change history is visible in order.

### Epic D — Subscriber Notifications (Owner: Justin Brown)

**US-D1: As a subscriber, I want to sign up for updates about specific
components or severities, so that I only get notified about what matters
to me.**
- Given I submit the subscription form with an email and at least one
  component or severity selected, when I submit, then a subscription
  record is created and I receive a confirmation with an unsubscribe link.
- Given I submit the form with no component or severity selected, when I
  click Submit, then the form rejects it with an inline error.

**US-D2: As a subscriber, I want to receive an email or webhook call when
an incident I'm subscribed to changes state, so that I stay informed
without checking the status page manually.**
- Given I am subscribed to a component, when an incident on that component
  changes state, then I receive a notification via my chosen channel
  within 60 seconds.
- Given I am subscribed by severity only, when an incident below my chosen
  severity threshold occurs, then I do not receive a notification.

**US-D3: As a subscriber, I want to unsubscribe without creating an
account, so that opting out is as easy as opting in.**
- Given I click the unsubscribe link from a notification email, when the
  link is followed, then my subscription is removed and no login is
  required.

### Epic E — AI-Assisted Incident Summaries (Owner: Roberta Henriquez)

**US-E1: As the system, I want to draft a plain-language incident summary
automatically, so that the person handling the incident doesn't have to
write one from scratch under pressure.**
- Given an incident is created or changes state, when the change is saved,
  then a draft AI-generated summary is attached to the incident within a
  reasonable time, visible only in the admin view.

**US-E2: As a team member, I want to review, edit, and approve an
AI-generated summary before it goes public, so that nothing inaccurate or
poorly worded reaches subscribers.**
- Given a draft summary exists, when I edit its text and click Approve,
  then the edited version — not the original draft — is what gets marked
  approved.
- Given no team member has approved a draft, when the public status page
  is requested, then that draft never appears on it.

**US-E3: As a subscriber/public visitor, I want to see the approved
summary on the status page, so that I understand what happened in plain
language.**
- Given a summary has been approved, when I load the public status page,
  then the approved summary text displays under that incident.

### Epic F — Status Page and Admin Views (Owner: Justin Brown, shared with team)

**US-F1: As a public visitor, I want to view current component statuses
and recent incidents without logging in, so that I can check service
health at a glance.**
- Given I am not logged in, when I visit the public status page, then I
  see all components' current statuses and a basic incident history, with
  no edit/delete/approve controls visible.

**US-F2: As an admin, I want an authenticated view for managing
components, incidents, and subscribers, so that internal operations stay
separate from what the public sees.**
- Given I am not logged in, when I request any admin-view URL directly,
  then I am redirected to the login page instead of seeing admin data.

### Epic G — Authentication and Subscriber Access (Owner: TBD — see Gap #1 below)

**US-G1: As an admin, I want to log in with a password-protected account,
so that only authorized team members can manage the system.**
- Given I enter valid credentials, when I submit the login form, then I am
  redirected to the admin dashboard and my session persists across page
  loads.
- Given I enter invalid credentials 3 times, when I submit the 3rd attempt,
  then I see a generic error that does not reveal whether the username
  existed.

---

## 5. Traceability Table

Reads in both directions: every charter item maps to at least one
requirement, and every requirement maps to at least one story.

| Charter Source | Requirement(s) | User Story(ies) | Notes |
|---|---|---|---|
| Scope: Component registry | FR-1, FR-2, FR-3 | US-A1, US-A2 | Direct |
| Scope: Scheduled health checks | FR-4, FR-5, FR-6, FR-7 | US-B1, US-B2 | Direct |
| Working assumption: check frequency 60s | FR-4 | US-B1 | Direct |
| Working assumption: "down" = 3 consecutive failures | FR-6 | US-C1 | Direct; also drives NFR-5 |
| Scope: Incident tracking, fixed states | FR-8, FR-9, FR-10, FR-11, FR-12 | US-C1, US-C2, US-C3 | Direct |
| Scope: Subscriber notifications (email/webhook) | FR-13, FR-14, FR-15, FR-16 | US-D1, US-D2, US-D3 | Direct |
| Scope: AI-generated summaries, human approval required | FR-17, FR-18, FR-19 | US-E1, US-E2, US-E3 | Direct; also drives NFR-6 |
| Working assumption: human-in-the-loop before publishing | FR-18 | US-E2 | Direct |
| Scope: Two views (public + admin) | FR-20, FR-21 | US-F1, US-F2 | Direct |
| Out of scope: two roles only (admin, subscriber) | FR-22 | US-G1 | Partial — see Gap #1 |
| Stakeholder: Course instructor — process discipline | (Playbook v0.1/v0.2, not a functional requirement) | N/A | Traces to Playbook, not this spec |
| Stakeholder: Subscribers — no surprises, no radio silence | FR-15, NFR-2 | US-D2 | Direct |
| Stakeholder: On-call engineers — low-friction logging | FR-9, FR-12 | US-C2, US-C3 | Direct |
| (uncovered) | NFR-1, NFR-3, NFR-4 | US-B1, US-F1, US-F2, US-G1 | NFRs trace to the stories whose endpoints/timing they constrain, not to a separate charter line — see Gap #2 |

---

## 6. Gaps Surfaced by the Traceability Exercise

**Gap #1 — Subscriber authentication is undefined.**
The charter states "v1 has two roles only (admin, subscriber)" but never
says whether a subscriber is an authenticated account or just an email/URL
on file. Building the traceability table exposed this: FR-22 has no single
story that fully satisfies it, because "subscriber" as used everywhere else
in the charter (US-D1, US-D3) describes an anonymous signup with an
unsubscribe token, not a login.

**Resolution:** we are treating "subscriber" as an unauthenticated role —
someone identified only by their email or webhook URL, with no login,
consistent with US-D1/US-D3 as written. "Admin" is the only role that logs
in (US-G1). We've documented this as a working assumption alongside the
ones already in the charter (Section 2, "Working assumptions") and it
should be confirmed at the same stakeholder interview referenced there.
**Confirmed by the full team (Roberta, Sam, Rommel, Justin) on September
21, 2026.**

**Gap #2 — NFRs don't map to a single charter line.**
The charter's Scope Statement describes *what* the system does, not
performance or security thresholds — those come from the Success Criteria
and general engineering practice instead. Rather than force an artificial
charter citation, we've traced each NFR to the specific FR/story whose
behavior it constrains (e.g., NFR-2's 60-second notification limit
constrains US-D2). This is a legitimate trace, not a gap in the same sense
as #1, but it's worth stating explicitly so the table doesn't look
incomplete where it isn't.

**Gap #3 — Manual incident creation (FR-9/US-C2) has no explicit
permission rule in the charter.**
The charter doesn't say whether only admins, or any authenticated team
member, can manually open an incident. Since the charter defines only two
roles and subscribers are unauthenticated (per Gap #1's resolution), we're
treating "team member" in US-C2 as synonymous with "admin" for v1. Flagging
this in case the stakeholder interview reveals a need for a third,
lower-privilege internal role later — that would be a scope change, not
something to build speculatively now.

---

## 7. Authorship Map

| Section | Author |
|---|---|
| Stakeholder Analysis | Roberta Henriquez |
| Functional Requirements | Sam Morris, Rommel Mensah, Justin Brown (each owns the FRs for their feature area — registry/health checks, incidents, notifications, respectively) |
| Non-Functional Requirements | Roberta Henriquez |
| User Stories and Epics | Sam Morris (Epics A–B), Rommel Mensah (Epic C), Justin Brown (Epics D, F), Roberta Henriquez (Epics E, G) |
| Traceability Table | Roberta Henriquez |
| Gaps Surfaced | Roberta Henriquez, confirmed by full team |
| AI-Use Disclosure Appendix | Roberta Henriquez |

**Note:** This document was drafted in a single chat session with Claude
based on the team's existing Team Charter (v0.1) and Playbooks (v0.1, v0.2)
as source material. Per Section 5.1 of Playbook v0.2, all four team members
reviewed their assigned sections above and take ownership of them before
submission, consistent with the team's AI-use policy that AI-drafted
content "must be reviewed, understood, and taken ownership of by a human
teammate before it counts as our work."

---

## 8. AI-Use Disclosure Appendix

Per Playbook v0.2, Section 5.1 and 5.2:

Generative AI (Claude) was used by this team in the production of this
document as follows:

- **Drafting:** the full structure and initial language of the
  stakeholder analysis, functional requirements, non-functional
  requirements, user stories/epics, and traceability table were drafted by
  Claude based on the team's existing Team Charter and Project Brief
  (v0.1) and Playbooks (v0.1, v0.2) as source material. No new scope,
  feature, or process decision was introduced beyond what those documents
  already established.
- **Gap identification:** the three items in Section 6 (Gaps Surfaced)
  were identified by Claude while constructing the traceability table, per
  the assignment's instruction that the table should surface gaps. The
  *resolution* chosen for Gap #1 (treating subscribers as unauthenticated)
  is a genuine design decision and needs team confirmation, not just
  AI-assisted acceptance — see Section 7.
- **Not used for:** team role assignments, sprint/process decisions, or
  final approval of scope — all of that was already decided by the team in
  the charter and playbook this document draws from.

Per team policy, this content must be reviewed, understood, and taken
ownership of by the team before it counts as the team's own work. Logged
per-contributor in `/ai-logs/roberta.md` per Playbook v0.2 Section 5.2.

---

## Revision History
- v1 — September 21, 2026 — Initial draft, built from Team Charter v0.1 and
  Playbooks v0.1/v0.2.
