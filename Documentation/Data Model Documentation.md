# Data Model Documentation

## 1. Overview

The Power BI solution uses a multi-fact analytical data model designed to support financial, sales, forecasting, pricing, customer, market, and supply-chain analysis.

The model separates transactional/business data into dedicated fact tables while using shared dimension tables for consistent filtering and analysis across the report.

The model also contains dedicated DAX measure and helper tables to support KPI calculations, financial reporting, toggles, and report metadata.

---

## 2. Model Architecture

The major components of the model are:

```text
                         ┌───────────────┐
                         │   dim_date    │
                         └───────┬───────┘
                                 │
          ┌──────────────────────┼──────────────────────┐
          │                      │                      │
          ▼                      ▼                      ▼
   ┌─────────────┐       ┌─────────────┐       ┌─────────────┐
   │ dim_product │       │dim_customer │       │  dim_market │
   └──────┬──────┘       └──────┬──────┘       └──────┬──────┘
          │                     │                     │
          └─────────────────────┼─────────────────────┘
                                │
                ┌───────────────┼────────────────┐
                │               │                │
                ▼               ▼                ▼
        ┌──────────────┐ ┌───────────────┐ ┌──────────────────┐
        │ Sales /      │ │ Forecast /    │ │ Cost / Pricing   │
        │ Actuals      │ │ Estimates     │ │ Facts            │
        └──────────────┘ └───────────────┘ └──────────────────┘
                                │
                                ▼
                         ┌──────────────┐
                         │ DAX Measures │
                         └──────────────┘
```

The actual Power BI model contains several interconnected fact tables and shared dimensions rather than a single fact table.

---

## 3. Dimension Tables

### `dim_date`

The central date dimension supports time-based analysis across the model.

Key fields visible in the model include:

- `date`
- `fiscal_year`
- `fy_month_num`
- `month`

It provides a consistent time reference for actuals, forecasts, costs, and financial analysis.

### `dim_product`

The product dimension provides product-level attributes.

Fields include:

- `category`
- `division`
- `product`
- `product_code`
- `segment`
- `variant`

This dimension allows metrics to be analyzed by product hierarchy and product attributes.

### `dim_customer`

The customer dimension provides customer and market segmentation.

Fields include:

- `channel`
- `customer`
- `customer_code`
- `market`
- `platform`
- `region`
- `sub_zone`

This enables customer, regional, channel, and market-level analysis.

### `dim_market`

The market dimension provides market hierarchy information.

Fields include:

- `market`
- `region`
- `sub_zone`

It is used for consistent market and geographical filtering across the model.

---

## 4. Fact Tables

### `fact_actual_estimates`

Contains actual/estimated business performance data.

Fields visible in the model include:

- `ads_promotions_pct`
- `category`
- `channel`
- `customer`
- `customer_code`
- `date`
- `division`
- `fiscal_year`
- `freight_cost`
- `gross_margin_amount`
- `gross_price`
- `gross_sales_amount`
- `manufacturing_cost`
- `market`
- `net_invoice_sales_amount`

This appears to be one of the central business-performance fact tables in the model.

---

### `fact_forecast_monthly`

Contains monthly forecast information.

Key attributes include:

- `category`
- `channel`
- `customer`
- `customer_code`
- `date`
- `division`
- `fiscal_year`
- `market`

This table supports comparison between forecasted and actual business performance.

---

### `Remaining Forecast`

Contains remaining forecast data used for forward-looking analysis.

It contains attributes such as:

- `category`
- `channel`
- `customer`
- `customer_code`
- `date`
- `division`
- `fiscal_year`
- `market`

This can be used to analyze expected future performance for the remaining period.

---

### `fact_freight_cost`

Contains freight-related cost information.

Fields include:

- `fiscal_year`
- `freight_pct`
- `market`
- `other_cost_pct`

This table supports freight and logistics cost analysis.

---

### `fact_pr_invoice_deductions`

Contains post/pricing-related invoice deduction information.

Visible fields include:

- `customer_code`
- `date`
- `discounts_pct`
- `other_deductions_pct`
- `product_code`

This allows invoice-level deductions to be incorporated into financial calculations.

---

### `fact_post_invoice_deductions`

Contains post-invoice deduction information.

Key fields include:

- `customer_code`
- `date`
- `discounts_pct`
- `other_deductions_pct`
- `product_code`

This supports analysis of deductions applied after invoicing.

---

### `fact_manufacturing_cost`

Contains manufacturing-cost information.

Fields include:

- `cost_year`
- `manufacturing_cost`
- `product_code`

This enables manufacturing costs to be analyzed at product level and incorporated into profitability calculations.

---

### `fact_gross_price`

Contains gross pricing information.

Fields include:

- `fiscal_year`
- `gross_price`
- `product_code`

This table provides pricing information used in sales and profitability calculations.

---

### `Operational Expense`

Contains operational expense information.

Visible fields include:

- `ads_promotions_pct`
- `fiscal_year`
- `market`
- `other_operational_expense_pct`

This supports operating-expense analysis by fiscal year and market.

---

### `fact_invoice_deduct...`

The model also contains a separate invoice deduction fact table supporting additional invoice deduction analysis.

---

## 5. Supporting / Calculation Tables

The model contains several helper tables that are not traditional transactional fact or dimension tables.

### `Report Measure`

A dedicated measure table containing DAX measures used throughout the report.

Centralizing measures provides:

- Consistent KPI definitions
- Reusable calculations
- Easier maintenance
- Cleaner report field organization

---

### `fiscal_year`

A supporting table used for fiscal-year related analysis and filtering.

---

### `NsGmTarget`

Contains target values associated with:

- GM targets
- Market
- Month
- NP targets

This table supports target-versus-actual performance analysis.

---

### `Set Toggle`

A helper table containing:

- `ID`
- `Label`

It is used to provide interactive report selection/toggle functionality.

---

### `P & L Rows`

A helper table containing:

- `Description`
- `Line Item`
- `Order`

This table is used to structure the Profit & Loss report layout and control the ordering of P&L lines.

---

### `P & L Columns`

A helper table containing:

- `Col Header`

This supports the column structure of the P&L reporting view.

---

### `LastSalesMonth`

A calculation/support table containing the `LastSalesMonth` value.

It can be used to dynamically determine the latest sales period available for reporting.

---

### `Report Refresh Date`

A report metadata table containing:

- `Date Last Refreshed`

This allows the dashboard to display the most recent data refresh date.

---

## 6. Relationship Design

The model uses relationships between shared dimensions and multiple fact tables.

The primary dimensions act as common filtering entities:

```text
dim_date
   │
   ├── fact_actual_estimates
   ├── fact_forecast_monthly
   ├── fact_freight_cost
   ├── Operational Expense
   ├── fact_pr_invoice_deductions
   └── fact_post_invoice_deductions

dim_product
   │
   ├── fact_actual_estimates
   ├── fact_manufacturing_cost
   ├── fact_gross_price
   └── deduction-related facts

dim_customer
   │
   ├── fact_actual_estimates
   ├── fact_forecast_monthly
   └── other customer-related analysis

dim_market
   │
   ├── fact_actual_estimates
   ├── fact_forecast_monthly
   ├── fact_freight_cost
   └── Operational Expense
```

This architecture allows the same business dimensions to filter multiple fact tables and enables cross-functional analysis.

---

## 7. Data Flow

```text
Source Data
     │
     ▼
Power Query
     │
     ├── Cleaning
     ├── Transformation
     ├── Data Type Handling
     └── Data Preparation
     │
     ▼
Power BI Data Model
     │
     ├── Dimensions
     │     ├── dim_date
     │     ├── dim_product
     │     ├── dim_customer
     │     └── dim_market
     │
     ├── Fact Tables
     │     ├── Actuals / Estimates
     │     ├── Forecasts
     │     ├── Costs
     │     ├── Pricing
     │     └── Deductions
     │
     └── Supporting Tables
           ├── Report Measure
           ├── P & L Rows
           ├── P & L Columns
           ├── Set Toggle
           └── Report Refresh Date
     │
     ▼
DAX Measures
     │
     ▼
Power BI Reports
```

---

## 8. Key Modeling Concepts

### Shared Dimensions

The use of common dimensions such as `dim_date`, `dim_product`, `dim_customer`, and `dim_market` allows different fact tables to be analyzed using consistent business attributes.

### Multiple Fact Tables

Instead of storing every metric in one large table, the model separates different business processes such as:

- Actual/estimated performance
- Forecasting
- Freight costs
- Manufacturing costs
- Gross pricing
- Invoice deductions
- Operational expenses

This makes the model more modular and easier to extend.

### Dedicated Measure Layer

The `Report Measure` table keeps DAX calculations separate from the underlying data tables.

### Financial Reporting Helpers

The `P & L Rows` and `P & L Columns` tables provide additional structure for building a dynamic Profit & Loss reporting experience.

---

## 9. Model Screenshot

![Power BI Data Model](../Screenshots/data-model.png)

The screenshot above shows the complete relationship structure between the model's dimensions, fact tables, measures, and supporting calculation tables.

---

## 10. Summary

The Power BI model is designed as a reusable analytical layer supporting multiple business functions.

The combination of:

- Shared dimensions
- Multiple business-process fact tables
- Centralized DAX measures
- Forecasting tables
- Financial reporting helper tables
- Interactive toggle tables
- Report metadata

provides a flexible foundation for Finance, Sales, Marketing, and Supply Chain reporting.