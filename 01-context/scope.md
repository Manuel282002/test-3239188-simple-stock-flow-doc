# System Scope

> **Why this document exists:** Scope prevents scope creep and aligns expectations.
> It is equally important to define what the system does NOT do as what it does.
> Review this document at the start of each planning cycle.

---

## In Scope

What the system **DOES build and maintain**:

### MVP Features

| # | Feature | Description | Responsible service |
|---|---------|-------------|---------------------|
| 1 | Catálogo de Productos | CRUD y gestión de artículos del catálogo con validación estricta de invariantes de stock no negativo y asignación a cinco categorías fijas de solo lectura. | simple-stock-flow-api |
| 2 | Registro de Ventas | Creación y almacenamiento de transacciones comerciales inmutables con congelación de nombres, precios y nombres de categorías en el instante de la venta. | simple-stock-flow-api |
| 3 | Reporte de Ventas Agregado | Generación de agregaciones y cálculos de reportes por producto sobre un rango de fechas, calculados directamente en el motor por un puerto de lectura. | simple-stock-flow-api |

### Included integrations

| External system | Integration type | Purpose |
|----------------|-----------------|---------|
| Almacenamiento Externo de Imágenes | SDK / API del Proveedor | Persistencia de binarios de imágenes de catálogo, gestionada en la base de datos únicamente a través de una clave opaca independiente (image_key). |

### Environments being built

| Environment | Purpose |
|-------------|---------|
| Local | Development on the developer's machine using localhost docker environments. |
| Development (dev) | Continuous integration and development testing against localized automated test suites. |
| Staging | Pre-production environment verified against PostgreSQL 16.14 engine instance wrapper. |
| Production | Production environment hosting the live core sales data. |

---

## Out of Scope

What the system **does NOT build** in this version and why:

| # | What is out of scope | Reason | Future version? |
|---|---------------------|--------|----------------|
| 1 | Atributos Extendidos de Producto | DP-03 prohíbe explícitamente incluir descripciones, SKUs o códigos de referencia. El producto solo tiene nombre, precio, stock, categoría e imagen. | No |
| 2 | Soporte Multimoneda | D-05 decreta que el sistema es monomoneda por construcción. No existen columnas ni lógica de conversión de divisas en ninguna tabla. | No |
| 3 | Desglose de Reportes por Vendedor | DP-02 prohíbe cruzar datos personales de operadores internos en proyecciones analíticas. El reporte solo agrega por producto. | No |
| 4 | Gestión de Clientes o Compradores | El glosario de dominio define que no existe entidad cliente ni comprador; el sistema solo registra al operador interno. | No |

### What another system / team handles (and why not us)

| Feature | Who builds it | Why not us |
|---------|--------------|-----------|
| Autenticación e Identidad (IAM) | Puerto de Infraestructura | El dominio solo maneja la huella irreversible (password_hash) provista externamente por diseño hexagonal (D-09). |
| Analítica Externa Completa | Sistema BI de Terceros | Fuera del alcance del core de la aplicación; el motor local solo resuelve la agregación del reporte nativo (D-06). |

---

## Scope assumptions

> These assumptions are taken to be true. If they change, the scope must be renegotiated.

| # | Assumption | Consequence if false |
|---|-----------|---------------------|
| 1 | El servidor y el motor de base de datos corren coordinados en UTC. | Los rangos de fecha de los reportes y el campo sold_at sufrirían desajustes horarios. |
| 2 | Las categorías del catálogo son un conjunto fijo y sembrado de cinco filas. | Tendríamos que construir un servicio CRUD y ventanas de mantenimiento para categorías. |
| 3 | Las ventas registradas son transacciones comerciales totalmente inmutables. | Se requeriría lógica compleja de auditoría de modificaciones e históricos de inventario. |

---

## Constraints

| Type | Description |
|------|-------------|
| **Time** | El MVP y el esquema singularizado deben estar operativos en la ventana de planeación de Octubre 2026. |
| **Budget** | Limitado al desarrollo sobre la arquitectura base del repositorio monolítico `simple-stock-flow-api`. |
| **Technology** | Obligatorio el uso del stack corporativo: C# (.NET) + PostgreSQL 16.14 bajo el esquema cualificado `sales`. |
| **Regulatory** | Cumplimiento estricto de la política de privacidad de datos personales (username y sold_by) restringidos en logs. |
| **Team** | Desarrolladores del core técnico asignados para la resolución de las tareas de base de datos de `tasks.md`. |

---

## External dependencies

| Dependency | Team / Provider | Required date | Status |
|-----------|----------------|--------------|--------|
| Motor PostgreSQL 16.14 | DevOps / Infraestructura | 2026-09-19 | 🟢 Available |
| Credenciales del Entorno (D-09, D-10) | Administrador de Sistemas | 2026-10-05 | 🟢 Available |
| Proveedor de Almacenamiento de Imágenes | Equipo de Infraestructura | 2026-10-05 | 🟡 In progress |

---

## How to update the scope

The scope can change, but the change has a process:

1. Document the proposed change in this file
2. Evaluate the impact on schedule and effort
3. Obtain approval from the Product Owner and Tech Lead
4. Update the roadmap in `adr/` or architecture records if core behavior shifts
5. Create or update technical debts or tasks in `tasks.md`

---

## Correlations

- Vision and roadmap → `plan.md`
- Term glossary → `project-glossary.md`
- System overview → `system-overview.md`
- Technical debt backlog → `tasks.md`

