# OVERVIEW

This repository contains Big Query code to build an automated end-to-end solution that collates data from ad-platforms, google analytics, and defined targets to deliver daily budget, conversion run-rate and pacing insights. 

# DASHBOARD & GOOGLE SHEET TEMPLATE

Google Sheet Template - https://docs.google.com/spreadsheets/d/1h9bdhZ442xld7QvSfQDYv1YYpQl0XNnF8Tj9kE4sR7s/edit?usp=sharing

Data Studio Dashboard - https://datastudio.google.com/reporting/d2b7b27e-3607-4e35-b957-68562d146bc2

The Google Sheet template shared is for setting channels targets and combined targets 

## ARCHITECTURE OVERVIEW

The reporting pipeline transforms raw data feeds into production-ready BigQuery models used directly by Data Studio.

## REPORTING VIEWS & LINEAGE MATRIX

### FINAL REPORTING VIEWS 

| Final Reporting View (`3.x`) | Upstream Dependencies (`2.x` Staging Views) |
| :--- | :--- |
| `3.0_master-daily-view` | `2.0_stg-paidmedia`<br>`2.1_stg-googleanalytics-daily`<br>`2.2_stg-shopify-daily` |
| `3.1_channel-view` | `2.0_stg-paidmedia`<br>`2.1_stg-googleanalytics-daily`<br>`2.2_stg-shopify-daily` |
| `3.2_channel-performance-view-withtargets` | `3.1_channel-view`<br>`2.3_stg-channeltarget` |
| `3.3_overall-performance-view-withtargets` | `2.2_stg-shopify-daily`<br>`2.1_stg-googleanalytics-daily`<br>`2.0_stg-paidmedia`<br>`2.4_stg-alltargets` |

### KEY DATA HEALTH & RECONCILIATION VIEWS (AUDIT TOOLS) 

| Utility / Audit View (`1.x` / `2.x`) | Purpose | Upstream Dependencies |
| :--- | :--- | :--- |
| `1.4_mtd-keymetrics-tracker` | Dynamic Month-to-Date key metrics tracker up to yesterday | `2.1_stg-googleanalytics-daily`<br>`2.2_stg-shopify-daily`<br>`2.0_stg-paidmedia`<br>`2.4_stg-alltargets` |
| `2.7_stg-daily_v_monthly` | Closed-month data integrity & variance checker (Daily vs. Monthly rollups) | `2.1_stg-googleanalytics-daily`<br>`2.2_stg-shopify-daily`<br>`2.5_stg-googleanalytics-monthly`<br>`2.6_stg-shopify-monthly` |

## Dashboard Lineage & Page Mapping DASHBOARD & PAGE MAPPING

| Dashboard Page | Primary Data Source (BQ View) 
| :--- | :--- |
| **01. Executive Overview** | `3.0_master-daily-view` | 
| **02. E-Commerce * Site Efficiency** | `3.0_master-daily-view` + `3.3_overall-performance-view-withtargets` | 
| **03. MCD Pacing & MoM Trends** |  `3.3_overall-performance-view-withtargets` 
| **04. Channel Campaign & Deep Dive** | `3.2_channel-performance-view-withtargets` + `3.3_overall-performance-view-withtargets` | 
| **05. Appendix** | `3.2_channel-performance-view-withtargets` + `3.3_overall-performance-view-withtargets` | 

## DATA REQUIREMENTS & SCHEMA

Unifing data requirements into a single analytical view.

### 1. PAID MEDIA DELIVERY SCHEMA
Tracks platform-level performance (Google Ads, Meta, TikTok, etc.) at a daily level.

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `Date` | `DATE` | Event date (`YYYY-MM-DD`) |
| `Channel` | `STRING` | Paid channel identifier (e.g., `Paid Search`, `Paid Social`) |
| `Campaign` | `STRING` | Campaign identifier (e.g., `Branded`, `BFCM-2026`) |
| `Cost` | `NUMERIC` | Gross spend amount (£) |
| `Impressions` | `INTEGER` | Total ad impressions |
| `Clicks` | `INTEGER` | Total ad clicks |

### 2. GOOGLE ANALYTICS DELIVERY SCHEMA
Tracks site-level sessions and conversion activity for **all traffic sources** (Paid & Organic).

Requires 2 staging views: Daily (`2.1_stg-googleanalytics-daily`) and Monthly (`2.5_stg-googleanalytics-monthly`).

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `Date` | `DATE` | Event date (`YYYY-MM-DD`) |
| `Channel` | `STRING` | Channel grouping (`Paid Search`, `Paid Social`, `Organic`, `Email`) |
| `Sessions` | `INTEGER` | Total site visits |
| `GA Transactions`| `INTEGER` | Completed conversions/orders |
| `GA Revenue` | `NUMERIC` | Total attributed revenue (£) |

### 3. KEY BUSINESS METRICS DELIVERY SCHEMA (SHOPIFY/ERP)
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

### 4. CHANNEL TARGETS SCHEMA (GOOGLE SHEETS)
Manually inputted targets benchmarks maintained in Google Sheets (`Marketing_Targets_Master`).

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `Month` | `DATE` | Target month start date (`YYYY-MM-DD`) |
| `Channel` | `STRING` | Target marketing channel |
| `Campaign` | `STRING` | Target campaign identifier |
| `Spend Target` | `NUMERIC` | Allocated monthly budget (£) |
| `Conversions Target` | `INTEGER` | Target channel conversion volume |
| `Target CPA` | `NUMERIC` | Benchmark cost-per-acquisition (£) |
| `Revenue Target` | `NUMERIC` | Target channel revenue (£) |
| `Notes` | `STRING` | Strategic context notes |

### 5. ALL TARGETS SCHEMA (GOOGLE SHEETS)
Manually inputted targets benchmarks maintained in Google Sheets (`Marketing_Targets_Master`).

| Field Name | Type | Description |
| :--- | :--- | :--- |
| `Month` | `DATE` | Target month start date (`YYYY-MM-DD`) |
| `Spend Target` | `NUMERIC` | Allocated monthly store budget (£) |
| `Conversions Target` | `INTEGER` | Target total order volume |
| `Revenue Target` | `NUMERIC` | Target total revenue (£) |
| `Total_Customers_Target` | `INTEGER` | Target total customer volume |
| `New_Customers_Target` | `INTEGER` | Target new customer volume |
| `Profit Target` | `NUMERIC` | Target gross profit (£) |

## MASTER DATA DICTIONARY & FIELD MAPPING REFERENCE 

Maps all raw BigQuery schema fields and defines standardised calculation logic for downstream SQL modeling and Data Studio reporting.

### BigQuery Views

| Source Sheet Tab | Google Sheet Header | BigQuery Field Name | Data Type | Notes / Clean Transformations |
| :--- | :--- | :--- | :--- | :--- |
| **`PaidMedia`** | Date | `Date` | `DATE` | `YYYY-MM-DD` |
| | Channel | `Channel` | `STRING` | Channel name |
| | Campaign | `Campaign` | `STRING` | Campaign name grouping |
| | Cost | `Cost` | `NUMERIC` | Ad spend |
| | Impressions | `Impressions` | `INTEGER` | Total ad impressions |
| | Clicks | `Clicks` | `INTEGER` | Total ad clicks |
| **`GoogleAnalytics`** | Date | `Date` | `DATE` | `YYYY-MM-DD` |
| | Channel | `Channel` | `STRING` | Web traffic source grouping |
| | Sessions | `Sessions` | `INTEGER` | GA4 session counts |
| | GA Transactions | `GA_Transactions` | `INTEGER` | Web order conversions |
| | GA Revenue | `GA_Revenue` | `NUMERIC` | E-commerce revenue |
| **`Shopify`** | Date | `Date` | `DATE` | `YYYY-MM-DD` |
| | Shopify_Orders | `Shopify_Orders` | `INTEGER` | Order count from Store |
| | Shopify_Revenue | `Shopify_Revenue` | `NUMERIC` | Gross store revenue |
| | Total_Customers | `Total_Customers` | `INTEGER` | Total active buying customers |
| | New_Customers | `New_Customers` | `INTEGER` | First-time buyers |
| | **Profit** | **`Profit`** | **`NUMERIC`** | **Net/Gross Profit (£)** |
| **`ChannelsTargets`** | Month | `Month` | `DATE` | `YYYY-MM-DD` |
| | Channel | `Channel` | `STRING` | Target channel |
| | Campaign | `Campaign` | `STRING` | Target campaign |
| | Spend Target | `Spend_Target` | `NUMERIC` | Planned channel spend |
| | Conversions Target | `Conversions_Target` | `INTEGER` | Target channel conversions |
| | Target CPA | `Target_CPA` | `NUMERIC` | Target CPA benchmark |
| | Revenue Target | `Revenue_Target` | `NUMERIC` | Target channel revenue |
| | Notes | `Notes` | `STRING` | Context notes |
| **`AllTargets`** | Month | `Month` | `DATE` | `YYYY-MM-DD` |
| | Spend Target | `Spend_Target` | `NUMERIC` | Total store spend target |
| | Conversions Target | `Conversions_Target` | `INTEGER` | Total store order target |
| | Revenue Target | `Revenue_Target` | `NUMERIC` | Total store revenue target |
| | Total_Customers_Target | `Total_Customers_Target` | `INTEGER` | Total buyer target |
| | New_Customers_Target | `New_Customers_Target` | `INTEGER` | New buyer target |
| | **Profit_Target** | **`Profit_Target`** | **`NUMERIC`** | **Store Gross Profit Target** |

## 2. DATA STUDIO CALCULATED FIELDS

Documents calculated field specifications for the BigQuery-backed Looker Studio Dashboard.

### Data Source 1: `3.3_overall-performance-view-withtargets`

*Primary dataset for executive summary, overall store health, profitability, customer acquisition, and storewide target pacing.*

#### A. Profitability & Unit Economics

| Field Name | Type | Formula | Description |
| :--- | :--- | :--- | :--- |
| **POAS (Profit on Ad Spend)** | Percent | `SUM(shopify_profit) / SUM(ad_spend)` | Ratio of total gross profit to ad spend |
| **Blended ROAS** | Decimal | `SUM(shopify_revenue) / SUM(ad_spend)` | Return on ad spend across all revenue |
| **Gross Profit Margin %** | Percent | `SUM(shopify_profit) / SUM(shopify_revenue)` | Proportion of net revenue that is gross profit |
| **Blended CPA** | Currency (£) | `SUM(ad_spend) / SUM(shopify_orders)` | Cost per completed order across all channels |
| **Blended CAC** | Currency (£) | `SUM(ad_spend) / SUM(new_customers)` | Cost to acquire a new customer |
| **Average Order Value (AOV)** | Currency (£) | `SUM(shopify_revenue) / SUM(shopify_orders)` | Average revenue generated per order |
| **Storewide Revenue Per Session** | Currency (£) | `SUM(shopify_revenue) / SUM(sessions)` | Monetary value generated per site session |
| **Storewide Cost Per Session** | Currency (£) | `SUM(ad_spend) / SUM(sessions)` | Ad spend cost per site session driven |

#### B. Target Pacing & Delivery %

| Field Name | Type | Formula | Description |
| :--- | :--- | :--- | :--- |
| **Revenue Target Delivery %** | Percent | `SUM(shopify_revenue) / SUM(daily_revenue_target)` | Target delivery pacing for revenue |
| **Gross Profit Target Delivery %** | Percent | `SUM(shopify_profit) / SUM(daily_profit_target)` | Target delivery pacing for gross profit |
| **Spend Budget Utilization %** | Percent | `SUM(ad_spend) / SUM(daily_spend_target)` | Ad spend budget consumption vs. daily target |
| **Order Target Delivery %** | Percent | `SUM(shopify_orders) / SUM(daily_orders_target)` | Target delivery pacing for total orders |

#### C. Storewide Variances (£)

| Field Name | Type | Formula | Description |
| :--- | :--- | :--- | :--- |
| **Revenue Variance (£)** | Currency (£) | `SUM(shopify_revenue) - SUM(daily_revenue_target)` | Net variance vs. revenue target (+/-) |
| **Gross Profit Variance (£)** | Currency (£) | `SUM(shopify_profit) - SUM(daily_profit_target)` | Net variance vs. profit target (+/-) |
| **Spend Variance (£)** | Currency (£) | `SUM(ad_spend) - SUM(daily_spend_target)` | Net variance vs. budget target (+/-) |

#### D. Customer & Web Analytics

| Field Name | Type | Formula | Description |
| :--- | :--- | :--- | :--- |
| **New Customer Share %** | Percent | `SUM(new_customers) / SUM(total_customers)` | Proportion of orders placed by new customers |
| **Returning Customer Volume** | Integer | `SUM(total_customers) - SUM(new_customers)` | Volume of returning customers |
| **Ecommerce CVR (GA4 %)** | Percent | `SUM(ga_transactions) / SUM(sessions)` | Conversion rate according to GA4 |
| **GA4 Tracking Coverage Ratio %** | Percent | `SUM(ga_transactions) / SUM(shopify_orders)` | GA4 transaction capture rate vs. Shopify |

---

### Data Source 2: `3.2_channel-performance-view-withtargets`

*Secondary dataset for channel breakdowns, campaign performance, ad efficiency, and unit economics (using safe division calculations)*

#### A. Core Efficiency & Delivery

| Field Name | Type | Formula / Source | Description / Notes |
| :--- | :--- | :--- | :--- |
| **GA Conversion Rate (CVR)** | Percent | `SAFE_DIVIDE(SUM(Actual_Transactions), SUM(Actual_Sessions))` | Channel conversion rate using GA4 sessions |
| **GA Average Order Value (AOV)** | Currency (GBP) | `SAFE_DIVIDE(SUM(Actual_Revenue), SUM(Actual_Transactions))` | Average revenue per completed order |
| **Channel Revenue Delivery %** | Percent | `SAFE_DIVIDE(SUM(Actual_Revenue), SUM(Target_Revenue))` | Revenue pacing relative to channel target |
| **Channel Conversion Delivery %** | Percent | `SAFE_DIVIDE(SUM(Actual_Transactions), SUM(Target_Conversions))` | Order volume pacing relative to channel target |
| **Channel Revenue Variance (£)** | Currency (GBP) | `SUM(Actual_Revenue) - SUM(Target_Revenue)` | Net difference vs channel revenue target |

#### B. Native Run-Rate & Projections (Pacing)

| Field Name | Type | Formula / Source | Description / Notes |
| :--- | :--- | :--- | :--- |
| **Projected Channel Revenue (Month End)** | Currency (GBP) | `Projected_Channel_Revenue` (BigQuery View) | Native BigQuery month-end revenue projection |
| **Projected Channel Spend (Month End)** | Currency (GBP) | `Projected_Channel_Spend` (BigQuery View) | Native BigQuery month-end spend projection |
| **Projected Channel Revenue Delivery %** | Percent | `SAFE_DIVIDE(SUM(Projected_Channel_Revenue), SUM(Target_Revenue))` | Projected revenue as a % of target |
| **Projected Channel Spend Utilization %** | Percent | `SAFE_DIVIDE(SUM(Projected_Channel_Spend), SUM(Target_Spend))` | Projected budget consumption as a % of target |

---

### Data Source 3: `3.3_overall-performance-view-withtargets` (Storewide Projections)

Primary dataset for executive summary, overall store health, storewide projections, target pacing, and run-rate analytics*

#### Overall Run-Rate & Projections (Pacing)

| Field Name | Type | Formula / Source | Description / Notes |
| :--- | :--- | :--- | :--- |
| **Expected Spend (To Date)** | Currency (GBP) | `SUM(Target_Spend) * (MAX(current_day_of_month - 1) / MAX(days_in_current_month))` | Target ad spend run-rate to current day |
| **Projected Total Spend (Month End)** | Currency (GBP) | `(SUM(Actual_Spend) / MAX(current_day_of_month - 1)) * MAX(days_in_current_month)` | Estimated total spend by month-end |
| **Projected Spend Pacing %** | Percent | `((SUM(Actual_Spend) / MAX(current_day_of_month - 1)) * MAX(days_in_current_month)) / SUM(Target_Spend)` | Projected budget utilization rate |
| **Expected Revenue (To Date)** | Currency (GBP) | `SUM(Target_Revenue) * (MAX(current_day_of_month - 1) / MAX(days_in_current_month))` | Target revenue run-rate to current day |
| **Projected Total Revenue (Month End)** | Currency (GBP) | `(SUM(Actual_Shopify_Revenue) / MAX(current_day_of_month - 1)) * MAX(days_in_current_month)` | Estimated total Shopify revenue by month-end |
| **Projected Total Revenue Delivery %** | Percent | `((SUM(Actual_Shopify_Revenue) / MAX(current_day_of_month - 1)) * MAX(days_in_current_month)) / SUM(Target_Revenue)` | Projected % of revenue target achieved |
| **Expected Total Profit (To Date)** | Currency (GBP) | `SUM(Target_Profit) * (MAX(current_day_of_month - 1) / MAX(days_in_current_month))` | Target profit run-rate to current day |
| **Projected Total Profit (Month End)** | Currency (GBP) | `(SUM(Actual_Profit) / MAX(current_day_of_month - 1)) * MAX(days_in_current_month)` | Estimated gross profit by month-end |
| **Projected Profit Delivery %** | Percent | `((SUM(Actual_Profit) / MAX(current_day_of_month - 1)) * MAX(days_in_current_month)) / SUM(Target_Profit)` | Projected % of profit target achieved |
| **Expected Total Orders (To Date)** | Number | `SUM(Target_Conversions) * (MAX(current_day_of_month - 1) / MAX(days_in_current_month))` | Target order volume run-rate to current day |
| **Projected Total Orders (Month End)** | Number | `(SUM(Actual_Shopify_Orders) / MAX(current_day_of_month - 1)) * MAX(days_in_current_month)` | Estimated total orders by month-end |
| **Projected Orders Delivery %** | Percent | `((SUM(Actual_Shopify_Orders) / MAX(current_day_of_month - 1)) * MAX(days_in_current_month)) / SUM(Target_Conversions)` | Projected % of order target achieved |


