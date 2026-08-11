# 📓 Complete Observability & Monitoring Notes
> **Subject:** System Monitoring, Metrics Analytics & Log Management  
> **Toolstack:** Grafana | Prometheus | Node Exporter | Loki | PromQL  
> **Author & Student Notes:** Group 06  

---

## 📑 Table of Contents
1. [Module 1: Introduction to Observability vs Monitoring](#module-1-introduction-to-observability-vs-monitoring)
2. [Module 2: Complete Architecture & Data Flow](#module-2-complete-architecture--data-flow)
3. [Module 3: Installation & Service Initialization](#module-3-installation--service-initialization)
4. [Module 4: Prometheus Data Model & Scraping](#module-4-prometheus-data-model--scraping)
5. [Module 5: PromQL Deep Dive & Query Mechanics](#module-5-promql-deep-dive--query-mechanics)
6. [Module 6: Building Grafana Dashboards & Panels](#module-6-building-grafana-dashboards--panels)
7. [Module 7: Dynamic Variables & Templating](#module-7-dynamic-variables--templating)
8. [Module 8: Alerting Architecture & Notification Channels](#module-8-alerting-architecture--notification-channels)
9. [Module 9: Log Aggregation with Grafana Loki](#module-9-log-aggregation-with-grafana-loki)
10. [Module 10: Viva, Interview & Troubleshooting Guide](#module-10-viva-interview--troubleshooting-guide)
11. [Module 11: Official Study Reference Docs & PDFs](#module-11-official-study-reference-docs--pdfs)

---

## Module 1: Introduction to Observability vs Monitoring

### 💡 What is Monitoring?
Monitoring is the process of collecting, analyzing, and using information to track a system's performance and status. It answers **"Is the system working?"** and **"What is broken?"**.

* **Key Focus:** Known unknowns (pre-defined metrics like CPU %, Disk Space, Memory).
* **Core Goal:** Alerting operators when thresholds are breached.

### 🔬 What is Observability?
Observability is a property of a system that allows us to infer its internal state based on its external outputs. It answers **"Why is the system broken?"**.

* **Key Focus:** Unknown unknowns (debugging novel edge-case failures).
* **The 3 Pillars of Observability (MEL):**
  1. **Metrics:** Numeric values measured over time intervals (Counter, Gauge, Histogram, Summary).
  2. **Events/Logs:** Time-stamped text records of discrete events (e.g., error log lines).
  3. **Traces:** End-to-end request lifecycle paths across microservices.

---

## Module 2: Complete Architecture & Data Flow

### 🏗️ Complete Pipeline Schematic
```text
┌────────────────────────────────────────────────────────┐
│                    TARGET HOSTS                        │
│                                                        │
│  ┌───────────────────────┐   ┌──────────────────────┐  │
│  │     Node Exporter     │   │      Promtail        │  │
│  │     (Port 9100)       │   │  (Log Collector)     │  │
│  └───────────┬───────────┘   └──────────┬───────────┘  │
└──────────────│──────────────────────────│──────────────┘
               │                          │
        HTTP Pull /metrics         HTTP Push Logs
               │                          │
               ▼                          ▼
┌──────────────────────────┐   ┌──────────────────────────┐
│        Prometheus        │   │       Grafana Loki       │
│  Time Series DB (9090)   │   │   Log Engine (3100)      │
└──────────────┬───────────┘   └──────────┬───────────────┘
               │                          │
               │    PromQL / LogQL        │
               └────────────┬─────────────┘
                            │
                            ▼
               ┌──────────────────────────┐
               │    GRAFANA DASHBOARD     │
               │       (Port 3000)        │
               └──────────────────────────┘
⚙️ How Components Work Together:Node Exporter: Runs as an agent on target Linux/Windows systems to read hardware sensors and OS stats, exposing them as plain text at http://localhost:9100/metrics.Prometheus: Pulls (scrapes) data from target endpoints periodically according to prometheus.yml configuration and stores it in its internal TSDB (Time Series Database).Grafana: Connects to Prometheus/Loki via HTTP, sends queries (PromQL/LogQL), and converts raw metric arrays into interactive visual charts, gauges, and tables.Module 3: Installation & Service Initialization🐧 Linux (Ubuntu/Debian) Systemd SetupStep 1: Install & Start Grafana ServerBashsudo apt-get update
sudo apt-get install -y apt-transport-https
sudo apt-get install -y grafana

# Start and enable on boot
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl enable grafana-server

# Verify service running state
sudo systemctl status grafana-server
Step 2: Install Node ExporterBashwget [https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz](https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz)
tar -xvf node_exporter-1.7.0.linux-amd64.tar.gz
cd node_exporter-1.7.0.linux-amd64
./node_exporter &
🐳 Docker Quick DeploymentBash# Create shared monitoring bridge network
docker network create monitoring

# Run Prometheus
docker run -d \
  --name=prometheus \
  --network=monitoring \
  -p 9090:9090 \
  prom/prometheus

# Run Grafana
docker run -d \
  --name=grafana \
  --network=monitoring \
  -p 3000:3000 \
  grafana/grafana
Module 4: Prometheus Data Model & Scraping🏷️ Metric Structure & NotationEvery metric stored in Prometheus follows this key-value format:Plaintextmetric_name{label_key1="label_val1", label_key2="label_val2"} value timestamp
Example:node_cpu_seconds_total{cpu="0", mode="idle"} 12840.42📊 The 4 Core Metric TypesCounter: A cumulative metric that only increases or resets to zero on restart.Use case: Total HTTP requests received, system uptime seconds.Gauge: A metric that can arbitrarily go up and down.Use case: Current CPU usage, available RAM memory, temperature.Histogram: Samples observations (usually durations or response sizes) and counts them in configurable buckets.Use case: API response latency distribution.Summary: Similar to histogram, but calculates configurable quantiles over a sliding time window.Module 5: PromQL Deep Dive & Query MechanicsPromQL (Prometheus Query Language) is a functional query language designed for evaluating time-series data.🎯 Key Operators & Functionsrate(): Calculates per-second average rate of increase over a time window (used on Counters).irate(): Instantaneous rate calculated using the last two points (better for rapid spikes).avg by (label): Aggregates values grouped by a specific dimension.sum by (label): Sums values across instances.🧠 Production PromQL Cheatsheet1. System CPU Utilization PercentageCode snippet100 - (avg by (instance) (
  rate(node_cpu_seconds_total{mode="idle"}[5m])
) * 100)
Explanation: Subtracts idle time percentage from 100 to get active usage percentage averaged over 5-minute windows.2. System RAM Usage PercentageCode snippet(100 - ((
  node_memory_MemAvailable_bytes * 100
) / node_memory_MemTotal_bytes))
Explanation: Measures available memory against total memory to calculate used memory %.3. Disk Storage Usage PercentageCode snippet100 - ((
  node_filesystem_avail_bytes{mountpoint="/"} * 100
) / node_filesystem_size_bytes{mountpoint="/"})
Explanation: Checks root (/) partition available bytes against total disk capacity.4. Network Transmit & Receive Rate (KB/s)Code snippet# Download Speed (Bytes/sec)
rate(node_network_receive_bytes_total[5m]) / 1024

# Upload Speed (Bytes/sec)
rate(node_network_transmit_bytes_total[5m]) / 1024
5. HTTP Error Rate Percentage (5xx Errors)Code snippet(sum(rate(http_requests_total{status=~"5.."}[5m]))
/ sum(rate(http_requests_total[5m]))) * 100
Module 6: Building Grafana Dashboards & Panels🎨 Selecting Panel TypesPanel TypeDescriptionWhen to UseTime SeriesStandard line/bar graph over timeResource usage tracking (CPU, RAM over 24h)StatSingle big numerical valueReal-time current CPU %, active online usersGaugeSpeedometer-style visualHighlighting safety bounds (Green/Yellow/Red)Bar ChartCategorical comparative chartComparing storage used across different disksTableMulti-column gridListing active node IPs, OS versions, uptimeLogsScrollable text panelViewing application/system log outputs🛠️ Dashboard Creation ChecklistClick Dashboards $\rightarrow$ New $\rightarrow$ New Dashboard.Click Add Visualization $\rightarrow$ Select Prometheus Data Source.Paste PromQL query into Code Panel.Set Panel Title in Right Sidebar.Set Unit (e.g., Percent (0-100), Bytes, Requests/sec).Configure Thresholds (e.g., Base=Green, 70=Yellow, 85=Red).Save Dashboard (Ctrl + S).Module 7: Dynamic Variables & TemplatingTo avoid creating separate dashboards for every single server, Grafana uses Variables to filter data dynamically via dropdowns.🔧 Setting Up Node VariableGo to Dashboard Settings (Gear Icon) $\rightarrow$ Variables $\rightarrow$ Add Variable.Name: nodeType: QueryData Source: PrometheusQuery Definition:Code snippetlabel_values(node_cpu_seconds_total, instance)
Usage inside Panel Queries:Code snippetnode_memory_MemAvailable_bytes{instance="$node"}
Module 8: Alerting Architecture & Notification Channels🚨 The Grafana Alerting LifecyclePlaintext[ Data Query ] ──> [ Evaluate Rule ] ──> [ Pending Buffer ] ──> [ Firing State ] ──> [ Notification ]
State - Normal: Query value is within threshold boundaries.State - Pending: Query breached threshold, but waiting out the For buffer duration (e.g., 5m) to prevent false alarms due to transient spikes.State - Firing: Threshold remained breached past the buffer time; alert trigger active.🔔 Setting Up Discord / Slack WebhookGo to Alerting $\rightarrow$ Contact Points $\rightarrow$ Add Contact Point.Select Integration Type: Discord (or Slack/Email).Paste Webhook URL generated from Discord channel settings.Click Test Contact Point to verify connectivity.Save Contact Point.Module 9: Log Aggregation with Grafana LokiLoki is Grafana’s log aggregation system inspired by Prometheus. It indexes metadata labels instead of full text contents, making it lightweight.📜 Basic LogQL SyntaxLogQL is the query language for Loki.Stream Selector:Code snippet{app="nginx", environment="production"}
Filter Expressions:Code snippet{app="nginx"} |= "error"
Exclude Filter:Code snippet{app="nginx"} != "debug"
Module 10: Viva, Interview & Troubleshooting Guide❓ Top Viva Questions & Precise AnswersQ1: What is the default port for Grafana, Prometheus, and Node Exporter?Grafana: 3000Prometheus: 9090Node Exporter: 9100Q2: Does Prometheus push metrics or pull metrics?Answer: Prometheus uses a Pull model (Scraping HTTP /metrics endpoints).Q3: What is the popular community pre-built Node Exporter Dashboard ID?Answer: Dashboard ID 1860 (Node Exporter Full).Q4: What is the difference between rate() and irate() in PromQL?Answer: rate() averages increments over an entire time window (smoother), while irate() looks only at the last two data points (captures sudden spikes).Q5: Why is Loki more storage-efficient than Elasticsearch?Answer: Loki only indexes labels/metadata rather than full-text indexing the entire log payload.Module 11: Official Study Reference Docs & PDFs🔗 Official Grafana Documentation🔗 Official Prometheus Overview Guide🔗 Grafana Getting Started Guide (PDF)🔗 Grafana Official Dashboards Library
