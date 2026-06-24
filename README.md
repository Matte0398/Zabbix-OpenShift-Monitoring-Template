# Zabbix OpenShift Monitoring Template

![Zabbix](https://img.shields.io/badge/Zabbix-7.0-red)
![OpenShift](https://img.shields.io/badge/OpenShift-4.x-red)
![Thanos](https://img.shields.io/badge/Thanos-Query%20API-blue)
![Status](https://img.shields.io/badge/Status-Active-brightgreen)

Custom **Zabbix 7.0** template designed to monitor **OpenShift 4.x** clusters through the **Thanos / Prometheus Query API**.

The template uses **PromQL-based HTTP agent items**, **Zabbix preprocessing**, **dependent items** and **low-level discovery rules** to identify problematic or critical OpenShift resources.

Instead of discovering every object in the cluster, the template focuses on **problem-based discovery**, creating monitoring entities mainly for resources that match warning or critical conditions.

---

## Overview

Traditional infrastructure monitoring does not always provide enough visibility into OpenShift and Kubernetes environments.

This project was developed to provide a lightweight and scalable monitoring approach for OpenShift clusters using Zabbix and metrics exposed through the Thanos / Prometheus Query API.

The main goals are:

- Reduce the number of HTTP requests sent to the cluster
- Avoid creating unnecessary monitoring objects in Zabbix
- Discover only problematic or critical resources
- Provide actionable alerts for infrastructure and platform teams
- Keep the template flexible through user macros and namespace filters

---

## Architecture

The template is based on the following logic:

1. Zabbix collects raw data using **HTTP agent items**
2. HTTP items query the Thanos / Prometheus API using PromQL
3. Raw JSON responses are processed through **Zabbix preprocessing**
4. **Dependent items** extract specific values from the raw data
5. **Low-level discovery rules** create items and triggers only for problematic resources
6. Triggers generate alerts when critical conditions are detected

This approach helps reduce the number of direct API calls while keeping the monitoring scalable and focused on real problems.

---

## Why this project?

This project was created to address common challenges encountered when monitoring OpenShift environments:

- Limited visibility with traditional host-based monitoring
- High number of cluster objects that can generate noise
- Difficulty identifying only critical OpenShift resources
- Need for automated discovery without manually configuring every object
- Need to integrate OpenShift monitoring into Zabbix-based enterprise environments
- Need to reduce alert noise and focus only on actionable conditions

The objective is to provide a reusable Zabbix monitoring template suitable for enterprise OpenShift environments.

---

## Features

- OpenShift cluster monitoring through Thanos / Prometheus Query API
- HTTP agent items with Bearer token authentication
- PromQL-based monitoring logic
- Dependent items to reduce duplicated API calls
- Low-level discovery for problematic resources
- Problem-based monitoring approach
- Custom triggers and alerting logic
- Namespace filtering through user macros
- Threshold customization through user macros
- Support for OpenShift and Kubernetes platform metrics
- Native integration with Zabbix 7.0

---

## Monitoring Approach

This template does **not** aim to create a full inventory of all OpenShift objects.

The monitoring logic is focused on **critical or problematic resources**.

For example:

- It does not discover all pods
- It discovers pods only when they are in a problematic state or have excessive restarts
- It does not discover all PVCs
- It discovers PVCs only when usage is above the configured critical threshold
- It does not discover all services
- It discovers services only when endpoints are missing or not ready

This design reduces noise and helps Zabbix focus on actionable monitoring data.

---

## Requirements

- Zabbix 7.0 or later
- OpenShift 4.x
- Access to Thanos Querier or Prometheus Query API
- Bearer token with permissions to query monitoring metrics
- Network connectivity from Zabbix Server or Zabbix Proxy to the Thanos / Prometheus endpoint
- Metrics exposed by the OpenShift monitoring stack

Required metric sources may include:

- kube-state-metrics
- OpenShift route metrics
- node filesystem metrics
- kubelet volume metrics
- ResourceQuota metrics
- etcd metrics
- Thanos / Prometheus query endpoint

---

## Monitored Components

The template focuses on critical or problematic OpenShift resources, including:

| Component | Monitoring logic |
|---|---|
| Thanos Query API | Checks if the query endpoint is reachable and returns valid data |
| Nodes | Detects nodes in `NotReady`, `False` or `Unknown` state |
| Node filesystems | Detects filesystems above the configured critical usage threshold |
| Pods | Detects pods in problematic states such as `Pending`, `Failed`, `Unknown` or unschedulable |
| Pod restarts | Detects pods with excessive restarts in a 15-minute window |
| Services | Detects services without available endpoints |
| Service endpoints | Detects endpoints in not-ready state |
| Routes | Detects routes that are not admitted |
| Persistent Volumes | Detects failed PVs or PVs stuck in terminating state |
| Persistent Volume Claims | Detects PVCs above the configured critical usage threshold |
| Jobs | Detects failed jobs |
| Long-running jobs | Detects jobs running longer than the configured threshold |
| ResourceQuota | Detects ResourceQuota usage above the configured threshold |
| etcd | Detects etcd members without a leader |
| etcd latency | Monitors p99 etcd request latency |

---

## Template Items

The template includes raw HTTP agent items such as:

| Item | Description |
|---|---|
| `ocp.thanos.query.health` | Checks Thanos query API health |
| `ocp.critical.nodes.not_ready.raw` | Raw data for nodes not ready |
| `ocp.critical.node.filesystems.raw` | Raw data for critical node filesystem usage |
| `ocp.critical.pods.problem.raw` | Raw data for pods in problematic states |
| `ocp.critical.pods.restarts.raw` | Raw data for excessive pod restarts |
| `ocp.critical.services.no_available_endpoints.raw` | Raw data for services without available endpoints |
| `ocp.critical.services.not_ready_endpoints.raw` | Raw data for services with not-ready endpoints |
| `ocp.critical.routes.not_admitted.raw` | Raw data for routes not admitted |
| `ocp.critical.pv.failed.raw` | Raw data for failed Persistent Volumes |
| `ocp.critical.pv.terminating.raw` | Raw data for terminating Persistent Volumes |
| `ocp.critical.pvc.usage.raw` | Raw data for PVC usage above threshold |
| `ocp.critical.jobs.failed.raw` | Raw data for failed jobs |
| `ocp.critical.jobs.longrunning.raw` | Raw data for long-running jobs |
| `ocp.critical.resourcequota.usage.raw` | Raw data for ResourceQuota usage above threshold |
| `ocp.critical.etcd.no_leader.raw` | Raw data for etcd members without leader |
| `ocp.etcd.request.latency.p99` | etcd p99 request latency |

---

## Discovery Rules

The template uses dependent low-level discovery rules based on raw Prometheus / Thanos query results.

Main discovery rules include:

| Discovery rule | Description |
|---|---|
| `OCP Critical Node Not Ready Discovery` | Discovers nodes that are not ready |
| `OCP Critical Node Filesystem Discovery` | Discovers node filesystems above critical usage |
| `OCP Critical Pod Problem State Discovery` | Discovers pods in problematic states |
| `OCP Critical Pod Restarts Discovery` | Discovers pods with excessive restarts |
| `OCP Critical Service No Available Endpoint Discovery` | Discovers services without available endpoints |
| `OCP Critical Service Not Ready Endpoint Discovery` | Discovers services with not-ready endpoints |
| `OCP Critical Route Not Admitted Discovery` | Discovers routes that are not admitted |
| `OCP Critical PV Failed Discovery` | Discovers failed Persistent Volumes |
| `OCP Critical PV Terminating Discovery` | Discovers Persistent Volumes stuck in terminating state |
| `OCP Critical PVC Usage Discovery` | Discovers PVCs above critical usage threshold |
| `OCP Critical Failed Job Discovery` | Discovers failed jobs |
| `OCP Critical Long Running Job Discovery` | Discovers jobs running for too long |
| `OCP Critical ResourceQuota Usage Discovery` | Discovers ResourceQuota resources above critical usage |
| `OCP Critical Etcd No Leader Discovery` | Discovers etcd members without a leader |

---

## Required Macros

Before using the template, configure at least the following macros on the host or template level.

| Macro | Default / Example | Description |
|---|---|---|
| `{$OCP.THANOS.URL}` | `https://thanos-querier.example.com` | Thanos Querier base URL |
| `{$OCP.TOKEN}` | secret value | Bearer token used to query the Thanos / OpenShift monitoring API |

---

## Threshold Macros

| Macro | Default | Description |
|---|---:|---|
| `{$OCP.NODE.FS.CRIT}` | `90` | Critical threshold for node filesystem usage percentage |
| `{$OCP.PVC.CRIT}` | `90` | Critical threshold for PVC usage percentage |
| `{$OCP.POD.RESTARTS.WARN}` | `3` | Pod restart threshold over 15 minutes |
| `{$OCP.JOB.MAX.RUN.TIME}` | `1800` | Maximum job runtime in seconds before it is considered long-running |
| `{$OCP.RQ.CRIT}` | `90` | ResourceQuota usage threshold percentage |
| `{$OCP.ETCD.LATENCY.P99.WARN}` | `1` | Warning threshold in seconds for etcd p99 latency |

---

## Namespace Filter Macros

These macros allow you to include or exclude namespaces from discovery.

| Macro | Default | Description |
|---|---|---|
| `{$OCP.NS.NOT_MATCHES}` | `^$` | Global regex to exclude namespaces. `^$` excludes nothing |
| `{$OCP.POD.NS.MATCHES}` | `.*` | Regex for namespaces where pod discovery is enabled |
| `{$OCP.PVC.NS.MATCHES}` | `.*` | Regex for namespaces where PVC discovery is enabled |
| `{$OCP.JOB.NS.MATCHES}` | `.*` | Regex for namespaces where job discovery is enabled |
| `{$OCP.SVC.NS.MATCHES}` | `.*` | Regex for namespaces where service endpoint discovery is enabled |
| `{$OCP.ROUTE.NS.MATCHES}` | `.*` | Regex for namespaces where route discovery is enabled |
| `{$OCP.RQ.NS.MATCHES}` | `.*` | Regex for namespaces where ResourceQuota discovery is enabled |

Example:

```text
{$OCP.NS.NOT_MATCHES}=^(openshift-.*|kube-.*)$
```

This excludes namespaces starting with `openshift-` or `kube-`.

---

## Service Filter Macros

| Macro | Default | Description |
|---|---|---|
| `{$OCP.SVC.NOT_MATCHES}` | `^$` | Regex to exclude services or endpoints from service discovery |

Example:

```text
{$OCP.SVC.NOT_MATCHES}=^(kubernetes|openshift.*)$
```

---

## ResourceQuota Filter Macros

| Macro | Default | Description |
|---|---|---|
| `{$OCP.RQ.RESOURCE.MATCHES}` | `.*` | Regex for ResourceQuota resources to include |
| `{$OCP.RQ.RESOURCE.NOT_MATCHES}` | `^$` | Regex for ResourceQuota resources to exclude |

Example:

```text
{$OCP.RQ.RESOURCE.MATCHES}=^(requests.cpu|requests.memory|limits.cpu|limits.memory|pods)$
```

---

## Node Filesystem Filter Macros

The template allows filtering filesystems, mountpoints and devices to avoid monitoring temporary, virtual or container-related filesystems.

| Macro | Description |
|---|---|
| `{$OCP.NODE.FS.FSTYPE.NOT_MATCHES}` | Filesystem types to exclude from node filesystem checks |
| `{$OCP.NODE.FS.MOUNTPOINT.NOT_MATCHES}` | Mountpoints to exclude from node filesystem checks |
| `{$OCP.NODE.FS.DEVICE.NOT_MATCHES}` | Devices to exclude from node filesystem checks |

Default excluded filesystem types include examples such as:

```text
tmpfs
overlay
squashfs
nsfs
proc
sysfs
devtmpfs
cgroup
cgroup2
mqueue
hugetlbfs
securityfs
debugfs
tracefs
```

---

## Installation

1. Import the YAML template into Zabbix.

2. Create or select a Zabbix host that represents the OpenShift cluster.

3. Link the template to the host.

4. Configure the required macros:

```text
{$OCP.THANOS.URL}
{$OCP.TOKEN}
```

5. Adjust thresholds and namespace filters if needed.

6. Verify that the following item returns `1`:

```text
OCP: Thanos query health
```

7. Run or wait for low-level discovery rules.

8. Check that dependent items and trigger prototypes are created only for problematic resources.

---

## Authentication

The template uses Bearer token authentication.

The token is passed in the HTTP header:

```text
Authorization: Bearer {$OCP.TOKEN}
```

The token must be able to query the Thanos / Prometheus API endpoint used by the template.

The required permissions may depend on the OpenShift cluster configuration and how the Thanos Querier endpoint is exposed.

---

## Example Host Macros

Example minimal configuration:

```text
{$OCP.THANOS.URL}=https://thanos-querier-openshift-monitoring.apps.example.com
{$OCP.TOKEN}=xxxxxxxxxxxxxxxxxxxxxxxx
```

Example threshold customization:

```text
{$OCP.NODE.FS.CRIT}=90
{$OCP.PVC.CRIT}=90
{$OCP.POD.RESTARTS.WARN}=3
{$OCP.JOB.MAX.RUN.TIME}=1800
{$OCP.RQ.CRIT}=90
{$OCP.ETCD.LATENCY.P99.WARN}=1
```

Example namespace filtering:

```text
{$OCP.NS.NOT_MATCHES}=^(openshift-.*|kube-.*)$
{$OCP.POD.NS.MATCHES}=.*
{$OCP.PVC.NS.MATCHES}=.*
{$OCP.JOB.NS.MATCHES}=.*
{$OCP.SVC.NS.MATCHES}=.*
{$OCP.ROUTE.NS.MATCHES}=.*
{$OCP.RQ.NS.MATCHES}=.*
```

---

## Alerting Logic

The template includes triggers for conditions such as:

- Thanos Query API not healthy
- Node not ready
- Node filesystem usage above critical threshold
- Pod in problematic state
- Pod with too many restarts
- Service without available endpoints
- Service with not-ready endpoints
- Route not admitted
- Persistent Volume failed
- Persistent Volume stuck in terminating state
- PVC usage above critical threshold
- Failed job
- Long-running job
- ResourceQuota usage above threshold
- etcd member without leader
- High etcd request latency p99

Most triggers are configured with manual close enabled.

---

## Value Maps

The template includes value maps to improve readability of numeric statuses.

| Value map | Values |
|---|---|
| `OCP status: OK / Problem` | `0 = OK`, `1 = Problem` |
| `OCP status: Yes / No` | `0 = No`, `1 = Yes` |

---

## Troubleshooting

### Thanos health item returns `0`

Check:

- Thanos URL is correct
- Token is valid
- Zabbix Server or Proxy can reach the endpoint
- TLS certificate is trusted or correctly handled
- The `/api/v1/query` endpoint is available

### Discovery does not create objects

This can be expected behavior.

The template uses problem-based discovery, so objects are created only when the PromQL query returns problematic resources.

If there are no issues in the cluster, some discovery rules may return an empty result.

### HTTP item returns unauthorized

Check:

- Bearer token validity
- Token permissions
- Token expiration
- Correct macro value in `{$OCP.TOKEN}`

### JSONPath preprocessing fails

Check:

- The query returns a valid Prometheus API JSON response
- The response contains `data.result`
- The PromQL query is valid
- The metric exists in the cluster

### Some metrics are missing

Check whether the required metrics are exposed by the OpenShift monitoring stack.

Depending on the cluster configuration, some metrics may not be available.

---

## Design Notes

This template is intentionally focused on **critical object discovery**.

The goal is not to duplicate the entire OpenShift object inventory inside Zabbix, but to create monitoring objects only when a relevant problem is detected.

This reduces:

- Zabbix item count
- Discovery noise
- Trigger noise
- HTTP requests
- Operational overhead

---

## Limitations

- The template depends on metrics exposed through Thanos / Prometheus
- Some checks may not work if the related metrics are disabled or unavailable
- The template does not currently provide full inventory discovery for all OpenShift objects
- The template does not currently include dedicated CronJob discovery
- The template does not currently include dedicated Cluster Operator discovery
- Metric names and labels may vary depending on OpenShift version and monitoring configuration

---

## Repository Structure

```text
Zabbix-OpenShift-Monitoring-Template/
├── template_openshift.yaml
└── README.md
```

---

## Author

Matteo Z.

---

## License

This project is provided as an example template for OpenShift monitoring with Zabbix.

Use and adapt it according to your environment, monitoring standards and operational requirements.
