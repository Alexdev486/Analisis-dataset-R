Descripción: Este README documenta el EDA realizado sobre el CSV /content/track_data_final.csv. Contiene: resumen de lo que hizo el script, cómo ejecutarlo, columnas nuevas añadidas al CSV procesado (y por qué son útiles para modelado de IA), y análisis de las 5 gráficas más representativas.

1) Resumen general del trabajo realizado

He creado un script modular y robusto EDA_Spotify_2009_2025_professional_v2.R que realiza un EDA completo y prepara el dataset para modelado. Las etapas realizadas son:

Lectura segura del CSV (readr::read_csv(..., col_types = cols(.default = "c")) para evitar coerciones sorpresivas.

Limpieza de nombres (snake_case) con janitor::clean_names.

Coerciones seguras de columnas numéricas y lógicas (helper parse_num_safe()).

Parseo robusto de fechas (album_release_date) con lubridate::parse_date_time.

Feature engineering orientado a modelado (nuevas columnas no destructivas).

Estadísticas resumidas por consola (media, sd, medianas de numéricas; conteos de categorías).

Generación de 8+ gráficas guardadas en eda_plots/ (archivos PNG).

Mapa de correlación mejorado (triángulo inferior, orden jerárquico) y export de matriz a CSV para inspección numérica.

Guardado del CSV procesado: processed_spotify_eda_final.csv.

El script fue adaptado para trabajar en un entorno tipo Colab / Rscript y tiene parámetros configurables (umbral de bins de popularity, rango de años 2000–2025, límite de variables para el heatmap).

2) Cómo ejecutar (breve)

En Colab (si usas rpy2):

Instala R / rpy2 y carga la extensión:

# en una celda %%bash
sudo apt-get update -y
sudo apt-get install -y r-base
pip install rpy2


Carga rpy2 en la celda Python:

%load_ext rpy2.ipython


Pega y ejecuta el script en una celda %%R o guarda el script en disco y ejecuta:

# desde una celda %%bash
cat > EDA_Spotify_2009_2025_professional_v2.R <<'EOF'
# (pega aquí el contenido del script)
EOF

Rscript EDA_Spotify_2009_2025_professional_v2.R


Archivos de salida:

processed_spotify_eda_final.csv — CSV procesado con nuevas columnas.

Carpeta eda_plots/ con PNGs: 01_popularity_hist.png, 02_top20_artists.png, 03_popularity_by_year_2000_2025.png, 04_tracks_per_year_2000_2025.png, 05_duration_hist.png, 06_correlation_heatmap_improved.png, etc.

eda_plots/correlation_matrix.csv — valores numéricos de la matriz de correlación (útil para inspección detallada).

3) Limpieza / Transformaciones clave realizadas

Convertí a numéricas columnas que venían como character: track_number, track_popularity, track_duration_ms, artist_popularity, artist_followers, album_total_tracks.

explicit convertido a lógico (TRUE/FALSE/NA) manejando variaciones ("True", "1", "yes", etc.).

album_release_date parseado a Date. Si hay formatos mixtos el parser intenta Ymd, ymd, Y-m-d, Y y dmy.

Strings vacíos limpiados a NA y trimws() aplicado donde procede.

Evité crear/remover columnas inesperadas y todo cambio está documentado y no sobreescribe sin comprobación los datos originales.

4) Columnas nuevas añadidas al CSV procesado (processed_spotify_eda_final.csv) y por qué

Nota: la columna original track_duration_ms sigue presente; duration_min es una columna nueva más útil para interpretación humana y modelado.

A continuación la lista de columnas añadidas / calculadas por feature_engineer() y la justificación razonada para cada una:

release_year (integer)

Año de lanzamiento del álbum.

Por qué: característica temporal clave para detectar drift, estacionalidad, y crear features como decade o lags.

release_month, release_day (integer)

Mes y día de lanzamiento.

Por qué: posibles efectos estacionales (ej. más lanzamientos en ciertos meses) y para ingeniería temporal.

release_decade (integer)

Década de lanzamiento (p.ej. 2010, 2020).

Por qué: agrupa por estéticas musicales de décadas y puede capturar cambios culturales.

duration_min (numeric)

Duración en minutos (convertida desde track_duration_ms).

Por qué: variable más interpretable que ms; útil para modelos y detectación de outliers.

popularity_label (factor con niveles regular, popular, hit, mega_hit)

Bins discretos creados a partir de track_popularity con umbrales configurables (mega_hit >= 90, hit >= 80, popular >= 60).

Por qué: útil para problemas de clasificación y para balancear/clasificar tracks por popularidad.

artist_followers_log (numeric)

log1p(artist_followers) (transformación log).

Por qué: reduce skew extremo en seguidores de artistas y estabiliza varianza (útil para regresiones y regularización).

artist_popularity_z (numeric)

Z-score de artist_popularity (normalización simple).

Por qué: feature normalizada para comparaciones e interacción con otras variables.

title_len, title_n_words (integers)

Largo en caracteres y número de palabras del track_name.

Por qué: pueden correlacionar con tipo de canción (p. ej. títulos largos en música alternativa) o ser usados por modelos de NLP / features de texto.

title_has_feat (logical)

Flag si el título contiene feat., ft. o variantes (indicador de colaboraciones).

Por qué: colaboraciones a menudo incrementan alcance/popularity; útil como predictor.

n_genres, genre_primary (integer / character)

Número de géneros registrados y el primer género listado.

Por qué: permite reducir cardinalidad (usar genre_primary o agrupar) y detectar artistas multi-genre.

track_pos_in_album (numeric)

Posición relativa del track en el álbum (track_number / album_total_tracks).

Por qué: pistas iniciales o singles suelen tener distinto comportamiento de popularity.

album_age_years (numeric)

Antigüedad del álbum en años tomando la fecha máxima del dataset como referencia.

Por qué: relaciona recencia con popularity y puede modelar decaimiento temporal.

artist_top20 (factor)

Nombre del artista si está en top-20 por número de tracks en el dataset, other en caso contrario.

Por qué: reduce cardinalidad para codificación y captura artistas dominantes (sesgos).

duration_min_z, track_popularity_z (numeric)

Normalizaciones (z-score) para inspección y uso directo en modelos que requieren features centradas/estandarizadas.

Estas columnas están pensadas para proporcionar features estructuradas y no destructivas útiles en tareas de:

regresión (predecir track_popularity),

clasificación (predecir popularity_label),

reducción de cardinalidad (artist_top20, genre_primary),

transformaciones y normalizaciones para algoritmos que requieren escalado.

5) Las 5 gráficas más representativas — explicación y análisis

A continuación se describen las cinco gráficas que considero más útiles, con su objetivo, cómo leerlas y qué decisiones de modelado se pueden tomar a partir de ellas.

Gráfica 1 — 01_popularity_hist.png

Tipo: Histograma de track_popularity.
Objetivo: visualizar la distribución del target (popularidad).
Qué muestra (interpretación):

Permite ver si track_popularity es simétrica, sesgada o multimodal.

En tu ejecución el dataset mostró una mediana cercana a ~58 y una distribución con muchos valores en rango medio (esto sugiere que la variable no es extremadamente escasa en la cola superior).

Además, se creó popularity_label con bins; el histograma ayuda a validar los umbrales seleccionados.

Uso para modelado:

Si la distribución está sesgada, considerar transformaciones o re-muestreo (SMOTE/undersampling/oversampling) para clasificación en popularity_label.

Selección de thresholds para convertir regresión a clasificación.

Gráfica 2 — 03_popularity_by_year_2000_2025.png

Tipo: Serie temporal — popularidad media por año (rango 2000–2025).
Objetivo: detectar tendencias temporales o drift en la popularidad media de las pistas por año de lanzamiento.
Qué muestra (interpretación):

Indica si las canciones lanzadas en ciertos años (o décadas) tienden a tener mayor o menor track_popularity.

Útil para detectar efectos históricos: picos o caídas que podrían deberse a cambios en la plataforma, hábitos de escucha o al sesgo de muestreo del dataset.

Uso para modelado:

Añadir variables temporales (release_year, release_decade, album_age_years) o usar time-based splitting (train/test) para evitar data leakage y evaluar drift.

Si existe drift, considerar retrain periódicos o features de fecha más complejos (polinomios, splines, lags).

Nota técnica: en el script se usa coord_cartesian(xlim = c(2000,2025)) para no eliminar observaciones fuera del rango visual y evitar warnings de scale_x_continuous(limits=...).

Gráfica 3 — 04_tracks_per_year_2000_2025.png

Tipo: Barras — número de tracks por año entre 2000 y 2025.
Objetivo: mostrar el volumen anual de datos dentro del rango de interés.
Qué muestra (interpretación):

Años con bajo número de muestras pueden resultar en estimaciones menos fiables si no se maneja (p. ej. 1950s o años con pocos registros).

Si algunos años están sobrerrepresentados (o faltan), conviene ponderar o filtrar.

Uso para modelado:

Decidir si filtrar a un rango más representativo (por ejemplo 2010–2025) o poner pesos por año.

Detectar sesgos de muestreo temporales que afecten a modelos que usan features temporales.

Gráfica 4 — 06_correlation_heatmap_improved.png

Tipo: Mapa de correlaciones (triángulo inferior, orden jerárquico).
Objetivo: detectar correlaciones fuertes y multicolinealidad entre variables numéricas.
Qué muestra (interpretación):

Agrupa variables correlacionadas juntas (orden jerárquico) y muestra coeficientes con 2 decimales, legible aun con ~20 variables.

Por ejemplo, artist_followers_log puede correlacionar fuertemente con artist_popularity y track_popularity en distintos grados; duration_min puede correlacionar con duration_min_z (obvio) o con track_popularity en baja medida.

Uso para modelado:

Evitar incluir variables altamente colineales en modelos lineales sin regularización (o usar PCA/selección).

Identificar candidatos a combinar (por ejemplo, crear energy_dance si energy y danceability están correlacionadas).

Se exporta eda_plots/correlation_matrix.csv para inspección numérica (recomendado antes de decisiones).

Técnica: el heatmap está limitado por max_vars_for_corr (por defecto 20) seleccionando top variables por varianza para mantener legibilidad. Si necesitas más variables, aumenta ese parámetro o inspecciona la CSV exportada.

Gráfica 5 — 02_top20_artists.png

Tipo: Barras horizontales — top 20 artistas por número de tracks en el dataset.
Objetivo: detectar artistas sobrerrepresentados (sesgo por artista).
Qué muestra (interpretación):

Algunos artistas tienen mucha presencia (en la ejecución observamos, por ejemplo, artistas como Taylor Swift, The Weeknd, Ariana Grande, etc. entre los top).

En la ejecución previa hubo una gran parte etiquetada other y ~20 artistas con representación significativa; en una ejecución previa resumen: other 6978, Taylor Swift 330, The Weeknd 151, ... (ejemplo extraído de un run).

Uso para modelado:

Reducir cardinalidad de artist_name (ej. top-k, target encoding, embeddings).

Tener cuidado con fugas: si un artista muy popular tiene siempre altos valores, el modelo puede “aprender” artista → popularity sin generalizar a artistas nuevos.

artist_top20 (creado) es útil para modelado: mantiene la información de artistas frecuentes y agrupa el resto como other.

6) Observaciones, limitaciones y recomendaciones prácticas

Fechas y year limits: el script fuerza visualización en 2000–2025 con coord_cartesian para no descartar puntos fuera del límite. Si quieres filtrar datos fuera del rango, filtra explícitamente filter(release_year >= 2000 & release_year <= 2025) antes de generar gráficas o antes de modelar.

Correlación: la imagen es legible hasta ~20 variables; la CSV correlation_matrix.csv es la versión canónica para inspección numérica.

Valores faltantes: el script convierte strings vacíos a NA. Para modelado hay que imputar: se recomienda mediana para features skewed (p. ej. artist_followers_log), KNN o model-based imputation si la estructura de missingness lo requiere.

Cardinalidad: artist_name y artist_genres pueden tener alta cardinalidad; estrategias recomendadas: top-k + other, target-encoding, embeddings (para redes neuronales).

Transformaciones: artist_followers se transforma con log1p para reducir el efecto de valores extremos; otras variables skewed deben examinarse (boxplots y histograms permiten ello).

Targeting: puedes usar popularity_label para clasificación (4 clases), o track_popularity para regresión. Si optas por clasificación, revisa balance de clases (en una ejecución previa fue: regular 4627, popular 3405, hit 697, mega_hit 49) y decide re-muestreo o ponderación de clases.

7) Siguientes pasos recomendados (prácticos)

Imputación y limpieza final:

Imputa album_total_tracks, track_duration y album_release_date si son importantes para tu modelo.

Revisa problems() de readr si read_csv() mostró parsing issues.

Feature selection / regularización:

Usa la matriz de correlación y VIF para eliminar multicolinealidad.

Prueba LASSO / tree-based feature importance / SHAP para seleccionar features.

Prueba de modelos:

Para clasificación: XGBoost / LightGBM con target popularity_label y encoding de artist_top20.

Para regresión: Random Forest / XGBoost en track_popularity (usar artist_followers_log, duration_min, title_has_feat, genre_primary, album_age_years).

Evaluación temporal:

Si hay drift temporal, valida con time-based split (train en años antiguos, validate en años recientes).

Enriquecimiento:

Agregar features externos (por ejemplo, datos de redes sociales, playlists, región de lanzamiento) y/o embeddings de artistas/track names.

8) Archivos generados por el script

processed_spotify_eda_final.csv — dataset final con features nuevas.

eda_plots/01_popularity_hist.png

eda_plots/02_top20_artists.png

eda_plots/03_popularity_by_year_2000_2025.png

eda_plots/04_tracks_per_year_2000_2025.png

eda_plots/05_duration_hist.png

eda_plots/06_correlation_heatmap_improved.png

eda_plots/correlation_matrix.csv — matriz numérica completa (exportada).

9) Texto de acompañamiento (breve) para presentar resultados

Realicé una inspección y limpieza robusta del dataset, añadí features orientadas a modelado (temporales, text-based, normalizaciones y flags), y generé gráficas que permiten detectar drift temporal, sesgos por artista y correlaciones relevantes entre variables. El CSV procesado está listo para la siguiente fase (imputación fina, selección de features y modelado). La matriz de correlación exportada (correlation_matrix.csv) debe revisarse antes de construir modelos lineales para evitar multicolinealidad.
