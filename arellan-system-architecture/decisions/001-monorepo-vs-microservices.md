# ADR 001: Multi-Repo con Dominios Desacoplados vs Monolito vs Microservicios

## Estado

`Aprobado`

## Contexto

Clínica Automotriz Arellan Hnos es un taller familiar (Surquillo, Lima) que opera actualmente con procesos analógicos y enfrenta tres vectores de fraude activos:

1. Empleado usando QR personal de Yape para desviar pagos
2. Comisiones no declaradas en importaciones (20-30%)
3. Uso no autorizado de vehículos del taller

Se necesita un ecosistema digital que:
- Sea desarrollable por un equipo pequeño (1-3 devs)
- Escale desde MVP en Railway hasta producción en AWS
- Pueda eventualmente convertirse en SaaS ("GarageCore OS") para otros talleres
- Tenga aislamiento de fallos entre módulos críticos (hardware IoT vs pagos vs UI)
- Permita que documentación, gobernanza y código coexistan sin acoplamiento

**Restricción clave:** El equipo no tiene experiencia en Kubernetes ni service mesh. La complejidad operacional debe ser mínima en el MVP.

## Alternativas Consideradas

### Opción 1: Monolito en un solo repo

Todo el código (backend, frontends, hardware bridge) en un único repositorio.

**Pros:**
- Setup más simple
- Un solo deploy
- Sin overhead de comunicación entre servicios

**Contras:**
- Fallo en el módulo IoT puede bloquear toda la aplicación
- Una sola base de código para módulos con ciclos de vida muy distintos (hardware vs UI web)
- Imposible empaquetar como SaaS sin llevar toda la complejidad del cliente

### Opción 2: Microservicios en monorepo (Turborepo/Lerna)

Servicios independientes en un único repo con herramientas de workspace.

**Pros:**
- Aislamiento de servicios
- Herramientas compartidas con workspace

**Contras:**
- Requiere orquestación (Docker Compose en dev, Kubernetes en prod)
- Sin experiencia del equipo en microservicios en producción
- Latencia de red entre servicios para cada request
- Complejidad de transacciones distribuidas para operaciones críticas (caja, pagos)

### Opción 3: Multi-Repo con Dominios Desacoplados ← Seleccionada

13 repositorios organizados por dominio de negocio. El backend es **monolítico modular** (un proceso NestJS), los repos de frontend son aplicaciones independientes, y los repos de hardware/IoT corren como procesos separados.

```
arellan-tech/ (GitHub org)
├── arellan-backend-core      ← Monolito modular NestJS (proceso único)
├── arellan-frontend-web      ← Next.js 14 (OWNER/ADMIN/FINANCE)
├── arellan-mechanic-ui       ← React + Vite PWA (tablets)
├── arellan-mobile-app        ← Expo React Native (dueños)
├── arellan-hardware-iot/     ← Procesos hardware separados
│   ├── arellan-iot-hardware-bridge   (ZKTeco ADMS listener)
│   └── arellan-vehicle-tracking      (GPS geofencing)
├── arellan-docs-governance   ← Este repo
└── ... (repos de gobernanza y roadmap)
```

**La decisión crítica:** El backend es **monolítico modular, no microservicios**. Un solo proceso NestJS con módulos bien separados (AuthModule, OrdersModule, FinanceModule, etc.). Los repos de hardware son procesos separados porque tienen dependencias de sistema distintas y ciclos de deploy independientes.

**Pros:**
- Aislamiento de fallos: crash en IoT bridge no afecta pagos ni órdenes de trabajo
- Deploy independiente por dominio según su ciclo de vida
- Sin latencia de red interna en el backend (módulos del monolito se comunican en memoria)
- Transacciones ACID en PostgreSQL sin coordinación distribuida
- Empaquetado SaaS: `arellan-backend-core` puede clonarse como template sin arrastrar IoT ni UI específica del cliente

**Contras:**
- Mayor overhead de setup inicial (13 repos vs 1)
- Sin shared types automáticos → resuelto con `arellan-shared-types` package publicado en GitHub Packages
- Onboarding requiere clonar múltiples repos → resuelto con `arellan-docs-governance/arellan-governance/onboarding.md`

## Decisión

Seleccionamos **Multi-Repo con Dominios Desacoplados** con backend monolítico modular.

```
                ┌──────────────────────────┐
                │   arellan-backend-core   │
                │   (NestJS monolito)      │
                │                          │
                │  AuthModule              │
                │  OrdersModule            │
                │  FinanceModule           │
                │  InventoryModule         │
                │  EmployeesModule         │
                │  VehiclesModule          │
                │  AuditModule             │
                └─────────────┬────────────┘
                              │ PostgreSQL 15
                              ▼
                ┌──────────────────────────┐
                │      Supabase/RDS        │
                └──────────────────────────┘

Procesos separados (repos distintos):
┌──────────────────────┐  ┌──────────────────────┐
│ iot-hardware-bridge  │  │  vehicle-tracking    │
│ (ZKTeco ADMS)        │  │  (GPS + geofence)    │
└──────────────────────┘  └──────────────────────┘
```

## Consecuencias

### Positivas

- **Fault isolation real:** IoT crash no bloquea caja ni OTs
- **Ciclos de deploy independientes:** firmware GPS se actualiza sin redeploy del backend
- **SaaS-ready:** backend core sin dependencias de hardware específico del cliente
- **Transacciones ACID:** sin coordinación distribuida para operaciones críticas de caja

### Negativas / Trade-offs

- Onboarding más complejo → documentado en `onboarding.md`
- Sin live reload cross-repo en desarrollo → aceptable, módulos rara vez cambian juntos
- `arellan-shared-types` debe publicarse en cada cambio de interfaz → proceso documentado en `git-workflow.md`

### Neutras

- Testing: cada repo tiene sus propios tests; E2E en `arellan-e2e-tests` orquesta todo

## Notas de Implementación

- Shared types: `@arellan/shared-types` en GitHub Packages (`npm install @arellan/shared-types`)
- Backend entry: `arellan-backend-core/src/main.ts`
- IoT bridge: proceso Node.js independiente, comunica con backend vía HTTP interno
- Ver `arellan-system-architecture/overview.md` para diagrama completo

---

*Fecha de decisión: 2025-Q1*
*Autor: Diego Soto (Tech Lead)*
*Revisado por: Edgar Arellan (Owner)*
