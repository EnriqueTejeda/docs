# Clusters Dashboard

{% hint style="warning" %}
The Clusters dashboard is currently a beta feature. Please read the documentation carefully.
{% endhint %}

## Overview

The Clusters dashboard provides a list of all your monitored clusters, as well as additional clusters detected in your cloud bill. The dashboard provides details about your clusters including cost, efficiency, and cloud provider. You are able to filter your list of clusters by when clusters were last seen, activity status, and by name (see below).

{% hint style="info" %}
Monitoring of multiple clusters is only supported in [Kubecost Enterprise](https://www.kubecost.com/pricing/) plans. Learn more about Kubecost Enterprise's multi-cluster view [here](https://docs.kubecost.com/install-and-configure/install/multi-cluster).
{% endhint %}

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption><p>Clusters dashboard</p></figcaption></figure>

## Enabling Clusters dashboard

To enable the Clusters dashboard, you must perform these two steps:

1. Enable [cloud integration](https://docs.kubecost.com/install-and-configure/install/cloud-integration) for any and all cloud service providers you wish to view clusters with
2. Enable Cloud Costs

Enabling Cloud Costs through Helm can be done using the following parameters:

```
kubecostModel:
  cloudCost:
     enabled: true
     labelList:
       isIncludeList: false
       # format labels as comma separated string (ex. "label1,label2,label3")
       labels: ""
     topNItems: 1000
```

## Usage

Clusters are primarily distinguished into three categories:

* Clusters monitored by Kubecost (green circle next to cluster name)
* Clusters not monitored by Kubecost (yellow circle next to cluster name)
* Inactive clusters (gray circle next to cluster name)

Monitored clusters are those that have cost metrics which will appear within your other Monitoring dashboards, like Allocations and Assets. Unmonitored clusters are clusters whose existence is determined from cloud integration, but have not been added to Kubecost. Inactive clusters are clusters Kubecost once monitored, but hasn't reported data over a certain period of time. This time period is three hours for Thanos-enabled clusters, and one hour for non-Thanos clusters.

Efficiency and Last Seen metrics are only provided for monitored clusters.

{% hint style="info" %}
Efficiency is calculated as the amount of node capacity that is used, compared to what is available.
{% endhint %}

Selecting any metric in a specific cluster's row will take you to a Cluster Details page for that cluster which provides more extensive metrics, including assets and namespaces associated with that cluster and their respective cost metrics.

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption><p>Cluster Details page</p></figcaption></figure>

### Filtering clusters

You are able to filter clusters through a window of when all clusters were last seen (default is _Last 7 days_). Although unmonitored clusters will not provide a metric for Last Seen, they will still appear in applicable windows.

{% hint style="danger" %}
As a beta feature, certain windows may result in long load times, or may not load properly. Thank you for your patience while using this feature. Load times will be improved in future releases.
{% endhint %}

You can also filter your clusters for _Active_, _Inactive_, or _Unmonitored_ status, and search for clusters by name.
