# 📊 E-Commerce Customer Lifecycle & RFM Segmentation

![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=flat&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Wrangling-150458?style=flat&logo=pandas)
![Seaborn](https://img.shields.io/badge/Seaborn-Data%20Viz-4c8cb5?style=flat)
![Status](https://img.shields.io/badge/Status-Completed-success)

##  Project Overview
This project delivers an end-to-end data pipeline and customer segmentation analysis (**RFM Analysis**) using transactional data from an online retailer (~540k raw records). 

The goal was to transform uncleaned transactional logs into actionable business insights, identifying high-value customers, assessing churn risk, and testing the applicability of the **Pareto Principle (80/20 Rule)** to the retailer's revenue.

---

## Tech Stack & Libraries
* **Language:** Python
* **Data Manipulation:** `pandas`, `numpy`
* **Data Visualization:** `seaborn`, `matplotlib`, `plotly`
* **Environment:** Jupyter Notebook / VS Code

---

## Data Pipeline & Methodology

### 1. Data Audit & Sanitization (EDA & Wrangling)
* **Missing Values Audit:** Identified **135,080 rows missing `CustomerID`** (~25% of total transactions). These rows were removed to ensure relational data integrity, reducing the dataset from 541,909 to **406,829 valid rows**.
* **Type Casting:** Converted `CustomerID` from `float64` to `int64` to correct identifier formats.
* **Business Rule Filtering:** Removed cancellations, negative quantities, and zero/negative unit prices by enforcing `Quantity > 0` and `UnitPrice > 0`.
* **Feature Engineering:** Created row-level transaction value: 
  $$\text{TotalRevenue} = \text{Quantity} \times \text{UnitPrice}$$

### 2. RFM Aggregation Engine
To avoid distortion from static historical dates (2010–2011 dataset), a dynamic snapshot reference date was defined:
$$\text{snapshot\_date} = \text{Max}(\text{InvoiceDate}) + 1\text{ day}$$

Data was then aggregated at the individual customer level (`CustomerID`):
* **Recency (R):** Days elapsed between the customer's last purchase and `snapshot_date`.
* **Frequency (F):** Total count of unique invoices (`InvoiceNo.nunique()`).
* **Monetary (M):** Total monetary value generated (`TotalRevenue.sum()`).

---

## Key Findings & Business Insights

### 1. Validation of the Pareto Principle (80/20 Rule)
By computing the cumulative revenue distribution across all sorted customer profiles, the analysis proved that:
> **The top 20.0% of customers generate 74.6% of total revenue.**

```python
# Cumulative revenue calculation snippet
monetary_sorted = rfm['Monetary'].sort_values(ascending=False)
cumulative_revenue = monetary_sorted.cumsum() / monetary_sorted.sum() * 100

top_20_index = int(len(monetary_sorted) * 0.2)
top_20_revenue = cumulative_revenue.iloc[top_20_index]
# Output: 74.6% 
```

The Pareto Principle is also verified with Spending Distribuition (Excluding Top 5%) and Monetary Dispersion with Outliers, given that most of Customers spend less money, but more frequent, while few customers are responsible for heavy payments.
 
<img width="1151" height="440" alt="image" src="https://github.com/user-attachments/assets/02cc9952-eb21-4c3a-81b4-9da0c7b83641" />

### 🧩 2. $5 \times 5$ RxF Matrix Density Analysis

The $5 \times 5$ Recency vs. Frequency (RxF) heatmap provides a clear blueprint of customer distribution across different stages of the lifecycle:

* **Champions Core ($R=5, F=5$):** Contains **439 highly active clients** who purchase frequently and have transacted very recently. This segment represents the key revenue engine and retention anchor for the business.
* **High-Value Loyalty ($R=4, F=4$ & $R=4, F=5$):** Demonstrates strong mid-to-high retention, with **258 and 248 clients** respectively, serving as the prime pipeline to convert into *Champions* through targeted upsell campaigns.
* **Onboarding & Second-Purchase Opportunity ($R=1, F=1$):** Registers **363 one-time buyers** who dropped off after their initial purchase. This represents the primary target for automated post-purchase onboarding workflows.
* **Low Attrition in Frequent Buyers ($R=1, F=5$):** Only **12 high-frequency clients** dropped to the lowest Recency score ($R=1$). This extremely low attrition rate confirms strong historical retention and brand loyalty among core accounts.
* 
<img width="872" height="629" alt="image" src="https://github.com/user-attachments/assets/de5b06e7-d675-4b9a-90c7-585163214574" />
