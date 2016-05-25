#Building Predictive Pipelines Incorporating Azure Data Lake and Azure Machine Learning

Big data is not a buzz word, but a reality. Yet another challenge is surfacing up: the number of the types of data is growing and new type of data are coming up. The data format is not just limited to text, audio, video, but also sensor data, clickstream, server logs  and many others. To deal with these new types of data, new applications need to be developed and still take their time to mature. Do you want to transform your native data to a defined structure where you don't even know what your business requirements are and throw the native data away before your applications find their ways to develop insights? The answer is of course not. And data lake comes into rescue.  

There is a trend  in industry  to extract and place data for analytics into a data lake repository without first transforming the data the way they would need to for a relational data warehouse or key-value store.
By storing data in its native format, data lake maintains data provenance and no loss of information content  resulting from the extraction, transformation and loading (ETL) process.  By shifting the process from ETL to ELT (extraction, loading and transformation), data lake uses a schema-on-read approaches, therefore eliminates the work of defining schemas before business requirements are clear and also saves greatly on computation, which is more expensive than the storage. As data volumes, data variety, and metadata richness grow, the benefit of the new approach magnifies.

Figure 1 and figure 2 below illustrates the shift from ETL to ELT.

<br/>
![](media/non_ADLS_analysis.png)
<br/>
Figure 1: Traditional Data Management and Analysis
<br/>
![](media/ADLS_analysis.png)
<br/>
Figure 2: Data Management and Analysis in an  Data Lake Environment.


<a href="https://azure.microsoft.com/en-us/solutions/data-lake/"/>Azure Data Lake</a>  is a Microsoft Data Lake product. It consists of <a href="https://azure.microsoft.com/en-us/documentation/services/data-lake-store/"/> Azure Data Lake Store</a> and <a href="https://azure.microsoft.com/en-us/documentation/services/data-lake-analytics/"> Azure Data Lake Analytics </a>. Azure Data Lake Store  is a data repository capable of holding an unlimited amount of data in its native, raw format, including structured, semi-structured, and unstructured data. With Azure Data Lake Analytics, you can define your schema on read by running U-SQL and implementing advanced data extraction and transformation by using C# code, which is extremely flexible. By using Azure Data Lake Analytics, the capability of running analytic on your data extends without a boundary.

In addition to Azure Data Lake, Microsoft Cortana Intelligence Suite provides many other products, especially Azure Machine Learning to allow you to build an end-to-end advanced analytic solution on your data lake data and make Intelligent actions. Azure Machine learning provides a fully managed cloud service to build, deploy and share predictive analysis. <a href="https://azure.microsoft.com/en-us/services/event-hubs/"/>Azure Event Hub</a> and <a href="https://azure.microsoft.com/en-us/services/stream-analytics/"/> Azure Stream Analytics</a> provides highly scalable data ingestion and event processing service. Azure SQL Data Warehouse provides high performing query on your structured data and Power BI renders visualization on your streaming data and data in your warehouse. Azure Data Factory schedules data transformation and data movement among all the Azure services that your solution needs to use.