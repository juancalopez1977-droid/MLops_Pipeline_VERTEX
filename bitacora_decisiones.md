# Bitácora de Decisiones — Proyecto Final

## Registro de decisiones técnicas y de negocio

> Documento que registra las decisiones importantes tomadas durante el
> desarrollo del Proyecto Final. Sigue un formato inspirado en los
> Architecture Decision Records (ADR), simplificado para el contexto
> académico. Es responsabilidad compartida del equipo mantener este
> registro actualizado cada vez que se toma una decisión relevante.

---

## Formato de cada decisión

Cada decisión se documenta con:

- **Identificador:** D-NN secuencial.
- **Título:** descripción corta de la decisión.
- **Fecha:** cuándo se tomó.
- **Estado:** Propuesta, Aceptada, Reemplazada, Deprecada.
- **Contexto:** la situación que motivó la decisión.
- **Decisión:** qué se decidió.
- **Alternativas consideradas:** qué otras opciones se evaluaron.
- **Consecuencias:** implicaciones positivas y negativas.
- **Responsable:** quién tomó o lideró la decisión.

---

## Plantilla para nuevas decisiones

```markdown
### D-NN — [Título corto de la decisión]

**Fecha:** [YYYY-MM-DD]
**Estado:** [Propuesta / Aceptada / Reemplazada / Deprecada]
**Responsable:** [Rol o nombre]

**Contexto:**
[Descripción de la situación que motiva la decisión.]

**Decisión:**
[Qué se decidió, en términos claros.]

**Alternativas consideradas:**
- [Alternativa 1] — [breve razón por la que no se eligió]
- [Alternativa 2] — [breve razón por la que no se eligió]

**Consecuencias:**
- Positivas: [lista]
- Negativas o trade-offs: [lista]

**Etapa asociada:** [Número de etapa o "transversal"]
```

---

## Decisiones registradas

### D-01 — Selección del dataset Olist

**Fecha:** Pre-PF
**Estado:** Aceptada
**Responsable:** Equipo (consenso)

**Contexto:**
El equipo necesitaba elegir un dataset que cumpliera los criterios de
los lineamientos del Proyecto Final: pertenecer a la industria de
comercio y consumo, permitir modelar interacción entre usuarios y
productos, y habilitar el cálculo de métricas como Precision@K y
Recall@K. Inicialmente se consideró el dataset Rossmann Store Sales,
pero se descartó por ser un problema de forecasting de ventas
agregadas que no permite modelar interacción usuario-producto.

**Decisión:**
Se adopta el **Brazilian E-commerce Public Dataset by Olist** (Kaggle)
como dataset principal del proyecto.

**Alternativas consideradas:**
- Rossmann Store Sales — descartado por incompatibilidad estructural
  con el problema requerido.
- Online Retail II (UCI) — opción viable pero con menor riqueza de
  variables auxiliares y sin ratings explícitos.
- H&M Personalized Fashion Recommendations — descartado por su
  volumen (31 millones de transacciones) excesivo para tres sprints.
- RetailRocket — descartado por anonimización extrema de items que
  limita interpretabilidad de negocio.

**Consecuencias:**
- Positivas:
  - Estructura nativa usuario-item-interacción.
  - Disponibilidad de ratings explícitos vía reseñas.
  - Riqueza de variables auxiliares (geografía, categorías, vendedores).
  - Tamaño manejable para tres sprints.
- Negativas o trade-offs:
  - Alta sparsity de la matriz cliente-producto (la mayoría de
    clientes hace una sola compra).
  - Reseñas en portugués (requiere traducción o manejo especial si se
    incorpora NLP).

**Etapa asociada:** 0

---

### D-02 — Tipo de sistema de recomendación

**Fecha:** Pre-PF
**Estado:** Aceptada
**Responsable:** Data Scientist + Product Owner

**Contexto:**
Definida la elección de Olist, se hace evidente que la alta sparsity
de la matriz cliente-producto hace inviable un sistema de
recomendación user-to-item tradicional. El equipo necesita definir
qué tipo específico de sistema de recomendación construir.

**Decisión:**
Se adopta un **sistema híbrido item-to-item con dos canales
complementarios**: productos similares (content-based filtering) y
productos comprados juntos (item-based collaborative filtering).

**Alternativas consideradas:**
- User-to-item collaborative filtering — descartado por la sparsity
  de la matriz cliente-producto.
- Recomendador basado solo en contenido — descartado por aprovechar
  pobremente la información de co-ocurrencias entre productos.
- Recomendador basado solo en colaborativo — descartado por no
  resolver bien el cold-start de productos nuevos.
- Recomendador basado en sesión — descartado por requerir datos de
  navegación que Olist no provee.

**Consecuencias:**
- Positivas:
  - Aprovecha tanto la información de catálogo como las
    co-ocurrencias.
  - Mapea a casos de uso reales del e-commerce (PDP y carrito).
  - Permite comparación rica entre enfoques en la evaluación.
- Negativas o trade-offs:
  - Mayor complejidad de implementación al manejar dos canales.
  - Necesita doble esfuerzo de evaluación.

**Etapa asociada:** 0

---

### D-03 — Indicadores clave de desempeño del proyecto

**Fecha:** Pre-PF
**Estado:** Aceptada
**Responsable:** Product Owner

**Contexto:**
La consigna del PF requiere definir KPIs de negocio asociados al
sistema de recomendación. Es necesario separar conceptualmente los
KPIs de negocio (los que se medirían en producción) de las métricas
técnicas (las que se calculan offline durante la evaluación).

**Decisión:**
Se adoptan dos KPIs de negocio:
- KPI principal: **Click-Through Rate (CTR)** sobre los productos
  recomendados.
- KPI secundario: **Ticket promedio por orden (Average Order Value,
  AOV)** en las órdenes que incluyeron al menos un producto
  recomendado.

Las métricas técnicas de evaluación offline son: Precision@K,
Recall@K, MAP@K, cobertura del catálogo y diversidad.

**Alternativas consideradas:**
- Tasa de conversión asistida como KPI principal — descartada por
  ser más difícil de definir operacionalmente en Olist.
- Retención del cliente — descartada por requerir periodo de
  observación largo, no medible en el alcance del proyecto.

**Consecuencias:**
- Positivas:
  - KPIs universales del e-commerce, fáciles de comunicar a
    stakeholders.
  - Separación clara entre lo que se reportaría en producción y lo
    que se evalúa offline.
- Negativas o trade-offs:
  - CTR no se puede medir directamente con datos históricos de Olist
    (no hay logs de clicks). Se evalúa mediante aproximaciones offline.

**Etapa asociada:** 0

---

### D-04 — Composición de roles del equipo

**Fecha:** Pre-PF
**Estado:** Aceptada
**Responsable:** Equipo (consenso)

**Contexto:**
La plantilla oficial del PF de Henry sugiere una composición de
cuatro integrantes (1 Product Owner, 2 Data Scientists, 1 Scrum
Master). El equipo está conformado por cinco personas. Es necesario
definir el quinto rol y la distribución de responsabilidades de
manera coherente con un equipo de Machine Learning moderno.

**Decisión:**
Se adopta una composición de cinco roles: **Product Owner, Scrum
Master, Data Analyst, Data Scientist, Machine Learning Engineer**.
Esta composición refleja un equipo de ML real moderno con
especialización clara: el Data Analyst lidera el entendimiento de
datos y EDA, el Data Scientist lidera el modelado, y el ML Engineer
lidera el despliegue y monitoreo.

**Alternativas consideradas:**
- Mantener dos Data Scientists y agregar un quinto Product Owner —
  descartado por crear redundancia en el liderazgo.
- Reemplazar Data Analyst por Data Engineer — opción válida con
  diferencias menores; se prefirió Data Analyst por el peso del EDA
  en Power BI en el proyecto.

**Consecuencias:**
- Positivas:
  - Refleja un equipo de ML real con responsabilidades claras.
  - Cada rol tiene una etapa de carga alta diferenciada.
  - Facilita el aprendizaje individual de cada miembro en su área.
- Negativas o trade-offs:
  - Requiere validar con el mentor que la plantilla acepta cinco
    roles.

**Etapa asociada:** Transversal

---

### D-05 — Stack tecnológico principal

**Fecha:** Pre-PF
**Estado:** Aceptada
**Responsable:** ML Engineer + Data Scientist

**Contexto:**
El equipo necesita definir las tecnologías principales que se
utilizarán durante el proyecto, considerando la disponibilidad de
habilidades en el equipo, la robustez de las herramientas y la
compatibilidad entre los componentes.

**Decisión:**
Se adoptan las siguientes tecnologías como stack principal:

- **Lenguaje:** Python 3.10+
- **Manipulación de datos:** pandas, numpy
- **Modelado:** scikit-learn
- **API:** FastAPI
- **Validación de datos en API:** Pydantic
- **Contenedorización:** Docker
- **Dashboard:** Streamlit
- **Visualización exploratoria:** Power BI (informe del Sprint 1) y
  Matplotlib/Seaborn (notebooks).
- **Serialización de modelos:** joblib
- **Notebooks:** Jupyter

**Alternativas consideradas:**
- Flask en lugar de FastAPI — descartado por menor adopción moderna
  y por no tener validación nativa con Pydantic.
- Dash o Plotly Dash en lugar de Streamlit — descartado por curva de
  aprendizaje mayor.
- Tableau en lugar de Power BI — descartado porque el lineamiento de
  Henry sugiere Power BI explícitamente.

**Consecuencias:**
- Positivas:
  - Stack ampliamente documentado y soportado.
  - Compatibilidad total entre componentes.
  - Skills relativamente comunes en perfiles de Data Science.
- Negativas o trade-offs:
  - Streamlit tiene limitaciones para dashboards muy complejos; será
    suficiente para el alcance del proyecto.

**Etapa asociada:** Transversal

---

### D-06 — Estructura del repositorio y estrategia de ramas

**Fecha:** Pre-PF
**Estado:** Aceptada
**Responsable:** ML Engineer + Scrum Master

**Contexto:**
El equipo necesita una estrategia de ramas que facilite el trabajo
colaborativo de cinco personas, garantice la revisión por pares y
mantenga la trazabilidad de las versiones.

**Decisión:**
Se adopta una estructura de **tres ramas largas** (`master`,
`certification`, `developer`) complementada con **ramas de feature**
para el trabajo diario. Cada cambio entra a `developer` vía pull
request con al menos un revisor distinto al autor. Las versiones se
promueven secuencialmente de `developer` a `certification` y de
`certification` a `master`.

**Alternativas consideradas:**
- GitFlow completo (con ramas release, hotfix, etc.) — descartado
  por ser excesivo para un proyecto de tres semanas.
- Trunk-based development — descartado por no garantizar la revisión
  por pares explícita.
- Una sola rama main con tags — descartado por no diferenciar
  estados de madurez del código.

**Consecuencias:**
- Positivas:
  - Trazabilidad clara entre desarrollo, certificación y producción.
  - Revisión por pares obligatoria.
  - Coherente con la guía de uso de GitHub provista por Henry.
- Negativas o trade-offs:
  - Mayor número de merges al cerrar cada etapa.

**Etapa asociada:** Transversal

---

### D-07 — Esquema de versionado semántico

**Fecha:** Pre-PF
**Estado:** Aceptada
**Responsable:** ML Engineer

**Contexto:**
El equipo necesita una convención de versionado clara que permita
identificar el estado del proyecto en cada momento y facilite la
trazabilidad entre etapas y entregables.

**Decisión:**
Se adopta **versionado semántico (SemVer)** con el patrón
`MAJOR.MINOR.PATCH`. El proyecto inicia en `V1.0.0` al cierre de la
Etapa 1 y avanza incrementalmente con cada etapa:

- Etapa 1: V1.0.0
- Etapa 2: V1.1.0
- Etapa 3: V1.2.0
- Etapa 4: V1.3.0
- Etapa 5: V1.4.0
- Etapa 6: V1.5.0
- Etapa 7: V1.6.0
- Etapa 8: V1.7.0
- Etapa 9: V1.8.0

Los tags se crean siempre desde `master` después de promover el
código a través de las tres ramas.

**Alternativas consideradas:**
- Versionado por fecha (CalVer) — descartado por menor estándar en
  proyectos de ML.
- Sin versionado formal — descartado por incompatible con la
  trazabilidad requerida.

**Consecuencias:**
- Positivas:
  - Estándar conocido y compatible con prácticas de la industria.
  - Trazabilidad clara entre etapas y artefactos generados.

**Etapa asociada:** Transversal

---

### D-08 — Gestión de Sprint Backlogs en GitHub Projects

**Fecha:** Pre-PF
**Estado:** Aceptada
**Responsable:** Scrum Master

**Contexto:**
El equipo necesita una herramienta para gestionar los Sprint Backlogs
y el tablero de trabajo durante el proyecto. La opción de mantener
estos artefactos como archivos markdown fue evaluada pero descartada
por baja agilidad de actualización.

**Decisión:**
Se adopta **GitHub Projects** como herramienta única para la gestión
del tablero de trabajo y los Sprint Backlogs. Las columnas estándar
del tablero son: To Do, In Progress, In Review, Done.

**Alternativas consideradas:**
- Jira — descartado por mayor curva de aprendizaje y por estar
  desacoplado del repositorio.
- Trello — descartado por ofrecer menos integración con GitHub.
- Archivos markdown en el repositorio — descartado por baja
  agilidad y tendencia a desactualizarse.

**Consecuencias:**
- Positivas:
  - Integración nativa con issues y pull requests.
  - Trazabilidad automática entre tareas y código.
  - Sin costo adicional.
- Negativas o trade-offs:
  - Será la primera vez que el Scrum Master gestiona un proyecto en
    GitHub Projects; requiere preparación previa al inicio del
    Sprint 1.

**Etapa asociada:** 1

---

### D-09 — Estrategia de evaluación offline mediante partición temporal

**Fecha:** Pre-PF
**Estado:** Propuesta (a confirmar en Etapa 6)
**Responsable:** Data Scientist

**Contexto:**
Para evaluar los modelos de recomendación offline, es necesario
definir cómo se divide el dataset entre entrenamiento y validación,
de modo que la evaluación sea realista y libre de leakage.

**Decisión:**
Se propone aplicar **partición temporal del dataset** (time-based
split): el modelo se entrena con datos del periodo anterior y se
evalúa con datos del periodo posterior. Las fechas exactas de corte
se definirán al inicio de la Etapa 3 según los hallazgos del EDA.

**Alternativas consideradas:**
- Random split — descartado por introducir leakage temporal
  (entrenamiento ve futuro).
- Leave-one-out — descartado por costo computacional excesivo.
- Cross-validation con folds temporales — descartado por
  sobrecomplejidad para el alcance del proyecto.

**Consecuencias:**
- Positivas:
  - Realista respecto a un escenario productivo.
  - Sin leakage temporal.
- Negativas o trade-offs:
  - Una sola pasada de evaluación; menos robusto que cross-validation.
  - Decisión de fecha de corte sensible a la distribución temporal de
    los datos.

**Etapa asociada:** 3 (confirmación en Etapa 6)

---

### D-10 — Mantener documentos de gestión del proyecto fuera del repositorio público

**Fecha:** Pre-PF
**Estado:** Aceptada
**Responsable:** Product Owner

**Contexto:**
El equipo dispone de varios documentos de planificación y gestión
(guía maestra, propuesta metodológica, instrucciones de Scrum,
instrucciones de flujo, guías por etapa, contextos previos) que sirven
como referencia personal del líder del proyecto y como contexto para
chats individuales con asistentes de inteligencia artificial. Es
necesario decidir si estos documentos se versionan en el repositorio
del proyecto o se mantienen aparte.

**Decisión:**
Los documentos de planificación, gestión y orientación se mantienen
**fuera del repositorio público de GitHub**. Solo los documentos
operativos del proyecto se versionan en el repositorio:
`product_backlog.md`, `bitacora_decisiones.md`, `registro_riesgos.md`,
y las actas de Sprint Review y Retrospective.

**Alternativas consideradas:**
- Versionar todos los documentos en una carpeta `docs/` del
  repositorio — descartado para mantener limpio el repositorio
  público.

**Consecuencias:**
- Positivas:
  - Repositorio enfocado en artefactos del producto.
  - Mayor flexibilidad para actualizar guías personales.
- Negativas o trade-offs:
  - El líder del proyecto debe organizar manualmente sus documentos
    personales y cargarlos a las herramientas de soporte
    (proyecto Claude, drive personal).

**Etapa asociada:** Transversal

---

### D-11 — Identidad corporativa del equipo

**Fecha:** Etapa 0
**Estado:** Aceptada
**Responsable:** Equipo (consenso)

**Contexto:**
La consigna del Proyecto Final solicita que el equipo opere como una
consultora ficticia con identidad propia. Era necesario definir el
nombre corporativo, el correo institucional y la misión del equipo,
elementos requeridos por la plantilla oficial de la propuesta.

**Decisión:**
Se adopta la siguiente identidad corporativa:

- **Nombre de la empresa:** Vertex Insights.
- **Correo institucional:** equipo.vertexinsight@gmail.com.
- **Misión:** "Vertex Insights es una consultora de datos que
  convierte información compleja en decisiones de negocio claras.
  Combinamos rigor analítico con un enfoque pragmático para entregar
  soluciones que impacten directamente la gestión de nuestros
  clientes."

Esta identidad se utilizará en todos los entregables y comunicaciones
formales del proyecto.

**Alternativas consideradas:**
- Misión con énfasis exclusivo en el caso de uso del PF — descartada
  por restringir innecesariamente la marca a un solo proyecto.
- Correo institucional en dominio propio — descartado por costo y
  por no aportar valor adicional en el contexto académico.

**Consecuencias:**
- Positivas:
  - Identidad reutilizable en todos los entregables y presentaciones.
  - Misión suficientemente amplia para sustentar la narrativa
    profesional del equipo.
- Negativas o trade-offs:
  - El correo institucional usa Gmail (dominio público); aceptable
    para el contexto académico.
  - Inconsistencia menor entre el nombre plural ("Vertex Insights") y
    el correo singular ("vertexinsight"); se mantiene tal como quedó
    registrada la cuenta.

**Etapa asociada:** 0

---

### D-12 — Aprobación de la propuesta del Proyecto Final

**Fecha:** Etapa 0
**Estado:** Aceptada
**Responsable:** Product Owner

**Contexto:**
La Etapa 0 requiere la aprobación formal de la propuesta del Proyecto
Final por parte del mentor o comité evaluador como condición para
iniciar el Sprint 1. La propuesta consolida la identidad del equipo,
el caso de negocio, las fuentes de datos y el plan de estrategia de
análisis.

**Decisión:**
La propuesta del Proyecto Final, en su versión consolidada con
identidad del equipo Vertex Insights, caso de negocio sobre Olist,
sistema de recomendación item-to-item y plan de despliegue con
FastAPI, Streamlit y Docker, se considera aprobada y vigente. La
aprobación habilita formalmente el inicio del Sprint 1 y confirma la
validez de las decisiones D-01 a D-05 y D-11 en el ámbito del
proyecto.

**Alternativas consideradas:**
No aplica; la decisión consolida los acuerdos previos del equipo y la
validación externa del mentor.

**Consecuencias:**
- Positivas:
  - Cierre formal de la Etapa 0.
  - Habilitación del inicio del Sprint 1.
  - Confirmación del alcance técnico y de negocio.
- Negativas o trade-offs:
  - Ninguno identificado.

**Etapa asociada:** 0

---

*Bitácora de decisiones del Proyecto Final. Las decisiones registradas
hasta esta versión corresponden a la fase de planificación y al cierre
de la Etapa 0. Nuevas decisiones se agregarán durante la ejecución del
proyecto.*
