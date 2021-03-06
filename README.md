# From SAP S/4HANA Core Data Services (CDS) to the Common Data Model (CDM) on Azure Data Lake Gen2 (ADLS Gen2) 
This demo describes the basic integration between S/4HANA Core Data Services Views (CDS Views) and the Microsoft Common Data Model (CDM)



## Introduction and motivation
When utilizing SAP S/4HANA or SAP ECC as data-source for modern Data-Lake implementations, the complexity of the relational SAP data-model and corresponding SAP tables is a well-known challenge. 
For data extraction based on SAP ABAP tables, deep SAP domain expertise is required to design specific data-models which fulfill the requirements of AI/ML or general analytical scenarios. 

This is the SAP Accounting Document Segment table as an example: [BSEG](https://www.se80.co.uk/saptables/b/bseg/bseg.htm)

Frequently in Azure Big Data analytics scenarios these SAP specific data models and semantics play in important role in the overall analytical data-models, hence this demo provides an efficient method the convert SAP S/4HANA CDS semantics to CDM models. 

ADLS Gen2 supports the Common Data Model (CDM) which enables consistent sharing of metadata between multiple data consumers and producers like PowerBI , Azure Databricks or Azure Data-Factory. Semantic and structural consistency of metadata, in context of Data-Lakes projects, is one of the key success factors to implement new innovative use-cases. ([Source](https://docs.microsoft.com/en-us/common-data-model/data-lake))

Like the SAP S/4HANA Core Data Services(CDS) views or SAP BW/4HANA business content, the Microsoft Common Data Model (CDM) provides semantically rich descriptions about datasets stored in the Azure Data Lake.


These resources can be used as introduction to the Common Data Model and the Azure Data Services integration:

* [The Common Data Model](https://github.com/Azure-Samples/cdm-azure-data-services-integration/blob/master/Tutorial/CDM-Azure-Data-Services-Integration-Tutorial.md)

* [CDM and Azure Data Services Integration](https://github.com/Azure-Samples/cdm-azure-data-services-integration/blob/master/Tutorial/CDM-Azure-Data-Services-Integration-Tutorial.md)

* [Microsoft Power BI: Common Data Model and Azure Data Services - BRK3033](https://www.youtube.com/watch?v=wzbjAkk6m6k)

## Introduction demo scenario and architecture 

The overall scenario of this tutorial and prototype is to transform SAP S/4HANA specific metadata and semantics into the CDM-format and to export the data to the ADLS Gen2.  
In practice this approach could be used for rapid prototyping using SAP S/4HANA as data-source. Productive usage beyond full data loads from is currently not recommended since the implementation of the ODP CDC/Delta API is not available in this demo yet. 
In later stages of the demo the incremental refresh feature of PBI could evaluated for this demo. ([PBI Incremental refresh](https://docs.microsoft.com/en-us/power-bi/service-dataflows-incremental-refresh))

The semantic model conversion and data-movement will be implemented using Power BI Premium Dataflows connected to an Azure Data Lake Gen2.
In S/4HANA analytical data models are described with CDS-view, based on SAP HANA tables, and provide option the option to export SAP-data leveraging open interfaces like OData. 

![Architecture](https://github.com/ROBROICH/SAP_AND_COMMON_DATA_MODEL_DEMO/blob/master/SCENARIO_ARCHITECTURE.png)

This series of blogs from Simon Kranig are an excellent introduction to CDS based data extraction: [Link](https://blogs.sap.com/2019/12/13/cds-based-data-extraction-part-i-overview/)

In addition to Simons Blog there is an introduction how to enable OData based extraction using CDS view:
[Link](https://github.com/ROBROICH/SAP_ODP_ODATA_CLIENT)

In this scenario the S/4HANA CDS view for extracting the S/4HANA Cash Flow was selected. This CDS view joins multiple tables or CDS-view to provide the S4/HANA cash-flow for data extraction:
![Cash-Flow](https://github.com/ROBROICH/SAP_AND_COMMON_DATA_MODEL_DEMO/blob/master/CDS_VIEW_CASHFLOW.png)

## Prerequisites 
To implement this prototype, the following prerequisites have to be implemented:

* An ABAP based CDS view must be exposed as ODP OData service
  *	The prerequisite for exposing CDS views as ODP is the CDS-tag data-extraction = true. 
   ![ExtractionTrue](https://github.com/ROBROICH/SAP_AND_COMMON_DATA_MODEL_DEMO/blob/master/CDS_EXTRACTION_TRUE.png)
  
 * The only required modification is to expose the CDS-view as OData service. Further details how this can be achieved and a description how to implement the CDC / delta-logic can be found here: [Link](https://github.com/ROBROICH/SAP_ODP_ODATA_CLIENT)

*	The PBI On-Premise Gateway was used to connect the OData service with the PBI Dataflow. The installation guide can be found here:[Link](https://docs.microsoft.com/en-us/power-bi/service-gateway-onprem)

## Connecting the PBI dataflow to S/4HANA
In this example a S/4HANA system running on Azure was used to implement the prototype.
To establish the connectivity between the S/4HANA system and the PBI dataflow the installation of the On-Premise data gateway is required. 

After the successful installation of the data gateway, following steps are required to consume the OData service using PBI DataFlow:
#### 1. Create a new PBI DataFlow 
Introduction for creating PBI Dataflows: [Link](https://docs.microsoft.com/en-us/power-bi/service-dataflows-create-use)
![PBIDATAFLOW](https://github.com/ROBROICH/SAP_AND_COMMON_DATA_MODEL_DEMO/blob/master/PBI_CREATE_DATA_FLOW.png)
#### 2. Select "Define new entities"
![PBISCREATENEWENTITY](https://github.com/ROBROICH/SAP_AND_COMMON_DATA_MODEL_DEMO/blob/master/PBI_CREATE_DATA_NEW_ENTITY.png)
#### 3. Select OData as data source 
![PBISCREATENEWENTITY](https://github.com/ROBROICH/SAP_AND_COMMON_DATA_MODEL_DEMO/blob/master/PBI_SELECT_ODATA.png)
#### 4. Enter the URL of the S/4 ODP-OData service and select the On-Premise data gateway
![PBIMAINTAINODATA](https://github.com/ROBROICH/SAP_AND_COMMON_DATA_MODEL_DEMO/blob/master/PBI_MAINTAIN_ODATA.png)
#### 5. Select the full load URL
![PBIODATAPREVIEWSERVIVCE](https://github.com/ROBROICH/SAP_AND_COMMON_DATA_MODEL_DEMO/blob/master/PBI_ODATA_PREVIEWS_SERVICE.png)
#### 6. If required, apply transformations in PBI Power Query
![PBIPOWERQUERY](https://github.com/ROBROICH/SAP_AND_COMMON_DATA_MODEL_DEMO/blob/master/PBI_POWERQUERY.png)
#### 7. Refresh the PBI Dataflow 
If everything is working, e.g. the data preview shown. Refresh the data flow once.
![PBIREFRESHDATAFLOW](https://github.com/ROBROICH/SAP_AND_COMMON_DATA_MODEL_DEMO/blob/master/PBI_REFRESH_DATA_FLOW.png)

## S/4HANA stored on Azure Data Lake Gen2
After the PBI Dataflow is executed, the S/4HANA data is stored in the Azure Data-Lake Gen2. 
The export consists of the following artifact:
### Folder structure: 
![ADATAEXPLORER](https://github.com/ROBROICH/SAP_AND_COMMON_DATA_MODEL_DEMO/blob/master/ADE_Model_JSON.png)
### Content Model JSON:
The model JSON describes the column name and the datatype. 
![VSCMODELSJON](https://github.com/ROBROICH/SAP_AND_COMMON_DATA_MODEL_DEMO/blob/master/VSC_MODELJSON.png)
### Snapshot CSV file:
![VSCMODELSJON](https://github.com/ROBROICH/SAP_AND_COMMON_DATA_MODEL_DEMO/blob/master/ADE_CSV.png)
From here the Model.json file is prepared to be consumed from PowerBI or Azure Databricks
## Connecting PowerBI Desktop to the PowerBI-Dataflow
Once the PBI DataFlow is created to the SAP S/4HANA data is ready to be consumed in PowerBI
### Select the PBI DataFlow as data-source 
![PBIDATAFLOW1](https://github.com/ROBROICH/SAP_AND_COMMON_DATA_MODEL_DEMO/blob/master/PBI_DESKTOP_DATAFLOW_1.png)
### Select PBI DataFlow for exporting the S/4HANA Cash-Flow
![PBIDATAFLOW1](https://github.com/ROBROICH/SAP_AND_COMMON_DATA_MODEL_DEMO/blob/master/PBI_DESKTOP_DATAFLOW_2.png)


## Create a PBI dashboard for the S/4HANA Cash-Flow data and get some Quick Insights 

### S/4HANA Cash Flow report in PBI Desktop
![PBIDASHBOARD1](https://github.com/ROBROICH/SAP_AND_COMMON_DATA_MODEL_DEMO/blob/master/PBI_DESKTOP_DASHBOARD_S4.png)

### S/4HANA Cash Flow report in PBI 
![PBIDASHBOARD2](https://github.com/ROBROICH/SAP_AND_COMMON_DATA_MODEL_DEMO/blob/master/PBI_DASHBOARD_S4.png)

### PBI Quick Insights generated from S/4HANA Cash Flow 
![QUICKINISHGTS](https://github.com/ROBROICH/SAP_AND_COMMON_DATA_MODEL_DEMO/blob/master/PBI_QUICKINSIGHTS.png)


## Summary and outlook 
This demo demonstrated the capability of PowerBI to consume S/4HANA semantics and data by utilizing S/4HANA CDS views in combination with the open interface OData. 
In addition to the S/4AHANA semantics, the Cash-Flow data model and the readable columns names, are available on the ADL Gen2 as CDM model and could be consumed by additional tools like Azure Databricks. 
While the preparation of the S/4HANA CDS as OData service is a change request for the IT department, the later consumption of the OData service with PBI does not require the involvement of the IT department and can be done by power users. 
As mentioned earlier, currently only full data-loads are possible with this demo scenario and therefore this demo is not yet ready for data-warehousing like reoccurring data-loads. 

Therefore, the next features for this demo scenario will likely be:

*	Consuming the CDM model and data using Azure Databricks
* Implement the reoccurring data loads  

##  Resource collection:

* [The Common Data Model](https://github.com/Azure-Samples/cdm-azure-data-services-integration/blob/master/Tutorial/CDM-Azure-Data-Services-Integration-Tutorial.md)

* [CDM and Azure Data Services Integration](https://github.com/Azure-Samples/cdm-azure-data-services-integration/blob/master/Tutorial/CDM-Azure-Data-Services-Integration-Tutorial.md)

* [Create schema documents for the Common Data Model](https://github.com/Azure-Samples/cdm-azure-data-services-integration/blob/master/Tutorial/CDM-Azure-Data-Services-Integration-Tutorial.md)
 
* [Create schema documents for the Common Data Model](https://docs.microsoft.com/en-us/common-data-model/creating-schemas)
 
* [How to add metadata to your Azure Data Lake](https://towardsdatascience.com/how-to-add-metadata-to-your-azure-data-lake-f8ec2022f50)
 
* [Microsoft Power BI: Common Data Model and Azure Data Services - BRK3033](https://www.youtube.com/watch?v=wzbjAkk6m6k)




