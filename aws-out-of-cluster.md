AWS Out of Cluster 
==================

Integrating Kubecost with your AWS data provides the ability to allocate out-of-cluster (OOC) costs, e.g. RDS instances and S3 buckets, back to Kubernetes concepts like namespace and deployment as well as reconcile cluster assets back to your billing data. The latter is especially helpful when teams are using Reserved Instances, Savings Plans, or Enterprise Discounts. All billing data remains on your cluster when using this functionality and is not shared externally. Read the [Cloud Integrations](/cloud-integration.md) documentation for more information on how Kubecost connects with Cloud Service Providers.

The following guide provides the steps required for enabling OOC costs allocation and accurate pricing, e.g. reserved instance price allocation. In a multi-account organization, all of the following steps will need to be completed in the payer account.

## Step 1: Create an AWS Cost and Usage Report (CUR) and Integrate it with Kubecost

Follow our guide on [AWS Cloud Integration](/aws-cloud-integrations.md).

## Step 2: Tag your resources
Kubecost utilizes AWS tagging to allocate the costs of AWS resources outside of the Kubernetes cluster to specific Kubernetes concepts, such as namespaces, pods, etc. These costs are then shown in a unified dashboard within the Kubecost interface.

To allocate external AWS resources to a Kubernetes concept, use the following tag naming scheme:

| Kubernetes Concept| AWS Tag Key | AWS Tag Value |
|--------------------|---------------------|---------------|
| Cluster | kubernetes_cluster	| cluster-name	|
| Namespace | kubernetes_namespace	| namespace-name |
| Deployment | kubernetes_deployment	| deployment-name |
| Label | kubernetes\_label\_NAME* | label-value    |
| DaemonSet | kubernetes_daemonset	| daemonset-name |
| Pod | kubernetes_pod	    | pod-name     |
| Container | kubernetes_container	| container-name |

> **Note**: In the `kubernetes_label_NAME` tag key, the `NAME` portion should appear exactly as the tag appears inside of Kubernetes. For example, for the tag `app.kubernetes.io/name`, this tag key would appear as `kubernetes_label_app.kubernetes.io/name`.

To use an alternative or existing AWS tag schema, you may supply these in your [values.yaml](https://github.com/kubecost/cost-analyzer-helm-chart/blob/v1.73.0/cost-analyzer/values.yaml#L589) under the `kubecostProductConfigs.labelMappingConfigs.\<aggregation\>\_external_label`. Also be sure to set `kubecostProductConfigs.labelMappingConfigs.enabled = true`.

More on AWS tagging [here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_Tags.html).

Tags may take several hours to show up in the *Cost Allocations Tags* section described in the next step.

### Custom tags mapping

Tags that contain `:` in the key may be converted to `_` in the Kubecost UI due to Prometheus readability. To use AWS Label Mapping Configs, use this mapping format:

```
kubecostProductConfigs:
  labelMappingConfigs:
    enabled: true
    namespace_external_label: mycompanybilling_namespace
    cluster_external_label: mycompanybilling_cluster
 ```
 
 To view examples of common label mapping configs, see [here](https://github.com/kubecost/cost-analyzer-helm-chart/blob/ef50e9b959e5b91890333b047830d604ec37d112/cost-analyzer/values.yaml#L947-L966).

## Step 3: Enable user-defined cost allocation tags

In order to make the custom Kubecost AWS tags appear on the CURs, and therefore in Kubecost, individual cost allocation tags must be enabled. Details on which tags to enable can be found in Step 2.

Instructions for enabling user-defined cost allocation tags [here](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/activating-tags.html)

### Viewing account-level tags

Account-level tags are applied (as labels) to all the Assets built from resources defined under a given AWS account. You can filter AWS resources in the Kubecost Assets View (or API) by account-level tags by adding them ('tag:value') in the Label/Tag filter.

If a resource has a label with the same name as an account-level tag, the resource label value will take precedence.

Modifications incurred on account-level tags may take several hours to update on Kubecost.

Your AWS account will need to support the `organizations:ListAccounts` and `organizations:ListTagsForResource` policies to benefit from this feature.

## Troubleshooting

* In the Kubecost UI, view the Allocations dashboard. If external costs are not shown, open your browser's Developer Tools > Console to see any reported errors.
* Query Athena directly to ensure data is available. Note: it can take up to 6 hours for data to be written. 
* You may need to upgrade your AWS Glue if you are running an old version. See [Upgrading to the AWS Glue Data Catalog step-by-step](https://docs.aws.amazon.com/athena/latest/ug/glue-upgrade.html) for more info.
* Finally, review pod logs from the `cost-model` container in the `cost-analyzer` pod and look for auth errors or Athena query results. 
