# Restricciones del sistema

Las restricciones establecen condiciones que deben respetarse durante
el diseño y desarrollo del marketplace.

## Restricciones indicadas en la guía

| ID | Restricción | Descripción |
|---|---|---|
| RC01 | Aplicación web | El marketplace debe ser accesible mediante un navegador web. |
| RC02 | Control de versiones | Los archivos del proyecto deben gestionarse con Git y mantenerse en un repositorio en GitHub. |
| RC03 | API REST | La comunicación entre la aplicación web y el backend debe realizarse mediante una API REST. |
| RC04 | Pasarela de pago externa | Los pagos deben procesarse mediante una pasarela de pago externa integrada con el marketplace. |
| RC05 | Servicio de envío externo | El sistema debe integrarse con un servicio externo para gestionar la información de entrega de los pedidos. |
| RC06 | Arquitectura de tres capas | La propuesta inicial debe organizarse en presentación, lógica de negocio y datos. |

## Restricciones adicionales de la propuesta

Las siguientes condiciones se adoptan para mantener la coherencia
con el alcance definido en el análisis del negocio.

| ID | Restricción | Descripción |
|---|---|---|
| RC07 | Facturación externa | La generación de comprobantes se realizará mediante un servicio de facturación externo. El marketplace enviará los datos necesarios y conservará la referencia del comprobante recibido. |
| RC08 | ERP como fuente de stock | Para los productos vinculados al ERP, la disponibilidad se actualizará a partir de la información proporcionada por dicho sistema. |
| RC09 | Datos de tarjetas | El marketplace no almacenará números completos de tarjetas ni códigos de seguridad. El tratamiento de esos datos se delegará a la pasarela de pago. |

## Efecto sobre el diseño

- La aplicación web utilizará la API REST para solicitar las operaciones
  del sistema.
- Las responsabilidades se distribuirán entre las tres capas.
- Las integraciones deberán adaptarse a las interfaces y mecanismos
  de autenticación de cada proveedor.
- El sistema deberá manejar los errores y tiempos de espera de los
  servicios externos.
- La integración con el ERP deberá definir cómo se identifican
  los productos y cuándo se actualiza su disponibilidad.
- Git permitirá registrar los cambios y GitHub permitirá compartir
  y revisar el avance del proyecto.

## Tecnologías pendientes de selección

En esta etapa no se establece un lenguaje de programación, un framework
ni un motor de base de datos obligatorio.

Node.js y PostgreSQL aparecen como ejemplos en el marco conceptual
de la guía. Su uso no se asume como una exigencia del caso práctico.

La selección de estas tecnologías deberá justificarse cuando
corresponda desarrollar la solución.
