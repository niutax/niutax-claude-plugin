---
name: emitir-documento
description: Prepara la emisión de documentos tributarios electrónicos en Niutax Facturación (niupos) — factura afecta o exenta, boleta, nota de crédito, nota de débito y guía de despacho — y deja la propuesta lista para aprobar. Usar cuando el usuario dice "emite una factura", "haz una boleta", "factúrale a [cliente]", "nota de crédito para la factura", "guía de despacho", "anula esta factura".
---

# Emitir un documento

Herramientas de Niutax Facturación: `listar-mis-empresas`, `buscar-auxiliar`, `crear-auxiliar`, `obtener-producto`, `crear-factura`, `crear-boleta`, `crear-nota-credito`, `crear-nota-debito`, `crear-guia-despacho`, `consultar-propuestas`, `descargar-pdf-documento`.

## Reglas de base

- Ninguna herramienta emite directo: dejan una **propuesta** que se aprueba en niupos (Configuración → Aprobaciones IA).
- Una factura con `send: true` y toda boleta 39 o 41 se emiten con folio y se envían al SII **apenas se aprueban: no se puede deshacer**. Dilo antes de pedir el sí.
- Muestra el documento completo y espera un **sí** en un mensaje posterior. Siempre `idempotency_key`.
- No inventes RUT de chofer o transportista, patentes, precios ni códigos de producto. Si faltan, pregúntalos.

## Pasos

1. Empresa emisora (carga `elegir-empresa` si falta) y tipo de documento.
2. **Receptor**: búscalo con `buscar-auxiliar` y confirma la razón social. Si no existe, pregunta antes de proponer `crear-auxiliar` (solo RUT, y correo o teléfono si el usuario los tiene; niupos trae del SII la razón social, dirección y giro).
3. **Ítems**: obtén cada producto con `obtener-producto` para su código de venta y precio. En facturas el precio es **neto**; en boletas es **bruto**.
4. **Datos del documento**: fecha, ubicación (sucursal), forma de pago, tipo de venta y de compra. Para nota de crédito o débito, el documento de referencia y el motivo. Para traslado de bienes, los datos de transporte.
5. Muestra el borrador: receptor, ítems, neto, IVA, total, y si se enviará al SII al aprobarse. Pregunta si lo propone. Termina el turno.
6. Con el sí, llama la herramienta. Responde en una línea que quedó pendiente de aprobación en niupos.
7. Si el usuario vuelve a preguntar, usa `consultar-propuestas` para ver el folio asignado y ofrece el PDF con `descargar-pdf-documento`.
