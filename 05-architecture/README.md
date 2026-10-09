# 05 — Architecture

## 1. Propósito

Este documento reconstruye la arquitectura identificable a partir de `spec/data-model.md`.

La descripción se centra en el modelo de persistencia, las entidades del dominio, las reglas y las responsabilidades que pueden inferirse de las operaciones documentadas.

No se afirma la existencia de controladores HTTP, interfaces gráficas, microservicios o mecanismos de mensajería que no estén respaldados por la fuente.

## 2. Vista general

La solución descrita utiliza PostgreSQL para almacenar la información del sistema en el esquema `sales`.

Las responsabilidades identificadas pueden organizarse conceptualmente en tres áreas:

1. **Dominio:** representa las entidades y aplica reglas como la confirmabilidad de una venta.
2. **Persistencia:** conserva productos, categorías, ventas, líneas de venta y usuarios.
3. **Consultas y reportes:** obtiene información derivada, incluidos agregados de productos.

Esta separación es una representación conceptual para documentar el modelo. La fuente debe consultarse para confirmar las clases, interfaces y componentes concretos de la implementación

## 3. Modelo de persistencia

### 3.1. Esquema

- Motor de persistencia: PostgreSQL.
- Esquema: `sales`.
- Tablas principales: `category`, `product`, `sale`, `sale_item` y `user`.

### 3.2. Entidades y responsabilidades

| Tabla | Responsabilidad |
|---|---|
| `category` | Mantener las categorías utilizadas por los productos. |
| `product` | Conservar información del catálogo, precio, stock, categoría e imagen opcional. |
| `sale` | Registrar la operación de venta y su información asociada. |
| `sale_item` | Registrar productos, cantidades y valores históricos de una venta. |
| `user` | Representar usuarios internos y sus roles. |

## 4. Relaciones de persistencia

### 4.1. Producto y categoría

`product.category_id` referencia la categoría asociada al producto.

La fuente identifica una política `RESTRICT` para esta relación, por lo que no debe describirse como una relación que permita eliminar una categoría referenciada sin restricciones.

### 4.2. Venta y líneas

` sale_item.sale_id` referencia `sale`.

La fuente describe una relación con comportamiento `CASCADE` para esta clave foránea. Su alcance es la relación física entre venta y líneas, y no debe interpretarse como autorización para editar o eliminar ventas desde el dominio.

### 4.3. Línea y producto

` sale_item` referencia al producto correspondiente.

La fuente indica que la clave foránea de producto para las líneas se incorpora mediante la tarea T-20. Se debe comprobar el estado final de esa tarea y la sección del modelo físico antes de afirmar su estado actual sin reservas.

### 4.4. Venta y usuario

` sale.sold_by_user_id` identifica al usuario asociado con la venta.

La clave foránea hacia `user` aparece como pendiente en la fuente. La documentación debe distinguir la existencia del campo de la existencia efectiva de la restricción física.




