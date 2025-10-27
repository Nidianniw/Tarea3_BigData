## 🧠 Desarrollo del procesamiento batch
En el procesamiento batch, se trabajó con un conjunto de datos almacenado en HDFS, correspondiente a los casos positivos de COVID-19 en Colombia.
Se desarrolló una aplicación en PySpark que realiza las siguientes tareas:

- Carga del conjunto de datos desde el sistema distribuido HDFS.
- Limpieza y transformación de los datos (manejo de valores nulos, conversión de tipos).
- Análisis exploratorio de datos (EDA) utilizando DataFrames.
- Generación de estadísticas descriptivas y conteos agrupados por sexo, edad y departamento.
- Almacenamiento de los resultados procesados nuevamente en HDFS.

**A continuación se describen los pasos realizados en el proyecto con los comandos utilizados.**

---

###  1️⃣ Crear la sesión de Spark

Se inicializa una sesión de Spark que permite ejecutar el procesamiento distribuido.  

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, count, when

spark = SparkSession.builder \
    .appName("Tarea3_Procesamiento_Batch_COVID19") \
    .getOrCreate()
```


###  2️⃣ Cargar el conjunto de datos desde HDFS


El dataset está almacenado en el sistema HDFS en la siguiente ruta:
```python
hdfs://localhost:9000/Tarea3/Casos_positivos_de_COVID-19_en_Colombia._20251014.csv
```

Código utilizado:
```python
df = spark.read.option("header", True).option("inferSchema", True) \
    .csv("hdfs://localhost:9000/Tarea3/Casos_positivos_de_COVID-19_en_Colombia._20251014.csv")

print("\n=== Vista previa del dataset ===\n")
df.sho 
```
Comando para ejecutar el script:
```python
spark-submit tarea3.py
```
###  3️⃣ Limpieza y transformación de datos

Se eliminan los valores nulos y se transforman columnas para garantizar la consistencia del análisis.

Código:
```python
print("\n=== Limpieza y transformación de datos ===\n")

#Conteo de valores nulo
df.select([count(when(col(c).isNull(), c)).alias(c) for c in df.columns]).show()

#Eliminar filas con valores nulos en campos clave
columnas_clave = ["Nombre departamento", "Nombre municipio", "Edad", "Sexo"]
df_clean = df.dropna(subset=columnas_clave)

#Convertir la edad a tipo entero
df_clean = df_clean.withColumn("Edad", col("Edad").cast("int"))
```
###  4️⃣ Análisis exploratorio de datos (EDA)

Se realiza un análisis descriptivo utilizando operaciones con DataFrames:
```python
print("\n=== Análisis exploratorio de datos (EDA) ===\n")

# Estadísticas descriptivas
print("Estadísticas descriptivas de la edad:")
df_clean.describe(["Edad"]).show()

# Conteo por sexo
print(" Conteo de casos por sexo:")
df_clean.groupBy("Sexo").count().orderBy("Sexo").show()

# Distribución por estado de recuperación
print("Distribución de casos por estado de recuperación:")
df_clean.groupBy("Recuperado").count().orderBy("count", ascending=False).show()

# Casos por departamento
print("Casos por departamento:")
df_clean.groupBy("Nombre departamento").count().orderBy("count", ascending=False).show(10)
```
### 5️⃣ Almacenamiento de resultados

Los datos limpios y transformados se guardan nuevamente en HDFS, en formato CSV llamado resultados.

Código:
```python
print("\n=== Guardando resultados limpios en HDFS ===\n")
df_clean.write.mode("overwrite").option("header", True).csv("hdfs://localhost:9000/Tarea3/resultados")

print("\n Proceso completado con éxito. Los resultados se guardaron en: hdfs://localhost:9000/Tarea3/resultados\n")

#Cierre de la sesión
spark.stop()
```
