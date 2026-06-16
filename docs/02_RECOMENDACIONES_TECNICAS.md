# Recomendaciones Técnicas para el Equipo
## Feature Engineering y Modelado - Sistema de Recomendación

**Proyecto:** VERTEX INSIGHTS - HU-05  
**Para:** Equipo de Data Science y MLOps  
**Fecha:** Junio 2026

---

## 🎯 CONTEXTO

Basado en el EDA completado, este documento proporciona guías técnicas específicas para los siguientes sprints del proyecto (Feature Engineering y Modelado).

---

## 📋 PRIORIDADES SEGÚN HALLAZGOS EDA

### 🔴 CRÍTICO (Implementar primero)

#### 1. Estrategia Content-Based First
**Justificación:** Sparsity 99.98% hace CF inviable como canal principal

**Acción:**
```python
# Implementar similitud coseno entre productos
from sklearn.metrics.pairwise import cosine_similarity

# Features sugeridos:
features = [
    'product_category_english',      # One-hot encoded
    'total_price',                   # Normalizado
    'product_weight_grams',          # Normalizado
    'product_photos_qty',            # Normalizado
    'volume_cm3',                    # Derivado: length × height × width
    'price_per_kg'                   # Derivado: price / weight
]
```

#### 2. Manejo de Cold Start
**Problema:** 96.8% clientes con 1 sola compra

**Solución Multi-tier:**
```python
def get_recommendations(user_id, product_id):
    if user_history.count() >= 3:
        # Usuario recurrente → Usar híbrido
        return hybrid_model(user_id, product_id)
    elif user_history.count() == 1:
        # Usuario nuevo → Content-based + popularidad
        return content_based(product_id) + popular_in_category(category)
    else:
        # Sin historial → Popularidad general
        return popular_products_overall()
```

#### 3. Imputación de Valores Faltantes
**Columnas prioritarias:**
- `product_weight_grams`: Imputar con **mediana por categoría**
- `product_photos_qty`: Imputar con **mediana por categoría**
- `product_description_length`: Imputar con 0 (asumir sin descripción)

```python
# Ejemplo: Imputación estratificada
for category in df['product_category_english'].unique():
    mask = (df['product_category_english'] == category) & df['product_weight_grams'].isna()
    median_weight = df[df['product_category_english'] == category]['product_weight_grams'].median()
    df.loc[mask, 'product_weight_grams'] = median_weight
```

---

### 🟡 IMPORTANTE (Implementar en Sprint 2)

#### 4. Tratamiento de Outliers
**Variables afectadas:**
- `total_price`: 4.7% outliers
- `product_weight_grams`: 5.2% outliers

**Estrategia: Winsorization (Cap en percentil 99)**
```python
from scipy.stats.mstats import winsorize

# Aplicar cap sin eliminar datos
df['total_price_capped'] = winsorize(df['total_price'], limits=[0.01, 0.01])
df['weight_capped'] = winsorize(df['product_weight_grams'], limits=[0.01, 0.01])
```

**Alternativa:** Transformación logarítmica para variables con long-tail
```python
import numpy as np
df['log_price'] = np.log1p(df['total_price'])  # log(1 + x) para evitar log(0)
```

#### 5. Feature Engineering - Derivados Críticos

```python
# 1. Volumen del producto (indicador de tamaño)
df['volume_cm3'] = (
    df['product_length_cm'] * 
    df['product_height_cm'] * 
    df['product_width_cm']
)

# 2. Precio por kilogramo (normalizado)
df['price_per_kg'] = df['total_price'] / (df['product_weight_grams'] / 1000)

# 3. Ratio precio/fotos (calidad de listing)
df['price_per_photo'] = df['total_price'] / df['product_photos_qty'].replace(0, 1)

# 4. Densidad del producto (precio/volumen)
df['price_density'] = df['total_price'] / df['volume_cm3']

# 5. Categoría de precio (bucketing)
df['price_category'] = pd.cut(
    df['total_price'], 
    bins=[0, 50, 100, 200, 500, float('inf')],
    labels=['Budget', 'Economy', 'Mid', 'Premium', 'Luxury']
)

# 6. Temporada de compra
df['purchase_month'] = pd.to_datetime(df['order_purchase_timestamp']).dt.month
df['is_holiday_season'] = df['purchase_month'].isin([11, 12])  # Nov-Dic

# 7. Velocidad de entrega
df['is_fast_delivery'] = df['delivery_days'] <= 7
```

---

### 🟢 DESEABLE (Optimización posterior)

#### 6. Embeddings de Categorías (TF-IDF)
**Para:** Capturar similitud semántica entre categorías

```python
from sklearn.feature_extraction.text import TfidfVectorizer

# Crear corpus de categorías
corpus = df.groupby('product_category_english')['product_category_english'].first()

# TF-IDF
vectorizer = TfidfVectorizer(max_features=50)
category_embeddings = vectorizer.fit_transform(corpus)
```

#### 7. Normalización de Variables Numéricas
**Método:** StandardScaler (media 0, std 1)

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
numeric_cols = ['total_price', 'product_weight_grams', 'product_photos_qty', 'volume_cm3']

df[numeric_cols] = scaler.fit_transform(df[numeric_cols])
```

---

## 🤖 MODELADO: GUÍA DE IMPLEMENTACIÓN

### Fase 1: Content-Based Filtering (Canal A)

#### Paso 1: Crear Matriz de Features
```python
from sklearn.preprocessing import OneHotEncoder
from sklearn.compose import ColumnTransformer

# Definir transformers
categorical_features = ['product_category_english', 'price_category']
numerical_features = ['total_price', 'product_weight_grams', 'volume_cm3', 'product_photos_qty']

preprocessor = ColumnTransformer(
    transformers=[
        ('cat', OneHotEncoder(handle_unknown='ignore'), categorical_features),
        ('num', StandardScaler(), numerical_features)
    ]
)

# Transformar
product_features = preprocessor.fit_transform(df)
```

#### Paso 2: Calcular Similitud
```python
from sklearn.metrics.pairwise import cosine_similarity

# Matriz de similitud (product × product)
similarity_matrix = cosine_similarity(product_features)

# Función de recomendación
def recommend_similar_products(product_id, top_n=10):
    idx = df[df['primary_product_id'] == product_id].index[0]
    sim_scores = list(enumerate(similarity_matrix[idx]))
    sim_scores = sorted(sim_scores, key=lambda x: x[1], reverse=True)
    sim_scores = sim_scores[1:top_n+1]  # Excluir el mismo producto
    
    product_indices = [i[0] for i in sim_scores]
    return df.iloc[product_indices][['primary_product_id', 'product_category_english', 'total_price']]
```

---

### Fase 2: Collaborative Filtering (Canal B)

#### Enfoque: Item-to-Item (Limitado a Top 1000 productos)

```python
from scipy.sparse import csr_matrix
from sklearn.neighbors import NearestNeighbors

# Filtrar top 1000 productos más populares
top_products = df['primary_product_id'].value_counts().head(1000).index
df_filtered = df[df['primary_product_id'].isin(top_products)]

# Crear matriz cliente-producto (sparse)
pivot = df_filtered.pivot_table(
    index='customer_unique_id',
    columns='primary_product_id',
    values='order_id',
    aggfunc='count',
    fill_value=0
)

sparse_matrix = csr_matrix(pivot.values)

# KNN para encontrar productos similares
model_knn = NearestNeighbors(metric='cosine', algorithm='brute', n_neighbors=11)
model_knn.fit(sparse_matrix.T)  # Transponer para item-to-item

def recommend_cf(product_id, top_n=10):
    if product_id not in pivot.columns:
        return []  # Fallback a popularidad
    
    product_idx = list(pivot.columns).index(product_id)
    distances, indices = model_knn.kneighbors(
        sparse_matrix.T[product_idx].reshape(1, -1),
        n_neighbors=top_n+1
    )
    
    similar_products = [pivot.columns[i] for i in indices.flatten()[1:]]
    return similar_products
```

---

### Fase 3: Sistema Híbrido

```python
def hybrid_recommendation(product_id, user_id=None, top_n=10, alpha=0.8):
    """
    Sistema híbrido con pesos ajustables
    
    Args:
        product_id: ID del producto actual
        user_id: ID del usuario (opcional)
        top_n: Número de recomendaciones
        alpha: Peso de content-based (1-alpha para CF)
    
    Returns:
        List de product_ids recomendados
    """
    # Canal A: Content-Based (80%)
    cb_recommendations = recommend_similar_products(product_id, top_n=20)
    cb_scores = {prod: alpha * (1 - i/20) for i, prod in enumerate(cb_recommendations['primary_product_id'])}
    
    # Canal B: Collaborative Filtering (20%)
    cf_recommendations = recommend_cf(product_id, top_n=20)
    cf_scores = {prod: (1-alpha) * (1 - i/20) for i, prod in enumerate(cf_recommendations)}
    
    # Combinar scores
    all_products = set(cb_scores.keys()) | set(cf_scores.keys())
    hybrid_scores = {
        prod: cb_scores.get(prod, 0) + cf_scores.get(prod, 0)
        for prod in all_products
    }
    
    # Ordenar y devolver top N
    sorted_products = sorted(hybrid_scores.items(), key=lambda x: x[1], reverse=True)
    return [prod for prod, score in sorted_products[:top_n]]
```

---

## 📊 EVALUACIÓN DEL MODELO

### Métricas Offline (Hold-out test set)

```python
from sklearn.model_selection import train_test_split

# Split temporal (últimos 20% como test)
df_sorted = df.sort_values('order_purchase_timestamp')
train_size = int(0.8 * len(df))
train_df = df_sorted[:train_size]
test_df = df_sorted[train_size:]

# Precision@K
def precision_at_k(recommendations, actual_purchases, k=10):
    rec_k = recommendations[:k]
    relevant = set(actual_purchases) & set(rec_k)
    return len(relevant) / k

# Recall@K
def recall_at_k(recommendations, actual_purchases, k=10):
    rec_k = recommendations[:k]
    relevant = set(actual_purchases) & set(rec_k)
    return len(relevant) / len(actual_purchases) if actual_purchases else 0

# Coverage
def catalog_coverage(all_recommendations, catalog_size):
    unique_recommended = set(all_recommendations)
    return len(unique_recommended) / catalog_size
```

---

## 🔧 HERRAMIENTAS Y LIBRERÍAS RECOMENDADAS

### Para Feature Engineering
- **pandas-profiling:** EDA rápido de nuevas features
- **feature-engine:** Pipelines de ingeniería de features
- **category_encoders:** Encoding avanzado de categóricas

### Para Modelado
- **scikit-learn:** Modelos baseline (KNN, cosine similarity)
- **implicit:** Collaborative Filtering optimizado (ALS, BPR)
- **lightfm:** Hybrid recommender systems
- **surprise:** Librería especializada en CF

### Para Evaluación
- **recmetrics:** Métricas de sistemas de recomendación
- **scikit-learn.metrics:** Precision, Recall, F1

### Para Deploy (Fase posterior)
- **FastAPI:** API REST ligera y rápida
- **Redis:** Cache de recomendaciones
- **Docker:** Containerización
- **MLflow:** Tracking de experimentos y versionado de modelos

---

## 🚨 RIESGOS Y MITIGACIONES

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|--------------|---------|------------|
| CF no converge por sparsity | Alta | Medio | Usar solo top 1000 productos + fallback |
| Cold start para nuevos productos | Alta | Alto | Recomendaciones basadas en categoría + popularidad |
| Latencia alta en inferencia | Media | Medio | Pre-calcular similitudes y usar Redis cache |
| Overfitting en productos populares | Media | Medio | Penalizar popularidad en scoring (diversity) |
| Bias geográfico (70% SE Brasil) | Baja | Bajo | Features de localización + filters regionales |

---

## 📅 CRONOGRAMA SUGERIDO

### Sprint 2: Feature Engineering (2 semanas)
- **Semana 1:** Imputación + outliers + features derivados
- **Semana 2:** Normalización + encoding + validación

### Sprint 3: Content-Based (2 semanas)
- **Semana 1:** Matriz de similitud + función de recomendación
- **Semana 2:** Evaluación + fine-tuning de pesos

### Sprint 4: Collaborative Filtering (2 semanas)
- **Semana 1:** Item-to-Item KNN + co-occurrence matrix
- **Semana 2:** Integración con sistema híbrido

### Sprint 5: Sistema Híbrido (1 semana)
- **Semana 1:** Weighted combination + evaluación final

---

## 📚 RECURSOS ADICIONALES

### Papers Relevantes
1. **"Amazon.com Recommendations: Item-to-Item Collaborative Filtering"** (Linden et al., 2003)
2. **"Content-Based Recommendation Systems"** (Pazzani & Billsus, 2007)
3. **"Hybrid Recommender Systems: Survey and Experiments"** (Burke, 2002)

### Tutoriales
- Scikit-learn: Content-Based Filtering
- Implicit library: ALS for Collaborative Filtering
- Building Recommender Systems with Python (Manning)

---

## ✅ CHECKLIST PRE-MODELADO

Antes de comenzar el modelado, asegurar:

- [ ] Dataset consolidado sin nulls en features críticos
- [ ] Outliers tratados (winsorization o caps)
- [ ] Features derivados creados y validados
- [ ] Train/test split temporal (no aleatorio)
- [ ] Baseline de popularidad implementado (para comparación)
- [ ] Métricas de evaluación definidas y automatizadas
- [ ] Pipeline de preprocessing documentado y reproducible

---

**Última actualización:** Junio 2026  
**Preparado por:** Equipo VERTEX INSIGHTS - Data Science  
**Estado:** ✅ Ready for Implementation
