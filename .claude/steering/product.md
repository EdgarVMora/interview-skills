# Product Vision — Interview Prep Lab

## Qué es

Un simulador de entrevistas operado desde Claude Code. Dado un JD (URL o
texto), produce una sesión de práctica realista en inglés con tres etapas
(screening → behavioral → technical), conducida por entrevistadores
simulados, evaluada con rúbrica BARS por un LLM-as-judge, y entregada como
reporte HTML self-contained.

## Para quién

Candidatos preparándose para roles **tech en mercados US** (FAANG,
startups, scale-ups). MVP en Fase 1 cubre Software Engineer / Backend; el
sistema está diseñado para extender a otras verticales (product,
consulting, sales) en iteraciones posteriores sin reescribir la
arquitectura.

## Por qué importa

Las herramientas de prep masivas (LeetCode, banks de preguntas) entrenan
contenido pero no la **ejecución bajo presión** ni la **estructura
narrativa**. La psicología organizacional ha demostrado durante 50+ años
que (a) las entrevistas estructuradas predicen desempeño 34% mejor que las
no estructuradas (Schmidt & Hunter, 1998), (b) la ansiedad y la amenaza de
estereotipo degradan desempeño aunque el contenido esté listo
(McCarthy & Goffin), y (c) la práctica simulada con feedback aumenta
autoeficacia y reduce ansiedad (Bandura, evidencia en sims clínicas).

Este producto convierte ese cuerpo de evidencia en práctica concreta:
escenarios calibrados al rol y empresa, rúbrica BARS anclada en conducta,
entrenamiento explícito en estructura (STAR/SOAR/CAR para behavioral,
clarify-decompose-trade-off para technical y system design).

## Alcance funcional

1. **Ingest**: JD (URL o texto pegado) → `jd.md` estructurado.
2. **Research**: dossier de la empresa vía Perplexity MCP (Fase 3).
3. **Plan**: `plan.json` validado contra schema, con preguntas, anclas
   BARS y mapeo etapa → competencia, calibrado a seniority del rol.
4. **Simulación**: tres etapas de entrevista, conducidas como
   role-play en inglés con personas simuladas distintas por etapa
   (recruiter, hiring manager, senior IC/staff).
5. **Evaluación**: LLM-as-judge corre por competencia (un pase por
   dimensión), cita evidencia textual del transcript, asigna score BARS
   y ensambla decisión final (Strong Hire / Hire / Lean No / No Hire).
6. **Reporte**: HTML self-contained con Tailwind + Chart.js, breakdown por
   competencia, transcripts anotados, recomendaciones priorizadas.

## Diferenciadores

- **Plan derivado del JD + research específico de la empresa**, no banks
  genéricos por título.
- **Rúbrica BARS auditada**: cada score anclado a conducta y citado del
  transcript (defensible, no impresión).
- **Modelado psicológico explícito**: el simulador puede activar sesgos
  parametrizados (primacy, halo, affinity, contrast) para entrenar
  resiliencia, mostrando al usuario score "objetivo" vs "percibido".
- **Verticales extensibles**: Fase 1 = tech; product/consulting/sales son
  swap-in en Fase 2+ vía nuevos templates de plan y rúbrica.

## No-objetivos (en MVP)

- No es ATS ni herramienta B2B para reclutadores.
- No automatiza decisiones de contratación reales — es prep, no filtro.
- No replica tests de GMA ni baterías psicométricas (riesgo de adverse
  impact; ver `methodology.md`).
- No promete pasar entrevistas; promete entrenamiento basado en evidencia.

## Métricas de éxito (Fase 1)

- `/interview screening` y `/evaluate` corren end-to-end con
  `templates/example-plan.json` sin intervención manual.
- El reporte HTML abre offline y muestra score por competencia + decisión
  final + transcript anotado.
- El usuario reporta que el feedback se siente accionable (no genérico).
