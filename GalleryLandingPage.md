#Building Predictive Pipelines Incorporating Azure Data Lake and Azure Machine Learning

##Summary

This is a tutorial of using Cortana Intelligent Suite to build predictive pipelines for telecommunication industries. Infrastructure cost is one of the major drivers of telecommunication profit and loss. Predictive analysis in telecommunication can help reduce infrastructure cost by forecasting call volume and data traffic. In addition, Predictive analysis can improve resource allocation to reduce maintenance effort and customer churns.   This tutorial demonstrates how to build an end-to-end solution to predict call failure on a phone switch by using Azure Data Lake with Azure Machine Learning. Click **Get Started** on the right to see why we choose Azure Data Lake.

##Description
The solution in this tutorial provides two data views to users: (1) a real-time view with the call failure number on a switch, being refreshed every one minute (2) a batch view with prediction of the next-half-hour call failure, being refreshed every fifteen minutes.

We use a data generator to simulate a switch. The data generator ingests call detail records to Azure Event Hub. Azure Stream Analytic will read data from Azure Event Hub and run a per-minute aggregation and send the result to Power BI for visualization. This completes the real-time view. Along this path, you will learn how to configure an Azure Stream Analytic job and also how to write Azure Stream Analytic SQL with tumbling window, together with how to use Power BI.

On the batch view path, Azure Stream Analytic job will read data from Azure Event Hub and store the data to Azure Data Lake Store. Azure Data Lake Analytic will run a U-SQL job to generate aggregated view and a copy-activity from Azure Data Factory will move the aggregated view data from Azure Data Lake to Azure SQL Data Warehouse. Orchestrated by Azure Data Factory, Azure Machine Learning reads data from Azure SQL Data Warehouse and send predictive results back to Azure SQL Data Warehouse. Power BI interacts with Azure SQL Data Warehouse to visualize prediction.  Along this path, you will learn how to write U-SQL query, creating tables in Azure SQL Data Warehouse, configuring linked services, datasets and pipelines in Azure Data Factory and how to user Power BI desktop to run queries on SQL Data Warehouse to generate advanced visualization.

This tutorial provides automated  components deployable through Azure Resource Manager (ARM) and will walk you through the manual steps as well. Click the green button **Get Started** on the right to take a look at the step-by-step tutorial:
•	*README.md* to guide you through.
•	*azuredeploy_part1.json* – The first ARM template – creation of Service Bus, Event Hub, Stream Analytics Jobs, Blob Storage, SQL Server, SQL Data Warehouse, Azure Data Lake Store, Azure Data Lake Analytics
•	*azuredeploy_part2.json* - The second ARM template - installs the Data Factory partially
•	*linkedservice, dataset, pipeline* – contains the parts to the Data Factory which need to be manually deployed
•	*DataGenerator* –contains the zip file that can be deployed as a WebApp to simulate a switch
•	*script* –contains the U-SQL job query
•	*PowerBI* – contains the Power BI template to extract data from SQL Data Warehouse
•	media - A folder containing images used by README.md
Click **Get Started** on the right to begin.
