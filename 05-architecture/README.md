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
## 5. Reglas de persistencia

### 5.1. Stock

Existe una restricción `CHECK` identificada como `ck_product_stock_non_negative`, que impide almacenar stock negativo.

Esta regla está implementada en PostgreSQL.

### 5.2. Precio

El precio positivo se describe como regla del dominio. La fuente contiene información contradictoria sobre el estado de la restricción física correspondiente.

No debe afirmarse que existe un `CHECK` para precio positivo sin comprobar el apartado físico y las tareas relacionadas.

### 5.3. Cantidad

La cantidad de una línea debe ser mayor que cero según la regla de dominio. El estado de la restricción física correspondiente debe verificarse en la fuente.

### 5.4. Valores calculados

Los subtotales de las líneas y el total de la venta se calculan a partir de los datos correspondientes.

No se deben documentar como columnas persistidas del modelo.

## 6. Dominio y responsabilidades

### 6.1. Product

Representa el producto y sus datos de catálogo.

Su responsabilidad incluye mantener la información relevante del producto y respetar las reglas descritas para el precio y el stock.

### 6.2. Sale

Representa la venta y controla la regla de confirmación documentada mediante `EnsureConfirmable`.

Una venta debe contener al menos una línea para ser confirmable.

La fuente describe la venta como inmutable y no presenta puertos para editarla o eliminarla.

### 6.3. SaleItem

Representa cada línea de una venta.

Conserva los valores históricos relevantes del producto y la cantidad vendida. El subtotal se deriva de la cantidad y del precio unitario histórico.

### 6.4. Category

Representa las categorías iniciales del catálogo. No debe suponerse una funcionalidad de administración CRUD de categorías si no está descrita en la fuente.

### 6.5. User

Representa usuarios internos y los roles `admin` y `seller`.

El dominio trabaja con `password_hash`, no con contraseñas en texto plano.

## 7. Operaciones y flujo de venta

La operación descrita de `Sale.AddItem` retira stock antes de añadir la línea a la venta.

El flujo conceptual de una venta es:

1. Identificar el producto y la cantidad.
2. Aplicar las validaciones correspondientes.
3. Retirar el stock según el comportamiento documentado.
4. Crear la línea conservando los valores históricos del producto.
5. Verificar que la venta tenga al menos una línea antes de confirmarla.
6. Calcular los subtotales y el total a partir de las líneas.

Este flujo resume el comportamiento del dominio. La fuente no permite especificar todos los detalles de transacciones, concurrencia, reversión de operaciones o tratamiento de fallos. Esos detalles deben confirmarse antes de documentarlos como decisiones de arquitectura.

## 8. Puertos y adaptadores

La evaluación solicita reconstruir la arquitectura a partir de la fuente. En consecuencia, esta sección diferencia las responsabilidades inferibles de las interfaces concretas.

### 8.1. Responsabilidades identificadas

- El dominio contiene reglas de entidades como `Sale` y `SaleItem`.
- PostgreSQL conserva las entidades y relaciones descritas.
- Las consultas agregadas permiten obtener información para reportes.
- Las reglas físicas y las reglas del dominio no son equivalentes.

### 8.2. Interfaces no confirmadas

No se declaran nombres concretos de repositorios, controladores, servicios de aplicación o adaptadores cuando no pueden verificarse en `spec/data-model.md`.

Si la fuente define puertos o interfaces explícitas en otra sección, deben documentarse con sus nombres y responsabilidades exactas, sin reemplazarlos por interfaces inventadas.

## 9. Reportes y consultas

El modelo contempla información agregada para reportes de productos.

Los resultados agregados se calculan en la base de datos y no se persisten como entidades independientes.

La fuente no contempla un desglose por vendedor.

Existe una decisión pendiente sobre la agrupación por categoría histórica: si el nombre de una categoría cambia, el reporte puede generar filas diferentes para nombres históricos distintos. Esta decisión debe contrastarse con la especificación de reporte señalada en el modelo y confirmarse con el responsable del sistema.

## 10. Fechas, moneda y configuración

### 10.1. Fechas

Las marcas de tiempo utilizan `timestamptz`, con UTC como referencia del servidor.

### 10.2. Moneda

El modelo no incorpora campos de moneda. La solución descrita es monomoneda.

### 10.3. Inicialización

La fuente indica que el usuario administrador inicial se crea al iniciar la aplicación mediante credenciales provenientes del entorno, no mediante una inserción SQL de inicialización.

La forma exacta de desplegar, almacenar o rotar esas credenciales debe documentarse únicamente si está descrita en la fuente.

## 11. Integridad, privacidad y conservación

- El stock persistido no puede ser negativo.
- Las ventas y sus líneas conservan información histórica de las operaciones.
- Los productos se retiran mediante eliminación lógica con `deleted_at`.
- Los hashes de contraseñas no deben exponerse en registros de aplicación.
- No se deben introducir datos de clientes como parte de la arquitectura porque el modelo no define una entidad de cliente.
- No se deben afirmar campos `created_at` o `updated_at`, ya que la fuente indica que no existen.

## 12. Decisiones pendientes y deuda técnica

| Elemento | Situación que debe verificarse |
|---|---|
| Precio positivo | La fuente presenta información contradictoria sobre su restricción física. |
| Cantidad positiva | Confirmar el estado final de la restricción física. |
| FK de `sale_item` a `product` | Verificar el estado final de T-20 y la definición física. |
| FK de `sale` a `user` | Aparece como pendiente en la fuente mediante T-12. |
| Agrupación de reportes por categoría | Existe una decisión pendiente sobre nombres históricos y renombrados. |
| Transacciones y concurrencia | No especificar comportamiento que no esté confirmado en la fuente. |

Los identificadores de tarea deben comprobarse directamente en `spec/data-model.md`. Esta tabla no afirma que las tareas estén resueltas.

## 13. Fuente de referencia

Toda la arquitectura documentada se deriva de `spec/data-model.md`, especialmente de las definiciones del modelo físico, las relaciones, las reglas del dominio, los reportes y las tareas pendientes.




