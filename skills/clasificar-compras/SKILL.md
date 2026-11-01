---
name: clasificar-compras
description: Clasifica los documentos pendientes de los libros de compras, ventas u honorarios en Niutax Finanzas, asignando cuenta y centro de costo para que se contabilicen al aprobarse. Usar cuando el usuario dice "clasifica las compras", "centraliza el libro de compras", "asigna cuentas a las facturas", "clasifica honorarios", "clasifica las ventas del mes".
---

# Clasificar libros

Herramientas de Niutax Finanzas: `listar-libro-compras`, `listar-libro-ventas`, `listar-libro-honorarios`, `resumen-libro-compras`, `plan-de-cuentas`, `clasificar-compras`, `clasificar-ventas`, `clasificar-honorarios`, `consultar-propuestas`.

## Reglas de base

- Las clasificaciones son **propuestas** que se aprueban en el panel (Ingreso Contable → Aprobaciones IA).
- Muestra la clasificación y espera un **sí** en un mensaje posterior antes de enviarla.
- `contabilizar: true` solo si el usuario pidió explícitamente que además se contabilicen.
- `autoaprobar` solo si el usuario escribió la palabra «autoaprobar». Siempre `idempotency_key`.
- La suma de las líneas de cada documento debe ser exactamente su monto a distribuir (neto + exento) que entrega `listar-libro-*`.

## Pasos

1. Empresa (carga `elegir-empresa` si falta), libro (compras por defecto) y mes. Termina el turno si falta algo.
2. Llama `listar-libro-*` del mes y quédate con los documentos sin clasificar. Si no hay, dilo y termina.
3. Carga el `plan-de-cuentas`. Para cada proveedor, revisa si en meses anteriores ya se clasificó (mismo RUT) y repite esa cuenta; si es nuevo, propone por el giro o la glosa. Marca "revisar" cuando no haya historial.
4. Muestra una tabla agrupada por proveedor: RUT, razón social, tipo y folio, monto a distribuir, cuenta, centro de costo, confianza. Si un documento se reparte entre cuentas, muestra cada línea.
5. Pregunta si envía. Con el sí, llama `clasificar-*` en lotes de hasta 100 documentos.
6. Cierra en una línea: documentos enviados, pendientes que quedan y que deben aprobarse en Aprobaciones IA.

## Cuando otra skill carga esta

Vuelve a la skill que la llamó con: libro, mes, documentos enviados y pendientes.
