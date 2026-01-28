# 🚀 Pipeline de Big Data: Del Origen al Análisis (UNEG)

Este proyecto implementa un ecosistema completo de Big Data. Procesa **100,000 registros** desde un almacenamiento NoSQL (**Cassandra**) hasta un Data Warehouse analítico (**ClickHouse**) usando **Apache Spark**.

---

## 🛠️ Paso 1: Instalación de Herramientas (Si no tienes nada)

Si tu computadora está "limpia", debes instalar lo siguiente en este orden:

1. **Docker Desktop:** [Descargar aquí](https://www.docker.com/products/docker-desktop/). Es el motor que correrá las bases de datos.
2. **Python 3.10+:** [Descargar aquí](https://www.python.org/downloads/). Asegúrate de marcar la casilla **"Add Python to PATH"** durante la instalación.
3. **Java JDK 11:** [Descargar aquí](https://www.oracle.com/java/technologies/downloads/). Necesario para que Spark funcione.

---

## 🚀 Paso 2: Configuración del Proyecto

1. **Clonar el repositorio:** Descarga este proyecto como ZIP y extráelo, o usa `git clone`.
2. **Levantar las Bases de Datos:** Abre una terminal (PowerShell o CMD) dentro de la carpeta del proyecto y ejecuta:
   ```bash
   docker-compose up -d
   ```
   Espera 1 minuto a que los motores arranquen por completo.
3. **Instalar conectores de Python:** En la misma terminal, ejecuta:
   ```bash
   pip install -r requirements.txt
   ```

---

## 📑 Paso 3: Ejecución del Pipeline (Jupyter)

Abre el archivo `.ipynb` con Jupyter Notebook o VS Code. Ejecuta todas las celdas en orden.

El script automáticamente:
- Creará los datos en Cassandra.
- Los procesará con Spark.
- Los cargará refinados en ClickHouse.

---

## 📊 Paso 4: ¿Cómo ver las tablas resultantes?

Para verificar que todo funcionó, usaremos la terminal para entrar a los contenedores y consultar las tablas:

### A. Ver Datos Crudos (Cassandra)

Aquí están los 100,000 registros originales. Ejecuta en tu terminal:
```bash
docker exec -it cassandra_db cqlsh -e "SELECT * FROM proyecto_bigdata.ventas LIMIT 10;"
```

### B. Ver Resumen Analítico (ClickHouse)

Aquí verás el resultado del procesamiento de Spark (Ventas totales por categoría). Ejecuta:
```bash
docker exec -it clickhouse_dw clickhouse-client -q "SELECT * FROM ventas_resumen ORDER BY total_ventas DESC FORMAT PrettyCompact;"
```

---

## 🏗️ Resumen de la Arquitectura

- **Capa 1 (Ingesta):** Cassandra recibe los datos crudos (Escritura rápida).
- **Capa 2 (Procesamiento):** Spark limpia duplicados y agrupa categorías.
- **Capa 3 (Servicio):** ClickHouse almacena el resumen para reportes (Lectura rápida).

---

## ⚠️ Solución de Errores Comunes

- **"Connection Refused":** Cassandra aún está cargando. Espera 30 segundos y reintenta.
- **"Java not found":** Verifica que instalaste el JDK y reiniciaste tu terminal.
- **Docker lento:** Asegúrate de tener al menos 4GB de RAM asignados a Docker en Settings -> Resources.
