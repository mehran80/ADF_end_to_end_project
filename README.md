# ✈️ End-to-End Airline Data Engineering Pipeline (Azure & ADF)

An enterprise-grade, end-to-end data engineering pipeline built using **Azure Data Factory (ADF)** to ingest, clean, and model airline booking and flight data. This project follows the **Medallion Architecture (Bronze -> Silver -> Gold)** and is managed professionally using **CI/CD** through **Azure DevOps**.

---

## 🏗️ Architecture Diagram
## 🏗️ Technical Architecture Diagram

```mermaid
graph TD
    %% Define Sources
    subgraph On-Premises
        A[Local Computer Files CSVs]
    end
    
    subgraph Cloud Storage & Databases
        C[Azure SQL Database Source Tables]
    end

    %% Define Orchestration, Connectivity, & Version Control
    subgraph CI/CD & DevOps
        B[Azure DevOps Git Repository]
    end
    
    SHIR[Self-Hosted Integration Runtime]
    ADF[Azure Data Factory ADF]

    %% Define Medallion Layers
    subgraph ADLS Gen2 Data Lake & Database
        Bronze[(Bronze Layer: Raw Files)]
        Silver[(Silver Layer: Cleaned Tables)]
        Gold[(Gold Layer: Star Schema & Data Marts)]
    end

    %% Define Business Intelligence
    PBI[Power BI Dashboards]

    %% Define Connections / Flow
    A -->|Secure Outbound| SHIR
    SHIR -->|Ingest Raw CSVs| ADF
    C -->|Extract Cloud Tables| ADF
    B <-->|CI/CD & Version Control| ADF
    
    ADF -->|Load Raw| Bronze
    
    %% Silver Transition
    Bronze -->|Data Flow: Convert 5 Date Columns via Column Patterns| ADF
    ADF -->|Save Cleaned Data| Silver
    
    %% Gold Transition
    Silver -->|Data Flow: Alter Row - Upsert If - true| ADF
    
    %% Gold Logic / Aggregations
    Gold -->|Gold Logic: SQL GROUP BY / DENSE_RANK| Gold

    %% Styling
    style SHIR fill:#f9f,stroke:#333,stroke-width:2px
    style ADF fill:#bbf,stroke:#333,stroke-width:2px
    style Gold fill:#bfb,stroke:#333,stroke-width:2px
    style B fill:#ddd,stroke:#333,stroke-width:1px

---

## 🛠️ Tech Stack & Azure Services
*   **Orchestration:** Azure Data Factory (ADF)
*   **Compute & Transformation:** ADF Mapping Data Flows (Column Patterns, Alter Row)
*   **Storage (Data Lake):** Azure Data Lake Storage Gen2 (ADLS Gen2)
*   **Database (Warehouse):** Azure SQL Database
*   **Connectivity:** Self-Hosted Integration Runtime (SHIR) for secure On-Premises connection
*   **CI/CD & Version Control:** Azure DevOps (Git Branching, Pull Requests)

---

## 📂 Project Structure (Medallion Architecture)

### 1. Bronze Layer (Raw Ingestion)
*   Ingested raw, unstructured airline transaction CSV files from an **on-premises** directory using a **Self-Hosted Integration Runtime**.
*   Implemented dynamic dataset parameters for modular file loading, reducing pipeline complexity from 5 separate connections to a single, parameterized Linked Service.

### 2. Silver Layer (Data Cleansing)
*   Designed **ADF Data Flows** using **Column Patterns** to dynamically identify and convert 5 separate date-time columns using the expression:
    `toTimestamp(toString($$), 'yyyy-MM-dd HH:mm:ss')`
*   Handled data quality anomalies by converting empty or "Unknown" string records into standardized `NaT` (Null) objects without breaking the pipeline.

### 3. 📈 Advanced Business Intelligence & Gold Layer Logics

In the **Gold Layer**, I designed high-performance SQL Aggregations (Data Marts) to pre-calculate key business and operational metrics for the Airline Executive team:

1. **Top Routes by Revenue (`gold.route_revenue_performance`):**
   * Groups transactional bookings by `origin` and `destination` airports to calculate total tickets sold and total revenue. This helps route planners optimize flight schedules for high-value corridors.

2. **Airline Market Share Summary (`gold.airline_performance_summary`):**
   * Aggregates total passenger trips and total revenue per airline, allowing leadership to analyze market share performance and average ticket prices across competitors.

3. **Top Flights Ranked by Airline (`gold.top_flights_ranked`):**
   * Implemented advanced SQL Window Functions (`DENSE_RANK() OVER (PARTITION BY ... ORDER BY ...)`) to rank individual flights within each airline based on revenue generation. This helps identify the most profitable specific flights across the entire network.


---

## 🚀 DevOps & CI/CD Practices
*   Followed a professional Git workflow using **Azure DevOps**.
*   Implemented strict branching strategies: developed features inside isolated feature branches (`feature-fix-dates`), utilized **Pull Requests (PR)** for peer reviews, and deployed changes to production via automated release pipelines.