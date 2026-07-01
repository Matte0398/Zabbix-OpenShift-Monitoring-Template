# Template OpenShift Cluster by Thanos API

![Zabbix](https://img.shields.io/badge/Zabbix-7.0-red)
![OpenShift](https://img.shields.io/badge/OpenShift-4.x-red)
![Thanos](https://img.shields.io/badge/Thanos-Query%20API-blue)
![Status](https://img.shields.io/badge/Status-Active-brightgreen)

## Overview

This template is intended for environments where OpenShift cluster state is queried through the **Thanos Querier / Prometheus API** and collected by Zabbix using HTTP agent items.

The template is mainly **problem-oriented**: most low-level discovery rules are based on PromQL queries that return only resources matching warning or critical conditions. This reduces the amount of discovered objects, item count and HTTP calls compared with discovering all OpenShift resources.

Some checks are intentionally always evaluated, such as the Thanos query health check and the aggregate etcd latency item, because they validate the monitoring path or provide cluster-level control-plane indicators.

---

## Requirements

- Zabbix **7.0+**
- OpenShift cluster with monitoring stack enabled
- Reachable Thanos Querier endpoint
- Bearer token with permission to query metrics from the OpenShift monitoring API
- Metrics exposed by:

  - `cluster_operator_conditions`
  - `kube-state-metrics`
  - `openshift-state-metrics`
  - node exporter / kubelet volume metrics
  - etcd metrics

## Template import

1. Go to **Data collection → Templates**
2. Click **Import**
3. Upload `template_openshift.yaml`
4. Import the template
5. Link the template to the Zabbix host representing the OpenShift cluster
6. Configure the required macros on the template or host

## Required host macros

At minimum, configure these macros:

| Macro               | Description                                                                                           |
| ------------------- | ----------------------------------------------------------------------------------------------------- |
| `{$OCP.THANOS.URL}` | Base URL of the Thanos Querier endpoint, for example `https://thanos-querier.example.com`.            |
| `{$OCP.TOKEN}`      | Bearer token used to query the Thanos/OpenShift monitoring API. This macro is defined as secret text. |

Example:

```text
{$OCP.THANOS.URL}=https://thanos-querier-openshift-monitoring.apps.example.com
{$OCP.TOKEN}=<bearer_token>
```

## Other macros

| Macro                                   | Default value                        | Description                                                                                            |
| --------------------------------------- | ------------------------------------ | ------------------------------------------------------------------------------------------------------ |
| `{$OCP.CO.PROGRESSING.MAX.TIME}`        | `1800`                               | Maximum time, in seconds, a ClusterOperator can remain in Progressing=True before triggering an alert. |
| `{$OCP.ETCD.LATENCY.P99.WARN}`          | `1`                                  | Warning threshold in seconds for etcd p99 latency.                                                     |
| `{$OCP.JOB.MAX.RUN.TIME}`               | `1800`                               | The number of seconds after which an active job is considered long-running.                            |
| `{$OCP.JOB.NS.MATCHES}`                 | `.*`                                 | Regex for the namespaces in which to enable critical job discovery.                                    |
| `{$OCP.NODE.FS.CRIT}`                   | `90`                                 | Critical threshold for file system node usage, expressed as a percentage.                              |
| `{$OCP.NODE.FS.DEVICE.NOT_MATCHES}`     | `See below`                          | Devices to exclude from filesystem node queries.                                                       |
| `{$OCP.NODE.FS.FSTYPE.NOT_MATCHES}`     | `See below`                          | File system types to exclude from node file system queries.                                            |
| `{$OCP.NODE.FS.MOUNTPOINT.NOT_MATCHES}` | `See below`                          | Mount points to exclude from filesystem node queries.                                                  |
| `{$OCP.NODE.FS.WARN}`                   | `75`                                 | Warning threshold for file system node usage, expressed as a percentage.                               |
| `{$OCP.NS.NOT_MATCHES}`                 | `^$`                                 | Global regex to exclude namespaces from problem-based discovery. ^$ = excludes nothing.                |
| `{$OCP.POD.NS.MATCHES}`                 | `.*`                                 | Regex for the namespaces in which to enable critical Pod discovery.                                    |
| `{$OCP.POD.RESTARTS.WARN}`              | `3`                                  | Restart threshold set to 15 minutes for generating critical Pod objects.                               |
| `{$OCP.PVC.CRIT}`                       | `90`                                 | Critical threshold for PVC use, expressed as a percentage.                                             |
| `{$OCP.PVC.NS.MATCHES}`                 | `.*`                                 | Regex for the namespaces in which to enable critical PVC discovery.                                    |
| `{$OCP.PVC.WARN}`                       | `75`                                 | Warning threshold for PVC use, expressed as a percentage.                                              |
| `{$OCP.ROUTE.NS.MATCHES}`               | `.*`                                 | Regex for the namespaces in which to enable critical route discovery.                                  |
| `{$OCP.RQ.CRIT}`                        | `90`                                 | ResourceQuota usage critical threshold, expressed as a percentage.                                     |
| `{$OCP.RQ.NS.MATCHES}`                  | `.*`                                 | Regex for the namespaces in which to enable critical ResourceQuota discovery.                          |
| `{$OCP.RQ.RESOURCE.MATCHES}`            | `.*`                                 | Regular expression for the ResourceQuota resources to include.                                         |
| `{$OCP.RQ.RESOURCE.NOT_MATCHES}`        | `^$`                                 | Regular expression for ResourceQuota resources to exclude. ^$ = excludes nothing.                      |
| `{$OCP.RQ.WARN}`                        | `75`                                 | ResourceQuota usage warning threshold, expressed as a percentage.                                      |
| `{$OCP.SVC.NOT_MATCHES}`                | `^$`                                 | Regex to exclude services/endpoints from Service discovery. ^$ = excludes nothing.                     |
| `{$OCP.SVC.NS.MATCHES}`                 | `.*`                                 | Regex for the namespaces in which to enable critical discovery of service endpoints.                   |
| `{$OCP.THANOS.URL}`                     | `https://thanos-querier.example.com` | Thanos Querier base URL.                                                                               |
| `{$OCP.TOKEN}`                          | `SECRET_TEXT`                        | Bearer token for querying the Thanos/OpenShift monitoring API.                                         |

### Node filesystem exclusion macros

The following macros control filesystem discovery exclusions:

```text
{$OCP.NODE.FS.DEVICE.NOT_MATCHES}=^(rootfs)$

{$OCP.NODE.FS.FSTYPE.NOT_MATCHES}=^(tmpfs|overlay|squashfs|nsfs|proc|sysfs|devtmpfs|cgroup2?|mqueue|hugetlbfs|securityfs|debugfs|tracefs)$

{$OCP.NODE.FS.MOUNTPOINT.NOT_MATCHES}=^/run($|/.*)|^/var/lib/kubelet/pods/.*|^/var/lib/containers/storage/.*|^/proc($|/.*)|^/sys($|/.*)|^/dev($|/.*)
```

## How data collection works

The template uses HTTP agent master items to query:

```text
{$OCP.THANOS.URL}/api/v1/query
```

Each raw item sends a PromQL query through the `query` request parameter and receives a JSON response from the Prometheus-compatible API.

The discoveries are dependent items based on raw master items. They extract only the metrics returned by problem-oriented PromQL queries and create item prototypes only for resources currently requiring attention.

## Raw items

| Raw item                                     | Key                                          | Description                                                                                                        |
| -------------------------------------------- | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `OCP: Critical ClusterOperators raw`         | `ocp.critical.clusteroperators.raw`          | Collects ClusterOperator conditions that indicate a problematic state.                                             |
| `OCP: Critical Etcd members status raw`      | `ocp.critical.etcd.members.status.raw`       | Collects etcd members matching a critical condition.                                                               |
| `OCP: Critical Jobs status raw`              | `ocp.critical.jobs.status.raw`               | Collects Jobs that are either failed or running longer than the configured threshold.                              |
| `OCP: Critical Node filesystems raw`         | `ocp.critical.node.filesystems.raw`          | Collects node filesystems whose usage is above the configured warning threshold.                                   |
| `OCP: Critical Nodes status raw`             | `ocp.critical.nodes.status.raw`              | Collects nodes that are not ready or marked as unschedulable.                                                      |
| `OCP: Critical Pod CrashLoopBackOff raw`     | `ocp.critical.pods.crashloopbackoff.raw`     | Collects Pods with containers currently in CrashLoopBackOff state.                                                 |
| `OCP: Critical Pod problem state raw`        | `ocp.critical.pods.problem_state.raw`        | Collects Pods in problematic states.                                                                               |
| `OCP: Critical Pod restarts raw`             | `ocp.critical.pods.restarts.raw`             | Collects Pods whose container restart count increased above the configured threshold during the evaluation window. |
| `OCP: Critical PV status raw`                | `ocp.critical.pv.status.raw`                 | Collects PersistentVolumes in failed or terminating state.                                                         |
| `OCP: Critical PVC usage raw`                | `ocp.critical.pvc.usage.raw`                 | Collects PersistentVolumeClaims whose used capacity is above the configured warning threshold.                     |
| `OCP: Critical ResourceQuota usage raw`      | `ocp.critical.resourcequota.usage.raw`       | Collects ResourceQuota resources whose usage percentage is above the configured warning threshold.                 |
| `OCP: Critical Routes status raw`            | `ocp.critical.routes.status.raw`             | Collects OpenShift Routes that are not admitted.                                                                   |
| `OCP: Critical Service endpoints status raw` | `ocp.critical.services.endpoints.status.raw` | Collects Services with unavailable or not-ready endpoints.                                                         |
| `OCP: Etcd request latency p99`              | `ocp.etcd.request.latency.p99`               | Collects the cluster-level p99 etcd request latency.                                                               |
| `OCP: Thanos query health`                   | `ocp.thanos.query.health`                    | Checks whether the Thanos query endpoint is reachable and able to return a valid Prometheus API response.          |

## Discovery rules and item prototypes

| Discovery rule                                   | Key                                              | Created item prototypes |
| ------------------------------------------------ | ------------------------------------------------ | ----------------------- |
| `OCP Critical ClusterOperator Discovery`         | `ocp.critical.clusteroperator.discovery`         | ClusterOperator {#CO}: {#CO_CONDITION} |
| `OCP Critical Etcd Member Discovery`             | `ocp.critical.etcd.member.discovery`             | Etcd member {#ETCD_INSTANCE}: Critical state |
| `OCP Critical Job Status Discovery`              | `ocp.critical.job.status.discovery`              | Job {#NAMESPACE}/{#JOB}: Failed count<br>Job {#NAMESPACE}/{#JOB}: Running seconds |
| `OCP Critical Node Filesystem Discovery`         | `ocp.critical.node.filesystem.discovery`         | Node {#NODE_INSTANCE}: Filesystem {#FS_MOUNTPOINT} used % |
| `OCP Critical Node Status Discovery`             | `ocp.critical.node.status.discovery`             | Node {#NODE}: Not ready<br>Node {#NODE}: Unschedulable |
| `OCP Critical Pod CrashLoopBackOff Discovery`    | `ocp.critical.pod.crashloopbackoff.discovery`    | Pod {#NAMESPACE}/{#POD}: CrashLoopBackOff |
| `OCP Critical Pod Problem State Discovery`       | `ocp.critical.pod.problem_state.discovery`       | Pod {#NAMESPACE}/{#POD}: Problem state |
| `OCP Critical Pod Restarts Discovery`            | `ocp.critical.pod.restarts.discovery`            | Pod {#NAMESPACE}/{#POD}: Restarts in 15m |
| `OCP Critical PV Status Discovery`               | `ocp.critical.pv.status.discovery`               | PV {#PV}: Deletion timestamp<br>PV {#PV}: Failed |
| `OCP Critical PVC Usage Discovery`               | `ocp.critical.pvc.usage.discovery`               | PVC {#NAMESPACE}/{#PVC}: Used % |
| `OCP Critical ResourceQuota Usage Discovery`     | `ocp.critical.resourcequota.usage.discovery`     | ResourceQuota {#NAMESPACE}/{#RESOURCEQUOTA}/{#RESOURCE}: Used % |
| `OCP Critical Route Status Discovery`            | `ocp.critical.route.status.discovery`            | Route {#NAMESPACE}/{#ROUTE}: Not admitted |
| `OCP Critical Service Endpoint Status Discovery` | `ocp.critical.service.endpoint.status.discovery` | Service {#NAMESPACE}/{#SERVICE}: Not ready endpoints<br>Service {#NAMESPACE}/{#SERVICE}: No available endpoints |

## Value mappings

| Value map                         | Mapping                |
| --------------------------------- | ---------------------- |
| `OCP status: Healthy / Unhealthy` | 0=Unhealthy, 1=Healthy |
| `OCP status: OK / Problem`        | 0=OK, 1=Problem        |
| `OCP status: Up / Down`           | 0=Down, 1=Up           |
| `OCP status: Yes / No`            | 0=No, 1=Yes            |

## PromQL queries used by raw items

### Thanos health

```promql
vector(1)
```

### Etcd p99 latency

```promql
histogram_quantile(0.99, sum(rate(etcd_request_duration_seconds_bucket[10m])) by (le)) or vector(0)
```

### ClusterOperators

```promql
max by(name,condition) ((cluster_operator_conditions{condition="Available",status=~"False|Unknown"}==1) or (cluster_operator_conditions{condition=~"Degraded|Progressing",status="True"}==1))
```

### Etcd critical state

```promql
max by(instance)((up{job=~".*etcd.*"}==bool 0) or (etcd_server_has_leader==bool 0))>0
```

### Jobs failed or long-running

```promql
label_replace((kube_job_status_failed>0) or (((time()-kube_job_status_start_time) and on(namespace,job_name) (kube_job_status_active>0))>={$OCP.JOB.MAX.RUN.TIME}),"check","any","job_name",".*") or label_replace(kube_job_status_failed>0,"check","failed","job_name",".*") or label_replace(((time()-kube_job_status_start_time) and on(namespace,job_name) (kube_job_status_active>0))>={$OCP.JOB.MAX.RUN.TIME},"check","longrunning","job_name",".*")
```

### Node filesystem usage

```promql
100 * (1 - node_filesystem_avail_bytes/node_filesystem_size_bytes) >= {$OCP.NODE.FS.WARN}
```

### Nodes not ready or unschedulable

```promql
label_replace(max by (node) ((kube_node_status_condition{condition="Ready",status=~"false|unknown"} == 1) or (kube_node_spec_unschedulable == 1)), "check", "any", "node", ".*") or label_replace(max by (node) (kube_node_status_condition{condition="Ready",status=~"false|unknown"} == 1), "check", "not_ready", "node", ".*") or label_replace(max by (node) (kube_node_spec_unschedulable == 1), "check", "unschedulable", "node", ".*")
```

### Pods in problem state

```promql
max by(namespace,pod)(kube_pod_status_phase{phase=~"Pending|Failed|Unknown"} or kube_pod_status_unschedulable or (kube_pod_status_ready{condition="false"} unless on(namespace,pod) kube_pod_status_phase{phase="Succeeded"}))>0
```

### Pod CrashLoopBackOff

```promql
sum by(namespace,pod)(kube_pod_container_status_waiting_reason{reason="CrashLoopBackOff"})>0
```

### Pod restarts

```promql
sum by(namespace,pod)(increase(kube_pod_container_status_restarts_total[15m]))>={$OCP.POD.RESTARTS.WARN}
```

### PersistentVolumes

```promql
label_replace((kube_persistentvolume_status_phase{phase="Failed"}==1) or (kube_persistentvolume_deletion_timestamp>0),"check","any","persistentvolume",".*") or label_replace(kube_persistentvolume_status_phase{phase="Failed"}==1,"check","failed","persistentvolume",".*") or label_replace(kube_persistentvolume_deletion_timestamp>0,"check","terminating","persistentvolume",".*")
```

### PVC usage

```promql
100 * (kubelet_volume_stats_used_bytes / kubelet_volume_stats_capacity_bytes) >= {$OCP.PVC.WARN}
```

### ResourceQuota usage

```promql
((100 * kube_resourcequota{type="used"} / on(namespace,resourcequota,resource) kube_resourcequota{type="hard"}) and on(namespace,resourcequota,resource) (kube_resourcequota{type="hard"} > 0)) >= {$OCP.RQ.WARN}
```

### Routes not admitted

```promql
((openshift_route_status{type="Admitted",status="True"} == bool 0) == 1)
```

### Service endpoints

```promql
label_replace((kube_endpoint_info unless on(namespace,endpoint)(sum by(namespace,endpoint)(kube_endpoint_address_available)>0)) or (sum by(namespace,endpoint)(kube_endpoint_address_not_ready)>0),"check","any","endpoint",".*") or label_replace(kube_endpoint_info unless on(namespace,endpoint)(sum by(namespace,endpoint)(kube_endpoint_address_available)>0),"check","no_available","endpoint",".*") or label_replace(sum by(namespace,endpoint)(kube_endpoint_address_not_ready)>0,"check","not_ready","endpoint",".*")
```

---

## Testing queries manually

Example query test with `curl`:

```bash
curl -k -s -G "{$OCP.THANOS.URL}/api/v1/query" \
  -H "Authorization: Bearer <TOKEN>" \
  --data-urlencode 'query=vector(1)' | jq
```

Example to list all metrics exposed by Thanos Querier:

```bash
curl -k -s "{$OCP.THANOS.URL}/api/v1/metadata" \
  -H "Authorization: Bearer <TOKEN>" \
  | jq -r '.data | keys[]' | sort
```

Example to list firing alerts from the OpenShift monitoring stack:

```bash
curl -k -s -G "{$OCP.THANOS.URL}/api/v1/query" \
  -H "Authorization: Bearer <TOKEN>" \
  --data-urlencode 'query=ALERTS{alertstate="firing"}' | jq
```

## Notes

- Most discoveries are intentionally problem-oriented and may not create items when the cluster is healthy.
- Lost discovered resources are removed according to the `Keep lost resources` value configured on each discovery rule.
- The template expects specific OpenShift/Kubernetes metrics to be available from Thanos.
- Some environments may require adjusting the etcd selector from `job=~".*etcd.*"` to a namespace-based selector such as `namespace="openshift-etcd"`.
