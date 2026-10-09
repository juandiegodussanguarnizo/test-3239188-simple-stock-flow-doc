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
## 4. Relaciones principales

| Relación | Significado |
|---|---|
| Category → Product | Una categoría puede clasificar varios productos. |
| Sale → SaleItem | Una venta contiene sus líneas de detalle. |
| Product → SaleItem | Una línea hace referencia al producto asociado. |
| User → Sale | Una venta se asocia con el usuario que la realiza mediante `sold_by_user_id`; la FK aparece pendiente en la fuente. |

Las relaciones anteriores representan el significado de negocio. El estado exacto de cada clave foránea debe verificarse en la sección física del modelo de datos.

## 5. Reglas de negocio

| ID | Regla | Estado o evidencia |
|---|---|---|
| DOM-01 | El stock de un producto no puede ser negativo. | Implementada en PostgreSQL mediante `ck_product_stock_non_negative`. |
| DOM-02 | El precio del producto debe ser positivo. | Regla de dominio; estado físico contradictorio en la fuente, requiere verificación. |
| DOM-03 | La cantidad de una línea debe ser mayor que cero. | Regla de dominio; restricción física pendiente o por verificar. |
| DOM-04 | Una venta debe tener al menos una línea para ser confirmable. | Validada mediante `Sale.EnsureConfirmable`. |
| DOM-05 | La venta es inmutable. | Regla del dominio; no se describen puertos de edición o eliminación. |
| DOM-06 | La línea conserva el precio histórico del producto. | Regla de conservación de datos de la venta. |
| DOM-07 | La línea conserva el nombre y la categoría históricos. | Regla de conservación de datos de la venta. |
| DOM-08 | El subtotal y el total se calculan, no se persisten como campos. | Cálculo derivado de las líneas. |
| DOM-09 | La retirada de un producto es lógica. | Se utiliza `deleted_at`. |
| DOM-10 | Las contraseñas se representan mediante un hash. | El dominio no maneja contraseñas en texto plano. |

Los identificadores DOM-01 a DOM-10 son identificadores documentales para esta reconstrucción; no se deben confundir con identificadores oficiales de la fuente.

## 6. Ciclo de una venta

A partir de las reglas documentadas, el proceso de negocio puede resumirse así:

1. Se identifica el producto que se venderá.
2. Se determina la cantidad solicitada.
3. La operación de agregar una línea retira stock antes de incorporar la línea, según el comportamiento descrito de `Sale.AddItem`.
4. Se conserva en la línea el precio, nombre y categoría históricos del producto.
5. Antes de confirmar la venta, `Sale.EnsureConfirmable` verifica que exista al menos una línea.
6. El subtotal de cada línea y el total de la venta se calculan a partir de sus datos.

Este resumen representa las reglas descritas, no una especificación completa de transacciones, concurrencia o recuperación ante fallos. Esos detalles deben documentarse solo si la fuente los confirma.

