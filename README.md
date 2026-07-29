# 🚀 DTC Commercial Operations & Customer Acquisition 

Built an end-to-end solution using **Google BigQuery**, **GoogleSheets** and **Data Studio** that collates data from ad-platforms, google analytics, and defined targets to deliver daily budget, conversion run-rate and pacing insights. 

https://datastudio.google.com/reporting/d2b7b27e-3607-4e35-b957-68562d146bc2

## 🏗️ Architecture Overview

The reporting pipeline transforms raw data feeds into production-ready BigQuery models used directly by Looker Studio.

| Final Reporting View (`3.x`) | Upstream Dependencies (`2.x` Staging Views) |
| :--- | :--- |
| **`3.0_master-daily-view`** | `2.0_stg-paidmedia`<br>`2.1_stg-googleanalytics-daily`<br>`2.2_stg-shopify-daily` |
| **`3.1_channel-view`** | `2.0_stg-paidmedia`<br>`2.1_stg-googleanalytics-daily`<br>`2.2_stg-shopify-daily` |
| **`3.2_channel-performance-view-withtargets`** | `3.1_channel-view`<br>`2.3_stg-channeltarget` |
| **`3.3_overall-performance-view-withtargets`** | `2.5_stg-googleanalytics-monthly`<br>`2.6_stg-shopify-monthly`<br>`2.0_stg-paidmedia`<br>`2.4_stg-alltargets` |

## Reporting Views & Lineage Matrix

### Final Reporting Views (Looker Studio Core)

| Final Reporting View (`3.x`) | Upstream Dependencies (`2.x` Staging Views) |
| :--- | :--- |
| `3.0_master-daily-view` | `2.0_stg-paidmedia`<br>`2.1_stg-googleanalytics-daily`<br>`2.2_stg-shopify-daily` |
| `3.1_channel-view` | `2.0_stg-paidmedia`<br>`2.1_stg-googleanalytics-daily`<br>`2.2_stg-shopify-daily` |
| `3.2_channel-performance-view-withtargets` | `3.1_channel-view`<br>`2.3_stg-channeltarget` |
| `3.3_overall-performance-view-withtargets` | `2.2_stg-shopify-daily`<br>`2.1_stg-googleanalytics-daily`<br>`2.0_stg-paidmedia`<br>`2.4_stg-alltargets` |

---

### Key Data Health & Reconciliation Views (Frequent Audit Tools)

| Utility / Audit View (`1.x` / `2.x`) | Purpose | Upstream Dependencies |
| :--- | :--- | :--- |
| `1.4_mtd-keymetrics-tracker` | Dynamic Month-to-Date key metrics tracker up to yesterday | `2.1_stg-googleanalytics-daily`<br>`2.2_stg-shopify-daily`<br>`2.0_stg-paidmedia`<br>`2.4_stg-alltargets` |
| `2.7_stg-daily_v_monthly` | Closed-month data integrity & variance checker (Daily vs. Monthly rollups) | `2.1_stg-googleanalytics-daily`<br>`2.2_stg-shopify-daily`<br>`2.5_stg-googleanalytics-monthly`<br>`2.6_stg-shopify-monthly` |

## 📊 Data Requirements & Schema

The system unifies three distinct data requirements into a single analytical view.

### 1. Paid Media Delivery Schema (e.g., via Funnel.io)
Tracks platform-level performance (Google Ads, Meta, TikTok, etc.) at a daily level.

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `Date` | `DATE` | Event date (`YYYY-MM-DD`) |
| `Channel` | `STRING` | Paid channel identifier (e.g., `Paid Search`, `Paid Social`) |
| `Campaign` | `STRING` | Campaign identifier (e.g., `Branded`, `BFCM-2026`) |
| `Cost` | `NUMERIC` | Gross spend amount (£) |
| `Impressions` | `INTEGER` | Total ad impressions |
| `Clicks` | `INTEGER` | Total ad clicks |

### 2. Google Analytics Delivery Schema (via GA4 Export)
Tracks site-level sessions and conversion activity for **all traffic sources** (Paid & Organic).

Requires 2 staging views: Daily (`2.1_stg-googleanalytics-daily`) and Monthly (`2.5_stg-googleanalytics-monthly`).

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `Date` | `DATE` | Event date (`YYYY-MM-DD`) |
| `Channel` | `STRING` | Channel grouping (`Paid Search`, `Paid Social`, `Organic`, `Email`) |
| `Sessions` | `INTEGER` | Total site visits |
| `GA Transactions`| `INTEGER` | Completed conversions/orders |
| `GA Revenue` | `NUMERIC` | Total attributed revenue (£) |

### 3. Key Business Metrics Delivery Schema (Shopify/ERP)
Tracks key storewide business and customer metrics.

Requires 2 staging views: Daily (`2.2_stg-shopify-daily`) and Monthly (`2.6_stg-shopify-monthly`).

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `Date` | `DATE` | Event date (`YYYY-MM-DD`) |
| `Shopify Orders` | `INTEGER` | Total completed orders from Shopify |
| `Shopify Revenue` | `NUMERIC` | Total gross revenue (£) from Shopify |
| `Total_Customers` | `INTEGER` | Total purchasing customer volume |
| `New_Customers` | `INTEGER` | First-time purchasing customer volume |
| `Profit` | `NUMERIC` | Gross/Net profit (£) |

### 4. Channel Targets Schema (Google Sheets Input)
Human-managed target benchmarks maintained in Google Sheets (`Marketing_Targets_Master`).

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `Month` | `DATE` | Target month start date (`YYYY-MM-01`) |
| `Channel` | `STRING` | Target marketing channel |
| `Campaign` | `STRING` | Target campaign identifier |
| `Spend Target` | `NUMERIC` | Allocated monthly budget (£) |
| `Conversions Target` | `INTEGER` | Target channel conversion volume |
| `Target CPA` | `NUMERIC` | Benchmark cost-per-acquisition (£) |
| `Revenue Target` | `NUMERIC` | Target channel revenue (£) |
| `Notes` | `STRING` | Strategic context notes |

### 5. All Targets Schema (Google Sheets Input)
Human-managed storewide target benchmarks maintained in Google Sheets (`Marketing_Targets_Master`).

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `Month` | `DATE` | Target month start date (`YYYY-MM-01`) |
| `Spend Target` | `NUMERIC` | Allocated monthly store budget (£) |
| `Conversions Target` | `INTEGER` | Target total order volume |
| `Revenue Target` | `NUMERIC` | Target total revenue (£) |
| `Total_Customers_Target` | `INTEGER` | Target total customer volume |
| `New_Customers_Target` | `INTEGER` | Target new customer volume |
| `Profit Target` | `NUMERIC` | Target gross profit (£) |

## 📖 Master Data Dictionary & Field Mapping Reference

This reference maps all raw Google Sheets tabs to raw BigQuery schema fields and defines standardized calculation logic for downstream SQL modeling and Looker Studio reporting.

### 📑 1. Raw Layer (`raw_`) — Google Sheets to BigQuery Tables

| Source Sheet Tab | Google Sheet Header | BigQuery Field Name | Data Type | Notes / Clean Transformations |
| :--- | :--- | :--- | :--- | :--- |
| **`PaidMedia`** | Date | `Date` | `DATE` | Link Range: `PaidMedia!A1:F` |
| | Channel | `Channel` | `STRING` | Standardized channel string |
| | Campaign | `Campaign` | `STRING` | Campaign name grouping |
| | Cost | `Cost` | `NUMERIC` | Raw ad spend |
| | Impressions | `Impressions` | `INTEGER` | Total ad impressions |
| | Clicks | `Clicks` | `INTEGER` | Total ad clicks |
| **`GoogleAnalytics`** | Date | `Date` | `DATE` | Link Range: `GoogleAnalytics!A1:E` |
| | Channel | `Channel` | `STRING` | Web traffic source grouping |
| | Sessions | `Sessions` | `INTEGER` | GA4 session counts |
| | GA Transactions | `GA_Transactions` | `INTEGER` | Web order conversions |
| | GA Revenue | `GA_Revenue` | `NUMERIC` | E-commerce revenue |
| **`Shopify`** | Date | `Date` | `DATE` | Link Range: `Shopify!A1:F` |
| | Shopify_Orders | `Shopify_Orders` | `INTEGER` | Order count from Store |
| | Shopify_Revenue | `Shopify_Revenue` | `NUMERIC` | Gross store revenue |
| | Total_Customers | `Total_Customers` | `INTEGER` | Total active buying customers |
| | New_Customers | `New_Customers` | `INTEGER` | First-time buyers |
| | **Profit** | **`Profit`** | **`NUMERIC`** | **Net/Gross Profit (£)** |
| **`ChannelsTargets`** | Month | `Month` | `DATE` | Link Range: `ChannelsTargets!A1:H` |
| | Channel | `Channel` | `STRING` | Target channel |
| | Campaign | `Campaign` | `STRING` | Target campaign |
| | Spend Target | `Spend_Target` | `NUMERIC` | Planned channel spend |
| | Conversions Target | `Conversions_Target` | `INTEGER` | Target channel conversions |
| | Target CPA | `Target_CPA` | `NUMERIC` | Target CPA benchmark |
| | Revenue Target | `Revenue_Target` | `NUMERIC` | Target channel revenue |
| | Notes | `Notes` | `STRING` | Context notes |
| **`AllTargets`** | Month | `Month` | `DATE` | Link Range: `AllTargets!A1:G` |
| | Spend Target | `Spend_Target` | `NUMERIC` | Total store spend target |
| | Conversions Target | `Conversions_Target` | `INTEGER` | Total store order target |
| | Revenue Target | `Revenue_Target` | `NUMERIC` | Total store revenue target |
| | Total_Customers_Target | `Total_Customers_Target` | `INTEGER` | Total buyer target |
| | New_Customers_Target | `New_Customers_Target` | `INTEGER` | New buyer target |
| | **Profit_Target** | **`Profit_Target`** | **`NUMERIC`** | **Store Gross Profit Target** |

### 🧮 2. Staging & Master Layer Metrics (`stg_` / `rpt_`)

#### Core Blended Metrics
* **POAS (Profit on Ad Spend):** `Shopify Profit / Paid Media Cost`
* **ROAS (Return on Ad Spend):** `Shopify Revenue / Paid Media Cost`
* **Gross Profit Margin %:** `Shopify Profit / Shopify Revenue`
* **Blended CPA:** `Paid Media Cost / Shopify Orders`
* **Blended CAC (New Customers):** `Paid Media Cost / New Customers`

#### Profit & Pacing SQL Logic

```sql
-- Daily Target Run-Rate (Overall Profit Target / Days in Month)
COALESCE(t.Profit_Target, 0) / EXTRACT(DAY FROM LAST_DAY(r.date)) AS daily_profit_target,

-- Month-to-Date Profit Delivery %
SAFE_DIVIDE(SUM(s.Profit), MAX(t.Profit_Target)) AS pct_profit_target_delivered,

-- Projected Month-End Profit Variance (£)
((SAFE_DIVIDE(SUM(s.Profit), EXTRACT(DAY FROM DATE_SUB(CURRENT_DATE(), INTERVAL 1 DAY))) * EXTRACT(DAY FROM LAST_DAY(CURRENT_DATE()))) - MAX(t.Profit_Target)) AS projected_profit_variance
```

## 📊 Looker Studio Calculated Fields Documentation

This section documents the calculated field specifications for the BigQuery-backed Looker Studio Dashboard.

### Data Source 1: `3.3_overall-performance-view-withtargets`
*Primary dataset for executive summary, overall store health, profitability, customer acquisition, and storewide target pacing.*

#### A. Profitability & Unit Economics

| Field Name | Formula | Type | Description |
| :--- | :--- | :--- | :--- |
| **POAS (Profit on Ad Spend)** | `SUM(shopify_profit) / SUM(ad_spend)` | Decimal / % | Ratio of total gross profit to ad spend |
| **Blended ROAS** | `SUM(shopify_revenue) / SUM(ad_spend)` | Decimal | Return on ad spend across all revenue |
| **Gross Profit Margin %** | `SUM(shopify_profit) / SUM(shopify_revenue)` | Percent | Proportion of net revenue that is gross profit |
| **Blended CPA** | `SUM(ad_spend) / SUM(shopify_orders)` | Currency (£) | Cost per completed order across all channels |
| **Blended CAC** | `SUM(ad_spend) / SUM(new_customers)` | Currency (£) | Cost to acquire a new customer |
| **Average Order Value (AOV)** | `SUM(shopify_revenue) / SUM(shopify_orders)` | Currency (£) | Average revenue generated per order |
| **Storewide Revenue Per Session** | `SUM(shopify_revenue) / SUM(sessions)` | Currency (£) | Monetary value generated per site session |
| **Storewide Cost Per Session** | `SUM(ad_spend) / SUM(sessions)` | Currency (£) | Ad spend cost per site session driven |

#### B. Target Pacing & Delivery %

| Field Name | Formula | Type | Description |
| :--- | :--- | :--- | :--- |
| **Revenue Target Delivery %** | `SUM(shopify_revenue) / SUM(daily_revenue_target)` | Percent | Target delivery pacing for revenue |
| **Gross Profit Target Delivery %** | `SUM(shopify_profit) / SUM(daily_profit_target)` | Percent | Target delivery pacing for gross profit |
| **Spend Budget Utilization %** | `SUM(ad_spend) / SUM(daily_spend_target)` | Percent | Ad spend budget consumption vs. daily target |
| **Order Target Delivery %** | `SUM(shopify_orders) / SUM(daily_orders_target)` | Percent | Target delivery pacing for total orders |

#### C. Storewide Variances (£)

| Field Name | Formula | Type | Description |
| :--- | :--- | :--- | :--- |
| **Revenue Variance (£)** | `SUM(shopify_revenue) - SUM(daily_revenue_target)` | Currency (£) | Net variance vs. revenue target (+/-) |
| **Gross Profit Variance (£)** | `SUM(shopify_profit) - SUM(daily_profit_target)` | Currency (£) | Net variance vs. profit target (+/-) |
| **Spend Variance (£)** | `SUM(ad_spend) - SUM(daily_spend_target)` | Currency (£) | Net variance vs. budget target (+/-) |

#### D. Customer & Web Analytics

| Field Name | Formula | Type | Description |
| :--- | :--- | :--- | :--- |
| **New Customer Share %** | `SUM(new_customers) / SUM(total_customers)` | Percent | Proportion of orders placed by new customers |
| **Returning Customer Volume** | `SUM(total_customers) - SUM(new_customers)` | Integer | Volume of returning customers |
| **Ecommerce CVR (GA4 %)** | `SUM(ga_transactions) / SUM(sessions)` | Percent | Conversion rate according to GA4 |
| **GA4 Tracking Coverage Ratio %** | `SUM(ga_transactions) / SUM(shopify_orders)` | Percent | GA4 transaction capture rate vs. Shopify |

### Data Source 2: `3.2_channel-performance-view-withtargets`
*Secondary dataset for channel breakdowns, campaign performance, ad efficiency, and unit economics.*

#### A. Core Efficiency & Static Delivery
* **GA Conversion Rate (CVR)**
  * **Type:** Percent
  * **Formula:** `SUM(Actual_Transactions) / SUM(Actual_Sessions)`
* **GA Average Order Value (AOV)**
  * **Type:** Currency (GBP)
  * **Formula:** `SUM(Actual_Revenue) / SUM(Actual_Transactions)`
* **Channel Revenue Delivery %**
  * **Type:** Percent
  * **Formula:** `SUM(Actual_Revenue) / SUM(Target_Revenue)`
* **Channel Conversion Delivery %**
  * **Type:** Percent
  * **Formula:** `SUM(Actual_Transactions) / SUM(Target_Conversions)`
* **Channel Revenue Variance (£)**
  * **Type:** Currency (GBP)
  * **Formula:** `SUM(Actual_Revenue) - SUM(Target_Revenue)`
* **Channel Conversion Variance (Orders)**
  * **Type:** Number
  * **Formula:** `SUM(Actual_Transactions) - SUM(Target_Conversions)`

#### B. Run-Rate & Projections (Pacing)
* **Expected Channel Spend (To Date)**
  * **Type:** Currency (GBP)
  * **Formula:** `SUM(Target_Spend) * ((EXTRACT(DAY FROM CURRENT_DATE()) - 1) / DATETIME_DIFF(DATETIME_ADD(DATETIME_TRUNC(MAX(Month), MONTH), INTERVAL 1 MONTH), DATETIME_TRUNC(MAX(Month), MONTH), DAY))`
* **Expected Channel Revenue (To Date)**
  * **Type:** Currency (GBP)
  * **Formula:** `SUM(Target_Revenue) * ((EXTRACT(DAY FROM CURRENT_DATE()) - 1) / DATETIME_DIFF(DATETIME_ADD(DATETIME_TRUNC(MAX(Month), MONTH), INTERVAL 1 MONTH), DATETIME_TRUNC(MAX(Month), MONTH), DAY))`
* **Projected Channel Revenue (Month End)**
  * **Type:** Currency (GBP)
  * **Formula:** `(SUM(Actual_Revenue) / (EXTRACT(DAY FROM CURRENT_DATE()) - 1)) * DATETIME_DIFF(DATETIME_ADD(DATETIME_TRUNC(MAX(Month), MONTH), INTERVAL 1 MONTH), DATETIME_TRUNC(MAX(Month), MONTH), DAY)`
* **Projected Channel Revenue Delivery %**
  * **Type:** Percent
  * **Formula:** `((SUM(Actual_Revenue) / (EXTRACT(DAY FROM CURRENT_DATE()) - 1)) * DATETIME_DIFF(DATETIME_ADD(DATETIME_TRUNC(MAX(Month), MONTH), INTERVAL 1 MONTH), DATETIME_TRUNC(MAX(Month), MONTH), DAY)) / SUM(Target_Revenue)`
* **Expected Channel Conversions (To Date)**
  * **Type:** Number
  * **Formula:** `SUM(Target_Conversions) * ((EXTRACT(DAY FROM CURRENT_DATE()) - 1) / DATETIME_DIFF(DATETIME_ADD(DATETIME_TRUNC(MAX(Month), MONTH), INTERVAL 1 MONTH), DATETIME_TRUNC(MAX(Month), MONTH), DAY))`
* **Projected Channel Conversions (Month End)**
  * **Type:** Number
  * **Formula:** `(SUM(Actual_Transactions) / (EXTRACT(DAY FROM CURRENT_DATE()) - 1)) * DATETIME_DIFF(DATETIME_ADD(DATETIME_TRUNC(MAX(Month), MONTH), INTERVAL 1 MONTH), DATETIME_TRUNC(MAX(Month), MONTH), DAY)`
* **Projected Channel Conversions Delivery %**
  * **Type:** Percent
  * **Formula:** `((SUM(Actual_Transactions) / (EXTRACT(DAY FROM CURRENT_DATE()) - 1)) * DATETIME_DIFF(DATETIME_ADD(DATETIME_TRUNC(MAX(Month), MONTH), INTERVAL 1 MONTH), DATETIME_TRUNC(MAX(Month), MONTH), DAY)) / SUM(Target_Conversions)`

### Data Source 3: `3.3_overall-performance-view-withtargets` (Storewide Projections)

#### Overall Run-Rate & Projections (Pacing)
* **Expected Spend (To Date)**
  * **Type:** Currency (GBP)
  * **Formula:** `SUM(Target_Spend) * (MAX(current_day_of_month - 1) / MAX(days_in_current_month))`
* **Projected Total Spend (Month End)**
  * **Type:** Currency (GBP)
  * **Formula:** `(SUM(Actual_Spend) / MAX(current_day_of_month - 1)) * MAX(days_in_current_month)`
* **Projected Spend Pacing %**
  * **Type:** Percent
  * **Formula:** `((SUM(Actual_Spend) / MAX(current_day_of_month - 1)) * MAX(days_in_current_month)) / SUM(Target_Spend)`
* **Expected Revenue (To Date)**
  * **Type:** Currency (GBP)
  * **Formula:** `SUM(Target_Revenue) * (MAX(current_day_of_month - 1) / MAX(days_in_current_month))`
* **Projected Total Revenue (Month End)**
  * **Type:** Currency (GBP)
  * **Formula:** `(SUM(Actual_Shopify_Revenue) / MAX(current_day_of_month - 1)) * MAX(days_in_current_month)`
* **Projected Total Revenue Delivery %**
  * **Type:** Percent
  * **Formula:** `((SUM(Actual_Shopify_Revenue) / MAX(current_day_of_month - 1)) * MAX(days_in_current_month)) / SUM(Target_Revenue)`
* **Expected Total Profit (To Date)**
  * **Type:** Currency (GBP)
  * **Formula:** `SUM(Target_Profit) * (MAX(current_day_of_month - 1) / MAX(days_in_current_month))`
* **Projected Total Profit (Month End)**
  * **Type:** Currency (GBP)
  * **Formula:** `(SUM(Actual_Profit) / MAX(current_day_of_month - 1)) * MAX(days_in_current_month)`
* **Projected Profit Delivery %**
  * **Type:** Percent
  * **Formula:** `((SUM(Actual_Profit) / MAX(current_day_of_month - 1)) * MAX(days_in_current_month)) / SUM(Target_Profit)`
* **Expected Total Orders (To Date)**
  * **Type:** Number
  * **Formula:** `SUM(Target_Conversions) * (MAX(current_day_of_month - 1) / MAX(days_in_current_month))`
* **Projected Total Orders (Month End)**
  * **Type:** Number
  * **Formula:** `(SUM(Actual_Shopify_Orders) / MAX(current_day_of_month - 1)) * MAX(days_in_current_month)`
* **Projected Orders Delivery %**
  * **Type:** Percent
  * **Formula:** `((SUM(Actual_Shopify_Orders) / MAX(current_day_of_month - 1)) * MAX(days_in_current_month)) / SUM(Target_Conversions)`
