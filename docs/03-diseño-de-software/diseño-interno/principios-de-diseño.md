# Principios de diseño (SOLID)

## Objetivo

Documentar cómo se aplican los principios SOLID en el backend del Marketplace, organizado con Clean Architecture, para que cada módulo se programe de forma uniforme y las reglas de negocio no dependan de la tecnología.

## Contexto

- **Estilo:** monolito modular (Node.js + Express).
- **Enfoque interno:** Clean Architecture (presentación, aplicación, dominio, infraestructura).
- **Módulo de referencia:** Pedidos, porque integra base de datos, pasarela de pagos y ERP.

## SOLID en el módulo Pedidos

![SOLID en el módulo Pedidos](/img/solid-modulo-pedidos.png)

| Principio | Dónde se aplica (capa) | Cómo se cumple en el Marketplace | Archivos |
|---|---|---|---|
| **S** · Responsabilidad única | Todas las capas | Cada clase tiene una sola tarea: el controlador recibe la petición HTTP, el caso de uso coordina la compra, la entidad aplica las reglas del pedido y el repositorio guarda en PostgreSQL | `order.controller.js`, `crear-pedido.use-case.js`, `pedido.js`, `postgres-order-repository.js` |
| **O** · Abierto/cerrado | Infraestructura | Para agregar una nueva pasarela (PayPal) se crea un nuevo adaptador; el caso de uso de compra no se modifica | `stripe-payment.adapter.js`, `paypal-payment.adapter.js` |
| **L** · Sustitución de Liskov | Infraestructura | Stripe y PayPal devuelven el mismo resultado `{ aprobado, autorizacion }`; el caso de uso funciona igual con cualquiera | `stripe-payment.adapter.js`, `paypal-payment.adapter.js` |
| **I** · Segregación de interfaces | Dominio | Cada interfaz tiene solo lo que Pedidos necesita: `PaymentPort` solo cobra, `ErpPort` solo registra el pedido, `OrderRepository` solo guarda y busca pedidos | `payment-port.js`, `erp-port.js`, `order-repository.js` |
| **D** · Inversión de dependencias | Aplicación | El caso de uso recibe interfaces por el constructor y no conoce Stripe, PostgreSQL ni el ERP; las implementaciones se conectan en `pedidos.module.js` | `crear-pedido.use-case.js`, `pedidos.module.js` |

## Qué pasaría sin SOLID en el Marketplace

| Situación real | Sin SOLID | Con SOLID |
|---|---|---|
| Se cambia Stripe por otra pasarela | Hay que modificar el caso de uso de compra | Solo se crea un nuevo adaptador |
| Se cambia el ERP | Se modifica el código del módulo Pedidos | Solo se reemplaza `ErpRestAdapter` |
| Se quiere probar la compra | Se necesita PostgreSQL y la pasarela real | Se prueba con implementaciones en memoria |
| Cambia una regla del pedido | Hay que buscarla en controladores y servicios | Está solo en la entidad `Pedido` |

## Relación con Clean Architecture y los patrones

| Principio | Lo cumple en el proyecto |
|---|---|
| Responsabilidad única | La separación en capas de Clean Architecture |
| Abierto/cerrado y Liskov | El patrón Adapter (pasarela de pagos y ERP) |
| Segregación de interfaces | Los puertos del dominio (`PaymentPort`, `ErpPort`, `OrderRepository`) |
| Inversión de dependencias | El patrón Repository y la regla de dependencia: las capas externas dependen del dominio |