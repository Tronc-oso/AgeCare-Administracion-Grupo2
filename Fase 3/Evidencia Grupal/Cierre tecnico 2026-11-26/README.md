# Cierre técnico de AgeCare Administrativo

Verificación realizada el 26 de noviembre de 2026 en Windows y Docker Desktop. Código base: `fdc9cf5d9ee0d5a1e8ec986db96d33e7548060e6` de Pombet/agecare-admin-api. Se añadió `tzdata>=2024.1` a requirements.txt para que America/Santiago esté disponible también en Windows. El archivo requirements-verified.txt registra las versiones usadas por las pruebas de Python en el host; no representa un congelamiento de las dependencias dentro de la imagen Docker.

## Resultados

- Suite principal sobre PostgreSQL 16: **111 aprobadas de 111**, 0 fallos, 0 errores y 0 omitidas. Evidencia: suite-clean.xml.
- Juegos aislados: **13 aprobadas**, con SQLite e identidad sustituida. Evidencia: games.xml. No verifica RLS PostgreSQL.
- Juegos JavaScript: **19 aprobadas**, sin fallos. Evidencia: games-node.log.
- Las **3 pruebas de despliegue** forman parte de la suite principal; deployment.xml registra además su ejecución separada. No se suman por segunda vez.
- Docker construyó la imagen, levantó PostgreSQL y ejecutó migraciones hasta 0011_billing y la carga demo antes de iniciar la API. Evidencia: docker-ps.json y docker-startup.txt.
- HTTP: salud 200, login 200, identidad 200, creación de ticket 201 y lectura posterior 200. SQL confirmó la fila guardada. Evidencia: runtime.json.
- RLS en support_tickets con dos organizaciones: propia 1 y ajena 0 en ambos sentidos, usando SET LOCAL ROLE agecare_admin_ro. La organización temporal y su ticket se revirtieron mediante rollback.

## Hallazgos y límites

La primera ejecución sobre la base local reutilizada produjo 107 aprobadas y 4 fallos: tres por ausencia de tzdata en Windows y uno por un correo fijo ya registrado por una ejecución anterior. Se declaró e instaló la dependencia y se repitió la suite sobre una base nueva migrada y sembrada. No se modificaron reglas del negocio para forzar el resultado. La suite conserva datos entre casos: repetirla requiere una base de pruebas nueva, especialmente por el correo fijo busqueda.unica@demo.cl. Nunca usar esta suite contra producción.

Hay advertencias de pytest sobre marcas asyncio en funciones síncronas y una clave JWT corta utilizada por las fixtures de pruebas. Registrar el resultado de las pruebas no implica recomendar esa clave para despliegues.

La pasarela se prueba mediante gateways falsos o transportes simulados, sin cobros reales. El repositorio incluye aportes de otros equipos: los resultados no atribuyen al Grupo 2 la autoría completa de juegos, marketplace o pagos. La integración con el frontend definitivo, la versión desplegada en Vercel y el funcionamiento de todas las fuentes externas siguen pendientes de validación conjunta. La prueba RLS adicional cubre una tabla y dos contextos, no todas las políticas y rutas del sistema.

El plan manual original identifica **P1 a P15**. Se consideran 15 casos documentados. La cifra histórica de 35 pruebas corresponde a una etapa anterior; esta ejecución tiene su propia referencia y fecha.

## Reproducción

Desde el repositorio de la API, con Docker operativo:

```powershell
docker compose config --quiet
docker compose up --build -d
docker compose ps
docker compose logs --tail 40 api
Invoke-RestMethod http://localhost:8000/health
```

Para esta revisión se usó un proyecto separado, agecare_evidence_20261126, y el override adjunto: PostgreSQL en 127.0.0.1:56439 y API en 127.0.0.1:58039, para evitar conflictos. La imagen se construyó sobre el commit base; la corrección tzdata fue comprobada con las pruebas del host.

Instalar requirements-dev.txt en un entorno virtual, crear una base exclusiva vacía y configurar ADMIN_DATABASE_URL para ella. Ejecutar alembic upgrade head y python -m scripts.seed. Luego configurar TEST_DATABASE_URL a esa misma base y ejecutar:

```powershell
python -m pytest tests --ignore=tests/games -vv -s -p no:cacheprovider --junitxml=suite.xml
python -m pytest tests/games --confcutdir=tests/games -q
node --test tests/games/new-games.test.cjs tests/games/collection-games.test.cjs
```

La base de la suite y la de la API HTTP se separaron para que las pruebas no dependieran de las escrituras de la demostración. Los datos utilizados son de ejemplo.

Corrección de dependencia publicada en [b060456](https://github.com/Pombet/agecare-admin-api/commit/b060456). La suite de cierre se ejecutó con ese único cambio respecto al commit base.
