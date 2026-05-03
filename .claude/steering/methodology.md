# Methodology — Interview Prep Lab

Esta es la base teórica del sistema. Toda decisión de diseño de plan,
preguntas, calibración por seniority y mitigación de sesgos se justifica
contra este documento. Las fuentes detalladas viven en `sources/`.

## 1. Marco teórico canónico

El sistema opera sobre cuatro pilares de psicología industrial-organizacional:

- **Validez predictiva (Schmidt & Hunter, 1998 + Sackett et al., 2022)**.
  La tabla de validez ancla nuestras decisiones: priorizamos los predictores
  fuertes (work samples r≈0.54, GMA r≈0.51, structured interviews r≈0.51) y
  evitamos los débiles (años de experiencia r≈0.18, educación r≈0.10,
  graphology r≈0). Combinación GMA + structured interview alcanza r≈0.63.
  Fuente: `sources/Validez y predicción de desempeño...md`,
  `sources/Frameworks.md`.

- **Behavioral Event Interview / Critical Incident Technique
  (McClelland 1973, Flanagan 1954)**. Toda pregunta conductual se construye
  desde un incidente crítico inferido del JD + dossier. Patrón forense:
  qué dijo, pensó, sintió, hizo. Fuente: `sources/Frameworks.md`,
  `sources/Job analysis...md`.

- **Dicotomía Janz (PBDI) vs Latham (Situational)**. PBDI es retrospectivo
  ("cuéntame de una vez en que..."), apoyado en consistencia conductual.
  Situational es prospectivo ("¿qué harías si...?"), apoyado en ATIC
  (Ability To Identify Criteria) y Goal-Setting Theory. La proporción se
  calibra por seniority (sección 3).

- **Big Five / OCEAN (Barrick & Mount 1991, Hurtz & Donovan 2000)**.
  Conscientiousness es el predictor universal; Emotional Stability es
  necesario pero no suficiente. Lo usamos como sustrato para inferir
  competencias, no como test directo (no aplicamos cuestionarios OCEAN al
  candidato).

## 2. Etapas del funnel y foco evaluativo

Cuatro etapas reconocidas en la industria (`sources/Etapas reales...md`).
MVP cubre las primeras tres:

| Etapa             | Duración típica | Foco                                | Fit          |
| ----------------- | --------------- | ----------------------------------- | ------------ |
| Recruiter screen  | 15-30 min       | Dealbreakers, motivación, salario   | P-J base     |
| Hiring manager    | 45-60 min       | Competencias core del rol + equipo  | P-J          |
| Technical / case  | 60-90 min       | Work sample (coding / system design)| P-J profundo |
| Cultural (futuro) | 30-60 min       | Valores, "culture add", no afinidad | P-O          |

P-J fit (Person–Job) domina las primeras tres. P-O fit (Person–Organization)
queda fuera del MVP; cuando entre en Fase 2+ se diseñará explícitamente
como "culture add", evitando que se convierta en filtro de afinidad
(`sources/Job analysis...md`).

## 3. Niveles de seniority y proporción de preguntas

Taxonomía: **Junior / Mid / Senior / Staff / Principal**. Pulakos & Schmitt
(1995) demostraron que Situational Interviews degradan a r≈-0.02 en roles
complejos; PBDI mantiene r≈0.32. Por tanto:

| Seniority           | % Behavioral (PBDI/BDI) | % Situational (SI) | Notas                                    |
| ------------------- | ----------------------- | ------------------ | ---------------------------------------- |
| Junior              | 30%                     | 70%                | SI con dilemas reales; falta de bagaje   |
| Mid                 | 50%                     | 50%                | Interpolación                            |
| Senior              | 70%                     | 30%                | Empieza a dominar el patrón conductual   |
| Staff / Principal   | 90-100%                 | 0-10%              | SI pierde validez; exigir historia real  |

Las SI siempre llevan un dilema incrustado (Kleinmann, Griffin); sin
dilema, se trivializan y caen a r≈0.

## 4. Job analysis: del JD al plan

Pipeline conceptual (`sources/Job analysis...md`):

1. **Task Inventory** — extraer tareas del JD con importancia/frecuencia.
2. **Critical Incident Technique** — sobre las top, inferir incidentes
   diferenciadores (Outstanding vs Typical).
3. **Functional Job Analysis** — etiquetar cada tarea/incidente como
   People / Data / Things para elegir formato (entrevista, case, role-play).
4. **KSAO mapping** — Knowledge / Skills / Abilities / Other characteristics
   por tarea, base para selección de competencias a evaluar.

`design-plan` automatiza una versión preliminar de este pipeline; el
output (`plan.json`) debe validar contra `templates/plan-schema.json`.

## 5. Frameworks por tipo de entrevista (Fase 1: tech)

Reconocidos por la industria (`sources/Frameworks específicos...md`):

- **Coding (CTCI-style)**: clarify → examples → high-level idea → code →
  test → complexity. El simulador penaliza saltar a code golf sin clarificar.
- **System design (Alex Xu)**: scope/requirements → high-level → deep dive
  en componentes → non-functionals (scale, latency, reliability). Fuerza
  secuencia: no permitir DB-deep-dive antes de clarificar tráfico.
- **Behavioral (STAR/SOAR/CAR)**: STAR es default; CAR para senior+
  (compresión); SOAR para preguntas de obstáculos/resiliencia.

Verticales fuera de tech (product CIRCLES/AARM, consulting MECE, sales
MEDDIC/Challenger) están documentados en `sources/` como extension points
para Fase 2+.

## 6. Mitigación de sesgos del entrevistador

El simulador puede operar en dos modos (`sources/Sesgos del entrevistador...md`):

- **Modo objetivo** (default): scoring estricto contra rúbrica BARS,
  preguntas idénticas, double-blind coding, calibration por competencia
  aislada.
- **Modo realista** (opt-in): el entrevistador virtual lleva parámetros de
  primacy, halo/horns, affinity, confirmation, recency, contrast. Score
  "objetivo" y "percibido" se calculan en paralelo y se muestran ambos al
  usuario para entrenar resiliencia.

Mitigaciones obligatorias en cualquier modo:

- Mismas preguntas a todos los candidatos del mismo plan.
- Evaluación por pregunta antes del consenso global (no holistic-first).
- LLM-as-judge anclado a evidencia textual del transcript.
- Aislamiento dimensional: un pase del juez por competencia, no todo junto.

## 7. Modelo psicológico del candidato

El sistema modela explícitamente (`sources/Psicología del candidato...md`):

- **Interview anxiety** (MASI, McCarthy & Goffin): 5 dimensiones
  (comunicación, apariencia, social, desempeño, conductual). Medible con
  ítems breves antes/después de sesión.
- **Self-efficacy** (Bandura): cuatro fuentes — mastery, vicarious,
  verbal persuasion, physiological. La progresión de dificultad de
  escenarios alimenta mastery; el feedback alimenta persuasion.
- **Impression management**: distinguir self-promotion honesta (premiada)
  de exagerada (penalizada); ingratiation sutil vs explícita.
- **Faking detection**: preguntas de seguimiento automáticas que exploran
  procesos, trade-offs y contexto; respuestas "too good to be true" sin
  cifras ni dificultades disparan flags.
- **Stereotype threat** (Steele & Aronson): el sistema evita activar
  identidad innecesariamente y ofrece psicoeducación si el usuario reporta
  bajo desempeño en escenarios de alta amenaza.

## 8. Verticales soportados

- **Fase 1 (MVP)**: Software Engineer / Backend, en inglés, mercado US.
- **Fase 2+ (extension points)**: Product Management, Consulting, Sales,
  Data Science. Cada vertical agrega su template de plan y conjunto de
  frameworks específicos sin reescribir la arquitectura.

## 9. Referencias internas

Toda afirmación teórica de este documento debe poder rastrearse a uno de
los archivos en `sources/`. Si una decisión de diseño no encuentra anclaje
ahí, se pide al usuario antes de incorporarla.
