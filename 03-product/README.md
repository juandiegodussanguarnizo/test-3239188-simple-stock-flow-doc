# 03 — Product

## 1. Propósito

Este documento describe el problema que aborda el sistema, su visión y el alcance del producto reconstruido a partir de `spec/data-model.md`.

Las declaraciones de visión y objetivos que no aparezcan literalmente en el modelo se identifican como formulaciones derivadas, no como requisitos textuales originales.

## 2. Problema identificado

El modelo representa información de productos, categorías, existencias, ventas, líneas de venta y usuarios internos.

Sin una estructura de datos coherente para estos conceptos, sería difícil mantener relaciones entre los productos vendidos, las cantidades registradas y los valores históricos de cada operación.

El modelo proporciona una estructura para registrar ventas y conservar información histórica de los productos involucrados.

**Trazabilidad:** entidades `category`, `product`, `sale`, `sale_item` y `user` en `spec/data-model.md`.

## 3. Visión del producto

**Visión derivada:** disponer de un sistema que permita administrar un catálogo de productos, registrar sus ventas, mantener la consistencia de las existencias y conservar el detalle histórico de las operaciones realizadas por usuarios internos.

Esta visión resume las capacidades respaldadas por el modelo de datos. No representa una declaración de visión original confirmada por el propietario del producto.

## 4. Objetivos del producto

Los siguientes objetivos se derivan del modelo:

1. Mantener un catálogo de productos clasificados por categorías.
2. Registrar y consultar información de precios y existencias.
3. Conservar el detalle de los productos y cantidades incluidos en cada venta.
4. Preservar los valores históricos relevantes de una operación.
5. Identificar al usuario interno asociado con una venta.
6. Calcular subtotales y totales a partir de las líneas registradas.
7. Evitar que las existencias persistidas sean negativas.
8. Retirar productos del catálogo sin eliminarlos físicamente.

Estos objetivos no implican que exista una interfaz específica ni que todas las operaciones estén expuestas mediante pantallas o API.

