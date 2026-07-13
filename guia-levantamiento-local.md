# Guia de levantamiento local CRIT MVP

Esta guia explica como levantar `crit-db`, `crit-api` y `crit-front` para probar el MVP end-to-end.

Esta pensada para dos escenarios:

1. **Instalacion local desde cero**: la persona aun no tiene DB local, `.env` ni datos demo.
2. **Ya tenian una version anterior**: la DB ya levanta, hay datos viejos, o quieren reiniciar todo para probar limpio.

Los comandos usan rutas relativas. Se asume que abres una terminal en una carpeta de proyecto que contiene los tres repos:

```text
crit-project/
  crit-db/
  crit-api/
  crit-front/
  Docs/
```

Si aun no tienes los repos, crea la carpeta y clonalos ahi:

```bash
mkdir crit-project
cd crit-project
git clone https://github.com/Opti-Stock/crit-db.git
git clone https://github.com/Opti-Stock/crit-api.git
git clone https://github.com/Opti-Stock/crit-front.git
git clone https://github.com/Opti-Stock/Docs.git
```

## Requisitos

- Docker Desktop corriendo.
- Node.js compatible con los repos.
- npm.
- Git.
- GitHub CLI solo si vas a revisar o crear PRs.
- Trabajar con los repos actualizados.

En Windows, para las URLs de PostgreSQL usadas por Node, usa `127.0.0.1` en lugar de `localhost` si ves timeouts desde la API hacia Docker.

## Ramas recomendadas

En `crit-api`, `crit-db` y `crit-front` normalmente se trabaja desde `dev`:

```bash
cd crit-api
git switch dev
git pull --ff-only
cd ..

cd crit-db
git switch dev
git pull --ff-only
cd ..

cd crit-front
git switch dev
git pull --ff-only
cd ..
```

El repo `Docs` actualmente usa `main` como rama base.

## Puertos locales

| Servicio | Puerto | URL |
| --- | ---: | --- |
| Main API | 3000 | `http://localhost:3000/api` |
| Admin API | 3001 | `http://localhost:3001/admin` |
| Check-in API | 3002 | `http://localhost:3002/checkin` |
| Super admin API | 3003 | `http://localhost:3003/super-admin` |
| Frontend | 5173 | `http://localhost:5173/` |
| PostgreSQL | 5432 | `127.0.0.1:5432` |

## Variables locales esperadas

### `crit-db/.env`

Si no existe, copia el ejemplo:

PowerShell:

```powershell
cd crit-db
Copy-Item .env.example .env -ErrorAction SilentlyContinue
cd ..
```

Git Bash:

```bash
cd crit-db
cp -n .env.example .env 2>/dev/null || true
cd ..
```

Valores minimos esperados:

```dotenv
POSTGRES_DB=crit_db
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
APP_DB_USER=crit_app
APP_DB_PASSWORD=crit_app
PLATFORM_DB_USER=crit_platform_app
PLATFORM_DB_PASSWORD=crit_platform_app
```

### `crit-api/.env`

Si no existe, copia el ejemplo:

PowerShell:

```powershell
cd crit-api
Copy-Item .env.example .env -ErrorAction SilentlyContinue
cd ..
```

Git Bash:

```bash
cd crit-api
cp -n .env.example .env 2>/dev/null || true
cd ..
```

Valores minimos recomendados:

```dotenv
NODE_ENV=development
DATABASE_URL=postgresql://crit_app:crit_app@127.0.0.1:5432/crit_db
PLATFORM_DATABASE_URL=postgresql://crit_platform_app:crit_platform_app@127.0.0.1:5432/crit_db
MAIN_API_PORT=3000
ADMIN_API_PORT=3001
CHECKIN_API_PORT=3002
SUPER_ADMIN_API_PORT=3003
JWT_SECRET=local_jwt_secret_at_least_32_chars
PLATFORM_JWT_SECRET=local_platform_secret_at_least_32_chars
CORS_ORIGIN=http://localhost:5173

PLATFORM_BOOTSTRAP_FULL_NAME=Platform Super Admin
PLATFORM_BOOTSTRAP_EMAIL=platform.admin@crit.test
PLATFORM_BOOTSTRAP_PASSWORD=local-platform-password-123

BOOTSTRAP_ADMIN_TENANT_CODE=CRIT-OCC-01
BOOTSTRAP_ADMIN_FULL_NAME=Local Smoke Admin
BOOTSTRAP_ADMIN_EMAIL=admin.local@crit.test
BOOTSTRAP_ADMIN_PASSWORD=local-admin-password-123
```

Estas variables no pueden quedar vacias:

```dotenv
PLATFORM_BOOTSTRAP_EMAIL
PLATFORM_BOOTSTRAP_PASSWORD
BOOTSTRAP_ADMIN_EMAIL
BOOTSTRAP_ADMIN_PASSWORD
```

El password de admin debe tener al menos 12 caracteres.

Para validar rapido:

PowerShell:

```powershell
cd crit-api
Select-String -Path .env -Pattern "PLATFORM_BOOTSTRAP|BOOTSTRAP_ADMIN"
cd ..
```

Git Bash:

```bash
cd crit-api
grep -E "PLATFORM_BOOTSTRAP|BOOTSTRAP_ADMIN" .env
cd ..
```

### `crit-front/.env`

Si no existe, copia el ejemplo:

PowerShell:

```powershell
cd crit-front
Copy-Item .env.example .env -ErrorAction SilentlyContinue
cd ..
```

Git Bash:

```bash
cd crit-front
cp -n .env.example .env 2>/dev/null || true
cd ..
```

Valores minimos:

```dotenv
VITE_MAIN_API_URL=http://localhost:3000/api
VITE_ADMIN_API_URL=http://localhost:3001/admin
VITE_CHECKIN_API_URL=http://localhost:3002/checkin
VITE_SUPER_ADMIN_API_URL=http://localhost:3003/super-admin
```

## Escenario A: instalacion local desde cero

Usa este camino cuando vas a preparar la app por primera vez en una maquina.

### A1. Actualizar repos

Desde `crit-project/`:

```bash
cd crit-db
git switch dev
git pull --ff-only
cd ..

cd crit-api
git switch dev
git pull --ff-only
cd ..

cd crit-front
git switch dev
git pull --ff-only
cd ..
```

### A2. Crear archivos `.env`

Sigue la seccion **Variables locales esperadas** y revisa especialmente `crit-api/.env`.

### A3. Levantar DB

PowerShell:

```powershell
cd crit-db
docker compose up --build --wait
.\scripts\verify-db.ps1
cd ..
```

Git Bash:

```bash
cd crit-db
docker compose up --build --wait
bash ./scripts/verify-db.sh
cd ..
```

No uses `.\scripts\verify-db.ps1` en Git Bash: Bash interpreta diferente las diagonales invertidas y puede terminar buscando `.scriptsverify-db.ps1`.

La verificacion debe terminar con:

```text
crit-db verification passed
```

### A4. Instalar API y crear usuarios base

Desde `crit-project/`:

```bash
cd crit-api
npm install
npm run db:check
npm run platform:bootstrap-super-admin
npm run admin:bootstrap
cd ..
```

Estos comandos crean:

| Usuario | Email | Password | Uso |
| --- | --- | --- | --- |
| Super admin | `platform.admin@crit.test` | `local-platform-password-123` | Entrar a `super-admin.html`, crear CRITs y primer admin |
| Admin local | `admin.local@crit.test` | `local-admin-password-123` | Entrar a `admin.html` en el tenant seed `CRIT-OCC-01` |

### A5. Crear datos demo completos

Desde `crit-project/`:

```bash
cd crit-api
npm run demo:seed-smoke
cd ..
```

La semilla usa por default:

```text
Tenant: CRIT-OCC-01
Password demo: DemoPassword123
```

Si necesitas cambiarlo:

PowerShell:

```powershell
cd crit-api
$env:DEMO_TENANT_CODE="CRIT-OCC-01"
$env:DEMO_USER_PASSWORD="DemoPassword123"
npm run demo:seed-smoke
cd ..
```

Git Bash:

```bash
cd crit-api
DEMO_TENANT_CODE=CRIT-OCC-01 DEMO_USER_PASSWORD=DemoPassword123 npm run demo:seed-smoke
cd ..
```

### A6. Levantar APIs con Docker

La forma recomendada para probar como equipo es levantar la API con Docker.
Este `docker compose` inicia los cuatro servicios:

- `main-api` en `3000`.
- `admin-api` en `3001`.
- `checkin-api` en `3002`.
- `super-admin-api` en `3003`.

Importante: primero debe estar arriba `crit-db`, porque `crit-api/docker-compose.yml` se conecta a la red externa `crit-db_default`.

```bash
cd crit-api
docker compose up --build
```

Si quieres dejar la terminal libre, puedes usar:

```bash
cd crit-api
docker compose up --build -d
```

Para ver logs:

```bash
cd crit-api
docker compose logs -f
```

Para apagar solo las APIs:

```bash
cd crit-api
docker compose down
```

Health checks:

PowerShell:

```powershell
Invoke-RestMethod http://localhost:3000/health
Invoke-RestMethod http://localhost:3001/health
Invoke-RestMethod http://localhost:3002/health
Invoke-RestMethod http://localhost:3003/health
```

Git Bash:

```bash
curl http://localhost:3000/health
curl http://localhost:3001/health
curl http://localhost:3002/health
curl http://localhost:3003/health
```

Alternativa para desarrollo sin Docker: si necesitas hot reload o depurar un servicio especifico, puedes levantar cada API con npm en terminales separadas desde `crit-api`:

```bash
npm run dev:main
npm run dev:admin
npm run dev:checkin
npm run dev:super-admin
```

### A7. Levantar frontend

En otra terminal:

```bash
cd crit-front
npm install
npm run dev
```

Entradas:

| App | URL |
| --- | --- |
| Main app | `http://localhost:5173/` |
| Admin app | `http://localhost:5173/admin.html` |
| Check-in app | `http://localhost:5173/checkin.html` |
| Super admin app | `http://localhost:5173/super-admin.html` |

## Escenario B: ya tenian una version anterior o quieren reiniciar DB

Usa este camino cuando:

- Ya tenias `.env` anteriores.
- La DB levanta, pero trae datos viejos.
- Quieres borrar volumenes y regenerar todo.
- El equipo no puede entrar porque faltan usuarios bootstrap.
- Cambiaron migraciones/seeds y quieres partir limpio.

### B1. Actualizar repos

Desde `crit-project/`:

```bash
cd crit-db
git switch dev
git pull --ff-only
cd ..

cd crit-api
git switch dev
git pull --ff-only
cd ..

cd crit-front
git switch dev
git pull --ff-only
cd ..
```

### B2. Revisar `.env` existentes

No sobrescribas `.env` si ya tiene valores utiles. Solo confirma que existan las variables nuevas.

En `crit-api/.env`, revisa:

```dotenv
CORS_ORIGIN=http://localhost:5173
DATABASE_URL=postgresql://crit_app:crit_app@127.0.0.1:5432/crit_db
PLATFORM_DATABASE_URL=postgresql://crit_platform_app:crit_platform_app@127.0.0.1:5432/crit_db
PLATFORM_BOOTSTRAP_EMAIL=platform.admin@crit.test
PLATFORM_BOOTSTRAP_PASSWORD=local-platform-password-123
BOOTSTRAP_ADMIN_EMAIL=admin.local@crit.test
BOOTSTRAP_ADMIN_PASSWORD=local-admin-password-123
```

Si `PLATFORM_BOOTSTRAP_PASSWORD` o `BOOTSTRAP_ADMIN_PASSWORD` estan vacios, los bootstrap van a fallar.

### B3. Reiniciar DB completamente

Advertencia: esto borra los datos locales porque elimina el volumen de Docker.

PowerShell:

```powershell
cd crit-db
docker compose down -v
docker compose up --build --wait
.\scripts\verify-db.ps1
cd ..
```

Git Bash:

```bash
cd crit-db
docker compose down -v
docker compose up --build --wait
bash ./scripts/verify-db.sh
cd ..
```

### B4. Reinstalar dependencias si cambio `package-lock.json`

```bash
cd crit-api
npm install
cd ..

cd crit-front
npm install
cd ..
```

### B5. Recrear usuarios base y datos demo

```bash
cd crit-api
npm run db:check
npm run platform:bootstrap-super-admin
npm run admin:bootstrap
npm run demo:seed-smoke
cd ..
```

### B6. Levantar todo otra vez

Usa los mismos pasos de:

- **A6. Levantar APIs con Docker**
- **A7. Levantar frontend**

## Usuarios demo

Password para usuarios demo:

```text
DemoPassword123
```

| Rol | Email | App sugerida |
| --- | --- | --- |
| Admin general | `demo.admin@crit.test` | `http://localhost:5173/admin.html` |
| Direccion todas clinicas | `demo.direccion@crit.test` | `http://localhost:5173/` |
| Recepcion general check-in global | `demo.recepcion.general@crit.test` | `http://localhost:5173/` |
| Recepcion Norte | `demo.recepcion.norte@crit.test` | `http://localhost:5173/` |
| Recepcion Sur | `demo.recepcion.sur@crit.test` | `http://localhost:5173/` |
| Recepcion Infantil | `demo.recepcion.infantil@crit.test` | `http://localhost:5173/` |
| Coordinador Norte | `demo.coordinador.norte@crit.test` | `http://localhost:5173/` |
| Coordinador Sur | `demo.coordinador.sur@crit.test` | `http://localhost:5173/` |
| Medico Norte | `demo.medico.norte@crit.test` | `http://localhost:5173/` |
| Medico Multi Clinica | `demo.medico.multi@crit.test` | `http://localhost:5173/` |
| Terapeuta Sur | `demo.terapeuta.sur@crit.test` | `http://localhost:5173/` |
| Terapeuta Infantil | `demo.terapeuta.infantil@crit.test` | `http://localhost:5173/` |
| Personal AP Norte | `demo.acompanamiento.norte@crit.test` | `http://localhost:5173/` |
| Personal AP Sur | `demo.acompanamiento.sur@crit.test` | `http://localhost:5173/` |
| Paciente/familia | `demo.familia@crit.test` | Solo si el front expone flujo familiar |

Aliases de compatibilidad:

```text
demo.recepcion@crit.test
demo.coordinador@crit.test
demo.medico@crit.test
demo.terapeuta@crit.test
demo.acompanamiento@crit.test
```

## Datos que crea `npm run demo:seed-smoke`

La semilla es idempotente: se puede ejecutar varias veces y actualiza los mismos datos demo sin duplicarlos.

Crea, como minimo:

- 1 tenant demo: `CRIT-OCC-01`.
- 1 super admin por bootstrap.
- 1 admin local por bootstrap.
- Varias clinicas demo.
- Varios consultorios/salas por clinica.
- Tipos de cita demo.
- Varios usuarios por rol.
- Pacientes demo con codigos de gafete.
- Citas en junio, julio y agosto de 2026.
- Citas para semana actual y siguiente, en distintos dias.
- Estados de cita `scheduled`, `rescheduled` y `cancelled`.
- Check-ins, asistencias, inasistencias y solicitudes de reagendar.
- Notas medicas.
- Notas de enlace.
- Notificaciones demo.

Tambien incluye un paciente con gafete valido sin citas para probar el flujo de recepcion:

```text
DEMO-PAT-SIN-001
```

## Smoke test recomendado

Con DB, APIs y front levantados:

1. Entrar a `http://localhost:5173/super-admin.html` con `platform.admin@crit.test` / `local-platform-password-123`.
2. Confirmar que el super admin puede ver o crear CRITs.
3. Entrar a `http://localhost:5173/admin.html` con `demo.admin@crit.test` / `DemoPassword123`.
4. Confirmar usuarios, clinicas, consultorios, colaboradores y tipos de cita.
5. Entrar a `http://localhost:5173/` con `demo.recepcion.general@crit.test` / `DemoPassword123`.
6. Abrir escaneo de gafete y confirmar que ve todas las citas para check-in general.
7. Entrar con `demo.recepcion.norte@crit.test` y confirmar que ve su alcance de recepcion.
8. Entrar con `demo.medico.norte@crit.test`.
9. Probar asistencias, inasistencias, solicitud de reagendar y nota medica.
10. Entrar con `demo.terapeuta.sur@crit.test` y probar notas de enlace.
11. Entrar con `demo.coordinador.norte@crit.test` y confirmar que ve lo propio y lo de su area.
12. Entrar con `demo.direccion@crit.test` y confirmar que consulta sin modificar.
13. Revisar que el globo de notificaciones solo cuente no leidas del usuario actual.

## Smoke login por terminal

Con APIs levantadas:

PowerShell:

```powershell
$platformLogin = Invoke-RestMethod `
  -Method Post `
  -Uri http://localhost:3003/super-admin/auth/login `
  -ContentType "application/json" `
  -Body '{"email":"platform.admin@crit.test","password":"local-platform-password-123"}'

$adminLogin = Invoke-RestMethod `
  -Method Post `
  -Uri http://localhost:3000/api/auth/login `
  -ContentType "application/json" `
  -Body '{"email":"demo.admin@crit.test","password":"DemoPassword123"}'
```

Git Bash:

```bash
curl -s http://localhost:3003/super-admin/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"platform.admin@crit.test","password":"local-platform-password-123"}'

curl -s http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"demo.admin@crit.test","password":"DemoPassword123"}'
```

Ambos deben regresar `success: true` y un `accessToken`.

Login operativo: enviar solo `email` y `password`; no enviar `tenantCode`.

## Validaciones de build

Desde cada repo:

```bash
cd crit-db
docker compose up --build --wait
bash ./scripts/verify-db.sh
cd ..

cd crit-api
npm run build
npm test
# requiere que crit-db siga levantado
docker compose up --build -d
docker compose ps
docker compose down
cd ..

cd crit-front
npm run build
cd ..
```

En PowerShell para `crit-db` puedes usar:

```powershell
.\scripts\verify-db.ps1
```

Resultados esperados:

- `crit-db`: `crit-db verification passed`.
- `crit-api`: build TypeScript sin errores y tests en verde.
- `crit-front`: build genera `dist/index.html`, `dist/admin.html`, `dist/checkin.html` y `dist/super-admin.html`.

## Problemas comunes

### `CORS_ORIGIN: Invalid input`

Agrega en `crit-api/.env`:

```dotenv
CORS_ORIGIN=http://localhost:5173
```

### Bootstrap falla por password requerido

Revisa que existan:

```dotenv
PLATFORM_BOOTSTRAP_PASSWORD=local-platform-password-123
BOOTSTRAP_ADMIN_PASSWORD=local-admin-password-123
```

`BOOTSTRAP_ADMIN_PASSWORD` debe tener al menos 12 caracteres.

### Git Bash dice `.scriptsverify-db.ps1: command not found`

Usa:

```bash
bash ./scripts/verify-db.sh
```

### API no conecta con Postgres

Confirma que DB esta levantada y que `crit-api/.env` usa `127.0.0.1`:

```dotenv
DATABASE_URL=postgresql://crit_app:crit_app@127.0.0.1:5432/crit_db
PLATFORM_DATABASE_URL=postgresql://crit_platform_app:crit_platform_app@127.0.0.1:5432/crit_db
```

### No aparecen cambios del front

Deten `npm run dev`, vuelve a levantarlo y haz hard refresh del navegador.

