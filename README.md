# Quiubo

Chat para conocer gente nueva. Las personas se registran con sus datos, ven un directorio de perfiles, conversan en una sala general y se escriben en chats uno a uno. Todo queda guardado en una base de datos SQLite.

- **Sin dependencias externas:** solo usa Node.js (módulos integrados, incluido `node:sqlite`). No hay `npm install`.
- **Requisito:** Node.js 22.13 o superior.
- **Se instala en el celular:** abre la página y usa "Añadir a pantalla de inicio" (es una PWA).

## Probarlo en tu computador

```bash
node src/server.js        # o: npm start
# abre http://localhost:3000
```

Pruebas automáticas (registro, sesiones, chats, bloqueos, reportes, borrado de cuenta):

```bash
npm test
```

## Departamentos y ciudades

Al registrarse la persona elige su departamento y luego su ciudad. La lista está en `public/colombia.js` e incluye los 32 departamentos y Bogotá, D.C., con sus ciudades y municipios principales (no los más de 1.100 del país). Si falta uno, la persona usa "Otro municipio" y lo escribe. Para añadir municipios, edita ese archivo.

## Qué guarda la base de datos

El archivo se crea solo en `data/quiubo.db` (o donde indique `DB_PATH`).

| Tabla | Contenido |
|---|---|
| `usuarios` | correo, contraseña con hash (scrypt), nombre, edad, departamento y ciudad (se eligen de una lista de Colombia), qué busca, intereses, bio, fecha de aceptación de normas |
| `sesiones` | sesiones activas (se guarda solo el hash del token) |
| `mensajes_sala` | mensajes de la sala general |
| `mensajes` | chats uno a uno, con marca de leído |
| `bloqueos` | quién bloqueó a quién |
| `reportes` | reportes enviados desde los chats |

Al eliminar una cuenta se borran en cascada su perfil, sus mensajes, bloqueos y reportes.

Para ver los datos: `sqlite3 data/quiubo.db "SELECT id, nombre, edad, ciudad FROM usuarios;"`

## Variables de entorno

Mira `.env.example`. Las principales son `PORT`, `DB_PATH`, `ADMIN_TOKEN` y `TRUST_PROXY`.

Para revisar reportes define `ADMIN_TOKEN` y consulta:

```bash
curl -H "x-admin-token: TU_CLAVE" https://tu-sitio/api/admin/reportes
```

## Subirlo a GitHub

1. Crea un repositorio vacío en https://github.com/new (por ejemplo `quiubo`, sin README).
2. En esta carpeta:

```bash
git init
git add .
git commit -m "Quiubo: chat para conocer gente"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/quiubo.git
git push -u origin main
```

`.gitignore` ya excluye `data/` y `.env`, así que la base de datos y los secretos no se suben.

## Ponerlo en internet

GitHub guarda el código, pero no ejecuta servidores. Para que la gente lo use necesitas un hosting con **disco persistente**, porque SQLite escribe en un archivo.

Con el `Dockerfile` incluido sirve cualquier plataforma que ejecute contenedores (Railway, Fly.io, Render con disco de pago, un VPS):

1. Conecta el repositorio de GitHub a la plataforma.
2. Monta un volumen en `/data`.
3. Define `ADMIN_TOKEN` (y deja `NODE_ENV=production`, ya viene en el Dockerfile).
4. Despliega. La plataforma te da una dirección HTTPS para compartir.

Sin disco persistente (por ejemplo, un plan gratuito que reinicia el contenedor) los registros se pierden en cada reinicio. Haz copias de seguridad del archivo `quiubo.db`.

## Seguridad y límites actuales

Incluido: contraseñas con scrypt, sesiones con cookie `HttpOnly` y `SameSite=Lax`, verificación de origen en las escrituras, límite de intentos de acceso y de mensajes, cabecera CSP, validación de edad (18+) en el servidor y chats que solo pueden leer sus dos participantes.

Pendiente antes de abrirlo a mucha gente:

- Los mensajes **no están cifrados de extremo a extremo**: quien administre el servidor puede leerlos.
- No hay verificación del correo, recuperación de contraseña ni moderación automática. La edad es una declaración de la persona, no se comprueba.
- Las conversaciones se actualizan consultando al servidor cada 3 segundos. Para miles de usuarios conviene pasar a WebSockets o SSE y a PostgreSQL.
- Un solo servidor: SQLite no está pensado para varias instancias a la vez.

## Estructura

```
src/server.js     servidor HTTP, API y archivos estáticos
src/db.js         esquema de la base de datos
public/           interfaz (index.html, styles.css, app.js, manifest, icono)
test/api.test.js  pruebas de la API
Dockerfile        imagen para desplegar
```
