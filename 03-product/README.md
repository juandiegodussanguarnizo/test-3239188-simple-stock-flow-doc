# 03 — Product Definition

## 1. Product name

**Simple Stock Flow**

## 2. Product vision

Simple Stock Flow represents a product catalog and sales workflow in which product information, available stock, sale transactions, and historical sale-line values are maintained consistently.

The product model emphasizes reliable stock rules, traceable sales, and the preservation of transaction values over time.

This vision is derived from the entities, invariants, and reporting capabilities documented in [`spec/data-model.md`](../spec/data-model.md). It does not imply that a particular user interface or deployment architecture has been confirmed.

## 3. Problem statement

A system that records products and sales must distinguish between the current catalog and the historical facts of a transaction.

For example, changing a product's price after a sale must not automatically change the price recorded for that earlier sale. Likewise, stock must not become negative through a valid domain operation or in the persisted database.

The supplied model addresses these concerns by representing products, sales, and sale lines separately; preserving historical values on sale lines; calculating subtotals and totals; and enforcing a database constraint against negative stock.

**Source references:** `spec/data-model.md`, §2.2–§2.4 and the physical model in §3.

## 4. Product objectives

The documented product model supports the following objectives:

1. Represent products with a name, positive price, stock quantity, category, and optional image key.
2. Associate products with the predefined categories.
3. Prevent negative stock from being persisted.
4. Represent sales and their individual product lines.
5. Preserve the relevant product values at the time of a sale.
6. Calculate sale-line subtotals and sale totals from their underlying values.
7. Represent internal users and the roles `admin` and `seller`.
8. Support the product-oriented aggregated reporting described by the source.
9. Retire products logically instead of physically deleting their records.

These are objectives derived from the data model, not a claim that every interaction or user-facing workflow has been specified.

## 5. Intended users

The model defines two internal role values:

- `admin`
- `seller`

The detailed permissions and user journeys for each role are not fully established by the supplied data model. They must be specified only when additional evidence is available.

The model does not define customers as a separate entity. Customer account management must therefore not be included as a confirmed product capability.

**Source reference:** `spec/data-model.md`, §1 and §2.5.

## 6. Product scope

### 6.1 Included

The documented scope includes:

- Product catalog representation.
- Product classification through predefined categories.
- Stock-related domain rules.
- Sale and sale-line representation.
- Historical sale-line values.
- Calculated subtotals and totals.
- Internal user roles and password hashes.
- Product-oriented aggregated reports.
- Logical product retirement.
- PostgreSQL persistence.

### 6.2 Excluded or unconfirmed

The following are not confirmed by the source:

- Customer management.
- Product descriptions, SKUs, and reference codes.
- Category CRUD operations.
- Reports grouped by seller.
- Multiple currencies.
- A specific frontend or graphical interface.
- REST endpoints or an API contract.
- Microservices, message brokers, or a distributed architecture.
- Specific availability, performance, or response-time targets.

These items must not be added to the confirmed scope without supporting requirements.

## 7. Product constraints

### Data integrity

The database enforces non-negative product stock through `ck_product_stock_non_negative`.

Positive product price and positive sale-line quantity are documented domain rules, but the physical enforcement status must be checked against the source's pending tasks and constraints.

### Historical accuracy

Sale lines retain relevant values from the transaction. These historical values are distinct from the current product catalog.

### Calculated amounts

Line subtotals and sale totals are calculated and are not stored as separate persisted columns.

### User credentials

The domain works with password hashes rather than plaintext passwords.

### Product retirement

Products use logical deletion through `deleted_at`. This must not be described as ordinary physical deletion.

### Reporting

The source describes aggregated product reporting. It does not establish seller-based report grouping.

**Source references:** `spec/data-model.md`, §2–§5 and the reporting sections.

## 8. Product success criteria

The following criteria can be checked against the supplied specification:

- Product information matches the defined fields.
- Product stock cannot be persisted below zero.
- Domain-only rules are not misrepresented as database constraints.
- Sale lines preserve historical values.
- Subtotals and totals are calculated rather than stored.
- The documented role values match the source.
- Product retirement is represented by `deleted_at`.
- Reports are described without unsupported grouping dimensions.
- Pending relationships and constraints remain explicitly identified.

These are documentation and model-consistency criteria. The source does not provide business KPIs or numerical targets for product success.

## 9. Assumptions and unresolved decisions

The supplied data model does not fully define the user interface, detailed role permissions, deployment topology, or all reporting decisions.

No numerical performance, availability, or adoption target is introduced because none is supported by the source.

The final implementation status of pending constraints and foreign keys must follow the source rather than an assumed future state.

## 10. Traceability

This product definition is derived from `spec/data-model.md`, including its glossary, entity invariants, physical schema, foreign-key policy, reporting behavior, and pending-task information.

If a product statement cannot be supported by the supplied model, it must be removed or explicitly marked as an assumption.
