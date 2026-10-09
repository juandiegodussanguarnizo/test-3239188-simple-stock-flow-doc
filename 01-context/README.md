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

## 5. Fuera del alcance confirmado

No existe evidencia suficiente en el modelo para afirmar que el sistema incluye:

- Gestión de clientes.
- Registro de datos personales de compradores.
- Gestión de proveedores.
- Órdenes de compra o reposición de inventario.
- Múltiples monedas.
- Desglose de reportes por vendedor.
- Auditoría mediante campos `created_at` y `updated_at`.
- Edición o eliminación de ventas ya registradas.
- Eliminación física de productos como operación habitual del catálogo.

Estos puntos no deben documentarse como funcionalidades existentes sin una fuente adicional.

## 6. Límites del sistema

### 6.1. Persistencia

PostgreSQL almacena las categorías, productos, ventas, líneas de venta y usuarios.

### 6.2. Reglas de negocio

Algunas reglas se encuentran implementadas mediante restricciones de base de datos, mientras que otras se validan únicamente en el dominio.

La documentación debe distinguir ambos casos.

### 6.3. Información histórica

Las líneas de venta conservan valores históricos del producto. Por ello, un cambio posterior en el catálogo no debe reescribir automáticamente la información registrada en ventas anteriores.

### 6.4. Credenciales

La información de usuarios contempla `password_hash`. El modelo no justifica almacenar ni procesar contraseñas en texto plano dentro del dominio.

## 7. Restricciones y consideraciones

- El sistema utiliza PostgreSQL.
- Las marcas de tiempo se almacenan con tipo `timestamptz` y el servidor utiliza UTC.
- El modelo es monomoneda: no contiene campos para moneda.
- Las categorías iniciales corresponden a cinco registros definidos en la fuente.
- Los productos se retiran mediante eliminación lógica con `deleted_at`.
- La fuente indica que el usuario administrador inicial se crea al iniciar la aplicación mediante credenciales provenientes del entorno, no mediante una inserción SQL de inicialización.

## 8. Incertidumbres identificadas

Hay decisiones y tareas pendientes que no deben presentarse como resueltas sin confirmación:

- El estado de algunas restricciones de integridad, especialmente las relacionadas con el precio y la cantidad.
- La clave foránea de `sale.sold_by_user_id` hacia `user`, identificada como pendiente en la fuente.
- La política definitiva para agrupar reportes por el nombre histórico de categoría cuando una categoría se renombra.
- Cualquier funcionalidad de interfaz o permiso no descrito por el modelo.

Las discrepancias deben contrastarse con las secciones pertinentes de `spec/data-model.md` antes de cerrar la documentación.

## 9. Fuente de referencia

La descripción de este contexto se deriva de `spec/data-model.md`, particularmente de las definiciones de entidades, relaciones, reglas de negocio, configuración y tareas pendientes.
