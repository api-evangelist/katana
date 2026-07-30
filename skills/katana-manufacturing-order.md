---
name: Run a Katana manufacturing order
description: Create a manufacturing order for a product variant, report production, and confirm completion.
api: openapi/katana-openapi-original.json
generated: '2026-07-19'
method: generated
operations:
- ManufacturingOrderController.createManufacturingOrder
- ManufacturingOrderController.getManufacturingOrder
- ManufacturingOrderProductionController.completePartially
- InventoryController.getInventories
---

# Run a Katana manufacturing order

Base URL: `https://api.katanamrp.com/v1`. Authenticate with
`Authorization: Bearer <api key>`.

## Steps

1. **Create the manufacturing order** with
   `ManufacturingOrderController.createManufacturingOrder`
   (`POST /manufacturing_orders`), referencing the `variant_id` to produce and
   the `location_id`. Capture the returned `id`.
2. **Report production** as work completes with
   `ManufacturingOrderProductionController.completePartially`
   (`POST /manufacturing_order_productions`), referencing the manufacturing
   order and the quantity produced (supports partial completion).
3. **Confirm status** with `ManufacturingOrderController.getManufacturingOrder`
   (`GET /manufacturing_orders/{id}`) and watch `status` move through
   `IN_PROGRESS` to `DONE`.
4. **Verify stock** with `InventoryController.getInventories`
   (`GET /inventory`) to confirm the finished product's `quantity_in_stock`
   increased and ingredient stock decremented.

## Rules

- Subscribe to `manufacturing_order.done` and
  `manufacturing_order.in_progress` webhooks for status changes instead of
  polling (see `asyncapi/katana-webhooks.yml`).
- Bin/batch/serial traceability constraints can return `422` — respect any
  inventory lock-date windows.
- No idempotency key: on retry, re-read the MO before re-posting production.
