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

Esta separación es una representación conceptual para documentar el modelo. La fuente debe consultarse para confirmar las clases, interfaces y componentes concretos de la implementación.

