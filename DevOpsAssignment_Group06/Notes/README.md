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

⚙️ How Components Work Together:
Node Exporter: Runs as an agent on target Linux/Windows systems to read hardware sensors and OS stats, exposing them as plain text at http://localhost:9100/metrics.

Prometheus: Pulls (scrapes) data from target endpoints periodically according to prometheus.yml configuration and stores it in its internal TSDB (Time Series Database).

Grafana: Connects to Prometheus/Loki via HTTP, sends queries (PromQL/LogQL), and converts raw metric arrays into interactive visual charts, gauges, and tables.

Module 3: Installation & Service Initialization

🐧 Linux (Ubuntu/Debian) Systemd Setup
Step 1: Install & Start Grafana Server

sudo apt-get update
sudo apt-get install -y apt-transport-https
sudo apt-get install -y grafana

# Start and enable on boot
sudo systemctl daemon-reload
sudo systemctl start grafana-server
sudo systemctl enable grafana-server

# Verify service running state
sudo systemctl status grafana-server

Step 2: Install Node Exporter
wget [https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz](https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz)
tar -xvf node_exporter-1.7.0.linux-amd64.tar.gz
cd node_exporter-1.7.0.linux-amd64
./node_exporter &

🐳 Docker Quick Deployment
# Create shared monitoring bridge network
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

Module 4: Prometheus Data Model & Scraping
🏷️ Metric Structure & Notation
Every metric stored in Prometheus follows this key-value format:
metric_name{label_key1="label_val1", label_key2="label_val2"} value timestamp

📊 The 4 Core Metric Types
Counter: A cumulative metric that only increases or resets to zero on restart.

Use case: Total HTTP requests received, system uptime seconds.

Gauge: A metric that can arbitrarily go up and down.

Use case: Current CPU usage, available RAM memory, temperature.

Histogram: Samples observations (usually durations or response sizes) and counts them in configurable buckets.

Use case: API response latency distribution.

Summary: Similar to histogram, but calculates configurable quantiles over a sliding time window.

Module 5: PromQL Deep Dive & Query Mechanics
PromQL (Prometheus Query Language) is a functional query language designed for evaluating time-series data.

🎯 Key Operators & Functions
rate(): Calculates per-second average rate of increase over a time window (used on Counters).

irate(): Instantaneous rate calculated using the last two points (better for rapid spikes).

avg by (label): Aggregates values grouped by a specific dimension.

sum by (label): Sums values across instances.

🧠 Production PromQL Cheatsheet
1. System CPU Utilization Percentage
100 - (avg by (instance) (
  rate(node_cpu_seconds_total{mode="idle"}[5m])
) * 100)
Explanation: Subtracts idle time percentage from 100 to get active usage percentage averaged over 5-minute windows.

2. System RAM Usage Percentage
(100 - ((
  node_memory_MemAvailable_bytes * 100
) / node_memory_MemTotal_bytes))
Explanation: Measures available memory against total memory to calculate used memory %.

3. Disk Storage Usage Percentage
100 - ((
  node_filesystem_avail_bytes{mountpoint="/"} * 100
) / node_filesystem_size_bytes{mountpoint="/"})

Explanation: Checks root (/) partition available bytes against total disk capacity.

4. Network Transmit & Receive Rate (KB/s)
# Download Speed (Bytes/sec)
rate(node_network_receive_bytes_total[5m]) / 1024

# Upload Speed (Bytes/sec)
rate(node_network_transmit_bytes_total[5m]) / 1024

5. HTTP Error Rate Percentage (5xx Errors)
(sum(rate(http_requests_total{status=~"5.."}[5m]))
/ sum(rate(http_requests_total[5m]))) * 100

## Module 6: Building Grafana Dashboards & Panels

### 🎨 Selecting Panel Types

* **Time Series:** Standard line or bar graph over time. Best used for tracking CPU and Memory trends over 24 hours.
* **Stat:** Single large numerical value callout. Best used for showing real-time metrics like current CPU % or online user count.
* **Gauge:** Speedometer-style visual indicator. Best used for safety threshold limits (Green, Yellow, Red).
* **Bar Chart:** Categorical comparative chart. Best used for comparing storage usage across different disk drives.
* **Table:** Multi-column structured grid. Best used for listing active node IPs, system versions, and server uptimes.
* **Logs:** Scrollable raw text panel. Best used for viewing real-time log streams.

---

### 🛠️ Dashboard Creation Step-by-Step

1. Click **Dashboards** ➔ **New** ➔ **New Dashboard**.
2. Click **Add Visualization** ➔ Select **Prometheus** Data Source.
3. Paste your PromQL query into the **Code Panel**.
4. Set the **Panel Title** in the right-side configuration bar.
5. Set the **Unit** (for example: `Percent (0-100)`, `Bytes`, or `Requests/sec`).
6. Configure **Thresholds** (Base = Green, 70 = Yellow, 85 = Red).
7. Save the Dashboard (`Ctrl + S`).

---

## Module 7: Dynamic Variables & Templating

To filter data dynamically using dropdowns instead of creating separate dashboards for every server:

### 🔧 Setting Up Node Variable
1. Go to **Dashboard Settings** (Gear Icon) ➔ **Variables** ➔ **Add Variable**.
2. **Name:** `node`
3. **Type:** `Query`
4. **Data Source:** `Prometheus`
5. **Query Definition:**
   ```promql
   label_values(
     node_cpu_seconds_total, 
     instance
   )

Module 8: Alerting Architecture & Notification Channels
🚨 The Grafana Alerting Lifecycle
[ Data Query ]
      │
      ▼
[ Evaluate Rule ]
      │
      ▼
[ Pending Buffer (5m) ]
      │
      ▼
[ Firing State ]
      │
      ▼
[ Discord / Slack Alert ]

