# Drivers arquitectónicos

Los drivers arquitectónicos son los requisitos, atributos de calidad
y restricciones que influyen de manera importante en el diseño
del marketplace.

## Drivers identificados

| ID | Driver arquitectónico | Origen | ¿Cómo influye en la arquitectura? |
|---|---|---|---|
| DA01 | Soportar el crecimiento de usuarios durante campañas comerciales. | AC03: Escalabilidad | Requiere considerar la ampliación de recursos y la posibilidad de ejecutar varias instancias del backend. Si se usan sesiones, su manejo no debe depender exclusivamente de la memoria de una instancia. |
| DA02 | Mantener tiempos de respuesta adecuados en consultas y operaciones del carrito. | AC01: Rendimiento; RF01, RF02, RF04 | Orienta el diseño de consultas, índices y paginación del catálogo, además de evitar procesamiento innecesario en cada solicitud. |
| DA03 | Proteger la información y limitar las operaciones según el usuario y su rol. | AC04: Seguridad; RF18, RF19, RF20 | Requiere autenticación y autorización en el backend, incluyendo la comprobación de que cada cliente o seller acceda únicamente a los datos que le corresponden. |
| DA04 | Procesar pagos mediante un proveedor externo sin almacenar datos completos de tarjetas. | RF10, RF11; RC04, RC09 | Requiere separar la integración con la pasarela de las reglas de pedidos, verificar sus confirmaciones y evitar registrar dos veces un mismo resultado de pago. |
| DA05 | Comunicar la aplicación web con el backend mediante una API REST. | RC03: API REST | Define una interfaz de entrada con rutas, solicitudes y respuestas. La interfaz delegará las operaciones a la lógica de negocio. |
| DA06 | Mantener una organización clara y facilitar los cambios. | AC05: Mantenibilidad; RC06: Tres capas | Determina la separación entre presentación, negocio y datos, así como la distribución de responsabilidades entre los módulos. |
| DA07 | Mantener el servicio disponible y controlar los fallos externos. | AC02: Disponibilidad; RC04, RC05, RC07, RC08 | Requiere tiempos de espera limitados, registro de errores y recuperación de operaciones pendientes. Un fallo de facturación o envío no debería impedir consultar el catálogo. |
| DA08 | Coordinar pedidos con los servicios de envío y facturación. | RF12, RF13, RF14, RF15; RC05, RC07 | Requiere componentes de integración separados y estados que permitan distinguir un pago aprobado, una facturación pendiente y un envío en preparación. |
| DA09 | Utilizar la información de stock del ERP para los productos vinculados. | RF05, RF21; RC08 | Requiere identificar los productos entre ambos sistemas, definir la actualización del stock y validar la disponibilidad al confirmar el pedido. |
| DA10 | Separar la información de los distintos vendedores. | RF03, RF16, RF20; AC04 | Influye en el modelo de datos: cada producto debe identificar a su seller y cada detalle de pedido debe permitir reconocer al vendedor correspondiente. |

## Decisiones iniciales derivadas

- Organizar la solución en tres capas: presentación, lógica de negocio
  y datos.
- Utilizar la API REST como punto de entrada a las funciones del backend.
- Distribuir la lógica de negocio en los módulos de Usuarios, Sellers,
  Catálogo, Carrito y Pedidos.
- Centralizar la autenticación y aplicar controles de autorización
  en cada operación protegida.
- Encapsular la comunicación con proveedores externos en componentes
  de integración utilizados por la lógica de negocio.
- Acceder a la base de datos mediante componentes de acceso a datos,
  evitando consultas directas desde la aplicación web.
- Registrar por separado los estados de pago, facturación y envío
  asociados a un pedido.
- Mantener la asociación entre vendedores, productos y detalles
  de los pedidos.

## Aspectos pendientes de precisar

- Los proveedores de pago, envío y facturación, junto con sus interfaces.
- La frecuencia y el mecanismo de actualización de información del ERP.
- El mecanismo para reservar o descontar stock y evitar que compras
  simultáneas utilicen las mismas existencias.
- La infraestructura necesaria para evaluar las metas de rendimiento,
  disponibilidad y escalabilidad.

## Consideración

Estos drivers justifican la propuesta inicial de arquitectura.
Su identificación no demuestra que las metas de calidad ya se cumplan;
eso deberá verificarse durante la implementación y las pruebas.