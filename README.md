# Proyecto 9 — PostgreSQL, pgvector y búsqueda semántica

## 1. Objetivo

Implementar una búsqueda semántica con PostgreSQL y pgvector, utilizando embeddings generados localmente con Sentence Transformers. El proyecto incorpora aislamiento de datos entre tenants y una medición del rendimiento de las consultas.

## 2. Tecnologías

- Python
- PostgreSQL 17 en Docker
- pgvector 0.8.7
- sentence-transformers
- Modelo: `paraphrase-multilingual-MiniLM-L12-v2`
- psycopg 3
- Índice HNSW para distancias coseno
- Seguridad por filas de PostgreSQL (RLS)

## 3. Ejecución

1. Iniciar Docker Desktop.
2. Comprobar que el contenedor `proyecto9-postgres` está activo.
3. Activar el entorno virtual de Python.
4. Configurar las variables de entorno `PGPASSWORD` y `APP_ROLE_PASSWORD` en PowerShell.
5. Ejecutar:

```powershell
python .\proyecto9_docker.py
```


## 4. Datos y búsqueda

Se almacenan cinco frases de seguros de automóvil para `aseguradora_A` y una frase adicional de prueba para `aseguradora_B`.

La consulta de ejemplo es:

> Mi vehículo se ha averiado y necesito ayuda

La consulta devuelve las dos frases más similares mediante la distancia coseno entre embeddings de 384 dimensiones.

## 5. Aislamiento entre tenants

La tabla utiliza Row-Level Security (RLS) con una política basada en la variable de sesión `app.tenant_id`.

Resultados de las pruebas:
- `aseguradora_A`: acceso a sus cinco frases.
- `aseguradora_B`: acceso únicamente a su frase de prueba.
- Sin tenant configurado: cero filas visibles.

Las consultas de recuperación también incluyen un filtro explícito por `tenant_id`.

## 6. Rendimiento

En una ejecución de prueba, PostgreSQL registró un tiempo de ejecución SQL de 0,139 ms mediante `EXPLAIN ANALYZE`.

Este valor corresponde a una tabla pequeña y a una ejecución concreta; no constituye una garantía de rendimiento con grandes volúmenes de datos. El índice HNSW está preparado, aunque PostgreSQL puede elegir otro plan para tablas pequeñas.

## 7. Archivos principales

- `proyecto9_docker.py`: creación de la estructura, generación y almacenamiento de embeddings, búsqueda semántica, comprobaciones de aislamiento y medición del rendimiento.
- `salida_proyecto9_docker.txt`: salida registrada de una ejecución del script.
- `README.md`: instrucciones y documentación del proyecto.

## 8. Consideraciones

El usuario de aplicación `app_pgvector` no es superusuario y tiene `NOBYPASSRLS`. La política RLS complementa el filtro de tenant en las consultas.

La configuración de `app.tenant_id` debe controlarse desde una capa de aplicación autenticada y autorizada: no debe permitirse que un usuario final elija libremente el tenant. El aislamiento debe probarse con el usuario de aplicación, no con el usuario administrador de PostgreSQL.