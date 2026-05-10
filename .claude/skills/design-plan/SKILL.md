---
name: design-plan
description: Generates a complete plan.json from jd.md and optionally dossier.md in the active session, calibrated to seniority per methodology.md, scored per rubric-framework.md, and validated against templates/plan-schema.json. Triggered when the user types /design-plan. Used both directly and as the third step inside /prep (after extract-jd and the company researcher).
allowed-tools: Read, Write, Edit, Bash, Glob
---

# Design Plan — `jd.md` (+ `dossier.md`) → validated `plan.json`

You are converting a structured JD plus optional company dossier into a complete interview plan that conforms to `templates/plan-schema.json` and reflects the methodology in `.claude/steering/methodology.md` and the rubric in `.claude/steering/rubric-framework.md`.

Status updates to the user in Spanish. The `plan.json` content is in English (interview content runs in English).

## 1. Resolve active session

1. Read `.claude/specs/.current`. If missing or stale, fall back to most recent session by mtime.
2. If no session exists, tell the user to run `/extract-jd` first and stop.

## 2. Load inputs

From the session directory:

- **`jd.md`** — required. Read frontmatter (title, company, company_slug, level) and body sections. If missing, stop and ask the user to run `/extract-jd` first.
- **`dossier.md`** — optional. If present, read it; treat its content as additional context about the company (culture, products, scale, declared values, known interview format). If absent, proceed with JD-only context and **note the limitation** in `_log.md` and in the user-facing message.

From the project root, read for canonical methodology:

- `.claude/steering/methodology.md` — etapa-by-etapa focus, seniority calibration table (§3), bias mitigation rules.
- `.claude/steering/rubric-framework.md` — 12 competencies, 4 Hogan domains, etapa→competencia mapping (§4), aggregation, decision thresholds, kill criteria.
- `templates/plan-schema.json` — the schema you must validate against.
- `templates/example-plan.json` — reference for shape, anchor density, and tone (this is the gold standard for a Senior Backend Engineer plan).

## 3. Job analysis pipeline

Apply the canonical pipeline from `methodology.md` §4:

1. **Task Inventory**: extract tasks from `jd.md` Responsibilities + Requirements. Estimate importance (1-3) and frequency (1-3) per task. Top 5-7 tasks become the basis for the plan.
2. **Critical Incident Technique**: for each top task, infer 1-2 plausible critical incidents (a high-stakes situation where a strong candidate would clearly outperform a typical one). These incidents seed the behavioral and technical questions.
3. **Functional Job Analysis (light)**: classify each top task on People / Data / Things axes. Use this to choose question format:
   - High People → behavioral question or role-play.
   - High Data → analytical / case / system design.
   - High Things → coding or hands-on technical.
4. **KSAO mapping**: for each top task, identify the Knowledge, Skill, Ability, or Other characteristic it demands. Map each KSAO to one of the 12 competencies in the rubric.

You don't need to write this pipeline output to a separate file — it's the reasoning chain that drives the plan. But include a one-paragraph summary of the inferred top tasks and competencies in `_log.md`.

## 4. Calibrate by seniority

Per `methodology.md` §3, the proportion of behavioral (PBDI) vs situational (SI) questions depends on `level`:

| Level     | Behavioral % | Situational % |
| --------- | ------------ | ------------- |
| Junior    | 30           | 70            |
| Mid       | 50           | 50            |
| Senior    | 70           | 30            |
| Staff     | 90           | 10            |
| Principal | 100          | 0             |

Apply this to **the behavioral stage** specifically. The screening and technical stages have their own canonical structure (see §5).

Set `role.seniority_proportions` in the plan accordingly.

## 5. Design the three stages

Generate exactly **three stages** with **five questions each**, matching the example-plan structure. Stage personas should reflect the company (use the dossier if available; otherwise generic but plausible).

### Screening (stage `screening`, 25 minutes)

- Persona: recruiter (warm but efficient, time-boxed, not technical).
- Focus competencies: `self_confidence`, `achievement_orientation`, `information_seeking`.
- Question template (adapt names and details to the role):
  1. "Tell me about yourself" — opens the conversation; tests narrative clarity.
  2. Recent project the candidate is proud of — tests Achievement Orientation with metrics.
  3. Why leaving current role — tests professionalism under a delicate question.
  4. Salary expectations — tests Self-Confidence and information seeking (anchor / ask total comp).
  5. Candidate's questions for the interviewer — tests research and engagement.

### Behavioral (stage `behavioral`, 60 minutes)

- Persona: hiring manager (calm, structured, STAR probing).
- Focus competencies: `initiative`, `flexibility`, `impact_and_influence`, `team_leadership`, `interpersonal_understanding`.
- Apply the seniority proportion from §4: at Senior, ~70% PBDI ("tell me about a time...") + ~30% Situational ("what would you do if..."). At Junior, the inverse. Always include at least one situational dilemma at Junior/Mid; for Staff/Principal, prefer pure PBDI.
- Tailor the situations to the role's domain. For tech roles, common axes:
  - Disagreement with a tech lead on architecture.
  - Mid-flight scope change.
  - Initiative outside assigned scope.
  - Leading a technical decision against skepticism.
  - Mentoring or unblocking a teammate.

### Technical (stage `technical`, 90 minutes)

- Persona: senior IC or staff engineer in the role's discipline.
- Focus competencies: `analytical_thinking`, `conceptual_thinking`, `information_seeking`. Optional add-ons: `team_leadership` (for staff+ roles where leading technical decisions appears), `flexibility` (when the question explicitly tests adaptability under new constraints).
- Five questions, balancing format:
  - 2 system design (calibrated in scope to level — at Junior, "design a URL shortener"; at Staff, "evolve a multi-region service for compliance").
  - 1 debugging / production incident.
  - 1 architecture evolution / refactor / migration.
  - 1 trade-off / tooling comparison rooted in a real constraint.
- Pull the tech stack from `jd.md` Tech stack section. If the JD says Python + Postgres + AWS, design questions in that ecosystem, not generic ones.

## 6. Write BARS anchors per question

For every question in every stage, write anchors at levels **1, 3, and 5** (levels 2 and 4 interpolate). Each anchor is **observable behavior**, not adjectives.

Calibration:

- **Level 1 (Does Not Meet)**: concrete failure mode. What does a weak candidate actually say or do? Be specific (e.g., "Jumps to 'use Redis' without clarifying scale or fairness").
- **Level 3 (Meets)**: solid, expected performance for this seniority. Names a reasonable approach with at least one trade-off acknowledged.
- **Level 5 (Outstanding)**: clearly above bar. Drives requirements gathering, names non-obvious failure modes, articulates what they'd defer to v2.

Avoid level 5 anchors that require omniscience. They should be ambitious but achievable for a strong real candidate.

## 7. Set thresholds and kill criteria

Use defaults from `rubric-framework.md` §6 unless the dossier suggests otherwise:

```json
"decision_thresholds": { "strong_hire": 4.5, "hire": 3.5, "lean_no": 2.5 }
"kill_criteria": [
  { "type": "honesty", "description": "..." },
  { "type": "ethics",  "description": "..." }
]
```

Reuse the kill_criteria descriptions verbatim from `templates/example-plan.json` unless the dossier names a domain-specific concern (e.g., for finance roles, add a regulatory honesty criterion).

## 8. Set competency weights

The `competencies[]` array lists every competency the plan evaluates with a `weight` (0-1). Weights MUST sum to 1.00.

Default weighting: cognitive competencies (Analytical, Conceptual) carry more weight at higher seniority for tech roles. Suggested distribution for a Senior Backend Engineer (matches example-plan):

```
analytical_thinking         0.18
conceptual_thinking         0.16
information_seeking         0.10
initiative                  0.10
self_confidence             0.08
achievement_orientation     0.08
flexibility                 0.08
impact_and_influence        0.08
team_leadership             0.08
interpersonal_understanding 0.06
                            ────
                            1.00
```

Adjust by ±0.02 increments based on JD signals. For Staff+ roles, shift weight toward `team_leadership` and `conceptual_thinking`.

## 9. Validate the plan

Before writing, validate the in-memory plan against `templates/plan-schema.json`. Use one of:

**Preferred** (formal validation via Python + jsonschema):

```bash
python3 -c "
import json
from jsonschema import validate
with open('templates/plan-schema.json') as f: schema = json.load(f)
with open('/tmp/plan-draft.json') as f: plan = json.load(f)
validate(plan, schema)
print('OK')
"
```

If `jsonschema` isn't available, create a venv: `python3 -m venv /tmp/.venv-validate && /tmp/.venv-validate/bin/pip install --quiet jsonschema && /tmp/.venv-validate/bin/python3 -c '...'`.

**Fallback** (manual structural check, only if Python isn't available): re-read the generated JSON and verify by hand that:

- All required top-level keys are present (`version`, `role`, `competencies`, `stages`, `decision_thresholds`, `kill_criteria`).
- Every question has `id`, `text`, `type`, `competencies[]`, and `bars_anchors` with keys `"1"`, `"3"`, `"5"`.
- Competency weights sum to exactly 1.00 (allow ±0.001 for floating point).
- Every competency `id` referenced in questions also appears in the top-level `competencies[]`.
- Every per-question competency `weight` is between 0 and 1.

If validation fails, fix the issues and re-validate. Do not write a broken plan.

## 10. Write `<session>/plan.json`

Once validation passes, write the JSON to `<session>/plan.json` with 2-space indent.

## 11. Append to `_log.md`

```markdown
## <ISO timestamp> — design-plan

- Tools used: Read, Write, Bash
- Inputs: jd.md (always), dossier.md (yes/no)
- Files written: plan.json
- Plan version: 1.0.0
- Role: <title> (<level>) at <company>
- Seniority proportions: <behavioral_pct>/<situational_pct>
- Top tasks identified: <comma-separated brief list>
- Competencies selected: <count> (weights sum to 1.00)
- Validation: jsonschema passed (or manual check, with caveat)
- Notes: <anything unusual — JD vagueness compensated for, dossier missing, dossier-driven custom kill criterion, etc.>
```

## 12. Hand off

Tell the user (in Spanish):

- Que el plan se generó y validó.
- Path de `plan.json`.
- Resumen de 1-2 líneas: rol, nivel, empresa, # competencias, proporción behavioral/situational.
- Si el dossier estaba ausente, recuerda que un dossier mejoraría el plan (sugerir poblar `<session>/dossier.md` y re-correr).
- Siguiente paso: `/interview screening`.

## Reference: dossier.md structure (when manually authored)

If the user populates `<session>/dossier.md` before running this skill, the expected shape is:

```markdown
# Dossier — <Company>

## Company snapshot
- Industry, size, stage, recent funding, geography.

## Products / business model
- Main products, revenue model, who pays.

## Engineering culture
- Declared values, blog posts, conference talks, public artifacts.

## Tech stack (verified, beyond what's in the JD)
- Languages, infra, databases, deployment, observability.

## Recent challenges, news, or strategic moves
- Layoffs, new product launches, leadership changes, etc.

## Known interview process
- Stages, format, common questions if known.

## Sources
- URLs and dates.
```

This will be auto-generated by the `company-researcher` agent in Phase 3. For Phase 2, it's manual or absent.

## Hard rules

- Plan must validate against the schema before being written. No exceptions.
- All competency `id`s referenced in questions must also exist in `competencies[]`.
- Competency weights must sum to exactly 1.00 (allow ±0.001 floating point).
- Five questions per stage, three stages — no more, no less, in Phase 2.
- BARS anchors at 1, 3, 5 are mandatory per question; observable behavior, not adjectives.
- Status updates in Spanish; plan content in English.
- Do not auto-invoke `interview/`. The user runs it explicitly.
