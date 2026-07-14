# Guion de prueba demo CRIT Assist

Este documento guia una prueba de punta a punta del demo publicado en Render. La idea es que cada persona tome un rol, siga una historia breve y deje evidencia verbal de que el flujo le hizo sentido.

No uses datos reales. Todos los pacientes, correos y codigos de esta guia son ficticios.

## URLs

| Aplicacion | URL |
| --- | --- |
| Operacion principal | `https://crit-assist-demo.onrender.com` |
| Admin tenant | `https://crit-assist-demo.onrender.com/admin.html` |
| Check-in | `https://crit-assist-demo.onrender.com/checkin.html` |
| Super admin | `https://crit-assist-demo.onrender.com/super-admin.html` |

## Passwords para la prueba

Por seguridad, este archivo no guarda contrasenas reales. El facilitador debe compartirlas por canal privado antes de la sesion.

| Password a compartir | Quien la usa |
| --- | --- |
| `PLATFORM_BOOTSTRAP_PASSWORD` | Super admin global |
| `DEMO_USER_PASSWORD` | Todos los usuarios `demo.*@crit.test` |
| `BOOTSTRAP_ADMIN_PASSWORD` | Solo si se prueba el admin creado por bootstrap antes de correr el seed |

## Usuarios demo

| Persona demo | Rol | Email | Password |
| --- | --- | --- | --- |
| Super Admin CRIT Demo | Super admin global | `super.admin@crit.test` | `PLATFORM_BOOTSTRAP_PASSWORD` |
| Andrea Morales Torres | Admin | `demo.admin@crit.test` | `DEMO_USER_PASSWORD` |
| Fernando Rivas Camacho | Direccion | `demo.direccion@crit.test` | `DEMO_USER_PASSWORD` |
| Laura Jimenez Prado | Recepcion general | `demo.recepcion.general@crit.test` | `DEMO_USER_PASSWORD` |
| Mariana Perez Salas | Recepcion Norte | `demo.recepcion.norte@crit.test` | `DEMO_USER_PASSWORD` |
| Sofia Herrera Lopez | Recepcion Sur | `demo.recepcion.sur@crit.test` | `DEMO_USER_PASSWORD` |
| Claudia Navarro Ruiz | Recepcion Infantil | `demo.recepcion.infantil@crit.test` | `DEMO_USER_PASSWORD` |
| Jorge Castillo Mendoza | Coordinador Norte | `demo.coordinador.norte@crit.test` | `DEMO_USER_PASSWORD` |
| Paola Sanchez Vega | Coordinador Sur | `demo.coordinador.sur@crit.test` | `DEMO_USER_PASSWORD` |
| Dr. Ricardo Aguilar Medina | Medico Norte | `demo.medico.norte@crit.test` | `DEMO_USER_PASSWORD` |
| Dra. Natalia Fuentes Lara | Medico multi clinica | `demo.medico.multi@crit.test` | `DEMO_USER_PASSWORD` |
| T.F. Daniela Ortega Cruz | Terapeuta Sur | `demo.terapeuta.sur@crit.test` | `DEMO_USER_PASSWORD` |
| T.L. Monica Reyes Pineda | Terapeuta Infantil | `demo.terapeuta.infantil@crit.test` | `DEMO_USER_PASSWORD` |
| Victor Salazar Nunez | Personal de acompanamiento Norte | `demo.acompanamiento.norte@crit.test` | `DEMO_USER_PASSWORD` |
| Elena Cardenas Soto | Personal de acompanamiento Sur | `demo.acompanamiento.sur@crit.test` | `DEMO_USER_PASSWORD` |
| Gabriela Lopez Martinez | Paciente/familia reservado | `demo.familia@crit.test` | `DEMO_USER_PASSWORD` |

## Historia sugerida para usuarios de prueba

### 1. Direccion crea el contexto del demo

Responsable: facilitador o persona con rol de direccion.

1. Entrar a `https://crit-assist-demo.onrender.com` con `demo.direccion@crit.test`.
2. Revisar que pueda ver informacion operativa de varias clinicas.
3. Confirmar que no esta capturando notas clinicas ni realizando acciones de recepcion.
4. Objetivo de validacion: direccion entiende el estado del centro sin meterse al trabajo clinico diario.

### 2. Super admin revisa el centro CRIT

Responsable: persona que prueba administracion global.

1. Entrar a `https://crit-assist-demo.onrender.com/super-admin.html`.
2. Iniciar sesion con `super.admin@crit.test`.
3. Revisar la lista de centros o tenants.
4. Confirmar que existe el tenant `CRIT-OCC-01`.
5. Si el flujo lo permite, abrir el detalle del tenant y validar que este activo.
6. Objetivo de validacion: el super admin puede ver y preparar centros sin leer contenido clinico.

### 3. Admin valida usuarios, roles y catalogos

Responsable: admin tenant.

1. Entrar a `https://crit-assist-demo.onrender.com/admin.html` con `demo.admin@crit.test`.
2. Revisar usuarios y confirmar que existen recepcion, coordinacion, medicos, terapeutas, direccion y acompanamiento.
3. Revisar roles y permisos disponibles.
4. Revisar clinicas y consultorios:
   - CRIT Medicina Fisica Norte.
   - CRIT Terapia Fisica Sur.
   - CRIT Lenguaje y Neurodesarrollo.
5. Revisar colaboradores asociados a sus clinicas.
6. Objetivo de validacion: admin puede preparar la operacion sin tocar notas medicas.

### 4. Recepcion general hace check-in de un paciente

Responsable: recepcion general.

1. Entrar a `https://crit-assist-demo.onrender.com/checkin.html` o al modulo de check-in disponible.
2. Iniciar sesion con `demo.recepcion.general@crit.test`.
3. Buscar o escanear al paciente `Santiago Ramirez Torres` con codigo `DEMO-PAT-SUR-001`.
4. Confirmar que aparecen sus citas del dia si aplica.
5. Registrar check-in.
6. Repetir con `Mateo Lopez Garcia` y codigo `DEMO-PAT-NORTE-001` si se quiere validar alcance global.
7. Objetivo de validacion: recepcion puede registrar llegada sin ver contenido de notas medicas.

### 5. Recepcion por clinica valida alcance limitado

Responsable: recepcion Norte o Sur.

1. Entrar con `demo.recepcion.norte@crit.test`.
2. Buscar `Mateo Lopez Garcia` (`DEMO-PAT-NORTE-001`). Debe ser un caso natural para Norte.
3. Buscar un paciente de Sur, por ejemplo `Santiago Ramirez Torres` (`DEMO-PAT-SUR-001`). Confirmar si el sistema limita el acceso segun la clinica.
4. Cerrar sesion.
5. Entrar con `demo.recepcion.sur@crit.test` y repetir al reves.
6. Objetivo de validacion: recepcion solo opera dentro de su alcance.

### 6. Medico registra asistencia y nota medica

Responsable: medico Norte.

1. Entrar a `https://crit-assist-demo.onrender.com` con `demo.medico.norte@crit.test`.
2. Abrir agenda/calendario.
3. Buscar la cita de `Mateo Lopez Garcia`.
4. Registrar asistencia como presente si el flujo lo permite.
5. Crear o revisar nota medica de la cita.
6. Cerrar sesion.
7. Entrar con recepcion y confirmar que recepcion no puede leer el contenido clinico.
8. Objetivo de validacion: el personal clinico captura informacion sensible y recepcion no la ve.

### 7. Terapeuta Sur atiende una cita y revisa notificaciones

Responsable: terapeuta Sur.

1. Entrar con `demo.terapeuta.sur@crit.test`.
2. Abrir agenda o asistencias.
3. Buscar a `Santiago Ramirez Torres` o `Camila Martinez Flores`.
4. Revisar si hay notificaciones pendientes.
5. Registrar o revisar asistencia segun el estado de la cita.
6. Objetivo de validacion: terapeuta ve su agenda y sus pendientes, no todo el centro.

### 8. Coordinador revisa agenda y reprogramaciones

Responsable: coordinacion Norte o Sur.

1. Entrar con `demo.coordinador.norte@crit.test`.
2. Revisar calendario y filtros por clinica.
3. Confirmar que puede ver la operacion de su area.
4. Entrar con `demo.coordinador.sur@crit.test`.
5. Revisar el caso de `Camila Martinez Flores`, que representa solicitud de reagendar.
6. Objetivo de validacion: coordinacion entiende carga, citas y cambios sin hacer trabajo clinico.

### 9. Personal de acompanamiento crea o revisa nota de enlace

Responsable: personal de acompanamiento.

1. Entrar con `demo.acompanamiento.norte@crit.test`.
2. Buscar un paciente con seguimiento, por ejemplo `Mateo Lopez Garcia`.
3. Crear o revisar una nota de enlace si el flujo esta disponible.
4. Confirmar que destinatarios reciben notificacion.
5. Objetivo de validacion: acompanamiento puede dejar contexto operativo sin crear nota medica.

### 10. Cierre de la prueba

Al terminar, cada persona responde:

1. Que accion intentaste hacer.
2. Si encontraste al paciente/cita esperado.
3. Si viste informacion que no deberias ver.
4. Si el flujo fue claro o hubo friccion.
5. Que dato faltaria para usarlo en una prueba mas real.

## Pacientes demo y codigos de barras

Los codigos son ficticios y corresponden a `external_id` de pacientes demo.

| Paciente | Codigo | Uso sugerido | Codigo de barras |
| --- | --- | --- | --- |
| Mateo Lopez Garcia | `DEMO-PAT-NORTE-001` | Asistencia presente, nota medica y check-in Norte | ![DEMO-PAT-NORTE-001](assets/barcodes/DEMO-PAT-NORTE-001.svg) |
| Valentina Hernandez Ruiz | `DEMO-PAT-NORTE-002` | Inasistencia y cita futura Norte | ![DEMO-PAT-NORTE-002](assets/barcodes/DEMO-PAT-NORTE-002.svg) |
| Santiago Ramirez Torres | `DEMO-PAT-SUR-001` | Check-in y terapia fisica Sur | ![DEMO-PAT-SUR-001](assets/barcodes/DEMO-PAT-SUR-001.svg) |
| Camila Martinez Flores | `DEMO-PAT-SUR-002` | Reagendar y notificaciones | ![DEMO-PAT-SUR-002](assets/barcodes/DEMO-PAT-SUR-002.svg) |
| Emiliano Sanchez Perez | `DEMO-PAT-INF-001` | Lenguaje infantil | ![DEMO-PAT-INF-001](assets/barcodes/DEMO-PAT-INF-001.svg) |
| Renata Gutierrez Morales | `DEMO-PAT-INF-002` | Cita futura infantil | ![DEMO-PAT-INF-002](assets/barcodes/DEMO-PAT-INF-002.svg) |
| Diego Torres Navarro | `DEMO-PAT-CAN-001` | Cita cancelada | ![DEMO-PAT-CAN-001](assets/barcodes/DEMO-PAT-CAN-001.svg) |
| Lucia Vargas Mendoza | `DEMO-PAT-SIN-001` | Paciente valido sin citas | ![DEMO-PAT-SIN-001](assets/barcodes/DEMO-PAT-SIN-001.svg) |

## Codigos en texto para copiar

```txt
DEMO-PAT-NORTE-001
DEMO-PAT-NORTE-002
DEMO-PAT-SUR-001
DEMO-PAT-SUR-002
DEMO-PAT-INF-001
DEMO-PAT-INF-002
DEMO-PAT-CAN-001
DEMO-PAT-SIN-001
```

## Notas para el facilitador

- Antes de iniciar, confirma que `npm run demo:seed-smoke` ya corrio contra Render.
- Comparte passwords reales por mensaje privado o verbalmente; no las pongas en el repositorio.
- Si un usuario no puede entrar, valida que este usando la password correcta: casi todos usan `DEMO_USER_PASSWORD`.
- Si no aparecen citas, confirma que el demo fue sembrado con `DEMO_YEAR=2026`.
- Si un lector de codigo no reconoce las imagenes, copia manualmente el codigo del paciente en el buscador del check-in.
