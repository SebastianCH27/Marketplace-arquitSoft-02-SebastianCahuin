# Escenarios de atributos de calidad del Marketplace

Este documento desarrolla los atributos definidos en
[04-atributos-de-calidad.md](04-atributos-de-calidad.md).

Cada escenario identifica seis elementos: fuente del estímulo,
estímulo, artefacto, entorno, respuesta y medida de la respuesta.
También indica cómo se verificará y su relación con los requisitos
y drivers del proyecto.

Las cifras son metas propuestas. Su cumplimiento deberá
comprobarse mediante pruebas sobre la implementación.

## EQ-01. Rendimiento: catálogo y carrito

| Elemento | Descripción |
|---|---|
| Atributo de calidad | AC01: Rendimiento. |
| Fuente del estímulo | Clientes que utilizan la plataforma durante una campaña comercial. |
| Estímulo | Los clientes buscan productos, consultan sus detalles y agregan, modifican o eliminan productos de sus carritos simultáneamente. |
| Artefacto | API REST, módulos Catálogo y Carrito y sus componentes de acceso a datos. |
| Entorno | Entorno de prueba representativo, con 100 usuarios concurrentes activos, cuentas y carritos separados y solicitudes válidas. Se propone mantener esta carga durante 30 minutos después del calentamiento. |
| Respuesta | El sistema devuelve los productos solicitados y actualiza correctamente cada carrito y su total, respetando la identidad del cliente. |
| Medida de la respuesta | Al menos el 95 % de las solicitudes de cada operación evaluada responde en 2 segundos o menos. Como criterio adicional propuesto, menos del 1 % de las solicitudes presenta errores técnicos inesperados. Los carritos y totales comprobados deben ser correctos. |
| Verificación | Ejecutar una prueba de carga y registrar tiempos, errores y resultados por operación. Documentar infraestructura, volumen de datos, frecuencia de acciones y configuración de caché. Medir desde el envío de la solicitud HTTP hasta la recepción completa de su respuesta. |
| Trazabilidad | RF01: búsqueda de productos; RF02: detalle del producto; RF04: gestión del carrito; DA02: rendimiento; DA05: comunicación mediante API REST. |
| Estado | Meta propuesta. Pendiente de revisar el protocolo y ejecutar la prueba sobre el backend implementado. |

### Consideraciones de la evaluación

- Los 100 usuarios concurrentes representan sesiones activas que
  realizan acciones con pausas documentadas. No equivalen a
  100 solicitudes por segundo.
- El criterio de 2 segundos se comprobará por separado para
  cada operación, evitando que las consultas rápidas oculten
  operaciones lentas del carrito.
- Los errores técnicos incluirán respuestas 5xx, tiempos de espera
  agotados y respuestas con resultados incorrectos.
- Los rechazos de negocio previstos se registrarán por separado.
- La duración de 30 minutos y el límite de errores inferior al 1 %
  son propuestas añadidas para precisar la evaluación.
- El tiempo de aprobación de pagos externos se evaluará
  por separado.

  ## EQ-02. Disponibilidad: continuidad de las operaciones críticas

| Elemento | Descripción |
|---|---|
| Atributo de calidad | AC02: Disponibilidad. |
| Fuente del estímulo | Clientes que consultan productos, gestionan sus carritos o generan pedidos. |
| Estímulo | Los clientes solicitan una operación crítica mientras el servicio está publicado; también puede producirse un fallo en un componente o proveedor externo. |
| Artefacto | Aplicación web, API REST, módulos Catálogo, Carrito y Pedidos, almacenamiento e integraciones necesarias para cada operación. |
| Entorno | Sistema desplegado con monitoreo durante un mes calendario. Se propone un horario de servicio continuo, sujeto a confirmación del negocio. |
| Respuesta | El sistema mantiene operativas las funciones que no dependen del componente averiado. Cuando una operación no puede completarse, informa del problema sin confirmar pagos o pedidos incorrectamente. Un fallo de envío o facturación no debe impedir consultar el catálogo. |
| Medida de la respuesta | Disponibilidad mensual de al menos 99,5 % para consultar productos, actualizar el carrito y generar pedidos. El conjunto se considera disponible cuando las tres operaciones pueden completarse correctamente. |
| Verificación | Realizar comprobaciones automáticas cada minuto con cuentas y datos de prueba, sin cargos reales. Registrar las interrupciones, estimar su duración y calcular la disponibilidad mensual. Complementar el monitoreo con pruebas de fallo y recuperación. |
| Trazabilidad | RF01 y RF02: consulta de productos; RF04: gestión del carrito; RF05: generación de pedidos; DA07: disponibilidad y control de fallos; DA08: coordinación con envío y facturación. |
| Estado | Meta propuesta. Pendiente de confirmar el horario de servicio, implementar el monitoreo y obtener registros de un mes. |

### Cálculo de disponibilidad

Disponibilidad (%) = (tiempo disponible / tiempo de servicio observado) × 100.

El tiempo disponible corresponde al periodo observado menos
las interrupciones detectadas. La precisión de esta estimación
dependerá de la frecuencia de las comprobaciones.

### Consideraciones de la evaluación

- Mostrar un error controlado no convierte una operación fallida
  en disponible.
- Las funciones que sigan operativas se registrarán por separado
  para identificar una interrupción parcial.
- Las comprobaciones deben ejecutar operaciones representativas;
  recibir una respuesta de la página principal no demuestra
  que el carrito o los pedidos funcionen.
- La medición incluirá las interrupciones por mantenimiento
  mientras no se apruebe y documente otra política.
- La disponibilidad de los proveedores externos se registrará
  por separado para identificar el origen de los fallos.
- Una prueba breve de recuperación no demuestra por sí sola
  el cumplimiento de la disponibilidad mensual.

  ## EQ-03. Escalabilidad: crecimiento de la demanda

| Elemento | Descripción |
|---|---|
| Atributo de calidad | AC03: Escalabilidad. |
| Fuente del estímulo | Aumento de clientes durante una campaña comercial. |
| Estímulo | La cantidad de usuarios activos concurrentes aumenta de 100 a 200. |
| Artefacto | API REST, módulos Catálogo y Carrito, instancias del backend, gestión de sesiones y almacenamiento. |
| Entorno | Prueba controlada con los mismos datos y distribución de acciones de EQ-01. Se documentan los recursos iniciales y los recursos o instancias añadidos. Se propone mantener 200 usuarios durante 30 minutos después del calentamiento. |
| Respuesta | El sistema amplía su capacidad y atiende la nueva carga, conservando los resultados correctos y la separación de sesiones y carritos. |
| Medida de la respuesta | Con 200 usuarios concurrentes, al menos el 95 % de las solicitudes de cada operación evaluada responde en 2 segundos o menos. Se mantiene el límite propuesto de errores técnicos inesperados inferior al 1 %. Los carritos y totales comprobados deben seguir siendo correctos. |
| Verificación | Comparar las pruebas con 100 y 200 usuarios. Registrar tiempos por operación, errores, solicitudes por segundo, uso de CPU, memoria y conexiones a datos. Documentar la ampliación realizada y comprobar si permitió conservar las metas. |
| Trazabilidad | RF01: búsqueda de productos; RF02: detalle del producto; RF04: gestión del carrito; DA01: crecimiento de usuarios; DA02: rendimiento; ADR-001: monolito modular. |
| Estado | Meta propuesta. Pendiente de definir la infraestructura y ejecutar las pruebas de crecimiento. |

### Consideraciones de la evaluación

- La ampliación puede realizarse aumentando los recursos del servidor
  o ejecutando más instancias del backend.
- Una instancia es una copia de la aplicación en ejecución.
  Varias instancias pueden atender solicitudes de distintos usuarios.
- Si se utilizan varias instancias, las sesiones no deben depender
  exclusivamente de la memoria de una sola instancia.
- Cada usuario debe conservar su propio carrito y sus permisos,
  independientemente de la instancia que atienda la solicitud.
- Se mantendrán comparables los datos, las acciones y las pausas
  de los usuarios para evaluar el efecto de ampliar los recursos.
- El escenario permite una ampliación planificada. El escalado
  automático requeriría una decisión adicional.
- Aumentar los recursos no demuestra por sí solo la escalabilidad:
  deben comprobarse los tiempos, los errores y la corrección
  de los resultados.

## EQ-04. Seguridad: acceso según rol y propiedad de los datos

| Elemento | Descripción |
|---|---|
| Atributo de calidad | AC04: Seguridad. |
| Fuente del estímulo | Usuario sin sesión, con credenciales inválidas o sin autorización sobre la operación o los datos solicitados. |
| Estímulo | El usuario intenta consultar pedidos ajenos, modificar productos de otro vendedor o ejecutar funciones administrativas sin el rol correspondiente. |
| Artefacto | API REST y controles de autenticación y autorización de los módulos Usuarios, Sellers, Catálogo y Pedidos. |
| Entorno | Entorno de prueba con dos clientes, dos vendedores y un administrador. Se envían solicitudes directamente a la API, incluyendo identificadores ajenos, sesiones vencidas y credenciales inválidas. |
| Respuesta | El backend rechaza el acceso, no ejecuta cambios no autorizados y no devuelve información de otro cliente o vendedor. Los usuarios autorizados pueden realizar las operaciones permitidas. |
| Medida de la respuesta | Rechazar el 100 % de los intentos no autorizados incluidos en la matriz de pruebas, sin cambios indebidos ni exposición de datos sensibles en las respuestas revisadas. Los accesos autorizados de la matriz deben funcionar. Las comunicaciones deben utilizar HTTPS y las contraseñas persistidas deben almacenarse mediante un hash adecuado para contraseñas. |
| Verificación | Ejecutar la matriz de accesos permitidos y denegados. Revisar las respuestas y comprobar los datos antes y después de cada intento. Inspeccionar la configuración HTTPS y el almacenamiento de contraseñas. |
| Trazabilidad | RF03: gestión de productos propios; RF06: consulta de pedidos propios; RF16: ventas del vendedor; RF18: sesiones; RF19: administración de cuentas; RF20: control de acceso; DA03 y DA10. |
| Estado | Criterios definidos. Pendiente de implementar y verificar los controles en el backend. |

### Matriz inicial de pruebas de acceso

| Usuario | Operación | Resultado esperado |
|---|---|---|
| Usuario sin sesión | Consultar pedidos de un cliente. | Acceso rechazado, sin información de pedidos. |
| Cliente A | Consultar sus propios pedidos. | Acceso permitido. |
| Cliente A | Consultar un pedido del cliente B cambiando su identificador. | Acceso rechazado, sin información del pedido ajeno. |
| Vendedor A activo | Modificar uno de sus productos. | Acceso permitido. |
| Vendedor A | Modificar un producto del vendedor B. | Acceso rechazado, sin modificar el producto. |
| Vendedor A | Consultar sus ventas. | Acceso permitido, mostrando únicamente la información correspondiente a ese vendedor. |
| Vendedor A | Consultar las ventas del vendedor B. | Acceso rechazado, sin información de ventas ajenas. |
| Cliente o vendedor | Administrar cuentas y asignar roles. | Acceso rechazado, sin cambios en cuentas o permisos. |
| Administrador autorizado | Administrar cuentas y asignar los roles definidos. | Acceso permitido. |
| Usuario con sesión vencida | Ejecutar una operación protegida. | Acceso rechazado hasta autenticarse nuevamente. |
| Usuario con credenciales inválidas | Iniciar sesión. | Inicio de sesión rechazado, sin crear una sesión válida. |

### Consideraciones de la evaluación

- La autorización debe comprobarse en el backend en cada
  operación protegida.
- Ocultar botones en la interfaz no sustituye los controles
  de acceso.
- Tener el rol correcto no permite acceder automáticamente
  a los datos de otro cliente o vendedor.
- Las respuestas de error no deben revelar contraseñas,
  credenciales de proveedores ni información ajena.
- Las contraseñas no deben almacenarse en texto plano.
- Superar esta matriz demuestra el cumplimiento de los casos
  evaluados; no garantiza la ausencia de todas las posibles
  vulnerabilidades.

  ## EQ-05. Mantenibilidad: cambio de una regla del carrito

| Elemento | Descripción |
|---|---|
| Atributo de calidad | AC05: Mantenibilidad. |
| Fuente del estímulo | Equipo de desarrollo ante un cambio aprobado de una regla de negocio. |
| Estímulo | Se modifica una regla de cálculo del total del carrito, por ejemplo, el porcentaje de comisión del marketplace, manteniendo los contratos de entrada y salida. |
| Artefacto | Reglas de precios del dominio, modelo Carrito y casos de uso que utilizan sus resultados. |
| Entorno | Código organizado mediante Clean Architecture, con interfaces estables y pruebas de cálculo, carrito y registro de compra. |
| Respuesta | El cambio se concentra en la lógica responsable y sus pruebas. Los consumidores siguen utilizando los mismos contratos y las funciones relacionadas mantienen su comportamiento correcto. |
| Medida de la respuesta | Ningún cambio necesario en la presentación ni en el acceso a datos mientras sus contratos y responsabilidades se mantengan. Debe aprobarse el 100 % de las pruebas acordadas para la nueva regla y las pruebas de regresión de las funciones afectadas. Las dependencias deben respetar ADR-002. |
| Verificación | Revisar los archivos modificados, ejecutar las pruebas de cálculo, carrito y compra e inspeccionar las importaciones. Comprobar que las reglas de negocio pueden probarse sin Angular, una base de datos o un proveedor de pagos concreto. |
| Trazabilidad | RF04: cálculo y actualización del carrito; RF05: generación del pedido; DA06: mantenibilidad; ADR-002: Clean Architecture. |
| Estado | Criterio definido. Pendiente de realizar y evaluar un cambio de negocio concreto. |

### Consideraciones de la evaluación

- El dominio debe conservar las reglas fundamentales del negocio
  sin depender de implementaciones de infraestructura.
- Los casos de uso deben utilizar las entidades y los contratos
  del núcleo, respetando la dirección de dependencias.
- El carrito y el pedido deben utilizar resultados consistentes
  después de modificar la regla de cálculo.
- Las pruebas de la nueva regla deben comprobar los resultados
  esperados y los casos límite correspondientes.
- Las pruebas de regresión deben comprobar que las funciones
  relacionadas continúan funcionando.
- Si el cambio requiere modificar un contrato, deberán identificarse
  y justificarse los cambios necesarios en sus consumidores.

### Evidencia disponible

Las 16 pruebas ejecutadas del boilerplate aportan evidencia inicial
de que sus reglas y casos de uso pueden probarse sin levantar Angular.

Este resultado no demuestra todavía que se haya realizado el cambio
descrito en este escenario ni que se cumplan las metas de rendimiento,
disponibilidad, escalabilidad o seguridad.

## Resumen de escenarios

| Escenario | Atributo | Comprobación principal |
|---|---|---|
| EQ-01 | AC01: Rendimiento | Al menos el 95 % de las solicitudes de cada operación evaluada responde en 2 segundos o menos con 100 usuarios concurrentes. |
| EQ-02 | AC02: Disponibilidad | Disponibilidad mensual de al menos 99,5 % para las operaciones críticas definidas. |
| EQ-03 | AC03: Escalabilidad | Mantener las metas de rendimiento al crecer hasta 200 usuarios concurrentes y ampliar recursos. |
| EQ-04 | AC04: Seguridad | Rechazar todos los accesos no autorizados de la matriz, sin exposición de información ni modificaciones indebidas. |
| EQ-05 | AC05: Mantenibilidad | Modificar la regla del carrito respetando los contratos, las dependencias y las pruebas acordadas. |
