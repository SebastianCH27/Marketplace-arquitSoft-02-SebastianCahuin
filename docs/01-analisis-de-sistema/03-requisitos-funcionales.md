# Requisitos funcionales

Los requisitos funcionales describen las funciones que debe ofrecer
el marketplace. Cada requisito se relaciona con una o más historias
de usuario.

## Lista de requisitos funcionales

| ID | Requisito funcional | Historia relacionada |
|---|---|---|
| RF01 | El sistema debe permitir buscar productos por nombre y filtrar los resultados por categoría. | HU01 |
| RF02 | El sistema debe mostrar las características, el precio, el vendedor y la disponibilidad de cada producto. | HU01 |
| RF03 | El sistema debe permitir a cada seller registrar, actualizar y consultar sus propios productos. | HU02 |
| RF04 | El sistema debe permitir agregar productos al carrito, modificar sus cantidades y eliminarlos, mostrando el total actualizado. | HU03 |
| RF05 | El sistema debe permitir generar un pedido a partir del carrito, verificando la disponibilidad y registrando los productos, las cantidades, los precios y el cliente. | HU04 |
| RF06 | El sistema debe permitir al cliente consultar sus pedidos y el estado de cada uno. | HU06 |
| RF07 | El sistema debe permitir al administrador registrar, actualizar y desactivar sellers. | HU05 |
| RF08 | El sistema debe permitir al cliente consultar el detalle de sus pedidos, incluyendo productos, cantidades, importes y dirección de entrega. | HU04, HU06 |
| RF09 | El sistema debe permitir al cliente registrar y seleccionar la dirección de entrega antes de confirmar un pedido. | HU04 |
| RF10 | El sistema debe enviar la solicitud de pago de un pedido a una pasarela de pago externa. | HU07 |
| RF11 | El sistema debe registrar el resultado comunicado por la pasarela, actualizar el estado del pago y mostrarlo al cliente. | HU07 |
| RF12 | El sistema debe enviar al servicio de envío la información del pedido y la dirección de entrega cuando se cumplan las condiciones para su despacho. | HU08 |
| RF13 | El sistema debe recibir las actualizaciones del servicio de envío y permitir al cliente consultar el estado de la entrega. | HU08 |
| RF14 | El sistema debe solicitar al servicio de facturación la generación del comprobante correspondiente a una compra con pago aprobado. | HU09 |
| RF15 | El sistema debe asociar el comprobante recibido con la compra y permitir que el cliente lo consulte. | HU09 |
| RF16 | El sistema debe permitir a cada seller consultar únicamente los productos, cantidades e importes que le corresponden dentro de los pedidos y ventas. | HU10 |
| RF17 | El sistema debe permitir el registro de clientes, verificando que el correo electrónico no esté asociado a otra cuenta. | HU11 |
| RF18 | El sistema debe permitir a los usuarios iniciar sesión con sus credenciales y cerrar su sesión. | HU11, HU12 |
| RF19 | El sistema debe permitir al administrador consultar, activar y desactivar cuentas de usuario y asignarles los roles definidos. | HU13 |
| RF20 | El sistema debe controlar el acceso a las funciones según el rol del usuario y comprobar que los datos solicitados le correspondan. | HU11, HU12, HU13 |
| RF21 | El sistema debe consultar la información de productos y stock del ERP y actualizar la disponibilidad de los productos vinculados con dicho sistema. | HU14 |

## Relación entre historias de usuario y requisitos

| Historia de usuario | Requisitos relacionados |
|---|---|
| HU01: Buscar y consultar productos | RF01, RF02 |
| HU02: Gestionar productos propios | RF03 |
| HU03: Gestionar el carrito | RF04 |
| HU04: Registrar dirección y generar pedido | RF05, RF08, RF09 |
| HU05: Administrar sellers | RF07 |
| HU06: Consultar pedidos | RF06, RF08 |
| HU07: Pagar un pedido y conocer el resultado | RF10, RF11 |
| HU08: Consultar el estado del envío | RF12, RF13 |
| HU09: Consultar el comprobante de compra | RF14, RF15 |
| HU10: Consultar pedidos y ventas del seller | RF16 |
| HU11: Crear una cuenta e iniciar sesión | RF17, RF18, RF20 |
| HU12: Acceder como seller o administrador | RF18, RF20 |
| HU13: Gestionar cuentas y roles | RF19, RF20 |
| HU14: Consultar disponibilidad actualizada desde el ERP | RF21 |

## Reglas consideradas para el diseño inicial

- Las cantidades solicitadas deben ser mayores que cero y no superar
  el stock disponible al confirmar el pedido.
- Los precios y totales del pedido deben calcularse y validarse
  en el servidor.
- El pago solo se considera aprobado cuando la pasarela lo confirma.
- Un pedido podrá solicitar su despacho cuando el pago esté aprobado
  y los productos estén listos para su envío.
- Un cliente solo podrá consultar sus propios pedidos y comprobantes.
- Un seller solo podrá gestionar sus productos y consultar la parte
  de los pedidos que le corresponda.
- Un seller desactivado no podrá publicar ni actualizar su oferta.
- Para los productos vinculados al ERP, se tomará dicho sistema
  como fuente de información del stock.