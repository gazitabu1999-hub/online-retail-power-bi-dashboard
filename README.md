# online-retail-power-bi-dashboard
Power BI dashboard analyzing 536K+ UK e-commerce transactions. Includes data cleaning (Power Query), star-schema modeling, and RFM customer segmentation built with DAX. Key finding: top 20% of customers drive 63% of revenue.
Dashboard Pages

**1. Sales Overview** — revenue trend, geographic breakdown, top products
**2. Customer Segmentation (RFM)** — Recency/Frequency/Monetary scoring, customer segments, revenue concentration
**3. Returns & Cancellations** — return rate, returned products, return patterns by country/time

 Key Insights

- **Champions (the top RFM segment) are ~20% of customers but drive ~63% of total revenue** — a clear 80/20 pattern worth acting on with a loyalty program.
- **Genuine product return rate is 4.64%**, not the 8.40% a naive calculation shows — ~47% of "returns" in the raw data are actually Amazon fee reversals, manual entries, and bank charges rather than real product returns.
- **Two single bulk-order cancellations (£246K combined) in Jan and Dec 2011** explain most of the spikes in the returns trend and the top-2 "most returned" products — likely data-entry or test orders rather than genuine customer behavior.
- Sales are heavily UK-concentrated (~91%), with a clear seasonal peak in November ahead of the holiday season.

Data Cleaning (Power Query)

- Removed 5,268 exact duplicate rows
- Fixed data types (InvoiceNo/StockCode set to Text, not Number, since mixed alphanumeric values like "C536379" and "POST" would break numeric typing)
- Flagged (not deleted) cancellations via `IsCancelled` (InvoiceNo starting with "C")
- Flagged non-product StockCodes (POST, DOT, M, BANK CHARGES, AMAZONFEE, etc.) via `IsNonProduct`, so postage/fee rows don't pollute product-level analysis
- Flagged £0/negative UnitPrice rows via `IsPriceAdjustment` (write-offs/samples, not real sales)
- Handled null CustomerID (~25% of rows, guest checkouts) by keeping it as a true null rather than filling with a placeholder ID — added a `CustomerType` (Guest/Registered) column instead, to avoid corrupting customer-level metrics like DISTINCTCOUNT and RFM scoring
- Trimmed whitespace on Description/Country; standardized null Descriptions

Data Model

Built as a star schema rather than one flat table:

- **Fact_Sales** — transaction-level fact table (`Sales` = Quantity × UnitPrice)
- **Cust_Dim** — one row per customer (CustomerID, Country, CustomerType, RFM columns)
- **Dim_Date** — calendar table built with `CALENDAR()`, marked as a Date table for time intelligence

Relationships: `Cust_Dim (1) → Fact_Sales (*)` and `Dim_Date (1) → Fact_Sales (*)`, both single-direction cross-filtering.

Files in this repo

- `Online_Retail_Dashboard.pbix` — the Power BI file
- `/screenshots` — page exports
- `Online_Retail.xlsx` — source data
