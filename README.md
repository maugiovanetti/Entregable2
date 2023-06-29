
## Descripción del Código
El código proporcionado es un script en Python que utiliza PySpark y la API de OpenWeatherMap para obtener datos climáticos de diferentes provincias de Argentina. Luego, almacena estos datos en una tabla de Amazon Redshift.



## Requisitos

- Python 3.x.
- Las bibliotecas `pyspark`, `googletrans==4.0.0-rc1` y `psycopg2` instaladas. 




### 1. Importación de bibliotecas
En esta sección, se importan las bibliotecas necesarias para el código, como pyspark.sql, googletrans y psycopg2. Estas bibliotecas proporcionan funcionalidades adicionales para trabajar con Spark, traducción de texto y conexión a bases de datos PostgreSQL, respectivamente.

```python
Copy code
from pyspark.sql.types import StructType, StructField, StringType, DoubleType, IntegerType
from googletrans import Translator
import psycopg2
from psycopg2 import sql
```

### 2. Creación de una instancia de SparkSession
En esta sección, se crea una instancia de SparkSession utilizando SparkSession.builder.getOrCreate(). SparkSession es la entrada principal para interactuar con Spark y permite ejecutar operaciones en un entorno distribuido.

```python
spark = SparkSession.builder.getOrCreate()

```

### 3. Configuración de la API key de OpenWeatherMap
En esta sección, se define la API key de OpenWeatherMap en la variable api_key. Esta clave es necesaria para realizar solicitudes a la API y obtener los datos climáticos.

```python

api_key = "cb3c7af6f8a3112d069b2cd42e3d2651"

```
### 4. Definición de las provincias
En esta sección, se define una lista llamada provincias que contiene las provincias de Argentina para las cuales se obtendrán los datos climáticos. Puedes modificar esta lista según tus necesidades.

```python

provincias = ["Buenos Aires", "Córdoba", "Santa Fe", "Mendoza", "Tucumán", "Parana", "Salta", "Resistencia", "Corrientes", "Misiones", "Santiago del Estero", "San Juan", "San Salvador de Jujuy", "Viedma", "Formosa", "Neuquén", "Rawson", "San Luis", "Catamarca", "La Rioja, AR", "Santa Rosa, AR", "Rio Gallegos", "Ushuaia"]

```

### 5. Obtención y procesamiento de los datos climáticos
En esta sección, se itera sobre la lista de provincias y se realiza una solicitud a la API de OpenWeatherMap para obtener los datos climáticos de cada provincia. Los datos se almacenan en formato JSON y se procesan para extraer la información relevante, a la vez se procede a la trasnfromacion de la descripcion al idioma españal y a expresar como porcentaje los datos obetenidos por humedad.

```python

datos_clima = []

translator = Translator()

for provincia in provincias:
    url = f"https://api.openweathermap.org/data/2.5/weather?q={provincia}&appid={api_key}&units=metric"
    respuesta = requests.get(url)
    data = respuesta.json()
    
    dato_clima = Row(
        ID=len(datos_clima)+1,
        longitud=data["coord"]["lon"],
        latitud=data["coord"]["lat"],
        descripcion = translator.translate(data["weather"][0]["description"], src='en', dest='es').text,
        temperatura=float(data["main"]["temp"]),
        temperatura```python
    temperatura_max=float(data["main"]["temp_max"]),
    temperatura_min=float(data["main"]["temp_min"]),
    presion=float(data["main"]["pressure"]),
    humedad="{}%".format(data["main"]["humidity"]),
    velocidad_viento=float(data["wind"]["speed"]),
    nombre=data["name"],
    zona_horaria=data["timezone"],
    pais=data["sys"]["country"]
)

datos_clima.append(dato_clima)


```

### 6. Creación y visualización del DataFrame
En esta sección, se define el esquema del DataFrame utilizando StructType y StructField. Luego, se crea el DataFrame utilizando el esquema y los datos climáticos obtenidos. Finalmente, se muestra el DataFrame por pantalla utilizando dataframe.show().


```python

esquema = StructType([
    StructField("ID", IntegerType(), nullable=False),
    StructField("longitud", DoubleType(), nullable=True),
    StructField("latitud", DoubleType(), nullable=True),
    StructField("descripcion", StringType(), nullable=True),
    StructField("temperatura", DoubleType(), nullable=True),
    StructField("temperatura_max", DoubleType(), nullable=True),
    StructField("temperatura_min", DoubleType(), nullable=True),
    StructField("presion", DoubleType(), nullable=True),
    StructField("humedad", StringType(), nullable=True),
    StructField("velocidad_viento", DoubleType(), nullable=True),
    StructField("nombre", StringType(), nullable=True),
    StructField("zona_horaria", StringType(), nullable=True),
    StructField("pais", StringType(), nullable=True)
])

dataframe = spark.createDataFrame(datos_clima, schema=esquema)

dataframe.show(dataframe.count(), truncate=False)
```

### 7. Conexión a Amazon Redshift y creación de la tabla
En esta sección, se establecen los datos de conexión a Amazon Redshift, como el host, puerto, base de datos, usuario, contraseña y esquema. Luego, se crea una conexión a Amazon Redshift y se crea una tabla utilizando SQL con el comando CREATE TABLE IF NOT EXISTS.

```python

host = "data-engineer-cluster.cyhh5bfevlmn.us-east-1.redshift.amazonaws.com"
port = 5439
database = "data-engineer-database"
user = "mau_giovanetti_coderhouse"
password = "5K6m1tR3h9"
schema = "mau_giovanetti_coderhouse"

conn = psycopg2.connect(
    host=host,
    port=port,
    dbname=database,
    user=user,
    password=password
)

cursor = conn.cursor()

table_name = "clima"
create_table_query = """
    CREATE TABLE IF NOT EXISTS {}.{} (
        ID INT PRIMARY KEY,
        longitud DOUBLE PRECISION,
        latitud DOUBLE PRECISION,
        descripcion VARCHAR(255),
        temperatura DOUBLE PRECISION,
        temperatura_max DOUBLE PRECISION,
        temperatura_min DOUBLE PRECISION,
        presion DOUBLE PRECISION,
        humedad VARCHAR(255),
        velocidad_viento DOUBLE PRECISION,
        nombre VARCHAR(255),
        zona_horaria VARCHAR(255),
        pais VARCHAR(255)
    )
""".format(schema, table_name)

cursor.execute(create_table_query)
conn.commit()
```
### 8. Inserción de datos en la tabla
En esta sección, se insertan los datos climáticos en la tabla de Amazon Redshift. Se verifica si los datos ya existen en la tabla antes de realizar la inserción.

```python

for dato_clima in datos_clima:
    id_clima```python
    id_clima = dato_clima.ID

    select_query = sql.SQL("""
        SELECT COUNT(*) FROM {}.{}
        WHERE ID = %s
    """).format(sql.Identifier(schema), sql.Identifier(table_name))

    cursor.execute(select_query, (id_clima,))
    result = cursor.fetchone()

    if result[0] == 0:
        insert_query = sql.SQL("""
            INSERT INTO {}.{} (ID, longitud, latitud, descripcion, temperatura, temperatura_max, temperatura_min, presion, humedad, velocidad_viento, nombre, zona_horaria, pais)
            VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
        """).format(sql.Identifier(schema), sql.Identifier(table_name))

        cursor.execute(insert_query, dato_clima)
        conn.commit()
    else:
        print("Los datos con ID {} ya existen en la tabla.".format(id_clima))

```


### 9. Cierre de la conexión
En esta sección, se cierra el cursor y la conexión a Amazon Redshift.

```python

cursor.close()
conn.close()

```

