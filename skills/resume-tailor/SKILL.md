---
name: resume-tailor
description: >-
  Tailors ATS-ready resumes and matching cover letters to a specific job
  posting, then verifies them with a second-model (codex) review. Use this
  skill whenever the user pastes or references a job posting / job description
  and wants a resume, CV, or cover letter for it — including phrasings like
  "make me a resume for this role", "tailor my CV to this posting", "I need a
  cover letter for X", "apply to this", or when they give a position title and
  ask for application documents. Also use for government / public-sector /
  competency-based applications, which need a specific format. Produces
  LaTeX→PDF output plus a plain-text ATS companion.
---

# Resume Tailor

Turn a job posting into a tailored, ATS-ready **resume + cover letter** (PDF
and plain-text), grounded in a master fact sheet and verified by a read-only
`codex` review. The whole point is to be **persuasive without ever
fabricating** — every claim traces to confirmed ground truth, and honest
hedges ("familiar / ramping", "ITIL-equivalent") are used where the fit is
partial.

## Operating context

This skill works in a LaTeX resume project containing:
- `awesome-cv.cls` + `fonts/` (the [Awesome-CV](https://github.com/posquit0/Awesome-CV) template). If `awesome-cv.cls` is missing, fetch it: `curl -fsSL -o awesome-cv.cls https://raw.githubusercontent.com/posquit0/Awesome-CV/master/awesome-cv.cls`
- `MASTER-PROFILE.md` — the candidate's ground-truth fact sheet (roles, exact dates, hours/week, confirmed skills, honest framings). **This is the anchor for everything.** If it doesn't exist, copy `assets/MASTER-PROFILE-TEMPLATE.md` (bundled with this skill) into the project, then interview the user to replace the example character with their real facts.
- Optionally, existing `*-resume-*.tex` variants to copy from as templates.

Compile with `xelatex` (run twice). Default to **US Letter** (`letterpaper`)
for Canada/US (A4 elsewhere).

## The pipeline

Work through these steps in order. Use a TodoWrite list to track them.

### 1. Read the master profile
Read `MASTER-PROFILE.md` first — it is the only source of claimable facts. If
it doesn't exist, build one from `assets/MASTER-PROFILE-TEMPLATE.md`. If the
user supplied an existing resume/cover letter for this posting, read those too;
they're authoritative for that role's voice and any role-specific facts.

**A thin profile produces thin resumes.** If the profile looks sparse for an
experienced candidate, enrich it before building: scan their **GitHub**
(`github.com/<user>?tab=repositories`), HuggingFace, portfolio, and any local
project folders to capture real projects, tech, production status, and repo
visibility — then write the verified finds into `MASTER-PROFILE.md`. Mine any
existing academic-CV sources (publications, funding, invited talks, software)
too. The candidate's **own** public repos/projects and CV sources are
reasonable to record as verified; **flag anything uncertain** (ambiguous
ownership, unclear production status) for the user to confirm before it's
claimed. Real depth on the page is the difference between a standout and a
generic resume.

### 2. Parse the posting
Extract: required + preferred skills, responsibilities, **competencies** (esp.
for government roles), residency/clearance requirements, education requirement,
and format hints. Note the employer type — it picks the format.

### 3. Gap check — the anti-fabrication gate (most important step)
Cross every JD requirement against `MASTER-PROFILE.md`:
- **Backed by the profile** → claim it.
- **Partially / adjacent** → use an honest hedge from the profile's "Honest framings" (e.g. a non-core framework = "(familiar)"; a standard you meet via an equivalent regime = "X-equivalent").
- **Not in the profile at all** → **STOP and ask the user** ("The posting wants X — do you have hands-on X? I'll only claim what you confirm."). Never invent a skill, tool, certification, employer, date, or metric. When the user confirms new facts, write them into `MASTER-PROFILE.md` so they're captured for next time.
- **Subjective / personal fields are the candidate's to name — ask, never guess.** "The accomplishment you're most proud of," motivation ("why us"), career goals, relocation willingness, values — these are personal. Ask the user; do not infer them from the profile. A guessed "proudest accomplishment" reads as inauthentic and the candidate will notice. (**Salary** is a special case: you *may* offer a market-informed range as advice — see Guardrails — but the candidate's actual floor/target is theirs to set; don't put a number into an application without asking.)

### 4. Choose format and build
Read the right format reference before building:
- Standard tech / private-sector ATS → `references/format-standard-ats.md`
- Government / public-sector / competency-based → `references/format-gov-competency.md`
- **Universities, research institutes, AI/ML institutes, national labs, PI/research/fellowship roles** → `references/format-academic-cv.md` — a **full ~3-page academic CV** with publications, research funding/grants, invited lectures, research software/projects, professional memberships, and teaching/supervision. Research institutes read academic CVs; a terse 2-page industry resume **sells the candidate short** there.

Then build (copy the closest existing `*-resume-*.tex` variant if one exists,
else start from the template). Conventions that always apply:
- US Letter, real institution names, month/year dates, reverse-chronological.
- Headline tagline (positioning) under the name. For **government / regulated** employers use **official titles** on experience entries (they verify; a reference check surfaces the real title). For **private-sector tech** the candidate may prefer **functional titles** (e.g. "Senior Software Engineer" instead of an internal title) — allowed, but never invent seniority or misrepresent the role. If unsure which applies, ask.
- **Hedge partial-competence skills up front — don't make codex catch them.** If the profile marks something "familiar / dabbled / working knowledge", tag it `(familiar)` in the skills list and do NOT list it parallel to core strengths in the summary. Frame it as breadth / fast-ramp, not depth. Same for tenure: use the profile's split ("N years in software, much of it web") rather than implying the whole career was the narrow thing the posting wants.
- **NEVER drop roles or truncate work history to hit a page target.** Keep the **full reverse-chronological work history** on every variant. Tailor the bullet *descriptions*, emphasis, density, and section order to the posting — but do not remove employers/roles unless the user explicitly asks. **Trimming history to fit a theme is the #1 way to sell a senior candidate short.** Recent/relevant roles get full tailored bullets; older roles get brief role-appropriate lines under "Earlier Experience" — but all stay present.
- **Length follows the format, not a fixed cap.** Standard industry resume → a *full* 2 pages (for a 20+ year candidate, one page under-sells). Academic/research CV → a *full* ~3 pages (see academic format). Aim for a clean, full final page — see page-fill discipline in step 6. Page count follows from the full history, not the other way around.
- Generate a plain-text `.txt` companion that mirrors the PDF for paste-into-form ATS fields (ASCII, accented names transliterated, `=` section dividers).

Build the matching **cover letter** (1 page, the candidate's voice, mapped to
the posting's requirements/competencies) using the chosen format reference.

### 5. Codex review — always run, then iterate
Run a read-only second-model review and act on it. This catches overclaims,
internal contradictions, and format issues a single model misses:

```bash
codex exec --sandbox read-only "Review <resume.tex> and <.txt> (and <coverletter.tex>) for <employer> <role>. Ground truth is MASTER-PROFILE.md in this directory — cross-check every claim against it. Report under 300 words grouped by severity: (1) fabrication/overclaim vs the profile, (2) internal/cross-doc inconsistency, (3) JD-fit gaps, (4) ATS/format issues, (5) go/no-go. Do NOT edit files."
```

Then: **apply the strictly-safer fixes yourself** (wording that reduces
overclaim, factual corrections), and **surface judgment calls** (tone, title
choices, anything touching the candidate's voice) to the user rather than
deciding for them. Re-run codex after substantive changes.

**Known false positives — don't loop on these.** Codex judges only against the
files it can see, so it keeps flagging things that are actually fine. Once
you've fixed the substantive overclaims, stop iterating and report — don't
chase a clean "GO" forever. Expected false positives:
- A **confirmed skill that's narrowly sourced** (e.g. a language used only in one component) — codex reads any listing as a "core depth" claim. It's legitimate to list; keep it.
- **Functional titles** for private-sector tech — codex flags them as "inflated vs official"; if it's the candidate's accepted preference, keep them.
Tell the codex prompt about these up front so it stops re-raising them. Real
overclaims (unsupported tools, fabricated metrics, over-stated competence,
tenure inflation, contradictions) always get fixed.

### 6. Compile and report
Compile each `.tex` twice with `xelatex`, clean `*.aux *.log`, and report page
counts + any residual flags. Confirm the resume and cover letter are
internally consistent (same dates, titles, location, tenure framing).

**Page-fill discipline — avoid awkward fractional pages.** A resume should be a
*clean, full* 1, 2, or 3 pages — never a page-and-a-third (last page only
~15-40% full), which reads as "ran out of content." Measure where the **body
text** ends, ignoring the footer (`\makecvfooter` sits at the bottom of every
page and fools naive measurement — exclude content in the bottom ~10%). With
`pdftotext -bbox <pdf> -`: take the max `yMax` of words whose
`yMax/page-height < 0.90` on the last page = where body content ends. If the
last page is awkwardly thin, **expand with supported material** (more bullets,
a fuller projects/publications section, the full work history) — preferred for
senior candidates — or tighten to the prior page. Never pad with filler.

**Verify public links before including them.** Any GitHub/HuggingFace/portfolio
URL must be confirmed reachable+public first
(`curl -s -o /dev/null -w "%{http_code}" https://api.github.com/repos/<user>/<repo>`
→ 200 public, 404 private/missing). Drop or replace dead/private links.

**Combined single PDF** (common request — "resume and cover letter in one
file"): merge cover-letter-first with
`qpdf --empty --pages cover.pdf resume.pdf -- <name>-application-<company>.pdf`.

## Default outputs per position
**Name every file after the company** so applications never collide and the
user can tell at a glance which posting a file belongs to. Use a lowercase,
hyphenated company slug (e.g. "Felix" → `felix`). When one employer has
several reqs, append a short role/req tag (`-cloud`, `-req83894`).
- `<name>-resume-<company>.tex` → PDF + `.txt`
- `<name>-coverletter-<company>.tex` → PDF
- **Combined application PDF** (on request — "one file"): `<name>-application-<company>.pdf` = cover letter + resume merged with `qpdf` (see step 6).
- **Free-form application answers** (on request — when a posting has open-text fields like "why us / proudest accomplishment / problem you solved"): `<name>-answers-<company>.md`. Each answer a **self-contained, plain, pasteable prose block** (no markdown headers/bold — they show as literal `**` in form fields), grounded in the profile, with personal fields filled by the user (see "ask, never guess").
- **STAR competency stories** (`<name>-<company>-STAR`) — generate *on request* (or proactively for competency-based government roles); interview prep, not submitted. See `references/format-gov-competency.md`.

Put the company + role in the resume footer (center slot of `\makecvfooter`),
e.g. `<Name> ~·~ <Company> — <Role>`, so a printed copy is self-identifying.

## Master profile
`MASTER-PROFILE.md` is the durable asset — keep it current. Whenever the user
confirms a new fact (a tool, a corrected date, a new role, an hours figure),
write it into the profile immediately so future tailoring is accurate. Its
structure (see `assets/MASTER-PROFILE-TEMPLATE.md`): identity/contact, headline
tagline options, experience (official title + exact dates + hours/week + bullet
facts), education, confirmed skills, **honest framings** (the hedges that keep
claims defensible), publications, and projects.

## Guardrails (why honesty is the whole game)
A resume that wins the screen but collapses in the interview or a reference
check is worse than useless — for government and regulated employers it can
end a candidacy. So: claim only what the profile supports, hedge partial fits
honestly, ask before asserting anything new, and let codex catch what slips
through. Persuasion comes from framing real strengths well, not inventing them.

**Known misattribution traps (pre-empt these — codex catches them repeatedly):**
- **Wrong-employer attribution** — don't put one job's work under another (e.g.
  cloud/AI tooling from a *consulting* engagement claimed on a different
  employer whose stack differs). Keep each role's bullets to what that role did.
- **Consulting ≠ salaried employment** — work done through the candidate's own
  practice for a client must read as consulting, not a salaried role at that
  client, and must not invent a recent start date. Name the contracting party
  honestly.
- **Stale "prototyped/not in production"** — if the profile now says a tool is
  in production, don't carry forward old "prototyped" wording; it *under-sells*.
- **Match exact scope/role framing** — "multi-tenant" vs "multi-site," "lead
  architect" vs "contributor"; don't upgrade scope or role.

**Salary questions.** When a posting/field asks expected salary, give a
*market-informed range* tied to seniority + location + role, explain how to
phrase it (range not single number; anchor on value; surface level/band; ask
about total comp), and caveat that it's an estimate. Offer to tailor to the
user's real current comp / floor. It's advice, not a profile fact.

## Dependencies
- `xelatex` (TeX Live or MacTeX) and the Awesome-CV class + fonts.
- `codex` CLI (OpenAI Codex) for the second-model review. If unavailable, skip step 5 but tell the user the review was skipped.
