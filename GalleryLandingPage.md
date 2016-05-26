#Building Predictive Pipelines Incorporating Azure Data Lake and Azure Machine Learning

##Summary

This is a tutorial about building predictive pipelines for telecommunication using Cortana Analytics Suite. Infrastructure cost  has a major impact on the revenue of a telecommunication enterprise. Predictive analytics in telecommunication can help reduce infrastructure cost by forecasting call volume , data traffic, service quality issues, infrastructure failure, etc. In addition, Predictive analytics can improve resource allocation to improve network reliability and quality of service and as a result reduce customer churn. This tutorial demonstrates how to build an end-to-end solution pipeline to predict an impending large network component failure by using call drop data . The solution is implemented using Azure Data Lake together with Azure Machine Learning. Click **Get Started** on the right to see why we choose Azure Data Lake.

##Description
The solution in this tutorial provides kinds of dashboards to the users: (1) real-time insights about the count of calls failing on a switch, refreshed every one minute (2) a batch analytics  that predicts the health of the switch for next half hour into the future, refreshed every fifteen minutes.

We use a data generator to simulate  the Call Detail Records (CDR) coming from a telephony switch. The data generator ingests the CDRs to Azure Event Hub. Azure Stream Analytic reads data from Azure Event Hub and run a per-minute aggregation job and send the result to the Power BI for visualization. This constitutes the real-time analytics pipeline. Along the way, you will learn how to configure an Azure Stream Analytic job, how to write Azure Stream Analytic SQL with tumbling window, and use Power BI to show real time data updates.

In the batch analytics pipeline, an Azure Stream Analytic job reads data from Azure Event Hub and store all the data to Azure Data Lake Store. Azure Data Lake Analytic will run a U-SQL job to generate time-based aggregates and a copy activity from Azure Data Factory will move the aggregated data from Azure Data Lake to Azure SQL Data Warehouse. Orchestrated by Azure Data Factory, Azure Machine Learning reads the data from Azure SQL Data Warehouse and sends the predictive results back to the Azure SQL Data Warehouse. Power BI interacts with the Azure SQL Data Warehouse to visualize prediction. Along the way, you will learn how to write U-SQL query, create tables in Azure SQL Data Warehouse, configure linked services, datasets and pipelines in Azure Data Factory and how to use Power BI desktop to run queries on data in SQL Data Warehouse to generate advanced visualization.

This tutorial provides automated components deployable through Azure Resource Manager (ARM) and will walk you through the manual steps as well. Clicking the green button  **Get Started** on the right will take you to a GitHub repository that includes the following:

- **README.md** - a complete step by step instructions manual to guide you through the deployment of the solution.
- **azuredeploy_part1.json** - Part one of the ARM template – that creates Service Bus, Event Hub, Stream Analytics Jobs, Blob Storage, SQL Server, SQL Data Warehouse, Azure Data Lake Store and, Azure Data Lake Analytics.
-	**azuredeploy_part2.json** - Part two of the ARM template – that installs parts the Data Factory
-	**linkedservice, dataset, pipeline** – contains the parts to the Data Factory which need to be manually deployed
-	**datagenerator.zip** – contains the zip file that can be deployed as a WebApp to simulate a phone switch
-	**script** –contains the U-SQL job query
-	**PowerBI** – contains the Power BI template to extract data from SQL Data Warehouse
-	**media** - A folder containing images used by README.md

Click **Get Started** on the right to begin.
