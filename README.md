# Enzō Spa · Nicole Villalobos

Página web y app instalable (PWA) de **Enzō Spa**, centro de relajación en Desamparados, San José.

🌐 **https://enzo-spa.github.io**

## Qué tiene

**Para los clientes**
- Portada con logo, estado "Abierto ahora / Cerrado" según el horario de Costa Rica.
- 12 servicios con precio y detalle (qué incluye, duración recomendada).
- **Agendar cita en 6 pasos**: servicio → lugar (Spa o Estudio) → duración (30 min a 2 h) → día y hora → datos → resumen y confirmación.
  - Las horas ocupadas se ven como **"Reservado"** (sin datos de otros clientes).
  - Opción de reservar o consultar por WhatsApp.
- Galería de fotos y videos (YouTube se ve en la página; Instagram, TikTok y Facebook abren en su app).
- Buzón de **sugerencias** (con o sin nombre).
- Horario, ubicación de Spa y Estudio con **Waze** y Google Maps, contacto.
- Se instala como app (Android e iPhone), con atajo "Agendar cita".

**Para Nicole** (Menú → 🔒 Acceso de Nicole)
- **Citas**: lista por día con lugar, servicio, nota; botones de WhatsApp, llamar y cancelar.
- Pasar citas al calendario del celular (solo las nuevas, sin duplicar).
- **Ingresos**: montos estimados por semana y mes, por servicio y por lugar.
- **Anotaciones** privadas en cada cita.
- **Encuesta de satisfacción**: desde cada cita se envía por WhatsApp un link (`#encuesta=ID`); los resultados se ven en **Opiniones** junto con las sugerencias.
- **Configuración**: notificaciones, avisos de ntfy, calendario, app y datos del negocio.
- **Galería**: subir fotos desde el celular (se comprimen solas), agregar videos por link, ordenar y quitar.
- Notificaciones del navegador para citas y sugerencias nuevas (con la app abierta o en segundo plano).

**Avisos con la app cerrada**: cada reserva y sugerencia se envía a la app **ntfy** (tema privado configurado en `index.html`, constante `NTFY_TEMA`).

## Cómo está hecho

| Parte | Tecnología |
|---|---|
| Página | Un solo `index.html` (HTML, CSS y JavaScript sin frameworks) |
| Hosting | GitHub Pages (gratis) |
| Datos | Firebase Firestore, plan Spark (gratis) |
| Inicio de sesión de Nicole | Firebase Authentication (correo y contraseña) |
| App instalable | `manifest.webmanifest` + `sw.js` (service worker) |
| Avisos | ntfy.sh + notificaciones del navegador |

### Archivos
- `index.html` — toda la página. Los datos fáciles de cambiar están al inicio del script: `SERVICIOS`, `HORARIO`, `LUGARES`, `DURACIONES`, `WHATSAPP`.
- `firestore.rules` — reglas de seguridad de la base de datos.
- `firebase.json` — indica a Firebase dónde están las reglas.
- `.github/workflows/firebase-reglas.yml` — publica las reglas en Firebase automáticamente cuando cambia `firestore.rules`.
- `manifest.webmanifest`, `sw.js`, `icon-*.png`, `captura-*.png` — app instalable.

### Colecciones en Firestore
| Colección | Quién lee | Quién escribe |
|---|---|---|
| `citas` | Solo Nicole | Cualquiera puede crear una reserva válida; Nicole cancela |
| `ocupados` | Todos (solo fecha y hora, sin datos personales) | Se crea junto con la cita; no se puede ocupar dos veces |
| `sugerencias` | Solo Nicole | Cualquiera envía; Nicole borra |
| `galeria` | Todos | Solo Nicole |
| `encuestas` | Solo Nicole | El cliente con el link de su cita (una sola respuesta por cita) |

## Configuración (una sola vez)

1. **Firebase** (proyecto `enzo-spa`): Firestore en modo producción y Authentication con correo/contraseña. Usuario de Nicole: `nicole@enzospa.app`.
2. **Secreto en GitHub**: `Settings → Secrets and variables → Actions → FIREBASE_SERVICE_ACCOUNT` con la llave JSON de la cuenta de servicio de Firebase. La cuenta de servicio necesita los roles *Administrador de reglas de Firebase* y *Consumidor de Service Usage*.
3. **Pages**: `Settings → Pages → Branch: main / (root)`.

> ⚠️ Nunca subir la llave JSON al repositorio ni compartirla. Solo va en *Secrets*.

## Próximos pasos

- [Roadmap de migración a React Native (Atomic Design + SOLID)](docs/roadmap-react-native.md)

## Hacer cambios

- Editar `index.html` y subirlo (o pedírselo a Claude, que tiene acceso a este repositorio). GitHub Pages publica en 1 a 5 minutos.
- Al cambiar archivos de la app, subir el número de `CACHE` en `sw.js` para que los celulares descarguen la versión nueva.
- Cambios en `firestore.rules` se publican solos en Firebase (pestaña **Actions → Reglas de Firebase**).

---
Hecho por **RBM Solutions** · Heredia, Costa Rica
