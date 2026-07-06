# Guia de levantamiento local CRIT Assist

Esta guia levanta los tres repos locales (`crit-db`, `crit-api`, `crit-front`) para probar el flujo completo del MVP sin usar SQL manual para crear CRITs o administradores iniciales.

## Requisitos

- Docker Desktop corriendo.
- Node.js 20 o posterior.
- npm.
- Repos locales en la misma carpeta padre:
  - `crit-db`
  - `crit-api`
  - `crit-front`

## 1. Base de datos

```powershell
cd C:\Users\esteb\apps\crit-project\crit-db
Copy-Item .env.example .env

docker compose up --build --wait
.\scripts\verify-db.ps1
```

La DB expone dos usuarios de aplicacion locales:

- `crit_app`: APIs por tenant.
- `crit_platform_app`: super admin global y provisioning.

## 2. API

```powershell
cd C:\Users\esteb\apps\crit-project\crit-api
npm install
Copy-Item .env.example .env
```

Editar `.env` y poner passwords locales fuertes en:

- `JWT_SECRET`
- `PLATFORM_JWT_SECRET`
- `PLATFORM_BOOTSTRAP_PASSWORD`
- `BOOTSTRAP_ADMIN_PASSWORD`

Validar y crear usuarios iniciales:

```powershell
npm run db:check
npm run platform:bootstrap-super-admin
npm run admin:bootstrap
```

Levantar procesos en terminales separadas:

```powershell
npm run dev:main
npm run dev:admin
npm run dev:checkin
npm run dev:super-admin
```

Health checks:

- `GET http://localhost:3000/health`
- `GET http://localhost:3001/health`
- `GET http://localhost:3002/health`
- `GET http://localhost:3003/health`

## 3. Frontend

```powershell
cd C:\Users\esteb\apps\crit-project\crit-front
npm install
Copy-Item .env.example .env
npm run dev
```

URLs locales:

- App operativa: `http://localhost:5173/`
- Admin por tenant: `http://localhost:5173/admin.html`
- Super admin global: `http://localhost:5173/super-admin.html`

Variables esperadas:

```env
VITE_MAIN_API_URL=http://localhost:3000/api
VITE_ADMIN_API_URL=http://localhost:3001/admin
VITE_SUPER_ADMIN_API_URL=http://localhost:3003/super-admin
VITE_APP_NAME=CRIT Assistance
```

## 4. Smoke test

1. Entrar a `super-admin.html` con `PLATFORM_BOOTSTRAP_EMAIL` y `PLATFORM_BOOTSTRAP_PASSWORD`.
2. Crear un CRIT nuevo.
3. Crear el primer usuario admin de ese CRIT desde la interfaz.
4. Entrar a `/` con email/password del admin creado.
5. Entrar a `admin.html` y crear catalogos minimos: usuarios, paciente, clinica, room, tipo de cita y colaborador.
6. Crear una cita desde la app operativa.
7. Entrar con usuario de recepcion y abrir el flujo de check-in contra `checkin-api`.
8. Registrar check-in de la cita como `present` o `late`.
9. Entrar con medico/terapeuta y registrar o editar asistencia/nota medica.
10. Confirmar que recepcion y super admin no ven contenido de notas medicas.

## Validaciones

```powershell
# crit-db
.\scripts\verify-db.ps1

# crit-api
npm run build
npm test

# crit-front
npm run build
```

Notas:

- No usar datos reales de pacientes.
- No commitear `.env`.
- Si Docker Desktop no esta corriendo, la verificacion de DB fallara antes de ejecutar SQL.