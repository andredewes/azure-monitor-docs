---
title: Stream Azure monitoring data to an event hub and external partners
description: Learn how to stream your Azure monitoring data to an event hub to get the data into a partner SIEM or analytics tool.
services: azure-monitor
author: EdB-MSFT
ms.author: edbaynash
ms.topic: conceptual
ms.date: 08/09/2023
ms.reviewer: lualderm
---
# Stream Azure monitoring data to an event hub or external partner

In most cases, the most effective method to stream data from Azure Monitor to external tools is by using [Azure Event Hubs](/azure/event-hubs/). This article provides a brief description on how to stream data and then lists some of the partners where you can send it. Some partners have special integration with Azure Monitor and might be hosted on Azure.

## Create an Event Hubs namespace

Before you configure streaming for any data source, you need to [create an Event Hubs namespace and event hub](/azure/event-hubs/event-hubs-create). This namespace and event hub is the destination for all of your monitoring data. An Event Hubs namespace is a logical grouping of event hubs that share the same access policy, much like a storage account has individual blobs within that storage account. Consider the following details about the Event Hubs namespace and event hubs that you use for streaming monitoring data:

* The number of throughput units allows you to increase throughput scale for your event hubs. Only one throughput unit is typically necessary. If you need to scale up as your log usage increases, you can manually increase the number of throughput units for the namespace or enable auto inflation.
* The number of partitions allows you to parallelize consumption across many consumers. A single partition can support up to 20 MBps or approximately 20,000 messages per second. Depending on the tool consuming the data, it might or might not support consuming from multiple partitions. Four partitions are reasonable to start with if you're not sure about the number of partitions to set.
* You set message retention on your event hub to at least seven days. If your consuming tool goes down for more than a day, this retention ensures that the tool can pick up where it left off for events up to seven days old.
* You should use the default consumer group for your event hub. There's no need to create other consumer groups or use a separate consumer group unless you plan to have two different tools consume the same data from the same event hub.
* For the Azure activity log, you pick an Event Hubs namespace, and Azure Monitor creates an event hub within that namespace called _insights-logs-operational-logs_. For other log types, you can either choose an existing event hub or have Azure Monitor create an event hub per log category.
* Outbound port 5671 and 5672 must typically be opened on the computer or virtual network consuming data from the event hub.

## Monitoring data available
[Sources of monitoring data for Azure Monitor and their data collection methods](../data-sources.md) describes the different kinds of data collected by Azure Monitor and the methods used to collect them. See that article for that data that can be streamed to an event hub and links to configuration details.


## Stream diagnostics data

Use diagnostics setting to stream logs and metrics to Event Hubs.
For information on how to set up diagnostic settings, see [Create diagnostic settings](./create-diagnostic-settings.md)

The following JSON is an example of metrics data sent to an event hub:

```json
[
  {
    "records": [
      {
        "count": 2,
        "total": 0.217,
        "minimum": 0.042,
        "maximum": 0.175,
        "average": 0.1085,
        "resourceId": "/SUBSCRIPTIONS/ABCDEF12-3456-78AB-CD12-34567890ABCD/RESOURCEGROUPS/RG-001/PROVIDERS/MICROSOFT.WEB/SITES/SCALEABLEWEBAPP1",
        "time": "2023-04-18T09:03:00.0000000Z",
        "metricName": "CpuTime",
        "timeGrain": "PT1M"
      },
      {
        "count": 2,
        "total": 0.284,
        "minimum": 0.053,
        "maximum": 0.231,
        "average": 0.142,
        "resourceId": "/SUBSCRIPTIONS/ABCDEF12-3456-78AB-CD12-34567890ABCD/RESOURCEGROUPS/RG-001/PROVIDERS/MICROSOFT.WEB/SITES/SCALEABLEWEBAPP1",
        "time": "2023-04-18T09:04:00.0000000Z",
        "metricName": "CpuTime",
        "timeGrain": "PT1M"
      },
      {
        "count": 1,
        "total": 1,
        "minimum": 1,
        "maximum": 1,
        "average": 1,
        "resourceId": "/SUBSCRIPTIONS/ABCDEF12-3456-78AB-CD12-34567890ABCD/RESOURCEGROUPS/RG-001/PROVIDERS/MICROSOFT.WEB/SITES/SCALEABLEWEBAPP1",
        "time": "2023-04-18T09:03:00.0000000Z",
        "metricName": "Requests",
        "timeGrain": "PT1M"
      },
    ...
    ]
  }
]
```

The following JSON is an example of log data sent to an event hub:


```json
[
  {
    "records": [
      {
        "time": "2023-04-18T09:39:56.5027358Z",
        "category": "AuditEvent",
        "operationName": "VaultGet",
        "resultType": "Success",
        "correlationId": "12345678-abc-4bc5-9f31-950eaf3bfcb4",
        "callerIpAddress": "10.0.0.10",
        "identity": {
          "claim": {
            "http://schemas.microsoft.com/identity/claims/objectidentifier": "123abc12-abcd-9876-cdef-123abc456def",
            "appid": "12345678-a1a1-b2b2-c3c3-9876543210ab"
          }
        },
        "properties": {
          "id": "https://mykeyvault.vault.azure.net/",
          "clientInfo": "AzureResourceGraph.IngestionWorkerService.global/1.23.1.224",
          "requestUri": "https://northeurope.management.azure.com/subscriptions/ABCDEF12-3456-78AB-CD12-34567890ABCD/resourceGroups/rg-001/providers/Microsoft.KeyVault/vaults/mykeyvault?api-version=2023-02-01&MaskCMKEnabledProperties=true",
          "httpStatusCode": 200,
          "properties": {
            "sku": {
              "Family": "A",
              "Name": "Standard",
              "Capacity": null
            },
            "tenantId": "12345678-abcd-1234-abcd-1234567890ab",
            "networkAcls": null,
            "enabledForDeployment": 0,
            "enabledForDiskEncryption": 0,
            "enabledForTemplateDeployment": 0,
            "enableSoftDelete": 1,
            "softDeleteRetentionInDays": 90,
            "enableRbacAuthorization": 0,
            "enablePurgeProtection": null
          }
        },
        "resourceId": "/SUBSCRIPTIONS/ABCDEF12-3456-78AB-CD12-34567890ABCD/RESOURCEGROUPS/RG-001/PROVIDERS/MICROSOFT.KEYVAULT/VAULTS/mykeyvault",
        "operationVersion": "2023-02-01",
        "resultSignature": "OK",
        "durationMs": "16"
      }
    ],
    "EventProcessedUtcTime": "2023-04-18T09:42:07.0944007Z",
    "PartitionId": 1,
    "EventEnqueuedUtcTime": "2023-04-18T09:41:14.9410000Z"
  },
...
```

## Manual streaming with a logic app

For data that you can't directly stream to an event hub, you can write to Azure Storage and then you can use a time-triggered logic app that [pulls data from Azure Blob Storage](/azure/connectors/connectors-create-api-azureblobstorage#add-action) and [pushes it as a message to the event hub](/azure/connectors/connectors-create-api-azure-event-hubs#add-action).


## Partner tools with Azure Monitor integration

Routing your monitoring data to an event hub with Azure Monitor enables you to easily integrate with external SIEM and monitoring tools. The following table lists examples of tools with Azure Monitor integration.

| Tool | Hosted in Azure | Description |
|:---|:---| :---|
|  IBM QRadar | No | The Microsoft Azure DSM and Microsoft Azure Event Hubs Protocol are available for download from [the IBM support website](https://www.ibm.com/support). |
| Splunk | No | [Splunk Add-on for Microsoft Cloud Services](https://splunkbase.splunk.com/app/3110/) is an open-source project available in Splunkbase. <br><br> If you can't install an add-on in your Splunk instance and, for example, you're using a proxy or running on Splunk Cloud, you can forward these events to the Splunk HTTP Event Collector by using [Azure Function for Splunk](https://github.com/Microsoft/AzureFunctionforSplunkVS). This tool is triggered by new messages in the event hub. |
| SumoLogic | No | Instructions for setting up SumoLogic to consume data from an event hub are available at [Collect Logs for the Azure Audit App from Event Hubs](https://help.sumologic.com/docs/integrations/microsoft-azure/audit/#collecting-logs-for-the-azure-audit-app-from-event-hub). |
| ArcSight | No | The ArcSight Azure Event Hubs smart connector is available as part of [the ArcSight smart connector collection](https://community.microfocus.com/cyberres/arcsight/f/arcsight-product-announcements/163662/announcing-general-availability-of-arcsight-smart-connectors-7-10-0-8114-0). |
| Syslog server | No | If you want to stream Azure Monitor data directly to a Syslog server, you can use a [solution based on an Azure function](https://github.com/miguelangelopereira/azuremonitor2syslog/).
| LogRhythm | No| Instructions to set up LogRhythm to collect logs from an event hub are available at [this LogRhythm website](https://logrhythm.com/six-tips-for-securing-your-azure-cloud-environment/).
|Logz.io | Yes | For more information, see [Get started with monitoring and logging by using Logz.io for Java apps running on Azure](/azure/developer/java/fundamentals/java-get-started-with-logzio).

## Next steps
* [Read the overview of the Azure activity log](../essentials/platform-logs-overview.md)
* [Set up an alert based on an activity log event](../alerts/alerts-log-webhook.md)
