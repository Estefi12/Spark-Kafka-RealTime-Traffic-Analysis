# Análisis de Congestión Vehicular en Tiempo Real con Spark y Kafka 


Este proyecto implementa una arquitectura de Big Data en Streaming diseñada para el monitoreo en tiempo real de la congestión vehicular. 
Utiliza una pila tecnológica distribuida para gestionar la alta velocidad de los datos de tráfico y proporcionar análisis de baja latencia.

## 1. Definición del Problema y Solución

El objetivo es resolver el desafío de la latencia en la gestión de tráfico urbano.

Problema: Los sistemas tradicionales no pueden procesar el volumen de datos de sensores de tráfico lo suficientemente rápido para emitir alertas de congestión instantáneas.

Solución: Utilizar Spark Streaming para calcular el Volumen Promedio de Vehículos y la Velocidad Media por cada Cruce (Junction ID) en ventanas de tiempo continuas (60 segundos).

## 2. Arquitectura de la Solución

El sistema se basa en un flujo distribuido de baja latencia: Productor Python $\rightarrow$ Apache Kafka $\rightarrow$ Spark Streaming $\rightarrow$ Salida (Consola/Web UI).

Apache Kafka: Funciona como el Buffer de Ingestión. Gestiona el flujo continuo y de alta velocidad de datos (simulados) y asegura la resiliencia del stream.

Spark Streaming: Es el Motor de Procesamiento Distribuido. Consume los datos de Kafka en micro-batches y realiza el análisis en paralelo.
La Web UI de Spark (puerto 4040) es la prueba de que los Jobs están activos.

Análisis por Ventana: Es la lógica central. Spark aplica una función de Agregación (AVG) de las métricas de tráfico (vehicle_count y avg_speed) en intervalos fijos de un minuto.
El Stage 3 de Spark (observado en las capturas) ejecuta esta lógica.

## 3. Modelo de Datos del Stream

El flujo continuo de datos (sensor_data en Kafka) simula la telemetría de un sensor de tráfico real. El stream contiene las siguientes variables:

junction_id: Identificador del Cruce/Sensor. Es la clave para la agregación (GROUP BY).
vehicle_count: Conteo de Vehículos. Es la métrica principal a promediar (Alineado con el temperature del código original).
timestamp: Momento de la Lectura. Es la base para la función de Ventana de 1 Minuto de Spark.

## 4. Guía de Ejecución Paso a Paso

### Paso 0: Pre-requisitos
Asegúrese de que Kafka/Zookeeper y Apache Spark estén instalados, y que las librerías kafka-python/pyspark estén disponibles.

### Paso 1: Iniciar Servicios y Crear Topic *
* Iniciar Kafka y Zookeeper
$ zookeeper-server-start.sh config/zookeeper.properties
$ kafka-server-start.sh config/server.properties

* Crear el topic 'sensor_data'
$ kafka-topics.sh --create --topic sensor_data --bootstrap-server localhost:9092

### Paso 2: Ejecutar el Productor (Simulación)
Abra una terminal y ejecute el simulador de tráfico:
$ python producer/kafka_producer.py

### Paso 3: Ejecutar el Consumidor de Spark (Análisis)
* Abra una terminal separada y ejecute el script de análisis de Spark:
* Ajuste la versión del package si es necesario
* $ spark-submit --packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.5.0 consumer/spark_traffic_analyzer.py

El resultado del promedio por minuto se imprimirá en esta consola, y la Web UI de Spark (localhost:4040) mostrará el procesamiento en curso.
