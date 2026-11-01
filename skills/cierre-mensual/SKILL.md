---
name: cierre-mensual
description: Recorre el cierre de un mes de una empresa usando todos los módulos de Niutax — gastos, clasificación de compras y ventas, conciliación bancaria, F29 y estados financieros — en el orden correcto y con una aprobación en cada paso. Usar cuando el usuario dice "cierra el mes", "cierre mensual", "deja lista la contabilidad de octubre", "qué falta para cerrar", "cierre de [cliente]".
---

# Cierre mensual

Esta skill coordina; el trabajo lo hacen las demás skills de Niutax. Carga cada una por su nombre cuando llegue su paso y respeta sus reglas de aprobación.

## 1. Punto de partida

Empresa (carga `elegir-empresa`) y mes (por defecto, el último cerrado). Luego, en paralelo y solo lectura:
- Finanzas: `diagnostico-empresa` con área `mes`, `movimientos-por-conciliar`, `listar-libro-compras` y `listar-libro-ventas` del mes.
- Tributario: `f29-del-rut` del mes.
- Gastos: `listar_gastos` del mes, si el módulo está conectado.

Muestra una lista de control con estado de cada paso:

| Paso | Estado |
|---|---|
| Gastos rendidos | n pendientes / al día |
| Compras clasificadas | n sin clasificar |
| Ventas clasificadas | n sin clasificar |
| Banco conciliado | n movimientos, $ pendiente |
| F29 | estado en el SII, monto, si está desactualizado |
| Estados financieros | listos cuando lo anterior esté al día |

Pregunta por cuál empezar y sugiere el orden de la tabla.

## 2. Avanzar paso a paso

Para cada paso pendiente que el usuario elija, carga la skill: `registrar-gasto`, `clasificar-compras`, `conciliar-banco`, `mes-tributario`. Al volver, actualiza la lista de control y pregunta por el siguiente. No saltes pasos sin preguntar.

## 3. Cierre

Cuando banco y libros estén al día, carga `reportes-financieros` para el mes y entrega el resumen. Recuerda que las propuestas enviadas siguen pendientes hasta que alguien las apruebe en Aprobaciones IA, y que el F29 se declara en el panel.

Termina con una línea: qué quedó listo, qué queda pendiente y quién debe aprobar qué.
