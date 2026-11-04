# Práctica 4 - Reto Text-based Personality Computing (Big-5 en Reddit)

Repositorio del proyecto para el reto de **Text-based Personality Computing (TPC)** de la materia **Tecnoloxías da Linguaxe** (Grao en Intelixencia Artificial - USC).

El objetivo es predecir las **5 dimensiones de personalidad del modelo Big Five** (*agreeableness*, *openness*, *conscientiousness*, *extraversion*, *neuroticism*), en una escala de `0` a `100`, a nivel de usuario/a a partir de su historial de publicaciones en Reddit.

---

## 📂 Estructura de Datos (`data/`)

> **Nota:** Los archivos `.csv` de datos están excluidos de Git mediante `.gitignore` debido a su tamaño (`posts_v2.csv` ~690 MB). Deben ubicarse en el directorio `data/`:

* `data/posts_v2.csv`: Publicaciones de los usuarios en Reddit (tanto de entrenamiento como de test).
* `data/authors_train.csv`: **1.288 usuarios** de entrenamiento con sus puntuaciones reales (`0.0` – `100.0`) en las 5 dimensiones Big Five.
* `data/authors_test.csv`: **323 usuarios** de evaluación para los que se deben estimar las 5 dimensiones.

### Formato de entrega de variantes (`.csv`)
Las predicciones se evalúan mediante el **RMSE medio** de las 5 dimensiones y deben seguir el formato:
```csv
team_name,variant_name,username,agreeableness,openness,conscientiousness,extraversion,neuroticism
```
Nombre del archivo: `nome_equipo_nome_variante.csv`.

---

## 🧠 Estrategia y Metodología

Nuestra línea principal parte de representaciones densas con **SBERT (Sentence-BERT)** —tanto en configuración supervisada como no supervisada mediante cuestionarios psicológicos— para después **comparar su rendimiento frente a otras arquitecturas** clásicas y modernas.

### 1. Enfoques basados en SBERT (Línea Principal)
* **1A. SBERT Supervisado (Agregación por usuario + Regresión):**
  * Extracción de embeddings por publicación con modelos Sentence-Transformers (`all-MiniLM-L6-v2`, `all-mpnet-base-v2`, `BAAI/bge-base-en-v1.5`).
  * Agregación a nivel de usuario (`mean pooling`, `std pooling`, cuantiles).
  * Cabezales de regresión multi-salida (`Ridge`, `ElasticNet`, `SVR`, `LightGBM` / `CatBoost`, `MLP`).
* **1B. SBERT No Supervisado / Híbrido (Alineamiento con NEO-FFI / IPIP):**
  * Codificación con SBERT de los ítems de cuestionarios psicológicos (**NEO-FFI** / **IPIP-NEO**), separando las afirmaciones del **polo positivo** (cercano a 100) y del **polo negativo** (cercano a 0).
  * Cálculo de similitud del coseno entre las publicaciones del usuario y cada polo para estimar cada rasgo y calibrarlo a la escala `0–100`.
* **1C. Filtrado Semántico de Publicaciones Relevantes:**
  * Uso de la similitud con los ítems del NEO-FFI para seleccionar el *Top-K* de publicaciones u oraciones más informativas psicológicamente de cada usuario, descartando ruido antes de entrenar o inferir.

### 2. Modelos de Comparación
1. **Baseline Clásico + Estilometría:**
   * `TF-IDF` (n-gramas de palabras y caracteres) + características psicolingüísticas/estilométricas (uso de pronombres, puntuación, longitud de textos, léxicos de emociones/sentimiento) con `Ridge` / `LightGBM`.
2. **Multiple Instance Learning (Attention Pooling sobre SBERT):**
   * Red neuronal con mecanismo de atención multi-cabeza que aprende a ponderar automáticamente qué publicaciones de un usuario son más relevantes para cada uno de los 5 rasgos.
3. **Fine-tuning de Transformers:**
   * Ajuste fino (*fine-tuning*) de un modelo `RoBERTa` / `DeBERTa-v3` sobre muestras concatenadas o bloques de publicaciones por usuario con pérdida `MSELoss` sobre las 5 salidas.
4. **Zero-shot / Few-shot con LLMs:**
   * Evaluación mediante prompting estructurado sobre las publicaciones filtradas de cada usuario.
5. **Ensemble / Stacking Final:**
   * Combinación ponderada (*blending* / *Ridge stacking*) validada por *K-Fold Cross-Validation* de las mejores variantes de SBERT + Estilometría/TF-IDF + Transformer.

---

## 📅 Hoja de Ruta y Entregas

| Hito | Fecha límite | Objetivo |
| :--- | :--- | :--- |
| **Entrega 1 (Variantes iniciales)** | 19 de noviembre (23:59) | Baselines: SBERT Supervisado, SBERT No Supervisado (NEO-FFI) y TF-IDF + Ridge. *(Leaderboard: 24 nov)* |
| **Entrega 2 (Variantes intermedias)** | 3 de diciembre (23:59) | Filtrado de posts relevantes, Attention Pooling (MIL) y Fine-tuning Transformer. *(Leaderboard: 7 dic)* |
| **Entrega Final de Variantes** | 13 de diciembre (23:59) | Mejores modelos individuales + Ensemble/Stacking final (2–3 variantes). |
| **Entrega del Notebook Final** | 14 de diciembre (23:59) | Entrega de `nome_equipo_reto.ipynb` autoexplicativo con código y resultados. |
| **Presentación en clase** | 15 y 16 de diciembre | Presentación de las soluciones y publicación de la leaderboard final. |
