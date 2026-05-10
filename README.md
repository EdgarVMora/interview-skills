# Interview Prep Lab

Un simulador de entrevistas de trabajo que practica contigo en inglés, te evalúa con criterios de psicología organizacional, y te entrega un reporte detallado al final.

## ¿Qué es esto?

Imagínate tener acceso a un entrenador de entrevistas que:

- **Te entrevista en serio**, asumiendo el rol de un reclutador, un manager o un ingeniero senior, según la etapa.
- **Te evalúa con un sistema usado por empresas reales** (BARS — Behaviorally Anchored Rating Scales, el estándar de oro en selección de personal).
- **Te entrega un reporte** con tu puntaje por competencia, decisión final (Strong Hire / Hire / Lean No / No Hire) y recomendaciones priorizadas.

Está diseñado para personas que se preparan para entrevistas en empresas tech del mercado estadounidense (FAANG, startups, scale-ups). La entrevista corre en inglés porque ese es el idioma del mercado objetivo.

## ¿Cómo funciona?

El proceso tiene tres etapas, igual que una entrevista real:

```
1. Screening (25 min)        → con un reclutador
   "Tell me about yourself", motivación, expectativas salariales

2. Behavioral (60 min)       → con el manager
   Historias de tu experiencia: liderazgo, conflictos, ownership

3. Technical (90 min)        → con un staff engineer
   Diseño de sistemas, debugging, trade-offs técnicos
```

Después de cada etapa puedes parar, descansar, y volver más tarde. Cuando termines las que quieras, corres la evaluación y obtienes tu reporte.

## Cómo usarlo

### Requisitos

1. **Claude Code** instalado — la herramienta de línea de comandos de Anthropic. Si no la tienes, ve a [claude.com/claude-code](https://claude.com/claude-code) y sigue las instrucciones.
2. **Perplexity MCP** (opcional pero recomendado) — para que `/prep` pueda investigar la empresa automáticamente. Setup una sola vez:
   ```
   claude mcp add perplexity --scope user \
     --env PERPLEXITY_API_KEY="<tu-key>" \
     -- npx -y @perplexity-ai/mcp-server
   ```
   Generá tu key en [perplexity.ai/account/api/keys](https://www.perplexity.ai/account/api/keys). Sin esto, `/prep` corre en modo degradado (sin dossier) y solo extrae el JD + arma el plan.

### Pasos para correr una sesión de práctica

1. **Abre tu terminal** en la carpeta de este proyecto:
   ```
   cd interview-skills
   claude
   ```

2. **Genera tu plan personalizado** a partir del URL de un job description:
   ```
   /prep https://jobs.empresa.com/posting-id
   ```
   Esto corre cuatro pasos en cadena:
   - Extrae el JD a markdown estructurado.
   - Investiga la empresa con Perplexity (~30-90s, ~$0.80 USD por run).
   - Renderiza un `dossier.html` para estudiar antes de la entrevista — founders, timeline de funding, tech stack, prep cards.
   - Diseña el plan calibrado a tu seniority y validado contra schema.

   Si la empresa ya fue investigada en una sesión previa, `/prep` te pregunta si reusar el dossier o re-correr (re-correr cuesta otros ~$0.80).

3. **Abre el dossier de estudio** antes de empezar:
   ```
   open .claude/specs/<empresa-slug>-<fecha>/dossier.html
   ```

4. **Empieza la primera etapa** cuando estés listo:
   ```
   /interview screening
   ```

5. **Conversa con el entrevistador.** Te va a saludar, hacer preguntas, y pedirte aclaraciones cuando algo quede vago. Responde como si fuera real. No te va a decir cómo lo estás haciendo durante la entrevista — eso viene después.

6. **Cuando termines la etapa**, seguí con la siguiente:
   ```
   /interview behavioral
   ```
   y luego:
   ```
   /interview technical
   ```

7. **Cuando estés listo para tu evaluación**, escribí:
   ```
   /evaluate
   ```
   El sistema lee todo lo que dijiste, te puntúa contra criterios objetivos, y genera un reporte en HTML.

8. **Abrí el reporte** en tu navegador. La ruta te la imprime el sistema al final, algo como:
   ```
   .claude/specs/<empresa-slug>-<fecha>/report.html
   ```

### Modo demo (sin URL real)

Si querés probar el flujo de entrevista sin generar tu propio plan, podés saltarte el paso 2 y usar el plan de ejemplo (Senior Backend Engineer en una empresa ficticia llamada Acme Cloud) — `/interview screening` te va a preguntar si querés usarlo.

## Qué vas a recibir

El reporte HTML incluye:

- **Decisión final**: Strong Hire, Hire, Lean No, o No Hire — con explicación.
- **Tu puntaje por competencia** (10 dimensiones como Pensamiento Analítico, Iniciativa, Liderazgo, etc.) en una escala 1-5, con gráficas.
- **Desglose por etapa**: pregunta por pregunta, qué dijiste, cómo se evaluó, y la cita textual que justifica el puntaje.
- **3-5 recomendaciones priorizadas** por impacto: qué practicar primero, con ejemplos concretos.

Todo el reporte funciona offline (no manda tus respuestas a ningún servidor) y es un solo archivo HTML que puedes guardar o compartir.

## Estado del proyecto

Este es un sistema en construcción por fases. Hoy puedes:

| Funcionalidad | Estado |
| --- | --- |
| Practicar las 3 etapas con un plan de ejemplo (Senior Backend Engineer) | Disponible |
| Recibir reporte HTML con puntaje y recomendaciones | Disponible |
| Generar un plan personalizado a partir de un job description (`/prep`) | Disponible |
| Investigar la empresa automáticamente vía Perplexity | Disponible |
| Vista de estudio HTML del dossier (founders, timeline, prep cards) | Disponible |

Si querés practicar para un rol específico, lo más simple es correr `/prep <url>`. Si preferís plan manual, podés editar `templates/example-plan.json`.

## ¿En qué se basa la evaluación?

No es un puntaje arbitrario. El sistema usa frameworks validados en psicología industrial-organizacional con décadas de evidencia:

- **Schmidt & Hunter (1998)** — la entrevista estructurada predice el desempeño laboral con r=0.51 (vs. 0.38 de las no estructuradas).
- **Behavioral Event Interview (McClelland, 1973)** — el comportamiento pasado predice el comportamiento futuro mejor que cualquier test de aptitud.
- **BARS (Smith & Kendall, 1963)** — escalas ancladas en conducta observable, no en adjetivos vagos.
- **STAR / SOAR / CAR** — estructuras narrativas que la industria espera escuchar.

La investigación completa que sustenta el diseño está en la carpeta `sources/` del repositorio.

## Estructura del proyecto (para quien quiera ver bajo el capó)

```
.
├── CLAUDE.md                            Instrucciones del sistema para Claude Code
├── .claude/
│   ├── steering/                        Metodología y rúbrica permanente
│   │   ├── product.md                   Visión del producto
│   │   ├── methodology.md               Marco teórico y bases científicas
│   │   └── rubric-framework.md          Sistema de puntuación BARS
│   ├── skills/                          Comandos disponibles
│   │   ├── prep/                        /prep <url> — orquestador end-to-end
│   │   ├── extract-jd/                  /extract-jd — JD crudo → jd.md
│   │   ├── design-plan/                 /design-plan — jd.md + dossier.md → plan.json
│   │   ├── interview/                   /interview <etapa>
│   │   └── evaluate/                    /evaluate
│   ├── agents/
│   │   └── company-researcher.md        Subagent Perplexity (invocado por /prep)
│   └── specs/                           Sesiones individuales (ignorado por git)
├── templates/
│   ├── plan-schema.json                 Validador del formato del plan
│   ├── example-plan.json                Plan de ejemplo: Senior Backend Engineer
│   ├── dossier-html-structure.md        Spec de la vista HTML del dossier
│   └── report.template.html             Plantilla del reporte de evaluación
└── sources/                             Investigación científica de respaldo
```

## Privacidad y costos

- Tus respuestas y artefactos se guardan localmente en `.claude/specs/` (esa carpeta está gitignored).
- El paso de investigación de `/prep` hace llamadas a la API de Perplexity con la key que vos configuraste — solo se envía el nombre de la empresa y el rol, no tus respuestas. Costo aproximado: **~$0.80 USD por empresa investigada**. Si re-corrés `/prep` sobre la misma sesión, el sistema te pregunta antes de gastar de nuevo.
- El resto (extracción del JD, plan, entrevista, evaluación) usa Claude Code con tu cuenta de Anthropic — no agrega costos extras más allá de tu uso normal.
- Los reportes HTML (`dossier.html`, `report.html`) usan Tailwind y Chart.js desde CDN, pero tus datos no salen del archivo.

## Licencia

Por definir.
