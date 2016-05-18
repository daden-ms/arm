Summary
=======

An Azure Data Lake Store is a flexible, scalable repository for any type of data. It provides unlimited storage with high frequency, low latency throughput capabilities and provides immediate read and analysis capabilities over your data. Once data is captured in the Data Lake, advanced transformation and processing of the data can be performed using Microsoft's extendable and scalable U-SQL language, integrated with Azure Data Lake Analytics, Azure Machine Learning, or any HDFS compliant project, such as Hive running in HD Insight cluster.

Some of the principal benefits of an Azure Data Lake Store include:

-   Unlimited storage space

-   High-throughput read/write

-   Security through integration with Active Directory

-   Automatic data replication

-   Compatibility with the Hadoop Distributed File System (HDFS).

-   Compatibility with  HDFS compliant project (e.g. Hive, HBase, Storm,  etc.)

The objective of this tutorial is to demonstrate techniques for the movement of data between an external data source, an Azure Data Lake Store and Azure SQL Data Warehouse while demonstrating using U-SQL for processing information in a Data Lake Store and perform advanced analytic through Azure Machine Learning  (AML).

This tutorial will be developed in reference to a use case described in the following section.

Use Case
========

Switch based telephone companies, both land line and cellular, produce very large volumes of information, principally in the form of call detail records. Each telecom switch records information on the calling and called numbers, incoming and outgoing trunks, and information of the time of the call along with a number of other features.

The duration that telephone companies keep their data has varied between land line and cellular companies from one to many years. Various legislation (e.g. the USA Freedom Act) is being considered that will require telecommunication companies to hold the data for a longer period of time. The amount of data can be extremely large. If you consider a modestly sized telecom carrier with 10M customers can readily produce over 1 billion description messages per day, including call detail records (CDR) at a size close to 1 TB per day. In shortly over ½ of a year this amount of data could begin to surpass the maximum capacity of an Azure Storage blob per storage account per subscription (500TB).

In scenarios such as this, the integrated SQL and C# capabilities of U-SQL, the unlimited data storage capacity, and the ability for high velocity data ingestion of the Azure Data Lake Store makes it an ideal technical solution for the persistence and management of telephony call data.

Telecommunication network optimization techniques can hugely benefit from getting switch overload or malfunction predictions ahead of time. Such predictions help maintain SLA and overall network health by allowing for mitigating actions to be taken proactively, such as, possibly rerouting calls and avoid call drops and perhaps an eventual switch shutdown. Microsoft’s capability to manage unlimited volumes of data within an Azure Data Lake Store combined with the powerful means for interacting with the Data Lake Store through U-SQL and the predictive modeling capabilities of Azure Machine Learning (AML) readily address all of the challenges with storage compliance and provide a seamless means for impactful analysis suitable for network optimization and other interaction..


The intent of this tutorial is to provide the engineering steps necessary to capture and reproduce completely  the scenario described above.

The tutorial will include:

- The generation and ingestion of CDR Data using an Azure Event Hub and Azure Streaming Analytics.
- The creation of an Azure Data Lake Store (ADLS) to meet long term CDR management requirements.
- Using Azure Data Lake Analytics (ADLA) and Microsoft’s U-SQL to interact with the Data Lake. The ADLA U-SQL job generate aggregate view over the ingested CDR data that stored in ADLS.
- Creation and integration of staging store for storing analytics results from U-SQL and predictions from Azure Machine Learning (AML). This staging store is implemented using Azure SQL Data Warehouse (SQL DW) and provides as a backend for Power BI dashboards.
- AML model which predicts the switch overload


The focus of this tutorial is on the architecture, data transformation, and the movement of data between the different storage architectures and the Azure Machine Learning (AML) environment. While this example demonstrates techniques for integrating AML into the solution architecture, the focus is not on machine learning. The machine learning model is used in this tutorial to predict switch overload with time series analysis by using random forest method.  Machine learning can be used in telecommunication industry for effective marketing campaign, reducing infrastructure cost and maintenance effort.      

Prerequisites
=============

The steps described later in this tutorial  requires the
following prerequisites:

1)  Azure subscription with login credentials
    (https://azure.microsoft.com/en-us/)

2)  Azure Machine learning Studio subscription
    (https://azure.microsoft.com/en-us/services/machine-learning/)

3)  a Microsoft Power BI account
    (https://powerbi.microsoft.com/en-us/)

4)  Azure Data Lake Store account
    (Access is requested through the Azure portal)

5)  Azure Data Analytics account.
    (Access is requested through the Azure portal)




Architecture
============

Figure 1 illustrates the Azure architecture developed in this sample.

![](media/architecture.png)
Figure 1: Architecture

Call detail record (CDR) data is generated via a data generator which simulates a phone switch and is deployed as an Azure Web Job. The CDR data is sent to an Event Hub. Azure Stream Analytics (ASA) takes in the CDR data flowed through Event hub, processes the data by using ASA SQL and sends the processed data to a) Power BI for real time visualization and b) Azure Data Lake Store for storage. Azure Data Lake Analytics runs a U-SQL job to pre-process the data before sending it to SQL Data Warehouse for Azure Machine Learning to run predictive analytics.

Predictive analytics is done by using the batch endpoint of an experiment published as a web service in the Azure Machine Learning Studio. The AML web service imports call failure number per minute from SQL Data Warehouse and exports the prediction, e.g. the scoring results back to SQL Data Warehouse. We use Azure Data Factory to orchestrates 1) U-SQL job in Azure Data Lake 2) Copy the results of the U-SQL job to SQL Data Warehouse 3) Predictive analytics in AML. The machine learning model here is used as an example experiment. You can use field knowledge and combine the available data dataset to build more advanced model to meet your business requirements.





Deploy
=====================

Below are the steps to deploy the use case into your Azure subscription. Note that to condense the steps somewhat, **>** is used between repeated actions. For example:

1. Click: **Button A**
1. Click: **Button B**

is written as

1. Click: **Button A** > **Button B**  



Step 1) Deployment of Multiple Resources, including:
-----------------------------------------
1. Service Bus,
2. Event Hub,
3. Stream Analytics Job
4. SQL Server, SQL Data Warehouse,
5. Azure Data Lake Store Account
6. Azure Data Lake Analytics Account

To get started, click the below button.

<a target="_blank" id="deploy-to-azure" href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fdaden-ms%2Farm%2Fmaster%2Fazuredeploy_part1.json"><img src="http://azuredeploy.net/deploybutton.png"/></a>

This will create a new "blade" in the Azure portal.

![arm1-image](./media/arm1.png)

1. Parameters
   1. Type: UNIQUE (string): **[*UNIQUE*]** (You need to select a globally unique string)
   1. Select: LOCATION: **[*LOCATION*]** (The region where everything will be deployed)
   1. Click: **OK**
1. Select: Subscription: **[*SUBSCRIPTION*]** (The Azure subscription you want to use)
1. Resource group
   1. Select: **New**
   1. Type: New resource group name: **[*UNIQUE*]** (Same as above)
1. Select: Resource group location: **[*LOCATION*]** (Same as above)
1. Click: **Review legal terms** > **Create**
1. Check: **Pin to dashboard** (If you want it on your dashboard)
1. Click: **Create**




The resource group will serve as an organizational framework for the
associated Azure services.

After logging into to the Azure Portal, select the “Resource Groups”
option from the menu. This will open a frame where you can select the
option to create a new resource group. You can name it what you wish,
but for the sake of this sample we will call it the “cdrresourcegroup.”

Step 2) Create an Azure Storage Account.
-----------------------------------------

The Azure Storage Account will be used to manage different storage blobs
that will be associated with this sample. The Azure Storage Account can
be created through the Portal.

![](media/storageAccount.png)

Record the keys and associated connection strings for the storage
account. This will be needed by other components wishing to store or
access information managed under this account.

Under the storage account, create a blob storage container named
“cdrdata.”

Ultimately this container will contain four folders: input, aggregate,
processed, and scripts.

The ‘input’ folder will serve as a staging area for incoming call detail
records (CDR) prior to moving them to the Data Lake Store. The
“aggregate” folder is the location for summary data produced by a USQL
script that processes the collection of CDR data residing in the Azure
Data Lake Store. This is the information that will be processed within
the Azure Machine Learning (AML) environment. The “processed” folder is
the location for the data produced by the AML model. The “scripts”
folder is the repository for the USQL scripts that will be run under the
Azure Data Lakes Analytics account and produce the summary CDR data.

Step 3) Create an Azure Data Lake Store.
----------------------------------------

The Data Lake Store (ADLS) will serve as the repository of all CDR data.
After creating the Data Lake Store a folder named cdrdata will be
created that will serve as the location for all the CDR data. The CDR
data will be transferred using an Azure Data Factory Pipeline from the
staging area in the blob storage (cdrdata/input).

Record the URL for the Data Lake Store. This will be needed by other
components wishing to store or access information managed under this
account.

Click on the “Data Explorer” menu item within the Data Lake Store just
created. Create a new folder named “cdrdata.” Within the cdrdata folder
you will need to create 2 more folders named:

(1) input: This will be the main CDR repository.

(2) aggregate: This folder will contain the results from USQL scripts
    that will periodically process the collected CDR data to produce
    summary statistics.

Step 3) Create an Azure Data Analytics Account
----------------------------------------------

The ADLA account will serve as the context and provide the credentials
for scheduling and running the USQL query over the ADLS CDR repository.

Step 4) Create an Azure Event Hub.
-----------------------------------------

Creating an Azure Event Hub is done in the original version of the Azure
Portal, not the new version. As shown in figure 2 below. The event hub
will be used to receive incoming CDRs from the simulated telco switches.

![](media/eventHub.png)

Figure 2: Creating the Azure Event Hub

### Creating the Event Hub Rules

You will next need to provision rights for sending and receiving
messages through the event hub. To do this you click on the “configure”
icon/label for the hub you just created. (see figure 3)

![](media/eventHubRules.png)

Figure 3: Configure the Event Hub


After creating the Event Hub rule, record the Policy Name, Azure
generated keys, and endpoints. This will be needed to send and retrieve
information from the Hub. Sample name and keys are provided below,
however your keys will be different.

    Policy Name: SendRule
    Policy Key: [send rule key>]
    Endpoint= [send endpoint]

    Policy Name: ReceiveRule
    Policy Key: [receive rule key]
    Endpoint= [receive endpoint]

Step 5) Create an Azure Stream Analytics Service.
-------------------------------------------------

Creating a Streaming Analytics Service is done through the new version
of the Portal.

![](media/streamingAnalytics.png)

Figure 4: Provisioning Azure Streaming Analytics (ASA)

The Streaming Analytics Service is responsible for receiving
information sent to the Event Hub that was just created, performing some
filter of that data to selecting a subset of the available CDR fields for storage, and persisting the CDR data in the Azure Storage Blob staging area.

![](media/asaInput.png)

Figure 5: Provisioning the ASA Input.

When the Streaming Analytics Service is initially created it will not
have any inputs or outputs provisioned. After creating the service, you
will be presented with a screen where you have the opportunity to set
the input to the event hub you previously created. By selecting the
“Inputs” section of the newly created service you will be provided with
a screen where you can enter the appropriate information for the event
hub.

![ASA Output](media/asaOutput.png)

Figure 6: Provisioning the ASA Output.

Change the “Event Hub Serialization Format”
field from JSON to CSV. Also insure that you enter the “Send Rule”
information that you previously captured.

You will next connect the Output of the ASA to the storage account
created in step 2 of this sample. Click on the “Outputs” section of the
ASA and add a new output. You will need to enter the storage account and
the storage account key for the azure blob storage. Insure that you set
the serialization format to comma separated CSV with UTF-8 encoding.

Set the Path Pattern for the stored information to “input/{date}/{time}”
Doing this will result in the ASA creating hourly CDR files in the
appropriate date/time subdirectories within the blob storage.

The final step in provisioning the ASA service is entering the query
that will select the appropriate fields from the CDR for storage in the
Azure Storage Blob staging area and the Azure data lake Store. Use the
following query for the ASA service:

    SELECT
      CallRecTime, SwitchNum, CallingIMSI, CallingNum, CalledIMSI, CalledNum,
      IncomingTrunk , OutgoingTrunk
    FROM
      cdrhubInput

Step 7) Download the CDR Data Generator
---------------------------------------

Unless you are in the telecommunications industry you presumably do not
have streaming CDR data available to you. Use the link below to download
a software program that will act as a “virtual telecom switch” and
generate a stream of CDR records. You will need to edit the app.config
file to provide the name of the event hub you created and the connection
string for the “SendRule.” If you are connected to the internet you can
run this program and begin to push generated CDRs through the event hub
and streaming analytics service to the Azure Storage Blob staging area.

[TelcoGenerator.zip](http://download.microsoft.com/download/8/B/D/8BD50991-8D54-4F59-AB83-3354B69C8A7E/TelcoGenerator.zip "TelcoGenerator.zip")

Step 8) Publishing the USQL script
----------------------------------

On an hourly basis a scheduled Azure Data Factory pipeline will send
initiate a USQL analytical job over the cdr data stored in the Azure
Data Lake cdrdata/input collection. USQL, a hybrid of standard SQL and
C\# functions and assemblies can be used to efficiently perform a number
of operations over this data. For this sample the USQL will generate
hourly CDR aggregate statistics of the number of calls by switch and
trunk. This data will be persisted to the /cdr/aggregate/ folder in the
Data Lake Storage repository where it will be transmitted to the storage
blob for processing through the Azure Machine Learning model.

You can learn more about U-SQL and Azure Data Lake Analytics at the
following:

[Overview of Microsoft Azure Data Lake Analytics](https://azure.microsoft.com/en-us/documentation/articles/data-lake-analytics-overview/)

Create a file named “cdrsummary.txt” and enter the following USQL text.

    @cdrdata =
     EXTRACT CallRecTime DateTime,
       Switch string,
       CallingIMSI string,
       CallingNum string,
       CalledIMSI string,
       CalledNum string,
       IncomingTrunk string,
       OutgoingTrunk string
     FROM "/cdrdata/input/{*}"
     USING Extractors.Csv();

    @rs1 =
     SELECT
       DateTime.Now AS ProcessingDate,
       Switch,
       OutgoingTrunk,
       COUNT(*) AS CallCount
    FROM
      @cdrdata
    WHERE
      CallRecTime &gt; DateTime.Now.AddHours(-1)
    GROUP
      BY Switch,OutgoingTrunk;

    OUTPUT @rs1
     TO "/cdrdata/aggregate/hourSummary.csv"
     USING Outputters.Csv();

Upload the file to the Azure Storage Blob under the cdrdata/scripts
directory. The ADF pipeline will read the script from this location
before sending it through the ADLA environment for processing.

Step 9) Create an Azure Machine Learning Model.
-----------------------------------------------

The objective of this sample is to demonstrate moving data from an
external source (a telco switch environment), process and store that
information in a Azure Data Lake Store, use USQL to triage that cdr
corpus to produce summary statistics, and demonstrate how this
information can be processed through an Azure Machine Learning Model for
performing anomaly or predictive analytic tasks. It is mot within the
scope of this sample to train the appropriate ML model. Instead, to
demonstrate the engineering aspects of processing the information
through an AML model we will create an “identity model” within the AML
environment which will serve as a proxy for a trained AML model.

![](media/mlmodel.png)

Figure 7: The 'Identity' Machine Learning Model

“identity Model.” This model expects 4 fields as input, the call time,
the switch, the outgoing trunk, and the number of calls over the past
hour on that switch. A trained model would use this information to
identity anomalous switch behavior or predict future possible issues
within the telecom company’s switch fabric. In this example the model
will return what is posted to it. This information will ultimately be
stored by the ADF ML Pipeline in the Azure storage blob under the
/cdrdata/summary location for use by network analysts.

The AML Model consists of a Python processing module, a “Manual Data
Entry” module that will represent “training data.” And the necessary
input and output web service endpoints.

![](media/manualDataEntry.png)

Figure 8: Adding manual training data

To create the model, create a new experiment and replicate the structure shown in the illustration within the AML Studio. Click on the “Enter Data Manually” module and enter
information as illustrated in figure 7. Be sure to unselect the “HasHeader” checkbox. This manual data entry module is serving only to inform the model that it expects 4 fields, the first three are strings and the fourth is numeric.

The python script requires no editing and should look as follows:

    def azureml_main(dataframe1 = None, dataframe2 = None):
       import pandas as pd
       return dataframe1



Save the model, then run and deploy as a web service. Record the API Key
and the Batch Execution endpoint. They will be required to call the
model from the ADF ML Pipeline.

Step 10) Create an Azure Data Factory
-------------------------------------

Next we will create an Azure Data Factory. The data factory will contain
the following 4 pipelines:

-   The “blobToDataLakePipeline” for moving CDR data from Azure Blob
    Storage to the Azure Data Lake

-   The “cdrSummaryPipeline” responsible for coordinating the processing
    of the USQL operation over the CDR data residing in the Azure Data
    Lake Store

-   The “dataLakeSummaryToBlobPipeline” for moving calculated summary
    statistics from the Azure Data Lake Store to the Azure Blob Storage
    for processing through the appropriate Azure Machine
    Learning models. In this scenario the “identity model”
    described above.

-   The “MLPipeline” for coordinating and scheduling interaction with
    the Azure Machine Learning models through the model’s batch
    web service.

The pipeline will maintain the following linked services:

-   The AzureDataLakeAnalyticsLinkedService responsible for maintaining
    access information for the Azure Data Lake Analytics environment.
    This is a compute node that will orchestrate the execution of the
    U-SQL script.

-   The AzureDataLakeStoreLinkedService responsible for managing access
    information for the Azure Data Lake Storage account.

-   The AzureStorageLinkedService responsible for maintaining access to
    the Azure Storage Account associated with the Azure Blob storage
    used in this sample.

-   The IdentityMLLinkedService encapsulating the compute node
    associated with the Identity AML model.

And the following data set identifiers:

-   AzureBlobCDRData. This dataset represents the CDR staged data prior
    to its periodic loading to the Azure Data Lake Store.

-   AzureBlobCDRAggregate. This dataset represents hourly summary data
    produced by the U-SQL query running over the Azure Data Lake Store
    CDR repository. This data originates with the ADLS but is moved by
    the dataLakeSummaryToBlobPipeline to the Azure Blob Storage area.

-   AzureBlobCDRAggregateSource. This dataset is an abstraction of the
    AzureBlobCDRAggregate representing a source dataset for MLPipeline.

-   AzureBlobCDRProcessed. This dataset in Azure Blob Storage represents
    the response from the Azure Machine Learning model. In this sample
    it is an Identity Model.

-   DataLakeTable. This data set in Azure Data Lake Storage represents
    the CDR warehouse.

-   DataLakeCDRAggregateTable. This dataset in Azure Data Lake Storage
    is a staging dataset for the results of the U-SQL query prior to the
    data being moved to the Azure Blob Storage.

-   DataLakeCDRAggregateSource. This dataset in Azure Data Lake Storage
    is an abstraction of the DataLakeCDRAggregateTable that is used as a
    source for the dataLakeSummaryToBlobPipeline.

**&lt;&lt;&lt; provide link to template for creating the ADF here
&gt;&gt;&gt;**

Conclusion
==========

This sample has illustrated the movement of data between an external
data source, an Azure Data Storage Blob, and an Azure Data Lake Store.
It demonstrated calling USQL for processing information residing in the
Data Lake Store and demonstrated processing information through a Azure
Machine Learning model. Future versions of this sample will include
integration with PowerBI for consumption by analysts.
