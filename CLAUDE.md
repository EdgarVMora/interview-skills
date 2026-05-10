# Interview Prep Lab

Sistema de simulación de entrevistas para roles tech en mercados US.
Dado un JD, extrae estructura, investiga la empresa, diseña un plan basado
en psicología organizacional, conduce la entrevista por etapas y genera un
reporte HTML con scoring BARS.

## Visión

Ver `.claude/steering/product.md`. La metodología (frameworks, etapas,
seniority, sesgos) vive en `.claude/steering/methodology.md`. La rúbrica
BARS, competencias y reglas de decisión viven en
`.claude/steering/rubric-framework.md`. Steering se carga siempre.

## Comandos disponibles

| Slash command          | Función                                                                  |
| ---------------------- | ------------------------------------------------------------------------ |
| `/prep <url\|texto>`   | Orquestador end-to-end: extract-jd → research → render-html → design-plan |
| `/extract-jd <input>`  | JD crudo → `jd.md` estructurado                                          |
| `/design-plan`         | `jd.md` + `dossier.md` → `plan.json` validado                            |
| `/interview <stage>`   | Manual. Conduce screening / behavioral / technical en EN                 |
| `/evaluate`            | Manual. LLM-as-judge sobre transcript → `report.html`                    |

`stage` ∈ {`screening`, `behavioral`, `technical`}.

## Cómo correr el sistema

1. Configurar Perplexity MCP una sola vez (key queda fuera del repo, en
   `~/.claude.json` — scope `user`):
   ```
   claude mcp add perplexity \
     --scope user \
     --env PERPLEXITY_API_KEY="<tu-key>" \
     -- npx -y @perplexity-ai/mcp-server
   ```
   Verificar con `claude mcp list`. Reversible con
   `claude mcp remove perplexity --scope user`.
2. `/prep <url>` — avisa que la investigación tarda 30-90s y cuesta ~$0.80
   USD por run. Si la sesión ya tiene `dossier.md`, pide confirmación
   antes de re-correr Perplexity (sobreescribir = $0.80 más).
3. `/interview screening` → `behavioral` → `technical` (en orden, manual).
4. `/evaluate` → abre `.claude/specs/<sesión>/report.html`.

**Salida de `/prep` por sesión** (`.claude/specs/<slug>/`):
- `jd.md` — JD estructurado (extract-jd)
- `dossier.md` — investigación de la empresa (company-researcher, Perplexity)
- `dossier.html` — vista de estudio self-contained (renderizado de dossier.md
  siguiendo `templates/dossier-html-structure.md`)
- `plan.json` — plan validado contra `templates/plan-schema.json`
- `_log.md` — registro de cada sub-paso
- `_costs.md` (raíz de `specs/`) — ledger acumulado de costos de Perplexity

## Convenciones

- **Idioma**: código + identificadores + entrevista simulada + reporte HTML
  en inglés. Status updates al usuario, comentarios técnicos y steering
  internos en español.
- **Sesiones**: cada run vive en `.claude/specs/<company-slug>-<YYYYMMDD>/`.
  El slug activo se guarda en `.claude/specs/.current` (escrito por `/prep`,
  leído por `/interview` y `/evaluate`). Fallback: mtime más reciente.
- **Logging**: cada skill que escriba en `specs/` también escribe `_log.md`
  con tools usadas, archivos generados y duración. Sin esto no debugueamos.
- **JSON estructurado**: cualquier JSON tiene schema en `templates/` y se
  valida antes de escribir.
- **HTML**: siempre self-contained. Tailwind y Chart.js por CDN.
- **Skills nuevas**: usar el skill-creator de Anthropic para generar la
  estructura SKILL.md inicial; ajustar manualmente después.
- **`data/`**: directorio opcional para inputs manuales (JDs pegados,
  dossiers escritos a mano). Gitignored. Se crea en runtime si se usa.
- **`sources/`**: investigación teórica de referencia (psicología
  organizacional). Citado por steering, no lectura obligatoria por skills.

## Anti-patrones

- NO usar `.claude/commands/` (deprecado). Skills only.
- NO convertir todo en agente. Solo `company-researcher` es subagent.
- NO autoinvocar `/interview` ni `/evaluate`. Son manuales por diseño;
  cada SKILL.md lo declara explícito en `description`.
- NO embeber API keys (Perplexity u otras) en archivos del repo.
- NO scrapear directamente LinkedIn/Glassdoor. Solo Perplexity MCP.
- NO mezclar `steering/` con `specs/`. Steering = siempre verdadero.
  Specs = artefactos de la sesión actual.
- NO inventar contenido de psicología organizacional. Las fuentes están
  en `sources/`. Si falta, preguntar.

## Fuente de verdad de la arquitectura

El init prompt original define la arquitectura canónica y el plan de fases
(0: skeleton, 1: MVP demoable, 2: auto-plan, 3: research agent). No
improvisar sobre eso sin discusión explícita.
