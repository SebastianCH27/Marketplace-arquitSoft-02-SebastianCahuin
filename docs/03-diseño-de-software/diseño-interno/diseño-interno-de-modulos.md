# Diseño interno de los módulos del Marketplace

## 1. Propósito y alcance

Este documento precisa cómo se organizarán las responsabilidades
internas de Usuarios, Sellers, Catálogo, Carrito, Pedidos y Pagos.

Desarrolla los componentes definidos en
[componentes-arquitectonicos.md](../../02-arquitectura-software/componentes-arquitectonicos.md)
y conserva la trazabilidad con los
[requisitos funcionales](../../01-analisis-de-sistema/03-requisitos-funcionales.md).

El backend mantendrá el monolito modular definido en
[ADR-001](../../02-arquitectura-software/decisiones/ADR-001-monolito-modular.md).

Su organización interna seguirá
[ADR-002: Clean Architecture](../../02-arquitectura-software/decisiones/ADR-002-clean-architecture.md).

Las entidades, casos de uso y contratos descritos forman parte
del diseño propuesto. Su presencia en este documento no significa
que ya estén implementados en el boilerplate.

## 2. Organización interna común

| Parte | Responsabilidad | Ejemplo en el Marketplace |
|---|---|---|
| Dominio | Definir entidades, reglas de negocio y contratos del núcleo. | Producto, Carrito, Pedido, reglas de precios y contratos de repositorios o pagos. |
| Aplicación | Coordinar casos de uso, verificar las condiciones de cada operación y utilizar las entidades y los contratos necesarios. | Consultar catálogo, actualizar carrito, generar pedido y gestionar un intento de pago. |
| Infraestructura | Implementar persistencia, caché e integración con proveedores, transformando sus datos al formato interno. | Repositorios, caché del catálogo y adaptadores de ERP, pagos, envíos y facturación. |
| Presentación del backend | Recibir solicitudes HTTP, validar su estructura, transformar los datos de entrada y delegar en los casos de uso. | Controladores de la API REST y respuestas para la aplicación web. |

La aplicación web consume la API del backend. Las reglas de negocio
y las comprobaciones que autorizan una operación se ejecutan
en el backend, aunque la interfaz también valide formularios
para facilitar su uso.

Los contratos de repositorios y proveedores se ubicarán en Dominio,
siguiendo la organización adoptada en ADR-002. Los adaptadores
de Infraestructura implementarán esos contratos.

La configuración y el arranque conectarán los casos de uso con
las implementaciones elegidas para cada entorno.

## 3. Reglas de dependencia y colaboración

1. Dominio no importa componentes de interfaz, controladores HTTP,
   implementaciones de base de datos ni clientes de proveedores.
2. Aplicación utiliza las entidades y los contratos del núcleo;
   recibe sus implementaciones mediante inyección de dependencias.
3. Infraestructura implementa los contratos y concentra los detalles
   de las tecnologías y servicios utilizados.
4. Presentación delega en los casos de uso. Las reglas de precios,
   existencias y estados se mantienen en el núcleo correspondiente.
5. Cada módulo controla sus datos. La colaboración entre módulos
   utiliza operaciones públicas e interfaces definidas, sin escribir
   directamente en las tablas o estructuras internas de otro módulo.
6. Cada caso de uso protegido comprueba el rol y la autorización
   sobre los recursos concretos, utilizando las funciones de Usuarios
   y la información de propiedad que conserva el módulo responsable.
7. Los datos de solicitud y respuesta entre módulos deben definirse
   explícitamente. Las credenciales del proveedor y los detalles
   del almacenamiento permanecen en Infraestructura.

Estas reglas desarrollan DA06 y permiten evaluar EQ-05.
Su cumplimiento se revisará en las importaciones, los contratos
y las pruebas del código implementado.

## 4. Módulo Usuarios

### 4.1. Responsabilidad y requisitos

Gestiona el registro de clientes, la autenticación, las sesiones,
el estado de las cuentas y la asignación de los roles definidos.

Proporciona la identidad y los permisos necesarios para las
operaciones protegidas del Marketplace.

Requisitos relacionados: RF17, RF18, RF19 y RF20.
Drivers relacionados: DA03 y DA06.
Atributos relacionados: AC04 y AC05.

Los nombres de los elementos siguientes son propuestas de diseño.

### 4.2. Elementos del dominio

| Elemento | Responsabilidad |
|---|---|
| Usuario | Mantener el identificador, correo, estado de la cuenta y roles asignados. Controlar las transiciones válidas entre cuenta activa e inactiva. |
| Rol | Representar los roles definidos: Cliente, Seller y Administrador, junto con las operaciones que permiten realizar. |
| ContextoAcceso | Representar la identidad y los permisos comprobados por el backend para ejecutar una operación. Sus datos no se aceptan directamente del cuerpo de una solicitud del cliente. |

Los datos de productos, vendedores, carritos y pedidos permanecen
en sus módulos responsables. Usuarios proporciona la identidad
y los permisos; cada módulo comprueba la propiedad de sus recursos.

### 4.3. Casos de uso

| Caso de uso propuesto | Operación y controles principales | Requisito |
|---|---|---|
| RegistrarCliente | Validar los datos, comprobar la unicidad del correo, proteger la contraseña y crear la cuenta con el rol Cliente. | RF17. |
| IniciarSesion | Verificar las credenciales y el estado activo de la cuenta antes de crear una sesión válida. | RF18. |
| CerrarSesion | Invalidar la sesión utilizada para que deje de autorizar operaciones protegidas. | RF18. |
| ConsultarUsuarios | Permitir al administrador autorizado consultar las cuentas sin devolver contraseñas ni sus hashes. | RF19 y RF20. |
| CambiarEstadoUsuario | Activar o desactivar una cuenta mediante una operación administrativa autorizada. | RF19 y RF20. |
| AsignarRol | Comprobar el permiso del administrador y asignar únicamente los roles definidos. | RF19 y RF20. |
| ComprobarAcceso | Verificar la sesión, la vigencia del acceso y los permisos necesarios para una operación protegida. La propiedad del recurso se comprueba con el módulo que lo controla. | RF20. |

### 4.4. Contratos e implementaciones

| Contrato del núcleo | Operaciones necesarias | Implementación en Infraestructura |
|---|---|---|
| RepositorioUsuarios | Buscar por identificador o correo, consultar cuentas y guardar cambios de estado o roles. | Acceso al almacenamiento de cuentas, preservando la unicidad del correo también ante registros simultáneos. |
| ProtectorCredenciales | Obtener un hash adecuado para contraseñas y verificar una contraseña contra el valor protegido. | Implementación del mecanismo de protección seleccionado, sin conservar la contraseña en texto plano. |
| GestorSesiones | Crear, validar e invalidar sesiones y relacionarlas con la cuenta correspondiente. | Implementación del mecanismo de sesiones elegido, compatible con varias instancias del backend cuando se utilicen. |

Los casos de uso reciben estos contratos mediante inyección
de dependencias. La configuración selecciona sus implementaciones.

El algoritmo de protección y el mecanismo de sesiones deberán
definirse y verificarse durante la implementación.

### 4.5. Presentación y colaboración

El controlador de Usuarios recibe las solicitudes de registro,
inicio y cierre de sesión y administración de cuentas. Valida
su estructura y delega en los casos de uso correspondientes.

Las operaciones protegidas obtienen su contexto de acceso
desde la sesión verificada por el backend. Recibir un identificador
de usuario o un rol en una solicitud no demuestra autorización.

Por ejemplo, al consultar un pedido, Usuarios comprueba la identidad
y los permisos generales. Pedidos verifica que el pedido solicitado
corresponda al cliente autorizado antes de devolver sus datos.

### 4.6. Reglas y verificación prevista

- El correo debe ser único; el almacenamiento debe impedir
  que dos registros simultáneos creen cuentas con el mismo correo.
- El registro de un cliente no permite asignarse el rol
  Administrador o Seller.
- Una cuenta inactiva no puede iniciar una sesión ni continuar
  realizando operaciones protegidas con una sesión anterior.
- El cierre de sesión invalida el acceso correspondiente.
- La asignación o retirada de roles debe reflejarse en las
  comprobaciones posteriores de permisos.
- Las contraseñas y sus hashes no se devuelven en las respuestas
  ni se incluyen en los registros de incidencias.
- Las operaciones administrativas requieren autorización
  comprobada en el backend.

Se verificarán registros válidos y duplicados, credenciales correctas
e incorrectas, sesiones cerradas o vencidas, cuentas inactivas,
cambios de roles y accesos administrativos permitidos o denegados.
Estas verificaciones complementan la matriz de EQ-04.

La implementación y las pruebas de este módulo quedan pendientes.

## 5. Módulo Sellers

### 5.1. Responsabilidad y requisitos

Gestiona el registro, actualización y desactivación de vendedores
por parte del administrador. También coordina la consulta de
los pedidos y ventas correspondientes al seller autenticado.

Requisitos relacionados: RF07, RF16 y RF20.
Colabora con Catálogo para aplicar RF03 y la restricción de oferta
de vendedores desactivados.

Drivers relacionados: DA03, DA06 y DA10.
Atributos relacionados: AC04 y AC05.

Los nombres de los elementos siguientes son propuestas de diseño.

### 5.2. Elementos del dominio

| Elemento | Responsabilidad |
|---|---|
| Seller | Mantener el identificador del vendedor, su vinculación con la cuenta de usuario, los datos que administra la plataforma y su estado. |
| EstadoSeller | Representar los estados activo e inactivo y las condiciones que permiten publicar o actualizar la oferta del vendedor. |

La cuenta, sus credenciales y sus roles pertenecen a Usuarios.
Los productos pertenecen a Catálogo y los pedidos a Pedidos.
Sellers conserva la identificación y el estado del vendedor.

### 5.3. Casos de uso

| Caso de uso propuesto | Operación y controles principales | Requisito |
|---|---|---|
| RegistrarSeller | Comprobar la autorización del administrador, validar los datos y la cuenta vinculada y registrar al vendedor. | RF07 y RF20. |
| ActualizarSeller | Verificar el permiso del administrador y modificar los datos permitidos del vendedor. | RF07 y RF20. |
| DesactivarSeller | Cambiar el estado del vendedor para impedir nuevas publicaciones y actualizaciones de su oferta, conservando su información histórica. | RF07 y RF20. |
| ConsultarVentasSeller | Obtener el vendedor vinculado al usuario autenticado y consultar en Pedidos únicamente sus productos, cantidades e importes. | RF16 y RF20. |
| ConsultarSellerVinculado | Proporcionar a los casos de uso autorizados el identificador y estado del vendedor asociado a una cuenta, para comprobar el acceso a sus operaciones. | Apoyo a RF03, RF16 y RF20. |

### 5.4. Contratos y colaboración

| Contrato del núcleo | Operaciones necesarias | Implementación o colaboración |
|---|---|---|
| RepositorioSellers | Buscar por identificador o cuenta vinculada, registrar vendedores y guardar cambios de datos o estado. | Persistencia de la información controlada por Sellers. |
| ConsultaCuentasUsuario | Comprobar la existencia, estado y permisos de la cuenta vinculada. | Colaboración con las operaciones públicas de Usuarios, sin acceder directamente a su almacenamiento. |
| ConsultaVentasSeller | Obtener una proyección de pedidos y ventas limitada al vendedor autorizado. | Colaboración con las operaciones públicas de Pedidos, sin modificar ni consultar directamente sus tablas. |

La respuesta de ventas es una proyección de consulta con los datos
permitidos para ese vendedor. No concede acceso al pedido completo
ni a los productos o importes de otros sellers.

### 5.5. Presentación y colaboración

El controlador de Sellers recibe las solicitudes administrativas
y las consultas de ventas. Valida su estructura y delega en
los casos de uso correspondientes.

El identificador del vendedor se obtiene de la vinculación comprobada
con la cuenta autenticada. Un identificador enviado por el cliente
no permite elegir libremente las ventas de otro vendedor.

Catálogo consulta la identificación y el estado del seller para
autorizar la gestión de sus productos. La escritura de productos
continúa siendo responsabilidad de Catálogo.

Pedidos conserva el identificador del vendedor en los detalles
registrados de cada compra y proporciona la proyección autorizada
que utiliza Sellers para presentar sus ventas.

### 5.6. Reglas y verificación prevista

- Registrar, actualizar o desactivar vendedores requiere
  autorización administrativa comprobada en el backend.
- Tener el rol Seller no sustituye la vinculación con el vendedor
  cuya información se solicita.
- Una cuenta inactiva no puede realizar operaciones protegidas,
  aunque el registro del vendedor siga activo.
- Un vendedor desactivado no puede publicar ni actualizar su oferta.
- Desactivar un vendedor no elimina los productos ni los detalles
  de pedidos históricos. Su información se conserva para las
  consultas autorizadas y la trazabilidad.
- La consulta de ventas debe excluir los detalles de otros sellers,
  incluso cuando un mismo pedido contenga productos de varios.
- Los importes históricos proceden de los detalles registrados
  en Pedidos, no de los precios actuales del catálogo.
- La consulta debe distinguir los pedidos pendientes de las compras
  cuyo pago está aprobado.

Se verificarán operaciones administrativas permitidas y denegadas,
cuentas vinculadas inválidas, restricciones de vendedores desactivados
y consultas de pedidos con productos de varios sellers.

También se comprobará que modificar el identificador de una solicitud
no permita acceder a las ventas de otro vendedor.

La implementación y las pruebas de este módulo quedan pendientes.

## 6. Módulo Catálogo

### 6.1. Responsabilidad y requisitos

Gestiona los productos, las búsquedas, los detalles y la disponibilidad.
Permite a cada seller registrar, actualizar y consultar sus productos.
Integra el stock del ERP para los productos vinculados a ese sistema.

Requisitos relacionados: RF01, RF02, RF03, RF20 y RF21.
Colabora con Carrito y Pedidos para aplicar RF04 y RF05.

Drivers relacionados: DA02, DA06, DA09 y DA10.
Atributos relacionados: AC01, AC04 y AC05.
Decisión relacionada: ADR-003, estrategia de caché.

Los nombres de los elementos siguientes son propuestas de diseño.

### 6.2. Elementos del dominio

| Elemento | Responsabilidad |
|---|---|
| Producto | Mantener identificador, nombre, categoría, características, precio, seller responsable y estado de publicación. Validar los datos necesarios para ofrecerlo. |
| DisponibilidadProducto | Representar las existencias conocidas, su fuente y la información de actualización necesaria para comprobar si pueden utilizarse en una compra. |
| VinculoProductoERP | Relacionar el producto interno con el identificador autorizado del ERP, evitando confundir registros al sincronizar su stock. |

La información del producto pertenece a Catálogo. El identificador
del seller se relaciona con Sellers y permite controlar su propiedad.

Los detalles de productos registrados en pedidos históricos
permanecen en Pedidos.

### 6.3. Casos de uso

| Caso de uso propuesto | Operación y controles principales | Requisito |
|---|---|---|
| BuscarProductos | Buscar por nombre y categoría y aplicar la paginación prevista para las consultas públicas. | RF01. |
| ConsultarDetalleProducto | Obtener características, precio, vendedor y disponibilidad del producto. | RF02. |
| RegistrarProducto | Obtener el seller autorizado, comprobar su estado y validar los datos antes de registrar el producto como suyo. | RF03 y RF20. |
| ActualizarProducto | Verificar propiedad y estado del seller, validar los cambios permitidos y guardar el producto. | RF03 y RF20. |
| ConsultarProductosSeller | Obtener únicamente los productos del vendedor autorizado, incluyendo su información de publicación. | RF03 y RF20. |
| SincronizarStockERP | Consultar la fuente autorizada, validar la correspondencia del producto y actualizar su disponibilidad conocida. | RF21. |
| ConsultarProductoParaCompra | Proporcionar a Carrito y Pedidos los datos vigentes necesarios para comprobar precio, publicación, vendedor y existencias. | Apoyo a RF04 y RF05. |
| ControlarDisponibilidadCompra | Validar las cantidades y coordinar el control de existencias al registrar una compra, según el mecanismo de reserva o descuento acordado. | Apoyo a RF05. |

### 6.4. Contratos e implementaciones

| Contrato del núcleo | Operaciones necesarias | Implementación o colaboración |
|---|---|---|
| RepositorioProductos | Buscar, obtener, registrar y actualizar productos, conservando su relación con el seller. | Persistencia de los datos controlados por Catálogo. |
| FuenteStockERP | Obtener información de productos y stock a partir de identificadores autorizados. | Adaptador de ERP que transforma y valida los datos del proveedor. |
| ConsultaSellers | Obtener el vendedor vinculado y comprobar su estado para gestionar su oferta. | Colaboración con las operaciones públicas de Sellers. |
| CacheConsultasCatalogo | Consultar, guardar e invalidar resultados públicos del catálogo. | Implementación de caché definida por ADR-003. |
| ControlStockCompra | Aplicar el mecanismo acordado de reserva, descuento o liberación de existencias para una compra identificada. | Control de concurrencia y coordinación con la fuente de stock; sus garantías deben precisarse según las capacidades del ERP. |

La existencia del contrato ControlStockCompra no supone que el ERP
ya permita reservar o modificar stock. Esa capacidad y el procedimiento
de coordinación deben acordarse antes de implementar la compra.

### 6.5. Presentación y colaboración

El controlador de Catálogo recibe búsquedas, consultas de detalle
y solicitudes de gestión de productos. Delega en los casos de uso
y devuelve únicamente los datos permitidos para cada operación.

Las consultas públicas podrán reutilizar resultados de caché según
ADR-003. Cuando no exista una entrada vigente, se consultarán
los datos persistentes y se actualizará la caché.

Carrito consulta los productos mediante la interfaz pública de Catálogo.
Pedidos vuelve a comprobar precios y disponibilidad al preparar
la compra y coordina con Catálogo el control de existencias.
Ninguno modifica directamente los datos internos de productos.

La actualización desde el ERP utiliza su adaptador. La frecuencia,
el mecanismo de sincronización y la vigencia aceptable de los datos
quedan pendientes de acuerdo con el proveedor.

### 6.6. Reglas y verificación prevista

- El producto debe conservar un seller responsable. Ese identificador
  no puede cambiarse libremente desde una solicitud de actualización.
- Registrar o actualizar productos requiere un vendedor autorizado
  y activo, además de la comprobación de propiedad correspondiente.
- Los precios deben ser válidos y positivos, y las existencias
  deben ser cantidades válidas y no negativas.
- Para los productos vinculados, el ERP es la fuente de stock.
  Los datos de una solicitud del vendedor no sustituyen esa fuente.
- Una comunicación fallida con el ERP no debe interpretarse
  como stock cero ni como disponibilidad confirmada para comprar.
- La caché no constituye la fuente definitiva de precios o stock
  al confirmar una compra.
- Los cambios de productos, precios, publicación o datos sincronizados
  incluidos en caché deben invalidar las entradas afectadas según ADR-003.
- Consultar existencias y descontarlas como operaciones independientes
  no basta para evitar que dos compras utilicen las mismas unidades.
  El mecanismo de control deberá preservar esa condición.
- El comportamiento de la oferta ya publicada cuando se desactiva
  un seller debe precisarse antes de implementar las compras.
  Se propone impedir nuevas compras y conservar los datos históricos.

Se verificarán búsquedas y detalles, gestión de productos propios,
rechazo de cambios sobre productos ajenos, restricciones del seller,
datos inválidos del ERP, invalidación de caché y compras concurrentes
que compitan por las mismas existencias.

La implementación y las pruebas de este módulo quedan pendientes.

## 7. Módulo Carrito

### 7.1. Responsabilidad y requisitos

Gestiona la selección de productos de cada cliente, sus cantidades
y el cálculo del total previo a la compra. Proporciona a Pedidos
una selección identificable para preparar el pedido.

Requisitos relacionados: RF04 y RF20.
Colabora con Catálogo y Pedidos para aplicar RF05.
Drivers relacionados: DA02, DA03 y DA06.
Atributos relacionados: AC01, AC04 y AC05.
Escenario relacionado: EQ-05, cambio de una regla del carrito.

Los nombres de los elementos siguientes son propuestas de diseño.

### 7.2. Elementos del dominio

| Elemento | Responsabilidad |
|---|---|
| Carrito | Mantener identificador, cliente propietario, líneas de productos y versión de la selección. Controlar las operaciones que modifican su contenido. |
| LineaCarrito | Representar el producto seleccionado, su seller y la cantidad solicitada, junto con los datos de precio utilizados para presentar su importe. |
| ReglasPrecio | Concentrar el cálculo de precios al cliente, importes, subtotal, comisión, impuesto y redondeo según las reglas adoptadas. |

Catálogo mantiene los datos vigentes de los productos. Carrito
conserva la selección y utiliza la información proporcionada
por ese módulo, sin modificar sus productos ni existencias.

### 7.3. Casos de uso

| Caso de uso propuesto | Operación y controles principales | Requisito |
|---|---|---|
| ConsultarCarrito | Obtener la selección del cliente autorizado y presentar cantidades, importes y disponibilidad conocida. | RF04 y RF20. |
| AgregarProductoAlCarrito | Verificar el cliente, consultar el producto y validar la cantidad final antes de agregarlo o acumular unidades en su línea existente. | RF04 y RF20. |
| ModificarCantidadCarrito | Comprobar la propiedad del carrito y validar la nueva cantidad y disponibilidad antes de guardar el cambio. | RF04 y RF20. |
| EliminarProductoDelCarrito | Comprobar la propiedad y retirar la línea correspondiente, actualizando los importes y el total. | RF04 y RF20. |
| ObtenerSeleccionParaPedido | Entregar a Pedidos una instantánea de la selección autorizada, identificando el carrito, su versión, productos y cantidades. | Apoyo a RF05 y RF20. |
| AplicarSeleccionComprada | Procesar una confirmación interna de Pedidos una sola vez. Si la versión sigue coincidiendo, retirar la selección comprada; si cambió, conservar el carrito e informar que no se limpió automáticamente. | Apoyo a RF05. |

Una instantánea conserva el contenido seleccionado en un momento
concreto. Los cambios posteriores del carrito no modifican los datos
ya registrados en un pedido.

AplicarSeleccionComprada es una operación interna identificada
por la compra confirmada; no se habilita como una solicitud libre
del cliente para declarar pagado un pedido.

### 7.4. Contratos y colaboración

| Contrato del núcleo | Operaciones necesarias | Implementación o colaboración |
|---|---|---|
| RepositorioCarritos | Obtener el carrito del cliente y guardar cambios comprobando su versión. | Persistencia compartida que detecta modificaciones concurrentes y evita sobrescribir cambios sin comprobarlos. |
| ConsultaProductosCatalogo | Obtener productos, precios y disponibilidad mediante las operaciones autorizadas de Catálogo. | Colaboración con la interfaz pública de Catálogo, sin acceder directamente a sus tablas. |
| ConsultaAccesoUsuario | Comprobar identidad, estado de la cuenta y permisos necesarios. | Colaboración con las operaciones públicas de Usuarios. |

Las reglas de precios permanecen en el dominio. Sus cálculos
no requieren una llamada a la base de datos ni a una pasarela.
Los casos de uso obtienen los datos necesarios mediante contratos
y aplican esas reglas antes de guardar o presentar el resultado.

### 7.5. Presentación y colaboración

El controlador de Carrito recibe las solicitudes de consulta,
adición, modificación y eliminación. Obtiene el contexto de acceso
verificado y delega en los casos de uso correspondientes.

El cliente solicita productos y cantidades. Los precios, importes
y totales enviados desde la interfaz no se aceptan como definitivos:
el backend realiza sus propios cálculos.

Pedidos utiliza la selección del carrito y vuelve a validar precios,
publicación y existencias con Catálogo antes de registrar la compra.
Si cambia el importe presentado, se muestra el nuevo total y se
solicita su confirmación antes del cobro, según ADR-003.

### 7.6. Reglas y verificación prevista

- Cada operación protegida comprueba que el carrito corresponda
  al cliente autorizado.
- Las cantidades deben ser enteras y mayores que cero.
  Para retirar una línea se utiliza la operación de eliminación.
- Agregar nuevamente un producto acumula unidades en su línea
  y valida la cantidad final frente a la disponibilidad conocida.
- Un carrito vacío puede consultarse, pero no permite generar
  un pedido.
- Las reglas de comisión, impuesto y redondeo se mantienen
  en el núcleo y se utilizan de forma consistente al preparar el pedido.
- Agregar un producto al carrito no reserva ni descuenta existencias.
  La disponibilidad puede cambiar antes de comprar.
- La preparación de una compra identifica la versión de la selección
  para detectar cambios durante el proceso.
- Las modificaciones simultáneas deben detectarse o coordinarse;
  no deben sobrescribir silenciosamente los cambios de otra solicitud.
- La actualización del carrito tras una compra debe realizarse
  mediante su propia operación y conservar los cambios posteriores
  que no formen parte de la selección comprada.

Se verificarán cantidades inválidas, acumulación de unidades,
stock insuficiente, eliminación de líneas, carritos vacíos,
totales calculados en el backend, accesos ajenos y modificaciones
concurrentes de una misma selección.

El boilerplate contiene Carrito, LineaCarrito y las funciones de
precios.ts como referencia. La identificación del cliente, los controles
de acceso, la persistencia y la coordinación de versiones forman parte
de la evolución propuesta y quedan pendientes de implementación.

## 8. Módulo Pedidos

### 8.1. Responsabilidad y requisitos

Genera y conserva los pedidos del cliente, su dirección de entrega
y los datos de la compra. Coordina con Pagos el resultado del cobro
y utiliza los adaptadores de envíos y facturación para continuar
el procesamiento cuando se cumplen las condiciones del negocio.

Requisitos relacionados: RF05, RF06, RF08, RF09, RF12, RF13,
RF14, RF15 y RF20.
Colabora con Pagos para RF10 y RF11 y con Sellers para RF16.
Drivers relacionados: DA03, DA04, DA06, DA07, DA08, DA09 y DA10.
Atributos relacionados: AC02, AC04 y AC05.
Decisión relacionada: ADR-004.

Los nombres de los elementos siguientes son propuestas de diseño.
Los adaptadores de envío y facturación pertenecen a Infraestructura
y son utilizados por Pedidos.

### 8.2. Elementos del dominio

| Elemento | Responsabilidad |
|---|---|
| Pedido | Conservar identificador, cliente, líneas, importes, moneda, dirección y referencia a la selección del carrito. Controlar sus transiciones y las condiciones para continuar la compra. |
| LineaPedido | Conservar producto, seller, descripción, cantidad y precio aplicado en la compra, junto con sus importes. |
| DireccionEntrega | Representar los datos de una dirección perteneciente al cliente. El pedido conserva una copia de la dirección seleccionada. |
| SeguimientoPedido | Mantener por separado el estado del pago, la preparación y el envío, y la facturación, con sus referencias y fechas. |

Las líneas y la dirección del pedido conservan los datos utilizados
al confirmar la compra. Un cambio posterior del producto, del precio
o de la dirección guardada no modifica esos datos históricos.

Pagos controla los intentos y resultados del cobro. Pedidos conserva
la referencia y el estado necesarios para coordinar la compra,
utilizando únicamente resultados verificados de ese módulo.

### 8.3. Casos de uso

| Caso de uso propuesto | Operación y controles principales | Requisito |
|---|---|---|
| RegistrarDireccionEntrega | Validar y guardar una dirección vinculada al cliente autorizado. | RF09 y RF20. |
| GenerarPedido | Obtener la selección del carrito, validar la dirección elegida, comprobar productos y existencias, calcular importes y registrar el pedido pendiente de pago. | RF05, RF09 y RF20. |
| ConsultarPedidosCliente | Recuperar únicamente los pedidos del cliente autorizado y presentar sus estados. | RF06 y RF20. |
| ConsultarDetallePedido | Presentar productos, cantidades, importes y dirección del pedido propio. | RF08 y RF20. |
| AplicarResultadoPago | Comprobar la referencia y los datos del resultado verificado por Pagos y actualizar el seguimiento de la compra una sola vez. | Colaboración con RF11. |
| SolicitarDespacho | Enviar la información necesaria del pedido y su dirección cuando el pago esté aprobado y los productos preparados. | RF12. |
| ActualizarEstadoEntrega | Asociar una actualización verificada del servicio de envío con el pedido correspondiente y permitir su consulta al cliente. | RF13 y RF20. |
| SolicitarComprobante | Solicitar la facturación de una compra cuyo pago esté aprobado. | RF14. |
| AsociarComprobante | Guardar la referencia del comprobante recibido y vincularla con la compra correspondiente. | RF15. |
| ConsultarComprobante | Permitir al cliente autorizado consultar el comprobante de su compra. | RF15 y RF20. |
| ConsultarPartePedidoSeller | Proporcionar a Sellers solamente las líneas, cantidades e importes del vendedor autorizado. | Apoyo a RF16 y RF20. |

La consulta del cliente también muestra el estado de entrega disponible.
La selección de una dirección comprueba que pertenezca al cliente
antes de incorporarla al pedido.

### 8.4. Contratos y colaboración

| Contrato del núcleo | Operaciones necesarias | Implementación o colaboración |
|---|---|---|
| RepositorioPedidos | Registrar y consultar pedidos, actualizar su versión y conservar las operaciones pendientes y los resultados ya procesados. | Persistencia que detecta cambios concurrentes y permite recuperar operaciones interrumpidas. |
| RepositorioDirecciones | Guardar y consultar direcciones comprobando su relación con el cliente. | Persistencia de las direcciones controladas por Pedidos. |
| OperacionesCarrito | Obtener la selección y su versión y aplicar la selección comprada tras la confirmación válida. | Interfaz pública de Carrito. |
| OperacionesCatalogo | Obtener datos vigentes para comprar y coordinar el control de existencias de una compra identificada. | Interfaz pública de Catálogo, respetando la fuente de stock acordada. |
| OperacionesPagos | Iniciar o consultar un pago y obtener sus resultados verificados. | Interfaz pública de Pagos, según ADR-004. |
| ServicioEnvios | Solicitar el despacho y obtener actualizaciones verificadas de la entrega. | Adaptador del proveedor de envíos. |
| ServicioFacturacion | Solicitar y recuperar el comprobante correspondiente a una compra identificada. | Adaptador del proveedor de facturación. |
| ConsultaAccesoUsuario | Comprobar identidad, estado de la cuenta y permisos necesarios. | Interfaz pública de Usuarios. |

Los casos de uso utilizan estos contratos. Los detalles de conexión,
autenticación técnica y formatos de cada proveedor permanecen
en Infraestructura.

### 8.5. Presentación y coordinación de la compra

Los controladores reciben las solicitudes del cliente y delegan
en los casos de uso. El cliente no puede establecer libremente
el propietario, el total ni los estados del pedido.

El flujo propuesto es el siguiente:

1. Comprobar el acceso, obtener la selección del carrito y validar
   la dirección elegida.
2. Consultar los datos vigentes con Catálogo y calcular los importes
   en el backend. Si cambian respecto al total presentado, solicitar
   la confirmación del nuevo importe antes del cobro.
3. Registrar el pedido pendiente de pago y coordinar con Catálogo
   el control de existencias antes de solicitar el cobro.
   El mecanismo deberá evitar que compras concurrentes comprometan
   las mismas unidades y permitir recuperar fallos intermedios.
4. Solicitar el pago mediante Pagos, conservando las referencias
   necesarias para consultar su resultado y recuperar la operación.
5. Aplicar el resultado verificado. Si el pago queda aprobado,
   registrar las acciones pendientes de facturación, preparación
   del envío y actualización del carrito.
6. Ejecutar esas acciones mediante sus operaciones identificadas.
   El despacho requiere además que los productos estén preparados;
   sus resultados se registran por separado.

La actualización del carrito utiliza la versión de la selección
comprada. Si el cliente lo modificó durante el proceso, se conserva
su contenido y se informa que no se limpió automáticamente.

Las notificaciones de envío y facturación ingresan por la API
y se verifican mediante el adaptador correspondiente antes
de aplicar cambios al pedido.

La preparación de los productos y la coordinación del stock necesitan
procedimientos definidos. Las capacidades del ERP y la recuperación
entre persistencia local y operaciones externas siguen pendientes
de acuerdo e implementación.

### 8.6. Reglas y verificación prevista

- Un pedido requiere cliente autorizado, productos, cantidades
  válidas, importes calculados por el backend y dirección de entrega.
- La generación repetida de una misma compra debe identificarse
  para evitar pedidos y acciones duplicados.
- El pago solo se considera aprobado por un resultado verificado
  de Pagos que corresponda al pedido, importe y moneda esperados.
- Un tiempo de espera agotado durante el cobro conserva la operación
  pendiente de verificación; no permite suponer un rechazo
  y realizar otro cobro sin comprobar el resultado anterior.
- Un fallo de envío o facturación conserva el pago confirmado
  y registra la acción pendiente de recuperación.
- La persistencia debe conservar conjuntamente el cambio de estado
  local y sus acciones pendientes, para poder continuar tras un fallo.
- Los reintentos de una misma acción conservan su identificación.
  Antes de repetir una solicitud externa de resultado incierto,
  se comprueba su estado mediante las capacidades del proveedor.
- Las actualizaciones repetidas o fuera de orden no deben generar
  un segundo efecto ni retroceder estados sin una transición válida.
- Las modificaciones concurrentes del pedido deben coordinarse
  para evitar perder cambios o procesar dos veces una acción.
- El cliente consulta solamente sus pedidos y comprobantes.
  El seller recibe únicamente la parte que le corresponde,
  distinguiendo pedidos pendientes de compras con pago aprobado.
- Ningún módulo modifica directamente las tablas o los datos
  internos controlados por otro módulo.

Se verificarán dirección ajena, carrito vacío, cambios de precio,
stock insuficiente, compras concurrentes, solicitudes duplicadas,
pagos pendientes o rechazados, fallos después de un pago aprobado,
actualizaciones externas repetidas y consultas de pedidos ajenos.

El boilerplate contiene Pedido y RegistrarCompraCasoUso como referencia.
Su flujo crea el pedido después de aprobar el cobro. La propuesta
evoluciona hacia el registro previo del pedido pendiente, la separación
de estados y la recuperación de operaciones definida en ADR-004.
Estas ampliaciones quedan pendientes de implementación y pruebas.

## 9. Módulo Pagos

### 9.1. Responsabilidad y requisitos

Gestiona los intentos de pago de un pedido y su comunicación
con la pasarela externa. Registra resultados verificados y los
comunica a Pedidos para continuar el procesamiento de la compra.

Requisitos relacionados: RF10, RF11 y RF20.
Drivers relacionados: DA03, DA04, DA06 y DA07.
Atributos relacionados: AC02, AC04 y AC05.
Restricciones relacionadas: RC04 y RC09.
Decisión relacionada: ADR-004.

Los nombres de los elementos siguientes son propuestas de diseño.
El proveedor y sus condiciones de integración siguen pendientes
de selección y verificación.

### 9.2. Elementos del dominio

| Elemento | Responsabilidad |
|---|---|
| IntentoPago | Conservar identificador, pedido, cliente, importe, moneda, referencia del proveedor, estado y fechas. Identificar la operación que se solicita o recupera. |
| EstadoPago | Representar estados como creado, pendiente de verificación, aprobado y rechazado, aplicando las transiciones permitidas. |
| ResultadoPagoVerificado | Representar la información del proveedor después de verificar su autenticidad y correspondencia con el intento, incluyendo referencia, importe, moneda y estado. |
| RegistroNotificacionPago | Conservar la identificación de una notificación y su procesamiento para controlar eventos repetidos. |

Un intento identifica una operación de cobro concreta.
Reintentar su comunicación conserva esa identificación.
Un nuevo intento requiere comprobar que el anterior terminó
sin cobro y que el pedido sigue admitiendo el pago.

Un fallo de comunicación mantiene el resultado pendiente
de verificación. No equivale a un rechazo confirmado.

### 9.3. Casos de uso

| Caso de uso propuesto | Operación y controles principales | Requisito |
|---|---|---|
| IniciarPago | Comprobar el acceso y las condiciones del pedido, obtener su importe del backend y registrar un intento antes de comunicarse con la pasarela. | RF10 y RF20. |
| ConsultarEstadoPago | Presentar al cliente autorizado el estado registrado y su actualidad, recuperando el resultado externo cuando sea necesario. | RF11 y RF20. |
| ProcesarNotificacionPago | Verificar el origen y contenido de la notificación, comprobar su correspondencia con el intento y registrar el resultado sin duplicar efectos. | RF11. |
| RecuperarPagoPendiente | Consultar a la pasarela utilizando la referencia conservada y resolver una operación interrumpida sin iniciar otro cobro. | Apoyo a RF11. |
| ComunicarResultadoAPedidos | Entregar el resultado verificado a la operación pública de Pedidos y conservar su comunicación pendiente hasta que pueda completarse. | Colaboración con RF11. |

La respuesta inicial de la pasarela, las consultas de recuperación
y las notificaciones aplican las mismas comprobaciones de negocio
antes de actualizar el intento.

### 9.4. Contratos e implementaciones

| Contrato del núcleo | Operaciones necesarias | Implementación o colaboración |
|---|---|---|
| RepositorioPagos | Guardar intentos y referencias, comprobar versiones, registrar notificaciones procesadas y conservar resultados pendientes de comunicar. | Persistencia compartida con controles de unicidad y concurrencia. |
| ProcesadorPagos | Iniciar una operación identificada, consultar su estado y verificar las notificaciones del proveedor, devolviendo información en el formato interno. | Adaptador simulado para pruebas o adaptador de la pasarela seleccionada. |
| ConsultaPedidoParaPago | Obtener el propietario, importe, moneda y condiciones vigentes del pedido para autorizar el intento. | Interfaz pública de Pedidos. |
| ComunicacionResultadoPedido | Aplicar al pedido el resultado verificado de un intento identificado. | Interfaz pública de Pedidos, con control de resultados repetidos. |
| ConsultaAccesoUsuario | Comprobar identidad, estado de la cuenta y permisos necesarios. | Interfaz pública de Usuarios. |

El contrato ProcesadorPagos utiliza conceptos del marketplace.
El adaptador concentra la autenticación técnica, las solicitudes
HTTP y la transformación de los datos del proveedor.

La configuración selecciona la implementación utilizada.
El dominio y los casos de uso permanecen independientes
de una pasarela o tecnología concreta.

### 9.5. Presentación y colaboración

El controlador de pagos recibe las solicitudes del cliente,
obtiene el contexto de acceso verificado y delega en los casos de uso.
El importe y la moneda se obtienen del pedido registrado;
los valores enviados por la interfaz no determinan el cobro.

La captura de tarjetas se delega a los mecanismos del proveedor.
El marketplace utiliza las referencias o tokens necesarios
según el contrato de integración acordado.

Las notificaciones externas ingresan por un endpoint específico.
Su verificación utiliza el mecanismo del proveedor, conservando
el formato original recibido cuando sea necesario para comprobarlo.
Una sesión del cliente o el retorno de su navegador no sustituyen
esa verificación.

Después de registrar un resultado válido, Pagos lo comunica a Pedidos
mediante su operación pública. La entrega repetida del mismo resultado
debe conservar un único efecto sobre el pedido.

Si la comunicación entre módulos se interrumpe, se conserva
la acción pendiente para continuarla. Pagos controla el cobro;
Pedidos coordina la facturación, la preparación y el envío.

### 9.6. Reglas y verificación prevista

- Cada consulta o solicitud del cliente comprueba su autorización
  sobre el pedido y el pago correspondiente.
- El intento se registra con su identificación antes de solicitar
  el cobro externo.
- La misma operación conserva su referencia al reintentar.
  Reutilizarla con otro pedido, importe o moneda debe rechazarse.
- Debe impedirse que solicitudes concurrentes creen cobros distintos
  para un pedido ya pagado o con un resultado todavía incierto.
- La aprobación requiere comprobar autenticidad, referencia,
  importe, moneda y estado de la información del proveedor.
- Las notificaciones duplicadas y las recibidas fuera de orden
  no deben repetir efectos ni provocar transiciones inválidas.
- El cambio de estado local y el resultado pendiente de comunicar
  deben conservarse conjuntamente para permitir su recuperación.
- Un tiempo de espera agotado obliga a verificar el resultado
  anterior antes de permitir otra operación de cobro.
- Los reintentos externos utilizan las capacidades de idempotencia
  y consulta acordadas con el proveedor. Si esas capacidades
  no permiten resolver una operación incierta, se mantiene pendiente
  y se aplica el procedimiento de revisión definido.
- Las comunicaciones utilizan HTTPS y las credenciales secretas
  permanecen en el backend, fuera del repositorio público.
- No se almacenan números completos de tarjetas ni códigos
  de seguridad en bases de datos, registros o archivos, según RC09.

Se verificarán pagos aprobados y rechazados, consultas ajenas,
solicitudes simultáneas, notificaciones inválidas o duplicadas,
resultados con datos incorrectos, tiempos de espera agotados,
fallos de persistencia después del cobro y recuperación de resultados
pendientes de comunicar a Pedidos.

El boilerplate incluye ProcesadorPagos, ProcesadorPagosSimulado
y ProcesadorPagosNiubiz como referencias de contrato y adaptadores.
Su resultado simplificado de aprobación o rechazo se ampliaría
con intentos persistentes, consultas, verificación y recuperación.

El adaptador HTTP del ejemplo no demuestra una integración real
completa con el proveedor. La implementación propuesta deberá
verificarse primero con simulaciones y después en el entorno
de pruebas de la pasarela seleccionada.