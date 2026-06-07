
# Data Quality a Open Beauty Facts 

## Problema y Solución

***Problema***

**Open Beauty Facts** es una plataforma colaborativa donde cualquier persona puede registrar productos cosméticos y dermocosméticos. Esa apertura es su mayor valor, pero también su mayor riesgo: **nadie verifica que los datos subidos sean fiables**.

El resultado es un catálogo de 50.000 productos donde conviven registros perfectamente documentados con productos sin categoría, ingredientes sin identificar, nombres duplicados y datos en idiomas mezclados — sin que el usuario ni el sistema sepa cuáles son cuáles.

El objetivo es responder:
> **¿Cómo identificar qué productos de Open Beauty Facts tienen datos suficientemente fiables para usarse en análisis o modelos futuros?**
> 

***Solución — MVP***

Un **pipeline automatizado en Python** de 4 fases que convierte datos crudos en un catálogo evaluado, segmentado y clasificable:

| Fase | Notebook | Qué hace |
|---|---|---|
| **I — Ingesta** | `01_ingestion_ETL.ipynb` | Descarga 50.000 productos de la API en 10 lotes paginados y los almacena en SQLite (`raw_products`) |
| **II — Data Quality** | `02_data_qa_layer.ipynb` | Calcula 20 métricas por producto (completitud, duplicidad, ingredientes, taxonomía) y genera un `product_quality_score` de 0.0 a 1.0 |
| **III — EDA** | `03_eda.ipynb` | Explora distribuciones, correlaciones y perfiles de calidad por marca para entender el catálogo |
| **IV — Clustering + Clasificador** | `04_cluster_ml.ipynb` | Segmenta los productos en 5 grupos de calidad con KMeans y entrena un Random Forest para clasificar nuevos registros |

### Los 5 segmentos resultantes

| Cluster | Perfil |
|---|---|
| Bien documentados | Alta calidad, ingredientes conocidos, datos completos |
|  Incompletos | Baja completitud, sin categoría, sin país |
|  Calidad mediocre | Datos parciales, ingredientes con problemas |
|  Populares / duplicados | Alto engagement pero nombres repetidos |
|  Sin categorizar | Sin taxonomía asignada, datos mínimos |


![Clusters resultante](/img/cluster_resultante.jpg)

## Conclusiones

El catálogo de Open Beauty Facts presenta problemas de calidad más profundos 
de lo esperado: **35.474 de los 50.000 productos extraídos (70,9%) recibieron 
score -1** por ausencia total de ingredientes declarados, quedando fuera de 
cualquier análisis fiable.

Del 29,1% restante (14.526 productos con score válido), la calidad no es 
uniforme. El clustering KMeans identificó 5 perfiles diferenciados — desde 
productos bien documentados con scores superiores a 0.8, hasta registros sin 
taxonomía ni datos mínimos. Esto confirma que **el catálogo no puede tratarse 
como un dataset homogéneo**.

Entre los problemas más frecuentes detectados por la capa de Data Quality:
- ~2.800 productos con nombre duplicado
- ~1.600 con conflicto de idioma entre países y categorías
- ~4.000 con discrepancia `Sum > total` en el conteo de ingredientes

**Colgate y Mercadona** lideran en calidad mediana de datos (~0.85), 
mientras que los productos sin marca identificada (`Unknown`) tienen 
consistentemente los scores más bajos.

El pipeline es reutilizable: ante una nueva descarga del dataset, los 4 
notebooks ejecutados en orden reproducen la evaluación completa y el 
clasificador Random Forest asigna automáticamente cada producto a su 
segmento de calidad.

## Estructura del proyecto

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


## Instalación

**1. Clonar el repositorio**
```bash
git clone https://github.com/MayeLabs/anban_explorador_data.git
cd anban_explorador_data
```

**2. Crear y activar el entorno virtual**

Windows:
```bash
python -m venv venv
.\venv\Scripts\activate
```
Linux/Mac:
```bash
python -m venv venv
source venv/bin/activate
```

**3. Instalar dependencias**
```bash
pip install -r requirements.txt
```
**4. Ejecutar notebooks**
```bash
jupyter lab
```

## Dataset — Open Beauty Facts 

* Fuente: API pública de Open Beauty Facts  
* https://world.openbeautyfacts.org/cgi/search.pl

***Descripción General***

| | |
|---|---|
| **Proveedor** | Open Beauty Facts (comunidad colaborativa) |
| **Acceso** | API REST pública, sin autenticación |
| **Formato** | `application/json` |
| **Volumen extraído** | ~50.000 productos (1.000 páginas × 10 lotes) |
| **Almacenamiento** | SQLite — tabla `raw_products` |
| **Licencia** | Open Database License (ODbL) |
| **Nota** | Los datos son de fuente pública y no contienen información personal identificable |

---

***Campos extraídos de la API***
<details open>
<summary></summary>
 
| Nº | Campo | Descripción | Tipo | Requerido |
|---|---|---|---|---|
| 0 | `_id` | Identificador interno del producto | `string` | ✅ |
| 1 | `code` | Código de barras del producto | `string` | ✅ |
| 2 | `rev` | Número de revisión del registro | `int` | ❌ |
| 3 | `update_key` | Clave de la última actualización | `string` | ❌ |
| 4 | `brands` | Marca(s) del producto | `string` | ❌ |
| 5 | `product_name` | Nombre del producto | `string` | ❌ |
| 6 | `product_type` | Tipo de producto (siempre `beauty` en este dataset) | `string` | ❌ |
| 7 | `countries` | Países donde se comercializa (texto libre) | `string` | ❌ |
| 8 | `countries_tags` | Países en formato de etiqueta normalizada | `list` | ❌ |
| 9 | `countries_hierarchy` | Jerarquía canónica de países (`en:france`, etc.) | `list` | ❌ |
| 10 | `categories` | Categorías del producto (texto libre) | `string` | ❌ |
| 11 | `categories_hierarchy` | Jerarquía canónica de categorías | `list` | ❌ |
| 12 | `product_quantity` | Cantidad del producto (valor numérico) | `float` | ❌ |
| 13 | `product_quantity_unit` | Unidad de la cantidad (`ml`, `g`, etc.) | `string` | ❌ |
| 14 | `quantity` | Cantidad tal como aparece en el envase | `string` | ❌ |
| 15 | `ingredients_n` | Número total de ingredientes declarados | `int` | ❌ |
| 16 | `known_ingredients_n` | Número de ingredientes reconocidos por la plataforma | `int` | ❌ |
| 17 | `unknown_ingredients_n` | Número de ingredientes no reconocidos | `int` | ❌ |
| 18 | `completeness` | Score de completitud asignado por Open Beauty Facts (0–1) | `float` | ❌ |
| 19 | `scans_n` | Número total de escaneos del producto | `int` | ❌ |
| 20 | `unique_scans_n` | Número de usuarios únicos que escanearon el producto | `int` | ❌ |
| 21 | `popularity_tags` | Etiquetas de popularidad asignadas por la plataforma | `list` | ❌ |
| 22 | `created_t` | Timestamp de creación del registro | `int` (Unix) | ❌ |
| 23 | `last_modified_t` | Timestamp de última modificación | `int` (Unix) | ❌ |
| 24 | `last_updated_t` | Timestamp de última actualización | `int` (Unix) | ❌ |
| 25 | `creator` | Usuario que creó el registro | `string` | ❌ |
| 26 | `last_editor` | Último usuario que editó el registro | `string` | ❌ |
| 27 | `last_modified_by` | Usuario responsable de la última modificación | `string` | ❌ |
| 28 | `data_quality_tags` | Etiquetas de problemas de calidad detectados por la plataforma | `list` | ❌ |
| 29 | `page` | Página de la API de la que se extrajo el registro | `int` | ✅ |
| 30 | `batch_id` | Lote de descarga al que pertenece el registro (1–10) | `int` | ✅ |
| 31 | `dtinserted` | Fecha y hora de inserción en la base de datos local | `datetime` | ✅ |

</details>

***Data Quality Layer - Métricas***

<details open>
<summary></summary>
 
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
</details>


