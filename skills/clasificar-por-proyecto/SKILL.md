---
name: clasificar-por-proyecto
description: Asigna cuenta contable y centro de costo a las facturas de compra, las boletas de honorarios y las facturas de venta de empresas que trabajan por proyecto, obra, contrato, sucursal o línea de negocio, para tener un estado de resultados por centro de costo. Usar cuando el usuario dice "asigna centro de costo a las compras", "clasifica las compras por proyecto", "clasifica los honorarios por proyecto", "clasifica las ventas por proyecto", "ventas sin centro de costo", "a qué proyecto va esta factura", "a qué proyecto va esta boleta", "costos por proyecto", "costos por obra", "facturas sin centro de costo", "honorarios sin centro de costo", "reclasifica al proyecto", o cuando la empresa tiene centros de costo y sus compras, honorarios o ventas están quedando sin asignar.
---

# Clasificar compras, honorarios y ventas por proyecto (cuenta + centro de costo)

Cada factura de compra, cada boleta de honorarios y cada factura de venta lleva **dos decisiones separadas**:

- **Cuenta** = qué se compró, qué servicio se recibió o qué se vendió (materiales, mobiliario, subcontrato, flete, honorarios de proyecto, teléfono, ingresos por ventas). Sale del tercero y de su historial.
- **Centro de costo** = para qué proyecto. No sale de los libros: sale de una evidencia (lo dice el usuario, lo dice el documento, o el tercero trabaja para un solo proyecto).

Los gastos de estructura (teléfono, banco, software, contabilidad, arriendo de oficina, honorarios de administración) van al centro de costo de administración de la empresa. Así, **"Sin centro de costo" siempre significa "falta clasificar"**, y el estado de resultados por proyecto cuadra con el de la empresa.

Herramientas de Niutax Finanzas: `listar-libro-compras`, `listar-libro-honorarios`, `listar-libro-ventas`, `libro-mayor` (modo `analitico`), `estado-resultados-gestion`, `plan-de-cuentas`, `clasificar-compras`, `clasificar-honorarios`, `clasificar-ventas`, `crear-comprobante-borrador`, `consultar-propuestas`, `crear-tarea`.

## Reglas de base

- Todo lo que se envía es una **propuesta** que una persona aprueba en el panel (Ingreso Contable → Aprobaciones IA).
- Muestra la clasificación y espera un **sí** en un mensaje posterior antes de enviarla.
- **Nunca adivines el proyecto.** Un monto parecido o un nombre que suena similar no es evidencia. Sin evidencia, el documento va a la lista "falta el proyecto".
- `contabilizar: true` solo si el usuario lo pidió. `autoaprobar` solo si escribió la palabra «autoaprobar». Siempre `idempotency_key`.
- La suma de las líneas de cada documento debe ser exactamente su `monto_a_distribuir`: neto + exento en compras y ventas, **bruto** en honorarios (la retención no se clasifica).
- No muestres identificadores internos; nombra los proyectos por su nombre.

## Pasos

### 1. Empresa, mes y libros

Carga `elegir-empresa` si falta. Por defecto se trabajan **los tres libros** del mes: compras, honorarios y ventas; si el usuario pidió uno o dos, solo esos.

En un grupo con varios RUT se trabaja **una empresa a la vez**: los centros de costo son de cada empresa, y un proyecto que recibe documentos por dos RUT necesita su centro de costo en ambos.

### 2. Mapa de proyectos

Llama `estado-resultados-gestion` (tipo `anual`, desglose `cc-detalle`) y toma `centros_de_costo_disponibles`. Muestra la lista de proyectos y pregunta dos cosas en un solo mensaje: si falta algún proyecto nuevo y cuál es el centro de costo de administración.

- Si falta un proyecto o no existe el de administración, se crean en el panel de Niutax (centros de costo). Esta skill no los crea: pide al usuario que lo haga y sigue con los demás.
- Si la empresa no tiene ningún centro de costo, explica la idea en dos líneas, pide crearlos y termina.

### 3. Qué falta clasificar

En paralelo, para el mes:
- `listar-libro-compras`, `listar-libro-honorarios` y `listar-libro-ventas` con estado `todos`.
- `libro-mayor` modo `analitico` con el rango de fechas del mes (si devuelve `movimientos_omitidos`, parte el mes en rangos más cortos).

Un documento está **por asignar** si está pendiente, o si ya se contabilizó pero su movimiento en el mayor (mismo RUT, tipo de documento y folio) tiene el centro de costo vacío. Muchas empresas contabilizan los libros automáticamente a una cuenta por defecto: ahí "pendientes" sale vacío y todo el trabajo está en los contabilizados sin centro de costo. Dilo por libro, con el número de documentos y el monto.

### 4. Historial por tercero

Con `libro-mayor` analítico de los 3 a 6 meses anteriores, arma por RUT qué cuentas y qué centros de costo se usaron. Lo aprobado en meses anteriores es la memoria de la empresa: no hace falta guardar reglas en otro lado.

| El proveedor, prestador o cliente… | Cuenta | Centro de costo | Confianza |
|---|---|---|---|
| Es de estructura (telefonía, banco, software, contabilidad, seguros, asesoría a la administración) | Gasto de administración que corresponda | Administración | Alta |
| Siempre fue a un mismo proyecto que sigue abierto | La del historial | Ese proyecto | Confirmar |
| Fue a varios proyectos | La del historial | No proponer | Falta el proyecto |
| Es nuevo | Propón por giro, razón social o lo que diga el usuario | No proponer | Falta el proyecto |

Si todo el historial está en una sola cuenta por defecto (por ejemplo "Costo de ventas" u "Honorarios") y sin centro de costo, el historial no sirve como guía: dilo y propone la cuenta por la naturaleza del tercero.

### 5. Evidencia del proyecto

En este orden:
1. Lo que el usuario dice en la conversación, incluidas reglas ("todo lo de este proveedor en octubre es del proyecto X", "este profesional trabaja solo en el proyecto Y").
2. El documento mismo, si el usuario lo adjunta: en una factura, la referencia, orden de compra, dirección de despacho o glosa; en una boleta de honorarios, la descripción del servicio.
3. El historial del tercero (tabla anterior).

### 6. Honorarios

- El prestador es una persona, y su nombre no dice qué hizo. Para cada prestador sin historial útil, pregunta una vez si es **de proyecto** (arquitecto, dibujante, calculista, maestro, instalador) o **de estructura** (contador, abogado, administración), y recuérdalo el resto de la conversación.
- Honorario de proyecto: va a una cuenta de **costo** con el centro del proyecto. Si el plan solo tiene la cuenta de honorarios en gastos de administración, dilo y pregunta si se usa esa con el centro del proyecto o si el usuario crea una cuenta de honorarios en costos; no elijas por tu cuenta.
- Honorario de estructura: gasto de administración, centro de administración.
- Un profesional que trabaja en varios proyectos el mismo mes: una línea por proyecto. Pide la base del reparto (horas, porcentajes o montos) y aplícala sobre el bruto.

### 7. Ventas

- La cuenta es la de ingresos que la empresa ya usa; lo que hay que resolver es el centro de costo. Cada factura de venta corresponde al contrato de un proyecto.
- Cliente con un solo proyecto abierto: propón ese proyecto como "por confirmar". Cliente con varios proyectos: pregunta, o lee la glosa o la referencia de la factura si el usuario la adjunta.
- Una factura que cobra varios proyectos: una línea por proyecto con el monto de cada uno.
- Una venta que no es de ningún proyecto (venta de un activo, recuperación de gastos): pregunta la cuenta y el centro; no la mandes a un proyecto para completar.
- Con las ventas asignadas a su centro, la provisión de ingresos de `resultado-por-proyecto` se revierte sola cuando se factura.

### 8. Otros casos especiales

- **Nota de crédito** (de compra o de venta): misma cuenta y mismo centro de costo que la factura que rebaja (mismo RUT; busca el folio referenciado o el mismo monto). Si una factura y su nota de crédito se anulan por completo, van juntas al mismo centro.
- **Factura para varios proyectos**: una línea por proyecto. Pide la base del reparto (montos o porcentajes) y deja la diferencia de redondeo en la última línea.
- **Compra de activo fijo o existencia**: no es costo del mes. Propón la cuenta de activo y pregunta antes de seguir.

### 9. Mostrar

Primero compras, después honorarios y al final ventas. En cada libro, tres bloques con RUT, razón social, tipo y folio, monto, cuenta, centro de costo y el motivo en pocas palabras:

1. **Listas** — evidencia clara.
2. **Por confirmar** — propuesta basada en el historial.
3. **Falta el proyecto** — con la cuenta ya propuesta y el centro de costo en blanco.

Para el bloque 3 pregunta por los proyectos. Si el usuario no lo sabe, ofrece dejar una tarea con `crear-tarea` para quien lleva los proyectos, con la lista de documentos. No envíes el bloque 3 a "sin centro de costo" para avanzar.

### 10. Enviar

Con el sí, envía los bloques confirmados en lotes de hasta 100 documentos: las compras con `clasificar-compras`, las boletas con `clasificar-honorarios` y las ventas con `clasificar-ventas`, cada libro en su propia propuesta.

Si la herramienta rechaza documentos porque ya están contabilizados, no insistas. Propón un comprobante de reclasificación con `crear-comprobante-borrador` (debe: cuenta nueva con su centro de costo; haber: cuenta original sin centro de costo; mismo RUT, tipo de documento y folio en cada línea) y envíalo solo con un sí aparte.

Si `centro_costo` no es reconocido, pide al usuario el código del centro tal como aparece en el panel.

### 11. Cierre

Una línea por libro: documentos enviados, monto del mes que queda sin centro de costo y cuántos esperan respuesta sobre su proyecto. Recuerda que las propuestas se aprueban en Aprobaciones IA y sigue el desenlace con `consultar-propuestas`.

## Cuando otra skill carga esta

Vuelve a la skill que la llamó con: mes y, por libro, documentos enviados, monto sin centro de costo y documentos que esperan proyecto.
