# 02 — Domain

## 1. Propósito

Este documento describe el dominio del sistema a partir de las entidades, relaciones y reglas presentes en `spec/data-model.md`.

El dominio representa los conceptos del negocio y sus invariantes, sin confundirlos con las tablas físicas ni con las decisiones específicas de PostgreSQL.

## 2. Lenguaje ubicuo

| Término | Significado en el sistema |
|---|---|
| Category | Clasificación a la que pertenece un producto. |
| Product | Producto del catálogo con nombre, precio, existencias y categoría. |
| Sale | Registro de una venta realizada por un usuario interno. |
| SaleItem | Línea que representa un producto y una cantidad dentro de una venta. |
| User | Usuario interno que opera el sistema. |
| Admin | Rol administrativo. |
| Seller | Rol de vendedor u operador de ventas. |
| Stock | Cantidad disponible registrada para un producto. |
| Historical snapshot | Valores del producto conservados en la línea de venta en el momento de la operación. |
| Soft delete | Retiro lógico de un producto mediante `deleted_at`, sin eliminar físicamente el registro. |
| Subtotal | Importe calculado para una línea de venta. |
| Total | Importe calculado a partir de las líneas de una venta. |

## 3. Entidades del dominio

### 3.1. Category

Representa una categoría utilizada para clasificar productos.

La fuente define cinco categorías iniciales:

- General
- Herramientas
- Electricidad
- Fontanería
- Pinturas

Las categorías se suministran mediante registros iniciales. El modelo no respalda una funcionalidad de creación, edición o eliminación de categorías; por tanto, no debe afirmarse que existe un CRUD de categorías.

**Relación:** una categoría puede asociarse con varios productos mediante `product.category_id`.

### 3.2. Product

Representa un producto del catálogo.

Sus datos relevantes incluyen:

- Nombre.
- Precio.
- Stock.
- Categoría.
- Imagen opcional.
- `deleted_at` para el retiro lógico del catálogo.

El modelo no define atributos de descripción, SKU o referencia comercial. No deben agregarse como atributos existentes sin evidencia.

Reglas identificadas:

- El stock no puede ser negativo; existe una restricción `CHECK` en PostgreSQL identificada como `ck_product_stock_non_negative`.
- La fuente describe la validación de precio positivo como una regla de dominio, pero contiene información contradictoria sobre el estado de su restricción en base de datos. Se debe verificar la sección de restricciones y las tareas pendientes antes de afirmar que también existe un `CHECK` físico.
- El retiro de un producto es lógico, no una eliminación física habitual.

### 3.3. Sale

Representa una venta registrada.

Incluye información de la operación, como la fecha, el usuario asociado y sus líneas de detalle.

Reglas identificadas:

- La venta es inmutable según el modelo.
- No se describen puertos para editar o eliminar una venta existente.
- `EnsureConfirmable` exige que la venta tenga al menos una línea antes de poder confirmarse.
- La relación con el usuario se representa mediante `sold_by_user_id`, aunque la clave foránea correspondiente aparece como pendiente en la fuente.

La inmutabilidad de una venta es una regla del dominio descrito; no implica que PostgreSQL tenga necesariamente una restricción física que impida toda actualización.

### 3.4. SaleItem

Representa una línea perteneciente a una venta.

Relaciona una venta con un producto e incluye la cantidad y una copia de valores históricos del producto.

Entre los valores conservados se encuentran:

- Identificador de la venta.
- Identificador del producto.
- Cantidad.
- Precio unitario histórico.
- Nombre histórico del producto.
- Categoría histórica del producto.

Reglas identificadas:

- La línea existe como parte de una venta.
- El precio y el nombre se congelan en el momento de la operación.
- La categoría histórica se conserva para mantener el contexto de la venta.
- El subtotal se calcula; no se almacena como campo persistido.
- La cantidad debe ser mayor que cero según la regla de dominio descrita.
- La fuente identifica trabajo pendiente relacionado con la restricción física de cantidad; no se debe asumir que existe un `CHECK` en PostgreSQL sin verificar el estado actual.

### 3.5. User

Representa a un usuario interno del sistema.

La información contempla credenciales representadas mediante `password_hash` y un rol.

Roles identificados:

- `admin`
- `seller`

El dominio no debe manejar contraseñas en texto plano. Los hashes no deben exponerse en registros de aplicación.

El modelo no representa a un cliente comprador mediante esta entidad.


