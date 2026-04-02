# Uptask — Backend

REST API para la aplicación de gestión de proyectos y tareas Uptask. Construida con Node.js, Express y MongoDB.

## Tech Stack

- **Runtime**: Node.js con TypeScript
- **Framework**: Express.js 4.18
- **Base de datos**: MongoDB + Mongoose 8
- **Autenticación**: JWT (jsonwebtoken) + bcrypt
- **Validación**: express-validator
- **Emails**: Nodemailer (compatible con Mailtrap / SMTP)
- **Dev tools**: ts-node, nodemon

## Requisitos

- Node.js >= 18
- MongoDB Atlas (o instancia local)
- Cuenta SMTP para emails (Mailtrap recomendado para desarrollo)

## Instalación

```bash
cd uptask_backend
npm install
```

Crear el archivo `.env` en la raíz del backend:

```env
DATABASE_URL=mongodb+srv://<usuario>:<password>@cluster.mongodb.net/uptask
FRONTEND_URL=http://localhost:5173
SMTP_HOST=sandbox.smtp.mailtrap.io
SMTP_PORT=2525
SMTP_USER=tu_usuario_mailtrap
SMTP_PASS=tu_password_mailtrap
JWT_SECRET=tu_secreto_jwt
```

## Scripts

```bash
npm run dev      # Servidor en modo desarrollo con nodemon
npm run build    # Compila TypeScript a dist/
npm start        # Corre la versión compilada
```

El servidor corre en `http://localhost:4000` por defecto.

## Estructura del proyecto

```
src/
├── config/         # Conexión a DB y configuración de servicios
├── controllers/    # Lógica de negocio (Auth, Project, Task, Team, Note)
├── models/         # Esquemas Mongoose (User, Project, Task, Note, Token)
├── routes/         # Definición de rutas de la API
├── middleware/     # Autenticación JWT, validación, verificación de recursos
├── emails/         # Templates y envío de emails con Nodemailer
├── utils/          # Helpers (JWT, tokens de confirmación)
├── server.ts       # Configuración de Express (CORS, middlewares, rutas)
└── index.ts        # Entry point — conexión a DB y arranque del servidor
```

## API Endpoints

### Autenticación — `/api/auth`

| Método | Ruta | Descripción |
|--------|------|-------------|
| POST | `/create-account` | Registrar nuevo usuario |
| POST | `/confirm-account` | Confirmar cuenta con token de email |
| POST | `/login` | Login — devuelve JWT |
| POST | `/request-code` | Reenviar código de confirmación |
| POST | `/forgot-password` | Solicitar reset de contraseña |
| POST | `/validate-token` | Validar token de reset |
| POST | `/update-password/:token` | Establecer nueva contraseña |
| GET | `/user` | Obtener usuario autenticado |
| PUT | `/profile` | Actualizar perfil |
| POST | `/update-password` | Cambiar contraseña actual |
| POST | `/check-password` | Verificar contraseña actual |

### Proyectos — `/api/projects`

| Método | Ruta | Descripción |
|--------|------|-------------|
| POST | `/` | Crear proyecto |
| GET | `/` | Listar proyectos del usuario |
| GET | `/:id` | Detalle de un proyecto |
| PUT | `/:projectId` | Editar proyecto (solo manager) |
| DELETE | `/:projectId` | Eliminar proyecto (solo manager) |

### Tareas — `/api/projects/:projectId/tasks`

| Método | Ruta | Descripción |
|--------|------|-------------|
| POST | `/` | Crear tarea |
| GET | `/` | Listar tareas del proyecto |
| GET | `/:taskId` | Detalle de una tarea |
| PUT | `/:taskId` | Editar tarea |
| DELETE | `/:taskId` | Eliminar tarea |
| POST | `/:taskId/status` | Actualizar estado de la tarea |

### Equipo — `/api/projects/:projectId/team`

| Método | Ruta | Descripción |
|--------|------|-------------|
| POST | `/find` | Buscar usuario por email |
| GET | `/` | Listar miembros del equipo |
| POST | `/` | Agregar miembro al equipo |
| DELETE | `/:userId` | Remover miembro del equipo |

### Notas — `/api/projects/:projectId/tasks/:taskId/notes`

| Método | Ruta | Descripción |
|--------|------|-------------|
| POST | `/` | Crear nota |
| GET | `/` | Listar notas de una tarea |
| DELETE | `/:noteId` | Eliminar nota |

## Modelos

### User
```
email | password (hash) | name | confirmed (boolean)
```

### Project
```
projectName | clientName | description | manager (ref User) | team (refs User[]) | tasks (refs Task[])
```

### Task
```
name | description | project (ref Project) | status (pending | onHold | inProgress | underReview | completed) | completedBy | notes
```

### Note
```
content | createdBy (ref User) | task (ref Task)
```

### Token
```
token | user (ref User) | expiresAt (TTL 10 minutos)
```

## Autenticación

Las rutas protegidas requieren el header:

```
Authorization: Bearer <jwt_token>
```

El JWT se genera al hacer login y tiene validez de 180 días.

## Flujo de registro

1. Usuario crea cuenta → se genera token de confirmación (10 min)
2. Se envía email con el código de 6 dígitos
3. Usuario confirma → cuenta activa
4. Login → JWT devuelto al cliente
