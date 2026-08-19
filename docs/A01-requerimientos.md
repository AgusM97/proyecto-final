# A01 — Determinación de requerimientos del sistema

**Proyecto:** Factibilidad de un Asistente Conversacional Inteligente para la Interpretación de Documentación Técnica  
**Estado:** Preliminar (punto de partida de A01)  
**Versión:** 0.1

Este documento propone un primer conjunto de requerimientos funcionales y no funcionales para el prototipo y para el estudio de factibilidad. Son un insumo de trabajo: se refinarán con A02 (estado del arte), A03–A04 (tecnologías y local vs. nube) y A05 (caso de uso y documentación concreta).

---

## 1. Propósito y alcance

El sistema objeto de estudio es un **asistente conversacional** que permite consultar documentación técnica en lenguaje natural y obtiene respuestas **ancladas** en esa documentación (enfoque RAG), en lugar de depender solo del conocimiento general del modelo de lenguaje.

El trabajo es un **estudio de factibilidad** con prototipo, no un producto de producción. Los requerimientos distinguen:

| Alcance | Qué cubre |
| --- | --- |
| Prototipo (Must / Should) | Lo mínimo para demostrar y validar el enfoque |
| Sistema objetivo (Could) | Capacidades deseables si la factibilidad se confirma |
| Fuera de alcance del prototipo | Lo que no se pretende construir en este proyecto |

**Caso de uso aún no fijado (A05).** Los requerimientos se formulan de manera independiente del dominio (manuales de equipos, documentación de lenguajes/APIs, guías de configuración, etc.). Al elegir el caso de uso, algunos ítems se especializarán (formatos, volumen, idioma, criticidad de un error).

---

## 2. Stakeholders

| Actor | Interés |
| --- | --- |
| Usuario consultante | Resolver una duda puntual sin leer el documento completo |
| Autor / dueño de la documentación | Que las respuestas respeten el contenido y la versión vigentes |
| Operador del prototipo (alumno / director) | Cargar documentos, elegir backend (local o nube) y registrar evidencia de validación |
| Evaluadores académicos | Poder juzgar factibilidad: precisión, utilidad, límites, costo, privacidad y hardware |

---

## 3. Premisas y restricciones (RC)

Estas restricciones condicionan el resto de los requerimientos y alimentan A03–A04.

| ID | Restricción |
| --- | --- |
| RC-01 | Recursos computacionales limitados (equipo personal / entorno académico; no se asume un cluster). |
| RC-02 | El prototipo debe poder compararse en al menos dos modos de inferencia: **modelo local** y **servicio externo en la nube**. |
| RC-03 | La documentación de entrada es **heterogénea** (manuales, especificaciones, referencias, historiales de versión) y no está necesariamente estructurada para búsqueda. |
| RC-04 | Un error en la respuesta puede tener **consecuencias relevantes** en el dominio real; el prototipo debe explicitar incertidumbre y citar fuentes, no “inventar” con aparente certeza. |
| RC-05 | Plazo y esfuerzo de un Proyecto Final: priorizar un recorte vertical demostrable (un caso de uso, un corpus acotado) frente a cobertura amplia. |
| RC-06 | El idioma de la interfaz y de las respuestas será, en el prototipo, **español y/o inglés**, según el corpus elegido en A05. |

---

## 4. Casos de uso de alto nivel

```text
CU-01  Consultar la documentación en lenguaje natural
CU-02  Hacer una pregunta de seguimiento (contexto de conversación)
CU-03  Recibir la respuesta con citas al documento (sección / página / fragmento)
CU-04  Indicar que la documentación no alcanza para responder
CU-05  Ingerir / indexar un conjunto de documentos técnicos
CU-06  Elegir backend de inferencia (local vs. nube)
CU-07  (opcional) Consultar por voz
CU-08  (opcional) Adjuntar una imagen (captura de error, diagrama, fragmento de manual)
CU-09  Registrar la interacción para evaluación (A08)
```

---

## 5. Requerimientos funcionales (RF)

Prioridad: **Must** (imprescindible en el prototipo), **Should** (importante si el tiempo lo permite), **Could** (sistema objetivo / multimodal), **Won't** (explícitamente fuera del prototipo).

### 5.1 Consulta conversacional

| ID | Requerimiento | Prioridad | Trazabilidad |
| --- | --- | --- | --- |
| RF-01 | El usuario podrá formular preguntas en lenguaje natural sobre la documentación cargada, sin conocer la estructura del documento. | Must | Objetivo general; CU-01 |
| RF-02 | El sistema generará la respuesta **apoyándose en fragmentos recuperados** de esa documentación (RAG), no solo en el conocimiento paramétrico del LLM. | Must | Introducción (RAG); RC-04 |
| RF-03 | Cada respuesta incluirá **citas** al origen: documento, ubicación (sección, página o identificador de fragmento) y, si es viable, un extracto. | Must | Precisión / confiabilidad |
| RF-04 | Si la evidencia recuperada es insuficiente o contradictoria, el sistema **lo declarará** (no rellenará con información no sustentada). | Must | RC-04 |
| RF-05 | El sistema mantendrá **contexto de conversación** para preguntas de seguimiento (“¿y el parámetro anterior?”, “mostrame el ejemplo”). | Should | CU-02 |
| RF-06 | Ante consultas ambiguas, el sistema podrá pedir **aclaración** (producto, versión, sección, síntoma) antes de responder. | Should | Utilidad percibida |
| RF-07 | El usuario podrá restringir la consulta a un subconjunto del corpus (un manual, una versión, un módulo). | Could | Documentación heterogénea |
| RF-08 | El sistema distinguirá, cuando sea posible, entre **procedimiento**, **referencia** y **advertencia/limitación** presentes en la fuente. | Could | Interpretación de docs técnicas |

### 5.2 Ingesta e indexación

| ID | Requerimiento | Prioridad | Trazabilidad |
| --- | --- | --- | --- |
| RF-09 | El prototipo permitirá incorporar documentación técnica del caso de uso en al menos **PDF** y un formato de texto estructurado (**Markdown** o **HTML**). | Must | A05; RC-03 |
| RF-10 | El pipeline de ingesta extraerá texto (y, si aplica, títulos / headings) y lo **fragmentará e indexará** para recuperación semántica (y, si se evalúa, léxica). | Must | RAG; A03 |
| RF-11 | Cada fragmento indexado conservará **metadatos** mínimos: documento de origen, versión (si existe), ubicación y fecha de ingesta. | Must | RF-03 |
| RF-12 | El prototipo soportará un corpus **acotado pero real** (orden de magnitud a fijar en A05: p. ej. un manual + anexos, o un conjunto de páginas de API). | Must | RC-05 |
| RF-13 | La reindexación de un documento actualizado reemplazará la versión anterior en el índice (o permitirá consultar por versión). | Should | Historiales de versión |
| RF-14 | Ingesta de código de ejemplo, tablas y listas sin perder del todo su estructura. | Could | Docs de APIs / lenguajes |
| RF-15 | OCR o parsing avanzado de PDFs escaneados / diagramas complejos. | Won't | Complejidad vs. plazo |

### 5.3 Despliegue e inferencia

| ID | Requerimiento | Prioridad | Trazabilidad |
| --- | --- | --- | --- |
| RF-16 | El prototipo podrá responder usando un **LLM local** (sin enviar el texto de la documentación a un proveedor externo). | Must | Objetivo específico local vs. nube |
| RF-17 | El prototipo podrá responder usando un **servicio de LLM en la nube**, con la misma interfaz de consulta, para comparación. | Must | A03–A04 |
| RF-18 | El modo de inferencia (local / nube) será **configurable** sin reescribir la lógica de recuperación. | Must | Arquitectura intercambiable (A06) |
| RF-19 | En modo local, el sistema funcionará **sin conexión a internet** una vez descargados modelo e índice (salvo la propia UI si se hospeda en red). | Should | Disponibilidad offline |
| RF-20 | El sistema expondrá (en logs o panel simple) **indicadores de corrida**: latencia de recuperación, latencia de generación, modelo usado, cantidad de fragmentos. | Should | A04 y A08 |

### 5.4 Multimodalidad

La propuesta menciona interfaces de texto, voz e imagen. Para un prototipo de factibilidad se sugiere **texto como Must** y el resto como Could, salvo que A05 justifique lo contrario (p. ej. un técnico de campo que necesita manos libres).

| ID | Requerimiento | Prioridad | Trazabilidad |
| --- | --- | --- | --- |
| RF-21 | Interfaz de consulta y respuesta en **texto** (chat). | Must | Introducción |
| RF-22 | Entrada y/o salida por **voz**. | Could | Interfaz multimodal |
| RF-23 | El usuario podrá adjuntar una **imagen** (captura de error, foto de una placa/etiqueta, recorte de un manual) para contextualizar la pregunta. | Could | Interfaz multimodal |
| RF-24 | El sistema podrá señalar en la respuesta **figuras o tablas** del documento cuando el fragmento recuperado las referencie. | Could | Docs técnicas ilustradas |

### 5.5 Evaluación (soporte a A08)

| ID | Requerimiento | Prioridad | Trazabilidad |
| --- | --- | --- | --- |
| RF-25 | El prototipo registrará, para cada consulta: pregunta, fragmentos recuperados, respuesta, citas, modo (local/nube) y tiempos. | Must | Metodología: validación |
| RF-26 | El evaluador (usuario real o simulado) podrá marcar la respuesta como útil / no útil y, opcionalmente, comentar el error (alucinación, omisión, cita incorrecta). | Should | Utilidad percibida |
| RF-27 | Existirá un conjunto de **preguntas de prueba** derivado del corpus (golden set) para medir acierto de forma repetible. | Must | Precisión de respuestas |

---

## 6. Requerimientos no funcionales (RNF)

### 6.1 Precisión y confiabilidad

| ID | Requerimiento | Prioridad | Notas de medición (A08) |
| --- | --- | --- | --- |
| RNF-01 | Las afirmaciones factuales de la respuesta deberán ser **trazables** a al menos un fragmento citado. | Must | Tasa de afirmaciones sin soporte |
| RNF-02 | Se buscará minimizar **alucinaciones** (contenido plausible pero ausente o contradictorio con la fuente). | Must | Tasa de alucinación sobre golden set |
| RNF-03 | Las citas deberán corresponder al fragmento realmente usado (no citar “de adorno”). | Must | Precisión de citas |
| RNF-04 | Ante conflicto entre documentos o versiones, el sistema no elegirá una en silencio: lo **explicitará**. | Should | Casos de prueba de conflicto |

Umbrales numéricos (p. ej. “≥ 80 % de respuestas aceptables”) se fijarán **después** de A05 y de un piloto sobre el corpus real; imponerlos ahora sería arbitrario.

### 6.2 Privacidad y tratamiento de datos

| ID | Requerimiento | Prioridad |
| --- | --- | --- |
| RNF-05 | En modo local, el texto de la documentación y de las consultas **no se enviará** a APIs externas. | Must |
| RNF-06 | En modo nube, se documentará qué se transmite (prompt, fragmentos, metadatos) y se usará un corpus **no confidencial** o con autorización explícita. | Must |
| RNF-07 | El prototipo no usará las consultas para reentrenar un modelo propio ni habilitará *training* en el proveedor, si este lo permite desactivar. | Should |

### 6.3 Desempeño y recursos

| ID | Requerimiento | Prioridad |
| --- | --- | --- |
| RNF-08 | En modo nube, el tiempo hasta la primera respuesta útil debería mantenerse en un rango **interactivo** (objetivo de diseño: segundos, no minutos), sujeto al proveedor. | Should |
| RNF-09 | En modo local, se aceptará mayor latencia; se **medirá y documentará** en el hardware de referencia (CPU / GPU, RAM, tamaño del modelo). | Must |
| RNF-10 | El índice y el modelo local deberán caber, para el corpus del prototipo, en el hardware disponible; si no caben, eso es un **resultado de factibilidad**, no un fallo silencioso. | Must |
| RNF-11 | Se registrará costo estimado por consulta en modo nube (tokens / precio) para el análisis de A04. | Should |

### 6.4 Disponibilidad y operación

| ID | Requerimiento | Prioridad |
| --- | --- | --- |
| RNF-12 | Modo local operable **offline** tras la instalación inicial (RF-19). | Should |
| RNF-13 | Fallos del proveedor nube no deberán corromper el índice; el usuario podrá conmutar a modo local si está configurado. | Could |
| RNF-14 | La ingesta de un documento fallido (PDF dañado, archivo vacío) se reportará con error claro, sin dejar el índice inconsistente a ciegas. | Should |

### 6.5 Usabilidad

| ID | Requerimiento | Prioridad |
| --- | --- | --- |
| RNF-15 | La interfaz de chat será usable por alguien que **no conoce RAG ni embeddings**; no exigirá armar queries booleanas. | Must |
| RNF-16 | La respuesta mostrará citas de forma visible (enlaces o referencias clicables / copiables). | Must |
| RNF-17 | El modo activo (local vs. nube) estará **siempre visible**, para no confundir privacidad y calidad. | Should |

### 6.6 Mantenibilidad y comparabilidad (clave en un estudio de factibilidad)

| ID | Requerimiento | Prioridad |
| --- | --- | --- |
| RNF-18 | La arquitectura permitirá **intercambiar** LLM, embedder y store vectorial con cambios localizados (no un monolito). | Must |
| RNF-19 | Las decisiones de diseño y los resultados de A03–A04 quedarán documentados (modelo, contexto, chunking, top-k, etc.). | Must |
| RNF-20 | El prototipo se versionará en este repositorio con instrucciones de corrida reproducibles. | Should |

### 6.7 Seguridad (recorte realista)

| ID | Requerimiento | Prioridad |
| --- | --- | --- |
| RNF-21 | Autenticación de usuarios, multi-tenant y cifrado en reposo: **Won't** para el prototipo, salvo que el caso de uso de A05 lo exija. | Won't |
| RNF-22 | No se expondrá de forma pública un endpoint con documentos sensibles ni claves de API en el repositorio. | Must |

---

## 7. Fuera de alcance del prototipo

Para no diluir el estudio de factibilidad:

- Asistente generalista no anclado a un corpus (ChatGPT “suelto”).
- Generación o modificación de la documentación (el sistema **interpreta**, no redacta el manual).
- Control directo de equipos industriales o ejecución automática de procedimientos (solo asistencia a la lectura).
- Entrenamiento / fine-tuning pesado de un LLM propio, salvo un experimento acotado que A03 justifique.
- App móvil nativa, integración con tickets/ERP, o portal corporativo completo.
- Garantía de corrección absoluta: se evalúa factibilidad y límites, no certificación de seguridad funcional.

---

## 8. Criterios que A08 deberá poder evaluar

Estos no son “features”, pero A01 debe dejarlos planteados porque la metodología de la propuesta los exige.

| Dimensión | Pregunta de factibilidad | Evidencia esperada |
| --- | --- | --- |
| Precisión | ¿Las respuestas son correctas respecto del documento? | Golden set + juicio humano |
| Utilidad percibida | ¿Ahorra tiempo frente a buscar en el PDF? | Encuesta / protocolo breve con usuarios reales o simulados |
| Limitaciones | ¿Dónde falla (tablas, código, versiones, PDFs mal extraídos, contexto largo)? | Catálogo de fallos |
| Local vs. nube | ¿Calidad, latencia, costo, privacidad, hardware? | Misma batería de preguntas en ambos modos (A04 + A08) |
| Heterogeneidad | ¿El mismo pipeline sirve para más de un tipo de documento? | Al menos el corpus de A05; si hay tiempo, un segundo documento distinto |

---

## 9. Priorización sugerida para el recorte del prototipo (MVP)

**Incluir sí o sí**

1. Chat de texto + RAG con citas (RF-01 a RF-04, RF-09 a RF-12, RF-21).
2. Dos backends de inferencia comparables (RF-16 a RF-18).
3. Registro de corridas y golden set (RF-25, RF-27, RNF-01 a RNF-03, RNF-09, RNF-18).

**Incluir si A05 o el tiempo lo justifican**

- Contexto conversacional, aclaraciones, reindexación por versión (RF-05, RF-06, RF-13).
- Modo offline estricto y panel de métricas (RF-19, RF-20).

**Dejar para el sistema objetivo / trabajo futuro**

- Voz e imagen (RF-22, RF-23), salvo que el caso de uso lo vuelva central.
- Auth empresarial, OCR de escaneos, integración con otras herramientas.

---

## 10. Cómo seguir A01 (y qué no cerrar todavía)

A01 no requiere congelar tecnologías (eso es A03). En las próximas iteraciones de esta sección conviene:

1. Elegir o acotar el **perfil de usuario** (técnico de mantenimiento, desarrollador que consume una API, etc.), aunque el corpus formal sea A05.
2. Convertir los RF/RNF en **historias o casos de uso detallados** (precondiciones, flujo, postcondiciones).
3. Definir **atributos de calidad medibles** una vez conocido el hardware y el tamaño del corpus.
4. Revisar prioridades de multimodalidad: texto basta para factibilidad RAG; voz/imagen solo si el escenario de uso lo exige.
5. Mantener trazabilidad: cada RF/RNF debería poder mapearse luego a un componente de A06 y a una prueba de A08.

**Dependencias:** A02 puede agregar requerimientos (p. ej. “human-in-the-loop”, evaluación RAGAS, *hybrid search*). A04 puede recortar RF-16/RF-19 si el hardware local resulta inviable: esa inviabilidad **es un hallazgo válido** del proyecto, no un incumplimiento.

---

## 11. Resumen ejecutivo de posibles requerimientos

Si hay que comunicar A01 en una página:

El sistema debe permitir **preguntar en lenguaje natural** sobre un corpus técnico cargado, **recuperar fragmentos**, **responder con citas** y **admitir que no sabe** cuando la fuente no alcanza. Debe poder correr con **LLM local y LLM en la nube** sobre la misma tubería RAG, para comparar calidad, costo, privacidad, latencia y hardware. La interfaz mínima es **chat de texto**; voz e imagen son deseables. Lo no funcional crítico es **anclaje a la fuente** (poca alucinación), **privacidad en modo local**, **arquitectura intercambiable** y **medición reproducible**. El prototipo no es un producto corporativo: es el vehículo para argumentar factibilidad y límites.
