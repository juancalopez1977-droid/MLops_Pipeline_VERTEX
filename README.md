# VERTEX INSIGHTS - Sistema de Recomendación MLOps
## Proyecto de Recomendación Item-to-Item para E-Commerce OLIST

[![Python](https://img.shields.io/badge/Python-3.11+-blue.svg)](https://www.python.org/)
[![Status](https://img.shields.io/badge/Status-EDA_Complete-green.svg)]()
[![License](https://img.shields.io/badge/License-Academic-yellow.svg)]()

**Proyecto Académico** - Sistema de recomendación híbrido (Content-Based + Collaborative Filtering) para OLIST Brazilian E-Commerce

---

## 📋 Descripción del Proyecto

Este proyecto implementa un **sistema de recomendación item-to-item** para la plataforma de e-commerce brasileña OLIST, utilizando técnicas de Machine Learning y MLOps para generar recomendaciones personalizadas de productos.

### Objetivos
- ✅ Analizar dataset OLIST (99,441 órdenes, 2016-2018)
- ✅ Evaluar viabilidad de arquitectura híbrida (Content-Based + Collaborative Filtering)
- 🔄 Desarrollar pipeline de Feature Engineering
- 🔄 Implementar modelos de recomendación
- 🔄 Desplegar sistema con MLOps (CI/CD, monitoreo)

### Historia de Usuario (HU-05)
**Como** usuario del e-commerce OLIST  
**Quiero** recibir recomendaciones relevantes de productos  
**Para** descubrir artículos que se ajusten a mis preferencias y necesidades

---

## 🏗️ Arquitectura Propuesta

### Sistema Híbrido de Dos Canales

```
┌─────────────────────────────────────────────────────────────┐
│              SISTEMA DE RECOMENDACIÓN HÍBRIDO               │
└─────────────────────────────────────────────────────────────┘
                          │
         ┌────────────────┴────────────────┐
         │                                 │
    CANAL A (80%)                     CANAL B (20%)
    Content-Based                   Collaborative Filtering
         │                                 │
    ├─ Similitud                    ├─ Item-to-Item
    │  por categoría                │  (Top 1000 productos)
    ├─ Similitud                    ├─ Co-ocurrencia
    │  por precio                   │  (Clientes recurrentes)
    ├─ Atributos                    └─ Fallback:
    │  físicos                         Popularidad
    └─ TF-IDF embeddings
```

**Justificación:** Matriz cliente-producto con sparsity del **99.98%** → Collaborative Filtering limitado

---

## 📊 Dataset

### Fuente
- **Origen:** Brazilian E-Commerce Public Dataset by Olist (Kaggle)
- **Período:** Septiembre 2016 - Septiembre 2018
- **Registros:** 99,441 órdenes consolidadas
- **Dimensiones:** 58 variables (numéricas + categóricas + temporales)

### Variables Clave
- **Cliente:** `customer_unique_id`, `customer_state`, `customer_city`
- **Producto:** `primary_product_id`, `product_category_english`, `total_price`, `product_weight_grams`
- **Orden:** `order_id`, `order_purchase_timestamp`, `order_status`
- **Review:** `review_score`, `review_comment_title`
- **Logística:** `delivery_days`, `freight_value`

---

## 📁 Estructura del Proyecto

```
MLops_Pipeline_VERTEX/
│
├── notebooks/
│   └── 01_eda_olist.ipynb          # EDA completo (10 fases) ✅
│
├── reports/
│   └── figures/
│       └── eda/                     # 20 visualizaciones generadas ✅
│
├── docs/
│   └── 01_EDA_CONCLUSIONES.md      # Conclusiones y hallazgos ✅
│
├── src/                             # Scripts de modelado (pendiente)
│   ├── feature_engineering/
│   ├── models/
│   └── evaluation/
│
├── OLIST DATASETS/                  # Datasets raw (8 archivos CSV)
│
├── olist_consolidated.csv          # Dataset consolidado ✅
├── consolidate_olist.py            # Script de consolidación ✅
├── requirements.txt                # Dependencias Python ✅
└── README.md                        # Este archivo
```

---

## 🔬 Resultados del EDA

### Hallazgos Clave

#### 1. **Sparsity Crítica** (99.98%)
- **96,096 clientes** × **32,951 productos** = 3.16B posibles interacciones
- Solo **98,666 interacciones reales** (densidad: 0.0031%)
- **Implicación:** Collaborative Filtering tradicional no viable

#### 2. **Cold Start Extremo**
- **96.8%** clientes con solo 1 compra (no hay historial)
- **58.3%** productos con solo 1 venta (difícil calcular similitudes CF)
- **Solución:** Priorizar content-based + fallback de popularidad

#### 3. **Concentración Geográfica**
- **70%** ventas en región Sudeste (São Paulo, Rio, Minas Gerais)
- São Paulo representa **41.7%** del total de órdenes
- Oportunidad para recomendaciones geo-localizadas

#### 4. **Estacionalidad**
- Pico de ventas en **Noviembre-Diciembre** (Black Friday + Navidad)
- Crecimiento sostenido 2017-2018
- Insight: Ajustar pesos de recomendación por temporada

#### 5. **Satisfacción vs Delivery**
- Correlación negativa: Entregas tardías → ratings bajos
- Entregas <7 días: **85% satisfacción**
- Entregas >15 días: **45% satisfacción**
- Recomendación: Incluir `delivery_estimate` en features

### Visualizaciones Generadas

Ver carpeta [reports/figures/eda/](reports/figures/eda/) para las 20 visualizaciones:
- Distribuciones (variables numéricas, categóricas, temporales)
- Mapas de calor (geografía, correlaciones, co-compra)
- Segmentación RFM
- Análisis de sparsity
- Perfiles de categorías

### Documentación Completa

📄 **Documento de Conclusiones:** [docs/01_EDA_CONCLUSIONES.md](docs/01_EDA_CONCLUSIONES.md)

---

## 🚀 Instalación y Uso

### Requisitos
- Python 3.11+
- Git
- Jupyter Notebook / JupyterLab

### Setup Rápido

```bash
# 1. Clonar repositorio
git clone <repo-url>
cd MLops_Pipeline_VERTEX

# 2. Crear entorno virtual
python -m venv venv

# 3. Activar entorno
# Windows (Git Bash)
source venv/Scripts/activate
# Linux/Mac
source venv/bin/activate

# 4. Instalar dependencias
pip install -r requirements.txt

# 5. Abrir notebook EDA
jupyter notebook notebooks/01_eda_olist.ipynb
```

### Generar Dataset Consolidado

```bash
# Si necesitas regenerar el dataset consolidado
python consolidate_olist.py
```

---

## 📦 Dependencias Principales

```
pandas>=2.1.0           # Manipulación de datos
numpy>=1.24.0           # Cálculos numéricos
matplotlib>=3.7.0       # Visualizaciones básicas
seaborn>=0.13.0         # Gráficos estadísticos
plotly>=5.17.0          # Visualizaciones interactivas
scikit-learn>=1.3.0     # Machine Learning (próxima fase)
jupyter>=1.0.0          # Notebooks interactivos
```

Ver archivo completo: [requirements.txt](requirements.txt)

---

## 📈 Roadmap del Proyecto

### ✅ Fase 1: Análisis Exploratorio (COMPLETADO)
- [x] Consolidación de 8 datasets OLIST
- [x] EDA exhaustivo (10 fases)
- [x] Análisis de sparsity
- [x] Validación de arquitectura híbrida
- [x] Documentación de hallazgos

### 🔄 Fase 2: Feature Engineering (EN PROGRESO)
- [ ] Imputación de valores faltantes
- [ ] Tratamiento de outliers
- [ ] Features derivados (volumen, precio/kg, etc.)
- [ ] Encoding de categorías (TF-IDF)
- [ ] Normalización de variables

### 📋 Fase 3: Modelado
- [ ] Modelo Content-Based (similitud coseno)
- [ ] Modelo Collaborative Filtering (Item-to-Item)
- [ ] Sistema híbrido (weighted average)
- [ ] Validación cruzada

### 📋 Fase 4: Evaluación
- [ ] Métricas offline (Precision@K, Recall@K, Coverage)
- [ ] A/B Testing framework
- [ ] Análisis de diversidad y serendipity

### 📋 Fase 5: MLOps y Deploy
- [ ] API REST (FastAPI)
- [ ] Containerización (Docker)
- [ ] CI/CD pipeline (GitHub Actions)
- [ ] Monitoreo (Prometheus + Grafana)
- [ ] Logging centralizado

---

## 🎯 Métricas de Éxito

### Negocio
- **Click-Through Rate (CTR):** Target >5%
- **Conversion Rate:** Target >2%
- **Average Order Value:** Incremento +15%

### Técnicas
- **Precision@10:** Target >0.15
- **Recall@10:** Target >0.08
- **Coverage:** Target >70% del catálogo
- **Latencia:** <100ms por recomendación

---

## 👥 Equipo

**VERTEX INSIGHTS - Data Science Team**

Proyecto académico desarrollado para demostrar capacidades en:
- Análisis exploratorio de datos (EDA)
- Machine Learning (sistemas de recomendación)
- MLOps (CI/CD, monitoreo, deployment)

---

## 📚 Referencias

### Dataset
- **Kaggle:** [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
- **Papers:** Collaborative Filtering, Content-Based Filtering, Hybrid Recommendation Systems

### Tecnologías
- **Python:** Lenguaje principal
- **Pandas/NumPy:** Procesamiento de datos
- **Scikit-learn:** Modelado ML
- **Plotly/Seaborn:** Visualizaciones
- **Jupyter:** Notebooks interactivos

---

## 📄 Licencia

Este proyecto es de carácter **académico** y no está destinado para uso comercial.

---

## 📧 Contacto

Para consultas sobre el proyecto, contactar al equipo VERTEX INSIGHTS.

---

**Última actualización:** Junio 2026  
**Estado Actual:** ✅ EDA Completo | 🔄 Feature Engineering Iniciando
