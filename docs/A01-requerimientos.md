# A01 — Determinación de requerimientos del sistema

**Proyecto:** Factibilidad de un Asistente Conversacional Inteligente para la Interpretación de Documentación Técnica  
**Estado:** Recorte vigente, tomado de la selección en [docs user](../docs%20user/Requerimientos_del_sistema_1e25.pdf)  
**Versión:** 0.2

El listado de este documento contiene **únicamente** los requerimientos funcionales, no funcionales y criterios de evaluación seleccionados en `docs user/Requerimientos_del_sistema_1e25.pdf`. El catálogo preliminar (versión 0.1) queda reemplazado: lo que no figura aquí no es un requerimiento del sistema.

Los identificadores que se conservan (RF-01, RF-02, etc.) son los del catálogo preliminar, para no romper la trazabilidad con [A02](A02-estado-del-arte.md). Los números que no aparecen quedaron fuera de la selección. RF-28 y RNF-23 no existían en 0.1: son ítems de la selección que no tenían identificador previo.

Prioridad, como en la selección: **Alta**, **Media**, **Baja**.

---

## 1. Propósito y alcance

El sistema objeto de estudio es un **asistente conversacional** que permite consultar documentación técnica en lenguaje natural y obtiene respuestas **ancladas** en esa documentación, en lugar de depender solo del conocimiento general del modelo de lenguaje.

El trabajo es un **estudio de factibilidad** con prototipo, no un producto de producción. El alcance de requerimientos es el de la selección:

| Prioridad | Qué implica en el prototipo |
| --- | --- |
| Alta | Imprescindible para demostrar el enfoque |
| Media | Importante si el tiempo de A07 lo permite |
| Baja | Deseable, no condiciona la factibilidad |

**Caso de uso aún no fijado (A05).** Los requerimientos se formulan de manera independiente del dominio (manuales de equipos, documentación de lenguajes/APIs, guías de configuración, etc.). Al elegir el caso de uso, algunos ítems se especializarán (volumen, idioma, criticidad de un error). El formato de entrada exigido por la selección es **PDF**.

---

## 2. Stakeholders

| Actor | Interés |
| --- | --- |
| Usuario consultante | Resolver una duda puntual sin leer el documento completo |
| Autor / dueño de la documentación | Que las respuestas respeten el contenido vigente |
| Operador del prototipo (alumno / director) | Cargar la documentación y registrar la validación |
| Evaluadores académicos | Poder juzgar factibilidad: precisión, utilidad y limitaciones |

---

## 3. Premisas y restricciones (RC)

Estas restricciones vienen de la propuesta de proyecto. No amplían el listado de requerimientos.

| ID | Restricción |
| --- | --- |
| RC-01 | Recursos computacionales limitados (equipo personal / entorno académico; no se asume un cluster). |
| RC-02 | La propuesta pide comparar, en A04, un modelo local y un servicio en la nube. El requerimiento seleccionado es más acotado: la arquitectura debe permitir **intercambiar LLMs de manera modular** (RNF-18). Implementar ambos modos no es, por sí, un requerimiento del prototipo. |
| RC-03 | La documentación de entrada es heterogénea y no está necesariamente estructurada para búsqueda. El requerimiento de ingesta seleccionado exige al menos **PDF**. |
| RC-04 | Un error en la respuesta puede tener consecuencias relevantes en el dominio real. Por eso la selección exige citas, abstención ante evidencia insuficiente o contradictoria, trazabilidad y minimizar alucinaciones. |
| RC-05 | Plazo y esfuerzo de un Proyecto Final: priorizar un recorte vertical demostrable (un caso de uso, un corpus acotado) frente a cobertura amplia. |
| RC-06 | El idioma de la interfaz y de las respuestas será, en el prototipo, **español y/o inglés**, según el corpus elegido en A05. |

---

## 4. Casos de uso de alto nivel

Solo los que se desprenden de los requerimientos seleccionados.

```text
CU-01  Consultar la documentación en lenguaje natural
CU-02  Recibir la respuesta anclada en la documentación, con citas
CU-03  Que el sistema declare evidencia insuficiente o contradictoria
CU-04  Incorporar documentación técnica en PDF
CU-05  Hacer una pregunta de seguimiento (contexto de conversación)
CU-06  Recibir un pedido de aclaración ante una consulta ambigua
CU-07  Adjuntar una imagen para contextualizar la pregunta
```

---

## 5. Requerimientos funcionales (RF)

### 5.1 Consulta e interfaz

| ID | Requerimiento | Prioridad |
| --- | --- | --- |
| RF-01 | El usuario podrá realizar consultas en lenguaje natural sobre la documentación cargada, sin necesariamente tener conocimiento detallado de la misma. | Alta |
| RF-21 | Interfaz de consulta y respuesta por texto (chat). | Alta |
| RF-02 | El sistema generará una respuesta basándose en la documentación, y no solo en sus propios conocimientos y lógica. | Alta |
| RF-03 | Cada respuesta incluirá citas al origen de la información: documento, ubicación (sección, página, etc.) y/o un extracto. | Alta |
| RF-04 | El sistema debe declarar cuando la evidencia recuperada es insuficiente o contradictoria. | Alta |
| RF-05 | El sistema mantendrá contexto de conversación para preguntas de seguimiento. | Media |
| RF-06 | Ante consultas ambiguas, el sistema podrá requerir una aclaración (producto, versión, sección, síntoma) antes de responder. | Media |

### 5.2 Documentación de entrada

| ID | Requerimiento | Prioridad |
| --- | --- | --- |
| RF-09 | Permitirá incorporar documentación técnica del caso de uso en al menos PDF. | Alta |

### 5.3 Imagen adjunta

La selección incluye las dos formulaciones siguientes, con prioridades distintas.

| ID | Requerimiento | Prioridad |
| --- | --- | --- |
| RF-23 | El usuario podrá adjuntar una imagen (captura de error, foto de una placa, recorte de un manual) para contextualizar la pregunta. | Media |
| RF-28 | El usuario podrá adjuntar una imagen (captura de error, foto de una placa/etiqueta, recorte de un manual) para contextualizar la pregunta. | Baja |

---

## 6. Requerimientos no funcionales (RNF)

| ID | Requerimiento | Prioridad |
| --- | --- | --- |
| RNF-01 | Las afirmaciones realizadas por el sistema deben ser trazables a la documentación. | Alta |
| RNF-02 | Se buscará minimizar alucinaciones. | Alta |
| RNF-23 | El sistema debe poder reconocer cuando no es capaz de responder una consulta. | Alta |
| RNF-08 | El tiempo de respuesta debe mantenerse en un rango aceptable (segundos, no minutos). | Alta |
| RNF-18 | La arquitectura permitirá intercambiar LLMs de manera modular. | Alta |

Umbrales numéricos de precisión se fijarán después de A05 y de un piloto sobre el corpus real.

---

## 7. Fuera de alcance del prototipo

Para no diluir el estudio de factibilidad:

- Asistente generalista no anclado a un corpus.
- Generación o modificación de la documentación (el sistema **interpreta**, no redacta el manual).
- Control directo de equipos industriales o ejecución automática de procedimientos (solo asistencia a la lectura).
- Entrenamiento / fine-tuning pesado de un LLM propio, salvo un experimento acotado que A03 justifique.
- App móvil nativa, integración con tickets/ERP, o portal corporativo completo.
- Garantía de corrección absoluta: se evalúa factibilidad y límites, no certificación de seguridad funcional.

Quedó fuera de este listado todo ítem del catálogo preliminar que la selección no incluyó (entre otros: voz, filtros por subconjunto del corpus, OCR, modo local y modo nube como requisitos del prototipo, registro de corridas y conjunto dorado).

---

## 8. Criterios a evaluar

Son los tres criterios de la selección. A08 deberá poder observarlos.

| Criterio | Pregunta |
| --- | --- |
| Precisión | Qué tan correctas son las respuestas. |
| Utilidad | Si el sistema es conveniente comparado con acceder a la documentación directamente. |
| Limitaciones | Formatos aceptables de documentación, latencia, tamaño de contexto, etc. |

---

## 9. Priorización del recorte

**Alta**

- Consulta en lenguaje natural, chat de texto, respuesta anclada en la documentación, citas y declaración de evidencia insuficiente o contradictoria (RF-01, RF-21, RF-02, RF-03, RF-04).
- Ingesta de documentación en PDF (RF-09).
- Trazabilidad, minimizar alucinaciones, reconocer que no se puede responder, latencia en segundos y arquitectura con LLMs intercambiables (RNF-01, RNF-02, RNF-23, RNF-08, RNF-18).

**Media**

- Contexto de conversación y pedido de aclaración (RF-05, RF-06).
- Imagen adjunta en la formulación de RF-23.

**Baja**

- Imagen adjunta en la formulación de RF-28.

---

## 10. Cómo seguir A01

A01 no congela tecnologías (eso es A03). En las próximas iteraciones conviene:

1. Elegir o acotar el **perfil de usuario** (técnico de mantenimiento, desarrollador que consume una API, etc.), aunque el corpus formal sea A05.
2. Convertir los RF/RNF de este listado en **historias o casos de uso detallados** (precondiciones, flujo, postcondiciones).
3. Definir **atributos de calidad medibles** una vez conocido el hardware y el tamaño del corpus.
4. Resolver el solapamiento entre RF-23 (Media) y RF-28 (Baja): la selección los trae los dos, con una diferencia de redacción (“placa” frente a “placa/etiqueta”).
5. Mantener trazabilidad: cada RF/RNF de este listado debería poder mapearse luego a un componente de A06 y a una prueba de A08.

**Dependencias:** [A02, sección 10](A02-estado-del-arte.md) propone candidatos (búsqueda híbrida, reescritura de la consulta, fragmentación estructural, preguntas incontestables en el protocolo de prueba, presupuesto de contexto). Esos candidatos **no** forman parte de este listado hasta que se los incorpore en una revisión posterior. A04 sigue siendo el análisis local frente a nube de la propuesta; si el hardware local resulta inviable, esa inviabilidad es un hallazgo del proyecto. El requerimiento vigente asociado es RNF-18, no la obligación de implementar ambos modos.

---

## 11. Resumen ejecutivo

El sistema debe permitir **preguntar en lenguaje natural** sobre la documentación cargada, **responder a partir de esa documentación** (no solo del conocimiento del modelo), **citar el origen** y **declarar** cuando la evidencia es insuficiente o contradictoria. Debe aceptar documentación en **PDF** y ofrecer la consulta por **chat de texto**. En prioridad media quedan el **contexto de conversación**, el **pedido de aclaración** y **adjuntar una imagen**; una segunda formulación de la imagen adjunta queda en prioridad baja.

En lo no funcional, las afirmaciones deben ser **trazables**, se busca **minimizar alucinaciones**, el sistema debe **reconocer cuándo no puede responder**, el tiempo de respuesta debe mantenerse en **segundos** y la arquitectura debe permitir **intercambiar LLMs** de forma modular.

La validación (A08) observa tres criterios: **precisión**, **utilidad** frente a leer la documentación directo, y **limitaciones** (formatos, latencia, tamaño de contexto).
