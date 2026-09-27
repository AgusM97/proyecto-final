# A02 — Estado del arte en asistentes conversacionales para la interpretación de documentación técnica

**Proyecto:** Factibilidad de un Asistente Conversacional Inteligente para la Interpretación de Documentación Técnica  
**Estado:** Preliminar (insumo de A02)  
**Versión:** 0.1

Este documento revisa el estado actual de los asistentes conversacionales aplicados a documentación técnica y a dominios afines. Es el producto de la actividad A02 de la propuesta. No selecciona tecnologías (eso corresponde a A03) ni cierra la comparación entre modelo local y servicio en la nube (A04). Sí fija el marco conceptual, los resultados que ya están establecidos y las brechas que justifican el estudio de factibilidad.

El recorte de requerimientos vigente está en [A01](A01-requerimientos.md). Donde la literatura sugiere un requerimiento nuevo o un ajuste de prioridad, se deja explícito en la sección 10 para incorporarlo en la próxima revisión de A01.

---

## 1. Pregunta de la revisión y alcance

La pregunta que organiza esta revisión es la siguiente: **qué se sabe hoy sobre asistentes conversacionales que interpretan documentación técnica anclándose en esa documentación, y qué sigue abierto para un prototipo de factibilidad con recursos limitados.**

| Dentro del alcance | Fuera del alcance |
| --- | --- |
| Asistentes de pregunta-respuesta sobre manuales, especificaciones, normativa técnica, soporte y documentación de APIs | Chatbots generalistas sin corpus propio |
| Recuperación aumentada por generación (RAG) y sus variantes relevantes | Catálogo de productos comerciales |
| Ingesta de PDF y documentos con estructura (secciones, tablas, figuras) | Entrenamiento de un modelo de lenguaje desde cero |
| Evaluación de fidelidad, citas, abstención y utilidad | Certificación de seguridad funcional |
| Evidencia sobre ejecución local y sobre servicios externos, como contexto de A04 | Elección del modelo, del framework o del hardware de referencia |

El caso de uso concreto sigue abierto (A05). Por eso la revisión cubre documentación técnica en sentido amplio: manuales de equipos, guías de soporte, referencias de software y corpus institucionales extensos. Los trabajos sobre normativa universitaria se incluyen porque comparten el problema de fondo —documentos largos, heterogéneos, en los que un error numérico o una omisión importan—, aunque el dominio no sea industrial.

---

## 2. Método

La revisión es narrativa y está organizada por etapas del pipeline, no por un protocolo sistemático tipo PRISMA. Las fuentes se eligieron con tres criterios:

1. Trabajos canónicos que definen RAG, la recuperación densa, las citas y la evaluación.
2. Reportes de experiencia y benchmarks que describen fallos reales al poner RAG en producción o en soporte técnico.
3. Prototipos y tesis recientes (2023–2026) cuyo objeto es consultar manuales, documentación técnica o corpus documentales extensos en lenguaje natural.

Se priorizaron artículos con lugar de publicación identificable (congreso, revista o repositorio institucional) y se evitaron afirmaciones de rendimiento tomadas de blogs o de fichas de producto. Cuando un resultado proviene de un dominio distinto al de los manuales —por ejemplo, pregunta-respuesta sobre Wikipedia—, se indica esa salvedad.

La literatura se mueve rápido. Esta versión cubre el núcleo consolidado hasta 2024 y los trabajos aplicados localizados hasta 2026. A03 puede actualizar el mapa de herramientas sin reescribir este marco.

---

## 3. Del documento estático al asistente anclado

El problema que plantea la propuesta no es nuevo: la información está en el manual, y el costo está en encontrarla. Lo que cambió es el mecanismo de respuesta.

| Etapa | Qué ofrece al usuario | Límite típico |
| --- | --- | --- |
| Búsqueda por palabras (índice, Ctrl+F, BM25) | Pasajes que contienen los términos de la consulta | Hay que saber cómo está escrito el concepto; no redacta una respuesta |
| Pregunta-respuesta extractiva | Un fragmento del documento | No integra pasos dispersos ni admite que la fuente no alcanza con una explicación |
| Modelo de lenguaje sin corpus | Una respuesta fluida a partir de su entrenamiento | Puede contradecir el manual vigente; no cita la versión cargada |
| Asistente RAG | Una respuesta en lenguaje natural condicionada por fragmentos recuperados del corpus | Sigue fallando si la ingesta, la recuperación o la abstención fallan |

Lewis et al. (2020) formularon RAG como la combinación de una memoria paramétrica (un modelo seq2seq preentrenado) y una memoria no paramétrica (un índice denso de pasajes). En tareas de generación, ese esquema produjo lenguaje más específico, diverso y factual que un modelo solo paramétrico. Dos motivos de ese trabajo coinciden con las restricciones de este proyecto: poder **atribuir** la respuesta a una fuente y poder **actualizar** el conocimiento sin reentrenar el modelo. En el prototipo, actualizar el conocimiento equivale a reindexar el manual (RF-13), no a hacer fine-tuning.

Gao et al. (2024) describen el salto posterior a los modelos de lenguaje grandes: RAG deja de ser un modelo entrenado de punta a punta y pasa a ser, sobre todo, un esquema de inferencia. El modelo recibe en el prompt los fragmentos recuperados y redacta la respuesta. Esa es la forma que usan casi todos los prototipos aplicados de la sección 7, y es la forma compatible con el plazo y el hardware de un Proyecto Final (RC-01, RC-05).

Ji et al. (2023) documentan la alucinación como un problema estructural de la generación: el modelo produce contenido fluido que no está sustentado en la fuente. RAG reduce ese riesgo cuando el fragmento correcto entra al contexto. No lo elimina. Chen et al. (2024) muestran, en el benchmark RGB, que los modelos conservan cierta robustez al ruido y, al mismo tiempo, fallan al rechazar preguntas sin soporte, al integrar información dispersa y al resistir evidencia contradictoria. Eso respalda RF-04 y RNF-02 como requisitos de primer orden, no como mejoras posteriores.

---

## 4. Tres paradigmas de RAG

Gao et al. (2024) ordenan el campo en tres paradigmas. Sirven para ubicar el prototipo y para decidir qué queda fuera.

| Paradigma | Idea | Qué implica para este proyecto |
| --- | --- | --- |
| RAG ingenuo | Indexar fragmentos, recuperar los *k* más similares y generar | Es el recorte mínimo demostrable (RF-01 a RF-04, RF-09 a RF-12) |
| RAG avanzado | Mejorar la consulta antes de recuperar, reordenar los pasajes y comprimir el contexto | Es el margen realista si el ingenuo no alcanza en el corpus de A05 |
| RAG modular | Intercambiar recuperador, generador y estrategias de aumento | Coincide con RNF-18: la comparación local/nube exige poder cambiar el generador sin rehacer la recuperación |

El prototipo de factibilidad cabe en el primer paradigma, con piezas puntuales del segundo (búsqueda híbrida, reescritura de seguimientos, reordenamiento) si las mediciones las piden. El tercer paradigma interesa como **criterio de arquitectura** (módulos intercambiables), no como obligación de implementar todas las variantes.

Dos líneas de investigación superan ese recorte y conviene nombrarlas para no redescubrirlas ni adoptarlas por inercia:

- **Self-RAG** (Asai et al., 2024) entrena al modelo para decidir cuándo recuperar y para criticar sus propios pasajes y su propia respuesta mediante tokens de reflexión. Mejora factualidad y calidad de citas, e implica un entrenamiento que A01 ya deja fuera del prototipo salvo un experimento acotado que A03 justifique.
- **GraphRAG** (Edge et al., 2024) construye un grafo de entidades y resúmenes de comunidades para preguntas globales sobre todo el corpus (“¿cuáles son los temas del manual?”). Esa clase de pregunta no es la consulta puntual que motiva el proyecto. Construir el grafo exige muchas llamadas a un modelo y un costo que el estudio de factibilidad no necesita pagar para demostrar el enfoque.

---

## 5. Por qué la documentación técnica es un caso más difícil

La recuperación de pasajes sobre Wikipedia, que es el escenario de Lewis et al. (2020) y de Karpukhin et al. (2020), no reproduce un manual. En documentación técnica la evidencia útil aparece repartida:

- prosa de procedimiento y advertencias;
- tablas de parámetros, códigos de error y unidades;
- ejemplos de código o de comandos, donde un carácter cambia el sentido;
- figuras, esquemas y capturas;
- referencias cruzadas y procedimientos que continúan en otra página;
- varias versiones del mismo documento.

Cheng (2025) lo formula de manera directa en RAG-Flow: un manual técnico no es un conjunto de párrafos. La respuesta correcta puede estar en una fila de tabla, en una etiqueta de un plano o en un procedimiento de varias páginas. Su resultado negativo es tan útil como el positivo: en un conjunto de 200 preguntas, la configuración de mayor contexto no fue la mejor, y entregar la imagen al modelo empeoró preguntas de conteo visual. La brecha que quedó se explica por la preservación de la evidencia, por la selección de esa evidencia y por la capacidad del modelo que responde, no solo por el recuperador de texto.

Proietti et al. (2025) comparan dos caminos sobre un manual real de puesta en marcha de una impresora 3D industrial (90 preguntas frecuentes). Describir cada página con un modelo de lenguaje y luego recuperar esas descripciones puede perder detalle. Un modelo de visión-lenguaje lee la página, y aun así tropieza con el detalle técnico fino. del Rio et al. (2026) llegan a una conclusión compatible en entornos industriales (aeronáutica, electrodomésticos, ensamblaje): la extracción solo textual es una línea de base funcional e insuficiente cuando la fidelidad exige alinear procedimientos con figuras. En su motor, integrar texto y gráficos mejoró un 66 % la extracción de nodos del grafo de conocimiento, y la etapa visual consumió alrededor de dos tercios del tiempo de procesamiento.

Para este proyecto, esa evidencia fija una prioridad ya anticipada en A01 y ahora mejor fundada:

- El prototipo debe demostrar factibilidad sobre **texto estructurado** (secciones, listas, tablas recuperables como texto).
- Voz e imagen permanecen en Could (RF-22, RF-23). La literatura muestra valor y también costo, regresiones y demanda de cómputo. Adoptarlas como Must solo tendría sentido si el caso de uso de A05 fuera inseparable de la figura (un técnico que fotografía una placa, por ejemplo).

---

## 6. Componentes que ya tienen evidencia

### 6.1 Ingesta

El PDF de un manual descarta, al imprimirse, buena parte de la estructura. Auer et al. (2024) presentan Docling como un conversor local, de licencia permisiva, que recupera orden de lectura, layout, tablas y metadatos en hardware modesto, pensado precisamente como entrada de pipelines RAG. No es la única herramienta, y A03 debe comparar alternativas. El hallazgo de estado del arte es otro: **la calidad de la respuesta está limitada por lo que sobrevive a la extracción**. Un índice semántico construido sobre texto basura no se arregla con un modelo más grande.

Eso refuerza RF-09, RF-10, RF-14 y RNF-14, y mantiene RF-15 (OCR de escaneos) como Won't mientras el corpus de A05 tenga capa de texto.

### 6.2 Fragmentación

El RAG ingenuo corta el texto en ventanas de tamaño fijo con solapamiento. Funciona como línea de base y rompe procedimientos, tablas y ejemplos. Cheng (2025) adopta fragmentos conscientes de la sección. La práctica convergente en los prototipos recientes es conservar, en cada fragmento, metadatos de origen: documento, sección, página y, si existe, versión. Eso es exactamente RF-11, y es la condición de RF-03 (citar) y de RF-13 (reemplazar una versión).

No hay un tamaño de fragmento universal. El valor útil se fija con el corpus y se mide. A02 deja ese parámetro para A05 y A08, con la obligación de registrarlo (RNF-19).

### 6.3 Recuperación

Hay dos familias consolidadas:

- **Léxica.** BM25 (Robertson y Zaragoza, 2009) puntúa solapamiento de términos. Recupera códigos de error, nombres de parámetros, identificadores de API y mensajes literales.
- **Densa.** Karpukhin et al. (2020) muestran que un codificador dual, entrenado con pocas preguntas y pasajes, supera a BM25 entre 9 y 19 puntos absolutos en accuracy de top-20 sobre datasets de pregunta-respuesta de dominio abierto. Ese resultado **no se traslada automáticamente** a un manual: allí la consulta del usuario y el texto fuente a menudo comparten la cadena exacta que hay que encontrar.

Por eso los prototipos sobre documentación real combinan ambas señales. Cheng (2025) usa recuperación densa y dispersa. Miralles Martín (2026) indexa cada fragmento de forma semántica y léxica sobre la normativa de la UPM. La fusión de rankings es una técnica anterior y conocida (Cormack, Clarke y Buettcher, 2009); el punto nuevo es que, en documentación técnica, vuelve a ser necesaria.

**Candidato para A01:** búsqueda híbrida (semántica + léxica) con prioridad Should. El Must sigue siendo la recuperación semántica de RF-10; la parte léxica se suma porque los identificadores son parte del dominio, no un refinamiento estético.

Liu et al. (2024) agregan una restricción al otro extremo del pipeline. Aunque el modelo acepte contextos largos, el rendimiento suele ser mayor cuando el pasaje relevante está al principio o al final, y cae cuando queda en el medio. Añadir más documentos recuperados no mejora de forma monótona. Para el prototipo: un *top-k* chico y un orden deliberado, medidos, valen más que volcar el capítulo entero en el prompt. RF-20 (registrar cantidad de fragmentos) pasa a ser un dato de evaluación, no solo de operación.

### 6.4 Generación anclada, citas y abstención

La generación actual condiciona al modelo con instrucciones y con los fragmentos. Tres resultados importan más que la marca del modelo:

1. **Citar es un problema distinto de responder bien.** Gao, Yen, Yu y Chen (2023) evalúan fluidez, corrección y calidad de cita por separado en ALCE. Una respuesta puede ser útil y citar el fragmento equivocado, o citar bien y omitir el paso decisivo. RNF-01 y RNF-03 quedan justificados como métricas distintas.
2. **Decir “no está en la documentación” es una capacidad débil.** En RGB, el rechazo de preguntas sin soporte (*negative rejection*) es uno de los puntos donde los modelos fallan (Chen et al., 2024). RF-04 no puede depender de una instrucción genérica sin casos de prueba. El conjunto dorado de RF-27 tiene que incluir preguntas incontestables con el corpus.
3. **El formato de la respuesta importa en soporte técnico.** Chen et al. (2025) muestran que las métricas genéricas de similitud no verifican términos clave ni el orden y la completitud de los pasos. Su marco TechSupportEval, construido sobre TechQA, introduce pruebas específicas para esos dos fallos. Si A05 elige un corpus de procedimientos, A08 tiene que mirar el orden de los pasos, no solo el parecido léxico con una respuesta de referencia.

### 6.5 Conversación

RF-05 pide contexto de diálogo. El recuperador, en el esquema ingenuo, solo ve la última utterance. “¿Y el parámetro anterior?” no se parece al pasaje del manual. El RAG avanzado reescribe esa utterance a una consulta autónoma antes de buscar (Gao et al., 2024). Conviene tratar la reescritura como parte de RF-05, no como un módulo opcional desconectado.

La memoria persistente editada por el usuario —guardar respuestas corregidas y reutilizarlas como fuente— aparece en prototipos aplicados. Es una forma de humano en el ciclo. Barnett et al. (2024) concluyen que la robustez de un RAG se observa en operación y evoluciona; no queda diseñada por completo al inicio. Para este proyecto, el humano en el ciclo entra por el **protocolo de evaluación** (RF-25, RF-26), no por un editor de conocimiento dentro del prototipo. Incorporarlo como producto diluiría el estudio.

---

## 7. Trabajos aplicados y dominios afines

La tabla siguiente no es exhaustiva. Reúne sistemas cuyo objeto se parece al de este proyecto, para ver qué ya se demostró y qué dejó cada uno sin resolver.

| Trabajo | Corpus y dominio | Enfoque | Qué muestra para este proyecto |
| --- | --- | --- | --- |
| Castelli et al. (2020), TechQA | Preguntas reales de foros IBM y technotes; incluye preguntas sin respuesta | QA de dominio, adaptación, no un chatbot generativo | El soporte técnico real tiene pocas preguntas etiquetadas y muchas consultas incontestables |
| Robles López (2023), ChatUPM | Normativa UPM | Embeddings y GPT con API web | Patrón temprano de “recuperar y generar” sobre PDF institucionales |
| López Montaner (2024) | Documentación de soporte de una empresa | RAG sobre servicios de nube | El asistente mejora el acceso; los autores dejan margen de mejora explícito |
| Righetto (2025) | Manuales técnicos en PDF | Índice vectorial y LLM en la nube; cliente en tablet | La dependencia de internet, la latencia y el dispositivo de campo son parte del problema, no un detalle de despliegue |
| Miralles Martín (2026) | Normativa UPM en PDF | Híbrido léxico-semántico y **modelo local** | Hay respuestas alineadas con la fuente y también errores numéricos y omisiones |
| Cheng (2025), RAG-Flow | 14 PDF de manuales; 200 preguntas de validación | Fragmentos por sección, híbrido, presets con y sin imagen | La evidencia visual no mejora todas las preguntas; preservar la evidencia es el cuello de botella |
| Proietti et al. (2025) | Manual de puesta en marcha industrial; 90 FAQ | RAG textual frente a modelo de visión-lenguaje | Cada camino pierde un tipo distinto de información |
| del Rio et al. (2026) | Manuales industriales multimodales | Grafo de conocimiento texto + visión | El salto de calidad existe y el costo de cómputo también |
| Yang et al. (2025), RAGVA | Asistente de una operadora de autopistas | Reemplazo de un asistente por reglas | Llevar RAG a un uso real sigue siendo un problema de ingeniería y de evaluación, no solo de modelo |
| Chen et al. (2025), TechSupportEval | Respuestas RAG sobre una adaptación de TechQA | Evaluación automática de términos y de pasos; comparan GPT-4o mini con Llama 3 de 8B y de 70B | En soporte técnico hay que medir otra cosa que la similitud del texto; el tamaño del modelo se puede comparar con la misma batería |

Lectura conjunta, relevante para la factibilidad:

- El patrón “PDF → fragmentos → índice → LLM → chat” ya está reproducido en tesis y en prototipos. **Construir un chat RAG no es, por sí solo, el aporte.**
- El aporte posible está en medir, sobre un corpus acotado y con el hardware disponible, precisión, citas, abstención, utilidad, costo, privacidad y el límite local frente a nube.
- Casi ningún trabajo cierra a la vez heterogeneidad documental, citas verificables, abstención, modo local y modo nube. Esa combinación es la que piden el objetivo general y RC-02.
- Los sistemas que suben a grafos multimodales o a entrenamiento propio obtienen cobertura mayor y salen del recorte de un prototipo académico con recursos limitados.

---

## 8. Evaluación: qué medir y dónde se rompe

### 8.1 Métricas

Es et al. (2024) proponen RAGAS para evaluar sin respuesta de referencia tres dimensiones: fidelidad de la respuesta al contexto recuperado, relevancia de la respuesta respecto de la pregunta y foco del contexto recuperado. En su conjunto WikiEval, las predicciones se alinean con el juicio humano sobre todo en fidelidad y relevancia de la respuesta. RAGAS sirve para iterar rápido. No reemplaza un conjunto dorado ni a un lector que conozca el manual.

Para A08, la combinación coherente con la literatura y con A01 es:

| Dimensión (A01, §8) | Evidencia que el estado del arte considera válida |
| --- | --- |
| Precisión | Conjunto dorado con respuesta de referencia y juicio humano |
| Anclaje | Fidelidad al fragmento (RAGAS u otra rúbrica) y trazabilidad de afirmaciones (RNF-01) |
| Citas | Precisión de cita, separada de la corrección (ALCE; RNF-03) |
| Abstención | Preguntas incontestables y *negative rejection* (RGB; TechQA) |
| Procedimientos | Términos clave y orden de pasos, si el corpus es operativo (TechSupportEval) |
| Utilidad | Protocolo breve con usuarios reales o simulados (RF-26) |
| Local vs. nube | La misma batería en ambos modos (A04 + A08) |

Los umbrales numéricos siguen sin fijarse, como ya dice A01: dependen del corpus y del piloto.

### 8.2 Siete puntos de fallo

Barnett et al. (2024) sintetizan tres casos (investigación, educación, biomedicina) en siete fallos de ingeniería. El mapeo al prototipo evita tratarlos como sorpresas tardías.

| Fallo | Qué ocurre | Requerimiento o decisión asociada |
| --- | --- | --- |
| FP1. Contenido ausente | La pregunta no se puede responder con los documentos | RF-04; preguntas incontestables en el conjunto dorado |
| FP2. No entra en el top-k | La respuesta está en el corpus y el ranking no la devuelve | Búsqueda híbrida; fragmentación; medir *top-k* |
| FP3. No entra al contexto | El pasaje se recuperó y la estrategia de consolidación lo descartó | Presupuesto de contexto (Liu et al., 2024) |
| FP4. No se extrae | El pasaje está en el prompt y el modelo no lo usa, a menudo por ruido o contradicción | RNF-02; RNF-04 |
| FP5. Formato incorrecto | La respuesta no respeta el tipo pedido (pasos, tabla, valor) | RF-08, en Could; rúbrica de A08 si el caso es procedural |
| FP6. Especificidad incorrecta | La respuesta es demasiado genérica o demasiado estrecha | RF-06 (pedir aclaración) |
| FP7. Respuesta incompleta | Falta una parte necesaria | Completitud en el conjunto dorado; pasos en orden |

Las dos conclusiones de Barnett et al. (2024) encajan con un estudio de factibilidad: la validación seria ocurre con el sistema en uso, y la robustez se ajusta con evidencia, no se declara en el diseño. Por eso RF-25 (registro de cada corrida) es Must.

---

## 9. Ejecución local y servicio externo

A04 debe comparar los dos modos. A02 solo establece qué dice ya la literatura, para que esa comparación no empiece de cero.

| Eje | Qué está reportado | Qué queda abierto |
| --- | --- | --- |
| Calidad | Modelos grandes de API y modelos locales grandes suelen redactar mejor que modelos locales chicos. TechSupportEval compara, en soporte técnico, GPT-4o mini con Llama 3 de 8B y de 70B (Chen et al., 2025). Miralles Martín (2026), con modelo local, observa aciertos y también errores numéricos | La brecha en el corpus y el hardware de este proyecto |
| Privacidad | El modo local puede evitar enviar el manual y la consulta a un tercero. Varios prototipos de campo y de empresa tratan la salida de datos como una restricción de diseño | Qué se transmite exactamente en el modo nube elegido (RNF-06) |
| Conectividad | Righetto (2025) señala la dependencia de internet como límite para uso en campo | Si el caso de A05 necesita offline (RF-19 es Should) |
| Hardware | La ingesta con herramientas como Docling cabe en hardware común (Auer et al., 2024). El modelo generativo es la pieza pesada. del Rio et al. (2026) muestran que la etapa visual domina el tiempo | Si el modelo local y el índice entran en el equipo de referencia (RNF-10). Si no entran, el resultado de factibilidad es negativo y sigue siendo un resultado |
| Costo | El modo nube se contabiliza por tokens; el modo local, por equipo y por latencia | Medición sobre la misma batería (RNF-09, RNF-11) |
| Arquitectura | El paradigma modular permite el mismo recuperador con dos generadores (Gao et al., 2024) | Diseño concreto en A06, alineado con RF-18 |

Nada en la literatura revisada declara inviable el modo local para un corpus acotado, ni lo declara equivalente en calidad a un servicio de frontera. Esa equivalencia, o su ausencia, es la pregunta empírica de A04.

---

## 10. Implicancias para A01 y para las actividades siguientes

### 10.1 Candidatos a incorporar en A01

No se modifican todavía los identificadores vigentes. Estos ítems se proponen para la próxima revisión de requerimientos.

| Candidato | Prioridad sugerida | Fundamento | Se apoya en |
| --- | --- | --- | --- |
| Búsqueda híbrida semántica y léxica | Should | Identificadores, códigos y nombres propios del manual | Cheng (2025); Miralles Martín (2026); salvedad sobre Karpukhin et al. (2020) |
| Reescritura de la consulta de seguimiento antes de recuperar | Should, como parte de RF-05 | El recuperador no interpreta elipsis ni deixis | Gao et al. (2024) |
| Fragmentación consciente de la estructura (secciones, listas, tablas) | Should, refina RF-10 y RF-14 | El corte fijo rompe procedimientos y tablas | Cheng (2025); Auer et al. (2024) |
| Preguntas incontestables dentro del conjunto dorado | Must, refina RF-27 | La abstención no se observa si todas las preguntas tienen respuesta | Chen et al. (2024); Castelli et al. (2020) |
| Rúbrica de cita distinta de la rúbrica de corrección | Must, explicita RNF-03 | Se puede acertar el hecho y citar mal, o al revés | Gao et al. (2023) |
| Métrica de fidelidad al contexto, complementaria del juicio humano | Should | Acelera la iteración; no sustituye al lector | Es et al. (2024) |
| Si el corpus es procedural: control de términos clave y de orden de pasos | Should, condicionado a A05 | La similitud de texto no ve un paso invertido | Chen et al. (2025) |
| Presupuesto de contexto y *top-k* registrados | Must de medición, ya insinuado por RF-20 y RNF-19 | Más fragmentos no implican mejor respuesta | Liu et al. (2024) |
| Humano en el ciclo como protocolo de evaluación, no como editor del índice | Should en A08 | La robustez se observa en uso | Barnett et al. (2024) |

Se mantienen las prioridades de A01 sobre multimodalidad (texto Must; voz e imagen Could) y sobre fine-tuning pesado (fuera del prototipo). Self-RAG, GraphRAG y el grafo multimodal industrial quedan como trabajo futuro o como experimento solo si A03 muestra un costo compatible con RC-01 y RC-05.

### 10.2 Qué debe tomar cada actividad posterior

| Actividad | Insumo de este informe |
| --- | --- |
| A03 | Comparar modelos, embeddings y almacenes dentro del paradigma modular. Incluir un recuperador híbrido entre las alternativas. Tratar el parser de PDF como componente de calidad, no como utilidad invisible. Dejar Self-RAG y GraphRAG fuera de la shortlist salvo justificación de costo |
| A04 | Misma batería, dos generadores. Medir calidad, abstención, citas, latencia, memoria, costo y si el texto sale del equipo. Un resultado “el modelo local no entra en el hardware” es un hallazgo válido (RNF-10) |
| A05 | Elegir un corpus con capa de texto, de tamaño acotado, y anotar de antemano dónde viven tablas, códigos y procedimientos. Definir preguntas contestables e incontestables |
| A06 | Pipeline con fronteras claras: ingesta, índice, recuperación, generación, registro. El modo local/nube se cambia en la frontera de generación |
| A08 | Rúbrica de la sección 8.1. Registro por consulta (RF-25). Juicio humano además de cualquier métrica automática |

---

## 11. Brechas que justifican el estudio

Después de esta revisión, el problema de la propuesta sigue abierto en cuatro puntos concretos:

1. **Heterogeneidad con recursos limitados.** Los sistemas que tratan de verdad tablas, figuras y procedimientos cruzados existen, y exigen parsers, modelos de visión o grafos cuyo costo hay que justificar. Falta evidencia de hasta dónde llega un pipeline textual cuidadoso —extracción con estructura, fragmentos con metadatos, búsqueda híbrida, citas— sobre un corpus técnico real y chico.
2. **Confiabilidad cuando el error importa.** Citas, abstención e integración de pasos siguen siendo fallos reportados incluso con RAG. La factibilidad no es “¿el chat responde?”; es “¿se puede saber cuándo la respuesta está sustentada?”.
3. **Local y nube en la misma tubería.** Hay prototipos de cada lado por separado. Hay poca evidencia pública que los compare, con el mismo índice y las mismas preguntas, en hardware de una persona y con privacidad como criterio de primer nivel.
4. **Evaluación transferible al dominio.** RAGAS y los benchmarks de Wikipedia no alcanzan para un manual operativo. TechSupportEval acerca la métrica al soporte técnico y sigue siendo un marco para adaptar, no un resultado del corpus que este proyecto todavía no eligió.

Esas cuatro brechas son el lugar del prototipo. El estado del arte ya ofrece el patrón de arquitectura y el catálogo de fallos. No ofrece el dictamen de factibilidad para el caso, el equipo y las restricciones de este Proyecto Final.

---

## 12. Lectura en una página

Un asistente útil para documentación técnica es, en el estado actual, un sistema RAG: recupera fragmentos del corpus cargado y redacta una respuesta condicionada por esos fragmentos, con cita y con permiso explícito para decir que la fuente no alcanza. El esquema ingenuo ya está muy repetido en tesis y prototipos; el aporte de un estudio de factibilidad está en medirlo.

La documentación técnica se distingue de la pregunta-respuesta de dominio abierto porque la evidencia está en tablas, códigos, pasos y, a veces, figuras. La búsqueda híbrida, la fragmentación por estructura y un contexto corto y bien ordenado son las mejoras con mejor relación entre evidencia y costo. Los grafos multimodales y el entrenamiento del modelo para que se autocritique mejoran cobertura o factualidad a un costo que este prototipo no debe asumir de entrada.

La evaluación tiene que separar corrección, fidelidad, cita, abstención y, si el corpus es un procedimiento, orden de los pasos. La comparación local/nube sigue siendo una pregunta empírica: la literatura muestra que ambos caminos se usan y que ninguno está cerrado para un corpus acotado.

---

## Referencias

Asai, A., Wu, Z., Wang, Y., Sil, A. y Hajishirzi, H. (2024). Self-RAG: Learning to retrieve, generate, and critique through self-reflection. *International Conference on Learning Representations (ICLR)*.

Auer, C., Lysak, M., Nassar, A., Dolfi, M., Livathinos, N. et al. (2024). *Docling technical report* (arXiv:2408.09869).

Barnett, S., Kurniawan, S., Thudumu, S., Brannelly, Z. y Abdelrazek, M. (2024). Seven failure points when engineering a retrieval augmented generation system. *Proceedings of the 3rd International Conference on AI Engineering (CAIN)*. arXiv:2401.05856.

Castelli, V., Chakravarti, R., Dana, S., Ferritto, A., Florian, R., Franz, M., Garg, D., Khandelwal, D., McCarley, S., McCawley, M., Nasr, M., Pan, L., Pendus, C., Pitrelli, J., Pujar, S., Roukos, S., Sakrajda, A., Sil, A., Uceda-Sosa, R., Ward, T. y Zhang, R. (2020). The TechQA dataset. *Proceedings of the 58th Annual Meeting of the Association for Computational Linguistics*, 1269–1278.

Chen, B., Sun, Y., Liu, Y., Xu, L., Xie, Z., Pei, C., Han, J., Ni, F., Cai, X., Yang, C. y Pei, D. (2025). TechSupportEval: An automated evaluation framework for technical support question answering. *International Joint Conference on Neural Networks (IJCNN)*. IEEE.

Chen, J., Lin, H., Han, X. y Sun, L. (2024). Benchmarking large language models in retrieval-augmented generation. *Proceedings of the AAAI Conference on Artificial Intelligence*.

Cheng, Z. (2025). *RAG-Flow: A multimodal retrieval-augmented generation pipeline for technical manuals* [Tesis de maestría, Politecnico di Milano].

Cormack, G. V., Clarke, C. L. A. y Buettcher, S. (2009). Reciprocal rank fusion outperforms Condorcet and individual rank learning methods. *Proceedings of the 32nd International ACM SIGIR Conference on Research and Development in Information Retrieval*, 758–759.

del Rio, A., Ruiz, V., Petit, A., Colomines, L., Choropanitis, P. y Serrano, J. (2026). A multimodal GraphRAG engine for semantic knowledge extraction in industrial environments. *Expert Systems with Applications*. https://doi.org/10.1016/j.eswa.2026.133441

Edge, D., Trinh, H., Cheng, N., Bradley, J., Chao, A., Mody, A., Truitt, S., Metropolitansky, D., Osazuwa Ness, R. y Larson, J. (2024). *From local to global: A GraphRAG approach to query-focused summarization* (arXiv:2404.16130).

Es, S., James, J., Espinosa Anke, L. y Schockaert, S. (2024). RAGAS: Automated evaluation of retrieval augmented generation. *Proceedings of the 18th Conference of the European Chapter of the Association for Computational Linguistics: System Demonstrations*, 150–158.

Gao, T., Yen, H., Yu, J. y Chen, D. (2023). Enabling large language models to generate text with citations. *Proceedings of the 2023 Conference on Empirical Methods in Natural Language Processing*.

Gao, Y., Xiong, Y., Gao, X., Jia, K., Pan, J., Bi, Y., Dai, Y., Sun, J., Wang, M. y Wang, H. (2024). *Retrieval-augmented generation for large language models: A survey* (arXiv:2312.10997).

Ji, Z., Lee, N., Frieske, R., Yu, T., Su, D., Xu, Y., Ishii, E., Bang, Y. J., Madotto, A. y Fung, P. (2023). Survey of hallucination in natural language generation. *ACM Computing Surveys, 55*(12).

Karpukhin, V., Oguz, B., Min, S., Lewis, P., Wu, L., Edunov, S., Chen, D. y Yih, W. (2020). Dense passage retrieval for open-domain question answering. *Proceedings of the 2020 Conference on Empirical Methods in Natural Language Processing*, 6769–6781.

Lewis, P., Perez, E., Piktus, A., Petroni, F., Karpukhin, V., Goyal, N., Küttler, H., Lewis, M., Yih, W., Rocktäschel, T., Riedel, S. y Kiela, D. (2020). Retrieval-augmented generation for knowledge-intensive NLP tasks. *Advances in Neural Information Processing Systems, 33*, 9459–9474.

Liu, N. F., Lin, K., Hewitt, J., Paranjape, A., Bevilacqua, M., Petroni, F. y Liang, P. (2024). Lost in the middle: How language models use long contexts. *Transactions of the Association for Computational Linguistics, 12*, 157–173.

López Montaner, I. (2024). *Asistente virtual para soporte técnico basado en inteligencia artificial* [Trabajo fin de grado, Universidad de Zaragoza].

Miralles Martín, C. (2026). *Diseño e implementación de un asistente conversacional RAG sobre la normativa de la Universidad Politécnica de Madrid* [Trabajo fin de máster, Universidad Politécnica de Madrid]. https://oa.upm.es/94615/

Proietti, S., Sabetta, N., Rosi, M., Fiocco, E., Colabianchi, S. y Cesarotti, V. (2025). What does it say? A comparative study of large language models and vision-language models in retrieval augmented generation for the analysis of manufacturing technical documentation. *XXX AIDI Summer School “Francesco Turco”*, Lecce.

Righetto, J. (2025). *Engineering a RAG chatbot for technical manual navigation through vector search and cloud LLM integration* [Tesis de maestría, Università degli Studi di Padova].

Robertson, S. y Zaragoza, H. (2009). The probabilistic relevance framework: BM25 and beyond. *Foundations and Trends in Information Retrieval, 3*(4), 333–389.

Robles López, P. (2023). *ChatUPM, aplicación de modelos GPT a la normativa de la Universidad Politécnica de Madrid* [Trabajo fin de máster, Universidad Politécnica de Madrid]. https://oa.upm.es/75824/

Yang, R., Fu, M., Tantithamthavorn, C., Arora, C., Vandenhurk, L. y Chua, J. (2025). RAGVA: Engineering retrieval augmented generation-based virtual assistants in practice. *Journal of Systems and Software*.
