# 05 — Architecture

## 1. Purpose

This document reconstructs the architecture supported by [`spec/data-model.md`](../spec/data-model.md).

The objective is to identify the domain responsibilities, persistence structure, relationships, rules, and reporting responsibilities that can be established from the supplied specification.

This document distinguishes confirmed facts from conceptual organization and unresolved implementation details.

## 2. Architectural overview

The confirmed persistence technology is PostgreSQL. The database is named `simple_stock_flow`, and the relevant schema is `sales`.

The physical model identifies five tables:

- `category`
- `product`
- `sale`
- `sale_item`
- `user`

The domain model identifies corresponding concepts, including `Category`, `Product`, `Sale`, `SaleItem`, and `User`.

The source also identifies domain behavior such as `Product.Withdraw`, `Sale.AddItem`, and `Sale.EnsureConfirmable`, as well as calculated reporting results.

These elements establish domain and persistence responsibilities. They do not, by themselves, prove the existence of a specific frontend, HTTP API, deployment topology, or microservice architecture.

**Source references:** `spec/data-model.md`, §0–§3 and the entity definitions in §2.

## 3. Architectural responsibilities

### 3.1 Domain responsibilities

The domain is responsible for the business rules explicitly described in the source, including:

- Validating product names and positive prices.
- Preventing domain operations from withdrawing more stock than is available.
- Maintaining the rules for sale-line quantities.
- Preventing duplicate products within a sale.
- Requiring at least one line before a sale can be confirmed.
- Preserving historical sale-line values.
- Calculating subtotals and sale totals.
- Treating registered sales as immutable in the documented domain model.

The source identifies the corresponding entities, value objects, and methods. The documentation must not replace those names with invented interfaces or services.

### 3.2 Persistence responsibilities

PostgreSQL stores the five modeled entities and their relationships.

The database also enforces specific constraints, including non-negative product stock and the unique indexes identified by the physical model.

The persistence layer must be documented separately from domain validation because a rule enforced only in the domain can be bypassed by writes that do not pass through that domain behavior.

### 3.3 Reporting responsibilities

The source describes aggregated product reporting. Report results are calculated from stored information rather than persisted as a separate reporting entity.

The documentation must not introduce seller-based report grouping or other dimensions not supported by the source.

## 4. Physical data model

### 4.1 Category

The `category` table represents the predefined product categories.

The source describes five seeded categories and a read-only category repository. Category CRUD operations must not be assumed.

### 4.2 Product

The `product` table represents catalog items.

Its documented fields include the product identifier, name, price, stock, category identifier, optional `image_key`, historical/category-related fields where specified by the physical model, and `deleted_at` for logical retirement.

The database constraint `ck_product_stock_non_negative` prevents persisted stock from becoming negative.

The source identifies the physical enforcement of positive price as pending or requiring verification. The domain rule and the physical constraint must therefore be documented separately.

### 4.3 Sale

The `sale` table represents sale records.

The sale is associated with an internal user through `sold_by_user_id`. The source identifies the foreign key to `user` as pending under T-12.

The presence of this column does not prove that PostgreSQL currently enforces the corresponding foreign key.

### 4.4 SaleItem

The `sale_item` table represents the lines belonging to a sale.

The source describes a foreign key to `sale` with `ON DELETE CASCADE`, and identifies the relationship to `product` in connection with T-20.

The final documentation must preserve the physical status recorded in the source, rather than assuming that all relationships are complete merely because they appear in the conceptual model.

Sale-line fields preserve historical values as described by the source. Subtotals are calculated rather than stored as separate columns.

### 4.5 User

The `user` table represents internal users.

The documented role values are `admin` and `seller`. The table includes `password_hash`, and the domain must not handle plaintext passwords.

The physical model and task list remain authoritative for the enforcement status of username normalization and role validation.

**Source references:** `spec/data-model.md`, §2–§5 and the relevant task entries.

## 5. Relationships and referential integrity

| Relationship | Documented behavior | Status to preserve |
|---|---|---|
| `product.category_id` → `category` | Product references a category; deletion policy is restrictive. | Follow the foreign-key definition in the physical model. |
| `sale_item.sale_id` → `sale` | A sale line belongs to a sale; the source describes `ON DELETE CASCADE`. | Confirmed according to the source's physical model. |
| `sale_item.product_id` → `product` | A sale line references a product. | Preserve the status associated with T-20. |
| `sale.sold_by_user_id` → `user` | A sale identifies the associated user. | Foreign key pending under T-12. |

The conceptual existence of a relationship must not be confused with the current existence of a physical foreign-key constraint.

**Source reference:** `spec/data-model.md`, §5 and the relevant entity and task definitions.

## 6. Domain rules and physical enforcement

### 6.1 Stock

The database enforces `stock >= 0` through `ck_product_stock_non_negative`.

The domain also prevents withdrawing more stock than is available. This business behavior is distinct from the database's non-negative constraint.

### 6.2 Price

The domain requires `price > 0`.

The source identifies the physical price constraint as pending or requiring verification. Do not claim that PostgreSQL currently enforces the positive-price rule unless the source confirms it.

### 6.3 Quantity

The domain requires a positive sale-line quantity through the `Quantity` value object.

The physical enforcement status must follow the source's constraint and task information.

### 6.4 Sale confirmability

`Sale.EnsureConfirmable` requires at least one line before a sale can be confirmed.

This is a domain rule. A normal row-level `CHECK` does not establish that a sale has at least one row in another table.

### 6.5 Sale immutability

The source describes a sale as immutable and identifies no domain ports for editing or deleting sales.

This does not prove that every direct database update or deletion is physically rejected.

### 6.6 Logical deletion

Product retirement uses `deleted_at` rather than normal physical deletion.

The source describes the corresponding logical-deletion mechanism. The architecture must not present logical deletion as a general hard-delete operation.

## 7. Sale-line creation flow

The documented `Sale.AddItem` behavior establishes the following conceptual sequence:

1. Receive the product and requested quantity.
2. Apply the relevant domain validations.
3. Withdraw stock through `Product.Withdraw`.
4. Create the sale line using the relevant historical product values.
5. Add the line to the sale.
6. Calculate the line subtotal from the historical unit price and quantity.
7. Calculate the sale total from its line subtotals when required.

The documented order is important: stock withdrawal occurs before the line is added.

This sequence describes domain behavior, not a complete infrastructure transaction design. The source must be consulted before claiming specific rollback guarantees, transaction boundaries, retry policies, or concurrency mechanisms.

**Source reference:** `spec/data-model.md`, §2.2–§2.4 and the relevant architectural decisions referenced by the source.

## 8. Ports and adapters

The supplied model supports the identification of domain behavior, persistence responsibilities, and aggregated read operations.

However, this document must not invent concrete controller names, repository interfaces, application services, or adapters that cannot be verified in the source.

Where the source identifies a port or interface, its actual name and responsibility should be used. Where the source does not establish a concrete implementation, document the responsibility without presenting a proposed component as an existing one.

The following distinction applies:

| Element | Documentation treatment |
|---|---|
| Domain entities and named methods | Document as identified by the source. |
| PostgreSQL tables and constraints | Document according to the physical model. |
| Aggregated reporting | Document the source-supported query responsibility. |
| Unconfirmed controllers and endpoints | Do not present as implemented. |
| Unconfirmed microservices or message brokers | Do not present as implemented. |
| Proposed future components | Label explicitly as proposals, not existing architecture. |

## 9. Reporting and historical data

The model supports aggregated product reporting computed from stored data.

The architecture must preserve the distinction between current product information and historical sale-line information.

The source identifies an unresolved decision involving reporting and historical category names. A category rename may affect how historical category values are grouped, depending on the reporting rule ultimately selected.

This behavior must remain an open decision until the source or an authorized decision resolves it.

The source does not establish seller-based reporting, and this architecture must not add it.

## 10. Dates, monetary values, and initialization

### 10.1 Dates and timestamps

The physical model uses `timestamptz`, and the source identifies UTC as the server time reference.

Do not invent additional timezone-conversion behavior beyond what the source establishes.

### 10.2 Monetary values

The monetary model uses a single currency by construction. No currency columns are defined.

`Money` rounds to two decimal places using `MidpointRounding.AwayFromZero`, and the physical monetary column uses `numeric(18,2)`.

### 10.3 Administrator initialization

The source describes initialization of an initial administrator at application startup using credentials supplied through the environment.

Do not document this as an SQL seed-user insertion if the source specifies application initialization.

## 11. Security and data integrity

The architecture must preserve the following source-supported controls:

- Persisted product stock cannot be negative.
- Passwords are represented by hashes in the domain and persisted model.
- Password hashes must not be exposed in application logs.
- Historical sale-line values must be retained as specified by the model.
- Products use logical retirement through `deleted_at`.
- Category and product relationships follow the documented foreign-key policy.
- Pending constraints and relationships must not be represented as confirmed.
- No customer entity or unsupported customer data is added to the model.

**Source references:** `spec/data-model.md`, the entity definitions, physical model, foreign-key policy, and credential-handling rules.

## 12. Unresolved architectural questions

| Item | Required treatment |
|---|---|
| Positive product-price constraint | Keep the physical enforcement status pending or requiring verification. |
| Positive sale-line quantity constraint | Verify against the source before claiming a database constraint. |
| `sale_item` to `product` foreign key | Preserve the status associated with T-20. |
| `sale` to `user` foreign key | Preserve pending status under T-12. |
| Reporting after category renaming | Do not resolve without an authoritative decision. |
| Transaction and concurrency guarantees | Do not claim behavior that the source does not establish. |
| Concrete controllers, APIs, and deployment components | Do not invent implementation details. |

## 13. Architecture consistency review

Before submitting, verify that:

- The architecture contains the same five tables as the source.
- Domain rules are not misrepresented as physical database constraints.
- The documented foreign-key status matches the source.
- The sale-line creation sequence reflects `Sale.AddItem`.
- Subtotals and totals are calculated, not stored.
- Historical values are distinguished from current catalog values.
- Reporting does not introduce unsupported seller-based grouping.
- Product retirement is documented as logical deletion.
- Unresolved decisions remain unresolved.
- The architecture agrees with the context, domain, product, and requirements documents.

## 14. Source of truth

All architectural claims in this document derive from [`spec/data-model.md`](../spec/data-model.md), including its glossary, entity definitions, physical model, foreign-key policy, reporting definitions, and pending-task information.

If an architectural statement cannot be supported by the supplied source, remove it or explicitly label it as an assumption or proposal.
