# 🛒 eCommerce Orders — Data Cleaning & Analysis (PostgreSQL)

## Project Overview

This project demonstrates a full ETL (Extract, Transform, Load) workflow on a simulated eCommerce orders dataset. The objective was to clean a dirty raw dataset, resolve data quality issues, and extract actionable business insights using PostgreSQL.

The dataset contains 130 orders across three markets (Canada, United States, United Kingdom) covering product categories including Skins, Phone Cases, and Accessories.

---

## 🔧 Technologies Used

- PostgreSQL (pgAdmin 4)
- SQL — data cleaning, transformation, aggregation
- Excel — output validation

---

## 📁 Project Structure

| File | Description |
|---|---|
| `ecommerce_orders_raw.csv` | Original dirty dataset (130 rows) |
| `ecommerce_orders_clean.csv` | Cleaned, analysis-ready dataset |
| `SQL Scripts` | All cleaning and analysis queries |
| `README.md` | Project overview and findings |

---

## ⚙️ ETL Process

### Step 1 — Extract
Loaded raw CSV into PostgreSQL using pgAdmin 4 import tool into table `ecommerce_orders_raw`.

### Step 2 — Transform (Data Cleaning)

The following issues were identified and resolved:

| Issue | Detail | Fix Applied |
|---|---|---|
| Duplicate row | Order 1001 appeared twice | Removed using ROW_NUMBER() OVER PARTITION |
| Inconsistent date format | Order 1016 stored as MM/DD/YYYY | Standardised to YYYY-MM-DD, cast to DATE |
| Case inconsistencies | product_category had mixed case (SKINS, phone cases, accessories) | Corrected using INITCAP() |
| Case inconsistencies | payment_method had paypal in lowercase | Corrected to PayPal |
| Price outlier | Order 1098 unit_price was 2999 instead of 29.99 | Corrected decimal point error |
| Missing values | Order 1121 had no customer_id, name, or email | Flagged with Unknown placeholders — order revenue retained |

### Step 3 — Load
Cleaned data stored in `ecommerce_orders_clean` table for analysis.

---

## 📊 Key Findings

**Revenue by Product Category**
| Category | Total Revenue |
|---|---|
| Skins | $1,988.43 |
| Phone Cases | $1,595.54 |
| Accessories | $707.13 |

Skins is the highest revenue category despite not being the highest unit volume — driven by higher average unit prices.

**Revenue by Market**
| Country | Total Revenue |
|---|---|
| Canada | $2,169.78 |
| United States | $1,640.48 |
| United Kingdom | $480.85 |

Canada is the strongest market, contributing 46% of total revenue.

**Top 5 Products by Units Sold**
| Product | Units Sold | Revenue |
|---|---|---|
| dbrand Grip Stand | 16 | $392.34 |
| Grip Case - Samsung Galaxy S24 | 12 | $405.88 |
| Screen Protector - iPhone 15 Pro | 11 | $164.89 |
| Grip Case - iPhone 15 Pro | 10 | $349.90 |
| Screen Protector - iPad Pro | 10 | $149.90 |

**Key Metrics**
- Average Order Value: $33.01
- Repeat Purchase Rate: 4.03%

The low repeat purchase rate (4%) suggests strong new customer acquisition but an opportunity to improve retention through post-purchase engagement.

---

## 💡 Business Insights

1. **Skins drives the most revenue** — prioritise new skin launches and seasonal campaigns around this category
2. **Canada outperforms the US per capita** — consider localised promotions to deepen Canadian market penetration
3. **Accessories have low revenue despite high unit volume** — low price point products; consider bundling with higher-value items to increase AOV
4. **4% repeat rate is low for DTC** — post-purchase email flows and loyalty incentives could significantly improve LTV

---

## 🧮 SQL Highlights

```sql
-- Remove duplicates using ROW_NUMBER()
CREATE TABLE ecommerce_orders_clean AS
SELECT * FROM (
  SELECT *,
    ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY order_id) AS row_num
  FROM ecommerce_orders_raw
) AS ranked
WHERE row_num = 1;

-- Standardise date format and cast to DATE
UPDATE ecommerce_orders_clean
SET order_date = '2024-01-25'
WHERE order_id = 1016;

ALTER TABLE ecommerce_orders_clean
ALTER COLUMN order_date TYPE DATE
USING order_date::DATE;

-- Fix case inconsistencies
UPDATE ecommerce_orders_clean
SET product_category = INITCAP(product_category);

UPDATE ecommerce_orders_clean
SET payment_method = 'PayPal'
WHERE LOWER(payment_method) = 'paypal';

-- Revenue by category
SELECT
  product_category,
  SUM(quantity * unit_price * (1 - discount_pct / 100)) AS total_revenue
FROM ecommerce_orders_clean
GROUP BY product_category
ORDER BY total_revenue DESC;
```

---

## About

Built as part of a data analytics portfolio to demonstrate SQL-based ETL, data quality management, and business insight generation from eCommerce order data.
