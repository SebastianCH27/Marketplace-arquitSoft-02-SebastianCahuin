# Patrones de diseño

## Objetivo

Documentar los patrones de diseño aplicados en el backend del Marketplace, indicando dónde se usan, qué problema resuelven y con qué tecnología se implementan, para que el equipo programe de forma uniforme.

## Contexto

- **Estilo:** monolito modular (Node.js + Express).
- **Enfoque interno:** Clean Architecture.
- **Base:** diagrama de componentes (C4 nivel 3).

## Patrones aplicados

| Patrón | Dónde se usa | Problema que resuelve | Se implementa con | Archivos en el código |
|---|---|---|---|---|
| Adapter | Pagos → Pasarela de pagos | La pasarela tiene su propia API | SDK o API REST de la pasarela | `payment-port.js`, `stripe-payment.adapter.js` |
| Adapter | Pedidos → ERP | El ERP tiene su propia API | API REST del ERP (axios o fetch) | `erp-port.js`, `erp.adapter.js` |
| Repository | Todos los módulos → Base de datos | El negocio no debe conocer SQL | PostgreSQL (librería `pg` u ORM) | `order-repository.js`, `postgres-order-repository.js` |
| Decorator | Catálogo → Caché | Consultar productos más rápido sin modificar el repositorio | Redis | `cached-product-repository.js` |

## Cómo funciona cada patrón

| Patrón | En pocas palabras |
|---|---|
| Adapter | Traduce: el sistema dice `cobrar(monto)` y el adaptador lo convierte en la llamada propia de la pasarela o del ERP. |
| Repository | El negocio dice `guardar(pedido)`; el repositorio escribe el SQL. |
| Decorator | Busca primero en Redis; si no está, consulta PostgreSQL y lo guarda en Redis para la próxima vez. |

## Beneficio

- Cambiar la pasarela, el ERP o la base de datos solo afecta al adaptador o al repositorio.
- La caché con Redis se puede activar o retirar sin tocar las reglas de negocio.

## Patrones previstos para siguientes iteraciones

| Patrón | Dónde se aplicaría | Cuándo |
|---|---|---|
| Observer | Pedidos → Notificaciones | Al implementar las notificaciones |
| Factory | Carrito → Pedidos | Si crear el pedido se vuelve complejo |
| Facade | Comunicación entre módulos | Cuando los módulos crezcan y necesiten una única puerta de entrada |