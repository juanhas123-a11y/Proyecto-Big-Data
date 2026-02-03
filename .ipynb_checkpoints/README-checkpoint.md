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

Proyecto desarrollado para la cátedra de Big Data - UNEG.


📝 Actualización: Resolución de la Fase 2 (Ingesta Masiva)
En esta etapa, se implementó la generación y carga de 100,000 registros en Apache Cassandra. Durante el desarrollo, se documentaron y resolvieron los siguientes puntos críticos:

🛠️ Solución de Errores Técnicos
ModuleNotFoundError (cassandra-driver): Se identificó que el entorno de Jupyter no contaba con el driver nativo. Se resolvió mediante la instalación dinámica dentro del notebook:

Python
!pip install cassandra-driver
UnresolvableContactPoints: Error de red donde Python no localizaba el contenedor. Se solucionó asegurando que el host en la configuración del clúster coincidiera con el nombre del servicio en docker-compose.yml (host: 'cassandra').

📥 Proceso de Ingesta

Generación de Datos: Se utilizó la librería uuid, random y datetime para crear un dataset de 100,000 filas con categorías como Electrónica, Ropa y Hogar.


Optimización: Se utilizó session.prepare() para pre-compilar la consulta de inserción, mejorando significativamente la velocidad de carga.

Validación Técnica: Se confirmó la carga exitosa mediante el comando:

SQL
SELECT COUNT(*) FROM proyecto_bigdata.ventas_crudas;

⚙️ Actualización: Resolución de la Fase 3 (Procesamiento con Spark)

En esta fase se implementó la capa de transformación (ELT) para convertir 100,000 registros de datos crudos en métricas de negocio útiles.

🛠️ Solución de Errores Técnicos

Py4JJavaError: Se identificó que Spark requería el conector oficial de Java para comunicarse con Cassandra. Se solucionó forzando la descarga del paquete spark-cassandra-connector al iniciar la SparkSession.

Optimización de Lectura: Se configuró el script para utilizar la Partition Key (fecha_venta) de Cassandra, permitiendo que Spark lea los datos de forma paralela y distribuida, evitando cuellos de botella.

🔄 Transformaciones Realizadas (Lógica de Negocio)

Lectura Distribuida: Conexión exitosa al Keyspace proyecto_bigdata para cargar el DataFrame inicial.

Limpieza de Datos: Se aplicó un filtro para eliminar registros inconsistentes (montos nulos o menores/iguales a cero), garantizando la integridad del análisis.

Agregación Paralela: Se procesaron los 100,000 registros para obtener:

Ventas totales por categoría y fecha.
Conteo masivo de transacciones por periodo.
📊 Validación
Se confirmó la transformación mediante la función .show(), visualizando las métricas consolidadas antes de su envío al Data Warehouse.