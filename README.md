### Overview

**Supplier Performance Analysis** is a data-driven project that evaluates how effectively suppliers contribute to an organization’s profitability and operational success.
Using **Python**, **SQL**, and **Power BI**, it assesses performance across cost, delivery timeliness, inventory efficiency, and profitability to support smarter procurement decisions.

---

### 🚀 Objectives

* Identify top- and low-performing suppliers.
* Analyze the effect of purchase volume on unit cost.
* Measure inventory turnover to improve operational efficiency.
* Assess the impact of freight and pricing on supplier profitability.
* Provide actionable insights for supplier renegotiation and sourcing.

---

### 🧩 Tools & Technologies

| Tool / Library           | Purpose                            |
| ------------------------ | ---------------------------------- |
| **Python**               | Data processing & automation       |
| **Pandas**               | Data cleaning & manipulation       |
| **SQL (SQLite)**         | Data extraction & aggregation      |
| **Power BI**             | Dashboard creation & visualization |
| **Matplotlib / Seaborn** | Visual analytics & EDA             |
| **Jupyter Notebook**     | Data exploration & documentation   |

---

### 🏗️ Project Workflow

The workflow integrates data ingestion, transformation, analysis, and visualization.

![Project Workflow](WhatsApp Image 2025-11-09 at 14.34.17_c7884ccb.jpg)

1. **Data Extraction:** Retrieve supplier, purchase, and sales data from `inventory.db`.
2. **Data Cleaning:** Handle missing values, validate quantities, and standardize supplier IDs.
3. **Data Aggregation:** Combine purchase, sales, and freight tables using SQL CTEs.
4. **Exploratory Analysis:** Visualize supplier trends using Matplotlib and Seaborn.
5. **Dashboard Creation:** Load aggregated results into Power BI for performance tracking.

---

### 💾 Database Connection Example

```python
from sqlalchemy import create_engine
import pandas as pd
import sqlite3

# Create SQLAlchemy engine
engine = create_engine('sqlite:///inventory.db')

# Connect to SQLite database
conn = sqlite3.connect('inventory.db')

# List tables in the database
tables = pd.read_sql_query("""
SELECT name 
FROM sqlite_master 
WHERE type='table' 
ORDER BY name ASC
""", conn)
tables
```

---

### 🧮 SQL Aggregation Query

This SQL query aggregates purchase, sales, and freight data to form the final `vendor_sales_summary` table:

```sql
WITH FreightSummary AS (
    SELECT VendorNumber, SUM(Freight) AS FreightCost
    FROM vendor_invoice
    GROUP BY VendorNumber
),
PurchaseSummary AS (
    SELECT p.VendorNumber, p.VendorName, p.Brand, p.Description,
           p.PurchasePrice, pp.Volume, pp.Price AS ActualPrice,
           SUM(p.Quantity) AS TotalPurchaseQuantity,
           SUM(p.Dollars) AS TotalPurchaseDollars
    FROM purchases p
    JOIN purchase_prices pp ON p.Brand = pp.Brand
    WHERE p.PurchasePrice > 0
    GROUP BY p.VendorNumber, p.VendorName, p.Brand, p.Description,
             p.PurchasePrice, pp.Price, pp.Volume
),
SalesSummary AS (
    SELECT VendorNo, Brand,
           SUM(SalesDollars) AS TotalSalesDollars,
           SUM(SalesPrice) AS TotalSalesPrice,
           SUM(SalesQuantity) AS TotalSalesQuantity,
           SUM(ExciseTax) AS TotalExciseTax
    FROM sales
    GROUP BY VendorNo, Brand
)
SELECT ps.*, ss.TotalSalesQuantity, ss.TotalSalesDollars,
       ss.TotalSalesPrice, ss.TotalExciseTax, fs.FreightCost
FROM PurchaseSummary ps
LEFT JOIN SalesSummary ss ON ps.VendorNumber = ss.VendorNo AND ps.Brand = ss.Brand
LEFT JOIN FreightSummary fs ON ps.VendorNumber = fs.VendorNumber
ORDER BY ps.TotalPurchaseDollars DESC;
```

---

### 📂 Folder Structure

```
Supplier_Performance_Analysis/
│
├── assets/
│   ├── workflow.png
│   ├── dashboard_1.png
│   └── dashboard_2.png
│
├── data/
│   └── inventory.db
│
├── sql/
│   └── queries.sql
│
├── notebooks/
│   └── Supplier_Performance_Analysis.ipynb
│
├── scripts/
│   └── ingest_to_db.py
│
├── powerbi/
│   └── dashboard.pbix
│
├── report/
│   └── final_report.pdf
│
├── logs/
│   └── ingestion.log
│
└── README.md
```

---

### 📈 Dashboard Insights

The Power BI dashboard visualizes critical supplier KPIs for quick decision-making.

![Dashboard Overview](assets/dashboard_1.png)
![Vendor KPI Comparison](assets/dashboard_2.png)

#### Key Metrics Displayed:

* Top suppliers by sales and profit contribution
* Underperforming suppliers and cost inefficiencies
* Freight cost share per supplier
* Inventory turnover ratios
* Profitability variance between brands

---

### 📊 Key Outcomes

* Ranked suppliers based on delivery, cost, and profitability.
* Identified opportunities for pricing renegotiation.
* Revealed inefficiencies in freight and purchase practices.
* Provided clear insights for procurement strategy optimization.
