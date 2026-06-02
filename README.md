# Data Quality a Open Beauty Facts 


Web: https://world.openbeautyfacts.org/


## Project Structure

- anban-explorador-data
  - data
    - openbeauty.db
    - retail_sales_for_automated.csv
  - img
    - adn_cluster.jpg
    - cluster_resultante.jpg
    - determinacion_cluster.jpg
  - notebooks
    - 01_ingestion_ETL.ipynb
    - 02_data_qa_layer.ipynb
    - 03_eda.ipynb
    - 04_cluster_ml.ipynb
    - clean_data_products.csv
    - data_quality_filter.csv
    - data_quality_no_filter.csv
    - final_dataset.csv
    - pre_dataset_products.csv
  - README.md

## instalation 

```bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```
## Phases

### Phase I -  Ingesta + Almacenamiento

Ingesta en lotes de los 50.000 productos

### Phase II - Capa de calidad de datos

 Crear metricas para la verificacion, calidad y exploracion y aplicacion de clusterizacion de la calidad de datos

#### Métricas

| Métrica | Descripción | Tipo |
|---|---|---|
| `completeness_ratio` | Proporción de campos no nulos sobre el total de `quality_columns` (7 campos) | `float [0.0, 1.0]` |
| `compl_critical_ratio` | Proporción de campos no nulos en las 4 columnas críticas: `product_name`, `brands`, `canonical_categories`, `canonical_countries` | `float [0.0, 1.0]` |
| `is_duplicate_code` | Indica si el código de producto está duplicado en el dataset | `int {0, 1}` |
| `is_duplicate_product_name` | Indica si el nombre del producto está duplicado en el dataset | `int {0, 1}` |
| `ingredients_match` | Verifica si `ingredients_n == known_ingredients_n + unknown_ingredients_n` | `int {0, 1}` |
| `ingredients_discrepancy_type` | Clasifica la discrepancia encontrada: `Perfect Match`, `Sum > total`, `Suma < total`, `Error` | `string` |
| `unknown_ingredients_ratio` | Proporción de ingredientes desconocidos sobre el total | `float [0.0, 1.0]` |
| `ingredients_quality_score` | Score de calidad de ingredientes: `1 - unknown_ingredients_ratio` | `float [0.0, 1.0]` |
| `countries_count` | Número de países asociados al producto | `int` |
| `categories_count` | Número de categorías asociadas al producto | `int` |
| `missing_country_flag` | `1` si el producto no tiene ningún país asignado | `int {0, 1}` |
| `missing_category_flag` | `1` si el producto no tiene ninguna categoría asignada | `int {0, 1}` |
| `missing_both_taxonomy_flag` | `1` si el producto no tiene ni país ni categoría | `int {0, 1}` |
| `country_languages` | Prefijos de idioma detectados en `canonical_countries` (ej. `en`, `fr`) | `set` |
| `category_languages` | Prefijos de idioma detectados en `canonical_categories` | `set` |
| `countries_mixl` | `1` si `canonical_countries` contiene más de un idioma | `int {0, 1}` |
| `categories_mixl` | `1` si `canonical_categories` contiene más de un idioma | `int {0, 1}` |
| `has_lang_conflict` | `1` si los idiomas entre países y categorías no comparten ningún prefijo en común | `int {0, 1}` |
| `taxonomy_score` | Score de consistencia taxonómica: `1 - (has_lang_conflict + countries_mixl + categories_mixl) / 3` | `float [0.0, 1.0]` |
| `product_quality_score` | Score final ponderado: `compl_critical_ratio × 0.50 + ingredients_quality_score × 0.35 + ((completeness_ratio + taxonomy_score) / 2) × 0.15`. Se fuerza a `0.0` si es duplicado, y a `-1` si el cálculo no pudo completarse. | `float {-1} ∪ [0.0, 1.0]` |

---

