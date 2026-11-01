---
name: inicio
description: Primer paso con Niutax en Claude. Revisa qué módulos están conectados, elige la empresa, diagnostica su configuración y propone un solo siguiente paso (conciliar, clasificar compras, revisar el F29 o ver los estados financieros). Usar cuando el usuario dice "empecemos", "configura Niutax", "qué puedes hacer con Niutax", "/niutax:inicio", o es su primera conversación con el plugin.
---

# Inicio con Niutax

Deja al usuario conectado, con una empresa elegida y con un siguiente paso claro. Cada mensaje lleva una sola pregunta.

## 1. Revisar conexiones

Comprueba qué conectores de Niutax responden: Finanzas, Tributario, Facturación y Gastos. Si alguno no está disponible o pide inicio de sesión, llama una herramienta de lectura de ese módulo para que aparezca su tarjeta de conexión. Si no aparece, indica que lo conecte en Configuración → Conectores. Sigue con los módulos que sí estén.

Muestra una línea por módulo: conectado o no conectado. Nada más.

## 2. Elegir la empresa

Carga la skill `elegir-empresa`. Con un estudio contable (muchos RUT en la cartera), pregunta con qué cliente partir.

## 3. Diagnosticar

En paralelo, cuando el módulo esté conectado:
- Finanzas: `diagnostico-empresa` (sin área, revisa todo).
- Tributario: `diagnostico-rut` y `resumen-de-hoy` con el RUT.

Resume en máximo 5 líneas lo que falta configurar (clave SII, plan de cuentas, cuentas bancarias, cartola, F29 automático) con la pantalla del panel donde se arregla. No inventes causas que el diagnóstico no muestra.

## 4. Proponer un siguiente paso

Elige **uno** según el estado:

| Si el diagnóstico muestra… | Propón |
|---|---|
| Movimientos de cartola sin conciliar | `conciliar-banco` |
| Documentos de compras sin clasificar | `clasificar-compras` |
| F29 del último mes sin declarar, observado o con alertas | `mes-tributario` |
| Anotaciones, giros o DJ pendientes | `contingencias-sii` |
| Todo al día | `reportes-financieros` (salud del negocio) |

Pregunta si lo hacen ahora. Con un sí, carga esa skill con la empresa ya elegida.

## Reglas

- Este flujo solo lee. No envíes propuestas desde aquí.
- No muestres identificadores internos.
- Si un dato viene "sin sincronizar" o sin consultar, dilo así: no es un cero.
