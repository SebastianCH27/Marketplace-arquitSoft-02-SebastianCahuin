# ADR-002: Aplicar Clean Architecture

- Fecha: 2026-09-28
- Estado: Aceptada para la propuesta arquitectónica.
- Driver principal: DA06 — Mantenibilidad y evolución modular.
- Atributo relacionado: AC05 — Mantenibilidad.
- Decisión relacionada: ADR-001 — Monolito modular.

## Contexto

El marketplace contiene reglas relacionadas con productos, existencias,
carrito, pedidos y pagos. Estas reglas deberán mantenerse y probarse
aunque cambien la interfaz de usuario, la tecnología de almacenamiento
o los proveedores externos.

La arquitectura inicial en capas permitió separar responsabilidades.
En esta etapa se necesita precisar la dirección de las dependencias
para evitar que las reglas del negocio queden vinculadas directamente
a detalles tecnológicos.

El monolito modular definido en ADR-001 establece la organización
global del backend. Esta decisión complementa esa organización
definiendo cómo estructurar sus responsabilidades internas.

## Decisión

Se aplicará Clean Architecture, organizando las responsabilidades
en Dominio, Aplicación, Infraestructura y Presentación.

Las dependencias del código se dirigirán hacia el núcleo, formado
por Dominio y Aplicación.

Los módulos definidos en ADR-001 conservarán sus responsabilidades.
La separación interna permitirá identificar sus reglas, casos de uso,
interfaces de entrada y adaptadores técnicos.

## Distribución de responsabilidades

| Parte | Responsabilidad | Ejemplos en el marketplace |
|---|---|---|
| Dominio | Contener entidades, reglas fundamentales y contratos del núcleo. | Producto, Carrito, Pedido, reglas de stock y cálculo de precios. |
| Aplicación | Coordinar los casos de uso y aplicar las reglas necesarias para completar cada operación. | Consultar catálogo, agregar al carrito y registrar una compra. |
| Infraestructura | Implementar los contratos de acceso a datos y comunicación con sistemas externos. | Repositorios y adaptadores de ERP, pagos, envíos y facturación. |
| Presentación | Recibir las acciones o solicitudes, transformar sus datos y presentar los resultados. | Componentes de la aplicación web y controladores de entrada de la API REST. |

En la solución completa, la aplicación web se comunicará con
el backend mediante la API REST. El backend será responsable
de comprobar permisos, precios, existencias y demás condiciones
necesarias para autorizar las operaciones.

## Reglas de dependencia

1. Dominio no importará componentes de Presentación,
   Infraestructura ni frameworks.

2. Aplicación dependerá de las entidades y contratos del núcleo.
   Sus casos de uso recibirán las dependencias necesarias mediante
   interfaces.

3. Infraestructura implementará los contratos definidos en el núcleo.
   Allí se resolverán las consultas a datos y las llamadas
   a proveedores externos.

4. Presentación delegará las operaciones a los casos de uso
   correspondientes. Las reglas fundamentales no se duplicarán
   en botones, formularios o controladores.

5. La configuración de la aplicación conectará los casos de uso
   con las implementaciones concretas.

Una llamada realizada durante la ejecución puede llegar a un adaptador
externo. Esto es compatible con la regla de dependencia porque el caso
de uso conoce el contrato del núcleo, mientras que el adaptador
implementa ese contrato.

## Referencia observada en el boilerplate

El ejemplo de la docente organiza su código dentro de
`boilerplate/src/app/` de la siguiente manera:

| Elemento | Evidencia en el código |
|---|---|
| Dominio | `dominio/modelos/` contiene las entidades y reglas; `dominio/contratos/` contiene las interfaces. |
| Aplicación | `aplicacion/` contiene los casos de uso, que importan entidades y contratos del dominio. |
| Infraestructura | `infraestructura/` contiene repositorios en memoria, implementaciones HTTP y adaptadores de pagos y notificaciones. |
| Presentación | `presentacion/` contiene los componentes de catálogo y carrito. |
| Configuración | `app.config.ts` selecciona las implementaciones y las entrega a los casos de uso. |

Por ejemplo, `RegistrarCompraCasoUso` recibe el contrato
`ProcesadorPagos` mediante su constructor. La implementación
`ProcesadorPagosSimulado` se selecciona en `app.config.ts`.

Esta organización permite probar el caso de uso con un procesador
simulado y preparar otras implementaciones que respeten el contrato.

El boilerplate demuestra el enfoque dentro de una aplicación Angular.
La construcción del backend y la integración con servicios reales
corresponden al desarrollo posterior de la solución.

## Alternativas consideradas

| Alternativa | Evaluación |
|---|---|
| Mantener capas tradicionales con dependencias directas desde el negocio hacia las implementaciones de datos | Separa responsabilidades, pero los cambios técnicos pueden propagarse hacia las reglas del negocio. |
| Aplicar Clean Architecture con contratos en el núcleo | Permite que los detalles técnicos dependan de las interfaces requeridas por el negocio. Es la alternativa seleccionada. |

## Justificación

La decisión responde a DA06 porque permite modificar reglas,
casos de uso e implementaciones técnicas dentro de límites definidos.

También contribuye a AC05: una modificación en el cálculo del carrito
debería concentrarse en la regla correspondiente y sus pruebas,
manteniendo las interfaces cuando el cambio lo permita.

La posibilidad de sustituir adaptadores facilita las pruebas
y reduce la dependencia de proveedores concretos.

## Consecuencias favorables

- Reglas de negocio identificables y comprobables por separado.
- Casos de uso independientes de las implementaciones técnicas.
- Posibilidad de utilizar adaptadores simulados durante las pruebas.
- Cambios de proveedor o almacenamiento concentrados en los
  adaptadores y en la configuración, si se conserva el contrato.

## Compromisos y condiciones

- Será necesario definir y mantener contratos y adaptadores.
- Los datos externos deberán transformarse al formato del núcleo.
- Los cambios en los contratos pueden afectar a sus consumidores
  y a sus implementaciones.
- Las dependencias deberán revisarse durante el desarrollo.
- La separación de carpetas deberá estar acompañada de una
  separación efectiva de responsabilidades.

## Verificación

En el boilerplate se ejecutó `npm run pruebas` y finalizaron
correctamente las 16 pruebas incluidas. Estas comprobaron reglas
y casos de uso utilizando adaptadores de prueba, sin iniciar
Angular ni un navegador.

La revisión de los archivos de Dominio y Aplicación mostró
dependencias internas hacia el dominio, sin importaciones
de Angular ni de adaptadores concretos.

Para la implementación completa se revisarán las dependencias
y se realizarán pruebas de reglas, casos de uso y adaptadores.
También se comprobará su integración con la API y los servicios
externos.