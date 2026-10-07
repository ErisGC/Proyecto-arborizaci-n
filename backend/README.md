# Backend - Inventario Arbóreo Escolar

API en FastAPI. La estructura de carpetas sigue el documento "Estructura de Carpetas del Backend".

## Configuración (variables de entorno)

Toda la configuración sale de variables de entorno, que se leen de `backend/.env`. Ese archivo **no se sube a Git**. La plantilla, sin secretos, es `backend/.env.example`:

```powershell
cd backend
copy .env.example .env
```

Luego edita `.env` y completa al menos `POSTGRES_PASSWORD`. En macOS o Linux el comando es `cp .env.example .env`.

| Variable | Para qué sirve | Valor por defecto |
|---|---|---|
| `APP_ENV` | `desarrollo`, `pruebas` o `produccion` | `desarrollo` |
| `API_PREFIX` | Prefijo de todas las rutas | `/api` |
| `POSTGRES_DB`, `POSTGRES_USER` | Base y usuario de PostgreSQL | `arborizacion` |
| `POSTGRES_PASSWORD` | Contraseña de PostgreSQL | *(obligatoria)* |
| `POSTGRES_HOST`, `POSTGRES_PORT` | Dónde está PostgreSQL | `localhost`, `5432` |
| `DATABASE_URL` | Opcional. Reemplaza las cinco anteriores, por ejemplo `sqlite:///./data/local.db` | *(vacía)* |
| `JWT_SECRET` | Clave para firmar las sesiones | *(obligatoria en producción)* |
| `JWT_ALGORITHM`, `JWT_EXPIRE_MINUTES` | Algoritmo y duración de la sesión | `HS256`, `480` |
| `CORS_ORIGINS` | Direcciones del frontend que pueden llamar a la API, separadas por coma | `http://localhost:5173` |
| `STORAGE_PATH` | Carpeta de las fotografías | `data/fotos` |

Reglas que se revisan al arrancar:

- En `produccion`, `JWT_SECRET` debe tener al menos 32 caracteres y `CORS_ORIGINS` no puede ser `*`. Si no se cumple, el backend no arranca y dice qué falta.
- En `desarrollo`, si `JWT_SECRET` está vacío se genera uno temporal y se muestra un aviso.
- Para generar un secreto: `python -c "import secrets; print(secrets.token_urlsafe(48))"`

En el código, la configuración se lee con `get_settings()` de `app/core/config.py`. Las contraseñas y el secreto son de tipo `SecretStr`, así que no aparecen si se imprimen por error en un log.

Si agregas una variable nueva a `Settings`, agrégala también a `.env.example`. Hay una prueba que falla si la plantilla queda incompleta o si trae una contraseña escrita.

## Correr con Docker Compose

Levanta el backend y PostgreSQL con la misma configuración en cualquier computador. Requiere [Docker Desktop](https://www.docker.com/products/docker-desktop/).

```powershell
cd backend
copy .env.example .env          # y completa POSTGRES_PASSWORD
docker compose up --build -d
```

- API: http://localhost:8000/api/docs
- Estado: http://localhost:8000/api/salud
- PostgreSQL: `localhost:5432`, solo desde este computador (para pgAdmin o DBeaver)

| Comando | Qué hace |
|---|---|
| `docker compose ps` | Muestra el estado. El backend aparece como `healthy` cuando ya se conectó a la base. |
| `docker compose logs -f backend` | Muestra los mensajes del backend. |
| `docker compose up --build -d` | Reconstruye la imagen después de cambiar el código o `requirements.txt`. |
| `docker compose down` | Detiene todo. Los datos se conservan. |
| `docker compose down -v` | Detiene todo y **borra** la base de datos y las fotos. |

Cómo está armado:

- **`db`**: PostgreSQL 17. Los datos viven en el volumen `datos_postgres`. El backend no arranca hasta que la base responde.
- **`backend`**: la imagen del `Dockerfile` (Python 3.12, usuario sin privilegios). Lee el `.env`, pero dentro de Docker usa `db` como servidor de base de datos. Las fotos van al volumen `datos_fotos`. Se marca como sano cuando `/api/salud` responde.
- Si los puertos 8000 o 5432 ya están ocupados, cámbialos con `BACKEND_PORT` y `DB_PORT` en el `.env`.

Para programar con recarga automática, una opción cómoda es levantar solo la base con Docker y correr el backend en tu computador:

```powershell
docker compose up -d db
uvicorn app.main:app --reload     # con POSTGRES_HOST=localhost en el .env
```

## Correr sin Docker

```powershell
python -m venv venv
venv\Scripts\activate
pip install -r requirements-dev.txt
uvicorn app.main:app --reload
```

- Documentación: http://127.0.0.1:8000/api/docs
- Estado del servidor y de la base de datos: http://127.0.0.1:8000/api/salud

Para probar sin PostgreSQL, pon `DATABASE_URL=sqlite:///./data/local.db` en el `.env`.

## Pruebas

```powershell
pytest
ruff check app tests
```
