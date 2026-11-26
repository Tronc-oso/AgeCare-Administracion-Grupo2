# AgeCare Administrativo | Grupo 2

Repositorio de evidencias del proyecto APT AgeCare Administrativo. El equipo desarrolló el backend de la consola administrativa de AgeCare para consultar indicadores y gestionar operaciones como tickets, contenido, catálogos y auditoría.

## Código y demostración

- [Repositorio del backend](https://github.com/Pombet/agecare-admin-api)
- [Consola de demostración en Vercel](https://agecare-admin-pruebas.vercel.app/demo/)
- [Swagger de la API de pruebas](https://agecare-admin-pruebas.vercel.app/api/v1/admin/docs)
- [Manual técnico](MANUAL_TECNICO_AGECARE.md)

## Tecnologías y arquitectura

La consola demo consume la API REST de FastAPI (Python 3.12). La API aplica permisos por rol y usa SQLAlchemy para acceder a PostgreSQL 16. Alembic gestiona las migraciones. Docker Compose levanta el entorno local y el entorno remoto de pruebas usa Vercel y Neon.

## Ejecución local

Desde la raíz del repositorio del backend, con Docker y Compose v2 disponibles:

```sh
docker compose up --build
```

Abrir la consola en `http://localhost:8000/demo/` o Swagger en `http://localhost:8000/api/v1/admin/docs`.

[Dockerfile](https://github.com/Pombet/agecare-admin-api/blob/main/Dockerfile) · [Docker Compose](https://github.com/Pombet/agecare-admin-api/blob/main/docker-compose.yml) · [Ejemplo de variables](https://github.com/Pombet/agecare-admin-api/blob/main/.env.example)

## Metodología y equipo

El Grupo 2 empleó la metodología en cascada para organizar los entregables de definición, requisitos, diseño, implementación, pruebas y entrega.

Sebastián Pombet, Frank Martínez y Joaquín Becerra colaboraron en el proyecto. El equipo compartió tareas de desarrollo, integración, despliegue y documentación.
