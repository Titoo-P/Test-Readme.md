# Proyecto de Contenerización - Workout Tracker (Grupo 07)

Este repositorio contiene la arquitectura de contenerización mediante Docker para la aplicación full-stack "Workout Tracker". El sistema se divide en tres servicios principales: Frontend (React/Nginx), Backend (FastAPI) y Base de Datos (PostgreSQL).

## 1. Rúbrica y Decisiones 

Durante el desarrollo de esta contenerización, implementamos las siguientes configuraciones para cumplir con los criterios de evaluación:

* **Calidad de Dockerfile (Seguridad y Optimización):**
  * **Multi-etapa (Multi-stage build):** En el Frontend usamos una primera etapa con Node.js solo para compilar el proyecto y una segunda etapa con Nginx para los archivos finales, logrando una imagen de Docker mucho más liviana.
  * **Usuarios Non-Root:** El Backend crea y utiliza un usuario `appuser` sin privilegios. El Frontend utiliza la imagen oficial `nginxinc/nginx-unprivileged`, exponiendo el puerto 8080 en lugar del 80 para evitar el requerimiento de permisos de administrador.
  * **Archivos `.dockerignore`:** Implementados en ambos servicios para excluir entornos virtuales, dependencias locales (`node_modules`) y cachés.

* **Configuración de Servicios, Dependencias y Healthchecks:**
  * **Orquestación Segura:** Se utiliza `depends_on` con `condition: service_healthy` en `docker-compose.yml`. Esto garantiza que el Backend no inicie hasta que PostgreSQL esté aceptando conexiones, y que el Frontend espere a que la API esté respondiendo.
  * **Healthchecks Activos:** 
    * `db`: Utiliza `pg_isready`.
    * `backend`: Utiliza `curl` hacia la ruta `/docs`.
    * `frontend`: Utiliza `curl` hacia el `localhost:8080`.
  * **Aislamiento de Red:** Todos los contenedores se comunican internamente a través de una red dedicada tipo bridge (`workouts_network`).

* **Volúmenes y Persistencia de Datos:**
  * Se definió un volumen gestionado por Docker (`postgres_data`) y mapeado al directorio `/var/lib/postgresql/data` del contenedor de la base de datos, asegurando que los usuarios, rutinas y registros sobrevivan a los reinicios de los contenedores.

* **Automatización de Instalación:**
  * A través del script `entrypoint.sh` en el Backend, el contenedor automatiza la inicialización del sistema: genera el archivo de migración de Alembic, aplica las migraciones a la base de datos vacía y ejecuta el script de Python que crea la cuenta de administrador, si no esta.

---

## 2. Requisitos Previos

Para compilar y ejecutar este proyecto, necesitas tener instalados los siguientes componentes en tu sistema (Windows, Linux, WSL mi caso):

* [Docker Engine](https://docs.docker.com/get-docker/) (v20.10 o superior)
* [Docker Compose](https://docs.docker.com/compose/install/) (Integrado en versiones modernas de Docker Desktop / Plugin en Linux)

---

## 3. Instrucciones de Ejecución y Comandos

### Levantar la Aplicación
En Windows por lo menos y WSL, se debe tener Docker Desktop abierto.
Abrir laterminal en la raíz del proyecto y ejecutar:

```bash
docker compose up -d --build
```
*En caso de WSL, tuve que anteponer sudo en el primer comando, y la clave del WSL.*

Este comando construirá las imágenes desde cero y levantará los contenedores en segundo plano (`-d`). 

### Acceso a los Servicios
Una vez que los contenedores indiquen el estado "Healthy", se puede acceder a travez de algun navegador Web

* **Aplicación Web (Frontend):** [http://localhost:8080](http://localhost:8080)
* **Documentación API REST (Backend):** [http://localhost:8000/docs](http://localhost:8000/docs)

### Credenciales de Administrador por Defecto
El sistema generará la siguiente cuenta automáticamente durante el despliegue:
* **Usuario/Email:** `admin@example.com`
* **Contraseña:** `admin123`

### Detener los Servicios
Para apagar la aplicación conservando intactos los datos de la base de datos:

```bash
docker compose down
```

### Reinicio Limpio (Destrucción de Datos)
Para borrar base de datos y empezar de cero `-v`:

```bash
docker compose down -v
```
###   Ejecutar pruebas automatizadas de la API
```bash
docker compose up -d --build
```

```bash
newman run postman/collections/WorkoutTracker.postman_collection.json -e postman/environments/WorkoutTracker.postman_Environment.json
```
