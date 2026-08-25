# E-Commerce-Customer-Lifecycle-RFM-Segmentation
# 📊 E-Commerce Customer Lifecycle & RFM Segmentation

![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=flat&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Wrangling-150458?style=flat&logo=pandas)
![Seaborn](https://img.shields.io/badge/Seaborn-Data%20Viz-4c8cb5?style=flat)
![Status](https://img.shields.io/badge/Status-Completed-success)

## 📌 Project Overview
This project delivers an end-to-end data pipeline and customer segmentation analysis (**RFM Analysis**) using transactional data from an online retailer (~540k raw records). 

The goal was to transform uncleaned transactional logs into actionable business insights, identifying high-value customers, assessing churn risk, and testing the applicability of the **Pareto Principle (80/20 Rule)** to the retailer's revenue.

---

## 🛠️ Tech Stack & Libraries
* **Language:** Python
* **Data Manipulation:** `pandas`, `numpy`
* **Data Visualization:** `seaborn`, `matplotlib`, `plotly`
* **Environment:** Jupyter Notebook / VS Code

---

## 🏗️ Data Pipeline & Methodology

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

## 📈 Key Findings & Business Insights

### 🎯 1. Validation of the Pareto Principle (80/20 Rule)
By computing the cumulative revenue distribution across all sorted customer profiles, the analysis proved that:
> **The top 20.0% of customers generate 74.6% of total revenue.**

```python
# Cumulative revenue calculation snippet
monetary_sorted = rfm['Monetary'].sort_values(ascending=False)
cumulative_revenue = monetary_sorted.cumsum() / monetary_sorted.sum() * 100

top_20_index = int(len(monetary_sorted) * 0.2)
top_20_revenue = cumulative_revenue.iloc[top_20_index]
# Output: 74.6%
