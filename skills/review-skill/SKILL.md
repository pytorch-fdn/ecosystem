---
name: review-skill
description: "Evaluate a PyTorch Ecosystem application (a PR or Issue in pytorch-fdn/ecosystem) against the Ecosystem Working Group acceptance checklist. Produces a short GitHub-flavored-markdown summary, written to an editable local file so the reviewer can revise it before pasting as a PR/issue comment, plus a local verification sidecar file with the links and expanded evidence behind each row. Use when asked to review, evaluate, or assess an ecosystem application, PR, or issue for inclusion."
metadata:
  tags:
    - pytorch
    - ecosystem
    - review
    - checklist
    - pull-request
    - open-source
---

# PyTorch Ecosystem Application Review

## Purpose

Produce a consistent, evidence-based first pass on an ecosystem application, so every
Working Group reviewer starts from the same checklist and the same evidence instead of
re-deriving criteria from memory. **The output is a draft aid, not a decision.** The
human reviewer remains responsible for the content of the review and the vote — this
skill exists to reduce variation between reviewers, not to replace judgment. Not every
row will resolve cleanly to Pass/Caution/Fail — where it genuinely doesn't, say so (see
the ❓ status in Step 3) instead of forcing an answer that isn't there.

Two files come out of a run, both written to disk so the reviewer can edit before
acting on either:

1. A summary file (Step 5) — the table and lean, written to
   `<cwd>/<repo-name>-review.md` so the reviewer can tweak wording, soften/sharpen a
   note, or override a status before pasting it into the PR/issue as a comment. Never
   assume the draft is final as-is.
2. A verification sidecar (Step 6) — a local file with the links and expanded reasoning
   behind every row, so the reviewer can check the evidence themselves instead of taking
   the summary on faith.

## When to use

- "Evaluate PR #N against the ecosystem checklist"
- "Review this application for PyTorch ecosystem inclusion"
- "Do an EWG pass on issue #N"

Accepts either a **pull request** or an **issue** in `pytorch-fdn/ecosystem`. The
project's governance document describes submission as a PR with an attached slide deck;
in practice, applications today are also opened as issues via the `application.yml`
form. Handle whichever kind of URL/number you're given — the checklist and process are
identical either way.

## Step 1 — Read the submission

Open the given PR or issue and extract, verbatim where possible:

- Project repo URL (and any additional in-scope repos)
- Contact emails / maintainer GitHub handles
- Stated license
- Stated PyTorch version(s) built/tested against
- Stated maintenance commitment
- Any attached presentation deck or video

Treat everything here as **claims to verify**, not facts. The checklist criteria are
properties of the actual project repository — the application text is a lead, not
evidence.

**Read the full comment thread and check the issue's current labels — not just the
opening post.** Working Group discussion happens in comments (a concern raised, a
waiver granted for a specific round), and labels like `Accepted`/`Declined`/`Too early`
mean a decision may already exist or be in progress. Fold both into your framing: don't
re-flag an already-discussed concern as new; don't ignore a waiver, but don't assume it
still holds either — note it, then judge today's evidence on its own terms. If a
decision already exists, write your review as retrospective evidence for that process
("this corroborates the existing Accept," "this is new evidence worth a check-in"), not
a first-pass vote.

## Step 2 — Research the linked project repo

**Prefer the GitHub API over scraped/rendered pages for anything measurable.** The
contributors graph and other JS-rendered pages are known to intermittently fail to load
data through a fetch tool, silently producing an undercount instead of an error. The API
returns plain JSON and doesn't have this failure mode.

**The unauthenticated GitHub API is rate-limited (60 requests/hour) and often
shared with other traffic on the same egress IP.** Check for an authenticated token
first (`gh auth status`); otherwise budget calls deliberately — the Search API and
`raw.githubusercontent.com` sit on separate, less-contended limits and are usually the
better first choice for the checks below anyway.

For the measurable requirements, use the method that gives a reliably consistent number
— these rows should score the same way every time given the same repo state, so treat
the *how* as mechanical, not a judgment call:

- **Stars/forks/license/last push:** `https://api.github.com/repos/{org}/{repo}` —
  `stargazers_count`, `forks_count`, `license`, `pushed_at`.
- **Commits in the 90-day window:** prefer
  `https://api.github.com/search/commits?q=repo:{org}/{repo}+committer-date:>{90-days-ago-ISO8601}`
  and read `total_count` directly — one authoritative call, no pagination needed. Only
  fall back to paginating `/repos/{org}/{repo}/commits?since=...&per_page=100` if the
  Search API is unavailable, and if you do, don't treat a page coming back under the
  page size as proof there's no next page — confirm via the `Link` header (or a
  genuinely empty next page) before treating the count as final. If a rate limit cuts
  pagination short, say so and report the count as a lower bound, not a finished number.
- **Contributors active in the 90-day window:** count distinct authors of commits that
  actually landed on the default branch in that window, not authors of open-but-unmerged
  PRs — the two can diverge sharply (many PR authors, almost no landed commits), and the
  checklist means the former.
- **CI status:** don't assume GitHub Actions is the only CI system in play — check
  `/repos/{org}/{repo}/actions/workflows` first, and if it's empty or thin, also check a
  recent commit's combined status/check-runs (`/commits/{sha}/status` or `/check-runs`),
  which surfaces externally-hosted CI (e.g. Buildkite) wired in via commit statuses.
  Identify which specific workflow/check would actually substantiate each claimed
  capability (a PyTorch-version matrix, CUDA/GPU, multi-OS) and score off *that one's*
  pass/fail history — an unrelated green check doesn't offset a red one, and "the
  overview page looked green" isn't evidence either way.

**Check time-sensitive facts live, every run — never from memory.** "Current PyTorch
version" and "today's date" both drift. Fetch `https://pypi.org/pypi/torch/json` (or the
PyTorch releases page) for the current version and cite it with its source and date in
the sidecar before scoring PyTorch Support or Ongoing Maintenance.

**Every repo the application declares in scope needs its own check, not just the
primary one** — confirm each additional declared repo actually resolves (not
private/404), and carry that finding into every row it could affect (Documentation,
Permissive Licensing, PyTorch Support) rather than mentioning it once and never
revisiting it. The same applies in reverse: if the pitch describes functionality that
isn't in *any* declared repo (e.g. the public repos are a thin shim around a separate
proprietary product), that's a scope finding in its own right — describe the mismatch
rather than scoring only the public part as if it were the whole pitch. If a declared
repo URL redirects to a new canonical location (an org/repo rename or transfer), treat
it as the same project and note the transfer — that's not a scope problem.

For the project repo (and any additional repos declared in scope), also check:

- Releases — latest release version and date, release cadence
- Required files — check **both** repo root and `.github/` for each:
  - `LICENSE` / `LICENSE.md`
  - `README.md`
  - `CONTRIBUTING.md`
  - `CODEOWNERS`
  - `CODE_OF_CONDUCT.md`
  - `GOVERNANCE.md` (or equivalent formal governance doc)
- Packages — search PyPI and Conda/Anaconda for the package name (repo name and obvious
  variants), then confirm it's actually *this* project's, not just similarly named: the
  PyPI JSON's `project_urls`/`home_page` (or the Conda recipe's source) should link back
  to the declared repo. A git-URL source install or an unaffiliated same-named package
  doesn't satisfy this row — note it as unverified/mismatched rather than crediting a
  Pass; a name-only match is a real supply-chain risk to hand a reviewer.
- Documentation — in-repo docs plus any external docs site linked from the README
- PyTorch integration — confirm PyTorch is an actual dependency/integration point, not
  just mentioned in passing; note the minimum/maximum PyTorch version supported

Note every 404, missing page, or inability to verify explicitly — don't silently treat
"couldn't check" as "passes." **Keep every URL you actually fetched or checked** (repo
page, commits page, contributors page, specific file blob URLs, PyPI/Conda page,
Actions page) — Step 6 turns these into a verification sidecar so the reviewer doesn't
have to redo your research to trust it.

## Step 3 — Score against the checklist

Score every row below. Use exactly these four statuses:

| Status | Meaning |
|--------|---------|
| ✅ Pass | Criterion is clearly and fully met, with evidence observed directly |
| ⚠️ Caution | Partially met, met with a real caveat, or evidence could not be fully gathered |
| ❌ Fail | Not met |
| ❓ Needs human judgment | You have the evidence, but scoring it is a genuine judgment call this checklist doesn't resolve |

**❓ is not a softer ⚠️ Caution, and it isn't an escape hatch from doing the research.**
Gather the evidence fully first. ⚠️ Caution is a statement about the *evidence*: you have
a clear read on what you found, and it's genuinely partial, caveated, or unconfirmable.
❓ is a statement about the *rubric*: the evidence is complete, but you can construct two
defensible readings that land on different statuses, or the situation isn't a shape this
checklist anticipated. When you use ❓, the Note must say what the fork is and what a
human would need to decide — "not sure" alone isn't usable. Reserve it for rows where you
can actually articulate the disagreement, not as a default when something takes effort.

### Functional Requirements

| Criterion | Description |
|-----------|-------------|
| PyTorch Support | Must demonstrate benefit to the PyTorch community and be validated with the latest versions of PyTorch. |
| Working CI | Demonstrable working CI workflow for all declared architectures. |
| Governance | Documented technical governance defined and implemented, represented in the repo (e.g. a `GOVERNANCE.md` — see the LF Minimum Viable Governance framework). |
| User Experience | Value proposition clearly documented in the README. |
| Quick Start | Working, demonstrable quick start. |
| Documentation | Clear and comprehensive documentation. |
| Permissive Licensing | e.g. BSD-3, Apache-2, MIT — no licensing concerns for users. |
| Functional Testing | Tests submitted as part of the project; confidence it works with the latest PyTorch releases. |
| Packages | Binary installation support via pip **or** Conda — either one alone is sufficient to Pass; only fail this if **neither** exists. |
| Ongoing Maintenance | Maintainers committed to 12+ months of support, and to updating for the latest two PyTorch releases. |

> **Don't double-count the same gap across PyTorch Support and Working CI.** A missing
> PyTorch-version test matrix in CI is a **Working CI** gap — score it there. **PyTorch
> Support** is about whether PyTorch is a genuine, direct integration point and whether
> the applicant's version claim is plausible (matches a real current release, dependency
> spec isn't absurd) — score it on that alone. If the same CI gap pulls both rows down,
> say so in both notes so a reader sees it's one gap, not two — but don't let it demote
> PyTorch Support from a Pass on the strength of a Working CI shortfall already penalized
> in its own row. Claimed capabilities CI doesn't actually exercise (CUDA, multi-OS, a
> specific PyTorch version) are a Working CI finding — see Step 2 for how to check.

### Measurable Requirements

| Criterion | Threshold |
|-----------|-----------|
| Core Maintainers | ≥ 2 |
| Contributors | ≥ 5 total contributors active in the last 90 days |
| Commits | ≥ 30 commits in the last 90 days |
| Stars | ≥ 200 GitHub stars |

### GitHub Layout Requirements

| File | Requirement |
|------|-------------|
| LICENSE.md | At repo root. Note license of any bundled third-party code if documented. |
| README.md | Welcomes new contributors, explains the project's value and how to get started; should include release methodology/cadence. |
| CONTRIBUTING.md | Explains contribution types and the development process. |
| CODEOWNERS | Defines individuals/teams responsible for code; ideally documents retired committers too. |
| CODE_OF_CONDUCT.md | Present; default expectation is the Linux Foundation CoC unless an alternate was pre-approved. |

### Disambiguation rules

These fall into two kinds. **Metrics** rules are mechanical — apply them the same way
every time, since consistency is the whole point of a checklist for these rows.
**Judgment** rules are guidance, not a lookup table — use them to inform your reasoning,
and reach for ❓ when a case genuinely falls outside what they cover, instead of forcing
it into Pass/Caution/Fail by resemblance to the nearest example.

**Metrics — apply mechanically:**

- License file exists but named `LICENSE` (no `.md`) → ⚠️ Caution, not a fail — note the
  naming gap.
- `CODEOWNERS` present only at `.github/CODEOWNERS` → ✅ Pass; that location is valid,
  don't penalize it.
- `CODE_OF_CONDUCT.md` uses Contributor Covenant instead of the LF CoC → ⚠️ Caution —
  note it requires prior WG approval as an alternate.
- Governance mentioned only informally inside `CONTRIBUTING.md` or an "about" page, with
  no standalone governance doc → ❌ Fail. This is a hard requirement — informal mention
  is not a substitute for a documented process, however credible the mention is.
- pip package exists, no Conda (or vice versa) → ✅ Pass, **provided the package is
  verifiably this project's** — its PyPI/Conda metadata links back to the declared repo,
  not just a name match (see Step 2). The checklist requires binary installation via pip
  **or** Conda, not both — don't penalize a project for only shipping one. Only mark this
  ⚠️ Caution/❌ Fail if **neither** pip nor Conda has a working, verifiably-linked package
  — a git-URL source install or an unaffiliated same-named package don't count.
- A measurable threshold (stars, contributors, commits) is met exactly at the floor →
  ✅ Pass, but say so explicitly in the note ("exactly at the 200-star minimum") so the
  reviewer can weigh the thin margin themselves.
- Any criterion you could not verify (page didn't load, private repo, rate-limited with
  no workaround, etc.) → ⚠️ Caution, and say what you tried and why it was inconclusive.
  Never silently score unverifiable items as ✅. (This is different from ❓ — here the
  *evidence* is the problem, not the *rubric*.)
- **Partial counts cut both ways.** An incomplete count that already clears a threshold
  is a Pass, not a Caution — more data only adds past activity, never removes it; note
  it's a partial sample. An incomplete count still under the threshold is not
  automatically a Fail — it may just mean you haven't looked far enough (see Step 2 for
  how to get a count you can trust). Only score ❌ Fail once you trust the count is
  complete; if a rate limit cut it short, score ⚠️ Caution and say how far you got.

**Judgment calls — use as guidance; flag ❓ when a case doesn't fit:**

- PyTorch used only indirectly (e.g. routes to a PyTorch backend without PyTorch as a
  direct dependency) → generally ⚠️ Caution, but if you genuinely can't tell how central
  PyTorch is to the value being delivered, ❓ is more honest than guessing.
- Core Maintainers: a formally-documented second maintainer (CODEOWNERS, GOVERNANCE,
  MAINTAINERS.md) is much stronger evidence than an informal mention on a team/about
  page. One person named with no other evidence anywhere → ❌ Fail. An informally-named
  second person with no corroborating file but no reason to doubt it either is a real
  split — use your judgment, or ❓ if you can't decide whether "informal but plausible"
  clears the bar.
- Permissive Licensing and scope: check beyond the root `LICENSE` file and the README's
  opening section — proprietary components or restricted tiers are often disclosed
  elsewhere (changelog, docs site, pricing page). If Quick Start, Documentation, or
  PyTorch Support would only Pass by crediting functionality outside what's actually
  declared/licensed in scope, say so rather than passing on the strength of it.
- Score User Experience and Quick Start on whether the *declared, in-scope* repo
  delivers what it promises — not penalized for unrelated pricing/tier disclosures
  (that's a Licensing finding), but not passed on a quick start that only works via
  something outside scope either. A reviewer's own missing local dependency isn't
  evidence the project's quick start is broken.
- A required file's substance delegated entirely to an external link (a link-only
  `CONTRIBUTING.md`, a README's release cadence living only in `RELEASE.md`) is weaker
  evidence than the same content living in the file itself — how much weaker depends on
  how substantive and clearly-linked the destination is (follow it and check, don't
  assume). Where that's a close call, ❓ is a legitimate answer.

## Step 4 — Self-check: does the evidence actually support the call?

Before writing anything, reread each row you scored in Step 3, for two different failure
modes.

**First — does the Note support the Status?** For each row, ask: **if someone read only
the Note, would they guess the same Status I assigned?** This catches the most common
failure mode of this skill — writing evidence that describes a gap, then scoring the row
as if the gap didn't exist.

- If a Note says something is "documented" only partially, only in one place, or
  "doesn't contain X section that the checklist asks for" — the Status must be ⚠️
  Caution or ❌ Fail, never ✅ Pass. A Pass row's Note should read as unqualified
  evidence, not evidence-with-a-caveat.
- If a Note says a claim "wasn't independently verified" or "couldn't be confirmed" —
  the Status must be ⚠️ Caution at most, per the disambiguation rules above.
- If you catch a mismatch, fix the **Status**, not the Note — the Note is the evidence
  you actually gathered; don't soften true evidence to match a status you'd rather give.

In testing, this exact failure mode produced a review that scored README.md ✅ Pass
while its own Note said the required release-methodology section was missing. The
mismatch can hide in any row — check all 19, every run.

**Second — introspect on the judgment calls.** For any row you scored using guidance
rather than a mechanical rule (the "Judgment calls" in Step 3), ask honestly: would a
colleague reviewing the same evidence have a real chance of landing on a different
Status — not from missing something, but because it's genuinely open to more than one
reading? If so, don't suppress that by picking whichever status feels more defensible on
paper — use ❓ instead, and write the Note as an honest account of the fork. A
confidently-wrong status is worse for the reviewer than an honestly-flagged open
question.

Do this for all 19 rows before moving to Steps 5 and 6.

## Step 5 — Write the summary file

Write **GitHub-flavored markdown** to `<cwd>/<repo-name>-review.md`. This is a draft for
the reviewer to read and edit, not a final artifact to paste unmodified — the content
should be sized to paste directly as a PR or issue comment once the reviewer is happy
with it, but the file itself is the deliverable, not chat output. Use exactly this
structure:

```markdown
## Ecosystem Review: {Project Name}

**Applicant repo:** https://github.com/{org}/{repo}
**Application:** {PR or Issue URL}
**Reviewed:** {today's date}

{1 sentence: what the project does, and where it sits relative to existing ecosystem
projects if there's a specific overlap/analog worth naming. Not a restatement of the
whole table — the applicant already knows what they built.}

{A short bulleted list — typically 3-6 items — of the load-bearing issues: the specific
Fails/Cautions that actually justify the Suggested lean below, in roughly descending
order of severity. Skip this list entirely if the lean is Approve and there's nothing
substantive to flag. Don't pad it out to look thorough — if there are only two real
issues, list two.}

| Criterion | Status | Notes |
|---|---|---|
| PyTorch Support | ✅/⚠️/❌/❓ | |
| Working CI | ✅/⚠️/❌/❓ | |
| Governance | ✅/⚠️/❌/❓ | |
| User Experience | ✅/⚠️/❌/❓ | |
| Quick Start | ✅/⚠️/❌/❓ | |
| Documentation | ✅/⚠️/❌/❓ | |
| Permissive Licensing | ✅/⚠️/❌/❓ | |
| Functional Testing | ✅/⚠️/❌/❓ | |
| Packages | ✅/⚠️/❌/❓ | |
| Ongoing Maintenance | ✅/⚠️/❌/❓ | |
| Core Maintainers (≥2) | ✅/⚠️/❌/❓ | |
| Contributors (≥5, 90d) | ✅/⚠️/❌/❓ | |
| Commits (≥30, 90d) | ✅/⚠️/❌/❓ | |
| Stars (≥200) | ✅/⚠️/❌/❓ | |
| LICENSE.md | ✅/⚠️/❌/❓ | |
| README.md | ✅/⚠️/❌/❓ | |
| CONTRIBUTING.md | ✅/⚠️/❌/❓ | |
| CODEOWNERS | ✅/⚠️/❌/❓ | |
| CODE_OF_CONDUCT.md | ✅/⚠️/❌/❓ | |

**Suggested lean:** {Approve / Needs more info / Too early / Decline} — {one-sentence
rationale}.

*This is a draft aid to speed up review and reduce inconsistency between reviewers —
the Working Group member remains responsible for the final assessment and vote.*
```

**Leave the Notes cell empty for a Pass that isn't interesting.** A clean, unremarkable
Pass doesn't need a sentence restating the Description column — the full evidence still
lives in the sidecar for anyone who wants it. Every ⚠️ Caution, ❌ Fail, and ❓ row still
needs a specific Note, and so does any ✅ Pass with a real caveat worth flagging (e.g.
"exactly at the threshold," "pip only, no Conda"). A table with several blank Notes cells
next to clean passes is the goal, not a gap to fill in — don't manufacture a sentence
just because the cell is there. Where a Note is present, it must cite specifics (exact
counts, file paths, license names, version ranges) — never approximate when the real
number is checkable.

Every Status cell must use one of the exact four strings `✅ Pass`, `⚠️ Caution`,
`❌ Fail`, or `❓ Needs human judgment` — not a bare emoji, abbreviation, or other
wording; this table is meant to be pasted as-is. If two or more rows are ❓, the
Suggested lean should almost always be "Needs more info," and the one-sentence rationale
should name which rows need a human call.

After writing the file, tell the reviewer where it is and that it's editable — don't
just print the same content again in chat as if that were the deliverable.

## Step 6 — Write the verification sidecar

The table in Step 5 is deliberately terse so it's pasteable. That terseness means a
reviewer has to trust your notes on faith unless they redo the research themselves. The
sidecar closes that gap: **one link and one expanded paragraph per checklist row**, so
the reviewer can click straight to the evidence and form their own judgment in seconds
instead of minutes.

Write it to `<cwd>/<repo-name>-verification.md`. This file is a local working artifact
for the reviewer running the skill — it is **not** meant to be pasted into the PR/issue,
and it is not part of the ecosystem repo's checked-in content.

Use this structure, one entry per checklist row from Step 3, in the same order as the
summary table:

```markdown
# Verification Sidecar: {Project Name}

Companion to the pasted review on {PR or Issue URL}. Every claim in that table is
expanded here with the source(s) used and enough detail to independently confirm or
challenge the status. Generated {today's date}.

## PyTorch Support — ✅/⚠️/❌/❓

- https://github.com/{org}/{repo} — repo homepage, dependency on PyTorch
- https://github.com/{org}/{repo}/blob/main/{setup.py or pyproject.toml} — declared
  torch version constraint

{2-4 sentences: what was checked, what was found, and — if the status is ⚠️ or ❌ —
exactly what's missing or what would need to change to move the status. If a claim in
the application couldn't be independently confirmed, say so here explicitly and suggest
the specific follow-up question to ask the applicant.}

## Working CI — ✅/⚠️/❌/❓

- https://github.com/{org}/{repo}/actions — workflow runs checked
- ...

{expanded paragraph}

... (repeat for every row in the Step 5 table, in the same order: PyTorch Support,
Working CI, Governance, User Experience, Quick Start, Documentation, Permissive
Licensing, Functional Testing, Packages, Ongoing Maintenance, Core Maintainers,
Contributors, Commits, Stars, LICENSE.md, README.md, CONTRIBUTING.md, CODEOWNERS,
CODE_OF_CONDUCT.md)

## Claims not independently verified

{Bulleted list of any applicant-supplied numbers or claims (benchmark results,
validation percentages, "long-term maintenance" pledges, etc.) that could not be
checked against the repo itself. Each bullet should name what was claimed, why it
couldn't be verified, and what would verify it (e.g. a published benchmark script, a
reproducible eval, a second maintainer accepting a formal role).}
```

- Every checklist row needs **at least one** link, even for a clean ✅ Pass — a link to
  the exact commits/contributors/file page checked, not just the repo root, so the
  reviewer isn't left re-discovering where the evidence lives.
- For file-existence rows (LICENSE.md, CONTRIBUTING.md, CODEOWNERS, CODE_OF_CONDUCT.md,
  governance), link the exact blob URL you checked — including the ones that 404'd, so
  the reviewer can re-check after the applicant adds them without guessing the path.
- For measurable rows (stars, commits, contributors), link the specific page/API
  endpoint used and state the exact number observed and the date it was observed on —
  these numbers move, so timestamp them.
- Keep the expanded paragraphs honest about uncertainty: "the visible commit sample
  suggests X but wasn't a complete count" is more useful to a reviewer than a
  confident-sounding guess.
- For any ❓ row, the paragraph must do three things: state the evidence you gathered
  (same as any other row), name the two or more readings you can construct from it, and
  say what a human reviewer would need to decide to resolve it. "Unclear" on its own is
  not a usable sidecar entry.

## Important reminders

- **Governance is a hard requirement.** A project can be excellent everywhere else and
  still fail this row if there's no standalone governance document.
- **90-day windows** are computed from today's date, not from when the application was
  submitted.
- **"Latest two PyTorch releases"** — check the current PyTorch release page for what
  that means today; it changes over time.
- **Don't let a good pitch substitute for verification.** The application text (and any
  attached slide deck) is the applicant's framing — every checklist row needs its own
  evidence from the actual repo.
- **The "Suggested lean" is not a vote.** Frame it as input, and never state or imply
  that the review is final or binding.
- **The sidecar is what makes the summary trustworthy.** A terse status table is only
  useful if the reviewer can verify it faster than doing the research from scratch —
  that's the sidecar's entire job. Don't skip it, and don't let it go stale relative to
  the summary (same rows, same order, same statuses).
