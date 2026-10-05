# Atributos de calidad

Durante una campaña comercial, el marketplace puede recibir muchas
consultas y compras simultáneas. Por eso, además de cumplir sus funciones,
debe responder adecuadamente, mantenerse disponible y proteger
la información de los usuarios.

## Atributos y escenarios de calidad

| ID | Atributo | Escenario de calidad | Criterio de evaluación propuesto |
|---|---|---|---|
| AC01 | Rendimiento | Durante una campaña comercial, muchos clientes consultan productos y modifican sus carritos al mismo tiempo. El sistema debe responder sin demoras excesivas. | En una prueba con 100 usuarios concurrentes, al menos el 95 % de las consultas de productos y operaciones del carrito debe responder en un máximo de 2 segundos. |
| AC02 | Disponibilidad | Los clientes necesitan consultar productos y realizar compras durante el horario de atención de la plataforma. El sistema debe mantenerse operativo. | Alcanzar una disponibilidad mensual de al menos 99,5 %, medida mediante comprobaciones periódicas del servicio. |
| AC03 | Escalabilidad | La cantidad de usuarios aumenta durante una campaña. El sistema debe permitir ampliar su capacidad para atender la nueva demanda. | Al pasar de 100 a 200 usuarios concurrentes y aumentar los recursos del servidor o las instancias de la aplicación, mantener el objetivo de respuesta de AC01. |
| AC04 | Seguridad | Un usuario intenta acceder a pedidos ajenos o ejecutar funciones que no corresponden a su rol. El sistema debe impedirlo y proteger los datos. | Rechazar todos los intentos de acceso no autorizado incluidos en las pruebas; utilizar HTTPS y almacenar contraseñas mediante un algoritmo de hash adecuado para contraseñas. |
| AC05 | Mantenibilidad | Se necesita modificar una regla de cálculo del carrito. La organización del sistema debe facilitar el cambio y evitar efectos innecesarios en otros módulos. | Realizar el cambio dentro de la lógica de negocio correspondiente, sin modificar la presentación ni el acceso a datos si sus interfaces se mantienen, y superar las pruebas de las funciones afectadas. |

## Relación con el diseño

- Rendimiento: influye en las consultas a la base de datos, la paginación
  del catálogo y el procesamiento de las solicitudes.
- Disponibilidad: requiere detectar fallos, facilitar la recuperación
  y controlar los errores de los servicios externos.
- Escalabilidad: exige considerar cómo ampliar los recursos y ejecutar
  varias instancias de la aplicación cuando sea necesario.
- Seguridad: requiere autenticación, autorización por roles, validación
  de datos y protección de las comunicaciones.
- Mantenibilidad: favorece la separación de responsabilidades entre
  capas y módulos con interfaces claras.

## Consideraciones

Los valores de concurrencia, tiempo de respuesta y disponibilidad
son metas propuestas para este proyecto académico. La guía no fija
estas cifras y todavía no se han realizado pruebas para verificarlas.

Las pruebas de rendimiento deberán documentar el entorno, el volumen
de datos y las operaciones ejecutadas para que sus resultados
puedan interpretarse correctamente.

Los tiempos de aprobación de pagos y de actualización de envíos
también dependen de los servicios externos, por lo que se evaluarán
por separado.