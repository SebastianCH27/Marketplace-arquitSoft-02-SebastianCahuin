# Diseño interno de módulos

> **Proyecto:** Marketplace · **Backend:** Node.js + TypeScript · **Módulo de referencia:** Pedidos  
> **Enfoque arquitectónico:** Clean Architecture dentro de cada módulo del monolito modular  
> **Nivel C4:** 4 – Código · **Versión:** 1.0

---

## 1. Propósito

Este documento describe **cómo se organiza el código dentro de un módulo** del backend. Tiene tres objetivos:

- que todos los módulos se construyan con la misma estructura;
- que las reglas de negocio queden aisladas de la tecnología;
- que cualquier integrante del equipo sepa dónde va cada archivo y de qué puede depender.


## 2. Estructura de carpetas

```text
src/
└── modules/
    └── pedidos/
        ├── dominio/
        │   ├── pedido.ts                    Entidad Pedido y sus reglas
        │   ├── item-pedido.ts               Objeto de valor ItemPedido
        │   ├── estado-pedido.ts             Enumeración EstadoPedido
        │   ├── order-repository.ts          Interfaz (puerto) de persistencia
        │   ├── payment-port.ts              Interfaz (puerto) de cobro
        │   └── resultado-cobro.ts           DTO de respuesta del cobro
        ├── aplicacion/
        │   ├── crear-pedido.use-case.ts     Caso de uso CrearPedido
        │   └── crear-pedido.command.ts      Datos de entrada del caso de uso
        ├── infraestructura/
        │   ├── postgres-order-repository.ts Implementación con PostgreSQL
        │   ├── stripe-payment.adapter.ts    Implementación con Stripe
        │   └── paypal-payment.adapter.ts    Implementación con PayPal
        ├── presentacion/
        │   ├── order.controller.ts          Traduce HTTP ⇄ caso de uso
        │   └── pedidos.routes.ts            Rutas /api/v1/pedidos
        ├── pedidos.module.ts                Raíz de composición del módulo
        └── index.ts                         API pública del módulo
```

| Archivo | Capa | Responsabilidad | Importa de |
|---|---|---|---|
| `pedido.ts` | Dominio | Crear pedidos válidos, calcular el total, cambiar de estado | Solo dominio |
| `item-pedido.ts` | Dominio | Representar un producto, su cantidad y su subtotal | Nada |
| `estado-pedido.ts` | Dominio | Definir los estados posibles del pedido | Nada |
| `order-repository.ts` | Dominio | Declarar **qué** operaciones de persistencia necesita el módulo | `pedido.ts` |
| `payment-port.ts` | Dominio | Declarar **qué** necesita el módulo para cobrar | `resultado-cobro.ts` |
| `resultado-cobro.ts` | Dominio | Estructura del resultado de un cobro | Nada |
| `crear-pedido.use-case.ts` | Aplicación | Coordinar los pasos de la compra | Dominio |
| `crear-pedido.command.ts` | Aplicación | Datos de entrada validados del caso de uso | Nada |
| `postgres-order-repository.ts` | Infraestructura | Guardar y leer pedidos en PostgreSQL | Dominio + driver de BD |
| `stripe-payment.adapter.ts` | Infraestructura | Traducir `cobrar()` a la API de Stripe | Dominio + SDK de Stripe |
| `paypal-payment.adapter.ts` | Infraestructura | Traducir `cobrar()` a la API de PayPal | Dominio + SDK de PayPal |
| `order.controller.ts` | Presentación | Leer la petición HTTP, invocar el caso de uso y responder | Aplicación |
| `pedidos.routes.ts` | Presentación | Registrar las rutas en Express | Presentación |
| `pedidos.module.ts` | Composición | Crear las instancias concretas e inyectarlas | Todas las capas |
| `index.ts` | Composición | Exponer solo lo que otros módulos pueden usar | `pedidos.module.ts` |

---

## 3. Capas del módulo y regla de dependencia

![Diagrama de paquetes del módulo Pedidos](/img/paquetes-modulo-pedidos.png)

**Regla de dependencia:** el código solo puede importar hacia el centro (dominio). El dominio no conoce Express, PostgreSQL ni Stripe.

| Capa | Contiene | Responsabilidad | Puede depender de | **No** puede depender de |
|---|---|---|---|---|
| **Dominio** | Entidades, objetos de valor, enumeraciones, interfaces (puertos) | Reglas de negocio que serían ciertas con cualquier tecnología | Nada externo | Aplicación, infraestructura, presentación, Express, ORM, SDK |
| **Aplicación** | Casos de uso, comandos | Orquestar el flujo: qué paso va primero y qué hacer si falla | Dominio | Infraestructura, presentación, Express, ORM |
| **Infraestructura** | Repositorios concretos, adaptadores de servicios externos | Implementar las interfaces del dominio con una tecnología | Dominio, librerías externas | Aplicación, presentación |
| **Presentación** | Controladores, rutas | Traducir HTTP a llamadas del caso de uso y viceversa | Aplicación | Infraestructura, base de datos |
| **Composición** | `pedidos.module.ts` | Elegir implementaciones concretas y conectarlas | Todas | — |

> **Cómo comprobarlo en el código:** basta revisar los `import` de `dominio/`. Si aparece `express`, `pg`, `stripe` o cualquier archivo de `infraestructura/`, la regla se rompió.

---

## 4. Diagrama de clases

![Diagrama de clases del módulo Pedidos](/img/clases-modulo-pedidos.png)

### 4.1 Notación UML utilizada

| Símbolo | Significado | Ejemplo en el diagrama |
|---|---|---|
| Línea continua con flecha abierta | Asociación: la clase guarda una referencia a la otra | `CrearPedidoUseCase` → `OrderRepository` |
| Línea discontinua con flecha abierta | Dependencia: la usa o la crea, pero no la guarda | `OrderController` «crea» `CrearPedidoCommand` |
| Línea discontinua con triángulo hueco | Realización: implementa una interfaz | `PostgresOrderRepository` ▷ `OrderRepository` |
| Rombo relleno | Composición: la parte no existe sin el todo | `Pedido` ◆ `ItemPedido` (1..*) |
| Nombre en cursiva + «interface» | Interfaz (contrato sin implementación) | `OrderRepository`, `PaymentPort` |
| `-` / `+` | Privado / público | `- estado`, `+ total()` |


## 5. Flujo de ejecución: crear pedido

![Diagrama de secuencia de crear pedido](/img/secuencia-crear-pedido.png)

| Paso | Origen → destino | Acción | Capa |
|---|---|---|---|
| 1 | Cliente web → `OrderController` | `POST /api/v1/pedidos` con `clienteId`, `items` y `tokenPago` | Presentación |
| 2 | `OrderController` → `CrearPedidoUseCase` | `ejecutar(cmd)` | Presentación → Aplicación |
| 3 | `CrearPedidoUseCase` → `Pedido` | `crear(clienteId, items)`; se validan RN-01 a RN-03; estado PENDIENTE | Aplicación → Dominio |
| 4 | `CrearPedidoUseCase` → `Pedido` | `total()` (RN-04) | Aplicación → Dominio |
| 5 | `CrearPedidoUseCase` → `PaymentPort` | `cobrar(monto, tokenPago)` | Aplicación → Dominio (interfaz) |
| 6 | `StripePaymentAdapter` → pasarela | Solicitud de cobro por HTTPS | Infraestructura → externo |
| 7 | `CrearPedidoUseCase` → `Pedido` | Si fue aprobado: `marcarPagado(autorizacion)` (RN-05) | Aplicación → Dominio |
| 8 | `CrearPedidoUseCase` → `OrderRepository` | `guardar(pedido)` | Aplicación → Dominio (interfaz) |
| 9 | `PostgresOrderRepository` → PostgreSQL | `INSERT` del pedido y sus ítems | Infraestructura → BD |
| 10 | `OrderController` → Cliente web | `201 Created` con `pedidoId` | Presentación |
| 11–12 | Alternativa | Si el pago es rechazado: no se guarda el pedido y se responde `402` | Aplicación → Presentación |


## 6. Código de referencia

### 6.1 Dominio · entidad e interfaces

```ts
// dominio/pedido.ts
import { ItemPedido } from './item-pedido';
import { EstadoPedido } from './estado-pedido';
import { ReglaNegocioError } from '../../../shared/errores';

export class Pedido {
  private constructor(
    readonly id: string,
    readonly clienteId: string,
    private readonly items: ItemPedido[],
    private estado: EstadoPedido,
    private autorizacion?: string,
  ) {}

  static crear(clienteId: string, items: ItemPedido[]): Pedido {
    if (!clienteId) throw new ReglaNegocioError('RN-01: el pedido debe tener un cliente');
    if (items.length === 0) throw new ReglaNegocioError('RN-02: el pedido debe tener ítems');
    return new Pedido(crypto.randomUUID(), clienteId, items, EstadoPedido.PENDIENTE);
  }

  total(): number {                                            // RN-04
    return this.items.reduce((suma, item) => suma + item.subtotal(), 0);
  }

  marcarPagado(autorizacion: string): void {                   // RN-05
    if (this.estado !== EstadoPedido.PENDIENTE) {
      throw new ReglaNegocioError('RN-05: solo un pedido pendiente puede pagarse');
    }
    this.estado = EstadoPedido.PAGADO;
    this.autorizacion = autorizacion;
  }

  cancelar(): void {                                           // RN-06
    if (this.estado === EstadoPedido.DESPACHADO || this.estado === EstadoPedido.ENTREGADO) {
      throw new ReglaNegocioError('RN-06: el pedido ya fue despachado');
    }
    this.estado = EstadoPedido.CANCELADO;
  }
}
```

```ts
// dominio/order-repository.ts
import { Pedido } from './pedido';

export interface OrderRepository {
  guardar(pedido: Pedido): Promise<void>;
  buscarPorId(id: string): Promise<Pedido | null>;
}
```

```ts
// dominio/payment-port.ts
import { ResultadoCobro } from './resultado-cobro';

export interface PaymentPort {
  cobrar(monto: number, token: string): Promise<ResultadoCobro>;
}
```

### 6.2 Aplicación · caso de uso

```ts
// aplicacion/crear-pedido.use-case.ts
import { Pedido } from '../dominio/pedido';
import { ItemPedido } from '../dominio/item-pedido';
import { OrderRepository } from '../dominio/order-repository';
import { PaymentPort } from '../dominio/payment-port';
import { CrearPedidoCommand } from './crear-pedido.command';
import { PagoRechazadoError } from '../../../shared/errores';

export class CrearPedidoUseCase {
  constructor(
    private readonly orderRepository: OrderRepository,   // interfaz, no PostgreSQL
    private readonly paymentPort: PaymentPort,           // interfaz, no Stripe
  ) {}

  async ejecutar(cmd: CrearPedidoCommand): Promise<string> {
    const items = cmd.items.map((i) => new ItemPedido(i.productoId, i.cantidad, i.precioUnitario));
    const pedido = Pedido.crear(cmd.clienteId, items);

    const cobro = await this.paymentPort.cobrar(pedido.total(), cmd.tokenPago);
    if (!cobro.aprobado) throw new PagoRechazadoError(cobro.motivoRechazo);

    pedido.marcarPagado(cobro.autorizacion);
    await this.orderRepository.guardar(pedido);
    return pedido.id;
  }
}
```

> El precio unitario de cada ítem debe obtenerse del módulo **Catálogo** a través de su API pública, no del frontend. Se omite aquí para mantener el ejemplo corto.

### 6.3 Infraestructura · adaptador

```ts
// infraestructura/stripe-payment.adapter.ts
import Stripe from 'stripe';
import { PaymentPort } from '../dominio/payment-port';
import { ResultadoCobro } from '../dominio/resultado-cobro';

export class StripePaymentAdapter implements PaymentPort {
  constructor(private readonly cliente: Stripe) {}

  async cobrar(monto: number, token: string): Promise<ResultadoCobro> {
    // Traduce el lenguaje del proveedor al lenguaje del dominio.
    // La llamada exacta depende de la versión del SDK y del flujo de pago elegido.
    const pago = await this.cliente.paymentIntents.create({
      amount: Math.round(monto * 100),
      currency: 'pen',
      payment_method: token,
      confirm: true,
    });
    return {
      aprobado: pago.status === 'succeeded',
      autorizacion: pago.id,
      motivoRechazo: pago.status === 'succeeded' ? undefined : pago.status,
    };
  }
}
```

### 6.4 Presentación · controlador

```ts
// presentacion/order.controller.ts
import { Request, Response } from 'express';
import { CrearPedidoUseCase } from '../aplicacion/crear-pedido.use-case';

export class OrderController {
  constructor(private readonly crearPedido: CrearPedidoUseCase) {}

  crear = async (req: Request, res: Response): Promise<void> => {
    const pedidoId = await this.crearPedido.ejecutar({
      clienteId: req.user.id,            // lo agrega el middleware JWT; no viene del body
      items: req.body.items,
      tokenPago: req.body.tokenPago,
    });
    res.status(201).json({ pedidoId });
  };
}
```

### 6.5 Raíz de composición

```ts
// pedidos.module.ts
import Stripe from 'stripe';
import { Pool } from 'pg';
import { PostgresOrderRepository } from './infraestructura/postgres-order-repository';
import { StripePaymentAdapter } from './infraestructura/stripe-payment.adapter';
import { CrearPedidoUseCase } from './aplicacion/crear-pedido.use-case';
import { OrderController } from './presentacion/order.controller';

export function crearModuloPedidos(db: Pool, stripe: Stripe) {
  const orderRepository = new PostgresOrderRepository(db);
  const paymentPort = new StripePaymentAdapter(stripe);   // cambiar a PayPal = cambiar esta línea
  const crearPedido = new CrearPedidoUseCase(orderRepository, paymentPort);
  return { controller: new OrderController(crearPedido) };
}
```