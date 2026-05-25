# ADR 003: Next.js + PWA sobre React Native para Interfaz de Mecánicos

## Estado

`Aprobado` — React Native Expo evaluado para Fase 3 (ver `arellan-sandbox-labs/EXP-005`)

## Contexto

Los mecánicos del taller necesitan una interfaz para:
- Registrar el ingreso de vehículos (5 fotos obligatorias + firma digital del cliente)
- Ver sus órdenes de trabajo asignadas
- Solicitar repuestos del inventario
- Registrar avance de trabajos
- Operar **offline** cuando el WiFi del taller falla

**Hardware objetivo:** Tablets Android (8-10") que el taller ya tiene o puede adquirir a bajo costo. No se distribuyen iPhones ni dispositivos iOS a mecánicos.

**Restricción de equipo:** El equipo frontend tiene experiencia en React y Next.js. Sin experiencia nativa (Swift, Kotlin) ni en React Native en producción.

**Restricción operacional:** Actualizar una app móvil publicada en Play Store requiere revisión de Google (1-3 días). El sistema necesita poder actualizarse sin esa fricción.

## Alternativas Consideradas

### Opción 1: React Native (Bare Workflow)

App nativa compilada distribuida vía Play Store.

**Pros:**
- Acceso a APIs nativas (cámara, biometría, GPS)
- Mejor performance en animaciones complejas

**Contras:**
- Play Store review delay para cada deploy (crítico en bug fixes de producción)
- Equipo sin experiencia → curva de aprendizaje de 2-3 meses
- Sin soporte iOS útil (mecánicos no usan iPhones)
- Build process más complejo (Gradle, signing keys, APK distribution)
- Sin beneficio real sobre PWA para este caso de uso (formularios + fotos, sin 3D ni gaming)

### Opción 2: React Native con Expo (Managed Workflow)

**Pros:**
- Expo Go simplifica desarrollo inicial
- OTA updates con Expo Updates (bypassa Play Store para JS changes)
- Acceso a cámara/biometría con Expo APIs

**Contras:**
- Expo OTA updates requieren plan pago para producción
- Aun necesita Play Store para instalar (o APK sideloading — riesgo de seguridad)
- Evaluado en EXP-005: "MVP sin urgencia, PWA suficiente"
- Tamaño de bundle mayor que PWA

### Opción 3: React 18 + Vite PWA (con Workbox) ← Seleccionada

Progressive Web App instalable en tablets Android vía Chrome "Agregar a pantalla de inicio".

**Pros:**
- Deploy instantáneo: actualización en el servidor = actualización en todas las tablets sin acción del usuario
- Offline-first con Workbox + IndexedDB (capacidad: 8 horas de trabajo, validado en EXP-004)
- Instalable como app nativa en Android (icono en pantalla de inicio, fullscreen, sin barra del browser)
- El equipo domina React + Vite → sin curva de aprendizaje
- Cámara accesible vía `getUserMedia()` en Chrome Android (Chrome 85+)
- Sin dependencia de Play Store ni proceso de distribución
- Build + deploy en minutos vs horas (APK compilation)
- Funciona en cualquier tablet Android con Chrome

**Contras:**
- Sin acceso a Bluetooth nativo (no necesario en MVP)
- Notificaciones push menos confiables que nativas en iOS (no relevante: tablets Android)
- `getUserMedia()` requiere HTTPS (resuelto: SSL en todos los environments)

## Decisión

Seleccionamos **React 18 + Vite + PWA** para `arellan-mechanic-ui`.

### Offline Architecture

```
┌─────────────────────────────────────────┐
│           arellan-mechanic-ui           │
│         (React 18 + Vite + PWA)         │
│                                         │
│  ┌──────────────┐  ┌──────────────┐    │
│  │ IndexedDB    │  │ Workbox SW   │    │
│  │ Action Queue │  │ Cache First  │    │
│  │ (8h offline) │  │ for assets   │    │
│  └──────┬───────┘  └──────────────┘    │
│         │ sync when online              │
└─────────┼───────────────────────────────┘
          │ HTTP + WebSocket
          ▼
   arellan-backend-core
```

### Offline Queue Strategy

```typescript
// Acciones que se encolan en IndexedDB cuando offline:
type OfflineAction =
  | { type: 'UPDATE_ORDER_STATUS'; payload: {...} }
  | { type: 'REQUEST_INVENTORY_ITEM'; payload: {...} }
  | { type: 'COMPLETE_WORK_STEP'; payload: {...} }

// Al recuperar conexión: sync automático en orden FIFO
// Capacidad: ~50 acciones en 8 horas de trabajo normal (EXP-004)
```

### UI Touch Rules

Mecánicos trabajan con guantes y tablets. Reglas de diseño no negociables:
- Botones: mínimo **56px de altura** (`min-h-14`)
- Texto: mínimo **2xl** (`text-2xl`, 24px)
- Tap targets: mínimo **44x44px** (WCAG 2.1 AA)
- Sin hover states como único feedback (sin mouse en tablet)
- Contraste: todos los elementos pasan WCAG AA (ratio ≥ 4.5:1)

## Consecuencias

### Positivas

- Deploy instantáneo sin Play Store → bug fixes en minutos
- Offline real con IndexedDB (validado 8 horas)
- Sin fricción de distribución → tablets se actualizan al refrescar
- Equipo productivo desde día 1 (React conocido)

### Negativas / Trade-offs

- Sin notificaciones push en background (tablets offline no reciben alertas) → aceptable: mecánicos ven la UI cuando trabajan
- Cámara via `getUserMedia()` menos features que cámara nativa → suficiente para 5 fotos de ingreso
- Si en Fase 3 se necesitan features nativas (Bluetooth a scanner de repuestos): migrar a Expo

### Plan de Migración (si se necesita en Fase 3)

EXP-005 evaluó Expo como opción viable. Si los mecánicos necesitan:
- Bluetooth para scanners de código de barras
- Biometría del dispositivo para aprobaciones
- Notificaciones push confiables en background

→ Migrar a `arellan-mechanic-ui-native` con React Native Expo (sin reescritura total: misma lógica de negocio, solo reemplazar componentes UI).

## Notas de Implementación

- Repo: `arellan-mechanic-ui`
- Build: `vite build` con `vite-plugin-pwa` (Workbox)
- Manifest: `public/manifest.json` con `display: standalone`
- Deploy: Vercel (mismo pipeline que `arellan-frontend-web`)
- Testing: Playwright con proyecto `iPad Pro` en `arellan-e2e-tests/playwright.config.ts`
- Sandbox: `arellan-sandbox-labs/EXP-004` (offline performance), `EXP-005` (React Native eval)

---

*Fecha de decisión: 2025-Q1*
*Autor: Diego Soto (Tech Lead)*
*Revisado por: Edgar Arellan (Owner)*
