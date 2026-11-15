---
name: cierre-mensual
description: Recorre el cierre de un mes de una empresa usando todos los módulos de Niutax — gastos, clasificación de compras, honorarios y ventas con su centro de costo, conciliación bancaria con anticipos, F29, provisión de ingresos por proyecto y estados financieros — en el orden correcto y con una aprobación en cada paso. Usar cuando el usuario dice "cierra el mes", "cierre mensual", "deja lista la contabilidad de octubre", "qué falta para cerrar", "cierre de [cliente]".
---

# Cierre mensual

Esta skill coordina; el trabajo lo hacen las demás skills de Niutax. Carga cada una por su nombre cuando llegue su paso y respeta sus reglas de aprobación.

## 1. Punto de partida

Empresa (carga `elegir-empresa`) y mes (por defecto, el último cerrado). Luego, en paralelo y solo lectura:
- Finanzas: `diagnostico-empresa` con área `mes`, `movimientos-por-conciliar`, `listar-libro-compras`, `listar-libro-honorarios` y `listar-libro-ventas` del mes, y `estado-resultados-gestion` (tipo `mensual` al mes, desglose `cc-detalle`).
- Tributario: `f29-del-rut` del mes.
- Payroll: `estado-del-mes` del mes, si la empresa tiene trabajadores en Payroll.
- Gastos: `listar_gastos` del mes, si el módulo está conectado.

La empresa **trabaja con centros de costo** si `centros_de_costo_disponibles` trae centros además de "Sin centro de costo". En ese caso la lista de control incluye las dos filas marcadas con (CC); si no, se omiten.

Muestra una lista de control con estado de cada paso:

| Paso | Estado |
|---|---|
| Gastos rendidos | n pendientes / al día |
| Compras clasificadas | n sin clasificar |
| Honorarios clasificados | n sin clasificar |
| Ventas clasificadas | n sin clasificar |
| Centro de costo asignado (CC) | $ de ingresos y $ de costos en "Sin centro de costo" |
| Banco conciliado | n movimientos, $ pendiente |
| Anticipos cruzados | $ de anticipos de clientes y proveedores con factura ya emitida |
| Remuneraciones | liquidaciones calculadas, mes cerrado, Previred y paso a Contabilidad |
| F29 | estado en el SII, monto, si está desactualizado |
| Ingresos de proyectos provisionados (CC) | hecho / pendiente este mes |
| Estados financieros | listos cuando lo anterior esté al día |

Pregunta por cuál empezar y sugiere el orden de la tabla.

## 2. Avanzar paso a paso

Para cada paso pendiente que el usuario elija, carga la skill: `registrar-gasto`, `clasificar-compras`, `clasificar-por-proyecto` (centro de costo de compras, honorarios y ventas), `conciliar-banco` (pagos, anticipos y su cruce con facturas), `mes-remuneraciones` (liquidaciones y su paso a Contabilidad, antes del F29 por las retenciones de segunda categoría), `mes-tributario`, `resultado-por-proyecto` (provisión de ingresos). Al volver, actualiza la lista de control y pregunta por el siguiente. No saltes pasos sin preguntar.

Un mes con monto en "Sin centro de costo" no está cerrado para una empresa que trabaja por proyecto: dilo si el usuario quiere pasar a los estados financieros con ese paso pendiente.

## 3. Cierre

Cuando banco y libros estén al día, carga `reportes-financieros` para el mes y entrega el resumen. Si la empresa trabaja con centros de costo, agrega el resultado por proyecto con `resultado-por-proyecto`. Recuerda que las propuestas enviadas siguen pendientes hasta que alguien las apruebe en Aprobaciones IA, y que el F29 se declara en el panel.

Termina con una línea: qué quedó listo, qué queda pendiente y quién debe aprobar qué.
