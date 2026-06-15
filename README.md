![Zabbix](https://img.shields.io/badge/Zabbix-7.0-red)
![OpenShift](https://img.shields.io/badge/OpenShift-4.x-red)
![Status](https://img.shields.io/badge/Status-Active-brightgreen)

# Zabbix OpenShift Monitoring Template

A custom Zabbix template designed to monitor OpenShift clusters through REST API integrations, providing advanced discovery mechanisms, health monitoring, service visibility and proactive alerting for enterprise containerized environments.

## Overview

The standard monitoring approaches available for Kubernetes and OpenShift do not always provide the level of visibility required in enterprise environments.

This project was developed to provide a flexible and scalable monitoring solution for OpenShift clusters using Zabbix, leveraging OpenShift and Prometheus/Thanos APIs to collect critical information about cluster health and resources.

The template is designed to reduce manual configuration efforts through automated discovery mechanisms and dynamic monitoring workflows.

## Why this project?

This project was created to address several common challenges encountered when monitoring OpenShift environments:

- Limited visibility using traditional infrastructure monitoring approaches
- High number of manually configured monitoring objects
- Difficulty identifying critical cluster components
- Need for automated discovery and monitoring workflows
- Integration requirements between OpenShift and Zabbix

The objective was to create a reusable and scalable monitoring solution suitable for enterprise environments.

## Features

- OpenShift cluster monitoring through REST APIs
- Automated resource discovery
- Dynamic creation of monitoring entities
- Custom triggers and alerting logic
- Health monitoring of critical cluster components
- Support for enterprise-scale environments
- Reduced manual configuration effort
- Native integration with Zabbix

## Monitored Components

The template can monitor several OpenShift resources, including:

- Nodes
- Pods
- Services
- Routes
- Persistent Volumes
- Persistent Volume Claims
- Jobs
- CronJobs
- Cluster Operators
- etcd components

Additional resources can be added through custom discovery rules.

## Installation

1. Import the template into Zabbix.
2. Create an API user on OpenShift.
3. Generate an API token.
4. Configure the required template macros.
5. Link the template to the target host.
6. Verify data collection and discovery execution.

## Required Macros

| Macro | Description |
|---------|-------------|
| {$OPENSHIFT.URL} | OpenShift API URL |
| {$OPENSHIFT.TOKEN} | API Authentication Token |
| {$OPENSHIFT.NAMESPACE} | Namespace filter (optional) |
