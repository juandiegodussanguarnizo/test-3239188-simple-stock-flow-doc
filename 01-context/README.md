# 01 — System Context

## 1. Purpose

This document defines the system boundary and general context of **Simple Stock Flow**, based on [`spec/data-model.md`](../spec/data-model.md).

The objective is to explain what the system represents, which responsibilities are supported by the available specification, and which capabilities cannot be confirmed from the source.

## 2. System overview

Simple Stock Flow is a system for maintaining a product catalog, controlling stock, recording sales, and obtaining aggregated sales information.

The supplied model identifies five principal entities:

- `Category`: classifies products.
- `Product`: represents catalog items, including price and available stock.
- `Sale`: represents a recorded sale.
- `SaleItem`: represents an individual product line within a sale.
- `User`: represents an internal operator who can be associated with a sale.

The persistence layer uses PostgreSQL, with the tables located in the `sales` schema.

**Source references:** `spec/data-model.md`, §1–§3 and the entity definitions in §2.

## 3. Business problem

The data model supports the consistent representation of products, stock quantities, and sales transactions.

It also preserves historical information for sale lines. A line records the relevant product values at the time of the sale, so later catalog changes do not automatically rewrite those historical values.

The model distinguishes between stored information and calculated information. Line subtotals and sale totals are calculated from their constituent values rather than stored as separate database columns.

**Source references:** the `Product`, `Sale`, and `SaleItem` definitions in `spec/data-model.md`, §2, and the physical model in §3.

## 4. System boundary

### 4.1 Included responsibilities

The documented system includes:

- Product catalog information.
- Product-to-category relationships.
- Stock withdrawal and restocking rules described by the domain.
- Sale and sale-line representation.
- Historical product values within sale lines.
- Internal users and the roles `admin` and `seller`.
- Product-oriented aggregated reporting as described by the source.
- Logical product retirement through `deleted_at`.
- PostgreSQL persistence and the constraints explicitly documented in the model.

These statements describe the responsibilities supported by the data model. They do not imply that every user interface, endpoint, or deployment component has been specified.

### 4.2 Excluded or unconfirmed capabilities

The source does not establish the following as confirmed system capabilities:

- Customer or buyer management.
- Product descriptions, SKUs, or reference codes.
- Category creation, editing, or deletion through a management interface.
- Sales reports grouped by seller.
- Multiple currencies.
- Physical deletion of products through the normal domain model.
- HTTP APIs, a graphical interface, or a microservice architecture.
- A message broker or event-driven infrastructure.

These capabilities must not be documented as implemented features without additional evidence.

## 5. Users and roles

The model defines internal users with two role values: `admin` and `seller`.

The detailed permission matrix for each role is not fully established by the supplied data model. Therefore, this document does not assign specific permissions to either role beyond what can be directly verified in the source.

The system does not define a separate customer entity.

**Source reference:** `spec/data-model.md`, §1 and the `User` definition in §2.5.

## 6. External dependencies and technical context

The confirmed persistence technology is PostgreSQL. The source identifies PostgreSQL 16.14, the database `simple_stock_flow`, the `sales` schema, and UTC as the server time reference.

The product image is represented by an opaque `image_key`; the model does not store image binary content in that column.

The source also describes initialization of an initial administrator using credentials supplied through the environment. The exact deployment process must not be expanded beyond what the source confirms.

**Source references:** `spec/data-model.md`, §0, §2.2, and the physical model and initialization sections.

## 7. Important system constraints

- Persisted product stock must not be negative; the database constraint is `ck_product_stock_non_negative`.
- A sale must contain at least one line to be confirmable, according to `Sale.EnsureConfirmable`.
- A sale is treated as immutable by the documented domain model.
- A product is retired logically using `deleted_at`.
- Historical sale-line values must be distinguished from current catalog values.
- Domain rules and database constraints must be documented separately.
- Relationships or constraints marked as pending must remain identified as pending.

**Source references:** `spec/data-model.md`, §2.2–§2.5, §3, §5, and the relevant task entries.

## 8. Assumptions and open questions

The data model does not provide enough evidence to confirm a complete user-interface design, deployment topology, API contract, or detailed role-permission matrix.

Those matters are outside the confirmed scope of this reconstruction. They must remain unspecified or be explicitly labeled as assumptions if a later document needs to discuss them.

The physical enforcement status of positive product price, positive sale-line quantity, and the foreign key from `sale` to `user` must be represented according to the source's current status and pending-task notes.

## 9. Traceability

This context document is derived from:

- `spec/data-model.md`, §0: schema and technical conventions.
- `spec/data-model.md`, §1: domain glossary.
- `spec/data-model.md`, §2: entity definitions and invariants.
- `spec/data-model.md`, §3: physical database model.
- `spec/data-model.md`, §5: foreign-key policy.
- The relevant reporting, initialization, and pending-task sections of the same source.

The supplied data model is the authority for resolving any inconsistency between this document and the source.
