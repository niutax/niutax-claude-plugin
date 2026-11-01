---
name: mes-tributario
description: Revisa el F29 y los pasos del mes tributario en Niutax Tributario, para un RUT o para toda la cartera de un estudio contable — estado en el SII, monto a pagar, libros, observaciones y vencimiento. Usar cuando el usuario pregunta "cómo va el F29", "qué clientes faltan por declarar", "revisa el IVA del mes", "F29 de [cliente]", "qué vence este mes", "por qué subió el IVA".
---

# Mes tributario (F29)

Herramientas de Niutax Tributario (solo lectura): `resumen-de-hoy`, `estado-f29-cartera`, `f29-del-rut`, `observaciones-f29`, `libros-del-mes`, `documentos-del-libro`, `acuses-de-recibo`, `vencimientos-tributarios`, `diagnostico-rut`, `indicadores-economicos`.

## Reglas

- Este módulo **no declara, no paga, no acepta ni reclama**: eso se hace en el panel de Niutax. No lo ofrezcas como algo que puedas hacer; entrega el enlace del panel que traen las respuestas.
- El monto es el del F29 armado en Niutax, no necesariamente lo declarado: dilo así.
- Un dato con `consultado_el` nulo o `sin_sincronizar` no es cero: dilo.
- Con personas naturales los montos vienen ocultos a propósito: no intentes deducirlos.
- Si hay `omitidos`, dilo o pide la página siguiente.

## Cartera (estudio contable)

1. Llama `resumen-de-hoy` y `estado-f29-cartera` del último mes cerrado (o del que pidan).
2. Presenta una tabla corta por grupo de estado: sin declarar, observados, con giro, con alertas, y el paso pendiente de cada RUT (libros, remuneraciones, F29, SII, aviso). Muestra los días que faltan para el vencimiento.
3. Ordena por urgencia: observados y vencidos primero.
4. Pregunta por cuál RUT seguir.

## Un RUT

1. `f29-del-rut` con el mes, `con_codigos` y `comparar_con_anterior`.
2. Si el F29 quedó **desactualizado frente a los libros**, dilo primero y revisa `libros-del-mes`.
3. Explica las variaciones grandes contra el mes anterior (débito, crédito, PPM, retenciones) y, si hace falta, baja a `documentos-del-libro`.
4. Si hay facturas por aceptar o reclamar, nómbralas desde `acuses-de-recibo` con su plazo.
5. Si está observado, trae `observaciones-f29` y explica qué pide el SII.
6. Si algo no avanza (no llegan libros, no se arma el F29), usa `diagnostico-rut` antes de suponer la causa.

Cierra con el paso que sigue y el enlace al panel donde se hace.

## Cuando otra skill carga esta

Vuelve a la skill que la llamó con: RUT, mes, estado en el SII, monto a pagar y paso pendiente.
