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

## 5. Usuarios identificados

### 5.1. Admin

Rol interno identificado por el valor `admin`.

La fuente también describe la creación inicial de un usuario administrador al iniciar la aplicación, mediante credenciales obtenidas del entorno.

Los permisos detallados del rol deben confirmarse en la implementación; no se deben inventar.

### 5.2. Seller

Rol interno identificado por el valor `seller`.

El modelo permite asociar una venta con un usuario mediante `sold_by_user_id`, aunque la clave foránea correspondiente aparece como pendiente en la fuente.

No se puede afirmar que el sistema tenga clientes compradores registrados porque el modelo no contiene una entidad de cliente.

## 6. Alcance del producto

### 6.1. Incluido en el alcance derivado

- Clasificación de productos por categorías.
- Datos de productos, precios, existencias e imagen opcional.
- Registro de ventas y líneas de detalle.
- Conservación de valores históricos de los productos vendidos.
- Cálculo de subtotales y totales.
- Identificación del usuario asociado con una venta.
- Retiro lógico de productos.
- Consulta agregada de información de productos para reportes.

### 6.2. No confirmado o fuera del alcance documentado

No se debe presentar como funcionalidad existente:

- Gestión de clientes.
- Gestión de proveedores.
- Órdenes de compra.
- Gestión de categorías mediante CRUD.
- Reportes con desglose por vendedor.
- Soporte para varias monedas.
- Edición o eliminación de ventas registradas.
- Auditoría mediante `created_at` y `updated_at`.
- Funcionalidades específicas de interfaz que no estén descritas en la fuente.

## 7. Restricciones del producto

- La base de datos utiliza PostgreSQL.
- El esquema de persistencia es `sales`.
- Existen cinco categorías iniciales definidas por la fuente.
- El stock no puede ser negativo según una restricción física existente.
- La venta debe tener al menos una línea para poder confirmarse, según la validación del dominio.
- Los importes históricos se conservan en las líneas de venta.
- Los subtotales y totales son valores calculados.
- Los productos se retiran mediante eliminación lógica.
- El modelo utiliza una única moneda, sin campos de moneda.
- Las marcas de tiempo se almacenan con `timestamptz` y se usa UTC.


