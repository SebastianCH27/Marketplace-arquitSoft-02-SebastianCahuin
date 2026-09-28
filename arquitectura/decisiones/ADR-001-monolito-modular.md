# ADR-001: Organizar el backend como un monolito modular

- Fecha: 2026-09-28
- Estado: Aceptada para la propuesta arquitectónica.
- Drivers principales: DA01 — Escalabilidad y DA06 — Mantenibilidad.
- Atributos relacionados: AC03 — Escalabilidad y AC05 — Mantenibilidad.

## Contexto

El marketplace debe permitir que los clientes consulten productos,
gestionen su carrito y realicen compras. Los sellers administrarán
sus productos y consultarán sus ventas, mientras que el administrador
gestionará las funciones que correspondan a su rol.

El sistema también deberá integrarse con una pasarela de pago,
un ERP y servicios de envío y facturación.

La propuesta inicial separó presentación, lógica de negocio y datos.
Ahora es necesario definir cómo organizar las funcionalidades del
backend para facilitar los cambios y permitir el crecimiento de
la aplicación durante campañas comerciales.

El boilerplate de Angular se utiliza como referencia para estudiar
la separación de responsabilidades. La implementación del backend
completo forma parte del desarrollo posterior del proyecto.

## Decisión

Se organizará el backend como un monolito modular: una aplicación
desplegable que contendrá módulos con responsabilidades e interfaces
definidas.

La aplicación web se comunicará con este backend mediante una API REST,
de acuerdo con RC03.

Los módulos internos se comunicarán mediante interfaces y llamadas
dentro de la aplicación. Se evitará que un módulo modifique
directamente los datos internos de otro.

## Organización de los módulos

| Módulo | Responsabilidad principal |
|---|---|
| Usuarios | Gestionar identidad, autenticación, roles y permisos. |
| Sellers | Gestionar la información de los vendedores y coordinar sus consultas de ventas. |
| Catálogo | Gestionar productos, búsquedas y disponibilidad, incluyendo la integración con el ERP para los productos vinculados. |
| Carrito | Gestionar los productos seleccionados, sus cantidades y los cálculos previos a la compra. |
| Pedidos | Coordinar la compra y gestionar el pedido y su relación con pagos, facturación y envíos. |
| Pagos | Coordinar las operaciones de pago mediante un contrato y registrar sus resultados, verificando las confirmaciones recibidas. |

Se conservan los módulos de la propuesta inicial y se hace explícita
la responsabilidad de Pagos, que anteriormente se coordinaba
desde Pedidos.

Los detalles de comunicación con ERP, pasarela de pago, facturación
y envíos se encapsularán en adaptadores de infraestructura.
Estos proveedores seguirán siendo sistemas externos.

## Alternativas consideradas

| Alternativa | Evaluación |
|---|---|
| Monolito sin límites claros entre módulos | Permite un despliegue conjunto, pero facilita que las responsabilidades se mezclen y que los cambios afecten partes innecesarias del sistema. |
| Monolito modular | Mantiene un despliegue conjunto y establece límites internos para facilitar el mantenimiento y las pruebas. Es la alternativa seleccionada. |
| Microservicios | Permiten despliegue y escalamiento independiente por servicio, pero añaden comunicación por red, coordinación de datos y mayor complejidad operativa. En esta etapa no se ha justificado esa necesidad. |

## Justificación

La decisión responde a DA06 porque distribuye las responsabilidades
y permite modificar una funcionalidad dentro de su módulo, respetando
los contratos utilizados por los demás.

También responde a DA01 porque permite ampliar los recursos del
backend o ejecutar varias instancias de la misma aplicación cuando
las pruebas demuestren que es necesario.

La organización modular permite evolucionar el sistema manteniendo
una gestión conjunta de su construcción, pruebas y despliegue.

## Consecuencias favorables

- Responsabilidades identificables para cada módulo.
- Menor complejidad de despliegue que una solución distribuida
  en varios servicios.
- Posibilidad de probar reglas y casos de uso por separado,
  además de verificar su integración.
- Posibilidad de ejecutar varias instancias del backend.

## Compromisos y condiciones

- Los módulos se desplegarán conjuntamente.
- El escalamiento mediante réplicas abarcará todo el backend.
- Un fallo que afecte al proceso completo puede afectar a todos
  sus módulos.
- Los límites entre módulos deberán respetarse durante el desarrollo.
- Si existen varias instancias, las sesiones y el estado compartido
  deberán gestionarse de forma consistente.
- La base de datos y las integraciones externas también pueden
  limitar el rendimiento.

## Verificación prevista

Se revisarán las responsabilidades y dependencias de los módulos
para comprobar que utilizan las interfaces definidas.

Se realizarán pruebas de integración para verificar la coordinación
entre catálogo, carrito, pedidos y pagos.

La escalabilidad se evaluará con las metas propuestas en AC03:
pasar de 100 a 200 usuarios concurrentes, aumentando los recursos
o las instancias y manteniendo el objetivo de respuesta de AC01.

La capacidad alcanzada se documentará mediante pruebas de carga
sobre la implementación y la infraestructura utilizadas.