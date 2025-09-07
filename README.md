# AWS Serverless ETL – Orders JSON → Parquet

![AWS](https://img.shields.io/badge/AWS-Lambda-orange?logo=amazonaws&logoColor=white)  
![Python](https://img.shields.io/badge/Python-3.9-blue?logo=python&logoColor=white)  
![Pandas](https://img.shields.io/badge/Pandas-Data%20Wrangling-lightgrey?logo=pandas)  
![Serverless](https://img.shields.io/badge/Serverless-ETL-success?logo=serverless)

This repo contains an **AWS Lambda function** that powers a serverless ETL pipeline.  
It listens to **S3 events**, processes raw **nested JSON order data**, transforms it into a **flattened tabular structure**, stores the output in **Parquet format**, and triggers an **AWS Glue Crawler** so that the data is queryable in **Amazon Athena**.

---

## 🚀 Architecture (GitHub-safe)

Simple flow (text):
S3 (raw JSON upload)
↓
Lambda function (reads JSON, flattens)
↓
Convert to Parquet (Pandas + PyArrow)
↓
Write Parquet to S3 (orders_parquet_datalake/)
↓
Trigger Glue Crawler (etl_pipeline_crawler)
↓
Query via Athena


---

## ⚙️ Workflow

1. **Extract** → Lambda is triggered when a JSON file is uploaded to S3.  
2. **Transform** →  
   - Flatten nested JSON (`customer` + `products`).  
   - Convert data into a Pandas DataFrame.  
   - Save as Parquet (optimized columnar storage).  
3. **Load** → Write processed file back to S3 in `orders_parquet_datalake/`.  
4. **Catalog** → Trigger Glue Crawler (`etl_pipeline_crawler`) to update schema.  
5. **Query** → Use Amazon Athena to run SQL queries on fresh data.

---

## 📂 Repo Structure
├── lambda_function.py # Lambda handler with logging & error handling
└── README.md # Documentation


---

## 🧑‍💻 Lambda Function

Key parts:

- **flatten()** → expands nested JSON into tabular rows.  
- **lambda_handler()** → reads file from S3, flattens JSON, saves Parquet, starts Glue Crawler.

Example output columns:
- `order_id`, `order_date`, `total_amount`
- `customer_id`, `customer_name`, `email`, `address`
- `product_id`, `product_name`, `category`, `price`, `quantity`

---

## 🛠️ Tech Stack

- **AWS Lambda** → serverless compute  
- **Amazon S3** → raw + processed storage  
- **Pandas** → data wrangling  
- **PyArrow** → Parquet conversion  
- **AWS Glue Crawler** → schema catalog  
- **Amazon Athena** → SQL queries

---

## 📊 Example Flow

### Sample Input JSON
```json
[
  {
    "order_id": "ORD123",
    "order_date": "2025-09-08",
    "total_amount": 250,
    "customer": {
      "customer_id": "CUST01",
      "name": "John Doe",
      "email": "john@example.com",
      "address": "NY"
    },
    "products": [
      {
        "product_id": "P001",
        "name": "Paracetamol",
        "category": "Medicine",
        "price": 50,
        "quantity": 2
      }
    ]
  }
]

#Output in S3

[s3://<bucket>/orders_parquet_datalake/orders_ETL_20250908_120000.parquet]

#Athena Query

SELECT customer_name, SUM(total_amount) AS total_revenue
FROM orders_parquet_datalake
GROUP BY customer_name;








