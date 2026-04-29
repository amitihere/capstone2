# BigBasket Product Pricing & Category Analysis

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python)
![Tableau](https://img.shields.io/badge/Tableau-Public-orange?logo=tableau)
![GitHub](https://img.shields.io/badge/Version%20Control-GitHub-black?logo=github)
![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)

---
Tableau Public URL - https://public.tableau.com/app/profile/aneesh.amiti/viz/bigbasket_dashboard/Dashboard2
---
[Download the Tableau Workbook By Clicking , This is a twbx zip file](https://github.com/user-attachments/files/27189996/bigbasket_dashboard.twbx.zip)


## Team

| Name | Role |
|------|------|
| Amiti Aneesh | Tableau Dashboard & Visualization |
| Lakshith | Exploratory & Statistical Analysis |
| Chanakya Sinde | Data Cleaning & ETL Pipeline |
| Sai Praneeth Sharma | Business Insights & Final Report |

> **Repository:** `capstone2` &nbsp;|&nbsp; **Domain:** E-Commerce / Retail &nbsp;|&nbsp; **Tech Stack:** Python · Tableau · GitHub

---

## Problem Statement

> *How can BigBasket attract more customers ? What are the key drivers of product pricing, discount patterns, and customer ratings across categories and brands on BigBasket, and how can these insights help an online grocery retailer optimize pricing strategy, improve product assortment, and boost customer satisfaction?*

---

## Project Overview

This project analyzes the **BigBasket Entire Product List Dataset** — a real-world product catalog dataset from India's largest online grocery supermarket, containing over **23,000 product listings** across multiple categories and brands.

The analysis uncovers insights into:

- Pricing trends across categories and sub-categories
- Discount patterns (difference between market price and sale price)
- Product ratings and their relationship to pricing and brand
- Brand performance across categories
- Product type and category distribution

The goal is to generate actionable business insights that help:

- Optimize pricing and discount strategies
- Identify high-performing brands and categories
- Understand what drives higher customer ratings
- Improve product assortment decisions

All analysis is performed using **Python** (ETL + EDA + Statistics) and **Tableau** (Interactive Dashboard).

---

## Dataset

| Property | Details |
|----------|---------|
| **Source** | [Kaggle – BigBasket Entire Product List (surajjha101)](https://www.kaggle.com/datasets/surajjha101/bigbasket-entire-product-list-28k-datapoints) |
| **Total Records** | 23,000+ product listings |
| **Columns** | 9 (raw) + engineered features |
| **Domain** | Online Grocery Retail — India |
| **Format** | Single CSV |

### Raw Columns

| Column | Type | Description |
|--------|------|-------------|
| `index` | Numerical | Simple row index identifier |
| `product` | Text | Title of the product as listed on BigBasket |
| `category` | Categorical | Broad category the product belongs to |
| `sub_category` | Categorical | Sub-category within the main category |
| `brand` | Categorical | Brand name of the product |
| `sale_price` | Numerical | Price at which the product is sold on BigBasket (₹) |
| `market_price` | Numerical | Standard market price of the product (₹) |
| `type` | Categorical | Product type classification |
| `rating` | Numerical | Consumer rating of the product (out of 5) |
| `description` | Text | Detailed product description |

### Engineered Features (created during ETL)

| Feature | Description |
|---------|-------------|
| `discount_amount` | `market_price − sale_price` |
| `discount_pct` | `(discount_amount / market_price) × 100` |
| `price_segment` | Binned price range: Budget / Mid-range / Premium |
| `is_discounted` | Boolean — `True` if `sale_price < market_price` |
| `rating_segment` | Categorized rating: Low (< 3) / Medium (3–4) / High (> 4) |

---

## Repository Structure

```
capstone2/
│
├── data/
│   ├── raw/                   # Original unedited dataset
│   └── processed/             # Cleaned and transformed output
│
├── notebooks/
│   ├── 01_data_overview.ipynb          # Initial data inspection
│   ├── 02_cleaning.ipynb               # ETL pipeline & feature engineering
│   ├── 03_eda.ipynb                    # Exploratory data analysis
│   ├── 04_statistical_analysis.ipynb   # Correlation, regression, hypothesis tests
│   └── 05_final_load_prep.ipynb        # KPI computation & Tableau-ready export
│
├── tableau/
│   ├── screenshots/           # Dashboard screenshots
│   └── dashboard_links.md     # Tableau Public URL
│
├── docs/
│   └── data_dictionary.md     # Full data dictionary
│
├── reports/
│   └── final_report.pdf       # Final project report (PDF)
│
└── README.md
```

---

## Data Preprocessing & ETL Pipeline

The raw dataset contains several real-world data quality issues handled in `notebooks/02_cleaning.ipynb`:

**Issues Found & Fixed:**
- Missing values in `rating`, `brand`, and `description` columns — handled via imputation or flagging
- Inconsistent `brand` name formatting — standardized to title case
- `sale_price` and `market_price` stored as strings with `₹` symbols — converted to float
- Products with `sale_price > market_price` — flagged as data anomalies
- Duplicate product entries — identified and removed
- `description` column — retained for reference, excluded from numerical analysis
- `index` column — dropped (non-analytical row identifier)

**Feature Engineering:**
New columns derived to enrich analysis and meet the 8+ meaningful column requirement, documented step-by-step in the cleaning notebook.

---

## Analysis Performed

### 1. Pricing & Discount Analysis
- Average sale price vs market price by category
- Discount percentage distribution across sub-categories
- Identification of highest and lowest discounted product segments
- Price segment (Budget / Mid-range / Premium) breakdown

### 2. Category & Brand Performance
- Top categories by product count and average rating
- Brand-wise average discount and rating comparison
- Sub-category level product distribution
- Most listed brands across the catalog

### 3. Rating Analysis
- Distribution of product ratings across the catalog
- Correlation between discount percentage and product rating
- Category-wise average rating comparison
- High-rated vs low-rated product profiling

### 4. Statistical Analysis
- Correlation between `sale_price`, `market_price`, `discount_pct`, and `rating`
- Hypothesis testing: Do higher-discounted products receive better ratings?
- Regression analysis: What features predict a product's rating?
- ANOVA: Is there a significant difference in ratings across categories?

---

## KPIs Tracked

| KPI | Description |
|-----|-------------|
| Average Discount (%) | Mean discount across all products |
| Average Sale Price (₹) | Mean sale price across catalog |
| Average Rating | Mean consumer rating across all products |
| Top Category by Product Count | Category with the most listed products |
| % Products Discounted | Share of products with sale price below market price |
| Top Brand by Avg Rating | Best-rated brand across the catalog |
| Price Segment Distribution | Share of Budget / Mid-range / Premium products |

---

## Key Business Insights

1. Certain **categories offer significantly higher discounts** than others, suggesting competitive pricing pressure
2. **Premium-priced products** do not always receive higher ratings — price alone does not drive satisfaction
3. A small number of **top brands** dominate multiple categories, indicating strong brand concentration
4. Products with **moderate discounts (10–30%)** tend to have better ratings than heavily discounted ones
5. Several **sub-categories have very low average ratings**, presenting an opportunity for assortment improvement
6. **Missing ratings** are concentrated in newer or less-purchased product types
7. The catalog shows a **long tail of products** — a few categories dominate product count
8. **Budget-segment products** have the highest volume but lower average ratings

---

## Business Recommendations

1. **Revisit discount strategy** for low-rated, heavily discounted products — deep discounts may signal poor quality to customers
2. **Improve product descriptions** in sub-categories with low ratings to set better customer expectations
3. **Focus on high-rating, mid-range brands** for marketing and homepage promotion
4. **Curate underperforming sub-categories** by reducing low-rated SKUs and replacing with better alternatives
5. **Use price segmentation** to design targeted promotions for Budget vs Premium customer segments

---

## Contribution Matrix

| Member | Contribution |
|--------|-------------|
| Amiti Aneesh | Data ingestion, Tableau dashboard design (`01_extraction.ipynb`, `05_final_load_prep.ipynb`) |
| Lakshith | trend analysis, statistical tests, KPI computation (`04_statistical_analysis.ipynb`) |
| Chanakya Sinde | EDA, ETL pipeline, feature engineering (`02_cleaning.ipynb`, `03_eda.ipynb`) |
| Sai Praneeth Sharma | Business insights, recommendations, final report & presentation |

