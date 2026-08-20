# =============================================================================
# Django Application Dockerfile
# =============================================================================

# Stage 1: Build stage with development dependencies
FROM python:3.9-slim AS builder

ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

COPY requirements.txt /tmp/
RUN pip install --no-cache-dir -r /tmp/requirements.txt


# Stage 2: Production stage
FROM python:3.9-slim AS production

# FIXED: Points to the actual settings file in this repository
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    DJANGO_SETTINGS_MODULE=hello_world_django_app.settings \
    PATH="/opt/venv/bin:$PATH"

RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq5 \
    curl \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean

RUN groupadd -r django && useradd -r -g django django

COPY --from=builder /opt/venv /opt/venv

WORKDIR /usr/src/app

COPY --chown=django:django . .

RUN mkdir -p /usr/src/app/static /usr/src/app/media /usr/src/app/logs \
    && chown -R django:django /usr/src/app

USER django

RUN python manage.py collectstatic --noinput --clear

HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD curl -f http://localhost:8000/ || exit 1

EXPOSE 8000

# FIXED: Updated WSGI application path to match the repository module name
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "3", "--timeout", "120", "--access-logfile", "-", "--error-logfile", "-", "hello_world_django_app.wsgi:application"]
