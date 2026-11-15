# Workout Tracker — Grupo 07

Aplicación full-stack para gestionar ejercicios y entrenamientos. El sistema se ejecuta con Docker y está compuesto por React/Nginx, FastAPI y PostgreSQL. Las pruebas funcionales de sistema utilizan Playwright y Page Object Model (POM).

## Requisitos previos

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) o Docker Engine con Docker Compose.
- [Node.js](https://nodejs.org/) y npm.
- Git (opcional).

En Windows, abre Docker Desktop antes de comenzar. Ejecuta los comandos de Docker desde la **raíz del repositorio**, donde se encuentra `docker-compose.yml`.

## Iniciar la aplicación

```bash
docker compose up -d --build
docker compose ps
```

Espera hasta que `db`, `backend` y `frontend` estén iniciados y saludables (`healthy`).

| Servicio | Dirección |
|---|---|
| Aplicación web | [http://localhost:8080](http://localhost:8080) |
| Documentación de la API | [http://localhost:8000/docs](http://localhost:8000/docs) |

### Cuenta de administrador

```text
Correo: admin@example.com
Contraseña: admin123
```

## Ejecutar las pruebas de sistema

Las pruebas recorren la interfaz como una persona usuaria. Se ejecutan en **Chromium, Firefox y WebKit**, emplean selectores orientados al usuario y esperas automáticas de Playwright.

### 1. Preparar Playwright

Mantén los contenedores activos y entra al frontend:

```bash
cd workouts_udec_frontend
npm install
npx playwright install
```

En Linux, si faltan dependencias del sistema:

```bash
sudo npx playwright install-deps
```

> En **PowerShell**, si `npx` está restringido, utiliza `npx.cmd`.

### 2. Ejecutar la suite completa

Git Bash, Linux o macOS:

```bash
npx playwright test
```

PowerShell:

```powershell
npx.cmd playwright test
```

Actualmente existen **12 casos funcionales**. Al ejecutarse en tres navegadores producen **36 ejecuciones**. La cantidad puede aumentar si se agregan nuevos casos.

### 3. Comandos útiles

| Objetivo | Git Bash, Linux o macOS | PowerShell |
|---|---|---|
| Ver los navegadores | `npx playwright test --headed` | `npx.cmd playwright test --headed` |
| Abrir el modo interactivo | `npx playwright test --ui` | `npx.cmd playwright test --ui` |
| Ejecutar un archivo | `npx playwright test tests/login.spec.ts` | `npx.cmd playwright test tests/login.spec.ts` |
| Comprobar estabilidad | `npx playwright test --repeat-each=3` | `npx.cmd playwright test --repeat-each=3` |
| Abrir el reporte HTML | `npx playwright show-report` | `npx.cmd playwright show-report` |

Playwright se ejecuta en modo **headless** de manera predeterminada; por eso no abre una ventana cuando las pruebas pasan. Usa `--headed` para observar las acciones o `--ui` para depurar paso a paso.

## Cobertura actual

| Área | Escenarios |
|---|---|
| Autenticación y sesión | Inicio correcto, contraseña incorrecta, cierre de sesión y redirección sin sesión |
| Registro | Registro correcto, contraseñas diferentes, correo inválido y correo existente |
| Control de acceso | Un usuario normal no dispone del acceso administrativo |
| Ejercicios | Creación correcta y rechazo de un nombre vacío |
| Entrenamientos | Creación y finalización de un entrenamiento |

La explicación del diseño y de cada caso está en [`documentaci_n_playwright.md`](documentaci_n_playwright.md).

## Organización y calidad de las pruebas

```text
workouts_udec_frontend/
├── playwright.config.ts
└── tests/
    ├── pages/
    │   ├── AdminExercisePage.ts
    │   ├── DashboardPage.ts
    │   ├── LoginPage.ts
    │   ├── RegisterPage.ts
    │   └── WorkoutPage.ts
    ├── access-control.spec.ts
    ├── exercises.spec.ts
    ├── login.spec.ts
    ├── register.spec.ts
    └── workout.spec.ts
```

La suite aplica **Page Object Model**: los archivos de `tests/pages/` encapsulan la navegación, los selectores y las acciones reutilizables; los archivos `*.spec.ts` contienen los escenarios y sus aserciones. Esta separación evita duplicación y facilita el mantenimiento.

Los casos incluyen datos representativos:

- **Entradas válidas:** inicio de sesión, registro y creación de ejercicios correctos.
- **Entradas inválidas:** contraseña incorrecta, contraseñas diferentes y correo duplicado.
- **Valores límite:** nombre vacío y formato de correo inválido.

Para reducir resultados inestables:

- Se priorizan `getByRole`, `getByLabel` y `getByText`, centrados en elementos reconocibles por una persona usuaria.
- Se utilizan las esperas automáticas de Playwright y aserciones como `toBeVisible()`; no se agregan pausas fijas.
- Los usuarios y ejercicios creados durante los tests reciben valores únicos.
- El flujo de entrenamiento elimina previamente cualquier entrenamiento activo que pudiera quedar de una ejecución anterior.

La estabilidad puede comprobarse repitiendo cada caso tres veces:

```bash
npx playwright test --repeat-each=3
```

Antes de incorporar los últimos casos de control de acceso, esta comprobación completó **108 ejecuciones consecutivas sin fallos**. Para obtener evidencia de la versión entregada, se recomienda volver a ejecutar el comando anterior antes del envío final.

## Detener o reiniciar el entorno

Detener conservando la base de datos:

```bash
docker compose down
```

Reiniciar desde una base de datos vacía:

```bash
docker compose down -v
docker compose up -d --build
```

> `docker compose down -v` elimina todos los datos persistentes. Úsalo solo cuando necesites restablecer el entorno.

## Solución de problemas

### Playwright no encuentra la aplicación

Desde la raíz, comprueba `docker compose ps` y verifica que [http://localhost:8080](http://localhost:8080) responda.

### PowerShell bloquea `npx`

```powershell
npx.cmd playwright test
```

### Faltan los navegadores

Desde `workouts_udec_frontend`:

```bash
npx playwright install
```

### La base de datos quedó en un estado inconsistente

Desde la raíz (este procedimiento elimina los datos):

```bash
docker compose down -v
docker compose up -d --build
```

## Pruebas de la API

Con los contenedores activos, la colección de Postman se ejecuta mediante Newman:

```bash
newman run postman/collections/WorkoutTracker.postman_collection.json -e postman/environments/WorkoutTracker.postman_Environment.json
```

## Decisiones de contenerización

- Construcción multi-etapa del frontend.
- Ejecución sin privilegios en frontend y backend.
- `.dockerignore` para excluir dependencias y cachés.
- `healthcheck` y dependencias condicionadas para respetar el orden de inicio.
- Red bridge dedicada (`workouts_network`).
- Volumen `postgres_data` para conservar la información.
- Migraciones y creación de la cuenta administrativa automatizadas.
