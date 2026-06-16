# Registro de Riesgos — Proyecto Final

## Identificación, evaluación y mitigación de riesgos del proyecto

> Documento que registra los riesgos identificados durante el
> desarrollo del Proyecto Final, junto con su probabilidad, impacto y
> plan de mitigación. La identificación de riesgos es continua: nuevos
> riesgos se agregan a este registro a medida que surgen, y los
> existentes se actualizan en su estado y plan de mitigación.

---

## Convenciones

**Probabilidad:**
- Baja: improbable que ocurra en el horizonte del proyecto.
- Media: posibilidad real de ocurrencia.
- Alta: muy probable que ocurra durante el proyecto.

**Impacto:**
- Bajo: afecta una tarea o entregable menor, recuperable rápidamente.
- Medio: afecta una etapa o varias tareas, requiere reorganización.
- Alto: pone en riesgo entregables clave del proyecto o el
  cumplimiento del plazo final.

**Nivel de riesgo (combinación):**

| Probabilidad / Impacto | Bajo | Medio | Alto |
|---|---|---|---|
| Baja | Bajo | Bajo | Medio |
| Media | Bajo | Medio | Alto |
| Alta | Medio | Alto | Crítico |

**Estado:** Activo, Mitigado, Materializado, Cerrado.

---

## Plantilla para nuevos riesgos

```markdown
### R-NN — [Título del riesgo]

**Categoría:** [Técnico / Datos / Tiempo / Equipo / Comunicación / Negocio]
**Probabilidad:** [Baja / Media / Alta]
**Impacto:** [Bajo / Medio / Alto]
**Nivel de riesgo:** [Bajo / Medio / Alto / Crítico]
**Estado:** [Activo / Mitigado / Materializado / Cerrado]
**Responsable de seguimiento:** [Rol o nombre]

**Descripción:**
[En qué consiste el riesgo y por qué es relevante.]

**Detonantes posibles:**
[Eventos o situaciones que pueden hacer que el riesgo se materialice.]

**Plan de mitigación:**
[Acciones preventivas para reducir la probabilidad o el impacto.]

**Plan de contingencia:**
[Qué hacer si el riesgo efectivamente se materializa.]

**Etapas afectadas:** [Lista de etapas potencialmente impactadas.]
```

---

## Riesgos identificados

### R-01 — Sparsity de la matriz cliente-producto en Olist

**Categoría:** Datos
**Probabilidad:** Alta
**Impacto:** Medio
**Nivel de riesgo:** Alto
**Estado:** Mitigado (con decisión de arquitectura)
**Responsable de seguimiento:** Data Scientist

**Descripción:**
La gran mayoría de los clientes en Olist realiza una sola compra en
el periodo cubierto por el dataset. Esto hace que la matriz
cliente-producto sea extremadamente sparse, lo cual deteriora el
rendimiento de los enfoques de user-based collaborative filtering
clásicos.

**Detonantes posibles:**
- Intentar implementar un recomendador user-to-item ignorando la
  sparsity.
- Diseñar métricas de evaluación que asumen historial de compras
  recurrentes por cliente.

**Plan de mitigación:**
- Se decidió desde la fase de planificación adoptar un enfoque
  item-to-item (referencia: D-02 en la Bitácora de Decisiones), que
  no depende de la densidad de la matriz cliente-producto.
- En la Etapa 2 (EDA), validar empíricamente la sparsity y ajustar
  los enfoques si la distribución cambia respecto a lo esperado.

**Plan de contingencia:**
- Si durante el EDA la sparsity es aún peor de lo esperado, reforzar
  el peso del canal content-based sobre el colaborativo, ya que el
  primero no depende de co-ocurrencias entre clientes.

**Etapas afectadas:** 2, 3, 4, 6

---

### R-02 — Duración corta del proyecto (tres semanas) frente al alcance

**Categoría:** Tiempo
**Probabilidad:** Media
**Impacto:** Alto
**Nivel de riesgo:** Alto
**Estado:** Activo
**Responsable de seguimiento:** Scrum Master + Product Owner

**Descripción:**
El proyecto tiene una duración total de tres semanas calendario,
distribuidas en una fase previa y dos sprints de una semana. El
alcance comprometido (EDA exhaustivo, dos canales de recomendación,
API, Docker, dashboard, monitoreo y documentación completa) es
ambicioso para este plazo, especialmente para un equipo que no ha
trabajado junto previamente.

**Detonantes posibles:**
- Sub-estimación del esfuerzo en alguna etapa.
- Curva de aprendizaje en nuevas tecnologías (FastAPI, Docker,
  Streamlit).
- Bloqueos no resueltos a tiempo.
- Cambios de alcance impulsados por el mentor.

**Plan de mitigación:**
- Mantener el alcance estrictamente acotado al plan: si surge algo
  nuevo, va al backlog para sprints futuros, no se introduce al
  sprint en curso.
- Las ceremonias de Daily Stand-up se ejecutan a primera hora todos
  los días para detectar tempranamente cualquier desviación.
- El Scrum Master vigila el avance contra el plan y escala
  rápidamente cualquier riesgo de tiempo.

**Plan de contingencia:**
- Si el Sprint 1 cierra con tareas incompletas críticas, se replantea
  el alcance del Sprint 2 priorizando lo mínimo viable para
  presentación.
- Como último recurso, el monitoreo (HU-16) es la historia con
  prioridad media y puede aplazarse o simplificarse si fuera
  necesario.

**Etapas afectadas:** Todas

---

### R-03 — Curva de aprendizaje en GitHub Projects

**Categoría:** Equipo
**Probabilidad:** Alta
**Impacto:** Bajo
**Nivel de riesgo:** Medio
**Estado:** Mitigado (con plan formativo)
**Responsable de seguimiento:** Scrum Master

**Descripción:**
El líder del proyecto y posiblemente otros miembros del equipo no han
usado antes GitHub Projects como herramienta de gestión de Sprint
Backlogs. La curva de aprendizaje puede generar fricción inicial.

**Detonantes posibles:**
- Iniciar el Sprint 1 sin haber preparado el tablero de trabajo
  previamente.
- Confusión sobre cómo vincular issues, pull requests y elementos del
  tablero.

**Plan de mitigación:**
- Se generará una guía detallada de configuración y uso de GitHub
  Projects al inicio del Sprint 1, como complemento de la guía de la
  Etapa 1 (referencia: D-08 en la Bitácora de Decisiones).
- El Scrum Master dedicará tiempo del Día 1 del Sprint 1 a configurar
  el tablero y socializarlo al equipo.

**Plan de contingencia:**
- Si la herramienta resulta excesivamente lenta de adoptar, se puede
  optar por un Trello temporal mientras se completa la curva de
  aprendizaje, sin afectar el repositorio.

**Etapas afectadas:** 1, 5

---

### R-04 — Curva de aprendizaje en tecnologías de despliegue (FastAPI, Docker, Streamlit)

**Categoría:** Equipo
**Probabilidad:** Media
**Impacto:** Medio
**Nivel de riesgo:** Medio
**Estado:** Activo
**Responsable de seguimiento:** Machine Learning Engineer

**Descripción:**
Las tecnologías de despliegue del Sprint 2 (FastAPI, Docker,
Streamlit) requieren habilidades específicas que pueden no estar
plenamente desarrolladas en todos los miembros del equipo.

**Detonantes posibles:**
- Subestimar el esfuerzo de configurar Docker desde cero.
- Problemas de compatibilidad entre dependencias.
- Configuración inicial de FastAPI que tome más tiempo del esperado.

**Plan de mitigación:**
- El ML Engineer es el responsable primario y debe llegar al Sprint 2
  con la curva de aprendizaje resuelta.
- En el Sprint 1, el ML Engineer puede comenzar a preparar el
  esqueleto del Dockerfile y la estructura básica de la API,
  aprovechando holguras de carga en esa fase.

**Plan de contingencia:**
- Si Docker presenta dificultades insuperables, el sistema puede
  desplegarse en un entorno virtual local documentado, sacrificando
  reproducibilidad pero manteniendo el resto del entregable.

**Etapas afectadas:** 7, 8

---

### R-05 — Bloqueos entre roles por dependencias secuenciales

**Categoría:** Equipo
**Probabilidad:** Media
**Impacto:** Medio
**Nivel de riesgo:** Medio
**Estado:** Activo
**Responsable de seguimiento:** Scrum Master

**Descripción:**
Las etapas del proyecto tienen dependencias secuenciales (el feature
engineering depende del EDA, el modelado depende del feature
engineering, el despliegue depende del modelado, etc.). Si una etapa
se demora, las siguientes pueden quedar bloqueadas.

**Detonantes posibles:**
- Una etapa se atrasa y no se comunica a tiempo al equipo.
- Algún rol queda esperando entregables sin actividades alternativas.
- Bloqueo técnico no comunicado.

**Plan de mitigación:**
- Las Daily Stand-ups exponen rápidamente los bloqueos.
- Cada rol tiene tareas paralelizables: por ejemplo, mientras el Data
  Analyst termina el EDA, el Data Scientist puede revisar
  documentación de scikit-learn pipelines; mientras el Data Scientist
  modela, el ML Engineer prepara el esqueleto de la API.
- El Scrum Master gestiona la redistribución de cargas cuando
  detecta un cuello de botella.

**Plan de contingencia:**
- Si una dependencia crítica se atrasa, el equipo se reúne para
  redistribuir tareas y dar apoyo al rol bloqueado, en lugar de
  esperar.

**Etapas afectadas:** Todas

---

### R-06 — Cambios de alcance impulsados por el mentor

**Categoría:** Negocio
**Probabilidad:** Media
**Impacto:** Medio
**Nivel de riesgo:** Medio
**Estado:** Activo
**Responsable de seguimiento:** Product Owner

**Descripción:**
Durante las revisiones de sprint o las interacciones con el mentor,
puede surgir feedback que sugiera cambios de alcance, nuevas
funcionalidades, o reorientación de algunos entregables.

**Detonantes posibles:**
- Feedback del mentor en la primera Sprint Review.
- Cambio en los criterios de evaluación del comité.
- Sugerencias específicas de profundización en alguna área.

**Plan de mitigación:**
- El Product Owner es el filtro único de las comunicaciones con el
  mentor.
- Cualquier sugerencia de cambio se evalúa contra el plan original
  antes de comprometerse.
- Si el cambio agrega valor sustancial, se documenta en la Bitácora
  de Decisiones y se ajusta el backlog.

**Plan de contingencia:**
- Para cambios pequeños que no comprometan el plan: aceptar e
  incorporar.
- Para cambios grandes: discutir con el mentor su prioridad real y
  buscar alcanzar el mínimo viable sin sacrificar entregables ya
  comprometidos.

**Etapas afectadas:** Todas

---

### R-07 — Calidad de los datos en categorías y atributos de productos

**Categoría:** Datos
**Probabilidad:** Media
**Impacto:** Medio
**Nivel de riesgo:** Medio
**Estado:** Activo
**Responsable de seguimiento:** Data Analyst

**Descripción:**
El dataset Olist tiene valores nulos conocidos en las categorías de
producto y en algunos atributos físicos (peso, dimensiones, fotos).
Esto afecta especialmente al canal content-based, que depende
fuertemente de los atributos del producto.

**Detonantes posibles:**
- Decisión apresurada de imputación que introduzca sesgos.
- Eliminación de filas con nulos que reduzca excesivamente el
  catálogo.
- Subvaloración del impacto de la calidad en la Etapa 2.

**Plan de mitigación:**
- El EDA de la Etapa 2 debe cuantificar explícitamente el porcentaje
  de nulos por columna y producto, y proponer una estrategia
  documentada de tratamiento.
- Las decisiones de tratamiento se registran en el documento de
  decisiones de feature engineering en la Etapa 3.

**Plan de contingencia:**
- Si el porcentaje de productos sin categoría es alto (>20%),
  considerar tratar "DESCONOCIDO" como una categoría legítima y
  añadir indicadores de missingness.
- Para atributos físicos, considerar imputación por mediana dentro de
  la categoría.

**Etapas afectadas:** 2, 3

---

### R-08 — Procesamiento de la matriz de co-ocurrencia para volumen grande

**Categoría:** Técnico
**Probabilidad:** Baja
**Impacto:** Medio
**Nivel de riesgo:** Bajo
**Estado:** Activo
**Responsable de seguimiento:** Data Scientist

**Descripción:**
La matriz de co-ocurrencia producto-producto en Olist puede tener
dimensiones del orden de 32,000 x 32,000 si se considera el catálogo
completo. Esto puede generar problemas de memoria si no se usa una
representación sparse adecuada.

**Detonantes posibles:**
- Implementación inicial con matrices densas.
- Cómputo de similitudes sin aprovechar sparsity.

**Plan de mitigación:**
- Usar `scipy.sparse` desde el inicio para la matriz de co-ocurrencia.
- Aprovechar `sklearn.neighbors.NearestNeighbors` con `algorithm='brute'`
  y métricas optimizadas para sparse arrays.

**Plan de contingencia:**
- Si la memoria es un problema, filtrar productos con menos de N
  apariciones (por ejemplo, N=5) para reducir el tamaño efectivo del
  catálogo modelable.

**Etapas afectadas:** 3, 4

---

## Resumen ejecutivo del registro

| Identificador | Categoría | Nivel | Estado |
|---|---|---|---|
| R-01 | Datos | Alto | Mitigado |
| R-02 | Tiempo | Alto | Activo |
| R-03 | Equipo | Medio | Mitigado |
| R-04 | Equipo | Medio | Activo |
| R-05 | Equipo | Medio | Activo |
| R-06 | Negocio | Medio | Activo |
| R-07 | Datos | Medio | Activo |
| R-08 | Técnico | Bajo | Activo |

**Riesgos críticos:** ninguno al inicio del proyecto.

**Riesgos altos no mitigados:** R-02 (duración corta vs alcance).

**Foco de seguimiento prioritario:** R-02 (escalable, requiere
disciplina de equipo durante todo el proyecto).

---

## Reglas de mantenimiento del registro

- El registro se revisa en cada Sprint Review como parte del cierre
  del sprint.
- Nuevos riesgos identificados durante el proyecto se agregan al
  documento con el siguiente identificador disponible.
- El estado de cada riesgo se actualiza cuando hay cambios
  significativos.
- Cuando un riesgo se materializa, su entrada se actualiza con la
  descripción de qué ocurrió y cómo se manejó.

---

*Registro de riesgos del Proyecto Final. Versión inicial al arranque
del proyecto. Documento vivo, actualizado durante la ejecución.*
