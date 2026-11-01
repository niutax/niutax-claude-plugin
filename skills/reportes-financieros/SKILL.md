---
name: reportes-financieros
description: Arma los estados financieros y un reporte de salud del negocio desde Niutax Finanzas — balance clasificado, estado de resultados, flujo de caja, cuentas por cobrar y por pagar, comparación con presupuesto y holdings. Usar cuando el usuario pide "el balance", "estado de resultados", "cómo va la empresa", "reporte para el directorio", "EERR comparativo", "flujo de caja", "real vs presupuesto".
---

# Reportes financieros

Herramientas de Niutax Finanzas: `balance-clasificado`, `estado-resultados`, `estado-resultados-gestion`, `flujo-de-caja-historico`, `cuentas-por-cobrar`, `cuentas-por-pagar`, `balance-ocho-columnas`, `consultar-presupuesto`, `comparativo-mensual-holding`, `listar-holdings`, `totales-libro-diario`.

## Reglas

- Usa solo cifras que devuelven las herramientas. No estimes ni completes datos faltantes.
- Montos en pesos chilenos salvo que el dato indique otra moneda.
- Antes de presentar, comprueba que el balance cuadra (activo = pasivo + patrimonio). Si no cuadra, dilo y sugiere revisar con `balance-ocho-columnas`.
- Si la contabilidad del período parece incompleta (mes sin movimientos, cartola sin conciliar), adviértelo antes de las conclusiones.

## Pasos

1. Empresa (o holding) y período. Por defecto, último mes cerrado con comparativo del período anterior.
2. Llama en paralelo lo que pida el reporte:
   - **Salud del negocio** (por defecto): balance clasificado, estado de resultados con comparativo, flujo de caja histórico, cuentas por cobrar y por pagar.
   - **Real vs presupuesto**: estado de resultados + `consultar-presupuesto`.
   - **Holding**: `comparativo-mensual-holding`.
3. Calcula indicadores sobre esas cifras: margen bruto y neto, liquidez corriente, endeudamiento, días de cobro y pago, EBITDA si las cuentas lo permiten.
4. Entrega:
   - En el chat: 3 a 5 conclusiones en lenguaje simple y la cifra que respalda cada una.
   - Si el usuario lo va a compartir o guardar: un documento o una página con los estados, los indicadores y un gráfico de evolución, con la marca Niutax (verde).
5. Termina con un siguiente paso concreto (por ejemplo, cobrar las facturas más vencidas).
