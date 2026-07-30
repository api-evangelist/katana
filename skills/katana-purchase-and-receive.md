---
name: Purchase from a supplier and receive stock in Katana
description: Create a supplier and purchase order, then record receipt of goods against it.
api: openapi/katana-openapi-original.json
generated: '2026-07-19'
method: generated
operations:
- SupplierController.createSupplier
- PurchaseOrderController.createPurchaseOrder
- PurchaseOrderReceiveController.receivePartially
- PurchaseOrderController.getPurchaseOrder
---

# Purchase from a supplier and receive stock in Katana

Base URL: `https://api.katanamrp.com/v1`. Authenticate with
`Authorization: Bearer <api key>`.

## Steps

1. **Create the supplier** (if new) with `SupplierController.createSupplier`
   (`POST /suppliers`); capture the `id` as `supplier_id`.
2. **Create the purchase order** with
   `PurchaseOrderController.createPurchaseOrder` (`POST /purchase_orders`),
   referencing `supplier_id`, a `location_id`, and rows (each with `variant_id`,
   `quantity`, and price/tax fields). Capture the PO `id`.
3. **Receive goods** with `PurchaseOrderReceiveController.receivePartially`
   (`POST /purchase_order_receive`), referencing the purchase order rows and the
   received quantities (supports partial receipts).
4. **Confirm** with `PurchaseOrderController.getPurchaseOrder`
   (`GET /purchase_orders/{id}`) and check `status`
   (`PARTIALLY_RECEIVED` / `RECEIVED`).

## Rules

- Subscribe to `purchase_order.received` and
  `purchase_order.partially_received` webhooks for receipt events
  (see `asyncapi/katana-webhooks.yml`).
- Honor `Retry-After` on `429`; default limit 60 requests / 60s.
- No idempotency key — re-read the PO before re-posting a receipt on retry.
