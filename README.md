# Niutax para Claude

Contabilidad, impuestos, facturación y gastos de empresas chilenas en Claude, sobre el ecosistema Niutax. Cada módulo es un conector propio:

| Conector | Qué hace | Escribe |
|---|---|---|
| Niutax Finanzas | Contabilidad: libros, cartola, conciliación, comprobantes, EEFF, presupuesto | Propuestas que se aprueban en Aprobaciones IA |
| Niutax Tributario | F29, F22, DJ, alertas y anotaciones del SII, vencimientos | No (solo lectura) |
| Niutax Facturación | Facturas, boletas, notas, guías, clientes, productos, ventas | Propuestas que se aprueban en niupos |
| Niutax Gastos | Registro y rendición de gastos | Gastos pendientes |

## Instalación

En Claude Code:

```
/plugin install niutax --marketplace niutax/niutax-claude-plugin
```

En Claude (web o escritorio): Personalizar → Plugins, busca **Niutax** en el directorio o sube el archivo `.plugin`.

Política de privacidad: https://niutax.cl/privacidad

## Skills

| Skill | Para qué |
|---|---|
| `/niutax:inicio` | Primer uso: conecta, elige empresa, diagnostica y propone un paso |
| `/niutax:elegir-empresa` | Elige empresa o RUT y la reconoce en los cuatro módulos |
| `/niutax:conciliar-banco` | Carga cartola, propone imputaciones y concilia |
| `/niutax:clasificar-compras` | Clasifica libros de compras, ventas y honorarios |
| `/niutax:reportes-financieros` | Balance, EERR, flujo, indicadores, real vs presupuesto |
| `/niutax:mes-tributario` | F29 de un RUT o de toda la cartera |
| `/niutax:contingencias-sii` | Anotaciones, observaciones, deudas y DJ pendientes |
| `/niutax:emitir-documento` | Factura, boleta, nota de crédito/débito, guía |
| `/niutax:registrar-gasto` | Rinde boletas y facturas desde fotos |
| `/niutax:cierre-mensual` | Recorre el cierre del mes con todos los módulos |

## Principios

- **Nada se aplica sin aprobación humana.** Claude muestra lo que propone, espera un sí y deja la propuesta en el panel para que una persona la apruebe.
- **Solo cifras de Niutax.** Los reportes usan únicamente datos que devuelven los conectores.
- **La empresa se reconoce por RUT** entre módulos; los identificadores internos nunca se muestran.
