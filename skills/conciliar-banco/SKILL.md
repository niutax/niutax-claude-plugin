---
name: conciliar-banco
description: Concilia la cuenta bancaria de una empresa en Niutax Finanzas: carga la cartola si falta, propone la imputación de cada movimiento pendiente y lo deja conciliado al aprobarse. Usar cuando el usuario dice "concilia el banco", "conciliación bancaria", "sube esta cartola", "qué falta por conciliar", "imputa los movimientos del banco", o adjunta una cartola en Excel, CSV o PDF.
---

# Conciliar el banco

Herramientas de Niutax Finanzas: `movimientos-por-conciliar`, `cargar-movimientos-banco`, `contabilizar-movimiento-banco`, `comentar-movimientos-banco`, `plan-de-cuentas`, `listar-auxiliares`, `cuentas-por-cobrar`, `cuentas-por-pagar`, `consultar-propuestas`.

## Reglas de base

- Ninguna herramienta escribe directo: todas dejan una **propuesta** que una persona aprueba en el panel (Ingreso Contable → Aprobaciones IA).
- Aun así, muestra siempre lo que vas a proponer y espera un **sí** del usuario en un mensaje posterior. La respuesta que entrega los datos no es ese sí.
- Envía `autoaprobar` solo si el usuario escribió literalmente la palabra «autoaprobar».
- Envía `idempotency_key` en cada escritura (por ejemplo `concil-<rut>-<movimiento_id>`).
- Para reconocer un movimiento del banco usa `contabilizar-movimiento-banco`, nunca `crear-comprobante-borrador`: así el comprobante nace con la fecha del banco y queda conciliado.
- No inventes montos, fechas ni cuentas. Si no sabes a qué cuenta va un movimiento, pregúntalo o déjalo con un comentario.
- Si algo falla dos veces, detente, di qué respondió el sistema y una acción para el usuario. Ante una falla de configuración, corre `diagnostico-empresa` con área `tesoreria`.

## 1. Empresa, cuenta y mes

Si no hay empresa elegida, carga `elegir-empresa`. Pregunta la cuenta bancaria y el mes si no los dio. Termina el turno.

## 2. Cartola

Llama `movimientos-por-conciliar` con el mes y la cuenta.

- Si el usuario adjuntó una cartola: léela y compárala con lo ya cargado. Muestra solo los movimientos que faltan (fecha, glosa, cargo o abono) y pregunta si los carga. Con un sí, llama `cargar-movimientos-banco` (máximo 100 por llamada; divide si hay más). Los duplicados por fecha y monto se omiten solos: no uses `permitir_duplicados` salvo que el usuario confirme que son movimientos distintos.
- Si no hay cartola y no hay movimientos cargados: pide la cartola y termina el turno.

## 3. Proponer la imputación

Con los movimientos pendientes:
1. Carga el `plan-de-cuentas` una vez.
2. Para cada movimiento, propone la contrapartida:
   - Abonos que calzan con una factura de `cuentas-por-cobrar` → cliente, con rut, tipo de documento y folio.
   - Cargos que calzan con `cuentas-por-pagar` → proveedor, con rut, tipo de documento y folio.
   - Comisiones, impuestos, remuneraciones, traspasos entre cuentas → la cuenta que corresponda del plan.
   - Sin pista clara → no lo propongas; márcalo "por definir".
3. Muestra una tabla: fecha, glosa, monto, cuenta propuesta, auxiliar/folio y nivel de confianza (alta / revisar). Agrupa los "por definir" al final y pregunta qué son.

Pregunta: "¿Envío estas N propuestas a Aprobaciones IA?" Termina el turno.

## 4. Enviar

Con el sí, llama `contabilizar-movimiento-banco` por cada movimiento aprobado. La suma de las líneas debe ser exactamente el monto del movimiento; una comisión descontada de un abono va como línea negativa. Para los que el usuario explicó pero no quiere contabilizar aún, ofrece dejar el comentario con `comentar-movimientos-banco`.

## 5. Cierre

Responde en una o dos líneas: cuántas propuestas quedaron pendientes en Aprobaciones IA, cuántos movimientos siguen por conciliar y el monto. Ofrece revisar el desenlace después con `consultar-propuestas` (rechazadas: corregir y reenviar).

## Cuando otra skill carga esta

Si viene de `inicio` o `cierre-mensual`, al terminar vuelve a esa skill en la misma respuesta con: cuenta, propuestas enviadas, pendientes restantes. No ofrezcas pasos propios.
