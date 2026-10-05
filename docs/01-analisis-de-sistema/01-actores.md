# Actores del sistema

Los actores del marketplace se dividen en usuarios y sistemas externos.
Cada uno participa en determinadas operaciones de la plataforma.

## Usuarios

| ID | Actor | ¿Qué necesita realizar? |
|---|---|---|
| ACT01 | Cliente | Buscar productos, consultar sus características y disponibilidad, gestionar su carrito, registrar la dirección de entrega, realizar pedidos, efectuar pagos y consultar el estado de sus compras. |
| ACT02 | Seller (vendedor) | Registrar y actualizar sus productos, consultar sus publicaciones y revisar los pedidos y las ventas que le corresponden. |
| ACT03 | Administrador | Registrar, actualizar y desactivar vendedores, gestionar el acceso de los usuarios y supervisar el funcionamiento de la plataforma. |

## Sistemas externos

| ID | Actor | ¿Cómo participa? |
|---|---|---|
| ACT04 | Pasarela de pago | Procesa las solicitudes de pago y comunica al marketplace si la operación fue aprobada o rechazada. |
| ACT05 | Servicio de envío | Recibe la información necesaria para gestionar la entrega y proporciona actualizaciones sobre el estado del envío. |
| ACT06 | Servicio de facturación | Genera los comprobantes de pago a partir de la información de las compras y los devuelve al marketplace. |
| ACT07 | ERP | Proporciona información de productos y stock para consultar y actualizar su disponibilidad en el marketplace. |

## Consideraciones

- Cada vendedor podrá gestionar únicamente sus propios productos y
  consultar la información de las ventas que le corresponden.
- Cada cliente podrá acceder únicamente a sus propios pedidos y datos
  personales.
- Los sistemas externos se consideran actores porque intercambian
  información con el marketplace.
- La base de datos forma parte de la solución propuesta, por lo que
  no se considera un actor externo.