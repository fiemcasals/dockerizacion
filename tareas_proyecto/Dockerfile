
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
CMD ["gunicorn", "tareas_proyecto.wsgi:application", "--bind", "0.0.0.0:8000"]
