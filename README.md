🎵 EDA Spotify 2009–2025

Exploratory Data Analysis (EDA) completo sobre track_data_final.csv, con limpieza, feature engineering y visualizaciones listas para modelado de IA.

🛠️ Resumen del Script
Objetivo: Limpiar, preparar y enriquecer el dataset para análisis y modelado.

Principales pasos realizados:

📥 Lectura segura del CSV (readr::read_csv, cols por defecto como character).

🧹 Limpieza de nombres (janitor::clean_names).

🔢 Coerción segura de columnas numéricas y lógicas (parse_num_safe()).

📅 Parseo robusto de fechas (lubridate::parse_date_time).

✨ Feature engineering: nuevas columnas no destructivas para modelado.

📊 Estadísticas resumidas (media, sd, mediana; conteos de categorías).

📈 Generación de 8+ gráficas guardadas en eda_plots/.

🗂️ Mapa de correlación mejorado y export a CSV (correlation_matrix.csv).

💾 Guardado final: processed_spotify_eda_final.csv.

El script funciona en Colab o Rscript y permite configurar umbrales, rangos de años y número de variables para el heatmap.
⚡ Cómo Ejecutar

En Colab con rpy2:

# Instalar R y rpy2
sudo apt-get update -y
sudo apt-get install -y r-base
pip install rpy2

# Cargar extensión
%load_ext rpy2.ipython

Archivos generados:

processed_spotify_eda_final.csv → dataset procesado

Carpeta eda_plots/ → PNGs de gráficas

eda_plots/correlation_matrix.csv → matriz de correlación numérica

✨ Columnas Nuevas y Utilidad
| Columna                                 | Tipo       | Propósito                                       |
| --------------------------------------- | ---------- | ----------------------------------------------- |
| `release_year`                          | int        | Año del álbum → drift, estacionalidad           |
| `release_month` / `release_day`         | int        | Efectos estacionales                            |
| `release_decade`                        | int        | Agrupación cultural por década                  |
| `duration_min`                          | numeric    | Duración en minutos (más interpretable)         |
| `popularity_label`                      | factor     | Clasificación de popularidad (4 bins)           |
| `artist_followers_log`                  | numeric    | log1p para reducir skew                         |
| `artist_popularity_z`                   | numeric    | Z-score para normalización                      |
| `title_len` / `title_n_words`           | int        | Largo y palabras → features de NLP              |
| `title_has_feat`                        | logical    | Colaboraciones → predictor de éxito             |
| `n_genres` / `genre_primary`            | int / char | Reducción de cardinalidad, análisis multi-genre |
| `track_pos_in_album`                    | numeric    | Pistas iniciales vs finales                     |
| `album_age_years`                       | numeric    | Relación recencia ↔ popularidad                 |
| `artist_top20`                          | factor     | Reduce cardinalidad de artistas dominantes      |
| `duration_min_z` / `track_popularity_z` | numeric    | Normalizaciones para modelado                   |
Estas columnas permiten regresión, clasificación, reducción de cardinalidad y preparación para algoritmos que requieren features escaladas.

📊 Gráficas Clave
| Gráfica                               | Tipo                | Insights                                                   |
| ------------------------------------- | ------------------- | ---------------------------------------------------------- |
| `01_popularity_hist.png`              | Histograma          | Distribución de `track_popularity` y validación de bins    |
| `03_popularity_by_year_2000_2025.png` | Serie temporal      | Tendencias y drift temporal                                |
| `04_tracks_per_year_2000_2025.png`    | Barras              | Volumen anual de datos; detectar años con pocas muestras   |
| `06_correlation_heatmap_improved.png` | Heatmap             | Multicolinealidad y relaciones entre variables             |
| `02_top20_artists.png`                | Barras horizontales | Sesgo por artista; reducir cardinalidad con `artist_top20` |
Cada gráfica guía decisiones de modelado: selección de features, imputación, manejo de drift temporal y balance de clases.

⚠️ Observaciones 

Fechas y año: filtradas visualmente a 2000–2025.

Valores faltantes: strings vacíos → NA. Imputar según tipo de feature (mediana, KNN, model-based).

Cardinalidad: artist_name y artist_genres → top-k + other o embeddings.

Targeting: usar popularity_label (clasificación) o track_popularity (regresión).

Correlación: revisar correlation_matrix.csv antes de modelos lineales.

🏁 Próximos pasos

Imputación final de valores faltantes.

Selección de features (VIF, LASSO, tree-based importance).

Prueba de modelos: XGBoost, LightGBM, Random Forest.

Validación temporal: split por años para evitar data leakage.

Enriquecimiento: features externas o embeddings de texto/artistas.



