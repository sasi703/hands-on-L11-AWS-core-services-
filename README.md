
# 🧠 AWS Core Services Hands-On: S3, Glue, CloudWatch, and Athena  

**Course:** ITCS 6190/8190 – Cloud Computing for Data Analysis  
**Instructor:** Marco Vieira  
**Semester:** Fall 2025  

---

## 📘 Objective  

This hands-on lab demonstrates how to use core AWS analytics services — **S3**, **IAM**, **Glue**, **CloudWatch**, and **Athena** — to ingest, catalog, and query e-commerce sales data in a cloud-native environment.  

You will:  
1. Upload raw CSV files into **Amazon S3**.  
2. Configure an **IAM Role** with Glue and S3 permissions.  
3. Create a **Glue Crawler** to automatically catalog the datasets.  
4. Monitor crawler execution using **CloudWatch Logs**.  
5. Query the crawled database in **Amazon Athena**.  

Dataset: [E-Commerce Sales Data (Kaggle)](https://www.kaggle.com/datasets/thedevastator/unlock-profits-with-e-commercesales-data)

---

## ⚙️ AWS Workflow Steps  

### 1️⃣ S3 Setup  
- Created two buckets:  
  - `ecommerce-raw-data` – for raw CSV uploads.  
  - `ecommerce-processed-data` – for Athena query outputs.  
- Uploaded the following source files:  
  - `Amazon Sale Report.csv`  
  - `International sale Report.csv`  
  - `Sale Report.csv`  
  - `Cloud Warehouse Compersion Chart.csv`  
  - `Expense IIGF.csv`  
  - `P L March 2021.csv`  
  - `May-2022.csv`

📸 *Screenshot:* `screenshots/s3_buckets.png`

---

### 2️⃣ IAM Role  
- Created `GlueCrawlerRole`.  
- Attached permissions:  
  - `AWSGlueServiceRole`  
  - `AmazonS3FullAccess`  
  - `CloudWatchLogsFullAccess`  

📸 *Screenshot:* `screenshots/iam_role.png`

---

### 3️⃣ Glue Crawler  
- Created crawler `ecommerce-raw-crawler`.  
- Source: `ecommerce-raw-data` bucket.  
- Target DB: `ecommerce_analysis_db`.  
- IAM Role: `GlueCrawlerRole`.  
- Run crawler → 7 tables discovered and added to Glue Catalog.

📸 *Screenshot:* `screenshots/glue_crawler.png`

---

### 4️⃣ CloudWatch Monitoring  
- Verified crawler logs under `/aws-glue/crawlers/ecommerce-raw-crawler`.  
- Confirmed tables created successfully and crawler state = READY.  

📸 *Screenshot:* `screenshots/cloudwatch_log.png`

---

### 5️⃣ Athena Configuration  
- Set query results location to `s3://ecommerce-processed-data/athena-results/`.  
- Selected database `ecommerce_analysis_db`.  
- Main dataset used: `amazon_sale_report_csv`.

📸 *Screenshot:* `screenshots/athena_query_results.png`

---

## 🧮 Queries Executed in Athena ( LIMIT 10 )

### **1️⃣ Cumulative Sales Over Time for 2022**
```sql
WITH t AS (
  SELECT 
    date_parse(Date, '%m-%d-%y') AS order_dt,
    CAST(Amount AS DOUBLE) AS sales
  FROM ecommerce_analysis_db.amazon_sale_report_csv
  WHERE Date IS NOT NULL
)
SELECT 
  order_dt,
  SUM(sales) OVER (ORDER BY order_dt) AS cumulative_sales
FROM t
WHERE year(order_dt) = 2022
ORDER BY order_dt
LIMIT 10;
```
➡ **Output:** [`results/cumulative_sales_over_time_2022.csv`](results/cumulative_sales_over_time_2022.csv)

---

### **2️⃣ Geographic Sales Analysis by State**
```sql
SELECT 
  "ship-state" AS state,
  SUM(CAST(Amount AS DOUBLE)) AS total_sales
FROM ecommerce_analysis_db.amazon_sale_report_csv
GROUP BY "ship-state"
ORDER BY total_sales DESC
LIMIT 10;
```
➡ **Output:** [`results/geographic_analysis.csv`](results/geographic_analysis.csv)

---

### **3️⃣ Sales Distribution by Category**
```sql
SELECT 
  Category,
  SUM(CAST(Amount AS DOUBLE)) AS total_sales,
  COUNT(DISTINCT "Order ID") AS total_orders
FROM ecommerce_analysis_db.amazon_sale_report_csv
GROUP BY Category
ORDER BY total_sales DESC
LIMIT 10;
```
➡ **Output:** [`results/category_sales_distribution.csv`](results/category_sales_distribution.csv)

---

### **4️⃣ Top 3 Selling Products per Category**
```sql
WITH ranked AS (
  SELECT 
    Category,
    Style,
    SUM(CAST(Amount AS DOUBLE)) AS total_sales,
    RANK() OVER (PARTITION BY Category ORDER BY SUM(CAST(Amount AS DOUBLE)) DESC) AS rnk
  FROM ecommerce_analysis_db.amazon_sale_report_csv
  GROUP BY Category, Style
)
SELECT Category, Style, total_sales
FROM ranked
WHERE rnk <= 3
LIMIT 10;
```
➡ **Output:** [`results/top3_products.csv`](results/top3_products.csv)

---

### **5️⃣ Monthly Sales Growth Analysis**
```sql
WITH monthly AS (
  SELECT 
    date_trunc('month', date_parse(Date, '%m-%d-%y')) AS month,
    SUM(CAST(Amount AS DOUBLE)) AS total_sales
  FROM ecommerce_analysis_db.amazon_sale_report_csv
  GROUP BY 1
)
SELECT
  month,
  total_sales,
  (total_sales - LAG(total_sales) OVER (ORDER BY month)) 
     / NULLIF(LAG(total_sales) OVER (ORDER BY month), 0) AS sales_growth_rate
FROM monthly
ORDER BY month
LIMIT 10;
```
➡ **Output:** [`results/monthly_sales_growth.csv`](results/monthly_sales_growth.csv)

---

## 📊 Result Files  

| Query | Output CSV | Insight |
|:--|:--|:--|
| 1 | `cumulative_sales_over_time_2022.csv` | Shows running total of sales in 2022. |
| 2 | `geographic_analysis.csv` | Top states by total sales. |
| 3 | `category_sales_distribution.csv` | Best-performing product categories. |
| 4 | `top3_products.csv` | Top 3 styles per category by sales. |
| 5 | `monthly_sales_growth.csv` | Month-to-month sales trend. |

---

## 📸 Screenshots  
Embed your screenshots in the repo:

```markdown
![S3 Buckets](screenshots/s3_buckets.png)
![IAM Role](screenshots/iam_role.png)
![CloudWatch Logs](screenshots/cloudwatch_log.png)
![Athena Query Results](screenshots/athena_query_results.png)
```

---

## 🧾 Conclusion  

This hands-on lab demonstrated the complete AWS data analytics pipeline:  
- **S3** provided secure and scalable storage.  
- **Glue** automatically cataloged datasets for Athena.  
- **CloudWatch** enabled monitoring and log visibility.  
- **Athena** allowed SQL-based serverless analytics directly on S3 data.  

The integration of these services provides an end-to-end cloud data solution capable of ingesting, preparing, and analyzing large datasets with minimal infrastructure overhead.  

---

**Author:** Atluri Sasi Vadana

