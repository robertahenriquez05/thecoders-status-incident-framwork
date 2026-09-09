# Playbook v0.1 — Working Agreement

**Team:** TheCoders — Service Status and Incident Framework
**Course:** Software Engineering
**Members:** Roberta Henriquez, Sam Morris, Rommel Mensah, Justin Brown
**Date:** September 9, 2026

This document is our engineering standard. Every rule below is written so that
someone could open our repository next week and say, with confidence, whether
we followed it.

---

## 1. Branching Strategy

We use **GitHub Flow**.

- `main` is always in a deployable state. Nothing broken is ever merged into it.
- All work happens on a feature branch cut from `main`, named
  `<type>/<short-description>` (e.g., `feature/incident-state-machine`,
  `fix/health-check-timeout`).
- **Branch lifetime rule:** a branch must be opened as a pull request within
  **2 business days** of its first commit, and merged (or closed) within
  **3 business days** of that pull request being opened. If a branch is still
  open past that window, it is flagged in the next standup and either split
  into smaller PRs or abandoned — it does not sit open indefinitely.
- No one commits directly to `main`. Every change reaches `main` through a
  pull request (see Section 4).

**Check next week by:** looking at the PR list for any branch older than 3
business days without a merge or a documented reason in the PR thread.

---

## 2. Definition of Done

A piece of work is **not done** until every box below is checked — not when
it "works on my machine."

- [ ] Code is merged into `main` through an approved pull request (not just opened)
- [ ] Automated tests exist for the new/changed behavior, and the full test suite passes in CI
- [ ] The build/lint pipeline passes with no errors
- [ ] Someone other than the author manually verified the feature against the
      acceptance criteria written in its issue/ticket
- [ ] Documentation affected by the change is updated (README, API docs, or
      inline comments — whichever applies)
- [ ] The linked issue/ticket is closed and references the merging PR
- [ ] No existing test that was passing before the change is now failing
- [ ] The feature runs on `main` with no undocumented manual setup steps

**Check next week by:** picking any closed issue and confirming its PR link,
passing CI run, and updated docs (if applicable) all exist.

---

## 3. Sprint Length and Ceremonies

- **Sprint length:** 2 weeks.
  *(Reasoning: everyone on this team is carrying a full course load elsewhere;
  a 2-week cadence gives enough time to ship a demoable slice without turning
  every week into a deadline. Revisit this if it isn't working after Sprint 2.)*
- **Sprint planning:** first day of each sprint, 30–45 min, whole team. We
  pull items from the backlog, assign owners, and confirm they fit the
  Definition of Done.
- **Standup:** twice a week, Monday and Wednesday, before class, in the
  team iMessage group chat. Each person posts: what I did, what I'm doing
  next, what's blocking me.
- **Sprint review/demo:** last day of the sprint, 20–30 min, whole team shows
  working software — not slides, not "trust me."
- **Sprint retro:** immediately after the review, 15–20 min. What worked,
  what didn't, one thing we change next sprint.

**Check next week by:** a calendar invite or channel post existing for each
ceremony, and retro notes existing from the previous sprint.

---

## 4. Pull Request and Review Rules

- Every PR requires **at least 1 approval** from someone other than the
  author before it can be merged. No self-merges.
- **Reviewer rotation** (so review responsibility belongs to a person, not
  "the team"): reviewers rotate weekly in this order —
  Roberta Henriquez → Sam Morris → Rommel Mensah → Justin Brown → repeat.
  The person whose week it is is the **primary reviewer** for every PR opened
  that week, unless they are the PR's author, in which case the next person
  in rotation reviews instead.
- **What the reviewer checks**, every time:
  - The change actually does what the PR description says it does
  - Tests are included and passing
  - Code is readable and consistent with the rest of the codebase
  - No unrelated changes snuck into the diff (scope matches the ticket)
  - No obvious security or data-handling issues (e.g., secrets committed, no
    input validation)
- **When a review stalls:** if the primary reviewer hasn't responded within
  **24 hours**, the author pings them directly in the team channel. If there
  is still no response after another 24 hours, the **backup reviewer** —
  the next person in the rotation order — reviews it instead so the work
  doesn't block the sprint.

**Check next week by:** looking at merged PRs for exactly one non-author
approval each, and checking timestamps on any PR that took more than 48
hours to confirm the stall rule was followed.

---

## Revision History
- v0.1 — September 9, 2026 — Initial working agreement, committed to team repository.
