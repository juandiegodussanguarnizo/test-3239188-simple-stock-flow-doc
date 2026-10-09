# 04 — Requirements

## 1. Propósito

Este documento reúne los requisitos funcionales y no funcionales que pueden derivarse de `spec/data-model.md`.

Los requisitos de este archivo son una reconstrucción documental. Sus identificadores se utilizan para facilitar la trazabilidad y no deben confundirse con identificadores oficiales de la fuente.

## 2. Convenciones

- **Derivado:** capacidad o regla respaldada por el modelo de datos.
- **Implementado en BD:** existe evidencia de una restricción o estructura física en PostgreSQL.
- **Validado en dominio:** la regla se describe en el dominio de la aplicación.
- **Pendiente / por verificar:** la fuente indica trabajo pendiente o presenta información contradictoria.

No se asignan métricas de rendimiento ni acuerdos de servicio porque el modelo no proporciona esos valores.

## 3. Requisitos funcionales

### RF-01. Clasificación de productos

**Descripción:** el sistema debe permitir asociar los productos con una categoría.

**Trazabilidad:** `product.category_id` y entidad `category`.

**Estado:** estructura de relación descrita en el modelo.

### RF-02. Catálogo de productos

**Descripción:** el sistema debe representar productos con nombre, precio, existencias, categoría e imagen opcional.

**Trazabilidad:** entidad `product`.

**Restricción:** no se deben añadir como existentes atributos no definidos, como SKU o descripción.

### RF-03. Control de stock no negativo

**Descripción:** el sistema debe impedir que el stock persistido de un producto sea negativo.

**Trazabilidad:** restricción `ck_product_stock_non_negative`.

**Estado:** implementado en PostgreSQL.

### RF-04. Validación de precio positivo

**Descripción:** el precio del producto debe ser positivo.

**Trazabilidad:** regla de precio descrita en las secciones de dominio del modelo.

**Estado:** validación de dominio; el estado de la restricción física requiere verificación debido a información contradictoria en la fuente.

### RF-05. Registro de ventas

**Descripción:** el sistema debe representar una venta con la información de la operación, incluida su fecha y el usuario asociado.

**Trazabilidad:** entidad `sale` y campo `sold_by_user_id`.

**Estado:** la relación lógica está descrita; la clave foránea hacia `user` figura como pendiente.

### RF-06. Registro de líneas de venta

**Descripción:** el sistema debe representar cada producto y cantidad incluidos en una venta mediante `sale_item`.

**Trazabilidad:** entidad `sale_item` y su relación con `sale` y `product`.

### RF-07. Validación de cantidad positiva

**Descripción:** la cantidad de una línea de venta debe ser mayor que cero.

**Trazabilidad:** regla de dominio asociada a `SaleItem`.

**Estado:** regla de dominio; la restricción física debe verificarse según las tareas pendientes de la fuente.

### RF-08. Confirmación de ventas

**Descripción:** una venta solo debe poder confirmarse cuando contiene al menos una línea.

**Trazabilidad:** método `Sale.EnsureConfirmable`.

**Estado:** validado en el dominio.

### RF-09. Conservación del precio histórico

**Descripción:** cada línea de venta debe conservar el precio del producto utilizado en la operación, sin cambiar automáticamente cuando se actualiza el catálogo.

**Trazabilidad:** valores históricos de `sale_item`.

### RF-10. Conservación de datos históricos

**Descripción:** cada línea de venta debe conservar el nombre y la categoría históricos del producto según lo descrito por el modelo.

**Trazabilidad:** campos históricos de `sale_item`.

### RF-11. Cálculo de subtotales

**Descripción:** el subtotal de una línea debe calcularse a partir de su cantidad y precio unitario histórico.

**Trazabilidad:** reglas de cálculo de `SaleItem`.

**Restricción:** el subtotal no se almacena como campo persistido.

### RF-12. Cálculo del total de la venta

**Descripción:** el total de una venta debe obtenerse a partir de sus líneas.

**Trazabilidad:** reglas de cálculo de la venta.

**Restricción:** el total no se almacena como campo persistido.

### RF-13. Inmutabilidad de las ventas

**Descripción:** una venta registrada debe tratarse como inmutable de acuerdo con las reglas del dominio.

**Trazabilidad:** definición de `Sale` y ausencia de puertos para editar o eliminar ventas.

**Estado:** regla del dominio; no implica por sí sola la existencia de una restricción física que impida cualquier actualización.

### RF-14. Retiro lógico de productos

**Descripción:** el sistema debe permitir representar el retiro de un producto mediante `deleted_at`, sin eliminar físicamente su registro.

**Trazabilidad:** entidad `product`.

### RF-15. Roles de usuario

**Descripción:** el modelo debe distinguir los roles internos `admin` y `seller`.

**Trazabilidad:** entidad `user` y sus valores de rol.

**Restricción:** los permisos detallados de cada rol no se especifican aquí porque la fuente no los define por completo.

### RF-16. Protección de credenciales

**Descripción:** el dominio debe trabajar con la representación hash de las contraseñas mediante `password_hash` y no con contraseñas en texto plano.

**Trazabilidad:** entidad `user` y reglas de credenciales.

### RF-17. Reporte agregado de productos

**Descripción:** el sistema debe poder obtener información agregada para reportes de productos, según las consultas descritas en la fuente.

**Trazabilidad:** sección de reportes de `spec/data-model.md`.

**Restricción:** no se debe afirmar que el reporte presenta un desglose por vendedor, porque la fuente no lo contempla.

### RF-18. Inicialización del administrador

**Descripción:** la aplicación contempla crear un usuario administrador inicial al iniciar, utilizando credenciales provenientes del entorno.

**Trazabilidad:** sección de inicialización/configuración de `spec/data-model.md`.

**Restricción:** la fuente indica que esta inicialización no se realiza mediante una inserción SQL de usuario administrador.

## 4. Requisitos no funcionales

### RNF-01. Persistencia relacional

**Descripción:** la persistencia descrita debe utilizar PostgreSQL.

**Trazabilidad:** definición del modelo físico.

### RNF-02. Consistencia de inventario

**Descripción:** la persistencia debe mantener la restricción de stock no negativo.

**Trazabilidad:** `ck_product_stock_non_negative`.

### RNF-03. Conservación histórica

**Descripción:** la información histórica de las líneas de venta debe preservarse frente a cambios posteriores del catálogo.

**Trazabilidad:** campos históricos de `sale_item`.

### RNF-04. Manejo seguro de credenciales

**Descripción:** el dominio no debe manejar contraseñas en texto plano y los hashes no deben exponerse en registros de aplicación.

**Trazabilidad:** reglas de `user` y privacidad de `spec/data-model.md`.

### RNF-05. Tratamiento de fechas

**Descripción:** las marcas de tiempo deben representarse mediante `timestamptz`, utilizando UTC según la configuración descrita.

**Trazabilidad:** convenciones de fecha y hora del modelo.

### RNF-06. Consistencia monetaria

**Descripción:** el modelo debe tratar los valores monetarios bajo una única moneda, dado que no existen campos de moneda.

**Trazabilidad:** estructura de precios e importes.

### RNF-07. Retención lógica

**Descripción:** la retirada de productos debe conservar el registro físico mediante `deleted_at`.

**Trazabilidad:** entidad `product`.

### RNF-08. Trazabilidad documental

**Descripción:** cada requisito derivado debe poder relacionarse con una entidad, regla, relación o decisión identificable de la fuente.

**Trazabilidad:** criterio de la evaluación SDD.

No se establecen objetivos de disponibilidad, tiempos de respuesta, capacidad concurrente ni métricas de rendimiento porque la fuente no aporta valores para ellos.

## 5. Requisitos pendientes o por verificar

| Tema | Situación descrita | Acción documental |
|---|---|---|
| Precio positivo | Hay información contradictoria sobre su restricción física. | Verificar la sección de restricciones y tareas de la fuente. |
| Cantidad positiva | La regla de dominio existe; la restricción física requiere confirmación. | No afirmar que existe un `CHECK` sin evidencia. |
| FK del usuario en la venta | `sale.sold_by_user_id` aparece como pendiente. | Marcar la relación física como pendiente. |
| Reporte por categoría | Hay una decisión pendiente sobre nombres históricos y renombrados. | Registrar la discrepancia y no resolverla por suposición. |

