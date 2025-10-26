# 🧩 Procesamiento Batch de Casos Positivos de COVID-19 en Colombia

## 📘 Descripción del proyecto

El presente trabajo corresponde al desarrollo de la tarea 3, donde se evidencia el como fue el desarrollo del procesamiento en batch haciendo uso de Apache Spark, con el objetivo de analizar un conjunto de datos reales sobre los casos positivos de COVID-19 en Colombia. El procesamiento se realiza sobre un archivo CSV almacenado en HDFS, y comprende las etapas de carga limpieza, transformación, análisis exploratorio de datos (EDA) y almacenamiento de los resultados procesados.



## 🧠 Desarrollo del procesamiento batch

A continuación se describen los pasos realizados en el proyecto con los **comandos utilizados**.

---

### 🟢 1️⃣ Crear la sesión de Spark

Se inicializa una sesión de Spark que permite ejecutar el procesamiento distribuido.  

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, count, when

spark = SparkSession.builder \
    .appName("Tarea3_Procesamiento_Batch_COVID19") \
    .getOrCreate()






