---
name: conciliar-banco
description: Concilia la cuenta bancaria de una empresa en Niutax Finanzas y registra su recaudación y sus pagos: carga la cartola si falta, calza cada abono o cargo con la factura que paga, registra los anticipos de clientes y de proveedores, los cruza contra la factura cuando se emite y deja cada movimiento conciliado al aprobarse. Usar cuando el usuario dice "concilia el banco", "conciliación bancaria", "recaudación y pagos", "sube esta cartola", "qué falta por conciliar", "imputa los movimientos del banco", "registra este anticipo", "anticipo de cliente", "anticipo de proveedor", "cruza los anticipos", "aplica el anticipo a la factura", o adjunta una cartola en Excel, CSV o PDF.
---

# Conciliar el banco: recaudación y pagos

Herramientas de Niutax Finanzas: `movimientos-por-conciliar`, `cargar-movimientos-banco`, `contabilizar-movimiento-banco`, `comentar-movimientos-banco`, `crear-comprobante-borrador`, `libro-mayor` (modo `analitico`), `plan-de-cuentas`, `listar-auxiliares`, `cuentas-por-cobrar`, `cuentas-por-pagar`, `consultar-propuestas`.

## Reglas de base

- Ninguna herramienta escribe directo: todas dejan una **propuesta** que una persona aprueba en el panel (Ingreso Contable → Aprobaciones IA).
- Aun así, muestra siempre lo que vas a proponer y espera un **sí** del usuario en un mensaje posterior. La respuesta que entrega los datos no es ese sí.
- Envía `autoaprobar` solo si el usuario escribió literalmente la palabra «autoaprobar».
- Envía `idempotency_key` en cada escritura (por ejemplo `concil-<rut>-<movimiento_id>`).
- Para reconocer un movimiento del banco usa `contabilizar-movimiento-banco`, nunca `crear-comprobante-borrador`: así el comprobante nace con la fecha del banco y queda conciliado. La única excepción es el cruce de anticipos (paso 5), que no es un movimiento del banco.
- No inventes montos, fechas ni cuentas. Si no sabes a qué cuenta va un movimiento o de quién es, pregúntalo o déjalo con un comentario.
- Si algo falla dos veces, detente, di qué respondió el sistema y una acción para el usuario. Ante una falla de configuración, corre `diagnostico-empresa` con área `tesoreria`.

## Cómo se registra un anticipo

Un anticipo es un abono de un cliente, o un pago a un proveedor, que llega **antes de que exista la factura**. No rebaja ninguna factura: queda en la cuenta de anticipos a nombre del tercero hasta que la factura se emite.

| Dato de la línea | Valor |
|---|---|
| Cuenta | La de anticipos del plan: anticipos de clientes (pasivo), anticipo a proveedores o anticipos de honorarios (activo) |
| RUT | El del cliente o proveedor |
| Tipo de documento | El tipo de documento de anticipo de la empresa, que lleva el mismo nombre que la cuenta |
| Folio | Siempre **1**: agrupa todos los anticipos de ese tercero en un solo saldo |

Busca las cuentas con `plan-de-cuentas` (búsqueda "anticipo"). Para el código del tipo de documento, mira cómo se registraron anticipos anteriores con `libro-mayor` analítico de esa cuenta; si no hay ninguno, pregúntalo una vez. No uses el código de una factura.

## 1. Empresa, cuenta y mes

Si no hay empresa elegida, carga `elegir-empresa`. Pregunta la cuenta bancaria y el mes si no los dio. Termina el turno.

## 2. Cartola

Llama `movimientos-por-conciliar` con el mes y la cuenta.

- Si el usuario adjuntó una cartola: léela y compárala con lo ya cargado. Muestra solo los movimientos que faltan (fecha, glosa, cargo o abono) y pregunta si los carga. Con un sí, llama `cargar-movimientos-banco` (máximo 100 por llamada; divide si hay más). Los duplicados por fecha y monto se omiten solos: no uses `permitir_duplicados` salvo que el usuario confirme que son movimientos distintos.
- Si no hay cartola y no hay movimientos cargados: pide la cartola y termina el turno.

## 3. Proponer la imputación

Carga una vez `plan-de-cuentas`, `cuentas-por-cobrar`, `cuentas-por-pagar` y `listar-auxiliares`. Identifica al tercero de cada movimiento por el RUT o el nombre que trae la glosa. Los abonos se buscan en cuentas por cobrar y los cargos en cuentas por pagar, en este orden:

| Caso | Qué se propone | Confianza |
|---|---|---|
| **Pago total**: el monto es igual al saldo de una factura | Una línea contra esa factura: rut, tipo de documento y folio | Alta si el tercero coincide o el monto es único entre los documentos abiertos; "revisar" si hay dos facturas con ese monto |
| **Varias facturas**: el monto es la suma de facturas abiertas del mismo tercero | Una línea por factura | Alta si la combinación es única |
| **Pago parcial**: tercero identificado y el monto es menor que su factura abierta | Abono a esa factura (la más antigua si hay varias) | Revisar |
| **Anticipo**: tercero identificado y sin factura abierta | Línea a la cuenta de anticipos con rut, tipo de documento de anticipo y folio 1 | Revisar |
| **Pago con excedente**: el monto supera lo que el tercero tiene abierto | Líneas contra sus facturas y el resto como anticipo | Revisar |
| Comisiones, impuestos, remuneraciones, traspasos entre cuentas | La cuenta que corresponda del plan | Según historial |
| Sin tercero ni pista clara | No lo propongas; márcalo "por definir" | — |

Un abono sin tercero identificado no es un anticipo: es "por definir" hasta saber de quién es.

Muestra una tabla: fecha, glosa, monto, caso, cuenta propuesta, auxiliar y folio, y confianza. Agrupa los "por definir" al final y pregunta qué son.

Pregunta: "¿Envío estas N propuestas a Aprobaciones IA?" Termina el turno.

## 4. Enviar

Con el sí, llama `contabilizar-movimiento-banco` por cada movimiento aprobado. La suma de las líneas debe ser exactamente el monto del movimiento; una comisión descontada de un abono va como línea negativa. Para los que el usuario explicó pero no quiere contabilizar aún, ofrece dejar el comentario con `comentar-movimientos-banco`.

## 5. Cruzar anticipos con facturas

Hazlo después del paso 4, o directamente cuando el usuario pida cruzar anticipos.

1. Saldo de anticipos por tercero: `libro-mayor` analítico de cada cuenta de anticipos, sumado por RUT.
2. Para cada RUT con saldo, busca sus facturas abiertas en `cuentas-por-cobrar` (clientes) o `cuentas-por-pagar` (proveedores y honorarios).
3. Donde haya anticipo y factura abierta, propone el cruce por el **menor** entre el saldo del anticipo y el saldo de la factura, partiendo por la factura más antigua:

| Cruce | Debe | Haber |
|---|---|---|
| Cliente | Anticipos de clientes (rut, documento de anticipo, folio 1) | Clientes (rut, tipo y folio de la factura) |
| Proveedor u honorarios | Proveedores u honorarios por pagar (rut, tipo y folio del documento) | Anticipo a proveedores u honorarios (rut, documento de anticipo, folio 1) |

4. Muestra una tabla: tercero, saldo de anticipo, factura, saldo de la factura, monto a cruzar y lo que queda de cada lado. Con el sí, envía un comprobante por factura con `crear-comprobante-borrador`, con la fecha de la factura (si ese mes ya está cerrado, pregunta la fecha).

Si el saldo de anticipos no tiene RUT (por ejemplo viene de la apertura como un solo monto), no se puede cruzar: dilo con el monto y pide al usuario el detalle por tercero para abrirlo.

## 6. Cierre

Responde en pocas líneas: propuestas pendientes en Aprobaciones IA, movimientos que siguen por conciliar con su monto, anticipos registrados y cruzados, y saldo de anticipos que queda sin factura. Ofrece revisar el desenlace después con `consultar-propuestas` (rechazadas: corregir y reenviar).

## Cuando otra skill carga esta

Si viene de `inicio` o `cierre-mensual`, al terminar vuelve a esa skill en la misma respuesta con: cuenta, propuestas enviadas, pendientes restantes y anticipos por cruzar. No ofrezcas pasos propios.
