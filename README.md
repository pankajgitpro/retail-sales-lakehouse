# retail-sales-lakehouse
End-to-end data engineering pipeline on Databricks implementing the medallion architecture (bronze → silver → gold) for the AdventureWorks sales dataset, with data quality checks and a dimensional star schema for analytics.

# Azure Lakehouse Data Engineering Pipeline – AdventureWorks

## Project Overview

This project demonstrates an **end-to-end Azure data engineering pipeline** built using **Azure Data Factory, Azure Data Lake Storage Gen2, Azure Databricks, PySpark, and Unity Catalog**.

The objective of the project is to ingest raw AdventureWorks data from an API, store it in a scalable data lake, transform and clean the data using Apache Spark, and create analytics-ready datasets using a **Medallion Architecture (Bronze → Silver → Gold)**.

The final Gold layer contains a **star schema and analytical data marts** that can be used for reporting, business intelligence, and downstream analytics.

## Architecture

```text
Source API
    │
    ▼
Azure Data Factory
    │
    │  Data Ingestion
    ▼
Azure Data Lake Storage Gen2
    │
    │  Bronze / Raw Data
    ▼
Unity Catalog
    │
    │  Governed access to ADLS Gen2
    ▼
Azure Databricks
    │
    │  PySpark / Apache Spark
    ▼
Silver Layer
Cleaned & Transformed Data
    │
    ▼
Gold Layer
Star Schema + Data Marts
    │
    ▼
Analytics / BI / Reporting
```

<img width="1672" height="941" alt="Azure Medallion Pipeline Architecture" src="https://github.com/user-attachments/assets/e1275ab2-9208-4dd5-9cf0-3e46a44b2042" />


## Technology Stack

| Technology                       | Purpose                                                   |
| -------------------------------- | --------------------------------------------------------- |
| **Azure Data Factory**           | Orchestrates ingestion of source data into the data lake  |
| **Azure Data Lake Storage Gen2** | Stores raw and processed datasets                         |
| **Azure Databricks**             | Distributed data processing and transformation platform   |
| **Apache Spark / PySpark**       | Performs scalable ETL transformations                     |
| **Unity Catalog**                | Provides governed access between Databricks and ADLS Gen2 |
| **Parquet**                      | Columnar storage format used for processed datasets       |

## Data Pipeline

### 1. Data Ingestion – Bronze Layer

Data is first extracted from the source **API using Azure Data Factory** and loaded into **Azure Data Lake Storage Gen2**.

The Bronze layer stores the source data with minimal modification, preserving the raw datasets for downstream processing.

The project includes datasets for:

* Calendar
* Customers
* Products
* Product Categories
* Product Subcategories
* Sales
* Returns
* Sales Territories

Databricks accesses the datasets stored in ADLS Gen2 through **Unity Catalog volumes**, providing governed and centralized access to the lake data.

---

### 2. Data Transformation – Silver Layer

The raw Bronze datasets are processed in **Azure Databricks using PySpark**.

The Silver layer focuses on data cleansing, standardization, type conversion, deduplication, and creation of useful derived attributes.

Key transformations include:

**Calendar**

* Standardized date formats
* Generated Year, Month, Month Name, Quarter and Day of Week
* Created an `IsWeekend` indicator
* Removed duplicate dates

**Customers**

* Converted annual income into a numeric datatype
* Standardized birth dates
* Calculated customer age
* Created customer age buckets
* Standardized gender and marital status
* Cleaned names and email addresses
* Converted home ownership values into Boolean values
* Removed duplicate customer records

**Products**

* Handled missing product size and style values
* Standardized product cost and price
* Calculated product profit margin
* Cleaned product descriptions
* Removed duplicate products

**Sales**

* Standardized order and stock dates
* Removed duplicate order-line records
* Filtered invalid sales quantities

**Returns**

* Standardized return dates
* Filtered invalid return quantities

**Territories**

* Cleaned Region, Country and Continent fields
* Removed duplicate territory records

Processed datasets are stored in **Parquet format** for efficient analytical processing.

---

## Data Quality Checks

A data quality checkpoint validates the relationships between the sales data and major dimension datasets.

The pipeline checks for sales records containing non-existent:

* `ProductKey`
* `CustomerKey`
* `TerritoryKey`

These checks help detect referential-integrity issues before data is promoted into the Gold layer.

---

## Gold Layer – Dimensional Model

The Gold layer transforms the cleaned datasets into an analytics-ready **Star Schema**.

### Dimension Tables

* `dim_date`
* `dim_customer`
* `dim_product`
* `dim_territory`

The Product dimension is denormalized by combining product, category, and subcategory information to reduce the number of joins required by downstream analytics.

### Fact Tables

#### `fact_sales`

Grain: **one record per sales order line**

The fact table includes calculated business metrics:

```text
Revenue = OrderQuantity × ProductPrice
Cost    = OrderQuantity × ProductCost
Profit  = Revenue - Cost
```

#### `fact_returns`

Contains product return transactions including:

* Return Date
* Territory
* Product
* Return Quantity

---

## Analytical Data Marts

In addition to the dimensional model, the project creates several business-focused analytical marts.

### Monthly Sales by Territory and Category

Aggregates:

* Total Revenue
* Total Profit
* Units Sold

Grouped by:

* Year / Month
* Region
* Country
* Product Category

<img width="1536" height="1024" alt="Star Schema Retail Data Map" src="https://github.com/user-attachments/assets/f28099b5-076c-4ae0-8eb4-74f15004abc1" />



### Product Return Rate

Calculates product-level return performance using:

```text
Return Rate = Units Returned / Units Sold
```

This dataset can help identify products with unusually high return rates.

### Customer RFM Analysis

Creates customer-level **RFM metrics**:

* **Recency** – Days since the customer's latest purchase
* **Frequency** – Number of unique orders
* **Monetary** – Total customer revenue

These metrics can be used for customer segmentation and behavioral analytics.

---

## Medallion Architecture

```text
BRONZE
Raw API Data
     │
     ▼
SILVER
Cleaned + Standardized + Validated Data
     │
     ▼
GOLD
Dimensions + Facts + Analytical Data Marts
```

This architecture separates raw ingestion from transformation and business-level modelling, making the pipeline easier to maintain, scale, and extend.

## Key Data Engineering Concepts Demonstrated

* End-to-end cloud ETL pipeline
* API data ingestion
* Azure Data Factory orchestration
* Azure Data Lake Storage Gen2
* Azure Databricks
* Apache Spark / PySpark transformations
* Unity Catalog governed data access
* Medallion Architecture
* Bronze, Silver and Gold data layers
* Data cleansing and standardization
* Data quality and referential-integrity checks
* Parquet storage
* Dimensional modelling
* Star schema design
* Fact and dimension tables
* Business metric calculation
* Analytical data marts
* Customer RFM analysis

## Repository Notebooks

### `data_ingestion.ipynb`

Loads the Bronze datasets from the Unity Catalog-accessible storage layer into Spark DataFrames for processing.

### `dataview_and_transform.ipynb`

Performs the main PySpark processing workflow, including:

**Bronze → Silver transformations → Data Quality Checks → Gold Star Schema → Analytical Data Marts**

---

## Project Goal

This project was built as a practical demonstration of designing a modern **Azure lakehouse data pipeline**, covering ingestion, cloud storage, distributed data processing, data governance, dimensional modelling, and preparation of analytics-ready datasets.
