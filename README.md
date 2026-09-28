# Marketplace de productos para mascotas

## Datos académicos

- Estudiante: Sebastian Cahuin.
- Curso: Arquitectura de Software — IS-488.
- Docente: Ing. Lizbeth Jaico Quispe.
- Universidad: Universidad Nacional de San Cristóbal de Huamanga.
- Semestre: 2026-II.

## Descripción

Este repositorio contiene el análisis y la propuesta arquitectónica
de un marketplace de productos para mascotas.

La plataforma permitirá que distintos vendedores ofrezcan sus
productos y que los clientes puedan consultarlos, agregarlos
al carrito y realizar pedidos.

El trabajo comprende la arquitectura inicial de la Guía 02
y su evolución en la Guía 03 hacia un backend organizado
como monolito modular con el enfoque Clean Architecture.

## Objetivo del trabajo

Analizar las necesidades del marketplace y justificar su arquitectura
mediante requisitos, atributos de calidad, restricciones,
drivers y decisiones arquitectónicas.

La propuesta parte de una organización inicial en tres capas
y precisa sus módulos, integraciones y dependencias internas.

## Análisis del caso de negocio

### Situación planteada

Una empresa dedicada a la venta y distribución de alimentos
y artículos para mascotas desea contar con un marketplace
donde distintos vendedores puedan ofrecer sus productos
y los clientes puedan realizar compras.

### Problema que se busca resolver

La empresa necesita una plataforma que reúna la oferta
de distintos vendedores y permita organizar la consulta
de productos, los pedidos, los pagos y la información de entrega.

### Objetivos del negocio

- Reunir la oferta de distintos vendedores en una plataforma.
- Facilitar la consulta de productos y la realización de compras.
- Permitir que cada vendedor gestione su oferta y consulte sus ventas.
- Coordinar los pedidos con pagos, facturación y servicios de envío.

### Solución propuesta

Se propone una aplicación web donde los clientes puedan buscar
productos, consultar sus características y disponibilidad,
agregarlos al carrito y realizar pedidos.

Los vendedores podrán registrar y actualizar sus productos,
además de consultar la información de sus ventas.

El administrador gestionará los vendedores y supervisará
la plataforma, de acuerdo con los permisos definidos.

### Alcance funcional

- Consulta de productos y disponibilidad.
- Registro y actualización de productos por parte de los vendedores.
- Gestión del carrito de compras.
- Registro y consulta de pedidos y direcciones de entrega.
- Gestión de cuentas y acceso según el tipo de usuario.
- Administración de vendedores y consulta de sus ventas.
- Integración con una pasarela de pago.
- Integración con un servicio de envío.
- Integración con un servicio de facturación.
- Consulta de información de productos y stock mediante un ERP.

### Referencia funcional

Las guías utilizan GoPet como referencia para comprender
el negocio de venta de productos para mascotas.

La arquitectura documentada corresponde a una propuesta
académica propia.

## Organización del repositorio

- `analisis-de-sistema/`: actores, historias de usuario, requisitos
  funcionales, atributos de calidad, restricciones y drivers.
- `arquitectura/`: arquitectura inicial y estilo arquitectónico.
- `arquitectura/decisiones/`: registros de decisiones arquitectónicas.
- `arquitectura/enfoque/`: responsabilidades y dependencias
  internas mediante Clean Architecture.
- `boilerplate/`: proyecto de referencia de la docente.
- `README.md`: presentación e índice del trabajo.
- `.gitignore`: reglas para excluir archivos generados y temporales.

## Entregables de la Guía 03

| Entregable | Ubicación |
|---|---|
| 1. Necesidad del negocio | Sección “Análisis del caso de negocio” de este README. |
| 2. Requisitos | [Actores](analisis-de-sistema/01-actores.md), [historias de usuario](analisis-de-sistema/02-historias-de-usuario.md), [requisitos funcionales](analisis-de-sistema/03-requisitos-funcionales.md) y [restricciones](analisis-de-sistema/05-restricciones.md). |
| 3. Atributos de calidad | [Atributos y escenarios de calidad](analisis-de-sistema/04-atributos-de-calidad.md). |
| 4. Drivers arquitectónicos | [Drivers y evolución de la propuesta](analisis-de-sistema/06-drivers-arquitectonicos.md). |
| 5. Decisiones arquitectónicas | [Registros ADR](arquitectura/decisiones/). |
| 6. Estilo arquitectónico | [Estilo y diagrama global](arquitectura/estilo-arquitectonico.md). |
| Enfoque arquitectónico | [Clean Architecture y diagrama de dependencias](arquitectura/enfoque/enfoque-arquitectonico.md). |

La [arquitectura inicial de la Guía 02](arquitectura/arquitectura-inicial.md)
se conserva como antecedente.

## Decisiones arquitectónicas

| Registro | Decisión |
|---|---|
| [ADR-001](arquitectura/decisiones/ADR-001-monolito-modular.md) | Organizar el backend como un monolito modular. |
| [ADR-002](arquitectura/decisiones/ADR-002-clean-architecture.md) | Aplicar Clean Architecture. |
| [ADR-003](arquitectura/decisiones/ADR-003-estrategia-cache.md) | Incorporar caché para consultas públicas del catálogo. |
| [ADR-004](arquitectura/decisiones/ADR-004-integracion-pagos.md) | Integrar pagos mediante interfaces y adaptadores. |

## Proyecto de referencia

El código de `boilerplate/` corresponde al ejemplo de la docente.
Se utiliza para ejecutar la aplicación, revisar las pruebas
y estudiar la separación de responsabilidades.

Su repositorio original, el fork y el commit de procedencia
se encuentran en [ORIGEN.md](boilerplate/ORIGEN.md).

La carpeta se incorporó mediante una descarga ZIP del fork.
Sus archivos constituyen una copia dentro de este repositorio.

## Ejecución del ejemplo

Entorno utilizado:

- Node.js: 22.23.3.
- npm: 10.9.9.
- Angular: versión 18, según las dependencias del proyecto.

Desde la raíz del repositorio, con Node.js 22 activo:

```sh
cd boilerplate
npm ci
npm run pruebas
npm start
```

Después del inicio del servidor, abrir la dirección indicada
en la terminal, normalmente:

http://localhost:4200

Mantener la terminal abierta mientras se utiliza la aplicación.
Para detener el servidor, presionar Ctrl + C.

## Verificaciones realizadas

- Ejecución satisfactoria de las 16 pruebas incluidas.
- Apertura del catálogo en el navegador.
- Revisión de responsabilidades y dependencias del ejemplo.
- Identificación de contratos, adaptadores y configuración.

El ejemplo utiliza pagos simulados y repositorios en memoria.
Los resultados obtenidos corresponden a ese entorno de referencia.

## Estado del trabajo

Se han documentado los seis entregables solicitados en la Guía 03
y el enfoque Clean Architecture.

La implementación completa del marketplace, las integraciones
reales y la comprobación de los atributos de calidad corresponden
a etapas posteriores.

Las metas de rendimiento, disponibilidad y escalabilidad
son propuestas pendientes de validación mediante pruebas.