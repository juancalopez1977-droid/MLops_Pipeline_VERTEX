# Conclusiones del Análisis Exploratorio de Datos (EDA)
## Sistema de Recomendación Item-to-Item - OLIST E-Commerce

**Proyecto:** VERTEX INSIGHTS - HU-05  
**Dataset:** OLIST Brazilian E-Commerce (2016-2018)  
**Fecha:** Junio 2026  
**Analista:** Equipo VERTEX INSIGHTS

---

## 📊 RESUMEN EJECUTIVO

Se realizó un análisis exploratorio exhaustivo del dataset consolidado de OLIST, compuesto por **99,441 órdenes** y **58 variables**, con el objetivo de evaluar la viabilidad de un sistema de recomendación híbrido item-to-item con dos canales:

- **Canal A:** Content-Based Filtering (basado en atributos de productos)
- **Canal B:** Collaborative Filtering (basado en patrones de co-compra)

### Hallazgo Principal
La matriz cliente-producto presenta una **sparsity del 99.98%**, lo que indica que el Collaborative Filtering tradicional será extremadamente desafiante. Se recomienda priorizar fuertemente el **Canal A (Content-Based)** y usar Canal B solo como complemento limitado.

---

## 🔍 ANÁLISIS POR FASE

### FASE 1-2: Calidad de Datos

#### ✅ Fortalezas
- Dataset consolidado exitosamente desde 8 fuentes diferentes
- **No hay duplicados** en order_id (integridad confirmada)
- 58 variables ricas para análisis multidimensional

#### ⚠️ Issues Identificados
1. **Valores Faltantes:** 20 columnas con nulls
   - `order_delivered_carrier_date`: 1.6% faltantes
   - `order_delivered_customer_date`: 2.7% faltantes
   - Múltiples columnas de productos con 20-30% nulls

2. **Outliers:** Detectados en variables críticas
   - `total_price`: 4.7% outliers (método IQR)
   - `product_weight_grams`: 5.2% outliers
   - `delivery_days`: 3.8% valores atípicos

**Acción Requerida:** Imputación estratégica pre-modelado

---

### FASE 3: Análisis Univariable

#### Variables Numéricas
- **Precio:** Mediana R$74.99, alta variabilidad (productos R$0.85 - R$6,735)
- **Review Score:** Sesgo positivo (70% ratings 4-5 estrellas)
- **Peso Productos:** Distribución long-tail (mayoría <2kg, máx 40kg)

#### Variables Categóricas
- **Categorías:** 73 categorías, concentración en top 10 (52% ventas)
- **Top 3 Categorías:**
  1. Bed Bath Table (11,119 órdenes)
  2. Health Beauty (9,669 órdenes)
  3. Sports Leisure (8,641 órdenes)

#### Temporal
- **Estacionalidad:** Pico en Nov-Dic (Black Friday / Navidad)
- **Tendencia:** Crecimiento sostenido 2017-2018

---

### FASE 4-5: Relaciones Bivariables y Multivariables

#### Precio vs Categoría
- Alta variabilidad intra-categoría
- Categorías premium: Computers, Telephony (precio medio >R$500)
- Categorías económicas: Fashion, Home (precio medio <R$100)

#### Satisfacción vs Delivery
- **Correlación negativa fuerte:** Entregas tardías → reviews bajos
- Entregas <7 días: 85% satisfacción (4-5 estrellas)
- Entregas >15 días: 45% satisfacción

#### Geografía
- **Concentración:** 70% ventas en región Sudeste (SP, RJ, MG)
- **São Paulo:** 41.7% del total de órdenes

#### Segmentación RFM
- **Champions (Recency baja, Frequency alta):** Solo 2.8% clientes
- **At Risk (compraron hace tiempo):** 18.5% clientes
- **Hibernating (inactivos):** 12.3% clientes

---

### FASE 6: **SPARSITY ANALYSIS** (CRÍTICO)

#### Métricas de la Matriz Cliente-Producto

| Métrica | Valor |
|---------|-------|
| **Clientes únicos** | 96,096 |
| **Productos únicos** | 32,951 |
| **Tamaño matriz teórico** | 3,166,618,896 celdas |
| **Interacciones reales** | 98,666 |
| **Density** | 0.0031% |
| **Sparsity** | **99.9969%** |

#### Interpretación
🔴 **EXTREMADAMENTE SPARSE (>99.5%)**
- Collaborative filtering será **muy desafiante**
- **Priorizar Canal A (content-based)** fuertemente
- Canal B solo como complemento para productos populares

#### Distribución de Compras

**Clientes:**
- 1 compra: 97,004 clientes (96.8%) ← **Cold Start crítico**
- 2 compras: 2,030 clientes (2.0%)
- 3+ compras: 1,062 clientes (1.1%)

**Productos:**
- 1 venta: 19,203 productos (58.3%) ← **Long tail extremo**
- 2 ventas: 5,125 productos (15.6%)
- 3+ ventas: 8,623 productos (26.2%)

---

### FASE 7: Collaborative Filtering Viability

#### 7.1. Co-compra de Productos (Dentro de Clientes Recurrentes)
- **Clientes recurrentes:** 3,092 (3.1% del total)
- **Pares únicos detectados:** 4,856
- **Co-ocurrencias totales:** 5,721
- **Promedio co-ocurrencias/par:** 1.18 (muy bajo)

**Conclusión:** Co-ocurrencias insuficientes para CF robusto

#### 7.2. Co-compra de Categorías (Mismo Pedido)
- **Órdenes multi-categoría:** 0 (0.0%)
- **Hallazgo:** Clientes NO mezclan categorías en misma orden

**Implicación:** Recomendaciones deben ser dentro de misma categoría

---

### FASE 8: Content-Based Filtering Features

#### Features Disponibles por Producto

**Categóricos:**
- `product_category_english` (73 categorías)
- `seller_state` (ubicación vendedor)

**Numéricos:**
- `total_price` (precio final)
- `product_weight_grams` (peso)
- `product_length_cm`, `product_height_cm`, `product_width_cm` (dimensiones)
- `product_photos_qty` (cantidad fotos)

**Derivados (sugeridos):**
- Volumen del producto (length × height × width)
- Precio por kg
- Ratio precio/fotos (indicador calidad listing)

#### Perfiles de Categorías (Top 10)
- **Categorías Premium:** Computers (precio alto, peso medio)
- **Categorías Funcionales:** Bed Bath Table (precio medio, peso alto)
- **Categorías Fashion:** Low price, low weight, high photo count

**Estrategia Content-Based:**
1. Similitud por categoría (mismo sector)
2. Similitud por precio (±20% rango)
3. Similitud por atributos físicos (peso, volumen)

---

### FASE 9: Consolidación de Issues de Calidad

#### Resumen por Severidad

| Severidad | Count | Issues Principales |
|-----------|-------|-------------------|
| **ALTA** | 2 | Sparsity extrema, Cold start clientes |
| **MEDIA** | 3 | Valores faltantes, Outliers, Cold start productos |
| **BAJA** | 2 | Duplicados esperados, Inconsistencias pago |

#### Prioridades de Acción

**1. INMEDIATO (Pre-modelado):**
- ✅ Aceptar que CF será limitado → Arquitectura content-first
- ✅ Estrategia de fallback para clientes nuevos (popularidad + categoría)

**2. CORTO PLAZO (Feature Engineering):**
- 🔧 Imputación de valores faltantes en atributos producto
- 🔧 Winsorization de outliers (cap en percentil 99)
- 🔧 Crear features derivados (volumen, precio/kg)

**3. LARGO PLAZO (Mejora Continua):**
- 📊 Monitorear nuevo historial de compras para mejorar CF
- 📊 A/B testing entre pesos de Canal A vs Canal B

---

## 🎯 ARQUITECTURA RECOMENDADA

### Sistema Híbrido Ajustado

```
┌─────────────────────────────────────────┐
│   SISTEMA DE RECOMENDACIÓN HÍBRIDO     │
└─────────────────────────────────────────┘
              │
              ├─► CANAL A (Content-Based)  → 80% PESO
              │   ├─ Similitud por categoría
              │   ├─ Similitud por precio
              │   ├─ Similitud por atributos físicos
              │   └─ Embeddings TF-IDF de categorías
              │
              └─► CANAL B (Collaborative)   → 20% PESO
                  ├─ Item-to-Item (solo top 1000 productos)
                  ├─ Co-ocurrencia en clientes recurrentes
                  └─ Fallback: Popularidad en categoría
```

### Scoring System (Automático - Fase 6)

**Collaborative Filtering Score:** 15/100
- Sparsity: >99.5% → 0 puntos (de 40 max)
- Clientes recurrentes: 96.8% one-time → 5 puntos (de 30 max)
- Productos multi-venta: 58.3% one-sale → 10 puntos (de 30 max)

**Interpretación:** CF extremadamente limitado → Priorizar content-based

---

## 📈 MÉTRICAS DE ÉXITO PROPUESTAS

### Métricas de Negocio
1. **Click-Through Rate (CTR):** % usuarios que hacen clic en recomendaciones
2. **Conversion Rate:** % recomendaciones que resultan en compra
3. **Average Order Value (AOV):** Incremento en ticket promedio
4. **Cross-Sell Success:** % órdenes con productos recomendados

### Métricas Técnicas
1. **Precision@K:** Precisión en top-K recomendaciones (K=5, 10, 20)
2. **Recall@K:** Cobertura de productos relevantes
3. **Coverage:** % catálogo que puede ser recomendado
4. **Diversity:** Variedad de categorías recomendadas
5. **Cold Start Performance:** Accuracy para usuarios/productos nuevos

---

## 🚀 ROADMAP DE PRÓXIMOS PASOS

### Sprint 1: Feature Engineering (2 semanas)
- [ ] Imputación de valores faltantes
- [ ] Tratamiento de outliers (winsorization)
- [ ] Crear features derivados (volumen, precio/kg, etc.)
- [ ] Normalización de variables numéricas
- [ ] Encoding de categorías (TF-IDF, embeddings)

### Sprint 2: Modelo Content-Based (3 semanas)
- [ ] Implementar similitud coseno entre productos
- [ ] Desarrollar matriz de similitud por categoría
- [ ] Sistema de ponderación de features
- [ ] Validación cruzada del modelo

### Sprint 3: Modelo Collaborative (2 semanas)
- [ ] Implementar Item-to-Item CF (top 1000 productos)
- [ ] Co-occurrence matrix para recurrentes
- [ ] Sistema de fallback (popularidad)

### Sprint 4: Sistema Híbrido (2 semanas)
- [ ] Integración Canal A + Canal B
- [ ] Sistema de pesos dinámicos (80/20 inicial)
- [ ] API de recomendaciones

### Sprint 5: Evaluación y Deploy (2 semanas)
- [ ] A/B Testing framework
- [ ] Métricas de negocio en producción
- [ ] Monitoreo y alertas
- [ ] Documentación final

---

## 📚 REFERENCIAS

### Datasets Originales
- **Fuente:** Brazilian E-Commerce Public Dataset by Olist (Kaggle)
- **Período:** Sep 2016 - Sep 2018
- **Registros:** 99,441 órdenes únicas
- **Link:** https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce

### Librerías Utilizadas
- `pandas` 2.x: Manipulación de datos
- `numpy` 1.26.x: Cálculos numéricos
- `matplotlib` 3.8.x: Visualizaciones
- `seaborn` 0.13.x: Gráficos estadísticos
- `plotly` 5.x: Visualizaciones interactivas

### Notebooks
- `notebooks/01_eda_olist.ipynb`: EDA completo (10 fases)

---

## 👥 EQUIPO

**VERTEX INSIGHTS - Data Science Team**  
Proyecto Académico - Sistema de Recomendación MLOps

---

**Última actualización:** Junio 2026  
**Versión:** 1.0  
**Estado:** ✅ EDA Completo - Ready for Feature Engineering
