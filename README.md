# 🚲 Hubway Bikes Data ETL Pipeline (PySpark & Neon Postgres)

![Apache Spark](https://img.shields.io/badge/Apache%20Spark-E25A1C?style=for-the-badge&logo=apachespark&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/Neon%20Postgres-316192?style=for-the-badge&logo=postgresql&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter%20/%20Colab-F37626?style=for-the-badge&logo=jupyter&logoColor=white)

An end-to-end **Extract, Transform, and Load (ETL)** data engineering pipeline built with **Apache PySpark** to process, enrich, and aggregate Hubway bike-sharing trip data, loading the final analytical dataset into a cloud-hosted **Neon Serverless PostgreSQL** database via JDBC.

---

## 🏗️ Pipeline Architecture & Workflow

```text
[ CSV Datasets ] ──▶ [ PySpark Ingestion ] ──▶ [ Joins & Cleaning ] ──▶ [ GroupBy Metrics ] ──▶ [ Neon Postgres ]
 (Trips, Stations,     (Schema Inference &      (Landmarks, State &     (Distinct Trips &      (JDBC SSL Sink)
   Zip Code Map)        Header Cleanup)          Month_Year)             Total Minutes)
```

### 📥 1. Data Ingestion (Extract)
* 🚴 **Trips Dataset:** Loaded bike trip records containing trip duration, start/end timestamps, terminal IDs, subscriber types, and customer zip codes.
* 📍 **Stations Dataset:** Loaded station metadata containing station IDs, names, geographic coordinates, dock counts, and municipality landmarks.
* 🗺️ **Zip Code Mapping Dataset:** Loaded US geographic zip code mappings to resolve customer zip codes into state abbreviations.
* 🧹 **Header Standardization:** Normalized column headers across DataFrames by replacing spaces with underscores (`_`) for Spark SQL compatibility.

### ⚙️ 2. Data Transformation (Transform)
* 🔗 **Multi-Join Enrichment:** Joined the Trips DataFrame with the Stations DataFrame twice (`Start Terminal` and `End Terminal`) to append `Starting_Landmark` and `Ending_Landmark` columns.
* 🏛️ **State Mapping & Null Handling:** Joined customer zip codes with the Zip Code Mapping dataset to extract customer `State`, assigning `"Unknown"` to missing or unmatched records.
* 📅 **Temporal Feature Extraction:** Parsed trip `Start Date` timestamps and derived a formatted `Month_Year` (`yyyy-MM`) column.
* ✂️ **Schema Pruning:** Retained only essential analytical columns (`Trip_ID`, `Duration`, `Starting_Landmark`, `Ending_Landmark`, `State`, `Subscriber_Type`, `Month_Year`).

### 📊 3. Data Aggregation
* 🗂️ **Grouping Dimensions:** Grouped transformed records by `Starting_Landmark`, `Ending_Landmark`, `State`, `Subscriber_Type`, and `Month_Year`.
* 🧮 **Calculated Metrics:**
  * `Number_of_Trips`: Count of distinct `Trip_ID`s.
  * `Duration_Total_Minutes`: Total trip duration converted from seconds to minutes (`sum(Duration) / 60`).

### 🗄️ 4. Data Storage (Load)
* 🔌 **JDBC Configuration:** Configured the PostgreSQL JDBC driver (`org.postgresql:postgresql:42.7.13`) within the Spark session.
* ☁️ **Cloud Database Sink:** Exported the aggregated Spark DataFrame into a **Neon Postgres** cloud database (`Trips_transformed_alishba`) using SSL-secured JDBC properties and verified the records in the Neon SQL Table viewer.

---

## 🛠️ Tech Stack
* ⚡ **Processing Engine:** Apache Spark / PySpark (`pyspark.sql`)
* 🐘 **Database:** Neon Serverless PostgreSQL (JDBC Sink)
* 📓 **Environment:** Google Colab / Jupyter Notebook

---

## 📁 Repository Contents
* 📘 `Hubway_ETL_Alishba_Naeem.ipynb` — Complete executed PySpark ETL notebook with visible cell outputs.
* 🖼️ `Neon postgres table data screenshot.png` — Verification screenshot of the populated table in Neon Postgres.
* 📄 `README.md` — Project overview and ETL pipeline documentation.

---

## 🚀 How to Run
1. Open `Hubway_ETL_Alishba_Naeem.ipynb` in **Google Colab** or a local **Jupyter** environment with PySpark installed.
2. Replace the Neon Postgres connection parameters (`PGHOST`, `PGDATABASE`, `PGUSER`, `PGPASSWORD`) with your own Neon credentials.
3. Run all cells sequentially to execute the ETL pipeline and write the aggregated results to PostgreSQL.