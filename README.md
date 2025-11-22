#  Wind Power Analytics Pipeline
<div align="center">

[![Microsoft Fabric](https://img.shields.io/badge/Microsoft_Fabric-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)](https://www.microsoft.com/en-us/microsoft-fabric)
[![PySpark](https://img.shields.io/badge/PySpark-E25A1C?style=for-the-badge&logo=apache-spark&logoColor=white)](https://spark.apache.org/docs/latest/api/python/)
[![Delta Lake](https://img.shields.io/badge/Delta_Lake-003366?style=for-the-badge&logo=delta&logoColor=white)](https://delta.io/)
[![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)](https://powerbi.microsoft.com/)

**End-to-end data pipeline on Microsoft Fabric for wind power production analytics**

[Architecture](#-architecture)  [Features](#-key-features)  [Technologies](#-technologies)  [Installation](#-installation)  [Pipeline Stages](#-pipeline-stages)  [Data Model](#-data-model)

</div>

---

##  Overview

A production-ready data engineering project implementing a complete **Medallion Architecture** (Bronze/Silver/Gold) on Microsoft Fabric for analyzing wind turbine power generation data. The pipeline automates data ingestion from GitHub, applies multi-stage transformations using PySpark, and delivers a dimensional model optimized for analytics and Power BI reporting.

###  Key Features

- **Medallion Architecture**: Industry-standard 3-layer data architecture (Bronze -> Silver -> Gold)
- **Dimensional Modeling**: Star schema with fact and dimension tables for optimized analytics
- **Automated Orchestration**: Microsoft Fabric Data Pipeline for scheduled execution
- **Delta Lake Integration**: ACID transactions and time-travel capabilities
- **Power BI Ready**: Pre-modeled data optimized for reporting and dashboards
- **PySpark Processing**: Scalable distributed data transformations
- **GitHub Integration**: Automated daily data ingestion from remote repository

---

##  Architecture

```

  GitHub Source    (Daily Wind Power Data)

         
         

  BRONZE LAYER (Raw Lakehouse)           
   Raw CSV ingestion                    
   Schema enforcement                   
   Delta Lake format                    

         
         

  SILVER LAYER (Cleaned Lakehouse)       
   Data quality checks                  
   Type conversions                     
   Null handling                        
   Business logic application           

         
         

  GOLD LAYER (Analytics Lakehouse)       
   Star schema model                    
   Fact: Turbine Production             
   Dimensions: Turbine, Date, Location  
   Aggregated metrics                   

         
         

    Power BI       (Dashboards & Reports)

```

---

##  Technologies

| Category | Tools |
|----------|-------|
| **Platform** | Microsoft Fabric |
| **Storage** | Delta Lake, OneLake |
| **Processing** | PySpark, SQL |
| **Orchestration** | Fabric Data Pipeline |
| **Visualization** | Power BI, DAX |
| **Data Format** | Parquet, Delta |

---

##  Project Structure

```
fabric-wind-power-pipeline/
 README.md
 .gitignore
 notebooks/
    bronze/
       NB_Get_Daily_Data_Python.ipynb         # Ingest raw data from GitHub
    silver/
       NB_Bronze_To_Silver_Transformations_Python.ipynb  # Clean & transform
    gold/
        NB_Silver_To_Gold_Transformations_Python.ipynb   # Build star schema
 documentation/
    architecture_diagrams/
 screenshots/
     00_github_repo_structure.png
     01_workspace_with_three_lakehouses.png
     02_bronze_table_data.png
     03_silver_table_schema.png
     04_gold_lakehouse_tables.png
     04_fact_table_preview.png
     04_dim_turbine_preview.png
     05_pipeline_execution_success.png
```

---

##  Installation

### Prerequisites

- Microsoft Fabric workspace (F64 or higher)
- Power BI Premium capacity
- GitHub repository access (for data source)

### Setup Steps

1. **Create Fabric Workspace**
```ash
# In Microsoft Fabric portal
Create workspace  Enable Fabric features
```

2. **Create Three Lakehouses**
```
- Bronze_Lakehouse  (raw data)
- Silver_Lakehouse  (cleaned data)
- Gold_Lakehouse    (dimensional model)
```

3. **Import Notebooks**
```ash
# Upload notebooks from notebooks/ directory to respective lakehouse contexts
Bronze  NB_Get_Daily_Data_Python.ipynb
Silver  NB_Bronze_To_Silver_Transformations_Python.ipynb
Gold  NB_Silver_To_Gold_Transformations_Python.ipynb
```

4. **Configure Data Pipeline**
```
Create Pipeline  Add Notebook Activities  Link Bronze -> Silver -> Gold
Set Schedule  Enable monitoring
```

---

##  Pipeline Stages

### 1 Bronze Layer: Data Ingestion

**Notebook**: \NB_Get_Daily_Data_Python.ipynb\

- Fetches CSV files from GitHub repository
- Validates schema and data types
- Writes raw data to Delta tables
- Preserves original source structure

**Output**: \ronze.turbine_raw_data\

### 2 Silver Layer: Data Transformation

**Notebook**: \NB_Bronze_To_Silver_Transformations_Python.ipynb\

- Removes duplicates and null values
- Standardizes timestamp formats
- Applies data quality rules
- Enriches with calculated fields

**Output**: \silver.turbine_cleaned_data\

### 3 Gold Layer: Dimensional Modeling

**Notebook**: \NB_Silver_To_Gold_Transformations_Python.ipynb\

Creates star schema:

**Fact Table**:
- \act_turbine_production\ (power output, wind speed, temperature)

**Dimension Tables**:
- \dim_turbine\ (turbine metadata, capacity, location)
- \dim_date\ (date hierarchy for time intelligence)

**Output**: Power BI-ready dimensional model

---

##  Data Model

### Fact Table Schema

| Column | Type | Description |
|--------|------|-------------|
| production_id | BIGINT | Surrogate key |
| turbine_id | INT | FK to dim_turbine |
| date_id | INT | FK to dim_date |
| power_output_mw | DECIMAL | Generated power (MW) |
| wind_speed_ms | DECIMAL | Wind speed (m/s) |
| temperature_c | DECIMAL | Ambient temperature |

### Dimension Tables

<details>
<summary><b>dim_turbine</b> (Click to expand)</summary>

| Column | Type |
|--------|------|
| turbine_id | INT |
| turbine_name | STRING |
| capacity_mw | DECIMAL |
| location | STRING |
| installation_date | DATE |

</details>

<details>
<summary><b>dim_date</b> (Click to expand)</summary>

| Column | Type |
|--------|------|
| date_id | INT |
| full_date | DATE |
| year | INT |
| quarter | INT |
| month | INT |
| day | INT |

</details>

---

##  Use Cases

- **Production Monitoring**: Track real-time wind power generation
- **Performance Analysis**: Compare turbine efficiency across locations
- **Capacity Planning**: Forecast power output based on weather patterns
- **Maintenance Scheduling**: Identify underperforming turbines

---

##  Example Queries

```sql
-- Total power production by turbine (last 30 days)
SELECT 
    t.turbine_name,
    SUM(f.power_output_mw) as total_production_mw
FROM gold.fact_turbine_production f
JOIN gold.dim_turbine t ON f.turbine_id = t.turbine_id
JOIN gold.dim_date d ON f.date_id = d.date_id
WHERE d.full_date >= CURRENT_DATE - INTERVAL 30 DAYS
GROUP BY t.turbine_name
ORDER BY total_production_mw DESC;
```

---

##  Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (\git checkout -b feature/amazing-feature\)
3. Commit your changes (\git commit -m 'Add amazing feature'\)
4. Push to the branch (\git push origin feature/amazing-feature\)
5. Open a Pull Request

---

##  License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

##  Acknowledgments

- Microsoft Fabric documentation and community
- Apache Spark team for PySpark
- Delta Lake project for table format innovations

---

<div align="center">

**[ Back to Top](#-wind-power-analytics-pipeline)**

Made with  by [Florian Abgrall](https://github.com/Flockyy)

</div>
