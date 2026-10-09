# 02 — Domain Model

## 1. Purpose

This document defines the business language, entities, invariants, and domain behavior identified in [`spec/data-model.md`](../spec/data-model.md).

It describes the domain independently of any unconfirmed user interface, API, or deployment architecture.

## 2. Domain glossary

| Term | Definition | Source representation |
|---|---|---|
| Product | An item in the catalog with a name, price, stock quantity, category, and optional image key. | `Product` / `product` |
| Category | A classification assigned to products. The source defines five seeded categories without category maintenance operations. | `Category` / `category` |
| Money | A value object representing a monetary amount. Its documented behavior includes rounding to two decimal places. | `Money` |
| Stock | The available quantity of a product. Persisted stock must not be negative. | `product.stock` |
| Sale | An immutable business record representing a completed sale. | `Sale` / `sale` |
| Sale item | A line belonging to a sale, with a product, quantity, and historical values. | `SaleItem` / `sale_item` |
| Quantity | A value object representing the quantity of items sold. The domain requires a positive value. | `Quantity` |
| Line subtotal | The historical unit price multiplied by the line quantity. It is calculated rather than persisted. | `SaleItem.Subtotal` |
| Sale total | The sum of the sale-line subtotals. It is calculated rather than persisted. | `Sale.Total` |
| User | An internal operator associated with sales. | `User` / `user` |
| Role | One of the two role values defined by the model: `admin` or `seller`. | `user.role` |
| Password hash | The stored hash representation of a user's password. The domain must not handle the plaintext password. | `user.password_hash` |
| Image key | An opaque reference to an image stored externally; it is not the image binary or a filesystem path. | `product.image_key` |
| Logical deletion | Retiring a product through `deleted_at` without physically deleting its database row. | `product.deleted_at` |
| Sales report | An aggregated read result computed from stored data rather than persisted as a separate entity. | Reporting model |

**Source reference:** `spec/data-model.md`, §1–§3 and the relevant reporting definitions.

## 3. Entities and responsibilities

### 3.1 Category

`Category` classifies products.

The model defines five seeded categories: General, Herramientas, Electricidad, Fontanería, and Pinturas.

Category maintenance is not exposed through the documented domain ports. The repository is described as read-only for categories.

The category name must be non-empty and trimmed by the domain. The source distinguishes this domain rule from the database's unique-name index.

**Source reference:** `spec/data-model.md`, §2.1 and §9.

### 3.2 Product

`Product` is the aggregate root for catalog information.

Its documented data includes:

- Name.
- Price.
- Stock.
- Category.
- Optional image key.
- Logical-deletion state represented by `deleted_at`.

Important invariants:

- The name must not be empty and is trimmed by the domain.
- The price must be greater than zero according to the domain rule.
- Stock must never become negative.
- Withdrawing more stock than is available must fail.
- A category must be associated with the product.
- An absent image key is represented as `NULL`, not an empty string.
- Product retirement uses logical deletion rather than physical deletion.

The database constraint `ck_product_stock_non_negative` enforces non-negative stock. The source identifies the physical price constraint as pending or requiring verification; therefore, the positive-price domain rule must not be presented as an already-enforced database constraint.

**Source reference:** `spec/data-model.md`, §2.2 and the relevant entries in §3 and the task list.

### 3.3 Sale

`Sale` is the aggregate root for a sale transaction.

Its responsibilities include managing sale lines, applying the documented stock-withdrawal behavior, calculating the total, and checking whether the sale can be confirmed.

Important invariants:

- A sale must have at least one line to be confirmable.
- A product cannot appear more than once in the same sale, according to the domain rule.
- Adding a sale line withdraws stock before adding the line.
- A registered sale is treated as immutable.
- The sale total is calculated from its lines rather than stored as a database column.

The unique database index on `(sale_id, product_id)` is identified in the source as part of T-20. The final documentation must preserve the source's status for this index and any related task.

**Source reference:** `spec/data-model.md`, §2.3 and the relevant physical-index and task entries.

### 3.4 SaleItem

`SaleItem` is an entity internal to the `Sale` aggregate. It must not exist independently of its sale in the documented domain model.

It represents the product and quantity included in a sale and preserves the relevant historical values from the transaction.

Important invariants:

- The line belongs to a sale.
- The quantity must be greater than zero according to the domain rule.
- The unit price and product name represent the values captured at the time of the sale.
- The historical category name is retained as described by the source.
- The subtotal is calculated from the historical unit price and quantity.

The source distinguishes domain validation from database enforcement. In particular, the physical enforcement status of the positive-quantity rule must not be overstated.

**Source reference:** `spec/data-model.md`, §2.4 and the physical model and foreign-key sections.

### 3.5 User

`User` represents an internal operator.

The documented roles are `admin` and `seller`.

Important invariants:

- The username must be unique.
- The domain normalizes usernames to lowercase and trims them.
- The password hash must be present and non-empty.
- The domain uses `password_hash`, not the plaintext password.
- Role validation is a domain responsibility unless the source confirms a corresponding database constraint.

The foreign key from `sale.sold_by_user_id` to `user` is identified as pending under T-12. The existence of the field does not establish that the database relationship is already enforced.

**Source reference:** `spec/data-model.md`, §2.5, §5, and T-12.

## 4. Value objects and calculations

### Money

The `Money` value object applies rounding to two decimal places using `MidpointRounding.AwayFromZero`, according to the source.

The physical monetary column uses `numeric(18,2)`. The system is monocurrency by construction; the database model does not contain currency columns.

The source notes that `Money` can represent zero, so the positive-price rule must be enforced by the relevant product behavior rather than assumed from the value object's constructor.

### Quantity

The `Quantity` value object rejects values that do not satisfy the domain's positive-quantity rule.

The existence of this validation does not prove that PostgreSQL has a corresponding `CHECK` constraint.

### Subtotal and total

The line subtotal is calculated as historical unit price multiplied by quantity.

The sale total is calculated by summing line subtotals.

Neither calculation requires a separate persisted subtotal or total column in the supplied model.

**Source reference:** `spec/data-model.md`, §1–§3 and §2.2–§2.4.

## 5. Domain rules versus database constraints

The following distinctions must remain explicit:

| Rule | Domain behavior | Database status described by the source |
|---|---|---|
| Stock is non-negative | Product stock operations enforce the rule. | Confirmed: `ck_product_stock_non_negative`. |
| Product price is positive | Product price-change behavior validates the rule. | Pending or requires verification; do not claim a confirmed `CHECK`. |
| Sale-line quantity is positive | `Quantity` validates the rule. | Physical constraint status requires verification. |
| A sale has at least one line before confirmation | `Sale.EnsureConfirmable`. | Domain-only; not guaranteed by a normal row-level `CHECK`. |
| A product is not repeated within a sale | `Sale.AddItem` rejects duplicates. | A unique index is identified in connection with T-20; retain the source's current status. |
| A sale line belongs to a sale | Domain ownership and sale-line relationship. | `sale_id` relationship uses `ON DELETE CASCADE` as described in the source. |
| A sale is associated with a user | `sold_by_user_id` represents the association. | Foreign key is pending under T-12. |

**Source reference:** `spec/data-model.md`, §2–§5 and the task list.

## 6. Domain behavior: adding a sale line

The documented behavior of `Sale.AddItem` is:

1. Receive a product and quantity for the sale.
2. Validate the relevant domain rules.
3. Withdraw the requested quantity from the product's stock.
4. Create the sale line with the relevant historical product values.
5. Add the line to the sale.
6. Calculate the subtotal from the historical unit price and quantity.

The operation's documented order matters: stock is withdrawn before the line is added.

The source does not establish every detail of database transactions, rollback behavior, or failure recovery. Those mechanisms must not be invented as confirmed architecture.

**Source reference:** `spec/data-model.md`, §2.2–§2.4.

## 7. Domain events

The supplied model identifies domain entities and behavior but does not, by itself, confirm an implemented domain-event mechanism or message broker.

Events such as `SaleConfirmed` or `StockWithdrawn` could be considered **proposed domain events** for future design, but they must not be documented as existing events without evidence from the source.

No event-driven infrastructure is assumed in this evaluation.

## 8. Open questions

The following items must remain visible until the source confirms their resolution:

- Physical enforcement of positive product price.
- Physical enforcement of positive sale-line quantity.
- Completion status of the product foreign key for `sale_item` under T-20.
- Completion status of the user foreign key for `sale` under T-12.
- Reporting behavior when historical category names differ after a category rename.

## 9. Traceability

This document is derived from the entity definitions, invariants, physical-model descriptions, foreign-key policy, and pending-task information in `spec/data-model.md`.

The supplied model takes precedence over unsupported assumptions or descriptions copied from unrelated projects.
