# Sesgos del entrevistador para simuladores de entrevistas con IA

## 1. Por qué tu simulador debe incluir sesgos

Entrevistar es una tarea humana fuertemente subjetiva: decisiones se toman bajo carga cognitiva, tiempo limitado y con información incompleta, lo que abre la puerta a múltiples sesgos cognitivos.[^1][^2]
Aunque las entrevistas estructuradas y los criterios objetivos reducen significativamente el impacto de estos sesgos, la práctica real en muchas organizaciones sigue incluyendo entrevistas poco estructuradas donde impresiones, similitud y contexto influyen en las evaluaciones.[^2][^3]

Para un simulador, reproducir al menos algunos de estos sesgos hace la experiencia más realista y prepara al candidato para navegar procesos donde el entrevistador no es un "evaluador perfecto" sino un humano con tendencias sistemáticas.[^4][^1]
Una forma robusta de hacerlo es separar dos capas: la puntuación "objetiva" basada en una rúbrica y la puntuación "percibida" por un entrevistador virtual con sesgos parametrizados, mostrando al usuario la discrepancia.

## 2. First-impression bias y primacy

### 2.1 Qué se sabe de las primeras impresiones en entrevistas

Estudios en psicología organizacional y resúmenes divulgados reportan que una proporción importante de entrevistadores forman una opinión sobre la idoneidad del candidato en los primeros minutos de la entrevista.[^5]
Por ejemplo, un artículo que cita investigación publicada en el Journal of Occupational and Organizational Psychology señala que alrededor del 60% de entrevistadores deciden sobre la adecuación del candidato en los primeros 15 minutos, y cerca del 30% lo hace en los primeros cinco minutos.[^5]

Esta rapidez se explica por el efecto de primacía: la información inicial se procesa con más peso y se convierte en un filtro a través del cual se interpretan las respuestas posteriores.[^6]
Una vez formada la impresión inicial (positiva o negativa), el entrevistador tiende a atender y recordar selectivamente aquello que la confirma, conectando con el sesgo de confirmación.[^6]

### 2.2 Implicaciones para el simulador

Para modelar first-impression bias, el simulador puede:

- Asignar un peso mayor a las respuestas iniciales (introducción, "tell me about yourself") en la puntuación percibida del entrevistador, de modo que condicionen la interpretación de respuestas posteriores.[^5][^6]
- Implementar una variable de "impresión inicial" que se forme tras los primeros turnos y que modifique cómo el agente entrevistador evalúa el resto de la conversación (p.ej. umbral más bajo para ver errores si la impresión fue positiva, o más alto si fue negativa).[^6]
- Mostrar al usuario, al final, cuánto de su resultado percibido se explica por los primeros minutos, y ofrecer recomendaciones específicas para optimizar esa parte.

## 3. Halo y horns effect

### 3.1 Definición y evidencia

El halo effect describe la tendencia a dejar que una característica positiva de una persona (p.ej. apariencia, confianza, simpatía) influya en la evaluación de otras dimensiones no observadas directamente (p.ej. competencia técnica).[^7][^8]
A la inversa, el horns effect se refiere a cuando una característica negativa (p.ej. llegar tarde, vestir de forma descuidada) contamina la evaluación global, haciendo que el entrevistador subestime fortalezas relevantes.[^9][^7]

Revisiones recientes subrayan que halo y horns tienen efectos sustanciales en decisiones de reclutamiento, y que el impacto puede variar según ocupación y contexto cultural.[^7]
Además, se ha documentado interacción con estereotipos de género, donde la apariencia física puede ayudar a algunos grupos pero penalizar a otros en roles tradicionalmente masculinizados.[^7]

### 3.2 Implicaciones para el simulador

El simulador puede parametrizar un "coeficiente de halo/horns" que haga que un rasgo inicial dominante (p.ej. nivel percibido de confianza, claridad al hablar, apariencia simulada si se usa video/avatares) arrastre el resto de las calificaciones.

- Si el valor de halo es alto y la primera impresión es positiva, el entrevistador virtual tenderá a sobreestimar competencias menos evidentes, a hacer preguntas de seguimiento más benevolentes y a interpretar ambigüedades a favor del candidato.[^8][^9]
- Con horns, lo contrario: un fallo inicial puede hacer que el agente sea más crítico, formule más preguntas de prueba y evalúe con mayor severidad incluso buenas respuestas.[^9][^7]

El feedback puede mostrar al usuario cuándo se benefició de halo o fue penalizado por horns, dejando claro que no todas las diferencias en puntuación se deben a contenido objetivo.

## 4. Similarity / affinity bias

### 4.1 Qué es y cómo influye en contratación

El affinity bias es la tendencia a favorecer a personas que comparten características, experiencias o intereses con quien evalúa (misma universidad, origen, hobbies, estilo de comunicación).[^10][^11]
Funciona como un atajo cognitivo: la familiaridad se percibe como seguridad, confianza y competencia, aunque no exista evidencia objetiva de mejor desempeño.[^11][^10]

Análisis bibliométricos sobre similarity-attraction en reclutamiento muestran que estas preferencias inconscientes obstaculizan la diversidad e impulsan la homogeneidad en equipos, al premiar repetidamente perfiles parecidos a quienes ya están dentro.[^12]
Además, se ha documentado que, en ausencia de criterios objetivos, muchos managers eligen a quien "les cae mejor" o con quien se imaginan trabajando cómodamente, lo que suele coincidir con personas similares a ellos.[^11]

### 4.2 Implicaciones para el simulador

Para introducir affinity bias de manera controlada, el simulador podría:

- Definir atributos de "background" del entrevistador virtual (universidad, sector, idioma, estilo de trabajo) y permitir que el candidato configure los propios; un alto grado de similitud aumentaría la puntuación percibida independientemente del contenido.[^10][^11]
- Modificar el tono del agente: entrevistadores con alta afinidad se muestran más cálidos, dan más tiempo, reformulan preguntas; con baja afinidad, son más fríos, interrumpen más y ofrecen menos pistas.[^10]
- Mostrar al usuario cómo, en ciertos runs, puntúa mejor con el mismo contenido solo por compartir similitudes con el entrevistador, ilustrando el impacto del sesgo en su contra o a favor.

## 5. Confirmation bias ligado a la impresión inicial

### 5.1 Cómo opera en entrevistas

El confirmation bias es la tendencia a buscar, interpretar y recordar información que confirma creencias o hipótesis previas, ignorando o minimizando datos que las contradicen.[^6]
En entrevistas, una vez que el entrevistador forma una impresión inicial (p.ej. "es fuerte" o "no encaja"), tiende a formular preguntas y ponderar respuestas de modo que refuercen esa hipótesis.[^1][^6]

Artículos recientes sobre los primeros momentos de la entrevista destacan que, tras una primera impresión positiva, los entrevistadores interpretan conductas ambiguas de forma favorable, mientras que tras una impresión negativa las mismas conductas se juzgan con dureza.[^5][^6]
Esto refuerza la importancia de los primeros minutos y se superpone con el efecto halo.

### 5.2 Implicaciones para el simulador

El agente entrevistador puede mantener un "estado de hipótesis" sobre el candidato (fuerte/débil/neutral) actualizado tras las primeras preguntas.

- Si la hipótesis es positiva, el agente tenderá a hacer más preguntas de profundización a logros, dar oportunidades de aclarar respuestas flojas y otorgar el beneficio de la duda.[^6]
- Si es negativa, hará más preguntas desafiantes, se centrará en inconsistencias y penalizará más los vacíos en las respuestas.

El simulador puede mostrar al candidato cómo, en runs distintos, la misma respuesta fue interpretada de manera diferente según la impresión inicial, y subrayar la importancia estratégica del opening.

## 6. Recency effect

### 6.1 Qué es y evidencia en entrevistas

El recency effect se refiere a la tendencia a recordar y ponderar más la información presentada al final de una serie de estímulos.[^2]
En el contexto de puntuación de entrevistas, investigaciones sobre errores de calificación señalan que los entrevistadores pueden sesgar sus ratings hacia lo más reciente, de modo que la última parte de la entrevista (o el último candidato del día) pesa desproporcionadamente.[^2]

Guías de mejores prácticas para entrevistas estructuradas señalan el recency effect como un error típico junto con la tendencia central, la leniencia/severidad y los contrast effects, y recomiendan anotar y calificar por pregunta para mitigarlo.[^3][^2]

### 6.2 Implicaciones para el simulador

Para reflejar recency effect, el simulador puede:

- Ponderar más las últimas respuestas al generar la "impresión global" del entrevistador virtual, especialmente si éste no califica pregunta por pregunta sino sólo al final.[^2]
- Permitir modos donde el candidato ve cómo un cierre fuerte compensa parcialmente un arranque flojo en la percepción global, o al revés, un cierre débil tira hacia abajo una entrevista sólida.

Esto puede usarse didácticamente para entrenar al usuario en la importancia de cerrar con mensajes claros (resumen final, preguntas inteligentes al entrevistador).

## 7. Contrast effect

### 7.1 Evidencia reciente en hiring

El contrast effect hace que la evaluación de un estímulo dependa del contexto inmediato: se juzga al candidato en relación con el anterior o el grupo de ese día, no sólo frente a un estándar absoluto.[^13][^2]
Un estudio reciente sobre entrevistas de admisión y contratación analiza datos de evaluaciones secuenciales y encuentra que la calidad del candidato inmediatamente anterior influye de forma sistemática en las puntuaciones del siguiente.[^13]

En particular, los candidatos tienden a ser evaluados peor cuando siguen a un candidato muy fuerte y mejor cuando siguen a uno débil, con efectos más intensos al inicio del día, cuando el entrevistador ha visto menos casos para calibrar.[^13]
Revisiones sobre buenas prácticas en entrevistas señalan el contrast effect como un error de rating que reduce la validez de la entrevista y recomiendan usar rúbricas claras y paneles para mitigarlo.[^2]

### 7.2 Implicaciones para el simulador

Para modelar el contrast effect, el simulador puede:

- Simular "sesiones" de entrevistas donde el agente ha visto previamente candidatos ficticios con perfiles fuertes o débiles, que influyen en su severidad al evaluar al usuario.[^13]
- Ajustar un parámetro de severidad en función del candidato anterior: después de un perfil simulado excelente, el agente se vuelve más exigente; después de uno mediocre, se vuelve más benevolente.[^13][^2]
- Mostrar al usuario cómo, con el mismo desempeño, su puntuación percibida varía según el contexto de "quién fue antes", enseñando que algunos resultados no dependen sólo de su actuación.

## 8. Cómo parametrizar un entrevistador sesgado en tu sistema

A efectos de diseño, todos estos sesgos pueden representarse mediante parámetros ajustables en el "agente entrevistador":

- **Peso de primacía**: cuánto pesan las primeras respuestas en la hipótesis global sobre el candidato.[^5][^6]
- **Sensibilidad a halo/horns**: cuánto arrastra un rasgo inicial dominante el resto de evaluaciones.[^8][^7]
- **Afinidad**: hasta qué punto la similitud percibida en background/estilo influye en la calificación y en el tono del entrevistador.[^11][^10]
- **Sesgo de confirmación**: cuánto modifica el agente la interpretación de nuevas respuestas en función de la impresión inicial.[^6]
- **Peso de recencia**: cuánto influyen las respuestas finales en la nota global cuando el agente no usa rúbrica pregunta por pregunta.[^2]
- **Sensibilidad a contraste**: cuánto cambia la severidad del agente en función de un "candidato anterior" simulado.[^13][^2]

La recomendación es calcular en paralelo:

1. Un score "objetivo" basado estrictamente en criterios de competencia definidos para el rol.
2. Un score "humano" que pasa por el filtro de un entrevistador virtual con parámetros de sesgo específicos.

Mostrar ambas capas al usuario (y la diferencia entre ellas) ayuda a:

- Entender qué aspectos de su resultado están bajo su control (claridad, ejemplos, estructura) y cuáles se deben a sesgos contextuales.[^4][^2]
- Entrenar estrategias para gestionar esos sesgos (maximizar primeros minutos, cerrar fuerte, mantener consistencia pese a un entrevistador más frío por baja afinidad).

Este enfoque permite que tu simulador se sienta "humano" y altamente realista, al tiempo que educa sobre sesgos frecuentes en selección y promueve una visión crítica de los procesos de entrevista.

---

## References

1. [Understanding Interviewer Bias and How to Prevent It | CHRMP](https://www.chrmp.com/interviewer-bias/) - Structured Interviews. One of the most effective ways to reduce bias in job interviews is by conduct...

2. [Best Practices for Reducing Bias in the Interview Process - PMC - NIH](https://pmc.ncbi.nlm.nih.gov/articles/PMC9553626/) - There is growing literature that using structured interviews reduces bias, increases diversity, and ...

3. [Structured Behavioral Interviews - Human Resources](https://hr.az.gov/structured-behavioral-interviews) - The structured behavioral interview has several strengths that contribute to reliability, validity, ...

4. [Eliminating Biases in Hiring: Structured Interviewing and AI Solutions](https://www.shrm.org/labs/resources/eliminating-biases-in-hiring--structured-interviewing-and-ai-solutions) - Structured interviews standardize questioning around job competencies so that all candidates, regard...

5. [Yale Exposes New Bias That Judges Interviewees Within First Few ...](https://www.forbes.com/sites/heidilynnekurter/2019/10/29/yale-exposes-new-bias-that-judges-interviewees-within-first-few-seconds-of-interview/) - A study by the Journal of Occupational and Organizational Psychology found 60% of interviewers know ...

6. [Why the first few moments of an interview are so crucial](https://www.leonid-group.com/insights/why-the-first-few-moments-of-an-interview-are-so-crucial/)

7. [How Do the Halo Effect and Horn Effect Influence the Human ...](https://www.ewadirect.com/proceedings/chr/article/view/4142) - The results of the literature reviews have indicated that halo and horn effects play a critical role...

8. [Job interviews and the Halo Effect: A hidden bias we need to tackle](https://lyser.com/halo-effect-in-job-interviews/) - The halo effect is a common bias where a positive trait can influence an interviewer's judgment of o...

9. [The Halo and Horns effect in hiring - Velora HR](https://www.velorahr.com/en/post/halo-hons-effect-in-hiring) - The Halo and Horns effect in hiring ... It is advisable to create a script for the interview, highli...

10. [What Is Affinity Bias? Hiring Bias Guide - Hyring](https://hyring.com/free-hr-toolkit/hr-glossary/affinity-bias) - Find out what affinity bias is, how it shapes hiring and promotions, real workplace examples, and pr...

11. [What Is Affinity Bias And Why Does It Matter?](https://www.forbes.com/sites/juliekratz/2024/02/21/what-is-affinity-bias-and-why-does-it-matter/) - Affinity bias is the tendency to favor people who share similar interests, backgrounds and experienc...

12. [[PDF] A Bibliometric Analysis of Similarity Attraction Bias in Recruitment](https://www.rmci.ase.ro/no26vol1/10.pdf)

13. [Sequential contrast effects in hiring and admission interviews - CEPR](https://cepr.org/voxeu/columns/sequential-contrast-effects-hiring-and-admission-interviews) - We also find that contrast effects are stronger at the beginning of an interview day, when evaluator...

