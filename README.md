# EdTech Azure Data Factory Pipeline

Welcome to the EdTech Azure Data Factory Pipeline project! This advanced system processes and analyzes educational data to provide insights into student performance, content effectiveness, and learning patterns using Azure Data Factory and related Azure services.

## Table of Contents
- [Project Overview](#project-overview)
- [Azure Architecture](#azure-architecture)
- [Project Structure](#project-structure)
- [Setup and Configuration](#setup-and-configuration)
- [Usage](#usage)
- [Example: Student Performance Analysis](#example-student-performance-analysis)
- [Example: Content Effectiveness Evaluation](#example-content-effectiveness-evaluation)
- [CI/CD with Azure DevOps](#cicd-with-azure-devops)
- [License](#license)

## Project Overview

Our EdTech Azure Data Factory Pipeline is designed to handle large-scale data processing for educational platforms. It includes data ingestion from various sources, processing, analysis, and visualization components to enhance learning experiences and provide valuable insights for educators and administrators.

Key features:
- Integration with Learning Management Systems (LMS) and Student Information Systems (SIS)
- Real-time student activity tracking and processing
- Scalable data processing using Azure Data Factory and Azure Databricks
- Machine learning models for personalized learning path recommendations
- Student performance analytics and early intervention systems
- Content effectiveness analysis and improvement suggestions
- Integration with Azure Cognitive Services for natural language processing of student feedback

## Azure Architecture

Our pipeline utilizes the following Azure services:

- Azure Data Factory: Orchestrates and automates the data movement and transformation
- Azure Blob Storage: Stores raw and processed data
- Azure Databricks: Performs complex data processing and runs machine learning models
- Azure SQL Database: Stores structured data and analysis results
- Azure Analysis Services: Creates semantic models for reporting
- Power BI: Provides interactive dashboards and reports
- Azure Key Vault: Securely stores secrets and access keys
- Azure Monitor: Monitors pipeline performance and health

## Project Structure

```
edtech-azure-pipeline/
│
├── adf/
│   ├── pipeline/
│   │   ├── ingest_lms_data.json
│   │   ├── process_student_performance.json
│   │   └── analyze_content_effectiveness.json
│   ├── dataset/
│   │   ├── lms_data.json
│   │   ├── sis_data.json
│   │   └── processed_data.json
│   └── linkedService/
│       ├── AzureBlobStorage.json
│       ├── AzureDataLakeStorage.json
│       └── AzureDatabricks.json
│
├── databricks/
│   ├── notebooks/
│   │   ├── student_performance_analysis.py
│   │   ├── content_effectiveness_evaluation.py
│   │   └── learning_path_recommendation.py
│   └── libraries/
│       └── education_utils.py
│
├── sql/
│   ├── schema/
│   │   ├── student_performance.sql
│   │   └── content_metrics.sql
│   └── stored_procedures/
│       ├── calculate_student_progress.sql
│       └── evaluate_content_engagement.sql
│
├── power_bi/
│   ├── StudentPerformanceDashboard.pbix
│   └── ContentEffectivenessReport.pbix
│
├── tests/
│   ├── unit/
│   └── integration/
│
├── scripts/
│   ├── setup_azure_resources.sh
│   └── deploy_adf_pipelines.sh
│
├── .azure-pipelines/
│   ├── ci-pipeline.yml
│   └── cd-pipeline.yml
│
├── requirements.txt
├── .gitignore
└── README.md
```

## Setup and Configuration

1. Clone the repository:
   ```
   git clone https://github.com/your-org/edtech-azure-pipeline.git
   cd edtech-azure-pipeline
   ```

2. Set up Azure resources:
   ```
   ./scripts/setup_azure_resources.sh
   ```

3. Configure Azure Data Factory pipelines:
   ```
   ./scripts/deploy_adf_pipelines.sh
   ```

4. Set up Azure Databricks workspace and upload notebooks from the `databricks/notebooks/` directory.

5. Create Azure SQL Database schema and stored procedures using scripts in the `sql/` directory.

6. Import Power BI reports from the `power_bi/` directory and configure data sources.

## Usage

1. Monitor and manage Azure Data Factory pipelines through the Azure portal or using Azure Data Factory SDK.

2. Schedule pipeline runs or trigger them manually based on your requirements.

3. Access Databricks notebooks for custom analysis and model training.

4. View reports and dashboards in Power BI for insights into student performance and content effectiveness.

## Example: Student Performance Analysis

Here's an example of how to use Azure Databricks to analyze student performance:

```python
# In Azure Databricks notebook

from pyspark.sql import SparkSession
from pyspark.sql.functions import avg, count

# Initialize Spark session
spark = SparkSession.builder.appName("StudentPerformanceAnalysis").getOrCreate()

# Read student performance data from Azure Data Lake
performance_data = spark.read.parquet("abfss://processed-data@yourdatalake.dfs.core.windows.net/student_performance/")

# Calculate average scores by subject
avg_scores = performance_data.groupBy("subject").agg(
    avg("score").alias("average_score"),
    count("student_id").alias("student_count")
)

# Identify subjects that need attention (average score < 70)
subjects_needing_attention = avg_scores.filter(avg_scores.average_score < 70)

# Display results
subjects_needing_attention.show()

# Write results back to Azure SQL Database
subjects_needing_attention.write \
    .format("jdbc") \
    .option("url", "jdbc:sqlserver://yourserver.database.windows.net:1433;database=yourdatabase") \
    .option("dbtable", "subjects_needing_attention") \
    .option("user", "yourusername") \
    .option("password", "yourpassword") \
    .mode("overwrite") \
    .save()
```

This example demonstrates how to:
1. Read processed student performance data from Azure Data Lake
2. Calculate average scores by subject
3. Identify subjects that need attention based on average scores
4. Write the results back to Azure SQL Database for reporting

## Example: Content Effectiveness Evaluation

Here's an example of how to evaluate content effectiveness using Azure Data Factory and Azure Databricks:

```python
# In Azure Databricks notebook

from pyspark.sql import SparkSession
from pyspark.sql.functions import col, datediff, avg

# Initialize Spark session
spark = SparkSession.builder.appName("ContentEffectivenessEvaluation").getOrCreate()

# Read content interaction data and assessment results
content_data = spark.read.parquet("abfss://processed-data@yourdatalake.dfs.core.windows.net/content_interactions/")
assessment_data = spark.read.parquet("abfss://processed-data@yourdatalake.dfs.core.windows.net/assessment_results/")

# Join content interaction data with assessment results
combined_data = content_data.join(assessment_data, "student_id")

# Calculate content effectiveness metrics
effectiveness_metrics = combined_data.groupBy("content_id").agg(
    avg("time_spent").alias("avg_time_spent"),
    avg("assessment_score").alias("avg_assessment_score"),
    avg(datediff(col("assessment_date"), col("interaction_date"))).alias("avg_days_to_assessment")
)

# Identify highly effective content (high assessment scores, reasonable time spent)
highly_effective_content = effectiveness_metrics.filter(
    (effectiveness_metrics.avg_assessment_score > 80) &
    (effectiveness_metrics.avg_time_spent < 60)  # Assuming time spent is in minutes
)

# Display results
highly_effective_content.show()

# Write results to Azure SQL Database
highly_effective_content.write \
    .format("jdbc") \
    .option("url", "jdbc:sqlserver://yourserver.database.windows.net:1433;database=yourdatabase") \
    .option("dbtable", "highly_effective_content") \
    .option("user", "yourusername") \
    .option("password", "yourpassword") \
    .mode("overwrite") \
    .save()
```

This example shows how to:
1. Read content interaction data and assessment results from Azure Data Lake
2. Join and analyze the data to calculate content effectiveness metrics
3. Identify highly effective content based on assessment scores and time spent
4. Write the results to Azure SQL Database for further analysis and reporting

## CI/CD with Azure DevOps

We use Azure DevOps for continuous integration and deployment. Our pipeline includes:

1. **Continuous Integration (CI)**
   - Triggered on every push and pull request to the `main` branch
   - Validates Azure Data Factory pipeline definitions
   - Runs unit tests for Databricks notebooks and custom modules
   - Lints SQL scripts and validates database objects

2. **Continuous Deployment (CD)**
   - Triggered on successful merges to the `main` branch
   - Deploys Azure Data Factory pipelines to a staging environment
   - Runs integration tests
   - Upon approval, deploys to the production environment

To view and modify these pipelines, check the `.azure-pipelines/` directory.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
