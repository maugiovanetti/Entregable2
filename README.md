# Entregable2
Entregable de la Semana 2 

Descripción del Código
El código proporcionado es un script en Python que utiliza PySpark y la API de OpenWeatherMap para obtener datos climáticos de diferentes provincias de Argentina. Luego, almacena estos datos en una tabla de Amazon Redshift.

El código se divide en las siguientes secciones:

1. Importación de bibliotecas
En esta sección, se importan las bibliotecas necesarias para el código, como pyspark.sql, googletrans y psycopg2. Estas bibliotecas proporcionan funcionalidades adicionales para trabajar con Spark, traducción de texto y conexión a bases de datos PostgreSQL, respectivamente.


```python
from pyspark.sql.types import StructType, StructField, StringType, DoubleType, IntegerType
from googletrans import Translator
import psycopg2
from psycopg2 import sql



### 2. Creación de una instancia de SparkSession

En esta sección, se crea una instancia de `SparkSession` utilizando `SparkSession.builder.getOrCreate()`. `SparkSession` es la entrada principal para interactuar con Spark y permite ejecutar operaciones en un entorno distribuido.

```markdown
```python
spark = SparkSession.builder.getOrCreate()
