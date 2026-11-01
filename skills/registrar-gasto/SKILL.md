---
name: registrar-gasto
description: Registra gastos en Niutax Gastos (NiuGastos) a partir de fotos o PDF de boletas, facturas y recibos, o de lo que el usuario dicte, con su categoría y medio de pago. Usar cuando el usuario dice "registra este gasto", "rinde estas boletas", "sube este comprobante", "anota que gasté", "rendición de gastos", o adjunta fotos de boletas.
---

# Registrar gastos

Herramientas de Niutax Gastos: `listar_empresas_autorizadas`, `listar_categorias`, `listar_medios_pago`, `crear_gasto`, `listar_gastos`, `actualizar_gasto`.

## Reglas

- Muestra los gastos leídos y espera un **sí** antes de crearlos. Siempre `idempotency_key` (por ejemplo `gasto-<rut emisor>-<folio>-<fecha>`).
- No inventes montos, fechas, RUT ni folios. Si una foto no se lee bien, dilo y pide el dato.
- Si el documento no está en pesos, registra `currency` y pregunta el monto en pesos (`amountCLP`) si no viene.
- Antes de crear, revisa con `listar_gastos` que no esté ya registrado (mismo monto, fecha y folio).

## Pasos

1. Empresa (carga `elegir-empresa` si falta). Carga sus categorías y medios de pago.
2. Lee cada comprobante: fecha, emisor y RUT, tipo (boleta, factura, recibo, ticket, sin comprobante), folio, monto y una descripción corta.
3. Propone categoría y medio de pago para cada uno. Muestra una tabla y marca lo dudoso.
4. Pregunta si los registra. Termina el turno.
5. Con el sí, llama `crear_gasto` por cada uno. Cierra en una línea: cuántos quedaron registrados como pendientes y el total.

Si el gasto es una **factura de proveedor**, recuerda que también llega al libro de compras: su clasificación contable se hace con `clasificar-compras`.
