# Inventory Replenishment Analytics

End-to-end analytics project for a fictional auto parts and hardware distributor: from a synthetic ERP to a Power BI dashboard and an automated purchase suggestion.

> **Status:** in progress. Data model defined; synthetic data generator under construction.

## Business problem

Distributors tie up most of their working capital in inventory and receivables. Without good visibility, they overstock slow movers, run out of fast movers, and don't notice which suppliers or customers are hurting cash flow.

This project answers four questions a purchasing or operations manager asks every week:

1. **What should I buy this week?** Suggested purchase order based on reorder point and safety stock.
2. **How much money is sitting in inventory?** Inventory value, days of inventory, and turnover by SKU, category, and warehouse.
3. **Which suppliers are failing me?** On-time delivery, fill rate, and actual vs. promised lead time.
4. **How much am I owed?** Accounts receivable aging and cash conversion cycle.

## The company (synthetic)

| | |
|---|---|
| Industry | Auto parts and hardware distribution |
| SKUs | ~500 across 10 categories |
| Warehouses | 3 |
| Suppliers | ~25 |
| Customers | ~150, on credit terms |
| History | 18 months, daily |

The data includes realistic patterns: ABC sales distribution (few SKUs drive most revenue), seasonality (batteries peak in winter, cooling in summer), stockouts, late and partial supplier deliveries, and customers who pay late.

## Architecture

_Diagram coming soon._

```
Synthetic ERP (FastAPI) → Python pipeline (daily incremental) → PostgreSQL (star schema) → SQL business logic → Power BI
                                                                                              ↓
                                                                     Automation: suggested PO (Excel) + email alerts
```

## Data model

Source tables: `products`, `suppliers`, `warehouses`, `customers`, `sales`, `invoices`, `inventory_snapshots`, `purchase_orders`.

Full definitions, keys, and consistency rules: [docs/data_model.md](docs/data_model.md)

## Tech stack

- **Python:** pandas, NumPy, Faker (synthetic data), FastAPI (source API)
- **PostgreSQL** in Docker
- **SQL:** dimensional model and business logic
- **Power BI:** dashboard (3 pages: Inventory, Purchasing & Suppliers, Finance)

## Repository structure

```
api/          Source ERP API
generator/    Synthetic data generator
pipeline/     Extraction and load into PostgreSQL
sql/          Star schema and business logic
powerbi/      .pbix file and screenshots
automation/   Suggested purchase order and email alerts
docs/         Data model and documentation
```

## How to run

_Coming soon._

## Dashboards

_Screenshots coming soon._

## Roadmap

- [x] Project structure and data model
- [ ] Synthetic data generator
- [ ] Source API
- [ ] Pipeline and star schema
- [ ] SQL business logic (reorder point, safety stock, service level, SKU risk scoring)
- [ ] Power BI dashboard
- [ ] Automated purchase suggestion and alerts
- [ ] Phase 2: AI agent that answers "what should I buy this week?"

## Author

Oscar Rojas, Business Intelligence Developer. [GitHub](https://github.com/oscar-rjs04)