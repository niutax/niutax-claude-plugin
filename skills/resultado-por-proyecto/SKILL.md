---
name: resultado-por-proyecto
description: Arma el estado de resultados por centro de costo (proyecto, obra, contrato, sucursal) en Niutax Finanzas y calcula la provisión de ingresos de cada proyecto según su contrato, para comparar el ingreso devengado con los costos del período. Usar cuando el usuario pide "EERR por centro de costo", "resultado por proyecto", "margen por proyecto", "cuánto llevamos gastado en el proyecto", "provisiona los ingresos de los proyectos", "ingreso según contrato", "grado de avance", "rentabilidad por obra".
---

# Resultado por proyecto

Cada proyecto es un centro de costo con dos lados: el **ingreso**, que se reconoce según el contrato aunque todavía no se facture, y los **costos**, que llegan con las compras, honorarios y remuneraciones asignados a ese centro.

Herramientas de Niutax Finanzas: `estado-resultados-gestion`, `libro-mayor` (modo `analitico`), `plan-de-cuentas`, `crear-comprobante-borrador`, `consultar-propuestas`.

## Reglas

- Usa solo cifras que devuelven las herramientas y datos de contrato que entrega el usuario. No estimes avances ni montos de contrato.
- El comprobante de provisión es una **propuesta** que se aprueba en Aprobaciones IA. Muéstralo y espera un **sí** antes de enviarlo. Siempre `idempotency_key`.
- El método para reconocer el ingreso lo decide el contador, no esta skill.
- No muestres identificadores internos.

## Pasos

### 1. Empresa y período

Carga `elegir-empresa` si falta. Por defecto, acumulado del año al último mes cerrado.

### 2. Control antes del informe

Llama `estado-resultados-gestion` (tipo `mensual` o `anual`, desglose `cc-detalle`). Mira la columna "Sin centro de costo":

- Si tiene ingresos o costos, dilo primero, con monto y porcentaje del total: el resultado por proyecto está incompleto en esa medida. Ofrece cargar `clasificar-por-proyecto` (compras, honorarios y ventas) antes de seguir.
- Si el usuario quiere ver el informe igual, entrégalo con la advertencia arriba.

### 3. Estado de resultados por proyecto

Una fila por centro de costo: ingresos, costos, margen y margen %. Administración va en su propia fila, y al final el total, que debe ser igual al resultado de la empresa. Para el detalle de un proyecto, vuelve a llamar con su `centro_costo_id` o usa `libro-mayor` analítico filtrando por ese centro.

Señala lo que pide atención: proyectos con costos y sin ingresos (falta provisionar o facturar), con ingresos y sin costos (faltan compras u honorarios por asignar) y con margen negativo.

### 4. Provisión de ingresos según contrato

Solo si el usuario la pide. Por cada proyecto abierto necesitas del usuario (el conector no tiene los contratos):

- Monto neto del contrato y sus modificaciones.
- El método de reconocimiento:
  - **Hitos o estados de pago**: lo aprobado por el cliente a la fecha.
  - **Grado de avance por costos**: costo acumulado del centro ÷ costo total presupuestado × contrato. Requiere el presupuesto de costos del proyecto.
  - **Lineal**: contrato ÷ meses del plazo × meses transcurridos.

Si el usuario adjunta una planilla de contratos, léela en vez de preguntar proyecto por proyecto.

Cálculo, siempre acumulado:

- **Ingreso devengado** a la fecha, según el método.
- **Ingreso registrado** en el centro: ventas facturadas y provisiones anteriores (del estado de resultados por centro de costo).
- **Ajuste del mes** = devengado − registrado.

| Ajuste | Asiento (con el centro de costo del proyecto en la línea de ingresos) |
|---|---|
| Positivo: falta reconocer | Debe: ingresos por facturar (activo) · Haber: ventas |
| Negativo: se facturó por adelantado | Debe: ventas · Haber: ingresos anticipados (pasivo) |

Como el cálculo es acumulado, cuando se emite la factura el ajuste del mes siguiente se revierte solo. Para que eso funcione, las facturas de venta también deben llevar el centro de costo del proyecto: si hay ventas del período sin centro, clasifícalas antes con `clasificar-por-proyecto`.

Busca las cuentas en `plan-de-cuentas`. Si no existen cuentas de ingresos por facturar o de ingresos anticipados, pide al usuario que las cree o que indique cuál usar; no uses una parecida por tu cuenta.

Muestra una tabla por proyecto (contrato, avance, devengado, registrado, ajuste) y el comprobante completo. Pregunta también si es solo financiero (`regulation: 1`) o también tributario: la provisión es un ajuste de gestión y su efecto tributario lo define el contador. Con el sí, envía un comprobante por mes con fecha del último día del mes.

### 5. Entrega

En el chat: la tabla por proyecto y 3 conclusiones con su cifra. Si el usuario lo va a compartir, un documento o página con la marca Niutax (verde). Termina con un siguiente paso concreto.

## Cuando otra skill carga esta

Vuelve con: período, monto sin centro de costo, proyectos con margen negativo y si la provisión quedó enviada.
