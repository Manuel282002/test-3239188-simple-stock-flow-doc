# Project Glossary

> **Instructions:** Define here all technical and business terms used in the project.
> This is the official dictionary — if there is ambiguity, this document wins.
> Add terms throughout the project, not only at the start.

---

## How to use this glossary

1. Before using a technical or business term in code, docs, or conversations: look it up here.
2. If it's not there: add it with its definition.
3. If there is disagreement about the definition: discuss it as a team and update this document.

---

## Domain terms

| Term | Definition | Notes / Synonyms |
|------|-----------|-----------------|
| Producto (Product) | Artículo del catálogo que posee nombre, precio, stock, categoría y una clave de imagen opcional. No tiene más campos. | AVOID: SKU, código de referencia o descripción. |
| Categoría (Category) | Clasificación a la que pertenece un producto. Conjunto fijo de cinco filas sembradas de solo lectura. | AVOID: Tablas de mantenimiento o CRUD de categorías. |
| Precio (Price) | Valor monetario vigente de un producto en el catálogo. Es estrictamente positivo (`price > 0`). | Vía `Money` (objeto de valor). Redondeo a 2 decimales. |
| Stock | Unidades disponibles de un producto en el catálogo. Nunca puede ser negativo (`stock >= 0`). | Restricción `ck_product_stock_non_negative` en motor. |
| Imagen (Image Key) | Clave opaca del binario guardado en el almacenamiento externo. Ausente se representa como `NULL`. | AVOID: Guardar rutas de archivos o bytes en la fila. |
| Venta (Sale) | Hecho comercial consumado e inmutable que registra quién realizó la venta, cuándo y qué líneas la componen. | AVOID: Modificar o eliminar una venta una vez registrada. |
| Línea de venta (SaleItem) | Renglón interno de una venta que almacena el producto, la cantidad y el precio congelado del momento. | No existe fuera de su venta (`ON DELETE CASCADE`). |
| Cantidad (Quantity) | Unidades vendidas dentro de una línea de venta específica. Es estrictamente positiva (`quantity > 0`). | Controlado por el objeto de valor de dominio `Quantity`. |
| Usuario (User) | Operador interno autenticado en el sistema que se encarga de registrar ventas. | AVOID: Entidad cliente, comprador o cuenta externa. |
| Rol (Role) | Atribución del nivel de privilegio del usuario dentro de un conjunto cerrado de dos: `admin` o `seller`. | Unicidad y validación bajadas al motor en T-20. |
| Hash de clave | Huella digital irreversible de la contraseña de un operador. | El dominio nunca ve ni almacena la clave en claro. |

---

## Technical terms of the project

| Term | Definition |
|------|-----------|
| Arquitectura Hexagonal | Patrón de diseño que aísla las reglas de negocio del dominio en el centro, comunicándose con el exterior mediante puertos y adaptadores. |
| Concurrencia Optimista | Estrategia de control de concurrencia basada en la columna de sistema `xmin` de Postgres, que evita sobrescrituras de stock concurrentes sin bloquear la tabla. |
| Objeto de Valor | Elemento del modelo de dominio que no posee identidad propia y se define por sus atributos (ej. `Money` o `Quantity`). Vive dentro de la fila de su dueño. |
| Baja Lógica | Técnica donde un registro nunca se elimina físicamente del motor. Utiliza una propiedad sombra `deleted_at` y un filtro global para ocultar activos dados de baja. |
| Propiedad Sombra | Atributo mapeado por Entity Framework que existe en la base de datos pero no está declarado explícitamente en las clases del modelo de dominio de C#. |
| Adaptador de Persistencia | Componente de infraestructura encargado de traducir las entidades de dominio y objetos de C# al modelo físico de la base de datos PostgreSQL. |
| Datos Semilla (Seed) | Registros obligatorios inyectados de forma fija en las migraciones iniciales para que el sistema sea funcional (ej. las cinco categorías fijas). |
| Esquema Cualificado | Práctica de anteponer el esquema `sales.` a todas las consultas de base de datos, lo que permite el uso de palabras clave como `user` sin entrecomillar. |

---

## Acronyms

| Acronym | Meaning |
|---------|---------|
| IAM | Identity and Access Management |
| JWT | JSON Web Token |
| API | Application Programming Interface |
| CRUD | Create, Read, Update, Delete |
| DTO | Data Transfer Object |
| FR | Functional Requirement |
| NFR | Non-Functional Requirement |
| SLO | Service Level Objective |
| SLA | Service Level Agreement |
| ADR | Architecture Decision Record |
| PR | Pull Request |
| DoD | Definition of Done |
| CI/CD | Continuous Integration / Continuous Delivery |

