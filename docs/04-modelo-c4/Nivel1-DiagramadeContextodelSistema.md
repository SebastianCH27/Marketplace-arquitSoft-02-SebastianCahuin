# Nivel 1 · Diagrama de contexto del sistema

> **Proyecto:** Marketplace  
> **Modelo:** C4 · Nivel 1 de 4 · **Versión:** 1.0

---

## 1. Objetivo

Mostrar el **Marketplace como una sola caja**: quiénes lo usan y con qué sistemas externos se comunica. Es la vista de mayor nivel y la primera que se presenta a cualquier persona, técnica o no.

| Aspecto | Descripción |
|---|---|
| Pregunta que responde | ¿Qué es el sistema, quién lo usa y con quién se integra? |
| Audiencia | Todos: negocio, docentes, equipo de desarrollo, operaciones |
| Qué **no** muestra | Tecnologías, servidores, bases de datos ni módulos internos |
| Siguiente nivel | `Nivel2-DiagramadeContenedores.md` |

---

## 2. Diagrama

![Diagrama de contexto del Marketplace](/img/nivel1-contexto.png)

> Archivo editable: `img/fuentes/nivel1-contexto.drawio`

---

## 3. Personas (usuarios)

| Persona | Descripción | Qué hace en el sistema |
|---|---|---|
| **Cliente** | Persona que compra productos | Busca productos, los agrega al carrito, confirma la compra, paga y consulta el estado de sus pedidos |
| **Seller** | Vendedor que ofrece productos en el marketplace | Publica productos, actualiza precios y stock, revisa los pedidos de sus productos |
| **Administrador** | Responsable de la operación del marketplace | Gestiona usuarios, sellers, productos y pedidos |

---

## 4. Sistemas

| Sistema | Tipo | Responsabilidad | Quién lo construye |
|---|---|---|---|
| **Marketplace** | Sistema en alcance | Permite a los sellers publicar productos y a los clientes comprarlos, pagarlos y recibirlos | El equipo del proyecto |
| **Pasarela de pago** | Sistema externo | Procesa los pagos con tarjeta y otros medios | Proveedor de pagos |
| **ERP** | Sistema externo | Gestión interna de la empresa (ventas, inventario, contabilidad) | Sistema existente de la empresa |
| **Servicio de envío** | Sistema externo | Gestiona las entregas de los pedidos | Empresa de courier |

---

## 5. Relaciones

| Origen | Destino | Descripción | Protocolo |
|---|---|---|---|
| Cliente | Marketplace | Busca productos, compra y paga sus pedidos | — |
| Seller | Marketplace | Publica y gestiona sus productos y stock | — |
| Administrador | Marketplace | Gestiona usuarios, productos y pedidos | — |
| Marketplace | Pasarela de pago | Solicita cobros | HTTPS/REST |
| Marketplace | ERP | Sincroniza pedidos | HTTPS/REST |
| Marketplace | Servicio de envío | Solicita y consulta envíos | HTTPS/REST |

> En el nivel 1 no se indica el protocolo entre personas y sistema: eso se detalla en el nivel 2.

---

## 6. Decisiones y supuestos

| # | Decisión o supuesto | Motivo |
|---|---|---|
| 1 | Los tres sistemas externos se integran mediante sus API REST | Es el mecanismo que ofrecen los proveedores y no requiere acceso a sus bases de datos |
| 2 | El Marketplace **no almacena datos de tarjetas** | Los datos de pago los procesa la pasarela; se reduce el riesgo de seguridad |
| 3 | Los sistemas externos no se modifican | Son de terceros; el Marketplace se adapta a ellos (patrón Adapter) |

### Pendientes por definir

| Tema | Detalle |
|---|---|
| Canal de notificaciones | El módulo Notificaciones avisa al cliente el estado del pedido; falta definir si será correo, SMS u otro servicio. Cuando se defina, se agregará como sistema externo |
| Sentido de la integración con el ERP | Confirmar con el negocio qué datos se envían y si el ERP también envía información al Marketplace |

---