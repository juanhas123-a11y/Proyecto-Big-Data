# 🚀 Pipeline de Big Data: Cassandra → Spark → ClickHouse

Este repositorio contiene la implementación de un ecosistema de datos End-to-End diseñado para simular un entorno de producción masivo. El proyecto demuestra la orquestación de una base de datos NoSQL (Cassandra), procesamiento distribuido (Apache Spark) y un Data Warehouse analítico (ClickHouse), todo integrado mediante Docker.

---

## 🏗️ Arquitectura del Sistema

La arquitectura se basa en el modelo de tres capas para el manejo eficiente de Big Data:

1. **Capa de Ingesta (OLTP):** Apache Cassandra se encarga de recibir datos crudos (100,000 registros) con alta disponibilidad.
2. **Capa de Procesamiento (ELT):** Apache Spark (PySpark) realiza la lectura paralela, limpieza de datos y agregaciones complejas.
3. **Capa Analítica (OLAP):** ClickHouse almacena el resumen procesado, optimizado para consultas de Business Intelligence en milisegundos.

---

## 🛠️ Requisitos e Instalación

### 1. Requisitos de Software

- **Docker Desktop** (con 4GB de RAM asignados como mínimo).
- **Python 3.10+** (para ejecución de scripts locales si es necesario).

### 2. Despliegue de Infraestructura

Desde la raíz del proyecto, levanta los contenedores:
```bash
docker-compose up -d
```
**Nota:** Cassandra suele tardar unos 45 segundos en inicializar completamente sus protocolos de red.

### 3. Configuración del Entorno Python

Instala las dependencias necesarias dentro de tu entorno de Jupyter o virtualenv:
```bash
pip install cassandra-driver clickhouse-connect pandas numpy
```

---

## 🚀 Guía de Ejecución

### Fase 1 y 2: Ingesta en Cassandra

Se generan 100,000 registros sintéticos que simulan ventas minoristas.

**Comando de validación:** Para verificar que los datos se cargaron correctamente en el clúster NoSQL:
```bash
docker exec -it cassandra_db cqlsh -e "SELECT COUNT(*) FROM proyecto_bigdata.ventas_crudas;"
```

### Fase 3: Procesamiento con PySpark

Se utiliza Spark para transformar el "Data Lake" (Cassandra) en información útil:

- **Limpieza:** Filtrado de registros inconsistentes (montos ≤ 0).
- **Agregación:** Reducción de 100k registros a un resumen diario por categoría.

### Fase 4: Carga al Data Warehouse (ClickHouse)

Los datos procesados se migran al esquema `dw_analitico`.

**Consultas Analíticas Finales:** Ejecuta estos comandos para obtener métricas de negocio:

#### A. Top 10 categorías (Volumen de ventas):
```bash
docker exec -it clickhouse_dw clickhouse-client -q "SELECT categoria, sum(ventas_totales) as total FROM dw_analitico.ventas_resumen GROUP BY categoria ORDER BY total DESC LIMIT 10;"
```

#### B. Promedio de ventas diarias por categoría:
```bash
docker exec -it clickhouse_dw clickhouse-client -q "SELECT categoria, avg(ventas_totales) as promedio FROM dw_analitico.ventas_resumen GROUP BY categoria ORDER BY promedio DESC;"
```

---

## 📊 Análisis de Rendimiento

Un punto clave de este proyecto es la comparativa de eficiencia:

- **Escritura (Cassandra):** Optimizada para la ingesta masiva de transacciones individuales.
- **Consulta (ClickHouse):** Gracias a su motor MergeTree y almacenamiento columnar, resuelve agregaciones (SUM/AVG) sobre miles de registros en una fracción del tiempo que tomaría en una base de datos tradicional.

---

## ⚠️ Solución de Problemas Comunes

- **Error de Conector Spark:** Si Spark no reconoce Cassandra, asegúrate de que la sesión incluya el paquete: `com.datastax.spark:spark-cassandra-connector_2.12:3.5.0`.
  
- **Acceso Denegado en ClickHouse:** Si el usuario no tiene permisos para crear tablas o insertar, ejecuta este parche de seguridad en la terminal:
  ```bash
  docker exec -it clickhouse_dw bash -c "echo '<clickhouse><users><default><access_management>1</access_management></default></users></clickhouse>' > /etc/clickhouse-server/users.d/access.xml"
  docker restart clickhouse_dw
  ```

- **Número de Categorías:** Si el reporte analítico solo muestra 5 categorías, es el comportamiento esperado. El generador de datos utiliza un catálogo maestro de 5 categorías (Hogar, Electrónica, Ropa, Alimentos, Deportes).

- **Compatibilidad Python 3.13+:** El proyecto incluye un Mock del módulo asyncore para mantener la compatibilidad con el driver de Cassandra.

---

Proyecto desarrollado para la cátedra de Big Data - Universidad Nacional Experimental de Guayana (UNEG).
