# 🚀 Pipeline de Big Data: De Cassandra a ClickHouse (UNEG)

Este proyecto implementa un ecosistema de datos completo diseñado para manejar volúmenes masivos. Automatiza el flujo de **100,000 registros** desde un almacenamiento NoSQL de alta disponibilidad (**Cassandra**) hacia un motor OLAP optimizado para analítica (**ClickHouse**).

---

## 🏗️ Arquitectura del Sistema

El pipeline sigue una arquitectura de tres capas diseñada para la eficiencia:

1. **Capa de Ingesta (Data Lake):** **Apache Cassandra** recibe datos atómicos (IDs, precios, fechas) simulando una base de datos transaccional de alta velocidad.
2. **Capa de Procesamiento (ETL):** Un motor de **Python + Pandas** (con parches de compatibilidad para Python 3.13) extrae, limpia y agrega los datos, transformando registros crudos en métricas de negocio.
3. **Capa Analítica (Data Warehouse):** **ClickHouse** almacena los datos procesados, permitiendo consultas complejas y reportes gerenciales en milisegundos.

---

## 🛠️ Requisitos Previos

Antes de comenzar, asegúrate de tener instalado:

1. **Docker Desktop:** Esencial para orquestar los contenedores de las bases de datos.
2. **Python 3.10+:** Lenguaje base del pipeline.
3. **Bibliotecas de Python:**
   ```bash
   pip install cassandra-driver clickhouse-connect pandas numpy
   ```

---

## 🚀 Configuración y Ejecución

### 1. Levantar Infraestructura

Desde la raíz del proyecto, ejecuta:
```bash
docker-compose up -d
```
**Nota:** El contenedor de Cassandra puede tardar hasta 45 segundos en estar listo para recibir conexiones.

### 2. Ejecución del Pipeline

Abre el archivo `Pipeline_BigData_UNEG.ipynb` y ejecuta las celdas en orden. El flujo realizará:

- Configuración de seguridad y parches de compatibilidad.
- Creación de esquemas en Cassandra y ClickHouse.
- Generación e ingesta masiva de 100,000 registros.
- Cálculo de agregaciones y carga en el Data Warehouse.

---

## 📊 Validación de Datos

Para verificar la integridad de los datos en cada etapa:

### A. Auditoría en Cassandra (Datos Crudos)

Verifica que los registros individuales existan con el formato correcto:
```bash
docker exec -it cassandra_db cqlsh -e "SELECT * FROM proyecto_bigdata.ventas_crudas LIMIT 5;"
```

### B. Auditoría en ClickHouse (Reporte Final)

Verifica los totales consolidados (Requiere credenciales configuradas):
```bash
docker exec -it clickhouse_dw clickhouse-client --user default --password 1234 -q "SELECT * FROM ventas_resumen FORMAT PrettyCompact;"
```

**Nota:** Estos comandos se ejecutan directamente en la terminal de tu sistema operativo (Windows/Linux/Mac), no dentro de Jupyter. Asegúrate de que `docker-compose up` se haya ejecutado correctamente antes de probarlos.

---

## ⚠️ Notas de Implementación (Solución de Problemas)

- **Compatibilidad Python 3.13:** El proyecto incluye un "Mock" del módulo asyncore para evitar errores de importación en el driver de Cassandra en versiones modernas de Python.
- **Seguridad:** ClickHouse está configurado con autenticación (user: default, pass: 1234).
- **Memoria:** Si Docker falla al iniciar, aumenta el límite de RAM en Docker Desktop -> Settings -> Resources (Se recomiendan al menos 4GB).

# 🚀 Pipeline de Big Data: De Cassandra a ClickHouse (UNEG) - Fases 2 y 3

Este documento detalla la implementación de la capa de ingesta y transformación del proyecto.

## 🛠️ Fase 2: Ingesta Masiva de Datos (NoSQL - Cassandra)

El objetivo de esta fase es poblar la tabla `ventas_crudas` con 100,000 registros ficticios para simular un entorno transaccional real.

### 1. Preparación del Entorno en Jupyter

Antes de iniciar, es necesario instalar el controlador de Cassandra dentro del contenedor de Jupyter:

```python
!pip install cassandra-driver pandas numpy

2. Script de Ingesta Masiva (Tarea 2.1 y 2.2)
Ejecuta el siguiente código en una celda de Jupyter para generar e insertar los datos. Se incluye un "Mock" de compatibilidad para evitar errores en versiones recientes de Python:

import sys
from cassandra.cluster import Cluster
from uuid import uuid4
from datetime import datetime, timedelta
import numpy as np

# Parche de compatibilidad para Python 3.13+
if 'asyncore' not in sys.modules:
    import types
    sys.modules['asyncore'] = types.ModuleType('asyncore')

# Conexión al clúster (Servicio: cassandra_db)
cluster = Cluster(['cassandra_db']) 
session = cluster.connect('proyecto_bigdata')

# Configuración de los 100,000 registros
n_registros = 100000
categorias = ['Electrónica', 'Ropa', 'Hogar', 'Alimentos', 'Deportes']
query = session.prepare("""
    INSERT INTO ventas_crudas (id_venta, fecha_venta, id_producto, categoria, monto_total, id_cliente)
    VALUES (?, ?, ?, ?, ?, ?)
""")

print("🚀 Iniciando ingesta masiva...")
for i in range(n_registros):
    session.execute(query, (
        uuid4(), 
        datetime.now().date() - timedelta(days=np.random.randint(0, 60)),
        f"PROD-{np.random.randint(100, 999)}",
        np.random.choice(categorias),
        float(np.round(np.random.uniform(5.0, 1000.0), 2)),
        f"CLI-{np.random.randint(1000, 5000)}"
    ))
print("✨ Ingesta completada.")

3. Validación de Ingesta (Tarea 2.3)
Ejecuta este comando en la terminal de tu sistema para confirmar el éxito de la operación:

docker exec -it cassandra_db cqlsh -e "SELECT COUNT(*) FROM proyecto_bigdata.ventas_crudas;"

Resultado esperado: 100,000 registros.

⚡ Fase 3: Procesamiento Paralelo y Transformación (Spark)
En esta fase se implementa la lógica de negocio (ETL) utilizando PySpark para transformar datos crudos en métricas analíticas.

1. Inicialización de Spark con Conectores (Tarea 3.1)
Para evitar el error DATA_SOURCE_NOT_FOUND, es obligatorio descargar el conector de Cassandra al iniciar la sesión:

from pyspark.sql import SparkSession
from pyspark.sql.functions import col, sum, count, round

spark = SparkSession.builder \
    .appName("Pipeline_BigData_Fase3") \
    .config("spark.jars.packages", "com.datastax.spark:spark-cassandra-connector_2.12:3.5.0") \
    .config("spark.cassandra.connection.host", "cassandra_db") \
    .getOrCreate()

2. Lectura y Limpieza (Tarea 3.1 y 3.3)
Leemos los datos desde Cassandra utilizando la Partition Key (fecha_venta) para optimizar el paralelismo:

# Lectura de la tabla cruda
df_crudo = spark.read \
    .format("org.apache.spark.sql.cassandra") \
    .options(table="ventas_crudas", keyspace="proyecto_bigdata") \
    .load()

# Limpieza: Eliminar montos nulos o negativos
df_limpio = df_crudo.filter(col("monto_total") > 0)

3. Lógica de Agregación (Tarea 3.2)
Transformamos los datos individuales en un resumen por fecha y categoría:

df_resumen = df_limpio.groupBy("fecha_venta", "categoria") \
    .agg(
        round(sum("monto_total"), 2).alias("ventas_totales"),
        count("id_venta").alias("cantidad_transacciones")
    )

# Visualización de resultados
df_resumen.orderBy("fecha_venta").show(10)

📋 Resumen de Consultas Clave
Conteo en Cassandra: SELECT COUNT(*) FROM ventas_crudas;
Vista Previa Cruda: SELECT * FROM ventas_crudas LIMIT 5;
Transformación Spark: Agrupación por fecha_venta y categoria con sumatorias y conteos.

Proyecto desarrollado para la cátedra de Big Data - UNEG.
