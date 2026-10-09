# ADR-005 — Política de Plataforma Monomoneda

- **ID:** ADR-005
- **Date:** 2026-10-08
- **Status:** Accepted
- **Authors:** Core Tech Lead

---

## Context

El diseño del modelo físico de la base de datos `simple_stock_flow` bajo el esquema `sales` requiere una definición estricta sobre el manejo de divisas en las tablas de precios de catálogo (`product.price`) y subtotales transaccionales (`sale_item.unit_price`). Dejar la puerta abierta a múltiples monedas sin una definición estructural introduce una desnormalización costosa, duplicación de columnas de control de divisas (`unit_price_currency`) y sobrecarga matemática en el cálculo dinámico del reporte agregado (Q9).

**Known constraints:**
- El sistema debe ser monomoneda por construcción (Restricción innegociable D-05).
- Las columnas de montos monetarios están mapeadas rígidamente como `numeric(18,2)` sin redondeos automáticos por defecto en el motor PostgreSQL 16.14.

---

## Decision

**We decided:** Diseñar y consolidar el sistema entero bajo una arquitectura estrictamente monomoneda, omitiendo cualquier columna de control de divisas en las tablas físicas y delegando la normalización de precisión de importes exclusivamente al Objeto de Valor `Money` de la capa de dominio en C# antes de confirmar transacciones.

**Justification:**
La simplicidad operativa y la estabilidad del histórico de reportes cerrados dictan el descarte de conversión de tipos de cambio en caliente. El sistema asume una moneda única corporativa implícita. Esto permite que la agregación costosa del reporte (Q9) realice operaciones de suma directa en el motor de forma instantánea sin necesidad de tablas puente de conversión de tasas de cambio.

---

## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|------------|------|------|-----------------------|
| Arquitectura Monomoneda [Chosen] | Eliminación completa de columnas redundantes de divisa; cálculo dinámico directo en motor. | No permite transacciones internacionales nativas con tipos de cambio cruzados. | — (Chosen por cumplimiento estricto de D-05) |
| Soporte Multimoneda con columnas de control | Permite flexibilidad comercial por línea de venta. | Encarece cada escritura; rompe la estructura limpia de una sola columna e invalida el GROUP BY simple en Q9. | Descartada por requerimiento explícito del propietario; alcance inventado (DP-03). |

---

## Consequences

**Positive:**
- Reducción del modelo físico a solo 22 columnas limpias sin excedente (§3).
- Optimización de índices compuestos en `sale_item` que no requieren evaluar el tipo de moneda para sumar importes.

**Negative / Trade-offs:**
- El sistema queda acoplado a una sola divisa implícita. Si el negocio se expande internacionalmente, requerirá una refactorización estructural pesada del esquema.

**Impact on the system:**
- Affected services: simple-stock-flow-api
- Documents that must be updated: `data_model.md`, `project-glossary.md`

---

## Risks

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|-----------|
| Recorte de precisión por truncado directo del motor | Low | Medium | El constructor de `Money` en C# aplica un redondeo estricto a 2 decimales con `MidpointRounding.AwayFromZero` antes de persistir la fila. |

---

## References

- Constitución del Sistema (Artículo VII - Suma de subtotales no almacenados)
- Mapeo físico de columnas de precisión en `data_model.md` §3
- Related to: ADR-004 (Reporte agregado y congelado)

