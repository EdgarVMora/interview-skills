---
name: extract-jd
description: Extracts a structured job description from a URL or pasted text. Triggered when the user types /extract-jd <url-or-text>. Creates or updates a session under .claude/specs/<company-slug>-<YYYYMMDD>/, writes jd.md with YAML frontmatter and sectioned markdown body, and updates .claude/specs/.current. Used both directly by the user and as the first step inside /prep.
allowed-tools: Read, Write, Edit, Bash, Glob, WebFetch
---

# Extract JD — JD URL or text → structured `jd.md`

You are turning a raw job description into a clean, structured artifact that the rest of the pipeline can consume. Status updates to the user are in Spanish; the `jd.md` file content is in English (matching the source JD and the project convention).

## 1. Parse the user input

The user invoked `/extract-jd <argument>`. The argument can be:

- **A URL** (starts with `http://` or `https://`).
- **Inline text** (the user pasted the JD directly into the prompt).
- **Empty** — in which case ask the user to paste the JD text in the next message.

Detect which case applies. If unsure (e.g., the argument is short and ambiguous), ask the user to clarify before proceeding.

## 2. Get the raw JD content

- **URL case**: use `WebFetch` to retrieve the page. If the page is a JS-heavy ATS (Greenhouse, Lever, Workday) and the fetch returns mostly chrome and no body, tell the user — they should paste the JD text manually.
- **Text case**: the argument (or the next user message) IS the raw JD. Skip fetch.

Reject silently if the input is clearly not a JD (e.g., a homepage, a 404, a search results page).

## 3. Identify role and company

From the raw content, extract:

- **Role title** (e.g., "Senior Backend Engineer").
- **Company name** (e.g., "Acme Cloud").
- **Company slug**: lowercase, kebab-case, ASCII only (e.g., `acme-cloud`). Strip suffixes like "Inc", "Ltd", ", Inc.".
- **Seniority level**: pick exactly one of `Junior`, `Mid`, `Senior`, `Staff`, `Principal`. Use signals: title prefix (e.g., "Sr.", "Staff", "Principal"), years of experience required, scope language ("lead", "architect", "drives org-wide"). If genuinely ambiguous, default to `Senior` and flag in `Notes`.

If you cannot identify the company, use slug `unknown-company` and tell the user so they can rename the session later.

## 4. Resolve or create the session

1. Compute target session slug: `<company-slug>-<YYYYMMDD>` using today's date in UTC.
2. Check `.claude/specs/<slug>/`:
   - If it exists and contains a `jd.md`, ask the user (in Spanish) whether to overwrite, append a suffix (`-v2`), or abort.
   - Otherwise, create the directory.
3. Write the slug to `.claude/specs/.current` (single line, no trailing newline).

## 5. Write `<session>/jd.md`

Use this exact structure. All fields in YAML frontmatter are required. Use `"not specified"` (string) for missing fields rather than omitting keys.

```markdown
---
title: "Senior Backend Engineer"
company: "Acme Cloud"
company_slug: "acme-cloud"
level: "Senior"
location: "Remote (US)"
employment_type: "Full-time"
source_url: "https://..."
extracted_at: "2026-05-03T14:22:00Z"
---

# <Role Title> @ <Company>

## Summary
<2-4 sentences: what the role does and why it exists. No marketing fluff.>

## Responsibilities
- <bullet>
- <bullet>

## Requirements
- <bullet — must-haves>

## Nice to have
- <bullet — preferred but not required>

## Tech stack
- <languages, frameworks, databases, infra explicitly mentioned>

## Compensation
- <range if disclosed; otherwise "not specified">
- <equity, bonus, benefits if mentioned>

## Interview process (if disclosed)
- <stages and format if the JD mentions them; otherwise "not specified">

## Notes
- <anything notable: red flags, ambiguity in seniority, unusual phrasing, missing info, etc.>
- <if the JD is vague on a critical dimension, name it here so design-plan can compensate>
```

Editorial rules:

- Bullets are short and concrete. No marketing fluff ("rockstar", "ninja", "fast-paced").
- Don't paraphrase requirements into something stronger or weaker than the source.
- If a section has nothing to say, write `- not specified` rather than deleting the section.
- Quote dollar ranges and percentages verbatim if the JD provides them.

## 6. Append to `<session>/_log.md`

```markdown
## <ISO timestamp> — extract-jd

- Tools used: WebFetch, Write
- Source: <URL or "pasted text">
- Files written: jd.md
- Session: <slug>
- Role: <title> (<level>) at <company>
- Notes: <anything unusual — extraction gaps, ambiguous seniority, JS-rendered page issues>
```

## 7. Hand off

Tell the user (in Spanish):

- Confirma la sesión creada y el path de `jd.md`.
- Resumen de 1 línea: rol + nivel + empresa.
- Si hubo ambigüedades (e.g., seniority dudoso, sin compensación), nómbralas brevemente.
- Sugerencia del siguiente paso: `/design-plan` (opcionalmente después de poblar `dossier.md` a mano).

## Hard rules

- Never invent content not present in the source. If a field is missing, mark it as such.
- Never embed scraped LinkedIn or Glassdoor content (project policy).
- Status updates in Spanish; file content in English.
- Do not auto-invoke `design-plan` — the user runs it explicitly. (Exception: when called from inside `/prep`, the orchestrator handles sequencing.)
- One session per JD. If the user re-runs on the same URL+date, ask before clobbering.
