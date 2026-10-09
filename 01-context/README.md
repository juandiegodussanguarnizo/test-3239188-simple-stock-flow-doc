# 01 — Context

## 1. Propósito

Este documento presenta el contexto general del sistema de gestión de productos, inventario y ventas descrito en `spec/data-model.md`.

Su objetivo es establecer qué información administra el sistema, cuáles son sus límites y qué actores se pueden identificar a partir de la fuente.

## 2. Descripción general

El sistema permite organizar productos mediante categorías, mantener información de precios y existencias, registrar ventas y conservar el detalle de los productos vendidos.

También contempla usuarios internos con roles de administración y venta.

El modelo de datos se encuentra en el esquema `sales` de PostgreSQL y contiene cinco entidades principales:

- `category`
- `product`
- `sale`
- `sale_item`
- `user`

**Fuente:** `spec/data-model.md`, definición de entidades y relaciones.

## 3. Contexto de uso

A partir del modelo se identifican dos roles internos:

- **Admin:** usuario con rol administrativo.
- **Seller:** usuario con rol de vendedor u operador de ventas.

La fuente no permite afirmar cuáles son todas las pantallas, permisos detallados o procesos de autenticación disponibles en la interfaz.

No se identifica una entidad de cliente o comprador. Por tanto, no se debe asumir que el sistema mantiene perfiles, direcciones o historiales individuales de clientes.

## 4. Alcance funcional identificado

El modelo de datos respalda las siguientes capacidades:

1. Organizar productos por categorías.
2. Mantener información de productos, precios y existencias.
3. Registrar una venta y asociarla con el usuario que la realiza, de acuerdo con el campo `sold_by_user_id`.
4. Registrar las líneas de una venta, indicando productos y cantidades.
5. Conservar el nombre, precio y categoría del producto tal como estaban representados en la línea de venta.
6. Calcular subtotales y totales sin almacenarlos como campos persistidos.
7. Consultar información agregada para reportes de productos.
8. Retirar productos del catálogo mediante eliminación lógica.
9. Distinguir usuarios internos por los roles `admin` y `seller`.

El alcance anterior describe capacidades derivadas del modelo, no una confirmación de que exista una interfaz completa para todas ellas.

