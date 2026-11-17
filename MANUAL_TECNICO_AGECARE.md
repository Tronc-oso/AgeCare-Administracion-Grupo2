# Manual técnico de AgeCare Administrativo

Este manual corresponde al backend administrativo del Grupo 2. La referencia de código revisada es el [commit fdc9cf5](https://github.com/Pombet/agecare-admin-api/tree/fdc9cf5d9ee0d5a1e8ec986db96d33e7548060e6). El commit que corre en Vercel se confirma en el panel del despliegue antes de registrar la versión entregada.

## Arquitectura y alcance

FastAPI recibe las solicitudes, valida el acceso por rol y consulta PostgreSQL mediante SQLAlchemy. Alembic versiona el esquema y Docker Compose prepara el entorno local. El entorno remoto de pruebas usa Vercel y Neon. La consola de demostración y Swagger permiten explorar la API con datos de prueba. La integración completa con el frontend definitivo queda pendiente de validar.

## Arranque local con Docker

Desde la raíz de una copia del [repositorio de la API](https://github.com/Pombet/agecare-admin-api), con Docker Desktop o Docker Engine y Compose v2 disponibles, ejecutar:

```powershell
docker compose config --quiet
docker compose up --build -d
docker compose ps
docker compose logs --tail=80 api
Invoke-RestMethod http://localhost:8000/health
```

Compose levanta PostgreSQL 16, espera a que esté disponible y ejecuta las migraciones de Alembic y la carga de datos demo antes de iniciar la API. Si el arranque se detiene, revisar los logs del servicio y verificar la disponibilidad de los puertos 5432 y 8000.

Abrir [Swagger local](http://localhost:8000/api/v1/admin/docs) y [la consola local](http://localhost:8000/demo/). Iniciar sesión con una cuenta de prueba y consultar Productos para verificar el acceso a datos. La ruta /health comprueba que el proceso HTTP responde. La consulta de Productos también comprueba que la API puede leer el catálogo. Para detener el entorno, ejecutar docker compose down.

### Evidencia de Docker para la entrega

La configuración de Compose se validó con docker compose config --quiet, que terminó con código 0. La ejecución de contenedores queda pendiente en el equipo de revisión porque el motor de Docker no estaba accesible.

Tras ejecutar el arranque en un equipo que tenga Docker operativo, guardar el commit, la fecha y una captura o registro de docker compose ps, de los logs de migración, de la respuesta de /health y de la consulta de Productos. Guardar las capturas sin contraseñas, tokens ni cadenas de conexión.

## Configuración del entorno remoto de pruebas

- [Consola de demostración en Vercel](https://agecare-admin-pruebas.vercel.app/demo/)
- [Swagger remoto](https://agecare-admin-pruebas.vercel.app/api/v1/admin/docs)
- [Variables, conexión con Neon e integración con frontend](https://github.com/Pombet/agecare-admin-api/blob/main/docs/FRONTEND.md#4-configuraci%C3%B3n-del-backend-remoto)
- [Guía de comprobación de escrituras en Neon](https://github.com/Pombet/agecare-admin-api/blob/main/docs/PRUEBAS_NEON.md)

Configurar la conexión PostgreSQL con TLS y los secretos en el entorno de Vercel. Aplicar las migraciones desde un entorno administrativo autorizado antes de usar un esquema nuevo. Cargar datos de ejemplo únicamente en una base de demostración. La guía de integración documenta la publicación por CLI. Registrar el commit del despliegue y comprobar los flujos después de publicar.

## Verificación de entrega

Para ejecutar las pruebas de despliegue desde la raíz del repositorio de la API, usar `pytest tests/test_deployment.py -q` después de instalar `requirements-dev.txt`. Estas pruebas verifican el arranque, TLS y CORS. Las comprobaciones de persistencia en PostgreSQL se documentan en la guía de Neon enlazada arriba.

| Comprobación | Evidencia disponible o pendiente |
| --- | --- |
| Consulta y filtro del catálogo | Observado en Vercel. El filtro Publicado dejó dos productos de los cuatro que devolvía la consulta general. |
| Auditoría de acceso | Observada en Vercel. El inicio de sesión de la cuenta demo apareció en Actividad. |
| Arranque con Docker | Configuración validada. Falta registrar el arranque de los contenedores y una consulta local. |
| Guardado de cambios | El repositorio contiene una guía de pruebas en Neon. Para acreditar la ejecución, guardar el resultado de una escritura controlada y su consulta posterior en la base demo. |
| Permisos y aislamiento | La interfaz muestra módulos distintos por rol. Falta adjuntar el resultado de las pruebas de autorización por operación y de aislamiento con dos organizaciones en PostgreSQL. |
| Frontend definitivo | Pendiente de validar los recorridos completos. Registrar la versión de ambos repositorios y qué comprobó cada equipo. |
| Datos de ejemplo | Identificar los flujos que usan datos demo y las fuentes reales que faltan integrar. |
