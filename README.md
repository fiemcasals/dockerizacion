# dockerizacion



````md
# 🚀 Guía para Dockerizar Proyecto Django: `tareas_proyecto`

Esta guía explica paso a paso cómo dockerizar el proyecto Django usando:

- `Dockerfile` → para crear la imagen de la aplicación.
- `docker-compose.yml` → para orquestar múltiples servicios como Django + PostgreSQL.

El objetivo es que cualquier persona pueda entender qué hace cada línea.

---

## 📌 1. ¿Qué es Docker y por qué lo usamos?

Docker permite empacar una aplicación junto con sus dependencias (Python, librerías, paquetes del sistema, etc.) en una **imagen** que puede ejecutarse en cualquier computadora o servidor sin necesidad de configuraciones manuales.

En lugar de instalar Python, pip, PostgreSQL, etc., en tu PC:

👉 Docker crea un contenedor aislado donde **todo ya está configurado.**

---

## 📌 2. Dockerfile (Construcción de la Imagen de Django)

Este archivo explica cómo construir la imagen del proyecto Django.

Crear archivo: `Dockerfile` en la raíz del proyecto (junto a `manage.py`):

```Dockerfile
# ============================================================
# DOCKERFILE PARA PROYECTO DJANGO "tareas_proyecto"
# ------------------------------------------------------------
# Este archivo define CÓMO construir la IMAGEN de Docker
# que contendrá:
#   - Un Linux mínimo
#   - Python
#   - Todas las librerías de requirements.txt
#   - Tu código Django
#   - El comando para ejecutar el servidor
# ============================================================

# 1) Imagen base:
#    Partimos de una imagen oficial de Python 3.12 basada en Debian "slim" (más liviana).
FROM python:3.12-slim

# 2) Variables de entorno útiles para Python:
#    PYTHONDONTWRITEBYTECODE = evita que Python genere archivos .pyc
#    PYTHONUNBUFFERED       = hace que la salida se muestre en tiempo real (logs directos)
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

# 3) Definimos el directorio de trabajo dentro del contenedor:
#    Todo lo que hagamos a partir de aquí asume que estamos parados en /app
WORKDIR /app

# 4) Instalamos paquetes del sistema necesarios para algunas librerías de Python
RUN apt-get update && apt-get install -y \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# 5) Copiamos SOLO el requirements.txt primero para usar cache de Docker
COPY requirements.txt /app/

# 6) Instalamos las dependencias del proyecto
RUN pip install --no-cache-dir -r requirements.txt

# 7) Ahora copiamos TODO el proyecto al contenedor
COPY . /app/

# 8) Exponemos puerto donde corre Django dentro del contenedor
EXPOSE 8000

# 9) Comando que ejecutará el contenedor al iniciar
CMD ["sh", "-c", "python manage.py migrate && python manage.py runserver 0.0.0.0:8000"]
````

---

## 📌 3. Archivo `docker-compose.yml`

Este archivo permite levantar **varios servicios al mismo tiempo**, por ejemplo:

| Servicio | Rol                      |
| -------- | ------------------------ |
| `db`     | Base de datos PostgreSQL |
| `web`    | Aplicación Django        |

Crear archivo en la raíz:

```yaml
version: "3.9"

services:
  db:
    image: postgres:16
    container_name: tareas_db
    restart: always
    environment:
      POSTGRES_DB: django_db
      POSTGRES_USER: django_user
      POSTGRES_PASSWORD: django_pass
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  web:
    build: .
    container_name: tareas_web
    restart: always
    depends_on:
      - db
    environment:
      DB_NAME: django_db
      DB_USER: django_user
      DB_PASSWORD: django_pass
      DB_HOST: db
      DB_PORT: "5432"
    ports:
      - "8000:8000"
    volumes:
      - .:/app

volumes:
  postgres_data:
```

---

## 📌 4. Configuración en `settings.py` para Postgres

Modificar la sección `DATABASES`:

```python
import os

DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": os.getenv("DB_NAME", "django_db"),
        "USER": os.getenv("DB_USER", "django_user"),
        "PASSWORD": os.getenv("DB_PASSWORD", "django_pass"),
        "HOST": os.getenv("DB_HOST", "localhost"),
        "PORT": os.getenv("DB_PORT", "5432"),
    }
}
```

---

# Importante: agregar a requierements.txt -> psycopg2-binary #esta libreria es para conectar Django con PostgreSQL y anteriormente no la tenian pq tenian otra base de datos
#Importante: agregar a la librerias a descargar en requierements.txt -> gunicorn>=21.2,<22

## 📌 5. Ignorar SQLite (Opcional pero recomendado)

Agregar a `.gitignore`:

```
db.sqlite3
```

---

## 📌 6. Comandos importantes 🧠

Construir imagen:

```bash
docker compose build
```

Levantar la app:

```bash
docker compose up
```

Ejecutarlo en segundo plano:

```bash
docker compose up -d
```

Ver logs:

```bash
docker compose logs -f web
```

Detener servicios:

```bash
docker compose down
```

---


### 📎 Próximo paso sugerido:

> Hacer versión de producción con `gunicorn + Nginx + volumes` para archivos estáticos.

---

#cuando modifican el docker-compose o el Dockerfile o el requirements.txt antes de hacer docker compose build, deben hacer docker compose down

📌 Regla general:
Qué cambiaste           	¿Necesita rebuild?      	¿Necesita bajar todo?
Código Python (.py), templates	❌ No hace falta build  	❌ No hace falta down
requirements.txt        	✅ Sí                   	❌ No obligatorio, pero recomendado
Dockerfile              	✅ Sí, siempre	                ❌ No obligatorio, pero recomendado
docker-compose.yml  		⚠ Depende (si cambian puertos/volúmenes/env)	muchas veces sí


#posibles errores:

el docker up lo levanta en el puerto 0.0.0.0 -> necesitamos tener en settings.py "ALLOWED_HOSTS = ['127.0.0.1', 'localhost', '0.0.0.0']"

#tiene que cambiar el nombre de la aplicacion en el Dockerfile por el suyo: cambiar tareas_proyecto por el de ustedes
 Comando que ejecutará el contenedor al iniciar
CMD ["gunicorn", "tareas_proyecto.wsgi:application", "--bind", "0.0.0.0:8000"]
