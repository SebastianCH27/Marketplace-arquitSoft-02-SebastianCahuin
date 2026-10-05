# ADR-004: Integrar pagos mediante interfaces y adaptadores

- Fecha: 2026-09-28
- Estado: Aceptada para la propuesta arquitectónica.
- Driver principal: DA04 — Integración con pagos externos.
- Drivers relacionados: DA03 — Seguridad y DA06 — Mantenibilidad.
- Requisitos relacionados: RF10 y RF11.
- Restricciones relacionadas: RC04 — Pasarela externa y RC09 — Datos de tarjetas.
- Decisiones relacionadas: ADR-001 y ADR-002.

## Contexto

El marketplace necesita procesar pagos mediante un proveedor externo
y conocer su resultado para actualizar los pedidos.

La integración debe permitir cambios de proveedor sin modificar
innecesariamente las reglas de compra. También debe contemplar
rechazos, interrupciones de comunicación y confirmaciones repetidas.

El proveedor concreto y sus condiciones de integración todavía
están pendientes de selección.

## Decisión

Los casos de uso dependerán de un contrato de pagos definido
en el núcleo. Un adaptador de Infraestructura implementará
ese contrato y resolverá la comunicación con la pasarela.

El contrato utilizará conceptos del marketplace, como referencia
del pedido, importe, moneda y estado del pago.

El adaptador traducirá esos datos al formato del proveedor
y convertirá sus respuestas al formato interno.

La configuración de la aplicación seleccionará el adaptador
correspondiente al entorno de pruebas o a la integración real.

## Distribución de responsabilidades

| Elemento | Responsabilidad |
|---|---|
| Módulo Pedidos | Coordinar la compra y utilizar el resultado verificado del pago para continuar su procesamiento. |
| Módulo Pagos | Gestionar intentos, referencias, estados y controles contra operaciones duplicadas. |
| Contrato de pagos | Definir las operaciones y los resultados que necesita la aplicación, incluyendo iniciar el pago y consultar su estado. |
| Adaptador de la pasarela | Implementar la comunicación, autenticación técnica y traducción de datos del proveedor. |
| Entrada de la API | Recibir las notificaciones externas y dirigirlas a su verificación y procesamiento. |
| Configuración | Conectar los casos de uso con las implementaciones seleccionadas. |

Las reglas para actualizar el pedido permanecerán en el núcleo.
Los detalles de la API externa se concentrarán en el adaptador.

## Flujo propuesto

1. El backend verifica al usuario, su autorización sobre la compra,
   los productos, las existencias y el importe vigente.

2. Se registra el pedido pendiente de pago y un intento identificable
   antes de solicitar la operación al proveedor.

3. El módulo Pagos utiliza el contrato para iniciar la operación
   mediante el adaptador.

4. La captura de los datos de tarjeta se delega a los mecanismos
   del proveedor. El marketplace utiliza las referencias o tokens
   necesarios para completar la integración.

5. El backend obtiene el resultado mediante una respuesta verificable,
   una consulta autenticada o una notificación del proveedor.

6. Se comprueban la referencia, el importe, la moneda y el estado
   antes de registrar el resultado y actualizar el pedido.

7. Cuando el pago esté confirmado, Pedidos continuará con las
   operaciones correspondientes de facturación y preparación
   del envío.

El retorno del navegador desde la pasarela se utilizará para
presentar el estado consultado al backend. Por sí solo no
demostrará que el pago fue aprobado.

## Duplicados y resultados inciertos

Cada intento tendrá una referencia persistente. Los reintentos
de la misma operación conservarán esa referencia y utilizarán
el mecanismo de idempotencia admitido por el proveedor.

Idempotencia significa que repetir una solicitud identificada
como la misma operación no debe producir un segundo cobro.

Las notificaciones recibidas se verificarán según el mecanismo
del proveedor. Se registrarán sus identificadores y se controlarán
las transiciones de estado para evitar procesar dos veces
el mismo resultado, incluso con varias instancias del backend.

Un tiempo de espera agotado no se interpretará automáticamente
como un pago rechazado. El intento quedará pendiente de verificación
y se consultará su resultado antes de permitir otro cobro.

Si el proveedor confirma el pago y ocurre un fallo al actualizar
los datos locales, se recuperará el estado utilizando la referencia
del intento. Esta recuperación deberá evitar repetir el cobro.

## Seguridad y conservación de información

- Las comunicaciones externas utilizarán HTTPS.
- Las credenciales secretas del proveedor permanecerán en el backend,
  fuera del código público y del repositorio.
- No se almacenarán números completos de tarjetas ni códigos
  de seguridad en bases de datos, registros o archivos.
- Se conservarán las referencias necesarias del pedido y la
  transacción, el importe, la moneda, las fechas y el estado.
- Los usuarios consultarán únicamente los pagos y pedidos
  para los que tengan autorización.

Un fallo posterior en facturación o envío se registrará por separado
y conservará el resultado del pago confirmado.

## Referencia observada en el boilerplate

El ejemplo incluye:

- `ProcesadorPagos`: contrato ubicado en Dominio.
- `RegistrarCompraCasoUso`: recibe ese contrato por constructor.
- `ProcesadorPagosSimulado`: implementación para pruebas.
- `ProcesadorPagosNiubiz`: ejemplo de adaptador HTTP que transforma
  una respuesta externa al formato interno.
- `app.config.ts`: selecciona la implementación utilizada.

La configuración ejecutada utiliza el procesador simulado.

El contrato del ejemplo devuelve un resultado simplificado
de aprobación o rechazo. Para la integración completa deberán
definirse también las referencias de los intentos, las consultas
de estado y el tratamiento de resultados pendientes.

El ejemplo permite estudiar la separación entre contrato
e implementación. La integración real deberá desarrollarse
y verificarse con la documentación del proveedor seleccionado.

## Alternativas consideradas

| Alternativa | Evaluación |
|---|---|
| Integrar la API del proveedor directamente en el caso de uso | Vincula la coordinación de la compra con formatos y detalles particulares de la pasarela. |
| Utilizar un contrato y un adaptador | Separa la necesidad de procesar pagos de la implementación del proveedor. Es la alternativa seleccionada. |

## Justificación

La decisión responde a DA04 porque proporciona un punto definido
para integrar la pasarela y tratar sus resultados.

También contribuye a DA06: sustituir un proveedor concentrará
los cambios en su adaptador y configuración, siempre que
el nuevo proveedor pueda cumplir el contrato.

Los controles de identidad, confirmación y duplicados
contribuyen a DA03.

## Consecuencias y condiciones

- Los casos de uso podrán probarse con adaptadores simulados.
- Los detalles de cada proveedor quedarán concentrados.
- Será necesario mantener contratos y pruebas de integración.
- Un cambio de proveedor puede exigir ajustes en el contrato
  si sus capacidades son diferentes.
- Deberán definirse la recuperación de operaciones pendientes
  y la coordinación entre pago, reserva de stock y pedido.
- Los tiempos y la disponibilidad de la pasarela seguirán
  siendo una dependencia externa.

## Verificación prevista

Se comprobarán los siguientes escenarios:

- Pago aprobado y pago rechazado.
- Repetición de la misma solicitud de pago.
- Notificaciones duplicadas o recibidas fuera de orden.
- Notificación con verificación inválida.
- Resultado con importe, moneda o referencia incorrectos.
- Interrupción de la comunicación durante el pago.
- Confirmación externa seguida de un fallo de persistencia local.
- Intento de consultar el pago de otro usuario.
- Fallo de facturación o envío después de confirmar el pago.

Las pruebas se realizarán primero con adaptadores simulados
y posteriormente en el entorno de pruebas del proveedor.

## Referencias técnicas

Las siguientes referencias ilustran mecanismos de integración.
La selección de la pasarela permanece pendiente.

- Stripe. Webhooks:
  https://docs.stripe.com/webhooks
- Stripe. Idempotent requests:
  https://docs.stripe.com/api/idempotent_requests