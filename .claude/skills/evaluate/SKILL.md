---
name: evaluate
description: Runs the LLM-as-judge over completed interview transcripts and produces a self-contained HTML report with BARS scoring, competency breakdown, decision, and recommendations. Only triggered manually when the user explicitly types /evaluate. Never auto-invoke. Never run during /interview or /prep — evaluation is always a deliberate, user-initiated action that closes a session.
allowed-tools: Read, Write, Edit, Bash, Glob
---

# Evaluate — LLM-as-judge against the plan

You are scoring the candidate's performance across whatever transcripts exist in the active session. Status updates to the user are in Spanish; the report itself is in English.

## 1. Resolve the active session

1. Read `.claude/specs/.current`. If missing or stale, fall back to most recent session by mtime.
2. If no session exists, tell the user (in Spanish) to run `/prep` or `/interview` first, and stop.

## 2. Load inputs

From the session dir:

- `plan.json` — required. Extract `role`, `competencies[]`, `stages[]`, `decision_thresholds`, `kill_criteria[]`.
- `transcript-screening.md`, `transcript-behavioral.md`, `transcript-technical.md` — load whichever exist. At least one is required.

If no transcripts exist, stop and tell the user to run `/interview <stage>` first.

## 3. Score each question (per-competency pass)

This is the core of the LLM-as-judge logic. Follow these rules from `steering/rubric-framework.md` §8:

**For each transcript loaded**, and **for each question in that stage**:

1. Locate the candidate's response (and any follow-up exchanges) in the transcript.
2. Read the question's `bars_anchors` — anchors at levels 1, 3, and 5.
3. **For each competency listed on the question**, run a separate scoring pass (do not score multiple competencies in one inference — aislamiento dimensional):
   - Compare the candidate's response against the anchor at level 1, 3, and 5.
   - Pick the closest anchor match. Interpolate to 2 or 4 if the response sits between anchors.
   - Cite at least one verbatim passage from the transcript that supports the score. This is mandatory.
   - If you cannot find a citation, score is 1 by default and note "no relevant evidence".
4. Record per-question, per-competency: `score` (1-5 integer), `citation` (verbatim quote), `rationale` (one sentence linking citation to anchor).

**Devil's advocate pass for borderline cases**: if any competency on a question scores ≤ 2.0, run a second pass explicitly searching for evidence that would raise the score. If no such evidence exists, confirm the score. This is a hard requirement before triggering the no-fail rule.

## 4. Aggregate

Per the rubric framework §5:

1. **Score per competency within a stage** = simple average across questions in that stage that evaluate this competency, weighted by the question's per-competency weight.
2. **Score per stage** = weighted average of competency scores using the stage's mapping in `methodology.md` §3.
3. **Score per competency across stages** = weighted average across stages where it appears, weighted by question count.
4. **Final score** = weighted average of competency scores using the `competencies[].weight` from `plan.json`.

Numbers carry to one decimal.

## 5. Apply decision rules

In order:

1. **Kill criteria check**: scan all transcripts for evidence of any `kill_criteria[]` violation. For honesty: require at least two cited contradicting passages. For ethics: require one clear quoted passage. If a kill criterion fires, decision is `No Hire` regardless of score; mark as `critical_issue` in the report.
2. **No-fail rule**: if any competency average is < 2.0, decision is `No Hire`.
3. **Threshold mapping** (only if neither above fired):
   - Final score ≥ `decision_thresholds.strong_hire` → `Strong Hire`
   - Final score ≥ `decision_thresholds.hire` → `Hire`
   - Final score ≥ `decision_thresholds.lean_no` → `Lean No`
   - Otherwise → `No Hire`

## 6. Generate recommendations

Produce 3-5 prioritized recommendations, ordered by impact:

- Lowest-scoring competencies first.
- Each recommendation: name the competency, summarize the gap (one sentence with citation), give a concrete next step ("practice STAR with metrics on every project story", "do 3 system design problems with explicit clarification phase").
- Where a high-scoring competency is also notable, end with one "double down" recommendation.

## 7. Render the HTML report

Read `templates/report.template.html`. Replace the placeholders:

| Placeholder                    | Source                                                            |
| ------------------------------ | ----------------------------------------------------------------- |
| `{{ROLE_TITLE}}`               | `plan.role.title`                                                 |
| `{{ROLE_LEVEL}}`               | `plan.role.level`                                                 |
| `{{COMPANY}}`                  | `plan.role.company`                                               |
| `{{SESSION_ID}}`               | session slug                                                      |
| `{{REPORT_DATE}}`              | ISO date                                                          |
| `{{PLAN_VERSION}}`             | `plan.version`                                                    |
| `{{DECISION_CLASS}}`           | one of `strong-hire`, `hire`, `lean-no`, `no-hire`                |
| `{{DECISION_LABEL}}`           | human-readable decision                                           |
| `{{FINAL_SCORE}}`              | aggregated score (one decimal)                                    |
| `{{DECISION_RATIONALE}}`       | 1-2 sentence summary of why this decision                         |
| `{{CRITICAL_ISSUE_BLOCK}}`     | rendered HTML block if kill criterion fired, else empty           |
| `{{COMPETENCY_ROWS}}`          | rendered list of competency rows (name + score + 1-line rationale + cite) |
| `{{STAGE_BLOCKS}}`             | rendered `<details>` block per stage with question-level scores and citations |
| `{{RECOMMENDATIONS}}`          | rendered `<li>` items                                             |
| `{{BIAS_COMPARISON_BLOCK}}`    | empty in default mode (Phase 1 does not implement bias mode)      |
| `{{REPORT_DATA_JSON}}`         | a JSON object with `domains.labels[]`, `domains.scores[]`, `competencies.labels[]`, `competencies.scores[]` |

Output goes to `<session>/report.html`. The file MUST be openable offline. The only network calls allowed are the Tailwind and Chart.js CDNs already declared in the template.

## 8. Append to `_log.md`

Append:

```markdown
## <ISO timestamp> — evaluate

- Tools used: Read, Write
- Inputs: plan.json, transcript-<stage>.md (×N)
- Files written: report.html, evaluation.json
- Final score: <score>
- Decision: <Strong Hire | Hire | Lean No | No Hire>
- Kill criterion fired: <none | type>
- No-fail rule triggered: <yes | no>
- Notes: <anything unusual>
```

Also write `<session>/evaluation.json` with the structured scoring data (per-question, per-competency, citations) — this is the machine-readable counterpart to `report.html`, useful for debugging and re-runs.

## 9. Hand off

Tell the user (in Spanish):

- Que el reporte se generó.
- Path absoluto a `report.html`.
- La decisión final y el score.
- Si se disparó una kill criterion, dilo arriba con la cita textual.
- Top 2 recomendaciones inline.
- Sugerencia: abrir el HTML en el navegador.

## Hard rules

- One LLM pass per (question, competency) pair. No bulk scoring.
- Every score must cite verbatim transcript text. No citation = score 1 with explicit "no evidence" note.
- Devil's advocate pass before any No Hire from no-fail rule.
- Temperature considerations: be deterministic. Same transcript should yield the same scores within ±0.5 across runs.
- Do not modify the transcripts. They are immutable artifacts.
- Do not auto-invoke `interview/` or `prep/`.
- Status updates in Spanish; report content in English.
