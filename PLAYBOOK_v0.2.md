# Playbook v0.2 — Working Agreement

**Team:** TheCoders — Service Status and Incident Framework
**Course:** Software Engineering
**Members:** Roberta Henriquez, Sam Morris, Rommel Mensah, Justin Brown
**Date:** September 14, 2026

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

- Code is merged into `main` through an approved pull request (not just opened)
- Automated tests exist for the new/changed behavior, and the full test suite passes in CI
- The build/lint pipeline passes with no errors
- Someone other than the author manually verified the feature against the
  acceptance criteria written in its issue/ticket
- Documentation affected by the change is updated (README, API docs, or
  inline comments — whichever applies)
- The linked issue/ticket is closed and references the merging PR
- No existing test that was passing before the change is now failing
- The feature runs on `main` with no undocumented manual setup steps

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
- **AI-assisted review:** see Section 5.3. In short — a tool reviewing a PR
  never substitutes for the primary reviewer's approval, but it can narrow
  what that reviewer needs to check by hand for low-risk changes.

**Check next week by:** looking at merged PRs for exactly one non-author
approval each, and checking timestamps on any PR that took more than 48
hours to confirm the stall rule was followed.

---

## 5. AI Tool Use

*This section supersedes any standalone AI-use agreement previously in
effect for this team.*

### 5.1 Approved Tools

AI tools are approved per use case, not blanket-approved. A tool being
permitted for one purpose does not authorize its use for another.

| Tool | Approved for | Not approved for |
|---|---|---|
| **GitHub Copilot** | In-editor code completion, boilerplate generation, writing tests for already-designed functions, refactoring suggestions within a file the author already understands | Generating entire modules or files unreviewed by the author; writing code the author cannot explain line-by-line in review |
| **ChatGPT / Claude** | Design discussion and brainstorming, drafting non-production documentation (READMEs, design docs, this playbook), drafting the AI-generated incident summaries that are a feature of our product (per the spec, these always require human approval before publishing), debugging discussion, explaining unfamiliar error messages or APIs | Writing production code without the author reviewing and understanding every line; generating commit messages or PR descriptions that misrepresent authorship; making architectural decisions without team discussion |

**General rule:** any AI-generated content that ships in the product, gets
committed to the repo, or gets submitted for a grade must be reviewed,
understood, and taken ownership of by a human teammate before it counts as
"our work." AI tools assist; they don't author.

**Check next week by:** spot-checking a merged PR that used Copilot or
Claude/ChatGPT and confirming the log entry (5.2) matches what the tool was
actually used for, per this table.

### 5.2 Prompt-Log Policy

- **What gets logged:** any prompt/response exchange that materially shaped
  code that gets merged, a design decision, or content in a graded
  deliverable. Trivial exchanges (e.g., "what does this error mean") don't
  need logging. The test: if a teammate reviewing the PR would want to know
  an AI was involved in producing this, log it.
- **Where it lives:** `/ai-logs/` in the team repo, one file per contributor
  (`ai-logs/roberta.md`, `ai-logs/sam.md`, `ai-logs/rommel.md`,
  `ai-logs/justin.md`), each entry dated with a one-line summary of the
  prompt's purpose and a link to the PR or commit it fed into. Full
  transcripts aren't required — a summary plus what was accepted, rejected,
  or edited is enough.
- **Who is responsible:** each teammate logs their own AI use as part of
  finishing the PR that uses it — logging happens before requesting review,
  not after. Roberta, as team lead, spot-checks the log during weekly
  standup and flags gaps, but authorship of the log entry stays with
  whoever ran the prompt.

**Check next week by:** picking any PR that used an approved AI tool and
confirming a matching entry exists in that contributor's `/ai-logs/` file.

### 5.3 AI-Assisted Code Review Policy

**Position: AI review partially satisfies the Section 4 approval
requirement — it does not replace the primary reviewer's approval, but it
changes what that reviewer is doing.**

If a tool (e.g., Copilot's PR review, or a teammate pasting a diff into
Claude/ChatGPT for a second opinion) reviews a pull request, that AI pass
counts as the *first* pass, not the *only* pass. The primary reviewer named
under Section 4's rotation still must review and approve every PR before
merge. What changes is scope: for low-risk, routine changes (formatting,
test additions, small refactors with no behavior change) where an AI review
has already flagged no concerns, the primary reviewer can do a
lighter-weight check — confirming the AI's read is correct and scanning for
what an AI reviewer characteristically misses (business logic correctness,
whether this actually matches the incident-management spec, security
implications) — rather than a full line-by-line review.

**Why:** our product depends on humans approving AI output before it's
trusted — the AI-drafted incident summaries have to clear human approval
before publishing — so it would be inconsistent to hold our own code to a
lower bar than we're building into the product. At the same time, treating
an AI review as worthless ignores that it reliably catches a real class of
issues (typos, obvious null-pointer risks, style violations) faster than a
human skim would; pretending otherwise just means reviewers rubber-stamp
PRs to save time, which is a worse outcome than a scoped, honest review.
Requiring full human review on every PR regardless of AI input sounds
safer on paper, but under deadline pressure in week nine it becomes the
rule that actually gets skipped. A rule that scales review effort to risk
is one the team will actually follow.

**In practice:**
- AI review output (Copilot suggestions, a pasted-in Claude/ChatGPT review)
  can be included in the PR description as context for the primary
  reviewer.
- The primary reviewer is still the one who clicks "Approve," and is
  accountable for that approval regardless of what a tool said.
- If a PR touches the incident state machine, notification delivery, or
  anything security/data-related, it always gets a full human review — no
  AI-assisted shortcut, regardless of what any tool flagged.

**Check next week by:** picking a merged PR tagged low-risk and confirming
the primary reviewer's approval and the AI tool's output are both visible
on the PR; picking a merged PR touching the state machine or notifications
and confirming it got a full review with no shortcut noted.

---

## Revision History
- v0.1 — September 9, 2026 — Initial working agreement, committed to team repository.
- v0.2 — September 14, 2026 — Added Section 5 (AI Tool Use): approved tools,
  prompt-log policy, and AI-assisted review policy. Supersedes any standalone
  AI-use agreement. Section 4 cross-referenced to point to the new review
  policy.
