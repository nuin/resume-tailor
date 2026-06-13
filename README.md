# resume-tailor

A Claude Code skill that turns a **job posting** into a tailored, **ATS-ready
resume + cover letter** (PDF and plain-text), grounded in a master fact sheet
and verified by a read-only second-model (`codex`) review.

The guiding principle is **persuasive without ever fabricating**: every claim
traces back to confirmed ground truth, partial fits are hedged honestly
("familiar / ramping", "ITIL-equivalent"), and the skill *stops and asks*
before claiming anything it can't find in your profile.

## What it does

Give Claude a job posting and it will:
1. Read your `MASTER-PROFILE.md` (the single source of truth).
2. Parse the posting (skills, responsibilities, competencies, format hints).
3. **Gap-check** every requirement against your profile and ask before claiming anything new — no fabrication.
4. Build a tailored resume + cover letter (US Letter, real names, month/year dates), in either a **standard tech ATS** format or a **government / competency-based** format.
5. Run a `codex` review to catch overclaims/contradictions and iterate.
6. Compile to PDF + a plain-text ATS companion, named after the company.

## Requirements

- **Claude Code**
- **XeLaTeX** (TeX Live / MacTeX) and the [Awesome-CV](https://github.com/posquit0/Awesome-CV) class + fonts in your resume project (the skill can fetch `awesome-cv.cls` automatically)
- **[OpenAI Codex CLI](https://github.com/openai/codex)** (`codex`) for the verification step — optional; the skill skips review and tells you if it's missing

## Install

### As a plugin marketplace (recommended)

```
/plugin marketplace add nuin/resume-tailor
/plugin install resume-tailor@resume-tailor
```

(Replace `nuin/resume-tailor` with wherever you host this repo.)

### Manually

Copy the skill folder into your skills directory:

```bash
cp -r skills/resume-tailor ~/.claude/skills/resume-tailor
```

## First-time setup

1. In your resume project (a folder with `awesome-cv.cls` + `fonts/`), create your fact sheet:
   ```bash
   cp ~/.claude/skills/resume-tailor/assets/MASTER-PROFILE-TEMPLATE.md MASTER-PROFILE.md
   ```
2. Open `MASTER-PROFILE.md` and replace the example character ("Alex Rivera") with your real roles, dates, hours, skills, and — most importantly — your **honest framings** (how to phrase partial fits truthfully).

## Usage

Just paste a job posting into Claude Code (mention if it's a government role).
The skill triggers automatically, builds your documents, runs the review, and
saves `you-resume-<company>.pdf/.txt` and `you-coverletter-<company>.pdf`.

When you confirm a new fact during a run, it gets written back into your
`MASTER-PROFILE.md`, so each application is faster and more accurate than the
last.

## Layout

```
resume-tailor/
├── .claude-plugin/marketplace.json
└── skills/resume-tailor/
    ├── SKILL.md
    ├── references/
    │   ├── format-standard-ats.md       # private-sector tech ATS conventions
    │   └── format-gov-competency.md     # government / competency-based format
    └── assets/
        └── MASTER-PROFILE-TEMPLATE.md   # fill-in fact sheet (fake example)
```

## Privacy

Your `MASTER-PROFILE.md` is **your** data and lives in your resume project — it
is never bundled with the skill. Only the blank template (with a fake example
character) ships here.

## License

MIT
