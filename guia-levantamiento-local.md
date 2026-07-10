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

Si ya tenias un `.env` creado antes de esta guia, revisa que no conserve
passwords vacios. Estas dos lineas no pueden quedar vacias:

```dotenv
PLATFORM_BOOTSTRAP_PASSWORD=local-platform-password-123
BOOTSTRAP_ADMIN_PASSWORD=local-admin-password-123
```

PowerShell rapido para confirmarlo:

```powershell
Select-String -Path .env -Pattern "PLATFORM_BOOTSTRAP_PASSWORD|BOOTSTRAP_ADMIN_PASSWORD"
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

## 2.2. Seed demo completo para smoke visual

Despues del bootstrap obligatorio, ejecuta esta semilla para crear usuarios de
todos los roles y datos minimos para probar el front completo:

PowerShell:

```powershell
cd C:\Users\esteb\apps\crit-project\crit-api
npm run demo:seed-smoke
```

Git Bash/MINGW64:

```bash
cd ~/apps/crit-project/crit-api
npm run demo:seed-smoke
```

La semilla usa por default el tenant `CRIT-OCC-01` y el password
`DemoPassword123`. Si necesitas cambiarlo, define variables antes de correrla:

PowerShell:

```powershell
$env:DEMO_TENANT_CODE="CRIT-OCC-01"
$env:DEMO_USER_PASSWORD="DemoPassword123"
npm run demo:seed-smoke
```

Git Bash/MINGW64:

```bash
DEMO_TENANT_CODE=CRIT-OCC-01 DEMO_USER_PASSWORD=DemoPassword123 npm run demo:seed-smoke
```

Usuarios creados por `npm run demo:seed-smoke`:

| Rol | Email | Password | App sugerida |
| --- | --- | --- | --- |
| Admin general | `demo.admin@crit.test` | `DemoPassword123` | `http://localhost:5173/admin.html` |
| Direccion todas clinicas | `demo.direccion@crit.test` | `DemoPassword123` | `http://localhost:5173/` |
| Recepcion general check-in global | `demo.recepcion.general@crit.test` | `DemoPassword123` | `http://localhost:5173/` |
| Recepcion Norte | `demo.recepcion.norte@crit.test` | `DemoPassword123` | `http://localhost:5173/` |
| Recepcion Sur | `demo.recepcion.sur@crit.test` | `DemoPassword123` | `http://localhost:5173/` |
| Recepcion Infantil | `demo.recepcion.infantil@crit.test` | `DemoPassword123` | `http://localhost:5173/` |
| Coordinador Norte | `demo.coordinador.norte@crit.test` | `DemoPassword123` | `http://localhost:5173/` |
| Coordinador Sur | `demo.coordinador.sur@crit.test` | `DemoPassword123` | `http://localhost:5173/` |
| Medico Norte | `demo.medico.norte@crit.test` | `DemoPassword123` | `http://localhost:5173/` |
| Medico Multi Clinica | `demo.medico.multi@crit.test` | `DemoPassword123` | `http://localhost:5173/` |
| Terapeuta Sur | `demo.terapeuta.sur@crit.test` | `DemoPassword123` | `http://localhost:5173/` |
| Terapeuta Infantil | `demo.terapeuta.infantil@crit.test` | `DemoPassword123` | `http://localhost:5173/` |
| Personal AP Norte | `demo.acompanamiento.norte@crit.test` | `DemoPassword123` | `http://localhost:5173/` |
| Personal AP Sur | `demo.acompanamiento.sur@crit.test` | `DemoPassword123` | `http://localhost:5173/` |
| Paciente/familia | `demo.familia@crit.test` | `DemoPassword123` | Login operativo solo si el front expone flujo familiar |

Tambien conserva aliases de compatibilidad de la semilla anterior:
`demo.recepcion@crit.test`, `demo.coordinador@crit.test`,
`demo.medico@crit.test`, `demo.terapeuta@crit.test` y
`demo.acompanamiento@crit.test`.

Tambien crea datos demo idempotentes:

- Clinicas `Smoke Norte Medicina`, `Smoke Sur Terapia` y `Smoke Infantil Lenguaje`.
- Ocho consultorios/salas repartidos entre esas clinicas.
- Tipos de cita `Smoke Medicina Fisica`, `Smoke Terapia Fisica`, `Smoke Terapia Lenguaje` y `Smoke Valoracion Inicial`.
- Ocho pacientes demo, incluido `DEMO-PAT-SIN-001` para probar gafete valido sin citas.
- Citas en junio, julio y agosto de 2026 con estados `scheduled`, `rescheduled` y `cancelled`.
- Citas con check-in, sin check-in, asistencia, inasistencia, solicitud de reagendar, nota medica y nota de enlace.
- Notificaciones demo para recepcion general, terapeutas y destinatarios de notas de enlace.

El comando es idempotente: se puede ejecutar varias veces y actualiza los mismos
datos demo sin duplicarlos.

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

Camino rapido para validar el front con datos listos:

1. Ejecutar los comandos de `2.1. Bootstrap obligatorio de usuarios locales`.
2. Ejecutar `npm run demo:seed-smoke` desde `crit-api`.
3. Levantar las cuatro APIs y `crit-front`.
4. Entrar a `admin.html` con `demo.admin@crit.test` / `DemoPassword123` y validar usuarios, clinicas, cuartos y colaboradores.
5. Entrar a `/` con `demo.recepcion@crit.test` / `DemoPassword123` y validar check-in/listados operativos.
6. Entrar a `/` con `demo.terapeuta@crit.test` o `demo.medico@crit.test` / `DemoPassword123` y validar asistencias, notas medicas y notas de enlace.
7. Entrar a `/` con `demo.direccion@crit.test` / `DemoPassword123` y validar vista global de notas de enlace y resumen operativo.

Camino manual para probar creacion desde cero:

1. Entrar a `super-admin.html` con `platform.admin@crit.test` / `local-platform-password-123`.
2. Crear un CRIT nuevo si quieres probar multi-CRIT, o usar el tenant seed `CRIT-OCC-01` con `admin.local@crit.test`.
3. Crear el primer usuario admin de ese CRIT desde super admin, o entrar a `admin.html` con el admin demo.
4. Entrar a la app/admin con el admin creado usando solo email y password.
5. Crear catalogos minimos: clinica, cuarto, tipo de cita, paciente, usuario recepcion, usuario medico/terapeuta y colaborador asociado.
6. Crear una cita en Main API/app.
7. Reprogramar o cancelar la cita si aplica.
8. Entrar como recepcion o abrir `checkin.html` con una sesion valida y registrar check-in. La respuesta no debe incluir contenido clinico ni notas medicas.
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
