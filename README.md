# Marketplace de productos para mascotas

## Estudiante
Sebastian Cahuin

## Descripción
Este proyecto presenta el análisis y el diseño inicial de un marketplace
de productos para mascotas. La plataforma permitirá que distintos
vendedores ofrezcan sus productos y que los clientes puedan consultarlos,
agregarlos al carrito y realizar pedidos.

## Caso de estudio
Se toma GoPet como referencia funcional para comprender el negocio
de venta de productos para mascotas.

## Curso
Arquitectura de Software — IS-488

## Docente
Ing. Lizbeth Jaico Quispe

## Universidad
Universidad Nacional de San Cristóbal de Huamanga

## Semestre
2026-II

## Objetivo
Analizar las necesidades del marketplace y proponer una arquitectura
inicial de tres capas: presentación, lógica de negocio y datos.

## Organización del repositorio
- analisis-de-sistema/: actores, historias de usuario, requisitos
  funcionales, atributos de calidad, restricciones y drivers arquitectónicos.
- arquitectura/: diagrama y descripción de la arquitectura inicial.
- README.md: presentación general del proyecto.
- .gitignore: reglas para excluir archivos del control de versiones.

## Análisis del caso de negocio

### Situación planteada
Una empresa dedicada a la venta y distribución de alimentos y artículos
para mascotas desea contar con un marketplace donde distintos vendedores
puedan ofrecer sus productos y los clientes puedan realizar compras.

### Problema que se busca resolver
La empresa necesita una plataforma que reúna la oferta de distintos
vendedores y permita organizar la consulta de productos, los pedidos,
los pagos y la información de entrega.

### Solución propuesta
Se propone una aplicación web donde los clientes puedan buscar productos,
consultar sus características y disponibilidad, agregarlos al carrito
y realizar pedidos.

Los vendedores podrán registrar y actualizar sus productos, además de
consultar la información de sus ventas. El administrador gestionará
los vendedores y supervisará la plataforma.

### Alcance inicial
- Consulta de productos y disponibilidad.
- Registro y actualización de productos por parte de los vendedores.
- Gestión del carrito de compras.
- Registro y consulta de pedidos.
- Administración de vendedores y acceso según el tipo de usuario.
- Integración con una pasarela de pago y un servicio de envío.
- Integración con un servicio de facturación para generar comprobantes.
- Consulta de información de productos y stock mediante un ERP.

### Referencia funcional
La guía propone GoPet como referencia para comprender las funciones
de una plataforma de productos para mascotas. Nuestra arquitectura
será una propuesta académica propia.
