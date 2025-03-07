# Allocation API

> **Note**: Throughout our API documentation, we use `localhost:9090` as the default Kubecost URL, but your Kubecost instance may be exposed by a service or ingress. To reach Kubecost at port 9090, run: `kubectl port-forward deployment/kubecost-cost-analyzer -n kubecost 9090`. When querying the cost-model container directly (ex. localhost:9003), the `/model` part of the URI should be removed.

{% swagger method="get" path="/allocation" baseUrl="http://<your-kubecost-address>/model" summary="Allocation API" %}
{% swagger-description %}
The Allocation API is the preferred way to query for costs and resources allocated to Kubernetes workloads and optionally aggregated by Kubernetes concepts like 

`namespace`

, 

`controller`

, and 

`label`

. Data is served from one of 

[Kubecost's ETL pipelines](cost-model-deprecated.md#caching-overview)

.
{% endswagger-description %}

{% swagger-parameter in="path" name="window" type="string" required="true" %}
Duration of time over which to query. Accepts words like `today`, `week`, `month`, `yesterday`, `lastweek`, `lastmonth`; durations like `30m`, `12h`, `7d`; comma-separated RFC3339 date pairs like

`2021-01-02T15:04:05Z,2021-02-02T15:04:05Z`; comma-separated Unix timestamp (seconds) pairs like `1578002645,1580681045`.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="aggregate" type="string" required="false" %}
Field by which to aggregate the results. Accepts:

`cluster`, `namespace`, `controllerKind`, `controller`, `service`, `node`, `pod`, `label:<name>`, and `annotation:<name>`. Also accepts comma-separated lists for multi-aggregation, like

`namespace,label:app`.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="accumulate" type="boolean" required="false" %}
If `true`, sum the entire range of sets into a single set. Default value is

`false`.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="idle" type="boolean" required="false" %}
If `true`, include idle cost (i.e. the cost of the un-allocated assets) as its own allocation. (See [special types of allocation](https://docs.kubecost.com/apis/apis-overview/allocation#special-types-of-allocation).) Default is

`true.`
{% endswagger-parameter %}

{% swagger-parameter in="path" name="external" type="boolean" required="false" %}
If 

`true`

, include 

[external costs](https://docs.kubecost.com/#advanced-configuration)

 in each allocation. Default is 

`false`

.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="filterClusters" type="string" required="false" %}
Comma-separated list of clusters to match; e.g.

`cluster-one,cluster-two`

will return results from only those two clusters.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="filterNodes" type="string" required="false" %}
Comma-separated list of nodes to match; e.g.

`node-one,node-two`

will return results from only those two nodes.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="filterNamespaces" type="string" required="false" %}
Comma-separated list of namespaces to match; e.g.

`namespace-one,namespace-two`

will return results from only those two namespaces.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="filterControllerKinds" type="string" required="false" %}
Comma-separated list of controller kinds to match; e.g.

`deployment,job`

will return results with only those two controller kinds.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="filterControllers" type="string" required="false" %}
Comma-separated list of controllers to match; e.g. 

`deployment-one,statefulset-two`

 will return results from only those two controllers.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="filterPods" type="string" required="false" %}
Comma-separated list of pods to match; e.g. `pod-one,pod-two`

will return results from only those two pods.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="filterAnnotations" type="string" required="false" %}
Comma-separated list of annotations to match; e.g. `name:annotation-one,name:annotation-two`

will return results with either of those two annotation key-value-pairs.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="filterLabels" type="string" required="false" %}
Comma-separated list of annotations to match; e.g. `app:cost-analyzer, app:prometheus`

will return results with either of those two label key-value-pairs.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="filterServices" type="string" required="false" %}
Comma-separated list of services to match; e.g. `frontend-one,frontend-two`

will return results with either of those two services.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="format" type="string" required="false" %}
Set to `csv` to download an accumulated version of the allocation results in CSV format. Set to `pdf`

to download an accumulated version of the allocation results in PDF format. By default, results will be in JSON format.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="shareIdle" type="boolean" required="false" %}
If `true`, idle cost is allocated proportionally across all non-idle allocations, per-resource. That is, idle CPU cost is shared with each non-idle allocation's CPU cost, according to the percentage of the total CPU cost represented. Default is

`false`.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="splitIdle" type="boolean" required="false" %}
If `true`, and `shareIdle == false`, Idle Allocations are created on a per cluster or per node basis rather than being aggregated into a single "_idle_" allocation. Default is

`false`.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="idleByNode" type="boolean" required="false" %}
If 

`true`

, idle allocations are created on a per node basis. Which will result in different values when shared and more idle allocations when split. Default is 

`false`

.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="reconcile" type="boolean" required="false" %}
If 

`true`

, pulls data from the Assets cache and corrects prices of Allocations according to their related Assets. The corrections from this process are stored in each cost categories cost adjustment field. If the integration with your cloud provider's billing data has been set up, this will result in the most accurate costs for Allocations. Default is 

`true`

.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="shareTenancyCosts" type="boolean" required="false" %}
If 

`true`

, share the cost of cluster overhead assets such as cluster management costs and node attached volumes across tenants of those resources. Results are added to the sharedCost field. As of v1.93.0 both cluster management and attached volumes are shared by cluster. Default is 

`true`

.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="shareNamespaces" type="string" required="false" %}
Comma-separated list of namespaces to share; e.g.

`kube-system, kubecost`

will share the costs of those two namespaces with the remaining non-idle, unshared allocations.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="shareLabels" type="string" required="false" %}
Comma-separated list of labels to share; e.g. 

`env:staging, app:test`

 will share the costs of those two label values with the remaining non-idle, unshared allocations.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="shareCost" type="float" required="false" %}
Floating-point value representing a monthly cost to share with the remaining non-idle, unshared allocations; e.g. 

`30.42`

 ($1.00/day == $30.42/month) for the query 

`yesterday`

 (1 day) will split and distribute exactly $1.00 across the allocations. Default is 

`0.0.`
{% endswagger-parameter %}

{% swagger-parameter in="path" name="shareSplit" type="string" required="false" %}
Determines how to split shared costs among non-idle, unshared allocations. By default, the split will be `weighted`; i.e. proportional to the cost of the allocation, relative to the total. The other option is `even`; i.e. each allocation gets an equal portion of the shared cost. Default is

`weighted`.
{% endswagger-parameter %}

{% swagger-response status="200: OK" description="" %}
```json
{
    "code": 200,
    "data": [
        {
            "aws-dev-1-niko/ip-192-168-12-152.us-east-2.compute.internal/kubecost/kubecost-cost-analyzer-5b84f94b7f-9lxx5/cost-model": {
                "name": "aws-dev-1-niko/ip-192-168-12-152.us-east-2.compute.internal/kubecost/kubecost-cost-analyzer-5b84f94b7f-9lxx5/cost-model",
                "properties": {
                    "cluster": "aws-dev-1-niko",
                    "node": "ip-192-168-12-152.us-east-2.compute.internal",
                    "container": "cost-model",
                    "controller": "kubecost-cost-analyzer",
                    "controllerKind": "deployment",
                    "namespace": "kubecost",
                    "pod": "kubecost-cost-analyzer-5b84f94b7f-9lxx5",
                    "services": [
                        "kubecost-frontend",
                        "kubecost-cost-analyzer"
                    ],
                    "providerID": "i-0f227c52893c7957f",
                    "labels": {
                        "app": "cost-analyzer",
                        "app_kubernetes_io_instance": "kubecost",
                        "app_kubernetes_io_name": "cost-analyzer",
                        "kubernetes_io_metadata_name": "kubecost",
                        "node_kubernetes_io_instance_type": "m5.large",
                        "pod_template_hash": "5b84f94b7f"
                    }
                },
                "window": {
                    "start": "2023-01-27T23:00:00Z",
                    "end": "2023-01-28T00:00:00Z"
                },
                "start": "2023-01-27T23:00:00Z",
                "end": "2023-01-27T23:16:00Z",
                "minutes": 16.000000,
                "cpuCores": 0.050000,
                "cpuCoreRequestAverage": 0.050000,
                "cpuCoreUsageAverage": 0.000964,
                "cpuCoreHours": 0.013333,
                "cpuCost": 0.000426,
                "cpuCostAdjustment": 0.000000,
                "cpuEfficiency": 0.019287,
                "gpuCount": 0.000000,
                "gpuHours": 0.000000,
                "gpuCost": 0.000000,
                "gpuCostAdjustment": 0.000000,
                "networkTransferBytes": 797941.832389,
                "networkReceiveBytes": 1243829.277373,
                "networkCost": 0.000000,
                "networkCrossZoneCost": 0.000000,
                "networkCrossRegionCost": 0.000000,
                "networkInternetCost": 0.000000,
                "networkCostAdjustment": 0.000000,
                "loadBalancerCost": 0.003333,
                "loadBalancerCostAdjustment": 0.000000,
                "pvBytes": 16169288643.764706,
                "pvByteHours": 4311810305.003922,
                "pvCost": 0.000550,
                "pvs": {
                    "cluster=aws-dev-1-niko:name=pvc-16465f04-6464-483e-89af-9e68b4a59e72": {
                        "byteHours": 4311810305.0039215,
                        "cost": 0.0005500940102068225
                    }
                },
                "pvCostAdjustment": 0.000000,
                "ramBytes": 330301440.000000,
                "ramByteRequestAverage": 330301440.000000,
                "ramByteUsageAverage": 202833175.272727,
                "ramByteHours": 88080384.000000,
                "ramCost": 0.000351,
                "ramCostAdjustment": 0.000000,
                "ramEfficiency": 0.614085,
                "sharedCost": 0.000000,
                "externalCost": 0.000000,
                "totalCost": 0.004661,
                "totalEfficiency": 0.288087
            },
            "...etc": {}
        },
        {
            "...etc": {}
        }
    ]
}
```
{% endswagger-response %}
{% endswagger %}

## Allocation schema

| Field                      | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| name                       | Name of each relevant Kubernetes concept described by the allocation, delimited by slashes, e.g. "cluster/node/namespace/pod/container"                                                                                                                                                                                                                                                                                                                                                      |
| properties                 | Map of name-to-value for all relevant property fields, including: `cluster`, `node`, `namespace`, `controller`, `controllerKind`, `pod`, `container`, `labels`, `annotation`, etc. Note: Prometheus only supports underscores (`_`) in label names. Dashes (`-`) and dots (`.`), while supported by Kubernetes, will be translated to underscores by Prometheus. This may cause the merging of labels, which could result in aggregated costs being charged to a single label.               |
| window                     | Period of time over which the allocation is defined.                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| start                      | Precise starting time of the allocation. By definition must be within the window.                                                                                                                                                                                                                                                                                                                                                                                                            |
| end                        | Precise ending time of the allocation. By definition must be within the window.                                                                                                                                                                                                                                                                                                                                                                                                              |
| minutes                    | Number of minutes running; i.e. the minutes from `start` until `end`.                                                                                                                                                                                                                                                                                                                                                                                                                        |
| cpuCores                   | Average number of CPU cores allocated while running.                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| cpuCoreRequestAverage      | Average number of CPU cores requested while running.                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| cpuCoreUsageAverage        | Average number of CPU cores used while running.                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| cpuCoreHours               | Cumulative CPU core-hours allocated.                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| cpuCost                    | Cumulative cost of allocated CPU core-hours.                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| cpuCostAdjustment          | Change in cost after allocated CPUs have been reconciled with updated node cost                                                                                                                                                                                                                                                                                                                                                                                                              |
| cpuEfficiency              | Ratio of `cpuCoreUsageAverage`-to-`cpuCoreRequestAverage`, meant to represent the fraction of requested resources that were used.                                                                                                                                                                                                                                                                                                                                                            |
| gpuCount                   | Number of GPUs allocated to the workload.                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| gpuHours                   | Cumulative GPU-hours allocated.                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| gpuCost                    | Cumulative cost of allocated GPU-hours.                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| gpuCostAdjustment          | Change in cost after allocated GPUs have been reconciled with updated node cost                                                                                                                                                                                                                                                                                                                                                                                                              |
| networkTransferBytes       | Total bytes sent from the workload                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| networkReceiveBytes        | Total bytes received by the workload                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| networkCost                | Cumulative cost of network usage.                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| networkCrossZoneCost       | Cumulative cost of Cross-zone network egress usage.                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| networkCrossRegionCost     | Cumulative cost of Cross-region network egress usage.                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| networkInternetCost        | Cumulative cost of internet egress usage.                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| networkCostAdjustment      | Updated network cost                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| loadBalancerCost           | Cumulative cost of allocated load balancers.                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| loadBalancerCostAdjustment | Updated load balancer cost.                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| pvBytes                    | Average number of bytes of PersistentVolumes allocated while running.                                                                                                                                                                                                                                                                                                                                                                                                                        |
| pvByteHours                | Cumulative PersistentVolume byte-hours allocated.                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| pvCost                     | Cumulative cost of allocated PersistentVolume byte-hours.                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| pvs                        | Map of PersistentVolumeClaim costs that have been allocated to the workload                                                                                                                                                                                                                                                                                                                                                                                                                  |
| pvCostAdjustment           | Updated persistent volume cost.                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ramBytes                   | Average number of RAM bytes allocated. An allocated resource is the source of cost, according to Kubecost - regardless of if a requested resource is used.                                                                                                                                                                                                                                                                                                                                   |
| ramByteRequestAverage      | Average of the RAM requested by the workload. Requests are a [Kubernetes tool](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#requests-and-limits) for preallocating/reserving resources for a given container.                                                                                                                                                                                                                                              |
| ramByteUsageAverage        | Average of the RAM used by the workload. This comes from moment-to-moment measurements of live RAM byte usage of each container. This is roughly the number you see under RAM if you pull up Task Manager (Windows), top on Linux, or Activity Monitor (MacOS).                                                                                                                                                                                                                              |
| ramByteHours               | Cumulative RAM byte-hours allocated.                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ramCost                    | Cumulative cost of allocated RAM byte-hours.                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ramEfficiency              | Ratio of `ramByteUsageAverage`-to-`ramByteRequestAverage`, meant to represent the fraction of requested resources that were used.                                                                                                                                                                                                                                                                                                                                                            |
| sharedCost                 | Cumulative cost of shared resources, including shared namespaces, shared labels, shared overhead.                                                                                                                                                                                                                                                                                                                                                                                            |
| externalCost               | Cumulative cost of external resources.                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| totalCost                  | Total cumulative cost                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| totalEfficiency            | Cost-weighted average of `cpuEfficiency` and `ramEfficiency`. In equation form: `((cpuEfficiency * cpuCost) + (ramEfficiency * ramCost)) / (cpuCost + ramCost)`                                                                                                                                                                                                                                                                                                                              |
| rawAllocationOnly          | Object with fields `cpuCoreUsageMax` and `ramByteUsageMax`, which are the maximum usages in the `window` for the Allocation. If the Allocation query is aggregated or accumulated, this object will be null because the meaning of maximum is ambiguous in these situations. Consider aggregating by namespace: should the maximum be the maximum of each Allocation individually, or the maximum combined usage of all Allocations (at any point in time in the `window`) in the namespace? |

## Quick start

Request allocation data for each 24-hour period in the last three days, aggregated by namespace:

{% tabs %}
{% tab title="Request" %}
```
$ curl http://localhost:9090/model/allocation \
  -d window=3d \
  -d aggregate=namespace \
  -d accumulate=false \
  -d shareIdle=false \
  -G
```
{% endtab %}

{% tab title="Response" %}
```
{
  "code": 200,
  "data": [
    {
      "__idle__": { ... },
      "default": { ... },
      "kube-system": { ... },
      "kubecost": { ... }
    },
    {
      "__idle__": { ... },
      "default": { ... },
      "kube-system": { ... },
      "kubecost": { ... }
    },
    {
      "__idle__": { ... },
      "default": { ... },
      "kube-system": { ... },
      "kubecost": { ... }
    },
    {
      "__idle__": { ... },
      "default": { ... },
      "kube-system": { ... },
      "kubecost": { ... }
    }
  ]
}
```
{% endtab %}
{% endtabs %}

> **Note**: Querying for `window=3d` will likely return a range of four sets because the queried range will overlap with four precomputed 24-hour sets, each aligned to the configured time zone. For example, querying `window=3d` on 2021/01/04T12:00:00 will return:

* 2021/01/04 00:00:00 until 2021/01/04T12:00:00 (now)
* 2021/01/03 00:00:00 until 2021/01/04 00:00:00
* 2021/01/02 00:00:00 until 2021/01/03 00:00:00
* 2021/01/01 00:00:00 until 2021/01/02 00:00:00

See [Querying](allocation.md#querying) for the full list of arguments and [Examples](allocation.md#query-examples) for more example queries.

## Special types of allocation

* `__idle__` refers to resources on a cluster that were not dedicated to a Kubernetes object (e.g. unused CPU core-hours on a node). An idle resource can be shared (proportionally or evenly) with the other allocations from the same cluster. (See the argument `shareIdle`.)
* `__unallocated__` refers to aggregated allocations without the selected `aggregate` field; e.g. aggregating by `label:app` might produce an `__unallocated__` allocation composed of allocations without the `app` label.
* `__unmounted__` (or "Unmounted PVs") refers to the resources used by PersistentVolumes that aren't mounted to a pod using a PVC, and thus cannot be allocated to a pod.

## Query examples

Allocation data for today unaggregated:

{% tabs %}
{% tab title="Request" %}
```
$ curl http://localhost:9090/model/allocation \
-d window=today \
-G
```
{% endtab %}

{% tab title="Response" %}
```
{
  "code": 200,
  "data": [
    {
      "__idle__": { ... },
      "cluster-one/gke-niko-pool-2-9182dfa7-okb2/kubecost/kubecost-cost-analyzer-94dc86fc-lwvrm/cost-model": { ... },
      "cluster-one/gke-niko-pool-2-9182dfa7-okb2/kubecost/kubecost-cost-analyzer-94dc86fc-lwvrm/cost-analyzer-frontend": { ... },
      "cluster-one/gke-niko-pool-2-9182dfa7-okb2/kubecost/kubecost-grafana-6df5cc66b6-dzszt/grafana": { ... }
    }
  ]
}
```
{% endtab %}
{% endtabs %}

Allocation data for last week, per day, aggregated by cluster:

{% tabs %}
{% tab title="Request" %}
```
$ curl http://localhost:9090/model/allocation \
  -d window=lastweek \
  -d aggregate=cluster \
  -G
```
{% endtab %}

{% tab title="Response" %}
```
{
  "code": 200,
  "data": [
    {
      "__idle__": { ... },
      "cluster-one": { ... },
      "cluster-two": { ... }
    },
    {
      "__idle__": { ... },
      "cluster-one": { ... },
      "cluster-two": { ... }
    },
    {
      "__idle__": { ... },
      "cluster-one": { ... },
      "cluster-two": { ... }
    },
    {
      "__idle__": { ... },
      "cluster-one": { ... },
      "cluster-two": { ... }
    },
    {
      "__idle__": { ... },
      "cluster-one": { ... },
      "cluster-two": { ... }
    },
    {
      "__idle__": { ... },
      "cluster-one": { ... },
      "cluster-two": { ... }
    },
    {
      "__idle__": { ... },
      "cluster-one": { ... },
      "cluster-two": { ... }
    }
  ]
}
```
{% endtab %}
{% endtabs %}

Allocation data for the last 30 days, aggregated by the "app" label, sharing idle allocation, sharing allocations from two namespaces, sharing $100/mo in overhead, and accumulated into one allocation for the entire window:

{% tabs %}
{% tab title="Request" %}
```
$ curl http://localhost:9090/model/allocation \
  -d window=30d \
  -d aggregate=label:app \
  -d accumulate=true \
  -d shareIdle=weighted \
  -d shareNamespaces=kube-system,kubecost \
  -d shareCost=100 \
  -G
```
{% endtab %}

{% tab title="Response" %}
```
{
  "code": 200,
  "data": [
    {
      "__unallocated__": { ... },
      "app=redis": { ... },
      "app=cost-analyzer": { ... },
      "app=prometheus": { ... },
      "app=grafana": { ... },
      "app=nginx": { ... },
      "app=helm": { ... }
    }
  ]
}
```
{% endtab %}
{% endtabs %}

Allocation data for 2021-03-10T00:00:00 to 2021-03-11T00:00:00 (i.e. 24h), multi-aggregated by namespace and the "app" label, filtering by `properties.cluster == "cluster-one"`, and accumulated into one allocation for the entire window.

{% tabs %}
{% tab title="Request" %}
```
$ curl http://localhost:9090/model/allocation \
  -d window=2021-03-10T00:00:00Z,2021-03-11T00:00:00Z \
  -d aggregate=namespace,label:app \
  -d accumulate=true \
  -d filterClusters=cluster-one \
  -G
```
{% endtab %}

{% tab title="Response" %}
```
{
  "code": 200,
  "data": [
    {
      "default/app=redis": { ... },
      "kubecost/app=cost-analyzer": { ... },
      "kubecost/app=prometheus": { ... },
      "kubecost/app=grafana": { ... },
      "kubecost/app=prometheus": { ... },
      "kube-system/app=helm": { ... }
    }
  ]
}
```
{% endtab %}
{% endtabs %}

Allocation data for today, aggregated by annotation. See [Enabling Annotation Emission](annotations.md) to enable annotations.

{% tabs %}
{% tab title="Request" %}
```
$ curl http://localhost:9090/model/allocation \
  -d window=today \
  -d aggregate=annotation:my_annotation \
  -G
```
{% endtab %}

{% tab title="Response" %}
```
{
  "code": 200,
  "data": [
    {
      "__unallocated__": { ... },
      "my_annotation=foo": { ... },
      "my_annotation=bar": { ... }
    }
  ]
}
```
{% endtab %}
{% endtabs %}

## Allocation of asset costs

Both the `reconcile` and `shareTenancyCosts` flags start processes that distribute the costs of Assets to Allocations related to them. For the `reconcile` flag, these connections can be straightforward like the connection between a node Asset and an Allocation where the CPU, GPU, and RAM usage can be used to distribute a proportion of the node's cost to the Allocations that run on it. For Assets and Allocations where the connection is less well-defined, such as network Assets we have opted for a method of distributing the cost that we call Distribution by Usage Hours.

Distribution by Usage Hours takes the usage of the windows (start time and end time) of an Asset and all the Allocations connected to it and finds the number of hours that both the Allocation and Asset were running. The number of hours for each Allocation related to an Asset is called `Alloc_Usage_Hours`. The sum of all `Alloc_Usage_Hours` for a single Assets is `Total_Usage_Hours`. With these values, an Assets cost is distributed to each connected Allocation using the formula `Asset_Cos`t \* `Alloc_Usage_Hours/Total_Usage_Hours`. Depending on the Asset type an Allocation can receive proportions of multiple Asset Costs.

Asset types that use this distribution method include:

* Network (`reconcile`): When the network pod is not enabled cost is distributed by usage hours. If the network pod is enabled cost is distributed to Allocations proportionally to usage.
* Load Balancer (`reconcile`)
* Cluster Management (`shareTenancyCosts`)
* Attached disks (`shareTenancyCosts`): Does not include PVs, which are handled by `reconcile`

## Querying on-demand (experimental)

> **Warning:** Querying on-demand with high resolution for long windows can cause serious Prometheus performance issues, including OOM errors. Start with short windows (`1d` or less) and proceed with caution.

Computing allocation data on-demand allows for greater flexibility with respect to step size and accuracy-versus-performance. (See `resolution` and [error bounds](allocation.md#theoretical-error-bounds) for details.) Unlike the standard endpoint, which can only serve results from precomputed sets with predefined step sizes (e.g. 24h aligned to the UTC time zone), asking for a "7d" query will almost certainly result in 8 sets, including "today" and the final set, which might span 6.5d-7.5d ago. With this endpoint, however, you will be computing everything on-demand, so "7d" will return exactly seven days of data, starting at the moment the query is received. (You can still use window keywords like "today" and "lastweek", of course, which should align perfectly with the same queries of the standard ETL-driven endpoint.)

{% swagger method="get" path="/allocation/compute" baseUrl="http://<kubecost>/model" summary="Allocation On-Demand API" %}
{% swagger-description %}

{% endswagger-description %}

{% swagger-parameter in="path" name="window" type="string" required="true" %}
Duration of time over which to query. Accepts words like `today`, `week`, `month`, `yesterday`, `lastweek`, `lastmonth`; durations like `30m`, `12h`, `7d`; RFC3339 date pairs like

`2021-01-02T15:04:05Z,2021-02-02T15:04:05Z`; Unix timestamps like `1578002645,1580681045`.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="resolution" type="string" required="false" %}
Duration to use as resolution in Prometheus queries. Smaller values (i.e. higher resolutions) will provide better accuracy, but worse performance (i.e. slower query time, higher memory use). Larger values (i.e. lower resolutions) will perform better, but at the expense of lower accuracy for short-running workloads. See 

[error bounds](allocation.md#theoretical-error-bounds)

 for details. Default is 

`1m`

.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="step" type="string" required="false" %}
Duration of a single allocation set. If unspecified, this defaults to the

`window`, so that you receive exactly one set for the entire window. If specified, it works chronologically backward, querying in durations of `step` until the full window is covered.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="aggregate" type="string" required="false" %}
Field by which to aggregate the results. Accepts: 

`cluster`

, 

`namespace`

, 

`controllerKind`

, 

`controller`

, 

`service`

, 

`label:<name>`

, and 

`annotation:<name>`

. Also accepts comma-separated lists for multi-aggregation, like 

`namespace,label:app`

.
{% endswagger-parameter %}

{% swagger-parameter in="path" name="accumulate" type="boolean" required="false" %}
If 

`true`

, sum the entire range of sets into a single set. Default value is 

`false`

.
{% endswagger-parameter %}

{% swagger-response status="200: OK" description="" %}
```javascript
{
                "name": "cluster-one//integration/integration-unmounted-pvcs/__unmounted__",
                "properties": {
                    "cluster": "cluster-one",
                    "container": "__unmounted__",
                    "namespace": "integration",
                    "pod": "integration-unmounted-pvcs"
                },
                "window": {
                    "start": "2023-01-17T15:00:00Z",
                    "end": "2023-01-17T16:00:00Z"
                },
                "start": "2023-01-17T15:00:00Z",
                "end": "2023-01-17T16:00:00Z",
                "minutes": 60.000000,
                "cpuCores": 0.000000,
                "cpuCoreRequestAverage": 0.000000,
                "cpuCoreUsageAverage": 0.000000,
                "cpuCoreHours": 0.000000,
                "cpuCost": 0.000000,
                "cpuCostAdjustment": 0.000000,
                "cpuEfficiency": 0.000000,
                "gpuCount": 0.000000,
                "gpuHours": 0.000000,
                "gpuCost": 0.000000,
                "gpuCostAdjustment": 0.000000,
                "networkTransferBytes": 0.000000,
                "networkReceiveBytes": 0.000000,
                "networkCost": 0.000000,
                "networkCrossZoneCost": 0.000000,
                "networkCrossRegionCost": 0.000000,
                "networkInternetCost": 0.000000,
                "networkCostAdjustment": 0.000000,
                "loadBalancerCost": 0.000000,
                "loadBalancerCostAdjustment": 0.000000,
                "pvBytes": 0.000000,
                "pvByteHours": 0.000000,
                "pvCost": 0.000003,
                "pvs": {
                    "cluster=cluster-one:name=pvc-demo": {
                        "byteHours": 0.000000,
                        "cost": 0.000000
                },
                "pvCostAdjustment": 0.000000,
                "ramBytes": 0.000000,
                "ramByteRequestAverage": 0.000000,
                "ramByteUsageAverage": 0.000000,
                "ramByteHours": 0.000000,
                "ramCost": 0.000000,
                "ramCostAdjustment": 0.000000,
                "ramEfficiency": 0.000000,
                "sharedCost": 0.000000,
                "externalCost": 0.000000,
                "totalCost": 0.000000,
                "totalEfficiency": 0.000000,
                "rawAllocationOnly": null
}
```
{% endswagger-response %}
{% endswagger %}

### On-demand query examples

Allocation data for the last 60m, in steps of 10m, with resolution 1m, aggregated by cluster.

{% tabs %}
{% tab title="Request" %}
```
$ curl http://localhost:9090/model/allocation/compute \
  -d window=60m \
  -d step=10m \
  -d resolution=1m \
  -d aggregate=cluster \
  -d accumulate=false \
  -G
```
{% endtab %}

{% tab title="Response" %}
```
{
  "code": 200,
  "data": [
    {
      "cluster-one": { ... },
      "cluster-two": { ... }
    },
    {
      "cluster-one": { ... },
      "cluster-two": { ... }
    },
    {
      "cluster-one": { ... },
      "cluster-two": { ... }
    }
  ]
}
```
{% endtab %}
{% endtabs %}

Allocation data for the last 9d, in steps of 3d, with a 10m resolution, aggregated by namespace.

{% tabs %}
{% tab title="Request" %}
```
$ curl http://localhost:9090/model/allocation/compute \
  -d window=9d \
  -d step=3d \
  -d resolution=10m
  -d aggregate=namespace \
  -d accumulate=false \
  -G
```
{% endtab %}

{% tab title="Response" %}
```
{
  "code": 200,
  "data": [
    {
      "default": { ... },
      "kubecost": { ... },
      "kube-system": { ... }
    },
    {
      "default": { ... },
      "kubecost": { ... },
      "kube-system": { ... }
    },
    {
      "default": { ... },
      "kubecost": { ... },
      "kube-system": { ... }
    }
  ]
}
```
{% endtab %}
{% endtabs %}

### Theoretical error bounds

Tuning the resolution parameter allows the querier to make tradeoffs between accuracy and performance. For long-running pods (>1d) resolution can be tuned aggressively low (>10m) with relatively little effect on accuracy. However, even modestly low resolutions (5m) can result in significant accuracy degradation for short-running pods (<1h).

Here, we provide theoretical error bounds for different resolution values given pods of differing running durations. The tuple represents lower- and upper-bounds for accuracy as a percentage of the actual value. For example:

* 1.00, 1.00 means that results should always be accurate to less than 0.5% error
* 0.83, 1.00 means that results should never be high by more than 0.5% error, but could be low by as much as 17% error
* \-1.00, 10.00 means that the result could be as high as 1000% error (e.g. 30s pod being counted for 5m) or the pod could be missed altogether, i.e. -100% error.

| resolution |    30s pod    |    5m pod    |    1h pod   |   1d pod   |   7d pod   |
| ---------: | :-----------: | :----------: | :---------: | :--------: | :--------: |
|         1m |  -1.00, 2.00  |  0.80, 1.00  |  0.98, 1.00 | 1.00, 1.00 | 1.00, 1.00 |
|         2m |  -1.00, 4.00  |  0.80, 1.20  |  0.97, 1.00 | 1.00, 1.00 | 1.00, 1.00 |
|         5m |  -1.00, 10.00 |  -1.00, 1.00 |  0.92, 1.00 | 1.00, 1.00 | 1.00, 1.00 |
|        10m |  -1.00, 20.00 |  -1.00, 2.00 |  0.83, 1.00 | 0.99, 1.00 | 1.00, 1.00 |
|        30m |  -1.00, 60.00 |  -1.00, 6.00 |  0.50, 1.00 | 0.98, 1.00 | 1.00, 1.00 |
|        60m | -1.00, 120.00 | -1.00, 12.00 | -1.00, 1.00 | 0.96, 1.00 | 0.99, 1.00 |
