# Guia de levantamiento local CRIT MVP

Esta guia levanta `crit-db`, `crit-api` y `crit-front` para probar el MVP end-to-end: super admin crea un CRIT, crea el primer admin, el admin crea catalogos, recepcion hace check-in y el clinico registra asistencia/nota.

## Requisitos

- Node.js compatible con los repos.
- Docker Desktop corriendo.
- GitHub CLI autenticado si se van a revisar PRs.
- Trabajar desde `dev` o un branch basado en `dev`.

En Windows usa `127.0.0.1` para las URLs de PostgreSQL del host. Evita `localhost` si ves timeouts desde Node hacia Docker.

## 1. Base de datos

PowerShell:

```powershell
cd C:\Users\esteb\apps\crit-project\crit-db
Copy-Item .env.example .env -ErrorAction SilentlyContinue
docker compose up --build --wait
.\scripts\verify-db.ps1
```

Git Bash/MINGW64:

```bash
cd ~/apps/crit-project/crit-db
cp -n .env.example .env 2>/dev/null || true
docker compose up --build --wait
bash ./scripts/verify-db.sh
```

No uses `.\scripts\verify-db.ps1` en Git Bash: las diagonales invertidas se interpretan distinto y Bash termina buscando `.scriptsverify-db.ps1`.

La verificacion debe terminar con:

```text
crit-db verification passed
```

Variables clave de `crit-db/.env`:

```dotenv
POSTGRES_DB=crit_db
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
APP_DB_USER=crit_app
APP_DB_PASSWORD=crit_app
PLATFORM_DB_USER=crit_platform_app
PLATFORM_DB_PASSWORD=crit_platform_app
```

## 2. API

```powershell
cd C:\Users\esteb\apps\crit-project\crit-api
Copy-Item .env.example .env -ErrorAction SilentlyContinue
npm install
```

En Git Bash:

```bash
cd ~/apps/crit-project/crit-api
cp -n .env.example .env 2>/dev/null || true
npm install
```

Config local minima en `crit-api/.env`:

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

`CORS_ORIGIN` tiene default local en la API, pero puedes dejarlo explicito en `.env` para que el archivo sea facil de leer.

## 2.1. Bootstrap obligatorio de usuarios locales

Estos comandos son necesarios en una base local limpia. Sin ellos no existe el
super admin de plataforma y nadie puede entrar a `super-admin.html`.

PowerShell:

```powershell
npm run db:check
npm run platform:bootstrap-super-admin
npm run admin:bootstrap
```

Git Bash/MINGW64:

```bash
npm run db:check
npm run platform:bootstrap-super-admin
npm run admin:bootstrap
```

Usuarios locales que quedan disponibles:

| App | URL | Email | Password | Uso |
| --- | --- | --- | --- | --- |
| Super admin | `http://localhost:5173/super-admin.html` | `platform.admin@crit.test` | `local-platform-password-123` | Crear CRITs y primer admin por CRIT |
| Admin demo | `http://localhost:5173/admin.html` | `admin.local@crit.test` | `local-admin-password-123` | Entrar al tenant seed `CRIT-OCC-01` y crear catalogos/usuarios |

Smoke login por terminal, con las APIs ya levantadas:

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
  -Body '{"email":"admin.local@crit.test","password":"local-admin-password-123"}'
```

Ambos comandos deben regresar `success: true` y un `accessToken`.

Notas importantes:

- `npm run platform:bootstrap-super-admin` es idempotente para
  `platform.admin@crit.test`: si ya existe, refresca nombre/password.
- `npm run admin:bootstrap` crea el primer admin del tenant seed
  `CRIT-OCC-01`. Si ya hay otro admin activo en ese tenant, el script falla para
  no pisar datos de otra persona.
- El tenant seed `CRIT-OCC-01` viene de `crit-db`; no trae usuarios por diseño.
- Para repetir desde cero en local, recrea la DB con cuidado:

```powershell
cd C:\Users\esteb\apps\crit-project\crit-db
docker compose down -v
docker compose up --build --wait
cd ..\crit-api
npm run platform:bootstrap-super-admin
npm run admin:bootstrap
```

Levantar APIs en cuatro terminales:

```powershell
npm run dev:main
npm run dev:admin
npm run dev:checkin
npm run dev:super-admin
```

Health checks:

```powershell
Invoke-RestMethod http://localhost:3000/health
Invoke-RestMethod http://localhost:3001/health
Invoke-RestMethod http://localhost:3002/health
Invoke-RestMethod http://localhost:3003/health
```

Bases de rutas:

- Main API: `http://localhost:3000/api`
- Admin API: `http://localhost:3001/admin`
- Check-in API: `http://localhost:3002/checkin`
- Super admin API: `http://localhost:3003/super-admin`

Login operativo: enviar solo `email` y `password`; no enviar `tenantCode`.

## 3. Frontend

```powershell
cd C:\Users\esteb\apps\crit-project\crit-front
Copy-Item .env.example .env -ErrorAction SilentlyContinue
npm install
npm run dev
```

En Git Bash:

```bash
cd ~/apps/crit-project/crit-front
cp -n .env.example .env 2>/dev/null || true
npm install
npm run dev
```

Config local minima en `crit-front/.env`:

```dotenv
VITE_MAIN_API_URL=http://localhost:3000/api
VITE_ADMIN_API_URL=http://localhost:3001/admin
VITE_CHECKIN_API_URL=http://localhost:3002/checkin
VITE_SUPER_ADMIN_API_URL=http://localhost:3003/super-admin
```

Entradas locales:

- Main app: `http://localhost:5173/`
- Admin app: `http://localhost:5173/admin.html`
- Check-in app: `http://localhost:5173/checkin.html`
- Super admin app: `http://localhost:5173/super-admin.html`

## 4. Smoke test end-to-end

1. Ejecutar los comandos de `2.1. Bootstrap obligatorio de usuarios locales`.
2. Entrar a `super-admin.html` con `platform.admin@crit.test` / `local-platform-password-123`.
3. Crear un CRIT nuevo si quieres probar multi-CRIT, o usar el tenant seed `CRIT-OCC-01` con `admin.local@crit.test`.
4. Crear el primer usuario admin de ese CRIT desde super admin, o entrar a `admin.html` con el admin demo.
5. Entrar a la app/admin con el admin creado usando solo email y password.
6. Crear catalogos minimos: clinica, cuarto, tipo de cita, paciente, usuario recepcion, usuario medico/terapeuta y colaborador asociado.
7. Crear una cita en Main API/app.
8. Reprogramar o cancelar la cita si aplica.
9. Entrar como recepcion o abrir `checkin.html` con una sesion valida y registrar check-in. La respuesta no debe incluir contenido clinico ni notas medicas.
10. Entrar como clinico, actualizar asistencia y crear/editar nota medica.
11. Regresar a super admin y verificar el resumen operativo del CRIT sin contenido clinico.

## 5. Validaciones esperadas

```powershell
# crit-db
.\scripts\verify-db.ps1

# crit-api
npm run build
npm test

# crit-front
npm run build
```

En Git Bash usa `bash ./scripts/verify-db.sh` para `crit-db`.

Resultados esperados actuales:

- `crit-db`: `crit-db verification passed`.
- `crit-api`: build TypeScript sin errores y suite de tests en verde.
- `crit-front`: build genera `dist/index.html`, `dist/admin.html`, `dist/checkin.html` y `dist/super-admin.html`.

## 6. PRs de frontend

Los PRs activos de `crit-front` deben apuntar a `dev`, no a `main`:

```powershell
gh pr view 11 --repo Opti-Stock/crit-front --json baseRefName,mergeable,url
gh pr view 12 --repo Opti-Stock/crit-front --json baseRefName,mergeable,url
gh pr view 13 --repo Opti-Stock/crit-front --json baseRefName,mergeable,url
```

Cada uno debe mostrar `baseRefName: dev` y `mergeable: MERGEABLE` antes de mergear.
