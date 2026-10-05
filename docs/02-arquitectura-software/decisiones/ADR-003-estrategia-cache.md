# ADR-003: Incorporar caché para consultas del catálogo

- Fecha: 2026-09-28
- Estado: Aceptada para la propuesta arquitectónica.
- Driver principal: DA02 — Rendimiento.
- Drivers relacionados: DA01 — Escalabilidad y DA09 — Stock del ERP.
- Atributo relacionado: AC01 — Rendimiento.
- Decisiones relacionadas: ADR-001 y ADR-002.

## Contexto

Durante las campañas comerciales, varios clientes pueden consultar
repetidamente los mismos productos. Resolver cada consulta completa
desde la base de datos puede aumentar la carga y el tiempo de respuesta.

El catálogo contiene información pública que puede reutilizarse.
Sin embargo, los precios, la disponibilidad y las condiciones
de venta pueden cambiar, por lo que deben existir controles
para evitar compras basadas en información desactualizada.

## Decisión

Se incorporará una caché de consultas públicas del catálogo
mediante la estrategia cache-aside, con carga bajo demanda.

La caché será una copia temporal y prescindible. Los datos
persistentes conservarán su función como fuente del sistema.

Cuando el backend funcione con varias instancias, estas utilizarán
una caché compartida.

La tecnología concreta se seleccionará posteriormente.
Esta decisión define la estrategia; su implementación queda
pendiente.

## Información incluida y excluida

| Información | Tratamiento propuesto |
|---|---|
| Datos públicos de productos | Almacenar temporalmente nombres, descripciones, categorías y referencias a imágenes. |
| Fichas y listados públicos | Almacenar resultados frecuentes y paginados, identificando sus filtros y ordenamiento. |
| Precios mostrados en el catálogo | Podrán incluirse en la copia temporal; deberán comprobarse nuevamente al preparar y confirmar la compra. |
| Existencias | Obtenerlas mediante el mecanismo de disponibilidad del catálogo; quedan fuera de esta caché inicial. |
| Carritos, pedidos y pagos | Quedan fuera del alcance de esta caché de consultas públicas. |
| Credenciales, permisos y datos privados | No se almacenarán en esta caché. |

Los listados almacenados contendrán únicamente los campos públicos
permitidos por esta decisión.

## Funcionamiento de las consultas

1. El componente de consulta busca una entrada vigente en la caché.
2. Si la encuentra, utiliza la información almacenada.
3. Si no existe o ha vencido, consulta los datos persistentes.
4. Guarda temporalmente el resultado y lo devuelve.

Las claves distinguirán el producto o los parámetros de consulta:
filtros, vendedor cuando corresponda, ordenamiento, página
y cantidad de resultados.

## Actualización e invalidación

Se propone un tiempo de vida inicial de 60 segundos para las entradas.
Este valor es configurable y deberá evaluarse con las pruebas.
La caducidad se contará desde la carga de la entrada.

Cuando se modifique un producto, su precio o su publicación,
primero se confirmará el cambio en los datos persistentes.
Después se invalidarán su ficha y los listados afectados.

La misma regla se aplicará cuando una sincronización externa
modifique campos incluidos en la caché.

Esta estrategia admite breves periodos con información anterior.
El tiempo de vida y la invalidación reducen ese riesgo, pero
no sustituyen las comprobaciones necesarias para comprar.

## Protección de la compra

Al preparar o confirmar una compra, el backend comprobará
el precio vigente, la publicación del producto y la disponibilidad
utilizando los datos y mecanismos autorizados para esas operaciones.

Para los productos vinculados al ERP se respetará RC08.
El mecanismo de reserva o descuento de existencias deberá
definirse para controlar las compras simultáneas.

Si cambia el importe presentado al cliente, se mostrará el nuevo
total y se solicitará su confirmación antes de realizar el cobro.

La autorización de la compra no utilizará la copia del catálogo
como fuente definitiva de precios o existencias.

## Ubicación arquitectónica

La implementación de la caché pertenecerá a Infraestructura
y se conectará mediante el contrato de consulta correspondiente.

El dominio conservará sus reglas sin depender de una tecnología
de caché. Las operaciones de compra dispondrán de acceso
a las fuentes necesarias para sus verificaciones.

## Alternativas consideradas

| Alternativa | Evaluación |
|---|---|
| Consultar siempre los datos persistentes | Simplifica la actualización de las lecturas, pero mantiene el costo de las consultas repetidas. |
| Precargar todo el catálogo | Puede acelerar las primeras consultas, pero consume recursos incluso para productos poco consultados. |
| Cargar bajo demanda mediante cache-aside | Permite reutilizar los resultados solicitados y controlar su caducidad. Es la alternativa seleccionada. |

## Consecuencias y condiciones

- Se espera reducir las consultas repetidas del catálogo.
- Se necesitarán recursos para almacenar y supervisar la caché.
- Deberán definirse límites de tamaño y eliminación de entradas.
- La invalidación añadirá trabajo a las modificaciones de productos.
- Una caída de la caché aumentará la carga sobre la base de datos.
- Su beneficio dependerá de la frecuencia de reutilización
  y de los resultados de las pruebas.

Ante un fallo de caché se intentará consultar la fuente persistente
con tiempos de espera y concurrencia limitados. Si esa fuente
tampoco responde, se informará el fallo de la consulta.

## Verificación prevista

Se comparará el comportamiento del catálogo sin caché,
con caché vacía y con entradas previamente cargadas.

Se medirán tiempos de respuesta, consultas a la base de datos,
proporción de consultas atendidas desde caché y consumo de recursos.

Se comprobarán la caducidad y la invalidación después de modificar
productos, precios o publicaciones.

También se verificará que una copia anterior del catálogo
no permita confirmar precios desactualizados ni comprar
productos sin disponibilidad.

Las pruebas incluirán fallos de caché y la meta propuesta en AC01:
con 100 usuarios concurrentes, al menos el 95 % de las consultas
de productos y operaciones del carrito responderá en un máximo
de 2 segundos.

El resultado deberá evaluarse sobre el sistema completo.
El cumplimiento de esta meta está pendiente de implementación
y pruebas.

## Referencias

- Microsoft Learn. Cache-Aside pattern:
  https://learn.microsoft.com/en-us/azure/architecture/patterns/cache-aside
- Microsoft Learn. Caching guidance:
  https://learn.microsoft.com/en-us/azure/architecture/best-practices/caching