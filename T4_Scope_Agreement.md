# T4 — Scope Agreement

**Team:** TheCoders — Service Status and Incident Framework
**Course:** Software Engineering (EECE-4081-002)
**Members:** Roberta Henriquez (Team Lead), Sam Morris, Rommel Mensah, Justin Brown
**Date:** September 29, 2026
**Traces to:** T2 — Requirements and Stakeholder Specification (v1, September 21, 2026)

---

## 1. Summary

By session 26 we commit to **six working capabilities**, one core loop from
end to end: a registered component goes down, the system detects it, opens
an incident, drafts an AI summary that an admin must approve, notifies
email subscribers, and shows the result on a public status page.

We are deliberately cutting breadth (webhooks, severity filtering,
"degraded" status, check-history views, login lockout) so that the core
loop works reliably in front of the class, rather than every T2 requirement
working halfway.

---

## 2. Committed Scope

Each item below is written so it can be shown working live at session 26
and judged as pass/fail.

### C1 — Component registry (Owner: Sam Morris)
A logged-in admin can add a component (name, URL, description), edit it,
and delete it from the admin view. Each component shows its current status
(operational or down) on both the admin view and the public status page.

**Demonstrated by:** adding a new component live, editing its URL, and
deleting a second component; the list updates each time.

### C2 — Scheduled health checks (Owner: Sam Morris)
Every registered component is checked by HTTP every 60 seconds. A check
passes only on a 2xx response within 5 seconds; anything else is a failure.
A component is marked **down only after 3 consecutive failures**. Every
check result (pass/fail, timestamp, response time) is stored in the
database.

**Demonstrated by:** turning off one of our two demo services and showing
the component stays "operational" after 1 and 2 failures, then flips to
"down" on the 3rd.

### C3 — Incident tracking and state machine (Owner: Rommel Mensah)
When a component goes down, an incident opens automatically in the
"investigating" state, linked to that component. An admin can also open an
incident manually. An admin can move an incident through the fixed
sequence investigating → identified → monitoring → resolved, and every
state change is saved with a timestamp and who made it (a user or "system").

**Demonstrated by:** the incident created automatically in C2's demo, then
advanced by hand to "resolved," with its full state history displayed.

### C4 — Email subscriber notifications (Owner: Justin Brown)
A visitor can subscribe with an email address to one or more components,
with no account. When an incident on a subscribed component changes state,
each matching subscriber gets an email within 60 seconds. Every email
includes an unsubscribe link that works without logging in.

**Demonstrated by:** subscribing a test inbox to the failing component,
showing the emails arrive as the incident changes state, then clicking the
unsubscribe link and showing no further emails arrive.

### C5 — AI-drafted incident summaries with human approval (Owner: Roberta Henriquez)
When an incident is created or changes state, the system generates a draft
plain-language summary visible only in the admin view. An admin can edit
the draft and approve it; only the approved (edited) version appears on
the public status page. Publishing without an approval record is rejected
by the server, not just hidden in the UI.

**Demonstrated by:** showing the generated draft, editing one sentence,
approving it, and showing the edited text on the public page. Then showing
that a direct request to publish an unapproved summary is rejected.

### C6 — Public status page, admin view, and admin login (Owner: Justin Brown, with Roberta)
The public status page loads without logging in and shows every
component's current status, active incidents with their approved summaries,
and the 10 most recently resolved incidents. It contains no admin controls.
The admin view requires login (Django's built-in authentication); every
admin URL redirects a logged-out visitor to the login page. Only two roles
exist: admin (logs in) and subscriber (email only, no login).

**Demonstrated by:** loading the public page in a private browser window,
then pasting an admin URL into the same window and being sent to login.

### Demo environment (shared)
Two small dummy services that we control (for example "Auth API" and
"Payments Gateway") with a switch to make them fail on demand, so the
failure in the demo is controlled and repeatable. The full system runs
locally on Roberta Henriquez's laptop for the demo, since
the project has been developed and run there throughout the semester.

---

## 3. Traceability: Committed Scope → T2

| Committed item | T2 functional requirements | T2 non-functional requirements | T2 user stories |
|---|---|---|---|
| C1 Component registry | FR-1, FR-2, FR-3 (operational/down only, see D3) | NFR-3 | US-A1, US-A2 |
| C2 Health checks | FR-4, FR-5, FR-6, FR-7 | NFR-1, NFR-5 | US-B1 |
| C3 Incident tracking | FR-8, FR-9, FR-10, FR-11, FR-12 | — | US-C1, US-C2, US-C3 |
| C4 Email notifications | FR-13 (by component only), FR-14 (email only), FR-15, FR-16 | NFR-2 | US-D1 (component only), US-D2 (email only), US-D3 |
| C5 AI summaries | FR-17, FR-18, FR-19 | NFR-6 | US-E1, US-E2, US-E3 |
| C6 Views and admin login | FR-20, FR-21, FR-22 | NFR-3, NFR-4 | US-F1, US-F2, US-G1 (first criterion only) |

Every committed item traces to at least one T2 functional requirement and
one T2 user story. Items marked "only" are committed in part; the rest of
that requirement appears in Section 4.

---

## 4. Explicitly Deferred Functionality

These are things we wanted and are choosing **not** to build this semester.
None will be shown at the sprint 2 or final demos.

| # | Deferred item | T2 source | Why we are deferring it |
|---|---|---|---|
| D1 | Webhook notifications | FR-14 (webhook part), US-D2 | Email alone proves the notification pipeline. Webhooks add a second delivery path, retries, and a receiving endpoint to demo. |
| D2 | Subscribing by severity | FR-13 (severity part), US-D1, US-D2 second criterion | T2 never defines severity levels for incidents, so this would require designing a new field first. Subscribing by component covers the core need. |
| D3 | "Degraded" component status | FR-3 (degraded) | T2 defines when a component is "down" (FR-6) but never defines "degraded." Components will show only operational or down. |
| D4 | Health-check history view for admins | US-B2 | Results are still stored (FR-7), and admins can see them in Django's built-in admin, but there's no custom history page. |
| D5 | Lockout / generic error after 3 failed logins | US-G1 second criterion | Django's standard login is enough for two roles in a class demo; rate limiting is hardening, not core function. |
| D6 | Configurable check interval and timeout per component | FR-4 ("by default") | Fixed at 60 seconds and 5 seconds for every component. |
| D7 | Public deployment to a hosted server | (not a T2 requirement; our earlier demo plan) | Running locally removes hosting, secrets, and uptime problems from the demo. |

Anything not listed in Section 2 is out of scope, including everything
already ruled out in the Team Charter.

---

## 5. Delivery Plan Across Two Sprints

**Sprint 1 — the monitoring backbone:** C1, C2, C3, admin login from C6,
a bare public status page, and the two demo services. At the end of sprint
1, turning off a demo service produces an incident automatically.

**Sprint 2 — everything that reacts to incidents:** C4, C5, the finished
public page from C6, integration testing of the full loop, and a timed
dry run of the demo at least 3 days before session 26.

Before sprint 1 ends, the team agrees on one shared hook: whenever an
incident is created or changes state, it calls both the notification code
(Justin) and the summary code (Roberta). Agreeing on that interface early
keeps C4 and C5 from blocking each other in sprint 2.

---

## 6. Risks to Delivery

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| R1 | **AI API problems** (key setup, cost, rate limits, or slow or bad responses during the demo) | Medium | High, because C5 is the most visible feature | Build the approval workflow first with a placeholder draft, then connect the AI. Keep a template fallback ready (see cut order). |
| R2 | **Background health-check scheduling in Django** is harder than expected (processes stopping, timing drift) | Medium | High, because C2 feeds C3, C4, and C5 | Use a simple scheduler (a looping management command or a lightweight scheduling library), not a full task-queue setup. Prove it works in week 1 of sprint 1. |
| R3 | **Integration between four owners**: incident state changes must trigger notifications and summaries | Medium | High | Agree on the shared hook in sprint 1 (Section 5). Hold an integration check at the midpoint of sprint 2, not the night before. |
| R4 | **Email delivery** (SMTP setup, emails landing in spam) | Low–Medium | Medium | Test with a dedicated team inbox early. Fall back to Django's console email backend if needed. |
| R5 | **Team availability**: a member loses time to other courses or illness | Medium | Medium | Every committed item has one named owner and one backup reviewer. The lead reassigns work at each standup if an item slips. |

### Cut order (what goes first if a sprint goes badly)

We will cut in this order, and only as far down as needed:

1. **Resolved-incident history on the public page** (part of C6). The page
   shows only current statuses and active incidents.
2. **Manual incident creation** (part of C3, FR-9). Automatic incidents
   still open when components go down.
3. **Unsubscribe link** (part of C4, FR-16). Admins remove subscribers by
   hand in the admin view instead.
4. **Live AI call** (part of C5). Drafts come from a fixed template filled
   in with the incident details. The edit-and-approve gate (FR-18, FR-19,
   NFR-6) stays either way, because that gate is the requirement, not the
   AI model.
5. **Real email delivery** (part of C4). Notifications are shown through
   Django's console email backend in the terminal instead of a real inbox.

**Never cut:** the component registry, the 3-failure rule for "down,"
automatic incident creation, the four-state incident sequence, the approval
gate on summaries, and login protection on the admin view. Without these
there is no working core loop to demo.

---

## 7. Team Agreement

By filling in our names below, we agree that Section 2 is what we will be
judged against at the sprint 2 and final demonstrations, and that any
change to it goes through the team lead and is recorded in this document's
revision history.

| Member | Role in scope | Agreed (name and date) |
|---|---|---|
| Roberta Henriquez | Team lead; C5; C6 admin login | Roberta Henriquez, Sep 29, 2026 |
| Sam Morris | C1, C2 | Sam Morris, Sep 29, 2026 (via team group chat) |
| Rommel Mensah | C3 | Rommel Mensah, Sep 29, 2026 (via team group chat) |
| Justin Brown | C4, C6 | Justin Brown, Sep 29, 2026 (via team group chat) |

---

## 8. AI-Use Disclosure

Per Playbook v0.2, Sections 5.1 and 5.2: this document was drafted with
Claude (claude.ai chat), using the team's T2 Requirements Specification as
source material. The AI proposed the split between committed and deferred
items, the traceability table, the risk list, and the cut order. Every
commitment was checked against T2, and the team reviewed and agreed to it
(Section 7) before submission. Logged in `/ai-logs/roberta.md`.

---

## Revision History
- v1 — September 29, 2026 — Initial scope agreement.
