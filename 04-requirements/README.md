# 04 — Requirements Specification

## 1. Purpose

This document derives functional and non-functional requirements for **Simple Stock Flow** from [`spec/data-model.md`](../spec/data-model.md).

The requirements describe behavior supported by the source. They do not establish unconfirmed screens, endpoints, deployment components, or business rules.

## 2. Requirement conventions

Each requirement includes an identifier, a statement, and a traceability reference.

The following status distinctions apply:

- **Confirmed:** directly supported by the source.
- **Domain-only:** enforced by domain behavior but not necessarily by PostgreSQL.
- **Pending verification:** the physical implementation is pending or the source contains an unresolved inconsistency.
- **Assumption:** not established by the source and therefore not treated as a confirmed requirement.

The identifiers below are assigned for this documentation and do not imply that the source already uses these IDs.

## 3. Functional requirements

### FR-01 — Product classification

The system must represent the association between a product and its category.

**Traceability:** `product.category_id`, `category`, and the `Product.SetCategory` behavior.

**Status:** The relationship is defined in the model. The physical foreign-key policy must follow the source.

### FR-02 — Product catalog

The system must represent a product with its name, price, stock, category, and optional image key.

**Traceability:** `product` and `Product`.

**Constraint:** Do not introduce SKU, description, or reference-code fields as existing model attributes.

### FR-03 — Non-negative stock

The system must prevent negative stock from being persisted.

**Traceability:** `ck_product_stock_non_negative`.

**Status:** Confirmed database constraint.

### FR-04 — Positive product price

The domain must reject product prices that do not satisfy the positive-price rule.

**Traceability:** `Product.ChangePrice` and the `Money` behavior described in the source.

**Status:** Domain rule. The source identifies the physical constraint as pending or requiring verification; do not claim that a database `CHECK` is already enforced.

### FR-05 — Sale representation

The system must represent a sale and its associated information, including the user identifier and sale timestamp as defined in the model.

**Traceability:** `sale` and `sale.sold_by_user_id`.

**Status:** The field is described by the model. The foreign key to `user` is pending under T-12.

### FR-06 — Sale-line representation

The system must represent each sale line through `sale_item`, including its relationship to a sale and the product-related values required by the source.

**Traceability:** `sale_item` and the physical model.

**Status:** Follow the current source status for the related foreign keys, including T-20.

### FR-07 — Positive sale-line quantity

The domain must reject a sale-line quantity that is not greater than zero.

**Traceability:** `Quantity` and the `SaleItem` rules.

**Status:** Domain rule. Physical enforcement must be verified against the source.

### FR-08 — Sale confirmability

A sale must contain at least one line before it can be confirmed.

**Traceability:** `Sale.EnsureConfirmable`.

**Status:** Domain-only rule; do not claim it is enforced by a normal database `CHECK`.

### FR-09 — Duplicate product prevention

The domain must prevent the same product from being added more than once to a sale.

**Traceability:** `Sale.AddItem` and the unique index on `(sale_id, product_id)` described in the source.

**Status:** The domain rule is defined. The physical index status must be reported according to the source and T-20.

### FR-10 — Stock withdrawal when adding a sale line

When a sale line is added, the documented domain behavior must withdraw the requested quantity from product stock before adding the line.

**Traceability:** `Sale.AddItem` and `Product.Withdraw`.

**Status:** Domain behavior. Do not infer transaction rollback or concurrency guarantees beyond those established by the source.

### FR-11 — Historical product values

A sale line must preserve the relevant product values captured at the time of the sale, including the historical unit price and product name.

**Traceability:** `Sale.AddItem`, `SaleItem`, and the historical fields in `sale_item`.

### FR-12 — Historical category name

A sale line must preserve the historical category name as described by the model.

**Traceability:** `sale_item.category_name` and the relevant historical-reporting decisions.

**Constraint:** Do not add a foreign key from the historical category name to the current category record unless the source explicitly requires it.

### FR-13 — Subtotal calculation

The subtotal of a sale line must be calculated from its historical unit price and quantity.

**Traceability:** `SaleItem.Subtotal`.

**Constraint:** The subtotal must not be documented as a persisted column in the supplied model.

### FR-14 — Sale total calculation

The sale total must be calculated from its sale-line subtotals.

**Traceability:** `Sale.Total`.

**Constraint:** The total must not be documented as a persisted column in the supplied model.

### FR-15 — Sale immutability

A registered sale must be treated as immutable by the documented domain model.

**Traceability:** The `Sale` definition and the absence of domain ports for editing or deleting a sale.

**Constraint:** Domain immutability does not, by itself, prove that PostgreSQL rejects every direct update or deletion.

### FR-16 — Logical product retirement

The model must represent product retirement through `deleted_at` rather than normal physical deletion.

**Traceability:** `product.deleted_at` and the logical-deletion behavior described in the source.

### FR-17 — Internal roles and credentials

The system must represent internal users with the role values `admin` and `seller`, and the domain must work with password hashes rather than plaintext passwords.

**Traceability:** `user.role`, `user.password_hash`, and the `User` rules.

**Constraint:** Detailed permissions for each role must not be invented.

### FR-18 — Aggregated product reporting

The system must support the aggregated product reporting described by the source, with calculated results rather than a separate persisted reporting entity.

**Traceability:** The reporting section of `spec/data-model.md`.

**Constraint:** Do not claim that reports are grouped by seller. Preserve any unresolved decision about historical category names.

### FR-19 — Initial administrator

The documented initialization process must reflect the source's description of an initial administrator created at application startup using credentials supplied through the environment.

**Traceability:** The initialization and configuration section of `spec/data-model.md`.

**Constraint:** Do not describe the administrator as a SQL seed user if the source specifies application initialization.

## 4. Non-functional requirements

### NFR-01 — Relational persistence

The documented persistence model must use PostgreSQL with the `sales` schema.

**Traceability:** The physical database model in `spec/data-model.md`.

### NFR-02 — Stock integrity

The database must enforce non-negative persisted product stock.

**Traceability:** `ck_product_stock_non_negative`.

### NFR-03 — Historical consistency

Historical sale-line values must remain distinct from current catalog values.

**Traceability:** `sale_item` historical fields and the `SaleItem` behavior.

### NFR-04 — Credential handling

The domain must not handle plaintext passwords, and password hashes must not be exposed in application logs.

**Traceability:** The `User` definition and credential-handling rules in the source.

### NFR-05 — Timestamp handling

Timestamp fields must follow the source's `timestamptz` representation and UTC server-time convention.

**Traceability:** The physical model and the source's date/time conventions.

### NFR-06 — Monetary consistency

The documented model must remain monocurrency because the schema does not contain currency fields.

**Traceability:** The monetary definitions and physical schema.

### NFR-07 — Logical deletion

Product retirement must preserve the database record through the documented `deleted_at` mechanism.

**Traceability:** `product.deleted_at` and the logical-deletion rules.

### NFR-08 — Documentation traceability

Every derived requirement must be traceable to a source section, entity, field, constraint, domain method, or task identifier. Unsupported statements must be labeled as assumptions or excluded.

**Traceability:** The evaluation instructions in the repository's root `README.md`.

### NFR-09 — No unsupported performance targets

No response-time, throughput, availability, or capacity target is established by this requirements document because the supplied source does not provide numerical values for these metrics.

**Traceability:** Absence of defined numerical service targets in the supplied model.

## 5. Pending items and verification

| Item | Source status | Required documentation treatment |
|---|---|---|
| Positive product price in PostgreSQL | Pending or inconsistent in the source | Keep the domain rule separate from physical enforcement. |
| Positive quantity in PostgreSQL | Physical enforcement requires verification | Do not claim a confirmed `CHECK` without evidence. |
| `sale.sold_by_user_id` foreign key | Pending under T-12 | Describe the field separately from the pending FK. |
| `sale_item` to `product` foreign key | Associated with T-20 | Use the physical-model and task status given by the source. |
| Unique `(sale_id, product_id)` index | Described in connection with T-20 | Preserve the source's documented status. |
| Reporting after category renaming | Unresolved decision | Do not silently select a reporting behavior. |

## 6. Traceability matrix

| Requirement | Main source element |
|---|---|
| FR-01 | `product.category_id`, `category` |
| FR-02 | `Product`, `product` |
| FR-03 | `ck_product_stock_non_negative` |
| FR-04 | `Product.ChangePrice`, `Money`, relevant pending task |
| FR-05 | `sale`, `sold_by_user_id`, T-12 |
| FR-06 | `sale_item` and its relationships |
| FR-07 | `Quantity`, `SaleItem` |
| FR-08 | `Sale.EnsureConfirmable` |
| FR-09 | `Sale.AddItem`, unique index, T-20 |
| FR-10 | `Sale.AddItem`, `Product.Withdraw` |
| FR-11 | Historical product values in `sale_item` |
| FR-12 | `sale_item.category_name` |
| FR-13 | `SaleItem.Subtotal` |
| FR-14 | `Sale.Total` |
| FR-15 | `Sale` immutability and absence of edit/delete ports |
| FR-16 | `product.deleted_at` |
| FR-17 | `user.role`, `user.password_hash` |
| FR-18 | Reporting definitions |
| FR-19 | Initial administrator initialization |
| NFR-01 | PostgreSQL physical model |
| NFR-02 | `ck_product_stock_non_negative` |
| NFR-03 | Historical sale-line values |
| NFR-04 | Credential-handling rules |
| NFR-05 | `timestamptz`, UTC |
| NFR-06 | Monocurrency model |
| NFR-07 | `product.deleted_at` |
| NFR-08 | Root evaluation instructions |
| NFR-09 | No numerical service targets in the source |

## 7. Acceptance checklist

- [ ] Every requirement is traceable to the source.
- [ ] Domain rules are distinguished from database constraints.
- [ ] Pending tasks are not described as completed without evidence.
- [ ] Calculated subtotals and totals are not represented as stored columns.
- [ ] Historical sale-line values are documented correctly.
- [ ] No unsupported customer, seller-reporting, API, or microservice features are introduced.
- [ ] No unsupported performance or availability targets are invented.
- [ ] The requirements agree with the domain, product, context, and architecture documents.
