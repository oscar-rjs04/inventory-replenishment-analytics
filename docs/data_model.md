# Data Model

Synthetic ERP for a fictional distributor (~500 SKUs, 3 warehouses, ~25 suppliers, ~150 customers, 18 months of history). Source tables mimic what a real ERP exposes; analytical logic (ABC class, reorder point, safety stock) is computed downstream in SQL.

## Source tables (ERP)

### products
One row per SKU.

| Column | Type | Key | Description |
|---|---|---|---|
| sku | VARCHAR(10) | PK | Product code, e.g. SKU-00001 |
| name | VARCHAR(100) | | Product name |
| category | VARCHAR(50) | | Product category |
| supplier_id | INT | FK → suppliers | Primary supplier |
| unit_cost | NUMERIC(10,2) | | Current purchase cost |
| unit_price | NUMERIC(10,2) | | Current list price |

Categories: Brakes, Suspension, Filters, Oils & Lubricants, Batteries & Electrical, Cooling, Lighting, Hand Tools, Power Tools, Fasteners.

### suppliers
One row per supplier.

| Column | Type | Key | Description |
|---|---|---|---|
| supplier_id | INT | PK | |
| name | VARCHAR(100) | | Supplier name |
| promised_lead_time_days | INT | | Lead time the supplier commits to |

### warehouses
One row per warehouse.

| Column | Type | Key | Description |
|---|---|---|---|
| warehouse_id | INT | PK | |
| name | VARCHAR(50) | | Warehouse name |
| city | VARCHAR(50) | | City |

### customers
One row per customer.

| Column | Type | Key | Description |
|---|---|---|---|
| customer_id | INT | PK | |
| name | VARCHAR(100) | | Customer name |
| credit_limit | NUMERIC(12,2) | | Maximum open balance allowed |
| credit_days | INT | | Payment terms (e.g. 15, 30, 60) |

### sales
One row per sale line (one SKU within one sale order).

| Column | Type | Key | Description |
|---|---|---|---|
| sale_line_id | INT | PK | |
| sale_id | INT | | Sale order number (groups lines) |
| sale_date | DATE | | |
| sku | VARCHAR(10) | FK → products | |
| warehouse_id | INT | FK → warehouses | Warehouse that fulfilled the line |
| customer_id | INT | FK → customers | |
| invoice_id | INT | FK → invoices | One invoice per sale order |
| qty | INT | | Units shipped |
| unit_price | NUMERIC(10,2) | | Actual price charged |

### invoices
One row per invoice (one per sale order).

| Column | Type | Key | Description |
|---|---|---|---|
| invoice_id | INT | PK | |
| customer_id | INT | FK → customers | |
| issue_date | DATE | | Same as sale_date |
| due_date | DATE | | issue_date + customer credit_days |
| amount | NUMERIC(12,2) | | Sum of qty × unit_price of its lines |
| paid_date | DATE | | NULL while unpaid |
| updated_at | TIMESTAMP | | Last change (used for incremental extraction) |

### inventory_snapshots
One row per date × SKU × warehouse (end-of-day stock).

| Column | Type | Key | Description |
|---|---|---|---|
| snapshot_date | DATE | PK | |
| sku | VARCHAR(10) | PK, FK → products | |
| warehouse_id | INT | PK, FK → warehouses | |
| on_hand_qty | INT | | Units on hand at end of day (≥ 0) |

### purchase_orders
One row per PO line (one SKU, delivered to one warehouse).

| Column | Type | Key | Description |
|---|---|---|---|
| po_line_id | INT | PK | |
| po_id | INT | | Purchase order number (groups lines) |
| supplier_id | INT | FK → suppliers | |
| sku | VARCHAR(10) | FK → products | |
| warehouse_id | INT | FK → warehouses | Destination warehouse |
| qty_ordered | INT | | |
| unit_cost | NUMERIC(10,2) | | Cost agreed on this PO |
| order_date | DATE | | |
| promised_date | DATE | | order_date + supplier promised lead time |
| received_date | DATE | | NULL while open |
| qty_received | INT | | NULL while open; may be < qty_ordered |
| updated_at | TIMESTAMP | | Last change (used for incremental extraction) |

## Relationships
- products.supplier_id → suppliers.supplier_id
- sales.sku → products.sku; sales.warehouse_id → warehouses; sales.customer_id → customers; sales.invoice_id → invoices
- invoices.customer_id → customers.customer_id
- inventory_snapshots.sku → products; inventory_snapshots.warehouse_id → warehouses
- purchase_orders.supplier_id → suppliers; purchase_orders.sku → products; purchase_orders.warehouse_id → warehouses

## Consistency rules (enforced by the generator)
1. on_hand(today) = on_hand(yesterday) + qty_received(today) − qty_sold(today), per SKU and warehouse.
2. on_hand_qty is never negative; demand above stock is lost and not recorded in sales.
3. received_date ≥ order_date; qty_received ≤ qty_ordered.
4. Late delivery = received_date > promised_date (some suppliers are systematically late).
5. invoices.amount = Σ(qty × unit_price) of its sales lines.
6. due_date = issue_date + customers.credit_days; some customers pay after due_date.