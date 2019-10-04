# Useful Azure Data Explorer queries for Azure Data Factory pipelines

## Monitoring the Azure Data Factory Pipelines

Monitoring Azure Data Factory is enabled through collecting diagnostic logs and posting them to Log Analytics (part of Azure Monitor). The other options include – Storage Account or Event Hub for custom processing. There are additional settings provide options to decide what information that can are to be captured. It is also possible to have multiple settings for collecting diagnostic logs, where it is possible to send different data to different collection stores.

The most readily available monitoring is available through Log Analytics Workspace and Azure Data Factory Workbook available in the Azure Marketplace. Detailed documentation to enable monitoring through is available [here] (https://docs.microsoft.com/en-us/azure/data-factory/monitor-using-azure-monitor#monitor-data-factory-metrics-with-azure-monitor).

## Useful Azure Data Explorer Queries

While the workbook is helpful in surfacing some of the key metrics across multiple data factory instance it is helpful to write couple of custom queries in Azure Data Explorer to review some performance statistics of a Pipeline run.

Following the instructions in the link provided in the section above, you will have created a Log Analytics Workspace to store the Azure Data Factory diagnostic data. The data for Pipeline run is available in table called – “ADFPipelineRun”.

### Viewing Duration of Pipeline Runs

One of the useful queries that I find is to view how long my succeeding pipeline are taking. For this example, we assume, what we are monitoring a pipeline called - *gsc2adlsgen2copy*. A query for the pipeline that would look like:

   ```sql
    ADFPipelineRun 
    | where Status == "Succeeded"
    | where PipelineName == "gcs2adlsgen2copy"
    | project PipelineName, RunId, Start, End, (End - Start)
  ```

The query above returns all the execution time of the pipeline. This query returns all the executions of the pipeline (within the time frame set by the query explorer) and how much time they took.

### Viewing number of Pipeline Runs exceeding a threshold

If we have established an SLA for the Pipeline run, we can add a threshold value into the query to view pipelines that exceed the threshold. The query would look like this with an arbitrary threshold of 15 seconds:

   ```sql
    ADFPipelineRun 
    | where Status == "Succeeded"
    | where PipelineName == "gcs2adlsgen2copy"
    | where (End - Start) > 15s
    | project PipelineName, RunId, Start, End, (End - Start)
   ```

#### Generating Alerts on missed SLA

To generate an alert from the query above, click on the “+ New Alert” button on top of the query window and select to build a new alert on the Custom query.

![New Alert Button](images\img001_NewAlert.png)

Shown below is the alert created for the query above, based on number of rows returned, monitored every 24 hours for past 24 hours.

![New Alert Action](images\img002_NewAlertAction.png)

## Conclusion

Azure Pipelines can be monitored using Azure Monitor. Alerts can be generated using custom Azure Data Explorer queries when a pipeline takes too long or does not run over a period. More reasons to rejoice and rest easy, my fellow cloud dwellers.
