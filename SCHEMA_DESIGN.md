# Olist Analytics - Star Schema Design

## Overview

This project implements a star schema for analyzing Brazilian e-commerce transaction data combined with real-time exchange rate APIs.

---

## Data Model

### FACT TABLE: `fct_orders`

The central fact table representing each order transaction.

| Column | Type | Description |
|----------|----------|----------|
| order_id | VARCHAR | Primary key - Unique order identifier |
| customer_id | VARCHAR | Foreign key to customer dimension |
| order_status | VARCHAR | Status: pending, processing, delivered, canceled |
| order_purchase_timestamp | TIMESTAMP | When order was placed |
| order_delivered_customer_date | TIMESTAMP | When customer received order |
| order_value | DECIMAL | Total order value in BRL |
| payment_status | VARCHAR | Payment completion status |
| days_to_delivery | INT | Days from order to delivery |

**Grain:** One row per order

**Rows:** ~99,441 orders

**Purpose:** Analyze order metrics, delivery performance, and payment success.

---

### DIMENSION: `dim_customers`

Customer master dimension with aggregated metrics.

| Column | Type | Description |
|----------|----------|----------|
| customer_key | INT | Surrogate key |
| customer_id | VARCHAR | Business key |
| customer_city | VARCHAR | City name |
| customer_state | VARCHAR | State code |
| lifetime_orders | INT | Total orders by customer |
| lifetime_value | DECIMAL | Total spent by customer |
| first_order_date | DATE | First purchase date |

**Grain:** One row per unique customer

**Rows:** ~99,441 customers

**Purpose:** Analyze customer behavior, retention, and customer lifetime value.

---

### DIMENSION: `dim_sellers`

Seller master dimension with performance metrics.

| Column | Type | Description |
|----------|----------|----------|
| seller_key | INT | Surrogate key |
| seller_id | VARCHAR | Business key |
| seller_city | VARCHAR | City name |
| seller_state | VARCHAR | State code |
| total_orders | INT | Orders fulfilled |
| total_revenue | DECIMAL | Revenue generated |
| avg_order_value | DECIMAL | Average order size |

**Grain:** One row per unique seller

**Rows:** ~3,095 sellers

**Purpose:** Analyze seller performance and geographic distribution.

---

### DIMENSION: `dim_date`

Time dimension for temporal analysis.

| Column | Type | Description |
|----------|----------|----------|
| date_key | INT | Surrogate key |
| calendar_date | DATE | Calendar date |
| year | INT | Year |
| month | INT | Month (1–12) |
| day | INT | Day of month |
| is_weekend | BOOLEAN | TRUE if Saturday/Sunday |
| month_name | VARCHAR | Month name |

**Grain:** One row per calendar day

**Rows:** ~1,461 days

**Purpose:** Support time-based aggregations and trend analysis.

---

## Entity Relationships

fct_orders
├── customer_id → dim_customers.customer_id
├── seller_id   → dim_sellers.seller_id
├── order_date  → dim_date.calendar_date
└── metrics: order_value, payment_status, days_to_delivery


---

## Data Sources

| Source | Type | Refresh Frequency |
|----------|----------|----------|
| Olist E-Commerce Dataset | CSV Files (8) | Historical Load |
| Exchange Rate API | REST API | Daily |

---

## Key Metrics

### Order Metrics
- Total Order Count
- Daily Order Volume
- Order Status Distribution

### Revenue Metrics
- Daily Revenue
- Monthly Revenue Growth
- Average Order Value (AOV)

### Customer Metrics
- Customer Lifetime Value (CLV)
- Customer Geographic Distribution

### Seller Metrics
- Revenue by Seller
- Order Count by Seller

### Delivery Metrics
- Average Delivery Time
- On-Time Delivery Rate
- Delivery Performance by Region

### Payment Metrics
- Payment Success Rate
- Payment Method Distribution

### Foreign Exchange Metrics
- FX Impact Analysis

---

## Architecture

CSV Files (Olist Dataset)
           │
           ▼
      ETL Pipeline
           │
           ▼
      PostgreSQL
           │
           ▼
      Star Schema
           │
           ▼
       Power BI
           │
           ▼
 Business Insights
