---
name: Create and track a Katana sales order
description: Create a customer and a sales order in Katana, then read it back to confirm status and fulfillment.
api: openapi/katana-openapi-original.json
generated: '2026-07-19'
method: generated
operations:
- CustomerController.createCustomer
- SalesOrderController.createSalesOrder
- SalesOrderController.getSalesOrder
- SalesOrderController.getAllSalesOrders
---

# Create and track a Katana sales order

Base URL: `https://api.katanamrp.com/v1`. Authenticate every request with
`Authorization: Bearer <api key>` (see `authentication/katana-authentication.yml`).
Responses to list calls are wrapped in a top-level `data` array. There is **no**
idempotency key — do not blindly retry a POST; on a network error, look the
resource up before re-creating it.

## Steps

1. **Create the customer** (if not already in Katana) with
   `CustomerController.createCustomer` (`POST /customers`). Capture the returned
   integer `id` as `customer_id`.
2. **Create the sales order** with `SalesOrderController.createSalesOrder`
   (`POST /sales_orders`). Reference the `customer_id`, a `location_id`, and one
   or more `sales_order_rows` (each with `variant_id`, `quantity`,
   `price_per_unit`, and optionally `tax_rate_id`).
3. **Read it back** with `SalesOrderController.getSalesOrder`
   (`GET /sales_orders/{id}`) and check `status` (e.g. `NOT_SHIPPED`),
   `invoicing_status`, and `sales_order_rows`.
4. **List / reconcile** open orders with `SalesOrderController.getAllSalesOrders`
   (`GET /sales_orders`); results come newest-first.

## Rules

- Handle `429` by honoring `Retry-After`; default limit is 60 requests / 60s.
- A `401` means the API key is missing/incorrect — regenerate under Settings > API.
- To react to status changes, subscribe to `sales_order.packed` /
  `sales_order.delivered` webhooks (see `asyncapi/katana-webhooks.yml`) rather
  than polling.
