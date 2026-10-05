# Nivel 4 · Diagrama de código

> **Proyecto:** Marketplace · **Módulos detallados:** Pedidos y Pagos  
> **Modelo:** C4 · Nivel 4 de 4 · **Notación:** UML · **Versión:** 1.0

---

## 1. Objetivo

Mostrar **cómo está organizado el código** de un componente del nivel 3: sus clases, interfaces y la forma en que colaboran. Se detalla el flujo **Confirmar compra**, que involucra a los módulos **Pedidos** y **Pagos** y a la integración con el ERP.

| Aspecto | Descripción |
|---|---|
| Pregunta que responde | ¿Qué clases existen dentro de un módulo y cómo se relacionan? |
| Audiencia | Desarrolladores |
| Notación | UML (diagrama de clases y diagrama de secuencia). El modelo C4 recomienda UML para este nivel |
| Por qué solo dos módulos | El nivel 4 es opcional en C4 y se dibuja solo para los componentes más importantes o complejos. Pedidos y Pagos concentran las reglas de compra y las integraciones externas |
| Nivel anterior | `Nivel3-Diagrama-de-Componentes.md` |

---

## 2. Diagrama de clases

![Diagrama de clases de los módulos Pedidos y Pagos](/img/nivel4-clases.png)

> Archivo editable: `img/fuentes/nivel4-clases.svg`

### 2.1 Capas de Clean Architecture

| Capa | Contiene | Regla |
|---|---|---|
| **Presentación** | Controladores | Traduce HTTP a llamadas del caso de uso y viceversa |
| **Aplicación** | Casos de uso | Coordina los pasos del proceso; no conoce la tecnología |
| **Dominio** | Entidades, objetos de valor, interfaces | Contiene las reglas del negocio; no importa nada de las otras capas |
| **Infraestructura** | Repositorios y adaptadores | Implementa las interfaces del dominio con una tecnología concreta |

> **Regla de dependencia:** las flechas del código apuntan hacia el dominio. Las clases de infraestructura implementan interfaces del dominio (triángulo hueco); el dominio nunca conoce a la infraestructura.

### 2.2 Clases del módulo Pedidos

| Clase | Tipo | Capa | Responsabilidad |
|---|---|---|---|
| `OrderController` | Controlador | Presentación | Recibe `POST /api/v1/pedidos` y responde al cliente |
| `CrearPedidoUseCase` | Caso de uso | Aplicación | Crea el pedido, solicita el cobro, lo guarda y lo envía al ERP |
| `Pedido` | Entidad | Dominio | Valida el pedido, calcula el total y controla sus estados |
| `ItemPedido` | Objeto de valor | Dominio | Producto, cantidad, precio unitario y subtotal |
| `EstadoPedido` | Enumeración | Dominio | PENDIENTE, PAGADO, DESPACHADO, ENTREGADO, CANCELADO |
| `OrderRepository` | Interfaz | Dominio | Contrato para guardar y buscar pedidos |
| `ErpPort` | Interfaz | Dominio | Contrato para registrar el pedido en el ERP |
| `PostgresOrderRepository` | Repositorio | Infraestructura | Implementa `OrderRepository` con PostgreSQL |
| `ErpRestAdapter` | Adaptador | Infraestructura | Implementa `ErpPort` llamando a la API REST del ERP |

### 2.3 Clases del módulo Pagos

| Clase | Tipo | Capa | Responsabilidad |
|---|---|---|---|
| `ProcesarPagoUseCase` | Caso de uso (API pública) | Aplicación | Único punto por el que otros módulos solicitan un cobro |
| `PaymentPort` | Interfaz | Dominio | Contrato para cobrar un monto |
| `ResultadoCobro` | DTO | Dominio | Indica si el cobro fue aprobado, su autorización o el motivo del rechazo |
| `StripePaymentAdapter` | Adaptador | Infraestructura | Implementa `PaymentPort` con el SDK de la pasarela |

### 2.4 Notación UML utilizada

| Símbolo | Significado | Ejemplo |
|---|---|---|
| Línea continua con flecha abierta | Asociación: la clase tiene una referencia a la otra | `CrearPedidoUseCase` → `OrderRepository` |
| Línea discontinua con flecha abierta | Dependencia: la usa o la crea | `CrearPedidoUseCase` «crea» `Pedido` |
| Línea discontinua con triángulo hueco | Realización: implementa una interfaz | `PostgresOrderRepository` ▷ `OrderRepository` |
| Rombo relleno | Composición: la parte no existe sin el todo | `Pedido` ◆ `ItemPedido` |
| Nombre en cursiva con «interface» | Interfaz | `PaymentPort` |
| `-` y `+` | Atributo o método privado y público | `- estado`, `+ total()` |

---

## 3. Diagrama de secuencia: confirmar compra

![Diagrama de secuencia de confirmar compra](/img/nivel4-secuencia.png)

> Archivo editable: `img/fuentes/nivel4-secuencia.svg`

| Paso | Origen → destino | Acción |
|---|---|---|
| 1 | Cliente web → `OrderController` | Envía `POST /api/v1/pedidos` con los ítems, el token de pago y el JWT |
| 2 | `OrderController` → `CrearPedidoUseCase` | Ejecuta el caso de uso |
| 3–4 | `CrearPedidoUseCase` → `Pedido` | Crea el pedido (estado PENDIENTE) y calcula el total |
| 5 | `CrearPedidoUseCase` → `ProcesarPagoUseCase` | Solicita el cobro al módulo Pagos |
| 6–7 | `StripePaymentAdapter` → Pasarela de pago | Envía la solicitud de cobro por HTTPS |
| 8 | `CrearPedidoUseCase` → `Pedido` | Si el cobro fue aprobado, marca el pedido como PAGADO |
| 9 | `CrearPedidoUseCase` → `OrderRepository` | Guarda el pedido en PostgreSQL |
| 10 | `CrearPedidoUseCase` → `ErpPort` | Registra el pedido en el ERP |
| 11 | `OrderController` → Cliente web | Responde `201 Created` con el número de pedido |
| 12–13 | Alternativa | Si el pago es rechazado, el pedido no se guarda y se responde `402 Payment Required` |

---

## 4. Patrones y principios que se ven en este nivel

| Elemento del diagrama | Patrón o principio |
|---|---|
| `StripePaymentAdapter`, `ErpRestAdapter` | Patrón **Adapter** |
| `OrderRepository` y `PostgresOrderRepository` | Patrón **Repository** |
| El caso de uso recibe interfaces, no clases concretas | Principio de **inversión de dependencias** (SOLID) |
| Una clase por responsabilidad (controlador, caso de uso, entidad, repositorio) | Principio de **responsabilidad única** (SOLID) |
| Pedidos usa a Pagos solo mediante `ProcesarPagoUseCase` | Regla del **monolito modular**: comunicación por la API pública del módulo |

> Detalle completo en `03-diseño-de-software/diseño-interno/patrones-de-diseño.md` y `principios-de-diseño.md`.

---