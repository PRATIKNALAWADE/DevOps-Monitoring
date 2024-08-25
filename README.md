# Ultimate DevOps Monitoring Project
---



Welcome to the Ultimate DevOps Monitoring Project! This repository contains the configuration and setup for a comprehensive monitoring stack using **Prometheus**, **Node Exporter**, **Alertmanager**, and **Blackbox Exporter**. The project demonstrates how to monitor systems and applications effectively, collect vital metrics, and set up alerting mechanisms to ensure system reliability.

## Table of Contents

- [Introduction](#introduction)
- [Project Architecture](#project-architecture)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Alerting](#alerting)
- [Contributing](#contributing)


## Introduction

This project is designed to provide hands-on experience with setting up and managing a robust monitoring infrastructure. It includes:

- **Node Exporter**: For collecting hardware and OS-level metrics.
- **Prometheus**: As the central monitoring system.
- **Alertmanager**: To handle alerts and notifications.
- **Blackbox Exporter**: For probing endpoints to check their availability and performance.

## Project Architecture

- **VM-1**: Hosts Node Exporter, which exposes system-level metrics to Prometheus.
- **VM-2**: Hosts Prometheus, Alertmanager, and Blackbox Exporter, responsible for scraping metrics, managing alerts, and probing endpoints.

## Prerequisites

Before you begin, ensure you have the following installed on both VMs:

- `wget`
- `tar`
- Appropriate permissions to download, extract, and execute binaries.

## Installation

### VM-1: Setting up Node Exporter

1. **Download Node Exporter**:
   ```bash
   wget https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
   ```

2. **Extract Node Exporter**:
   ```bash
   tar xvfz node_exporter-1.8.1.linux-amd64.tar.gz
   ```

3. **Start Node Exporter**:
   ```bash
   cd node_exporter-1.8.1.linux-amd64
   ./node_exporter &
   ```

### VM-2: Setting up Prometheus, Alertmanager, and Blackbox Exporter

#### Prometheus

1. **Download Prometheus**:
   ```bash
   wget https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-amd64.tar.gz
   ```

2. **Extract Prometheus**:
   ```bash
   tar xvfz prometheus-2.52.0.linux-amd64.tar.gz
   ```

3. **Start Prometheus**:
   ```bash
   cd prometheus-2.52.0.linux-amd64
   ./prometheus --config.file=prometheus.yml &
   ```

#### Alertmanager

1. **Download Alertmanager**:
   ```bash
   wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
   ```

2. **Extract Alertmanager**:
   ```bash
   tar xvfz alertmanager-0.27.0.linux-amd64.tar.gz
   ```

3. **Start Alertmanager**:
   ```bash
   cd alertmanager-0.27.0.linux-amd64
   ./alertmanager --config.file=alertmanager.yml &
   ```

#### Blackbox Exporter

1. **Download Blackbox Exporter**:
   ```bash
   wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz
   ```

2. **Extract Blackbox Exporter**:
   ```bash
   tar xvfz blackbox_exporter-0.25.0.linux-amd64.tar.gz
   ```

3. **Start Blackbox Exporter**:
   ```bash
   cd blackbox_exporter-0.25.0.linux-amd64
   ./blackbox_exporter &
   ```

## Configuration

### Prometheus Configuration (`prometheus.yml`)

Define your global settings, scrape intervals, and targets in the `prometheus.yml` file:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['<Node_Exporter_IP>:9100']

  - job_name: 'blackbox'
    metrics_path: /probe
    static_configs:
      - targets:
          - http://prometheus.io
          - https://prometheus.io
          - http://<Your_Target_IP>:8080/
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: <Blackbox_Exporter_IP>:9115
```

### Alertmanager Configuration (`alertmanager.yml`)

Set up alerting routes, receivers, and inhibition rules in `alertmanager.yml`:

```yaml
route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  receiver: 'email-notifications'

receivers:
  - name: 'email-notifications'
    email_configs:
      - to: your_email@example.com
        from: alert@example.com
        smarthost: smtp.gmail.com:587
        auth_username: your_email
        auth_password: "your_smtp_password"
        send_resolved: true

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

## Usage

Once everything is set up, you can start monitoring your systems. Navigate to Prometheus’s web UI (`http://<Prometheus_IP>:9090`) to explore metrics and set up alerts.

To test the Blackbox Exporter, navigate to `http://<Blackbox_Exporter_IP>:9115/probe?target=http://yourtarget.com`.

## Alerting

The project includes predefined alert rules in `alert_rules.yml` for common scenarios such as:

- **InstanceDown**: Triggered when a target is unreachable.
- **WebsiteDown**: Triggered when a website probe fails.
- **HostOutOfMemory**: Triggered when available memory is low.
- **HostOutOfDiskSpace**: Triggered when disk space is running out.

These alerts are configured to send notifications via Alertmanager.

## Contributing

Contributions are welcome! Please fork the repository and submit a pull request.

