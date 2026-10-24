# Roadmap: migración de Enzō Spa a React Native

**Objetivo:** convertir la PWA actual (`index.html` + Firebase) en una app nativa para Android e iOS, publicada en Play Store y App Store, sin perder datos ni funciones, con una base de código ordenada con **Atomic Design** y principios **SOLID**.

**Estado actual (punto de partida):** PWA en GitHub Pages · Firestore (`citas`, `ocupados`, `sugerencias`, `galeria`, `encuestas`) · Firebase Auth (cuenta de Nicole) · avisos por ntfy y notificaciones del navegador · reglas publicadas desde GitHub Actions.

---

## 1. Decisiones técnicas

| Tema | Elección | Por qué |
|---|---|---|
| Framework | **Expo (SDK estable) + React Native + TypeScript** | Compilación en la nube (EAS), actualizaciones OTA, sin Mac para Android |
| Navegación | **expo-router** (rutas por archivos) | Deep links (`/encuesta/:id`, `/agendar`) iguales a los de la web |
| Backend | **El mismo Firebase** (`enzo-spa`) con `@react-native-firebase/*` | Cero migración de datos; mismas reglas |
| Estado del servidor | **TanStack Query** + listeners de Firestore | Caché, reintentos y estados de carga uniformes |
| Estado local | **Zustand** (paso de la reserva, filtros del calendario) | Simple y tipado |
| Formularios | **react-hook-form + zod** | Validación compartida con las reglas |
| Estilos | **Design tokens** propios + `StyleSheet` (o NativeWind) | Paleta negro / marfil / dorado centralizada |
| Notificaciones | **expo-notifications + FCM/APNs** y **Cloud Functions** | Push real con la app cerrada (reemplaza ntfy). Requiere plan **Blaze** |
| Calendario | **expo-calendar** | Agrega las citas directo al calendario del teléfono, sin archivos `.ics` |
| Imágenes | **expo-image-picker + expo-image-manipulator** | Subir y comprimir fotos de la galería |
| Pruebas | **Jest + React Native Testing Library**, **Maestro** (E2E) | Unitarias, de componentes y de flujo |
| Calidad | ESLint, Prettier, `tsc --noEmit`, Husky | Bloquea errores antes del commit |
| CI/CD | **GitHub Actions + EAS Build/Submit/Update** | Builds y publicación automáticos |

---

## 2. Arquitectura (capas y SOLID)

```
src/
├─ app/                    # Rutas (expo-router): pantallas = "pages" de Atomic Design
├─ ui/                     # Atomic Design (solo presentación, sin Firebase)
│  ├─ tokens/              # colores, tipografía, espacios, radios, sombras
│  ├─ atoms/
│  ├─ molecules/
│  ├─ organisms/
│  └─ templates/
├─ features/               # Un módulo por dominio
│  ├─ booking/             # reservar: hooks, store del paso a paso, validaciones
│  ├─ calendar/            # vistas Día/Semana/Mes de Nicole
│  ├─ income/              # ingresos estimados
│  ├─ feedback/            # encuestas y sugerencias
│  ├─ gallery/
│  └─ settings/
├─ domain/                 # Reglas de negocio puras (sin React ni Firebase)
│  ├─ entities/            # Cita, Servicio, Lugar, Horario, Encuesta…
│  ├─ services/            # Disponibilidad, PrecioEstimado, Ocupación…
│  └─ ports/               # Interfaces: CitasRepo, AuthService, Notifier, CalendarPort…
├─ infrastructure/         # Implementaciones concretas de los ports
│  ├─ firebase/            # FirestoreCitasRepo, FirebaseAuthService…
│  ├─ notifications/       # ExpoPushNotifier
│  └─ calendar/            # ExpoCalendarAdapter
└─ di/                     # Contenedor / Providers que conectan ports con implementaciones
```

### Cómo se aplica SOLID

| Principio | En este proyecto |
|---|---|
| **S** – Responsabilidad única | `Availability.calcular()` solo calcula horas libres; `CitasRepo` solo lee y escribe; los componentes solo muestran. Hoy todo vive en un solo archivo: se separa. |
| **O** – Abierto/cerrado | Nuevos lugares, servicios o duraciones se agregan como datos (configuración remota), sin tocar la lógica. Nuevos canales de aviso (push, WhatsApp, correo) implementan `Notifier` sin cambiar a quien lo usa. |
| **L** – Sustitución de Liskov | `FirestoreCitasRepo` y `InMemoryCitasRepo` (pruebas / modo demo) cumplen el mismo contrato y son intercambiables. |
| **I** – Segregación de interfaces | Interfaces pequeñas: `CitasLectura` (cliente: solo ocupados), `CitasAdmin` (Nicole: leer, cancelar, anotar). El cliente nunca depende de métodos de administración. |
| **D** – Inversión de dependencias | Las features dependen de `domain/ports`, no de Firebase. Se inyectan con Providers (`di/`), lo que permite probar sin red y cambiar de backend en el futuro. |

### Reglas de dependencia
`app → features → domain` · `infrastructure → domain` · `ui` no importa nada de `features`, `domain` ni `infrastructure`.

---

## 3. Atomic Design: inventario de componentes

| Nivel | Componentes (tomados de la PWA actual) |
|---|---|
| **Tokens** | `colors` (negro `#0e0d0c`, marfil `#f6f2eb`, oro `#c9a45c`…), `typography` (Cormorant Garamond, Jost, Great Vibes), `spacing`, `radius`, `shadows` |
| **Átomos** | `Text`, `Heading`, `Button` (oro, línea, oscuro), `Chip`, `Icon` (loto, WhatsApp, pin…), `Input`, `TextArea`, `Star`, `Badge`, `Divider`, `Avatar/Logo` |
| **Moléculas** | `ServiceOption` (nombre + precio), `PlaceOption`, `DurationPicker`, `DayButton`, `TimeSlot` (libre / reservado / tu elección), `StarRating`, `FormField`, `ContactRow`, `PriceTag` (con precio anterior tachado), `StepProgress` |
| **Organismos** | `Hero`, `ServiceGrid`, `ScheduleTable`, `LocationTabs` (mapa + Waze/Maps), `BookingStep*`, `BookingSummary`, `WeekGrid`, `MonthGrid`, `DayTimeline`, `AppointmentModal`, `IncomeSummary`, `IncomeChart`, `SurveyForm`, `GalleryGrid`, `SettingsSection`, `NotificationCard` |
| **Templates** | `PublicLayout` (barra superior + menú), `WizardLayout` (barra de pasos + Atrás/Siguiente), `AdminLayout` (pestañas Citas/Ingresos/Opiniones/Galería + Configuración) |
| **Páginas** | `Inicio`, `Servicio/[id]`, `Agendar`, `Galería`, `Sugerencias`, `Encuesta/[id]`, `Admin/Login`, `Admin/Citas`, `Admin/Ingresos`, `Admin/Opiniones`, `Admin/Galería`, `Admin/Configuración` |

Cada componente con **Storybook** (React Native Storybook) para revisarlo aislado.

---

## 4. Fases

### Fase 0 · Preparación (1 semana)
- [ ] Crear repo `enzo-spa-app` (monorepo opcional con la PWA).
- [ ] Expo + TypeScript + ESLint/Prettier + Husky + Jest.
- [ ] Cuentas: Apple Developer (USD 99/año), Google Play Console (USD 25 una vez), Expo/EAS.
- [ ] Pasar Firebase a **plan Blaze** con alerta de presupuesto (USD 1–5) para push y Functions.
- [ ] Registrar apps Android/iOS en Firebase (`google-services.json`, `GoogleService-Info.plist`).

### Fase 1 · Dominio y design system (1–2 semanas)
- [ ] Extraer de `index.html` a `domain/`: entidades, `Availability` (horas libres, ocupación por bloques de 30 min), `PrecioEstimado`, horario de Costa Rica (UTC-6).
- [ ] Pruebas unitarias del dominio (objetivo ≥ 90 % de cobertura).
- [ ] Tokens + átomos + moléculas + Storybook.
- **Entregable:** catálogo visual aprobado por Nicole.

### Fase 2 · App pública (2 semanas)
- [ ] Inicio, servicios, horario, ubicación (Waze / Google Maps), galería, sugerencias.
- [ ] **Agendar** en 6 pasos con resumen, usando `CitasRepo` (escritura en lote cita + ocupados, igual que hoy).
- [ ] Deep links: `enzospa://agendar`, `https://enzo-spa.github.io/encuesta/:id` → app (App Links / Universal Links).
- **Entregable:** build interno (TestFlight / prueba interna de Play).

### Fase 3 · Panel de Nicole (2–3 semanas)
- [ ] Login con Firebase Auth (y opcional **Face ID / huella** con `expo-local-authentication`).
- [ ] Calendario Día / Semana / Mes / Lista + modal de la cita (WhatsApp, llamar, cancelar, anotaciones).
- [ ] Ingresos, Opiniones (encuestas + sugerencias), Galería (subir y ordenar fotos).
- [ ] **expo-calendar**: "Agregar al calendario" directo, con ID guardado para actualizar en vez de duplicar.
- [ ] Configuración.

### Fase 4 · Notificaciones push reales (1 semana)
- [ ] Guardar el token de push de Nicole en `dispositivos/{id}` (solo ella puede escribir).
- [ ] **Cloud Function** `onCreate(citas)` / `onCreate(encuestas)` / `onCreate(sugerencias)` → envía push con el logo del spa.
- [ ] Recordatorio automático programado (Function con *scheduler*) 24 h antes, opcional por WhatsApp.
- [ ] Retirar ntfy cuando el push esté estable.

### Fase 5 · Calidad y publicación (1–2 semanas)
- [ ] Pruebas E2E con Maestro: reservar, cancelar, responder encuesta.
- [ ] Accesibilidad (lectores de pantalla, tamaños de texto), modo oscuro del sistema.
- [ ] Monitoreo: Sentry o Firebase Crashlytics; Firebase Analytics para embudo de reserva.
- [ ] Fichas de tienda: capturas, descripción, política de privacidad (datos de clientes: nombre y celular).
- [ ] **EAS Submit** a Play Store y App Store.

### Fase 6 · Después del lanzamiento
- [ ] Actualizaciones OTA con **EAS Update** para cambios de textos y precios.
- [ ] Configuración remota (precios, horario, servicios) editable por Nicole desde la app.
- [ ] Bloqueo de horas (almuerzo, vacaciones), paquetes y promociones, reseñas públicas.
- [ ] Mantener la PWA como versión web, compartiendo `domain/` (monorepo con `packages/domain`).

**Duración estimada total:** 8–11 semanas a medio tiempo.

---

## 5. Equivalencias PWA → React Native

| Hoy (PWA) | En la app |
|---|---|
| `localStorage` | `expo-secure-store` (sesión) / MMKV (preferencias) |
| Service worker y caché | Persistencia offline de Firestore + caché de TanStack Query |
| `navigator.share` / archivo `.ics` | `expo-sharing` / **expo-calendar** |
| Notificaciones del navegador + ntfy | **Push nativo** (expo-notifications + Cloud Functions) |
| Lightbox y `iframe` de YouTube | `expo-image` + `react-native-youtube-iframe` |
| `wa.me` y `tel:` | `Linking.openURL` |
| Aviso "Instalar app" | Publicación en tiendas |
| Hash `#encuesta=ID` | Ruta `encuesta/[id]` con deep link |

---

## 6. Riesgos y cómo mitigarlos

| Riesgo | Mitigación |
|---|---|
| Costos de Blaze | Alertas de presupuesto; Functions mínimas; uso actual muy por debajo de la capa gratuita |
| Revisión de App Store | Política de privacidad, cuenta de prueba para Apple, sin contenido de terceros |
| Dos versiones (web y app) desincronizadas | Dominio compartido en `packages/domain` y mismas reglas de Firestore |
| Reglas de seguridad | Pruebas de reglas con el **Firebase Emulator** en CI antes de publicar |
| Dependencia de una sola persona | README, ADRs (decisiones de arquitectura) en `docs/adr/`, Storybook como catálogo |

---

## 7. Definición de "terminado" por pantalla
1. Componentes en el nivel correcto de Atomic Design y en Storybook.
2. Lógica en `domain/` con pruebas; la pantalla solo usa hooks de `features/`.
3. Sin imports de Firebase fuera de `infrastructure/`.
4. Accesible (etiquetas, contraste) y probado en Android e iPhone.
5. Revisado por Nicole en el build de prueba.

---
*Documento vivo · RBM Solutions · última actualización: octubre 2026*
