# 🚀 Desarrollo del Procesamiento en Tiempo Real con Apache Kafka y Spark Streaming

En el procesamiento en tiempo real, se implementó una simulación de sensores que generan datos continuamente mediante Kafka Producer, enviando la información a un topic llamado sensor_data.

Posteriormente, una aplicación Spark Streaming consume los mensajes en tiempo real, transformando los datos y calculando estadísticas promedio de temperatura y humedad por ventanas de tiempo.

 ## **A continuación se describe los pasos de instalación y ejecución**
 
### 1️⃣ Instalar librería de Kafka en Python
 código
```python
pip install kafka-python
```
### 2️⃣ Descargar e instalar Apache Kafka
código
```python
wget https://downloads.apache.org/kafka/3.6.2/kafka_2.13-3.6.2.tgz
tar -xzf kafka_2.12-3.5.6.tgz
sudo mv kafka_2.12-3.5.6 /opt/Kafka
```
### 3️⃣ Iniciar los servicios de Kafka

 código
 ```python
# Iniciar ZooKeeper
sudo /opt/Kafka/bin/zookeeper-server-start.sh /opt/Kafka/config/zookeeper.properties &

# Iniciar Kafka Server
sudo /opt/Kafka/bin/kafka-server-start.sh /opt/Kafka/config/server.properties &
```
### 4️⃣ Crear un tópico (topic) de Kafka

Código
```python
/opt/Kafka/bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic sensor_data
```
### 💾 Script 1: kafka_producer.py
Genera datos simulados y los envía al tópico sensor_data.
Código
```python
import time
import json
import random
from kafka import KafkaProducer

# Función que genera datos de sensores simulados
def generate_sensor_data():
    return {
        "sensor_id": random.randint(1, 10),
        "temperature": round(random.uniform(20, 30), 2),
        "humidity": round(random.uniform(30, 70), 2),
        "timestamp": int(time.time())
    }

# Configuración del productor de Kafka
producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_serializer=lambda x: json.dumps(x).encode('utf-8')
)

# Bucle infinito que envía datos cada segundo
while True:
    sensor_data = generate_sensor_data()
    producer.send('sensor_data', value=sensor_data)
    print(f"Sent: {sensor_data}")
    time.sleep(1)
```
**📍 Ejecución del productor:**
Código
```python
python3 kafka_producer.py
```
### ⚡ Script 2: spark_streaming_consumer.py
Lee los datos enviados a Kafka y calcula estadísticas en tiempo real.

Código
```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import from_json, col, window
from pyspark.sql.types import StructType, StructField, IntegerType, FloatType, TimestampType

# Crear sesión de Spark
spark = SparkSession.builder.appName("KafkaSparkStreaming").getOrCreate()
spark.sparkContext.setLogLevel("WARN")

# Esquema de los datos recibidos
schema = StructType([
    StructField("sensor_id", IntegerType()),
    StructField("temperature", FloatType()),
    StructField("humidity", FloatType()),
    StructField("timestamp", TimestampType())
])

# Leer los datos del topic sensor_data
df = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "sensor_data") \
    .load()

# Parsear el JSON recibido
parsed_df = df.select(from_json(col("value").cast("string"), schema).alias("data")).select("data.*")

# Agrupar los datos por ventana de 1 minuto y calcular promedio
windowed_stats = parsed_df.groupBy(
    window(col("timestamp"), "1 minute"), col("sensor_id")
).agg({"temperature": "avg", "humidity": "avg"})

# Mostrar los resultados en consola
query = windowed_stats.writeStream.outputMode("complete").format("console").start()
query.awaitTermination()
```
**📍 Ejecución del consumidor:**

Código
```python
spark-submit --packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.5.6 spark_streaming_consumer.py
```
### 📊 Visualización y Análisis
Los resultados se muestran en tiempo real en la consola de Spark.

Se puede monitorear el procesamiento accediendo desde tu navegador a:

Código
```python
http://192.168.1.32:4040
```
