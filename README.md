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

### Requisito único

Necesitas tener instalado **Claude Code**, la herramienta de línea de comandos de Anthropic. Si no la tienes:

1. Ve a [claude.com/claude-code](https://claude.com/claude-code) y sigue las instrucciones de instalación.
2. Configura tu cuenta cuando te lo pida.

### Pasos para correr una sesión de práctica

1. **Abre tu terminal** en la carpeta de este proyecto:
   ```
   cd interview-skills
   claude
   ```

2. **Empieza la primera etapa** escribiendo:
   ```
   /interview screening
   ```
   La primera vez te va a preguntar si quieres usar el plan de ejemplo (Senior Backend Engineer en una empresa ficticia llamada Acme Cloud). Dile que sí.

3. **Conversa con el entrevistador.** Te va a saludar, hacer preguntas, y pedirte aclaraciones cuando algo quede vago. Responde como si fuera real. No te va a decir cómo lo estás haciendo durante la entrevista — eso viene después.

4. **Cuando termines la etapa**, puedes seguir con la siguiente:
   ```
   /interview behavioral
   ```
   y luego:
   ```
   /interview technical
   ```

5. **Cuando estés listo para tu evaluación**, escribe:
   ```
   /evaluate
   ```
   El sistema lee todo lo que dijiste, te puntúa contra criterios objetivos, y genera un reporte en HTML.

6. **Abre el reporte** en tu navegador. La ruta exacta te la imprime el sistema al final, algo como:
   ```
   .claude/specs/acme-cloud-demo-20260503/report.html
   ```

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
| Generar un plan personalizado a partir de un job description | En desarrollo |
| Investigar la empresa automáticamente | En desarrollo |

Si quieres practicar para un rol específico distinto al de ejemplo, por ahora puedes editar manualmente `templates/example-plan.json`. La versión automática (que recibe un URL del job description y arma todo) llega en las próximas fases.

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
├── CLAUDE.md                       Instrucciones del sistema para Claude Code
├── .claude/
│   ├── steering/                   Metodología y rúbrica permanente
│   │   ├── product.md              Visión del producto
│   │   ├── methodology.md          Marco teórico y bases científicas
│   │   └── rubric-framework.md     Sistema de puntuación BARS
│   ├── skills/                     Comandos disponibles
│   │   ├── interview/              /interview <etapa>
│   │   └── evaluate/               /evaluate
│   └── specs/                      Sesiones individuales (ignorado por git)
├── templates/
│   ├── plan-schema.json            Validador del formato del plan
│   ├── example-plan.json           Plan de ejemplo: Senior Backend Engineer
│   └── report.template.html        Plantilla del reporte
└── sources/                        Investigación científica de respaldo
```

## Privacidad

- Tus respuestas se guardan localmente en tu computadora, en `.claude/specs/`.
- Nada se sube a ningún servidor externo (más allá de las llamadas al modelo de Claude que tú ya autorizaste al usar Claude Code).
- El reporte HTML usa Tailwind y Chart.js cargados desde CDN, pero tus datos no salen del archivo.

## Licencia

Por definir.
