# 📦 Proyecto DataCo: Smart Supply Chain for Big Data Analysis

## 📖 1. Introducción y Contexto del Proyecto

Este proyecto nace con el objetivo de analizar y optimizar la cadena de suministro global de **DataCo Global**, una empresa multinacional que combina el sector retail con servicios avanzados de suministro y logística tecnológica. Utilizando datos masivos de sus operaciones comerciales, este trabajo implementa un flujo completo de Ciencia de Datos y Machine Learning para dar solución a ineficiencias logísticas y financieras que afectan directamente la experiencia del cliente y la rentabilidad de la compañía.

### 📊 Origen de los Datos
Los datos analizados en este proyecto son reales y han sido obtenidos de la plataforma **Kaggle**, correspondientes al dataset público:
* **Enlace oficial:**[ [DataCo SMART SUPPLY CHAIN FOR BIG DATA ANALYSIS](https://kaggle.com)](https://www.kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis)

### ⏱️ Alcance Temporal y Cobertura
El conjunto de datos comprende un histórico operativo continuo que abarca **desde enero de 2015 hasta febrero de 2018**. Durante este periodo, se registran transacciones globales en áreas clave:
* **Sistemas de pago empleados:** Transferencias bancarias, pagos con tarjetas de débito/crédito, efectivo y plataformas electrónicas.
* **Flujo logístico:** Monitoreo desde que el cliente realiza el pedido hasta que el producto llega a su destino final, evaluando los tiempos de envío reales frente a las fechas programadas de entrega en múltiples regiones internacionales.

---

## 📊 2. Estructura y Flujo de Trabajo del Repositorio

El ciclo de vida del proyecto está completamente desarrollado y documentado dentro de la carpeta `notebooks/` a través de los siguientes componentes principales:

### A. Ingesta, Limpieza y Procesamiento de Datos (EDA)
* **Documento:** `notebooks/Análisis Exploratorio de Datos - DataCo Smart Supply Chain.ipynb`
* **Flujo de Trabajo:** 
  * Se realiza la carga del dataset bruto (`DataCoSupplyChainDataset.csv`) aplicando una normalización homogénea de todas las columnas a formato *snake_case*.
  * Se realiza un análisis de calidad de datos, donde se detecta y elimina la columna `product_description` (100% de registros nulos) y la columna `order_zipcode` (más del 86% de valores ausentes, inviable para imputación).
  * Se corrigen los tipos de datos transformando las fechas de formato texto a objetos temporales (`datetime64`) y optimizando las variables categóricas.
  * **Conclusiones del EDA:** Se identifican los mercados líderes en ventas (Europa y LATAM) y las categorías reinantes (Fishing y Cleats). Se detecta una anomalía: el 18.71% de las transacciones registran un beneficio económico negativo (pérdidas) y un 54.83% de los envíos presentan un retraso real frente a lo programado, estableciendo los dos problemas de negocio a resolver mediante Machine Learning. El resultado final se exporta limpio en `data/processed/DataCoSupplyChain_clean.csv`.

### B. Modelado Predictivo (Machine Learning)
* **Documento:** `notebooks/Machine Learning - DataCo Smart Supply Chain.ipynb`
* **Flujo de Trabajo:** Se aíslan las variables predictoras evitando la fuga de información (*data leakage*) y se entrenan modelos de clasificación para los dos problemas de negocio detectados:
  * **Modelo de Operaciones con Pérdidas (NO VIABLE):** Se entrenaron algoritmos de Regresión Logística y XGBoost Classifier para predecir si una venta generaría pérdidas. Los modelos resultaron **no viables** debido a una desconexión matemática entre la logística y las finanzas del dataset (XGBoost obtuvo un **ROC-AUC de 0.5161**, equivalente al azar). Se concluyó técnicamente que el dataset carece de variables de costes reales para hacer esta predicción viable en producción.
  * **Modelo de Riesgo de Retrasos en Envíos (VIABLE Y DEFINITIVO):** Se entrenó un modelo para anticipar retrasos logísticos utilizando únicamente variables disponibles antes del despacho. Tras una optimización de hiperparámetros mediante `GridSearchCV` (C = 100) y un ajuste del umbral de clasificación a **0.40** para priorizar la detección de incidentes, el modelo definitivo de **Regresión Logística obtuvo un Recall del 78.32% y un ROC-AUC de 0.7824**. Este modelo es altamente viable para que la empresa intervenga los envíos críticos antes de que se venza el plazo comprometido. Las predicciones completas fueron exportadas a `ml_delay_predictions.csv`.

### C. Diseño y Carga de la Base de Datos Relacional (SQL)
* **Documento:** `notebooks/Base de datos y SQL - DataCo Smart Supply Chain.ipynb`
* **Flujo de Trabajo:** 
  * Se realiza un análisis de unicidad y cardinalidad, determinando que `order_item_id` identifica de forma única cada una de las 180,519 líneas del dataset, actuando como clave primaria (PK).
  * Se eliminan las redundancias y columnas duplicadas detectadas en las parejas (`customer_id` / `order_customer_id`), (`product_card_id` / `order_item_cardprod_id`) y (`category_id` / `product_category_id`).
  * Se crea una base de datos local utilizando **SQLite** (`dataco_supply_chain.db`) y se estructuran las tablas bajo un modelo relacional de Snowflake/Estrella: `customers`, `orders`, `order_items`, `products`, `categories` y `departments`.
  * Se integra con éxito la tabla de predicciones generada por el modelo definitivo de Machine Learning (`ml_delay_predictions`), vinculada directamente a las líneas de pedido.
  * Se realizan consultas SQL de validación avanzada (`JOIN`, `GROUP BY`, `SUM`) confirmando que la ingesta de los datos concuerda al 100% con el análisis.

### D. Visualización Avanzada (Power BI)
* **Flujo de Trabajo:** Para la construcción del cuadro de mandos interactivo final en Power BI, no se realiza una importación estática de archivos planos. En su lugar, **se conecta Power BI directamente a la base de datos relacional mediante un script de Python**. Este script ejecuta las transformaciones de código necesarias en tiempo real, cruzando la estructura del negocio de DataCo con las probabilidades predictivas de retraso calculadas por el modelo definitivo de Machine Learning, logrando un panel de control de Business Intelligence dinámico, automatizado y orientado a la auditoría de riesgos logísticos.

---

## 💾 3. Descarga y Configuración de Datos


1. Descargue el conjunto de datos bruto directamente desde el enlace oficial de Kaggle provisto en la introducción.
2. Coloque el archivo descargado dentro de la ruta `data/raw/DataCoSupplyChainDataset.csv` en su entorno local.
3. Ejecute los cuadernos en orden cronológico (`Análisis` -> `Machine Learning` -> `Base de Datos`) para reconstruir la base de datos local de SQLite y las predicciones finales.
4. Asegúrese de que la estructura local de su proyecto mantenga el siguiente orden para que las rutas relativas de los cuadernos funcionen correctamente:

```text
PROYECTO DataCo/
├── data/
│   ├── raw/
│   │   └── DataCoSupplyChainDataset.csv
│   └── processed/
│       ├── DataCoSupplyChain_clean.csv
│       └── dataco_supply_chain.db
├── notebooks/
│   ├── Análisis Exploratorio de Datos - DataCo Smart Supply Chain.ipynb
│   ├── Machine Learning - DataCo Smart Supply Chain.ipynb
│   └── Base de datos y SQL - DataCo Smart Supply Chain.ipynb
├── .gitignore
├── ml_delay_predictions.csv
├── requirements.txt
└── README.md
```

---

## 🛠️ 4. Requisitos e Instalación del Entorno

Para recrear el entorno virtual con las mismas versiones de librerías utilizadas en el desarrollo (`pandas`, `numpy`, `scikit-learn`, `xgboost`, `sqlite3`, `matplotlib`, etc.), ejecute los siguientes comandos en su terminal:

1. **Crear y activar el entorno virtual:**
   ```bash
   python -m venv .venv
   .venv\Scripts\activate
   ```
2. **Instalar dependencias desde la receta del proyecto:**
   ```bash
   pip install -r requirements.txt
   ```
