# RFM Customer Segmentation Analysis

A practical implementation of the RFM (Recency, Frequency, Monetary) model for e-commerce customer stratification, built on real transaction data from a Chinese online retail brand.

## Overview

| Step | Description |
|------|-------------|
| 1. Data Overview | Load and inspect raw transaction records |
| 2. Data Cleaning | Remove refunded orders, extract key fields |
| 3. Dimension Scoring | Construct and score R, F, M values per customer |
| 4. Score Calculation | Apply scoring rules using `pd.cut` |
| 5. Customer Segmentation | Classify all customers into 8 strategic tiers |

## Dataset

**Source file:** `PYTHON-RFM实战数据.xlsx`

E-commerce order records from a Chinese retail brand ("数据不吹牛"), covering January – June 2019.

**Data source:** The dataset is a teaching sample published by the Chinese data analytics education account *数据不吹牛* ("Data Don't Brag") as part of their Python RFM practical tutorial series, distributed via the Alibaba Cloud Developer Community:
- Part 1: https://developer.aliyun.com/article/911309
- Part 2: https://developer.aliyun.com/article/911317

| Field | Type | Description |
|-------|------|-------------|
| 品牌名称 | string | Brand name |
| 买家昵称 | string | Buyer username |
| 付款日期 | datetime | Payment timestamp |
| 订单状态 | string | Order status (completed / refunded) |
| 实付金额 | int | Actual amount paid (CNY) |
| 邮费 | int | Shipping fee |
| 省份 / 城市 | string | Province / city |
| 购买数量 | int | Purchase quantity |

- **Raw records:** 28,833
- **After removing refunds:** 27,793
- **Unique customers:** 25,420
- **Reference date for R calculation:** 2019-07-01

## RFM Metric Construction

### R — Recency (days since last purchase)
Days elapsed between a customer's most recent purchase date and 2019-07-01. Smaller R → more recent → higher score.

| Score | R range | Meaning |
|-------|---------|---------|
| 5 | [0, 30) | Purchased within the last month |
| 4 | [30, 60) | 30–59 days ago |
| 3 | [60, 90) | 60–89 days ago |
| 2 | [90, 120) | 90–119 days ago |
| 1 | [120, ∞) | 120+ days ago |

### F — Frequency (purchase days)
Number of **distinct days** on which the customer placed a successful order (multiple orders in one day = 1 purchase day). Higher F → more loyal → higher score.

| Score | F value | Meaning |
|-------|---------|---------|
| 1 | 1 | Purchased on 1 day |
| 2 | 2 | 2 days |
| 3 | 3 | 3 days |
| 4 | 4 | 4 days |
| 5 | 5+ | 5 or more days |

### M — Monetary (average spend per purchase day)
Total spend divided by purchase-day count (M = total amount / F). Scored in ¥50 increments, validated against the actual distribution plot.

| Score | M range (CNY) | Meaning |
|-------|--------------|---------|
| 1 | [0, 50) | Avg spend < ¥50 |
| 2 | [50, 100) | ¥50–99 |
| 3 | [100, 150) | ¥100–149 |
| 4 | [150, 200) | ¥150–199 |
| 5 | [200, ∞) | ¥200+ |

## Customer Segmentation

Each customer is tagged by whether their F, M, and R scores exceed the respective means (1 = above average, 0 = below). The three binary flags form an index (F×100 + M×10 + R×1) that maps to 8 customer tiers:

| F | M | R | Segment | Description |
|---|---|---|---------|-------------|
| 1 | 1 | 1 | **重要价值客户** (High-Value) | Recent, frequent, high spend — core VIPs |
| 1 | 1 | 0 | **重要价值流失预警客户** (At-Risk High Value) | High-value but recently inactive — urgent retention |
| 1 | 0 | 1 | **消费潜力客户** (Spend-Potential) | Recent, frequent, lower spend — upsell candidates |
| 1 | 0 | 0 | **一般客户** (General) | Frequent but inactive and low-spend |
| 0 | 1 | 1 | **提频深耕客户** (Frequency-Boost) | Recent, high spend, infrequent — deepen engagement |
| 0 | 1 | 0 | **高消费找回客户** (Win-Back High Spend) | High historic spend but dormant — recovery priority |
| 0 | 0 | 1 | **新客户** (New Customer) | Recent first-time or low-activity buyers |
| 0 | 0 | 0 | **流失客户** (Churned) | Inactive, infrequent, low spend — likely lost |

## Key Findings

| Segment | Customers | Customer % | Revenue (CNY) | Revenue % |
|---------|-----------|------------|---------------|-----------|
| 高消费找回客户 (Win-Back High Spend) | 7,338 | 28.9% | 1,338,153 | **38.1%** |
| 流失客户 (Churned) | 6,680 | 26.3% | 444,617 | 12.7% |
| 提频深耕客户 (Frequency-Boost) | 5,427 | 21.3% | 981,893 | 27.9% |
| 新客户 (New Customer) | 4,224 | 16.6% | 270,869 | 7.7% |
| 重要价值客户 (High-Value) | 756 | 3.0% | 269,230 | 7.7% |
| 消费潜力客户 (Spend-Potential) | 450 | 1.8% | 64,075 | 1.8% |
| 重要价值流失预警客户 (At-Risk High Value) | 360 | 1.4% | 116,665 | 3.3% |
| 一般客户 (General) | 185 | 0.7% | 25,803 | 0.7% |

**Key insights:**
- The top two groups by revenue contribution — Win-Back High Spend (38.1%) and Frequency-Boost (27.9%) — together account for **66%** of total revenue despite being separate retention challenges.
- True High-Value customers (VIPs) are only 3% of the base but generate 7.7% of revenue; protecting them is critical.
- Over a quarter of customers (26.3%) are classified as churned, representing a significant win-back opportunity if cost-effective campaigns can be designed.

## Setup & Usage

### Requirements

```
pandas
numpy
seaborn
matplotlib
openpyxl
```

Install with:

```bash
pip install pandas numpy seaborn matplotlib openpyxl
```

### Run

1. Place `PYTHON-RFM实战数据.xlsx` in the path referenced in the notebook (`Downloads/Document/`), or update the `pd.read_excel()` path.
2. Open `RFM模型实战.ipynb` in Jupyter Notebook or JupyterLab.
3. Run all cells sequentially.

The final output is the `rmf` DataFrame with each customer's R/F/M raw values, scores, and segment label, plus summary tables for customer count and revenue contribution per segment.
