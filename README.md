# Zabbix OpenShift Monitoring Template

![Zabbix](https://img.shields.io/badge/Zabbix-7.0-red)
![OpenShift](https://img.shields.io/badge/OpenShift-4.x-red)
![Thanos](https://img.shields.io/badge/Thanos-Query%20API-blue)
![Status](https://img.shields.io/badge/Status-Active-brightgreen)

# Template OpenShift Cluster by Thanos API

Template for monitoring OpenShift clusters through the Thanos Query API.

This template uses PromQL-based HTTP agent items, dependent items and LLD rules to detect problematic or critical OpenShift resources. It is designed to reduce HTTP requests, limit unnecessary object discovery and focus monitoring on actionable conditions.

Author: Matteo Z.

## Overview

This Zabbix template monitors OpenShift clusters through the **Thanos Query API** using HTTP agent master items, PromQL queries, low-level discovery rules and dependent item prototypes.

The template follows a **critical/problem-based discovery** approach: instead of discovering every object in the cluster, most discovery rules return only resources that are currently problematic or above a configured threshold. This helps reduce the number of created items and limits unnecessary load on the Zabbix Server, Zabbix Proxy and database.

## Main goals

- Monitor OpenShift cluster health through Thanos/Prometheus metrics.
- Reduce the number of HTTP requests by grouping related checks into shared raw items.
- Reduce the number of discovered objects by using PromQL queries that return only critical/problematic resources.
- Avoid JavaScript preprocessing where possible.
- Use PromQL for calculations and JSONPath for value extraction.
- Keep item history/trends low for dynamic or problem-based resources.

## Requirements

- Zabbix 7.0 or later.
- Access from Zabbix Server or Proxy to the Thanos Querier endpoint.
- A valid bearer token with permission to query OpenShift monitoring metrics.
- OpenShift monitoring stack exposing the required metrics through Thanos.

## Template metadata

| Field             | Value                                      |
| ----------------- | ------------------------------------------ |
| Template name     | `Template OpenShift Cluster by Thanos API` |
| Template group    | `Templates/Applications`                   |
| UUID              | `9c7a805f5a4f4a86ad26d73c89482ab9`         |
| Main data source  | Thanos Query API                           |
| Collection method | Zabbix HTTP agent                          |
| Discovery model   | Critical/problem-based LLD                 |

## Template tags

- `datasource = thanos`
- `platform = openshift`
- `target = critical-object`

## Required macros

At minimum, configure these macros on the host representing the OpenShift cluster:

| Macro               | Description                                                                                         |
| ------------------- | --------------------------------------------------------------------------------------------------- |
| `{$OCP.THANOS.URL}` | Base URL of the Thanos Querier endpoint. Example: `https://thanos-querier.example.com`              |
| `{$OCP.TOKEN}`      | Bearer token used by Zabbix HTTP agent items to query Thanos. This is configured as a secret macro. |

## Template macros

| Macro                                   |                                 Default | Description                                                                                                  |
| --------------------------------------- | --------------------------------------: | ------------------------------------------------------------------------------------------------------------ |
| `{$OCP.CO.PROGRESSING.MAX.TIME}`        |                                  `1800` | Maximum time, in seconds, a ClusterOperator can remain in `Progressing=True` before triggering an alert.     |
| `{$OCP.CRONJOB.MAX.LAST.SUCCESS}`       |                                 `86400` | Maximum allowed time, in seconds, since the last successful CronJob run.                                     |
| `{$OCP.CRONJOB.NS.MATCHES}`             |                                    `^$` | Regex used to include namespaces in the critical CronJob discovery. Default `^$` disables CronJob discovery. |
| `{$OCP.ETCD.LATENCY.P99.WARN}`          |                                     `1` | Warning threshold in seconds for etcd p99 latency.                                                           |
| `{$OCP.JOB.MAX.RUN.TIME}`               |                                  `1800` | Number of seconds after which an active Job is considered long-running.                                      |
| `{$OCP.JOB.NS.MATCHES}`                 |                                    `.*` | Regex for the namespaces in which to enable critical Job discovery.                                          |
| `{$OCP.NODE.FS.CRIT}`                   |                                    `90` | Critical threshold for node filesystem usage, expressed as a percentage.                                     |
| `{$OCP.NODE.FS.DEVICE.NOT_MATCHES}`     | See filesystem exclusion defaults below | Devices to exclude from node filesystem queries.                                                             |
| `{$OCP.NODE.FS.FSTYPE.NOT_MATCHES}`     | See filesystem exclusion defaults below | Filesystem types to exclude from node filesystem queries.                                                    |
| `{$OCP.NODE.FS.MOUNTPOINT.NOT_MATCHES}` | See filesystem exclusion defaults below | Mount points to exclude from node filesystem queries.                                                        |
| `{$OCP.NODE.FS.WARN}`                   |                                    `75` | Warning threshold for node filesystem usage, expressed as a percentage.                                      |
| `{$OCP.NS.NOT_MATCHES}`                 |                                    `^$` | Global regex to exclude namespaces from problem-based discovery. `^$` excludes nothing.                      |
| `{$OCP.POD.NS.MATCHES}`                 |                                    `.*` | Regex for the namespaces in which to enable critical Pod discovery.                                          |
| `{$OCP.POD.RESTARTS.WARN}`              |                                     `3` | Restart threshold over 15 minutes for generating critical Pod objects.                                       |
| `{$OCP.PVC.CRIT}`                       |                                    `90` | Critical threshold for PVC usage, expressed as a percentage.                                                 |
| `{$OCP.PVC.NS.MATCHES}`                 |                                    `.*` | Regex for the namespaces in which to enable critical PVC discovery.                                          |
| `{$OCP.PVC.WARN}`                       |                                    `75` | Warning threshold for PVC usage, expressed as a percentage.                                                  |
| `{$OCP.ROUTE.NS.MATCHES}`               |                                    `.*` | Regex for the namespaces in which to enable critical Route discovery.                                        |
| `{$OCP.RQ.CRIT}`                        |                                    `90` | Critical threshold for ResourceQuota usage, expressed as a percentage.                                       |
| `{$OCP.RQ.NS.MATCHES}`                  |                                    `.*` | Regex for the namespaces in which to enable critical ResourceQuota discovery.                                |
| `{$OCP.RQ.RESOURCE.MATCHES}`            |                                    `.*` | Regex for the ResourceQuota resources to include.                                                            |
| `{$OCP.RQ.RESOURCE.NOT_MATCHES}`        |                                    `^$` | Regex for ResourceQuota resources to exclude. `^$` excludes nothing.                                         |
| `{$OCP.RQ.WARN}`                        |                                    `75` | Warning threshold for ResourceQuota usage, expressed as a percentage.                                        |
| `{$OCP.SVC.NOT_MATCHES}`                |                                    `^$` | Regex to exclude services/endpoints from Service discovery. `^$` excludes nothing.                           |
| `{$OCP.SVC.NS.MATCHES}`                 |                                    `.*` | Regex for the namespaces in which to enable critical Service endpoint discovery.                             |
| `{$OCP.THANOS.URL}`                     |    `https://thanos-querier.example.com` | Thanos Querier base URL.                                                                                     |
| `{$OCP.TOKEN}`                          |                           `SECRET_TEXT` | Bearer token for querying the Thanos/OpenShift monitoring API.                                               |

### Node filesystem exclusion macros

The following macros define the default regex filters used to exclude devices, filesystem types and mount points from node filesystem monitoring.

```text
{$OCP.NODE.FS.DEVICE.NOT_MATCHES}=^(rootfs)$

{$OCP.NODE.FS.FSTYPE.NOT_MATCHES}=^(tmpfs|overlay|squashfs|nsfs|proc|sysfs|devtmpfs|cgroup2?|mqueue|hugetlbfs|securityfs|debugfs|tracefs)$

{$OCP.NODE.FS.MOUNTPOINT.NOT_MATCHES}=^/run($|/.*)|^/var/lib/kubelet/pods/.*|^/var/lib/containers/storage/.*|^/proc($|/.*)|^/sys($|/.*)|^/dev($|/.*)
```

## Value mappings

### OCP status: Healthy / Unhealthy

- `0` → `Unhealthy`
- `1` → `Healthy`

### OCP status: OK / Problem

- `0` → `OK`
- `1` → `Problem`

### OCP status: Up / Down

- `0` → `Down`
- `1` → `Up`

### OCP status: Yes / No

- `0` → `No`
- `1` → `Yes`

## Raw HTTP agent items

These are the master items that execute PromQL queries against Thanos. Most dependent item prototypes are based on these raw JSON responses.

| Item                                       | Purpose                                                             |
| ------------------------------------------ | ------------------------------------------------------------------- |
| OCP: Critical ClusterOperators raw         | Detect problematic ClusterOperators.                                |
| OCP: Critical Jobs status raw              | Detect failed or long-running Jobs.                                 |
| OCP: Critical Node filesystems raw         | Detect node filesystems above warning threshold.                    |
| OCP: Critical Nodes status raw             | Detect nodes that are not ready or unschedulable.                   |
| OCP: Critical Pods status raw              | Detect problematic pods and pods with excessive restarts.           |
| OCP: Critical PV status raw                | Detect failed or terminating PersistentVolumes.                     |
| OCP: Critical PVC usage raw                | Detect PVCs above warning threshold.                                |
| OCP: Critical ResourceQuota usage raw      | Detect ResourceQuota resources above warning threshold.             |
| OCP: Critical Routes status raw            | Detect routes that are not admitted.                                |
| OCP: Critical Service endpoints status raw | Detect services with no available endpoints or not-ready endpoints. |
| OCP: Etcd members status raw               | Collect etcd member target-up and leader status.                    |
| OCP: Etcd request latency p99              | Collect etcd request p99 latency.                                   |
| OCP: Thanos query health                   | Check that Thanos query endpoint returns a valid response.          |

## PromQL queries used by raw items

### OCP: Critical ClusterOperators raw

```promql
(cluster_operator_conditions{condition="Available",status!="True"} == 1) or (cluster_operator_conditions{condition=~"Degraded|Progressing",status="True"} == 1)
```

### OCP: Critical Jobs status raw

```promql
label_replace((kube_job_status_failed>0) or (((time()-kube_job_status_start_time) and on(namespace,job_name) (kube_job_status_active>0))>={$OCP.JOB.MAX.RUN.TIME}),"check","any","job_name",".*") or label_replace(kube_job_status_failed>0,"check","failed","job_name",".*") or label_replace(((time()-kube_job_status_start_time) and on(namespace,job_name) (kube_job_status_active>0))>={$OCP.JOB.MAX.RUN.TIME},"check","longrunning","job_name",".*")
```

### OCP: Critical Node filesystems raw

```promql
100 * (1 - node_filesystem_avail_bytes/node_filesystem_size_bytes) >= {$OCP.NODE.FS.WARN}
```

### OCP: Critical Nodes status raw

```promql
label_replace(max by (node) ((kube_node_status_condition{condition="Ready",status=~"false|unknown"} == 1) or (kube_node_spec_unschedulable == 1)), "check", "any", "node", ".*") or label_replace(max by (node) (kube_node_status_condition{condition="Ready",status=~"false|unknown"} == 1), "check", "not_ready", "node", ".*") or label_replace(max by (node) (kube_node_spec_unschedulable == 1), "check", "unschedulable", "node", ".*")
```

### OCP: Critical Pods status raw

```promql
label_replace((max by(namespace,pod)((kube_pod_status_phase{phase=~"Pending|Failed|Unknown"}==1) or (kube_pod_status_unschedulable==1) or (kube_pod_status_ready{condition="false"}==1) or (kube_pod_container_status_waiting_reason{reason="CrashLoopBackOff"}==1))>0) or (sum by(namespace,pod)(increase(kube_pod_container_status_restarts_total[15m]))>={$OCP.POD.RESTARTS.WARN}),"check","any","pod",".*") or label_replace(max by(namespace,pod)((kube_pod_status_phase{phase=~"Pending|Failed|Unknown"}==1) or (kube_pod_status_unschedulable==1) or (kube_pod_status_ready{condition="false"}==1) or (kube_pod_container_status_waiting_reason{reason="CrashLoopBackOff"}==1))>0,"check","problem_state","pod",".*") or label_replace(sum by(namespace,pod)(increase(kube_pod_container_status_restarts_total[15m]))>={$OCP.POD.RESTARTS.WARN},"check","restarts","pod",".*")
```

### OCP: Critical PV status raw

```promql
label_replace((kube_persistentvolume_status_phase{phase="Failed"}==1) or (kube_persistentvolume_deletion_timestamp>0),"check","any","persistentvolume",".*") or label_replace(kube_persistentvolume_status_phase{phase="Failed"}==1,"check","failed","persistentvolume",".*") or label_replace(kube_persistentvolume_deletion_timestamp>0,"check","terminating","persistentvolume",".*")
```

### OCP: Critical PVC usage raw

```promql
100 * (kubelet_volume_stats_used_bytes / kubelet_volume_stats_capacity_bytes) >= {$OCP.PVC.WARN}
```

### OCP: Critical ResourceQuota usage raw

```promql
((100 * kube_resourcequota{type="used"} / on(namespace,resourcequota,resource) kube_resourcequota{type="hard"}) and on(namespace,resourcequota,resource) (kube_resourcequota{type="hard"} > 0)) >= {$OCP.RQ.WARN}
```

### OCP: Critical Routes status raw

```promql
((openshift_route_status{type="Admitted",status="True"} == bool 0) == 1)
```

### OCP: Critical Service endpoints status raw

```promql
label_replace((kube_endpoint_info unless on(namespace,endpoint)(sum by(namespace,endpoint)(kube_endpoint_address_available)>0)) or (sum by(namespace,endpoint)(kube_endpoint_address_not_ready)>0),"check","any","endpoint",".*") or label_replace(kube_endpoint_info unless on(namespace,endpoint)(sum by(namespace,endpoint)(kube_endpoint_address_available)>0),"check","no_available","endpoint",".*") or label_replace(sum by(namespace,endpoint)(kube_endpoint_address_not_ready)>0,"check","not_ready","endpoint",".*")
```

### OCP: Etcd members status raw

```promql
label_replace(up{job=~".*etcd.*"},"check","target_up","instance",".*") or label_replace(etcd_server_has_leader,"check","has_leader","instance",".*")
```

### OCP: Etcd request latency p99

```promql
histogram_quantile(0.99, sum(rate(etcd_request_duration_seconds_bucket[10m])) by (le)) or vector(0)
```

### OCP: Thanos query health

```promql
vector(1)
```

## Monitored areas

### ClusterOperators

The template detects ClusterOperators that are not available, degraded or progressing. The discovery creates one item per problematic ClusterOperator condition.

The ClusterOperator item uses the value map `OCP status: OK / Problem`.

### Nodes

The template discovers nodes only when they are in one of the critical states returned by the node status query:

- `Not ready`
- `Unschedulable`

Node filesystem monitoring is also problem-based. Filesystems are discovered only when their usage exceeds `{$OCP.NODE.FS.WARN}`. Two trigger levels are then applied:

- Warning: `{$OCP.NODE.FS.WARN}`
- Critical: `{$OCP.NODE.FS.CRIT}`

### etcd

The template monitors etcd in two ways:

- Member-level status through the `OCP: Etcd members status raw` item.
- Aggregate p99 request latency through the `OCP: Etcd request latency p99` item.

The etcd member discovery creates items for:

- `Target up`
- `Has leader`

### Jobs

The Jobs discovery is merged into a single rule. It discovers Jobs when they are either:

- Failed.
- Running longer than `{$OCP.JOB.MAX.RUN.TIME}`.

### Pods

The Pod status discovery is merged into a single rule. It discovers Pods when they have either:

- A problem state such as Pending, Failed, Unknown, Unschedulable, NotReady or CrashLoopBackOff.
- Restart count in the last 15 minutes greater than or equal to `{$OCP.POD.RESTARTS.WARN}`.

### PersistentVolumes

The PV status discovery is merged into a single rule. It discovers PVs that are:

- Failed.
- Terminating.

### PersistentVolumeClaims

PVC discovery is problem-based and creates items only for PVCs with used percentage greater than or equal to `{$OCP.PVC.WARN}`. Two trigger levels are configured:

- Warning: `{$OCP.PVC.WARN}`
- Critical: `{$OCP.PVC.CRIT}`

### ResourceQuota

ResourceQuota discovery is problem-based and creates items only for quota resources with usage greater than or equal to `{$OCP.RQ.WARN}`. Two trigger levels are configured:

- Warning: `{$OCP.RQ.WARN}`
- Critical: `{$OCP.RQ.CRIT}`

### Routes

The template discovers routes that are not admitted.

### Services

Service endpoint discovery is merged into a single rule. It discovers services when:

- There are no available endpoints.
- There are one or more not-ready endpoints.

## Critical/problem-based discovery behavior

Most discovery rules are intentionally based on queries that return only objects in a problematic state. This means:

- Objects in a healthy state are not discovered.
- Item prototypes are created only when a problem exists.
- When the object returns to a healthy state, it disappears from the PromQL result.
- The dependent item fallback value is set to `0` where configured.
- The object is removed after the discovery rule's keep-lost period.

This design reduces item count significantly, especially in large OpenShift clusters.

## Important operational notes

### Warning and critical thresholds

For checks that have both warning and critical trigger levels, the PromQL discovery query must use the lower threshold. For example:

- PVC discovery should use `{$OCP.PVC.WARN}`, not `{$OCP.PVC.CRIT}`.
- Node filesystem discovery should use `{$OCP.NODE.FS.WARN}`, not `{$OCP.NODE.FS.CRIT}`.
- ResourceQuota discovery should use `{$OCP.RQ.WARN}`, not `{$OCP.RQ.CRIT}`.

Otherwise, Zabbix would not create the item when the resource is only in warning state.

### Namespace filtering

Namespace filtering is controlled by macros such as:

- `{$OCP.POD.NS.MATCHES}`
- `{$OCP.JOB.NS.MATCHES}`
- `{$OCP.PVC.NS.MATCHES}`
- `{$OCP.ROUTE.NS.MATCHES}`
- `{$OCP.SVC.NS.MATCHES}`
- `{$OCP.RQ.NS.MATCHES}`
- `{$OCP.NS.NOT_MATCHES}`

Use these macros at host level to restrict discovery to specific namespaces.

Example:

```text
{$OCP.POD.NS.MATCHES} = ^(app-prod|batch-prod)$
{$OCP.NS.NOT_MATCHES} = ^(openshift-|kube-).*
```

### History and trends

The template keeps raw items with `History = 0` and `Trends = 0`. Dynamic problem-based item prototypes generally use short history and no trends to reduce database load.

## Testing Thanos access

Use this test from the Zabbix Server or Proxy:

```bash
curl -ks -H "Authorization: Bearer ${TOKEN}" --get \
  --data-urlencode 'query=vector(1)' \
  "${THANOS}/api/v1/query" | jq
```

Expected result:

```json
{
  "status": "success",
  "data": {
    "resultType": "vector",
    "result": [
      {
        "metric": {},
        "value": [ ... , "1" ]
      }
    ]
  }
}
```
