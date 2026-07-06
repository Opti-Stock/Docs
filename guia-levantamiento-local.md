# Guia de levantamiento local CRIT MVP

Esta guia levanta `crit-db`, `crit-api` y `crit-front` para probar el MVP end-to-end: super admin crea un CRIT, crea el primer admin, el admin crea catalogos, recepcion hace check-in y el clinico registra asistencia/nota.

## Requisitos

- Node.js compatible con los repos.
- Docker Desktop corriendo.
- GitHub CLI autenticado si se van a revisar PRs.
- Trabajar desde `dev` o un branch basado en `dev`.

En Windows usa `127.0.0.1` para las URLs de PostgreSQL del host. Evita `localhost` si ves timeouts desde Node hacia Docker.

## 1. Base de datos

```powershell
cd C:\Users\esteb\apps\crit-project\crit-db
Copy-Item .env.example .env -ErrorAction SilentlyContinue
docker compose up --build --wait
.\scripts\verify-db.ps1
```

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
PLATFORM_BOOTSTRAP_FULL_NAME=Platform Super Admin
PLATFORM_BOOTSTRAP_EMAIL=platform.admin@crit.test
PLATFORM_BOOTSTRAP_PASSWORD=local-platform-password-123
```

Validar DB y crear/refrescar el super admin inicial:

```powershell
npm run db:check
npm run platform:bootstrap-super-admin
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

Config local minima en `crit-front/.env`:

```dotenv
VITE_MAIN_API_URL=http://localhost:3000/api
VITE_ADMIN_API_URL=http://localhost:3001/admin
VITE_SUPER_ADMIN_API_URL=http://localhost:3003/super-admin
```

Entradas locales:

- Main app: `http://localhost:5173/`
- Admin app: `http://localhost:5173/admin.html`
- Super admin app: `http://localhost:5173/super-admin.html`

## 4. Smoke test end-to-end

1. Entrar a `super-admin.html` con `platform.admin@crit.test` / `local-platform-password-123`.
2. Crear un CRIT nuevo.
3. Crear el primer usuario admin de ese CRIT.
4. Entrar a la app/admin con el admin creado usando solo email y password.
5. Crear catalogos minimos: clinica, cuarto, tipo de cita, paciente, usuario recepcion, usuario medico/terapeuta y colaborador asociado.
6. Crear una cita en Main API/app.
7. Reprogramar o confirmar la cita.
8. Entrar como recepcion y hacer check-in desde Check-in API/app. La respuesta no debe incluir contenido clinico ni notas medicas.
9. Entrar como clinico, actualizar asistencia y crear/editar nota medica.
10. Regresar a super admin y verificar el resumen operativo del CRIT sin contenido clinico.

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

Resultados esperados actuales:

- `crit-db`: `crit-db verification passed`.
- `crit-api`: build TypeScript sin errores y suite de tests en verde.
- `crit-front`: build genera `dist/index.html`, `dist/admin.html` y `dist/super-admin.html`.

## 6. PRs de frontend

Los PRs activos de `crit-front` deben apuntar a `dev`, no a `main`:

```powershell
gh pr view 11 --repo Opti-Stock/crit-front --json baseRefName,mergeable,url
gh pr view 12 --repo Opti-Stock/crit-front --json baseRefName,mergeable,url
gh pr view 13 --repo Opti-Stock/crit-front --json baseRefName,mergeable,url
```

Cada uno debe mostrar `baseRefName: dev` y `mergeable: MERGEABLE` antes de mergear.