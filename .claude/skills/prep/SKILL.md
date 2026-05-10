---
name: prep
description: End-to-end orchestrator for interview prep. Triggered when the user types /prep <url-or-text>. Runs four steps in sequence inside a single session under .claude/specs/<slug>/ — (1) extract-jd, (2) company-researcher subagent via Perplexity MCP, (3) render dossier.html study view, (4) design-plan with schema validation. Status updates in Spanish; artifacts in English. Never auto-invokes /interview or /evaluate — those remain manual by design.
allowed-tools: Read, Write, Edit, Bash, Glob, WebFetch, Task
---

# Prep — End-to-end orchestrator: JD → dossier → HTML view → validated plan

You orchestrate four building blocks. You do **not** re-implement their logic — you read each SKILL.md (and the relevant template specs) and follow them, then chain them. Status updates to the user in Spanish; every artifact you produce is in English.

## 1. Parse the user input

The user invoked `/prep <argument>`. The argument can be:

- **A URL** (starts with `http://` or `https://`).
- **Inline text** — the user pasted the JD directly.
- **Empty** — ask the user to paste the JD or provide a URL in the next message, then continue.

Tell the user (in Spanish) what you're about to do and warn that the research step takes 30-90 seconds and costs ~$0.80 in Perplexity API calls:

> 🚀 Arrancando `/prep`. Voy a (1) extraer el JD, (2) investigar la empresa con Perplexity (~30-90s, ~$0.80 USD por run), (3) renderizar `dossier.html` para estudiar, (4) diseñar el plan validado contra schema. Status en español, artefactos en inglés.

## 2. Pre-flight: verify Perplexity MCP is connected

Before spending tokens on extract-jd, confirm Perplexity MCP is available. Run:

```bash
claude mcp list 2>&1 | grep -i perplexity
```

Decision tree:

- **If the line shows `perplexity` and looks healthy** → continue to step 3.
- **If grep finds nothing or the server is in error state** → tell the user (in Spanish):

  > ⚠️ Perplexity MCP no está configurado. Para Fase 3 lo necesito. Pasos:
  >
  > 1. Generá tu key en <https://www.perplexity.ai/account/api/keys>.
  > 2. Corré:
  >    ```
  >    claude mcp add perplexity --scope user --env PERPLEXITY_API_KEY="<tu-key>" -- npx -y @perplexity-ai/mcp-server
  >    ```
  > 3. Verificá con `claude mcp list`.
  >
  > ¿Querés que continúe igual, en modo degradado (sin `dossier.md`)? El plan saldrá calibrado solo por el JD. Respondé "sí degradado" o configurá el MCP y volvé a correr `/prep`.

  Wait for the user. Only proceed in degraded mode on explicit confirmation. Otherwise stop here.

## 3. Step 1 — Extract the JD

Status to user: `📄 Paso 1/4 · Extrayendo JD…`.

Follow `.claude/skills/extract-jd/SKILL.md` exactly (sections 1-7 of that file). The end state of this step:

- `.claude/specs/<company-slug>-<YYYYMMDD>/` exists.
- `<session>/jd.md` is written with full frontmatter and the 9 canonical sections.
- `.claude/specs/.current` contains the new slug.
- `<session>/_log.md` has an `extract-jd` entry.

If extract-jd cannot identify the company or the page is JS-rendered chrome, surface the exact extract-jd guidance to the user and stop. Do not continue with `unknown-company` for the research step — the dossier would be useless.

## 4. Step 2 — Research the company

Status to user: `🔎 Paso 2/4 · Investigando empresa con Perplexity (~30-90s)…`.

In **degraded mode** (Perplexity not configured and the user said "sí degradado"): skip this step entirely. Append a one-line entry to `<session>/_log.md`:

```markdown
## <ISO timestamp> — prep (degraded)
- Skipped company-researcher: Perplexity MCP not available, user confirmed degraded mode.
- dossier.md: not generated
```

Then jump to step 5.

### 4.1 Cost-aware re-run protection (mandatory before invoking the subagent)

Each `company-researcher` invocation costs ~$0.80 on the Perplexity API (see `.claude/specs/_costs.md` for actuals). Don't re-run unless the user explicitly says so.

Before invoking the subagent, check the active session for evidence of a prior research run:

```bash
test -f "<session>/dossier.md" && echo "DOSSIER_EXISTS"
grep -l "company-researcher" "<session>/_log.md" 2>/dev/null && echo "LOG_HAS_RESEARCH"
```

If either signal fires, **stop and ask the user explicitly** (use the `AskUserQuestion` tool, in Spanish):

> Ya hay evidencia de un run de Perplexity previo en esta sesión (`dossier.md` existe / `_log.md` tiene una entrada de `company-researcher`). Re-correr cuesta ~$0.80 USD. ¿Cómo procedo?
>
> 1. **Reusar el dossier existente** (recomendado, $0)
> 2. **Re-correr la investigación** (sobreescribe `dossier.md` y agrega entrada al log; ~$0.80)
> 3. **Abortar `/prep`**

Decision tree on the answer:

- **Reusar** → skip the subagent call, log the decision, jump to step 5 (dossier.html render). The existing `dossier.md` is the source of truth.
- **Re-correr** → proceed with the subagent invocation below. Append a note to `_log.md` that the user explicitly authorized the re-run.
- **Abortar** → stop the orchestrator. Leave the session untouched.

If neither signal fires (clean session), proceed without asking — there's no prior work to preserve.

### 4.2 Subagent invocation

Invoke the `company-researcher` subagent via the Task tool. Use this exact shape:

```
Task(
  subagent_type: "company-researcher",
  description: "Research <company> for dossier",
  prompt: "Generate dossier.md for the active session (already written by /prep to .claude/specs/.current). Read <session>/jd.md, run targeted Perplexity queries per your SKILL instructions, and write <session>/dossier.md plus a _log.md entry. Return a 1-2 line Spanish summary."
)
```

Wait for the subagent to return. Expected outcomes:

- **Success** → `<session>/dossier.md` exists with the 7 canonical sections; `_log.md` has a `company-researcher` entry. Continue to step 5.
- **Subagent aborted** (Perplexity unreachable mid-flight) → log the failure to `_log.md` (yourself, since the subagent may not have done it) and ask the user whether to continue in degraded mode or stop.
- **Partial dossier** (some sections empty) → that's acceptable; the renderer in step 5 and `/design-plan` in step 6 both handle missing sections. Continue.

Do not retry the subagent automatically. One shot per `/prep` run keeps the cost envelope predictable.

## 5. Step 3 — Render `dossier.html` (study view)

Status to user: `🎨 Paso 3/4 · Renderizando dossier.html…`.

The HTML view is a self-contained, study-focused render of `dossier.md` with derived "Prep cards" (things to mention, questions to ask, avoid, logistics). Same look-and-feel across companies.

Follow `templates/dossier-html-structure.md` exactly. Key obligations:

- Read `<session>/dossier.md` and `<session>/jd.md` and synthesize the HTML in one pass.
- Output path: `<session>/dossier.html`. Self-contained, Tailwind + Chart.js via CDN, no other external assets.
- All 11 sections in `dossier-html-structure.md` §4 must be present in order. If a dossier section is missing, render the anchor with a "Not found in research" placeholder card — never silently drop sections.
- The "Prep cards" section (§5 in the structure spec) is **derived**, not transcribed. Synthesize from the dossier + JD; do not invent facts.
- Use the reference implementation at `.claude/specs/simcare-20260509-v2/dossier.html` as the visual baseline.

After writing, verify the file is non-empty and references both CDNs:

```bash
test -s "<session>/dossier.html" && \
  grep -q 'cdn.tailwindcss.com' "<session>/dossier.html" && \
  grep -q 'chart.js' "<session>/dossier.html" && \
  echo "HTML_OK"
```

Append a `prep (render-html)` entry to `_log.md`:

```markdown
## <ISO timestamp> — prep (render-html)
- Tools used: Read, Write
- Files written: dossier.html
- Source: dossier.md (<size>) + jd.md (<size>)
- Sections rendered: 11
- Notes: <anything unusual — sections rendered as placeholders, degraded-mode banner inserted, etc.>
```

Skip this step entirely in **degraded mode** (no `dossier.md`). Note the skip in `_log.md`:

```markdown
## <ISO timestamp> — prep (render-html, skipped)
- Reason: degraded mode, no dossier.md to render.
```

## 6. Step 4 — Design the plan

Status to user: `🧭 Paso 4/4 · Diseñando plan calibrado…`.

Follow `.claude/skills/design-plan/SKILL.md` exactly (sections 1-12 of that file). Key points to remember in this orchestration context:

- The session is **already** active — do not re-resolve from scratch, use the slug from `.claude/specs/.current` that you just wrote.
- `dossier.md` will exist (normal mode or reuse) or be absent (degraded mode). The skill already handles both branches and notes the limitation in `_log.md`.
- Validation against `templates/plan-schema.json` is **mandatory**. If validation fails, fix and re-validate before writing. Never write a broken plan.

End state of this step:

- `<session>/plan.json` written, schema-valid, 2-space indent.
- `<session>/_log.md` has a `design-plan` entry with seniority proportions, top tasks, and validation result.

## 7. Final summary to the user

Tell the user (in Spanish) in 6-8 lines max:

```
✅ /prep listo · sesión <slug>

- 📄 jd.md            <Role> (<Level>) @ <Company>
- 📚 dossier.md       <N sources cited> · <empty sections, if any>
                      (o: "no generado, modo degradado" si aplica)
                      (o: "reusado del run anterior" si aplica)
- 🎨 dossier.html     vista de estudio renderizada
                      (o: "no generado, modo degradado" si aplica)
- 🧭 plan.json        validado · <competencies> competencias
                      <behavioral_pct>/<situational_pct> behavioral/situational

Siguiente paso: /interview screening
```

Path absoluto a la sesión: `.claude/specs/<slug>/`. Mencioná el HTML como `open .claude/specs/<slug>/dossier.html` para abrirlo.

Mention briefly any caveat surfaced by a sub-step (e.g., "interview process unknown — el plan usa estructura canónica de 3 etapas").

## Hard rules

- `/prep` is the **only** skill in this project that auto-orchestrates other steps. `/interview` and `/evaluate` are manual by design — do not invoke them from here under any circumstance.
- Pre-flight the Perplexity MCP **before** running extract-jd. If you skip the pre-flight and the research step fails after extract-jd succeeds, you've burned half the user's time for nothing.
- **Cost-aware re-run protection is mandatory.** Never invoke `company-researcher` if `<session>/dossier.md` already exists or `_log.md` has a prior `company-researcher` entry, without explicit user confirmation. Each unnecessary re-run costs ~$0.80.
- The `dossier.html` render is deterministic and free (no API cost) — re-rendering is fine and encouraged after dossier edits, but doesn't happen automatically outside of `/prep`.
- Never embed any API key or env var value in user-facing output. The setup command above intentionally uses `<tu-key>` placeholder.
- Status updates to the user in Spanish; every artifact (`jd.md`, `dossier.md`, `dossier.html`, `plan.json`, `_log.md`) in English.
- Never write outside `.claude/specs/<slug>/` plus `.claude/specs/.current`.
- One session per `/prep` call. If a session for `<company-slug>-<YYYYMMDD>` already has a `jd.md`, defer to extract-jd's own clobber-protection prompt.
- The `_log.md` is the only debugging tool for this pipeline. Each step **must** append; if you catch a failure mid-step, write a `prep` entry to `_log.md` before surfacing the error.
