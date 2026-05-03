# Rubric Framework — Interview Prep Lab

Definición canónica de la escala BARS, las competencias evaluadas, el
mapeo etapa → competencia, las reglas de agregación, la decisión final y
la calibración del LLM-as-judge. Este documento es el contrato que
`templates/plan-schema.json` y la skill `evaluate/` deben respetar.

## 1. Escala BARS (1-5)

Behaviorally Anchored Rating Scale (Smith & Kendall, 1963). Cada punto se
ancla a conducta observable, no a adjetivos:

| Score | Label              | Behavioral anchor (genérico)                                  |
| ----- | ------------------ | ------------------------------------------------------------- |
| 1     | Does Not Meet      | Falla sistémica. Confusión, evasión, sin estructura, sin datos |
| 2     | Needs Improvement  | Intervención inconsistente. Identifica el problema sin resolver |
| 3     | Meets              | Suficiencia técnica estándar. Resuelve siguiendo protocolos   |
| 4     | Exceeds            | Proactivo, anticipa fricción, enseña matices al panel         |
| 5     | Outstanding        | Maestría. Re-encuadra el problema, conecta dimensiones        |

Cada pregunta del `plan.json` lleva su anchor específico para 1, 3 y 5
(rubric anchoring). Los niveles 2 y 4 se interpolan. Esto es **obligatorio**
para que el LLM-as-judge tenga calibración consistente entre runs.

## 2. Competencias core (12)

Set canónico derivado del diccionario de McClelland/Spencer & Spencer.
Cada plan selecciona un subset; no todas las competencias se evalúan en
todas las etapas (ver §3).

| #  | Competencia              | Definición operativa                                          |
| -- | ------------------------ | ------------------------------------------------------------- |
| 1  | Achievement Orientation  | Orientación al logro, ownership, métricas concretas           |
| 2  | Analytical Thinking      | Descomposición de problemas, razonamiento cuantitativo         |
| 3  | Conceptual Thinking      | Abstracción, síntesis, ver patrones cross-domain              |
| 4  | Developing Others        | Mentoring, code review, feedback constructivo                 |
| 5  | Flexibility              | Adaptabilidad ante ambigüedad, cambio de scope                |
| 6  | Impact and Influence     | Persuasión técnica, alineación de stakeholders                |
| 7  | Information Seeking      | Clarificación, requisitos no explícitos, hacer preguntas      |
| 8  | Initiative               | Acción sin instrucción explícita, anticipación                |
| 9  | Interpersonal Understanding | Lectura de equipo, contexto humano, empatía técnica         |
| 10 | Organizational Awareness | Política interna, dependencias cross-team, prioridades reales |
| 11 | Self-Confidence          | Compostura bajo presión, defensa de decisiones, voice         |
| 12 | Team Leadership          | Coordinación de IC's, decisiones técnicas con consenso        |

## 3. Agrupación en dominios (Hogan)

Para reportes y agregación visual, las 12 se agrupan en los 4 dominios de
Hogan:

- **Intrapersonal**: Self-Confidence, Achievement Orientation
- **Interpersonal**: Impact and Influence, Interpersonal Understanding,
  Developing Others
- **Business / Cognitive**: Analytical Thinking, Conceptual Thinking,
  Information Seeking, Initiative, Flexibility
- **Leadership**: Team Leadership, Organizational Awareness

## 4. Mapeo etapa → competencias

Cada etapa cubre un subset, con peso explícito por competencia:

| Etapa            | Competencias evaluadas (peso ∈ [0,1])                          |
| ---------------- | -------------------------------------------------------------- |
| Screening        | Self-Confidence (0.4), Achievement Orientation (0.4), Information Seeking (0.2) |
| Behavioral       | Initiative (0.25), Flexibility (0.2), Impact and Influence (0.2), Team Leadership (0.2), Interpersonal Understanding (0.15) |
| Technical        | Analytical Thinking (0.4), Conceptual Thinking (0.3), Information Seeking (0.3) |

Los pesos suman 1 dentro de cada etapa. `plan.json` puede sobrescribirlos
por rol cuando el JD lo justifique (e.g., un Staff Engineer pesa más
Conceptual Thinking que Analytical).

## 5. Agregación de scores

Pipeline de cálculo:

1. **Score por pregunta**: 1-5 contra anchor específico.
2. **Score por competencia (en una etapa)**: promedio simple de las
   preguntas que evalúan esa competencia.
3. **Score por etapa**: suma ponderada por los pesos de §4.
4. **Score por competencia (cross-etapa)**: promedio ponderado por #
   preguntas en cada etapa donde aparece.
5. **Score final**: promedio ponderado de competencias core (pesos según
   plan.json).

**Regla no-fail**: si cualquier competencia evaluada cae < 2.0 en
promedio, la decisión se baja a No Hire automáticamente, sin importar el
score final. Esto previene hires con un blind spot crítico.

## 6. Decisión final (4 niveles)

Mapeo de score final a decisión, asumiendo que ninguna competencia falló
(§5 no-fail rule):

| Score final | Decisión     | Significado                                                    |
| ----------- | ------------ | -------------------------------------------------------------- |
| ≥ 4.5       | Strong Hire  | Excede expectativas en core; señales claras de senior potential |
| 3.5 - 4.49  | Hire         | Cumple sólido; gaps menores que se cierran on-the-job          |
| 2.5 - 3.49  | Lean No      | Mixed signals; gaps en competencia core o ejecución débil       |
| < 2.5       | No Hire      | No cumple. O activación de no-fail rule. O kill criterion.     |

## 7. Red flags / kill criteria

Comportamientos que disparan No Hire automático y terminan la sesión
(documentado en el report con cita textual):

- **Honestidad**: faking flagrante. Inconsistencias entre etapas (e.g.,
  rol descrito en screening cambia de seniority en behavioral). Logros
  inverificables sin contexto. El LLM-as-judge debe citar al menos dos
  pasajes contradictorios para activar.
- **Ético**: comportamiento ofensivo, despectivo o discriminatorio en
  role-play. Violación clara de profesionalismo.

Cualquiera de los dos cierra la sesión y produce report con sección
"Critical Issue" al frente. No se permite continuar a la siguiente etapa.

## 8. Calibración del LLM-as-judge

Reglas obligatorias para `evaluate/`:

1. **Rubric anchoring**: cada pregunta lleva ejemplos de respuesta nivel 1,
   3 y 5 en `plan.json`. El juez compara contra esos anchors antes de
   asignar score, no al rango libre.
2. **Aislamiento dimensional**: un pase del LLM por competencia, no
   evaluación holística. Esto previene halo entre dimensiones.
3. **Cita textual obligatoria**: cada score debe citar al menos un pasaje
   del transcript que justifique la decisión. Reports sin citas son
   inválidos y deben re-correrse.
4. **Doble pase para No Hire**: cuando el score inicial sugiere No Hire o
   se dispara una kill criterion, el LLM corre un segundo pase explícito
   buscando evidencia en contra (devil's advocate) antes de confirmar.
5. **Determinismo**: temperatura 0 para el juez, contexto idéntico entre
   runs. Si dos runs sobre el mismo transcript divergen > 0.5 puntos en
   alguna competencia, se reporta como warning de calibración.

## 9. Bias-mode score (modo realista, opcional)

Cuando la sesión activa el modo realista (ver `methodology.md` §6), se
calcula un segundo score "percibido" aplicando los parámetros del
entrevistador sesgado (primacy, halo, affinity, etc.) sobre el score
objetivo. El report muestra ambos lado a lado y la diferencia ("perdiste
0.6 puntos por horns effect tras un opening débil"), entrenando
explícitamente la gestión de impresión.

## 10. Versionado

Cualquier cambio en escala BARS, set de competencias, pesos por etapa,
umbrales de decisión o kill criteria requiere bumpear el `version` campo
en `templates/plan-schema.json`. Los reports incluyen la versión usada
para que sean auditables a posteriori.
