# DRAGA · Tareas del edificio

App web mobile-first para administrar las tareas de mantenimiento del edificio. Single-page application en HTML + JS vanilla, lista para desplegar en **Firebase Hosting** con **Firestore**, **Storage** y **Auth**.

---

# 🚀 GUÍA DE INSTALACIÓN — seguí los pasos EN ORDEN

> Hacé los pasos **en este orden exacto**. Saltear o cambiar el orden es la causa #1 de errores en el deploy (por ejemplo, el error `Could not find rules for the following storage targets: rules` aparece si intentás desplegar Storage antes de crear el bucket en el paso 5).

## Paso 1 — Crear el proyecto en Firebase

1. Entrá a [console.firebase.google.com](https://console.firebase.google.com).
2. **Agregar proyecto** → ponele un nombre (ej: `draga-inn`) → seguí el asistente (Google Analytics es opcional, podés desactivarlo).

> Si ya creaste el proyecto, pasá al paso 2.

## Paso 2 — Registrar la app web (de acá sale el `firebaseConfig`)

1. Dentro del proyecto, clic en el ⚙️ (engranaje, arriba a la izquierda) → **Configuración del proyecto**.
2. Bajá hasta la sección **Tus apps**.
3. Clic en el icono **`</>`** (Web).
4. Apodo de la app: `draga-web`. **NO** marques la casilla "Firebase Hosting" (eso lo hacemos aparte). Clic en **Registrar app**.
5. Firebase te muestra un bloque de código así (con TUS valores reales):

   ```js
   const firebaseConfig = {
     apiKey: "AIzaSyD-EJEMPLO_xxxxxxxxxxxxxxxxxxxx",
     authDomain: "draga-inn.firebaseapp.com",
     projectId: "draga-inn",
     storageBucket: "draga-inn.appspot.com",
     messagingSenderId: "123456789012",
     appId: "1:123456789012:web:a1b2c3d4e5f6g7h8"
   };
   ```

6. **Copiá ese bloque entero** y dejalo a mano (lo vas a pegar en el paso 6). Si después lo necesitás de nuevo, está siempre en ⚙️ Configuración del proyecto → Tus apps → tu app web → **Config**.

> ℹ️ El `apiKey` **no es secreto**: es una clave pública de cliente, está bien que sea visible en el HTML. La seguridad real la dan las reglas de Firestore y Storage (incluidas en este proyecto).

## Paso 3 — Habilitar Authentication

1. Menú izquierdo → **Authentication** → **Comenzar**.
2. Pestaña **Sign-in method** → habilitá **Correo electrónico/contraseña** → Guardar.

## Paso 4 — Habilitar Firestore Database

1. Menú izquierdo → **Firestore Database** → **Crear base de datos**.
2. Elegí modo **producción**.
3. Región: `southamerica-east1` (São Paulo, más cerca de Uruguay) o `us-central1`. → Habilitar.

## Paso 5 — Habilitar Storage (para las fotos de las tareas)

1. Menú izquierdo → **Storage** → **Comenzar**.
2. Te va a pedir **actualizar al plan Blaze** (pay-as-you-go). Esto es obligatorio para usar Storage desde octubre 2024. **Seguí leyendo la sección de abajo antes de asustarte por los costos** — en la práctica esta app no genera cobros.
3. Una vez en Blaze, completá el asistente de Storage:
   - Modo **producción**.
   - **Ubicación del bucket: elegí `us-central1`** (importante: esta región tiene tier "Always Free" de Google Cloud Storage). → Listo.

### 💳 Cómo NO pagar nada con el plan Blaze (leé esto)

Blaze es "pago por uso", pero incluye un tier gratuito generoso. Para una app de tareas de un edificio (pocas fotos por día) es prácticamente imposible salir del tier gratuito. Aun así, **configurá una alerta de presupuesto** para quedarte tranquilo:

1. Andá a [console.cloud.google.com/billing](https://console.cloud.google.com/billing) → elegí tu proyecto.
2. Menú → **Presupuestos y alertas** (*Budgets & alerts*) → **Crear presupuesto**.
3. Monto: **USD 1**. Activá alertas al 50%, 90% y 100%.
4. Guardar. Si alguna vez el consumo se acercara a 1 dólar, te llega un mail. (No va a pasar con esta app, pero es tu red de seguridad.)

Free tier de Storage en `us-central1`: 5 GB almacenados, 1 GB de descarga/día, 50.000 lecturas/día y 20.000 escrituras/día. Más que suficiente.

## Paso 6 — Configurar el `index.html`

1. Abrí el archivo `index.html` con cualquier editor de texto (Bloc de notas, VS Code, Sublime, etc.).
2. Cerca del comienzo del `<script type="module">` vas a ver este bloque:

   ```js
   const firebaseConfig = {
     apiKey: "REEMPLAZAR_API_KEY",
     authDomain: "REEMPLAZAR.firebaseapp.com",
     ...
   };
   ```

3. Reemplazá ese bloque **completo** por el que copiaste en el paso 2.
4. (Opcional) Unas líneas más abajo, cambiá `const BUILDING_NAME = "DRAGA";` si querés otro nombre de edificio.
5. Guardá el archivo.

## Paso 7 — Instalar Firebase CLI y desplegar

Abrí una terminal **ubicada en la carpeta donde está `index.html`** (junto con `firebase.json`, `firestore.rules`, etc.) y ejecutá:

```bash
# 1. Instalar la herramienta de Firebase (una sola vez en tu computadora)
npm install -g firebase-tools

# 2. Iniciar sesión con tu cuenta de Google (se abre el navegador)
firebase login

# 3. Vincular esta carpeta con tu proyecto Firebase
firebase use --add
#    → elegí "draga-inn" de la lista y ponele un alias (ej: "default")

# 4. Desplegar todo
firebase deploy --only hosting,firestore:rules,storage:rules
```

Al terminar te muestra una URL tipo **`https://draga-inn.web.app`** — esa es tu app online.

> Si te falta `npm`, instalá [Node.js](https://nodejs.org) primero (viene con npm incluido).

## Paso 8 — Crear el primer usuario ADMIN (a mano, una sola vez)

La app no tiene registro libre por seguridad. El primer administrador se crea manualmente; después, desde la app, ese admin crea a todos los demás.

1. Consola Firebase → **Authentication** → pestaña **Users** → **Add user**.
2. Poné un email y una contraseña → Add user.
3. Copiá el **UID** del usuario recién creado (columna User UID, hay un botón para copiarlo).
4. Andá a **Firestore Database** → **Iniciar colección**.
5. ID de la colección: escribí exactamente `users` → Siguiente.
6. **ID del documento**: pegá el **UID** que copiaste en el paso 3 (NO pongas un ID automático).
7. Agregá estos 4 campos (todos de tipo **string**):

   | Campo | Valor |
   |---|---|
   | `name` | tu nombre (ej: `Juan Pérez`) |
   | `email` | el mismo correo que pusiste en Authentication |
   | `role` | `admin` |
   | `apartment` | dejalo vacío o poné tu apto |

8. **Guardar**.

## Paso 9 — ¡Listo! Empezar a usar la app

1. Entrá a tu URL (`https://draga-inn.web.app`).
2. Logueate con el email y contraseña del admin.
3. Andá a la pestaña **Usuarios** y creá los propietarios, el encargado y administración con sus roles.
4. Avisales a cada uno su email y contraseña inicial (que la cambien después).

## Paso 10 (opcional) — Activar las notificaciones por email

Sin esto la app funciona igual, pero no manda correos. Para activarlos:

1. Consola → **Extensions** (Extensiones) → buscar **"Trigger Email from Firestore"** → Install.
2. Configurá:
   - **Email documents collection**: `mail`
   - **SMTP connection URI**: la URL de tu proveedor de correo. Ejemplos:
     - Brevo (gratis): `smtps://USUARIO:CLAVE_SMTP@smtp-relay.brevo.com:465`
     - Gmail (con contraseña de aplicación): `smtps://tucorreo@gmail.com:APP_PASSWORD@smtp.gmail.com:465`
   - **Default FROM address**: `tareas@tudominio.com` o tu Gmail.
3. El resto se deja por defecto. Instalar.

---

# 📱 La app — cómo funciona

## 4 tipos de usuarios

- **Propietario** — crea tareas (con fotos), comenta, ve la evolución de las tareas.
- **Encargado** — ve todas las tareas. Puede marcarlas **realizadas**, **rechazarlas** o **enviarlas a administración**. Puede comentar.
- **Administración** — ve las tareas que llegaron a administración (y las que ya pasaron por ahí). Puede marcarlas **realizadas**, **rechazarlas** o **devolverlas al encargado**. Puede comentar.
- **Admin (sistema)** — gestiona usuarios y roles. Acceso total.

## Estados de una tarea (4)

| Estado | Significado | Color |
|---|---|---|
| `pendiente` | Recién creada, esperando al encargado. | ámbar |
| `realizada` | Completada (por encargado o administración). | verde |
| `rechazada` | Rechazada (por encargado o administración). | rojo |
| `en_administracion` | Pasada a administración por el encargado. | azul |

## Flujo de estados

```
                  ┌──────────────┐
   Propietario ──▶│  PENDIENTE   │◀─────────────────────────┐
   crea           └──────┬───────┘                          │
                         │                                  │
                         │ Encargado decide:                │
            ┌────────────┼────────────────┐                 │
            ▼            ▼                ▼                 │
      ┌──────────┐  ┌──────────┐   ┌──────────────────┐     │
      │REALIZADA │  │RECHAZADA │   │EN_ADMINISTRACION │     │
      └────▲─────┘  └────▲─────┘   └────────┬─────────┘     │
           │             │                  │               │
           │ (Admin.)    │ (Admin.)         │ Administración│
           │             │                  │ decide:       │
           └─────────────┴──────────────────┼───────────────┤
                                            │               │
                            ┌───────────────┼───────────────┘
                            ▼               ▼
                      REALIZADA        RECHAZADA
```

| Desde | Hasta | Quién puede |
|---|---|---|
| `pendiente` | `realizada` | Encargado |
| `pendiente` | `rechazada` | Encargado |
| `pendiente` | `en_administracion` | Encargado |
| `en_administracion` | `realizada` | Administración |
| `en_administracion` | `rechazada` | Administración |
| `en_administracion` | `pendiente` (devuelta) | Administración |
| `realizada` / `rechazada` | `pendiente` (reabrir) | Encargado |
| `realizada` / `rechazada` | `en_administracion` (reabrir) | Administración |
| Cualquiera | Cualquier estado | Admin (sistema) |

## Notificaciones por email

| Acción | Quién dispara | Quién recibe |
|---|---|---|
| Crea tarea | Propietario | Encargado |
| Comenta | Propietario | Encargado |
| Comenta | Encargado | Propietario que creó la tarea |
| Comenta | Administración | Propietario + Encargado |
| Estado → `en_administracion` | Encargado | Administración + Propietario |
| Estado → `realizada` o `rechazada` | Encargado | Propietario |
| Estado → `realizada` o `rechazada` | Administración | Propietario + Encargado |
| Estado → `pendiente` (devuelta) | Administración | Encargado + Propietario |

## Visibilidad de tareas por rol

- **Propietario**: ve todas las tareas del edificio. Puede crear nuevas.
- **Encargado**: ve todas. Acciona sobre las que están en `pendiente`.
- **Administración**: ve solo las que están o estuvieron alguna vez en administración.
- **Admin (sistema)**: ve todo, transiciona a cualquier estado, gestiona usuarios.

## Reportes

Disponibles para todos los roles: agrupación de tareas por estado en un rango de fechas, cantidad total y tiempo promedio de resolución.

---

# 📂 Estructura de datos (Firestore)

```
/users/{uid}
  ├─ name, email, role, apartment, createdAt

/tasks/{taskId}
  ├─ title, description
  ├─ status: pendiente | realizada | rechazada | en_administracion
  ├─ photos: array<string>   (URLs de Storage)
  ├─ createdBy, createdByName, createdByEmail, createdByApt, createdByRole
  ├─ commentsCount
  ├─ history: array<{ from, to, at, byUid, byName, byRole }>
  ├─ createdAt, updatedAt
  └─ /comments/{commentId}
       ├─ text, byUid, byName, byEmail, byRole, createdAt

/mail/{mailId}              (lo procesa la extensión Trigger Email)
  ├─ to: array<string>
  └─ message: { subject, html }
```

---

# 🛠 Solución de problemas

**`Could not find rules for the following storage targets: rules`**
No creaste el bucket de Storage todavía. Hacé el **Paso 5** (Storage → Comenzar). Mientras tanto podés desplegar sin Storage: `firebase deploy --only hosting,firestore:rules`.

**`Error 402` o `403` al subir fotos**
El proyecto no está en plan Blaze. Volvé al Paso 5.

**No me llegan los correos**
Falta la extensión Trigger Email (Paso 10), o el SMTP está mal configurado. Revisá los logs de la extensión en la consola.

**"Tu usuario no tiene perfil"**
El documento en `/users/{uid}` no existe o el ID del documento no coincide con el UID de Authentication. Revisá el Paso 8 (el ID del doc tiene que ser exactamente el UID).

**Pantalla en blanco / no carga**
Abrí la consola del navegador (tecla F12 → pestaña Console). Los errores de Firebase aparecen ahí con mensaje claro. Lo más común: `firebaseConfig` mal pegado (Paso 6).

**Probar local sin desplegar**
`firebase serve --only hosting` — o cualquier servidor estático. NO abras `index.html` con doble clic (`file://`): los módulos ES no funcionan así.

---

# 🔮 Próximos pasos sugeridos

- Reset de contraseña por mail ("olvidé mi contraseña").
- Push notifications (Firebase Cloud Messaging).
- Búsqueda y filtros más finos.
- Editar/borrar comentarios propios.
- Modo oscuro.
- Exportar reportes a CSV/PDF.
