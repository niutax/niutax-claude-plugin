---
name: elegir-empresa
description: Elige la empresa o el RUT de trabajo y la reconoce en los cuatro módulos de Niutax (Finanzas, Tributario, Facturación y Gastos). Usar cuando el usuario dice "cambia de empresa", "trabajemos con [cliente]", "¿qué empresas tengo?", "elige empresa", o cuando otra skill de Niutax necesita saber sobre qué empresa operar.
---

# Elegir la empresa de trabajo

Niutax es un ecosistema de módulos separados. Cada conector tiene su propia lista de empresas y su propio identificador interno, así que la misma empresa se reconoce **por su RUT**, nunca por el id.

| Módulo | Cómo lista empresas | Identificador que pide |
|---|---|---|
| Finanzas | `listar-mis-empresas` (acepta `busqueda`, `holding`) | `empresa_id` |
| Tributario | `listar-mi-cartera` (acepta `buscar`, `grupo`) | `rut` |
| Facturación | `listar-mis-empresas` (acepta `busqueda`) | `empresa_id` |
| Gastos | `listar_empresas_autorizadas` | `companyId` |

## Pasos

1. Si el usuario ya nombró una empresa o RUT, búscala con `busqueda`/`buscar` en el módulo que la tarea necesita. Si no, lista las empresas de Finanzas (o la cartera de Tributario si la tarea es tributaria) y pregunta cuál usar. Termina el turno.
2. Con la empresa elegida, guarda en la conversación su **nombre y RUT**. Cuando otra skill necesite otro módulo, busca ese mismo RUT en la lista de ese módulo para obtener su identificador.
3. Si el RUT no aparece en un módulo, dilo así: "Esta empresa no está habilitada en [módulo] para tu usuario" y sigue con los módulos donde sí está. No supongas que el id de un módulo sirve en otro.

## Reglas

- No muestres nunca `empresa_id` ni `companyId` al usuario: nombra la empresa por su razón social o RUT.
- Si hay más de 100 empresas o páginas omitidas, acota con la búsqueda en vez de listar todo.
- Cuando otra skill cargó esta, vuelve a esa skill en la misma respuesta con la empresa elegida, sin anunciar el traspaso.
